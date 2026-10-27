# Fixing broken Dockerfiles

Each Dockerfile below builds the app but has one problem. For each one: what we see, why it happens, and how we fix it.

## 1. Works inside the container, but not from the browser

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "127.0.0.1", "--port", "8000"]
```

**What we see.** `curl http://127.0.0.1:8000/health` from the host gives `Empty reply from server` or a connection reset, and the browser shows an error. Inside the container the same request works:

```
docker exec b1 python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:8000/health').read())"
```

**Why.** `127.0.0.1` inside a container is the container's own loopback interface. Docker delivers the published port (`-p 8000:8000`) to the container's network address, where nothing is listening.

**Fix.** Listen on all interfaces:

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

After the fix, `curl` from the host returns `{"status":"ok"}`.

## 2. `docker stop` waits 10 seconds

```dockerfile
CMD uvicorn app.main:app --host 0.0.0.0 --port 8000
```

**What we see.** `time docker stop b2` takes about 10 seconds. `docker top b2` shows `/bin/sh -c uvicorn app.main:app ...` as the first process.

**Why.** The shell form of `CMD` runs the command through `/bin/sh -c`, so the shell is process 1. `docker stop` sends SIGTERM to process 1, and the shell may not pass it on to uvicorn. After 10 seconds Docker gives up and sends SIGKILL.

**Fix.** Use the exec form, so uvicorn is process 1 and receives the signal itself:

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

After the fix, `docker top` shows uvicorn as the first process and `docker stop` finishes in about a second.

## 3. Every small code change reinstalls everything

```dockerfile
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
```

**What we see.** After changing `app/main.py` and rebuilding, the `pip install` step runs again instead of being marked `CACHED`, and the build is slow.

**Why.** Docker caches each instruction as a layer. `COPY . .` comes before `pip install`, so any changed file invalidates that layer and every layer after it, including the installation of the dependencies.

**Fix.** Copy `requirements.txt` first, install, and copy the code last:

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app ./app
```

Now a change in `app/main.py` only rebuilds the last layer, and `pip install` is taken from the cache. A `.dockerignore` file (`.git`, `.venv`, `tests`, `docs`) also keeps unneeded files out of the build.

## 4. A secret inside the image

```dockerfile
ENV API_TOKEN=s3cr3t-token-123
```

**What we see.** The token is readable without running the container:

```
docker history --no-trunc broken4 | grep API_TOKEN
docker inspect --format '{{.Config.Env}}' broken4
```

Both commands print `API_TOKEN=s3cr3t-token-123`.

**Why.** `ENV` is stored in the image metadata and in the layer history. Anyone who can pull or load the image can read it, and removing the variable in a later instruction does not remove it from the history.

**Fix.** Remove the `ENV` line from the Dockerfile and pass the value when the container starts:

```
docker run -d -e API_TOKEN=<value> notes-api
docker run -d --env-file .env notes-api
```

Keep `.env` out of Git and out of the image (`.gitignore` and `.dockerignore`). A token that was ever stored in an image should be treated as leaked and replaced with a new one.