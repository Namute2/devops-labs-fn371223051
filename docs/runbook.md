# Runbook

> Fill in every TODO with exact commands. / Попълнете всяко TODO с точни команди.

## Start locally / Локално стартиране

Python 3.12 is required.

```bash
python3.12 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements-dev.txt

uvicorn app.main:app --reload
```

The app runs at http://127.0.0.1:8000. Without `DATABASE_URL`, notes are kept in memory and lost on restart.

## Check that it works / Проверка, че работи

Which address do you open and what should you see? / кой адрес отваряте и какво трябва да видите?

Open http://127.0.0.1:8000/docs. You should see the generated API documentation page listing the endpoints.

Or check from the terminal:

```bash
curl http://127.0.0.1:8000/health
```

Expected: `{"status": "ok"}`

## Run the tests / Тестове

```bash
pytest
ruff check .
```

`pytest` runs the tests in `tests/`, and `ruff check .` runs the style check. Both need `requirements-dev.txt` installed.

## When something breaks / Когато нещо се счупи

The first three things you check. / първите три неща, които проверявате.

1. Check the health endpoints: `curl -i http://127.0.0.1:8000/health` should return 200 with `{"status": "ok"}`, and `curl -i http://127.0.0.1:8000/ready` should not return 503 (503 means the database cannot be reached, so check `DATABASE_URL` and that PostgreSQL is running).
2. Check the environment: run `python --version` (must be 3.12), make sure the virtualenv is active with `source .venv/bin/activate`, and reinstall with `pip install -r requirements-dev.txt`.
3. Check the port and the logs: read the first error in the terminal running `uvicorn`. If port 8000 is taken, find the process with `lsof -i :8000`, or start on another port with `uvicorn app.main:app --reload --port 8001`.
## Run with Docker

The image is tagged `notes-api:0.3.0`.

Build:

```
docker build -t notes-api:0.3.0 .
```

Run (notes are kept in memory and lost when the container stops):

```
docker run -d --name notes -p 8000:8000 -e APP_VERSION=0.3.0 notes-api:0.3.0
```

Check that it works:

```
curl http://127.0.0.1:8000/health
curl http://127.0.0.1:8000/version
```

To keep notes in PostgreSQL, add `-e DATABASE_URL=postgresql://notes:secret@db:5432/notes` (see Lab 4). Without a reachable database, `/ready` returns 503.

Logs:

```
docker logs notes
docker logs -f notes
```

Stop and remove:

```
docker stop notes
docker rm notes
```