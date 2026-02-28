# Docker Cheatsheet

> Quick-reference for building, running, and debugging containers.

## 🔥 Most Used

| Command | What it does | Example |
|---|---|---|
| `docker build -t name .` | Build image from Dockerfile | `docker build -t myapp:v1 .` |
| `docker run -d -p H:C img` | Run container detached with port mapping | `docker run -d -p 8080:80 nginx` |
| `docker ps` | List running containers | `docker ps -a` (include stopped) |
| `docker logs -f <ctr>` | Follow container logs | `docker logs -f --tail 100 myapp` |
| `docker exec -it <ctr> sh` | Shell into running container | `docker exec -it myapp /bin/bash` |
| `docker compose up -d` | Start all services in background | `docker compose up -d --build` |
| `docker stop <ctr>` | Stop a container | `docker stop myapp` |
| `docker rm <ctr>` | Remove a container | `docker rm -f myapp` |

## Build

| Command | What it does | Example |
|---|---|---|
| `docker build -t name:tag .` | Build with tag | `docker build -t api:v2 .` |
| `docker build -f <file> .` | Build from custom Dockerfile | `docker build -f Dockerfile.prod .` |
| `docker build --no-cache .` | Build without layer cache | `docker build --no-cache -t app .` |
| `docker build --build-arg K=V .` | Pass build-time variable | `docker build --build-arg ENV=prod .` |
| `docker build --target <stage> .` | Build specific stage (multi-stage) | `docker build --target builder -t app-build .` |
| `docker buildx build --platform <p>` | Cross-platform build | `docker buildx build --platform linux/amd64,linux/arm64 -t app .` |

### Common Build Flags

| Flag | Purpose |
|---|---|
| `-t name:tag` | Name and tag the image |
| `-f <file>` | Specify Dockerfile path |
| `--no-cache` | Ignore layer cache |
| `--build-arg` | Inject build-time env vars |
| `--target` | Build up to a specific stage |
| `--progress=plain` | Show full build output (useful for debugging) |

## Run

| Command | What it does | Example |
|---|---|---|
| `docker run -d <img>` | Detached mode (background) | `docker run -d nginx` |
| `docker run -it <img> sh` | Interactive shell | `docker run -it ubuntu bash` |
| `docker run --rm <img>` | Auto-remove on exit | `docker run --rm alpine echo hi` |
| `docker run -p H:C` | Map host:container ports | `docker run -p 3000:3000 app` |
| `docker run -v H:C` | Bind mount host dir | `docker run -v $(pwd):/app node` |
| `docker run --name <n>` | Assign container name | `docker run --name db postgres` |
| `docker run -e KEY=VAL` | Set env variable | `docker run -e DB_HOST=localhost app` |
| `docker run --env-file .env` | Load env from file | `docker run --env-file .env app` |
| `docker run --network <net>` | Attach to network | `docker run --network mynet app` |
| `docker run --restart unless-stopped` | Restart policy | `docker run --restart always nginx` |

## Debug

| Command | What it does | Example |
|---|---|---|
| `docker logs <ctr>` | View container logs | `docker logs --tail 50 myapp` |
| `docker logs -f <ctr>` | Follow logs in real-time | `docker logs -f myapp` |
| `docker exec -it <ctr> sh` | Exec into running container | `docker exec -it myapp /bin/sh` |
| `docker inspect <ctr>` | Full container metadata (JSON) | `docker inspect myapp \| jq '.[0].NetworkSettings'` |
| `docker top <ctr>` | Processes in container | `docker top myapp` |
| `docker stats` | Live resource usage | `docker stats --no-stream` |
| `docker diff <ctr>` | Changed files in container | `docker diff myapp` |
| `docker cp <ctr>:/path ./local` | Copy file out of container | `docker cp myapp:/app/log.txt .` |
| `docker events` | Real-time Docker daemon events | `docker events --filter container=myapp` |

### Debug a Failing Container

```bash
# 1. Check exit code and logs
docker ps -a | grep myapp
docker logs myapp

# 2. Run interactively to debug
docker run -it --entrypoint sh myapp:v1

# 3. Inspect filesystem of stopped container
docker cp myapp:/app/config.toml ./debug-config.toml
```

## Images

| Command | What it does | Example |
|---|---|---|
| `docker images` | List local images | `docker images --filter dangling=true` |
| `docker pull <img>` | Pull from registry | `docker pull rust:1.75-slim` |
| `docker push <img>` | Push to registry | `docker push registry.io/app:v1` |
| `docker tag <src> <dst>` | Tag an image | `docker tag app:latest reg.io/app:v1` |
| `docker rmi <img>` | Remove image | `docker rmi app:old` |
| `docker history <img>` | Show image layers | `docker history --no-trunc app` |
| `docker save -o f.tar <img>` | Export image to tarball | `docker save -o app.tar app:v1` |
| `docker load -i f.tar` | Import image from tarball | `docker load -i app.tar` |

## Cleanup

| Command | What it does | Example |
|---|---|---|
| `docker system prune` | Remove unused data | `docker system prune -a --volumes` |
| `docker container prune` | Remove stopped containers | `docker container prune -f` |
| `docker image prune` | Remove dangling images | `docker image prune -a` |
| `docker volume prune` | Remove unused volumes | `docker volume prune -f` |
| `docker network prune` | Remove unused networks | `docker network prune -f` |

## Multi-stage Build Tips

```dockerfile
# Stage 1: Build
FROM rust:1.75 AS builder
WORKDIR /app
COPY . .
RUN cargo build --release

# Stage 2: Minimal runtime
FROM debian:bookworm-slim
COPY --from=builder /app/target/release/myapp /usr/local/bin/
CMD ["myapp"]
```

| Tip | Why |
|---|---|
| Use `AS builder` to name stages | Readable + lets you `--target builder` |
| Copy only the binary in final stage | Smaller image, no build tools |
| Use `slim` / `alpine` base for runtime | Minimize attack surface and size |
| Put rarely-changing layers first | Better cache hits |

## Local Kubernetes Image Workflows (minikube/kind)

```bash
# Minikube: build directly in minikube's Docker daemon
eval $(minikube docker-env)
docker build -t greeting-service:latest .

# OR build locally and load into minikube
docker build -t greeting-service:latest .
minikube image load greeting-service:latest

# kind: load local image into kind cluster nodes
docker build -t greeting-service:latest .
kind load docker-image greeting-service:latest --name kind
```

| Tip | Why |
|---|---|
| Keep image tag stable (`:latest`) while learning | Fewer manifest edits during fast iteration |
| Use `imagePullPolicy: Never` for local-only images | Prevents `ImagePullBackOff` |
| Switch to full registry paths + `Always` for remote clusters | Matches production pull behavior |

## Compose Basics

| Command | What it does | Example |
|---|---|---|
| `docker compose up` | Start services | `docker compose up -d` |
| `docker compose down` | Stop & remove services | `docker compose down -v` (+ volumes) |
| `docker compose build` | Rebuild services | `docker compose build --no-cache` |
| `docker compose ps` | List running services | `docker compose ps` |
| `docker compose logs` | View service logs | `docker compose logs -f api` |
| `docker compose exec <svc> sh` | Shell into service | `docker compose exec db psql -U postgres` |
| `docker compose pull` | Pull latest images | `docker compose pull` |
| `docker compose restart <svc>` | Restart a service | `docker compose restart api` |
| `docker compose config` | Validate & show merged config | `docker compose config` |

---

*See [Kubernetes 101](../tutorials/rust-k8s-operator/README.md#part-3-kubernetes-101--deploy-with-kubectl) for deploying containers to K8s.*
