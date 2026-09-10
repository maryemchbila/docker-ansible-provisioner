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

## Declarative application deployment

Application deployment is described in `vars/app_config.yml`. The file names
the container and image, declares its host and container ports, and selects a
restart policy. `playbooks/provision.yml` loads that specification and passes
it to the reusable `app_deploy` role after the security baseline and Docker
Engine have been configured.

The role validates the specification, pulls the requested image, and ensures
the configured container is running with the requested port mapping and
restart policy. It uses `community.docker.docker_container` instead of
`docker run` or shell commands so Ansible compares the desired container
configuration with its current state and changes it only when necessary.

To reuse the role for another application later, supply the same `application`
mapping with a different name, image, ports, and restart policy. The role
contains no NGINX-specific values. This lab intentionally deploys only the
single demonstration application for now.

Run the provisioner from the repository root:

```bash
ansible-playbook playbooks/provision.yml
```

## Status

Work in Progress
