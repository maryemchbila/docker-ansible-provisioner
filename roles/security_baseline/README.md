# Security Baseline Role

## Purpose

The `security_baseline` role prepares a minimal Ubuntu application container by
installing basic administration packages, creating an unprivileged application
account, and creating a protected application directory.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `security_baseline_packages` | `ca-certificates`, `curl` | Minimal packages installed for trusted HTTPS access and basic administration. |
| `app_user` | `appuser` | Name of the non-root application user. |
| `app_group` | `appgroup` | Primary group for the application user and directory. |
| `app_home` | `/home/appuser` | Home directory assigned to the application user. |
| `app_directory` | `/opt/apps` | Directory in which application files can be placed. |

Override these defaults in inventory variables or playbook variables when using
the role in another environment.

## Security decisions

The application runs under a dedicated non-root account. Its login shell is
`/usr/sbin/nologin`, because the account is intended to run applications rather
than provide interactive access. The application directory is owned by the
application account and uses mode `0750`, granting full access to the owner,
read and execute access to the group, and no access to other users.

## Docker-lab limitations

The target containers do not run SSH or systemd. This role therefore does not
configure SSH, a firewall, or systemd-managed services. The Docker connection
runs as root, so the playbook does not use privilege escalation (`become`).
