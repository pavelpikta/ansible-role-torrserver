# Ansible Role: TorrServer

An Ansible role that installs and configures [TorrServer](https://github.com/YouROK/TorrServer) MatriX on Linux systems.

![GitHub License](https://img.shields.io/github/license/pavelpikta/ansible-role-torrserver?style=flat&label=License)
[![CI](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/ci.yml/badge.svg)](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/ci.yml)
[![Release](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/release.yml/badge.svg)](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/release.yml)
[![GitHub Tag](https://img.shields.io/github/v/tag/pavelpikta/ansible-role-torrserver?sort=semver&style=flat&label=Release)](https://github.com/pavelpikta/ansible-role-torrserver/tags)

## Features

- Installs specific or latest version of TorrServer.
- Configures systemd service.
- Optional BBR (Bottleneck Bandwidth and Round-trip propagation time) congestion control optimization.
- Support for multiple architectures (amd64, arm64, arm7, arm5, 386).
- HTTP Authentication support.
- Read-only database mode support.

## Requirements

- Ansible version: `2.13.0` or higher.
- Collections:
  - `community.general`
  - `ansible.posix`

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_version` | `latest` | TorrServer version to install (e.g., `133`, `latest`). |
| `torrserver_user` | `torrserver` | System user for TorrServer. |
| `torrserver_group` | `torrserver` | System group for TorrServer. |
| `torrserver_install_dir` | `/opt/torrserver` | Directory where TorrServer will be installed. |
| `torrserver_port` | `8090` | Port for TorrServer web interface. |
| `torrserver_service_name` | `torrserver` | Name of the systemd service. |
| `torrserver_read_only` | `false` | Enable read-only database mode (`--rdb`). |
| `torrserver_enable_log` | `false` | Enable logging to file. |
| `torrserver_log_path` | `{{ torrserver_install_dir }}/{{ torrserver_service_name }}.log` | Path to log file. |
| `torrserver_enable_auth` | `false` | Enable HTTP Authentication. |
| `torrserver_auth_users` | `{ torrserver: torrserver }` | Dictionary of users and passwords for authentication. |
| `torrserver_enable_bbr` | `true` | Enable BBR congestion control for better streaming performance. |

### Advanced Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_bin_prefix` | `TorrServer-linux` | Binary filename prefix. |
| `torrserver_repo_url` | `https://github.com/YouROK/TorrServer` | GitHub repository URL. |
| `torrserver_api_url` | `https://api.github.com/repos/YouROK/TorrServer` | GitHub API URL. |

## Dependencies

None.

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: pavelpikta.torrserver
      vars:
        torrserver_version: latest
        torrserver_port: 8090
        torrserver_enable_auth: true
        torrserver_auth_users:
          admin: "your_secure_password"
```

## Supported Platforms

- Debian (all versions)
- Ubuntu (all versions)
- EL (Enterprise Linux) 7, 8, 9
- Fedora (all versions)
- ArchLinux (all versions)

## License

This project is licensed under the [Apache License 2.0](LICENSE).

## Author Information

Created by [pavelpikta](https://github.com/pavelpikta).
