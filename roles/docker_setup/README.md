# Docker Setup Role

## Purpose

The `docker_setup` role installs Docker Engine from Ubuntu's `docker.io`
package, grants configured application users access to the `docker` group,
starts the daemon when necessary, waits for it to become ready, and verifies
availability with `docker info`.

The role is intended to run after `security_baseline`, which creates the
application user referenced by `docker_setup_users`.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `docker_setup_prerequisite_packages` | `ca-certificates`, `curl` | OS packages used for trusted HTTPS access and administration. |
| `docker_setup_packages` | `docker.io` | Ubuntu packages that provide Docker Engine and its runtime dependencies. |
| `docker_setup_users` | The value of `app_user`, or `appuser` | Existing users added to the `docker` group. |
| `docker_setup_daemon_binary` | `/usr/bin/dockerd` | Docker daemon executable. |
| `docker_setup_daemon_log` | `/var/log/dockerd.log` | File receiving daemon output. |
| `docker_setup_daemon_retries` | `30` | Maximum number of Docker readiness checks. |
| `docker_setup_daemon_retry_delay` | `2` | Seconds between readiness checks. |

Membership in the `docker` group effectively grants root-level control over
the Docker host. Only trusted application users should be included in
`docker_setup_users`.

## Why systemd is not used

The target containers are intentionally minimal and do not run systemd as PID
1. The role therefore cannot use `ansible.builtin.service` to manage Docker.
It checks `docker info` first and invokes `dockerd` in the background only when
the daemon is unavailable. A later `docker info` task retries until the daemon
responds successfully.

## Docker-in-Docker limitation

The daemon runs inside a privileged target container and shares the host
kernel. Its state lasts only as long as the target container unless storage is
configured separately. Starting the daemon this way also does not provide the
lifecycle supervision that a real init system would provide.

Privileged containers have broad host access and substantially weakened
isolation. This architecture is suitable only for a local educational lab; it
must not be treated as a production Docker deployment or security pattern.
