# Docker Ansible Provisioner

A reusable and idempotent provisioning lab that uses Ansible and Docker containers as simulated target hosts.

## Technologies

- Ansible
- Docker
- YAML
- Git

## Docker Lab

This is a local educational Docker-in-Docker lab. Each simulated target runs
in privileged mode so that a nested Docker daemon can access the kernel
features it requires. Privileged containers have broad access to the host and
must not be treated as a production security pattern. Use isolated hosts and a
properly designed container runtime architecture for production workloads.

Build the shared target image and start all three containers:

```bash
docker compose -f docker/docker-compose.yml up --build -d
```

List the lab containers:

```bash
docker compose -f docker/docker-compose.yml ps
```

Enter a container (replace `ansible-node1` with another node when needed):

```bash
docker exec -it ansible-node1 bash
```

Stop the lab without removing its containers:

```bash
docker compose -f docker/docker-compose.yml stop
```

Destroy the lab containers and network:

```bash
docker compose -f docker/docker-compose.yml down
```

## Status

Work in Progress
