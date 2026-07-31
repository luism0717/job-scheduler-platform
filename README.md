# Setup

This will show you the enviornment/tools setup

## VSCODE

`${USER_REPO_PATH}/scheduler_system/.vscode` provides the setting to setup vs code.

- `extensions.json`
- `settings.json`

## install `kubectl`

Run:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

> **NOTE:** needed to install `kubectl` via curl cause WSL is clunky

If you'd still rather use snap run: `sudo snap install kubectl --classic`

## Docker Desktop configure with WSL/Cluster

1. Open Docker Desktop on Windows
2. Go to `Settings -> Resources -> WSL Integration`
3. Go to `Settings -> Kubernetes -> Enable Kubernetes -> Apply`
4. Make sure `Enable integration with my default WSL distro` is on, and toggle on the switch for
   your specific distro (looks like Ubuntu or similar to desktop name distro)
5. Click Apply & Restart
6. Back in your WSL terminal, run docker info to confirm it connects. If it hangs,
   fully close Docker Desktop and reopen it
7. `kubectl cluster-info --context docker-desktop`
8. Confirm context is active: `kubectl config current-context` should show `docker-desktop`

## Local infra containers (Docker Desktop)

**Postgres** run:
`docker run -d --name pg -e POSTGRES_PASSWORD=devpass -p 5432:5432 postgres:16`

**Keycloak** run:
`docker run -d --name keycloak -p 8081:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev`

**Kafka** run:

```bash
docker run -d --name kafka \
  -p 9092:9092 \
  -e KAFKA_NODE_ID=0 \
  -e KAFKA_PROCESS_ROLES=broker,controller \
  -e KAFKA_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
  -e KAFKA_CONTROLLER_LISTENER_NAMES=CONTROLLER \
  -e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT \
  -e KAFKA_CONTROLLER_QUORUM_VOTERS=0@localhost:9093 \
  apache/kafka:latest
```

**ActiveMQ** run:
`docker run -d --name activemq -p 61616:61616 -p 8161:8161 apache/activemq-classic:latest`

Confirm all containers are running `docker ps` or via Docker Desktop

## Github CI

CI runs via GitHub Actions (`.github/workflows/ci.yml`) on every push and PR to `main` Two jobs run in parallel:

- python - installs dependencies from `requirements.txt` / `requirements-dev.txt` if present, runs `pytest`
- java - builds and tests with Maven (mvn -B verify)

> **NOTE:**Hosted GitHub runners are used, so no local runner registration is needed.
