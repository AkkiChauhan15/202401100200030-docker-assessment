# Evidence runbook

The personalized values are REGNO `202401100200030`, PORT `8030`, and Docker Hub
username `akkichauhan15`. Run these commands yourself. Screenshots must be
unedited, show the full window, show the REGNO prompt, and use the listed filename.
The verified local Trivy and cosign binaries are under the ignored `.tools/` folder:

```sh
export PATH="$PWD/.tools:$PATH"
```

## Stage 1

Temporarily restore the Stage 1 base image and requirements from Git history, then:

```sh
docker build -t fest-backend:0.1 backend
docker image ls fest-backend:0.1
# S1-2.png

docker run -d --name backend -p "${PORT}:5000" -e REGNO="${REGNO}" fest-backend:0.1
docker ps --filter name=backend
curl "http://localhost:${PORT}/api/health"
docker logs backend
docker rm -f backend
# S1-3.png
```

## Stage 2

```sh
docker compose -f compose.yaml -f compose.build.yaml up -d --build
docker compose exec frontend ping -c 1 db
# S2-2.png (failure proves network isolation)

# Add two registrations in the browser at http://localhost:<PORT>.
# S2-3.png

docker compose down
docker compose -f compose.yaml -f compose.build.yaml up -d
docker volume ls
# Reload the browser and capture both retained registrations as S2-4.png.
```

## Stage 3

```sh
mkdir -p reports
trivy image fest-backend:0.1 | tee reports/trivy-before.txt
# S3-1.png

docker build --build-arg APP_VERSION=0.2 -t fest-backend:0.2 backend
trivy image fest-backend:0.2 | tee reports/trivy-after.txt
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
echo $?
# S3-3.png
```

## Stage 4

```sh
docker build --build-arg APP_VERSION=1.0.0 -t "${DOCKERHUB_USER}/fest-backend:1.0.0" backend
docker tag "${DOCKERHUB_USER}/fest-backend:1.0.0" "${DOCKERHUB_USER}/fest-backend:1.0"
docker push "${DOCKERHUB_USER}/fest-backend:1.0.0"
docker push "${DOCKERHUB_USER}/fest-backend:1.0"
# S4-1.png; capture the Docker Hub Tags page as S4-2.png.

docker image rm "${DOCKERHUB_USER}/fest-backend:1.0.0" \
  "${DOCKERHUB_USER}/fest-backend:1.0" fest-backend:0.2 fest-backend:0.1
docker compose up -d
# Reload the page; capture backend 1.0.0 and retained rows as S4-3.png.
```

## Stage 5

```sh
cosign generate-key-pair
# Commit cosign.pub only. Never commit cosign.key. Capture S5-1.png.

DIGEST=$(docker inspect --format='{{index .RepoDigests 0}}' \
  "${DOCKERHUB_USER}/fest-backend:1.0.0")
cosign sign --key cosign.key "$DIGEST"
# S5-2.png
cosign verify --key cosign.pub "$DIGEST"
# S5-3.png

docker build --build-arg APP_VERSION=1.0.1 -t "${DOCKERHUB_USER}/fest-backend:1.0.1" backend
docker push "${DOCKERHUB_USER}/fest-backend:1.0.1"
UNSIGNED_DIGEST=$(docker inspect --format='{{index .RepoDigests 0}}' \
  "${DOCKERHUB_USER}/fest-backend:1.0.1")
cosign verify --key cosign.pub "$UNSIGNED_DIGEST"
# The failure is required evidence: S5-4.png.
```
