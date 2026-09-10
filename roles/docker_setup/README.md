# Docker Setup Role

## Purpose

The `docker_setup` role installs Docker Engine from Ubuntu's `docker.io`
package, grants configured application users access to the `docker` group,
configures the nested daemon to use VFS storage, starts the daemon when
necessary, waits for it to become ready, and verifies availability with
`docker info`.

The role is intended to run after `security_baseline`, which creates the
application user referenced by `docker_setup_users`.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `docker_setup_prerequisite_packages` | `ca-certificates`, `curl`, `procps`, `python3-docker` | OS packages used for trusted HTTPS access, manual daemon process management, and the Python Docker SDK required by Ansible's Docker modules. |
| `docker_setup_packages` | `docker.io` | Ubuntu packages that provide Docker Engine and its runtime dependencies. |
| `docker_setup_users` | The value of `app_user`, or `appuser` | Existing users added to the `docker` group. |
| `docker_setup_daemon_binary` | `/usr/bin/dockerd` | Docker daemon executable. |
| `docker_setup_daemon_log` | `/var/log/dockerd.log` | File receiving daemon output. |
| `docker_setup_daemon_retries` | `30` | Maximum number of Docker readiness checks. |
| `docker_setup_daemon_retry_delay` | `2` | Seconds between readiness checks. |
| `docker_setup_daemon_stop_timeout` | `30` | Maximum seconds to wait for the Docker socket to disappear during a configuration restart. |

Membership in the `docker` group effectively grants root-level control over
the Docker host. Only trusted application users should be included in
`docker_setup_users`.

## Why systemd is not used

The target containers are intentionally minimal and do not run systemd as PID
1. The role therefore cannot use `ansible.builtin.service` to manage Docker.
It checks `docker info` first and invokes `dockerd` in the background only when
the daemon is unavailable. A later `docker info` task retries until the daemon
responds successfully.

The role manages `/etc/docker/daemon.json` before checking or starting the
daemon. If that file changes, handlers stop the manually managed `dockerd`
process, wait for its socket to disappear, and start it again. If the file is
already correct and the daemon is running, no restart occurs. The daemon
status and readiness checks always report `changed: false`.

## Nested storage driver

The inner daemon uses Docker's `vfs` storage driver and disables the
containerd snapshotter. This avoids attempting to mount an overlay filesystem
on top of Docker Desktop/WSL2's outer container storage, which can fail with
`invalid argument` in this nested lab architecture.

VFS performs full copies of filesystem layers. It therefore uses more disk
space and generally performs worse than overlay-based storage. It is selected
only to make this privileged, educational Docker-in-Docker lab portable; it
is not the recommended storage configuration for production Docker hosts.

## Docker-in-Docker limitation

The daemon runs inside a privileged target container and shares the host
kernel. Its state lasts only as long as the target container unless storage is
configured separately. Starting the daemon this way also does not provide the
lifecycle supervision that a real init system would provide.

Privileged containers have broad host access and substantially weakened
isolation. This architecture is suitable only for a local educational lab; it
must not be treated as a production Docker deployment or security pattern.
