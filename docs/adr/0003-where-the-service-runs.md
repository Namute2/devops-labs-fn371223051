# 0002. Where the service runs

**Status / Статус:** Proposed

## Context / Контекст

Notes API is a small FastAPI service that will be packaged as a container and, in later labs, deployed automatically by a pipeline. We need to decide where it runs so that users can reach it, deployments are repeatable, and a rollback is possible if a release goes wrong. The repository already has an `infra/servers` folder for describing servers.

## Options / Варианти

**Option 1: A virtual machine running Docker**

- Pros: simple and close to how many teams start; full control over the server; the pipeline can deploy by pulling a new container image; easy to explain and debug in a course.
- Cons: we maintain the operating system, updates and security ourselves; scaling and recovery are manual.

**Option 2: A managed platform (PaaS)**

- Pros: the provider handles servers, updates and scaling; fast to deploy; built-in health checks and rollbacks.
- Cons: less control and less visibility into how deployment works; the free tier may be limited; risk of tying the setup to one provider.

**Option 3: Kubernetes**

- Pros: automatic restarts, scaling and rolling updates; a standard in larger organizations.
- Cons: a lot of complexity for one small service; a long setup time that takes attention away from the course goals.

## Decision / Решение

We run the service as a container on a virtual machine with Docker. The server is described in `infra/servers`, and the pipeline deploys by pulling the new image and restarting the container.

## Consequences / Последствия

- We are responsible for updating the server and keeping it secure.
- Deployment is simple to understand, and rollback means starting the previous image version.
- The `/health` and `/ready` endpoints are used to check the service after each deployment.
- If the service grows, we may need to revisit this decision and consider a managed platform or Kubernetes.