# Application Deployment Role

## Purpose

`app_deploy` deploys one Dockerized application from a declarative
`application` mapping. It validates the input, pulls the requested image, and
ensures the named container is running with the requested published port and
restart policy. Application-specific values are not hardcoded in the role.

## Variables

| Variable | Required/default | Description |
| --- | --- | --- |
| `application.name` | Required | Docker container name. |
| `application.image` | Required | Image name and tag to deploy. |
| `application.host_port` | Required | Port published on the target node. |
| `application.container_port` | Required | Port exposed by the application container. |
| `application.restart_policy` | Required | Docker restart policy: `no`, `on-failure`, `always`, or `unless-stopped`. |
| `app_deploy_pull_image` | `true` | Whether to ensure the requested image is pulled before container deployment. |

## Idempotence

The role uses `community.docker.docker_image` and
`community.docker.docker_container`. These modules inspect Docker's current
state and report a change only when the image or container must be created or
updated. Re-running the role with an already-current image and a matching,
running container leaves it unchanged.

## Reuse

Another application can reuse this role by providing the same `application`
mapping with different values. For example, a future configuration can select
a different image and ports without changing the role tasks. Deploying
multiple applications in one run would require an explicit future extension;
the current interface intentionally handles one application specification.
