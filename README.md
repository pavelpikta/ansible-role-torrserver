# Ansible Role: TorrServer

An Ansible role that installs and configures [TorrServer](https://github.com/YouROK/TorrServer) MatriX on Linux systems.

![GitHub License](https://img.shields.io/github/license/pavelpikta/ansible-role-torrserver?style=flat&label=License)
[![CI](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/ci.yml/badge.svg)](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/ci.yml)
[![Release](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/release.yml/badge.svg)](https://github.com/pavelpikta/ansible-role-torrserver/actions/workflows/release.yml)
[![GitHub Tag](https://img.shields.io/github/v/tag/pavelpikta/ansible-role-torrserver?sort=semver&style=flat&label=Release)](https://github.com/pavelpikta/ansible-role-torrserver/tags)

## Features

- Installs specific or latest version of TorrServer from GitHub releases.
- Install TorrServer from a local file (copy from control node to target host).
- Configures systemd service.
- Optional BBR (Bottleneck Bandwidth and Round-trip propagation time) congestion control optimization.
- Support for multiple architectures (amd64, arm64, arm7, arm5, 386).
- HTTP Authentication support.
- Read-only database mode support.
- BitTorr settings configuration via `settings.json` with merge support.

## Requirements

- Ansible version: `2.13.0` or higher.
- Collections:
  - `community.general`
  - `ansible.posix`

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_version` | `latest` | TorrServer version to install (e.g., `133`, `latest`). Ignored when `torrserver_local_file` is set. |
| `torrserver_local_file` | `null` | Path to local TorrServer binary file on the control node. If set, the role will copy this file instead of downloading from GitHub. |
| `torrserver_user` | `torrserver` | System user for TorrServer. |
| `torrserver_group` | `torrserver` | System group for TorrServer. |
| `torrserver_install_dir` | `/opt/torrserver` | Directory where TorrServer will be installed. |
| `torrserver_port` | `8090` | Port for TorrServer web interface (`--port`). |
| `torrserver_service_name` | `torrserver` | Name of the systemd service. |
| `torrserver_read_only` | `false` | Enable read-only database mode (`--rdb`). |
| `torrserver_enable_log` | `false` | Enable logging to file. |
| `torrserver_log_path` | `{{ torrserver_install_dir }}/{{ torrserver_service_name }}.log` | Path to log file (`--logpath`). |
| `torrserver_weblog_path` | `null` | Path to web access log file (`--weblogpath`). |
| `torrserver_enable_auth` | `false` | Enable HTTP Authentication (`--httpauth`). |
| `torrserver_auth_users` | `{ torrserver: torrserver }` | Dictionary of users and passwords for authentication. |
| `torrserver_ssl_enable` | `false` | Enable HTTPS for web server (`--ssl`). |
| `torrserver_ssl_port` | `8091` | HTTPS port for web server (`--sslport`). |
| `torrserver_ssl_cert_path` | `null` | Path to SSL certificate file (`--sslcert`). |
| `torrserver_ssl_key_path` | `null` | Path to SSL key file (`--sslkey`). |
| `torrserver_dontkill` | `false` | Don't kill server on signal (`--dontkill`). |
| `torrserver_torrents_dir` | `null` | Autoload torrents from directory (`--torrentsdir`). |
| `torrserver_torrent_addr` | `null` | Torrent client address, format `[IP]:PORT` (`--torrentaddr`). |
| `torrserver_pubipv4` | `null` | Set public IPv4 address (`--pubipv4`). |
| `torrserver_pubipv6` | `null` | Set public IPv6 address (`--pubipv6`). |
| `torrserver_searchwa` | `false` | Allow search without authentication (`--searchwa`). |
| `torrserver_enable_bbr` | `true` | Enable BBR congestion control for better streaming performance. |

### BitTorr Settings

The role supports configuration of BitTorr settings via `settings.json`. You can override any default setting by defining `torrserver_bittorr_settings` in your playbook. The role will merge your custom settings with the defaults.

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_bittorr_settings` | `{}` | Dictionary of BitTorr settings to override. See below for available settings. |
| `torrserver_bittorr_defaults` | See `defaults/main.yml` | Default BitTorr settings (internal use). |

**Available BitTorr Settings:**

- `CacheSize` (integer): Cache size in bytes. Default: `67108864` (64 MB)
- `ConnectionsLimit` (integer): Maximum number of connections. Default: `25`
- `DisableDHT` (boolean): Disable DHT. Default: `false`
- `DisablePEX` (boolean): Disable PEX. Default: `false`
- `DisableTCP` (boolean): Disable TCP. Default: `false`
- `DisableUPNP` (boolean): Disable UPNP. Default: `false`
- `DisableUTP` (boolean): Disable UTP. Default: `false`
- `DisableUpload` (boolean): Disable upload. Default: `false`
- `DownloadRateLimit` (integer): Download rate limit in bytes/sec (0 = unlimited). Default: `0`
- `EnableDLNA` (boolean): Enable DLNA. Default: `false`
- `EnableDebug` (boolean): Enable debug mode. Default: `false`
- `EnableIPv6` (boolean): Enable IPv6. Default: `false`
- `EnableRutorSearch` (boolean): Enable Rutor search. Default: `false`
- `EnableTorznabSearch` (boolean): Enable Torznab search. Default: `false`
- `ForceEncrypt` (boolean): Force encryption. Default: `false`
- `FriendlyName` (string): Friendly name. Default: `""`
- `PeersListenPort` (integer): Peers listen port (0 = auto). Default: `0`
- `PreloadCache` (integer): Preload cache percentage. Default: `50`
- `ReaderReadAHead` (integer): Reader read ahead percentage. Default: `95`
- `RemoveCacheOnDrop` (boolean): Remove cache on drop. Default: `false`
- `ResponsiveMode` (boolean): Responsive mode. Default: `true`
- `RetrackersMode` (integer): Retrackers mode. Default: `1`
- `ShowFSActiveTorr` (boolean): Show filesystem active torrents. Default: `true`
- `SslCert` (string): SSL certificate path. Default: `""`
- `SslKey` (string): SSL key path. Default: `""`
- `SslPort` (integer): SSL port. Default: `0`
- `StoreSettingsInJson` (boolean): Store settings in JSON. Default: `true`
- `StoreViewedInJson` (boolean): Store viewed in JSON. Default: `false`
- `TorrentDisconnectTimeout` (integer): Torrent disconnect timeout in seconds. Default: `30`
- `TorrentsSavePath` (string): Torrents save path. Default: `""`
- `TorznabUrls` (list/null): List of Torznab URLs. Default: `null`
- `UploadRateLimit` (integer): Upload rate limit in bytes/sec (0 = unlimited). Default: `0`
- `UseDisk` (boolean): Use disk for cache. Default: `false`

**TorznabUrls format:**

```yaml
TorznabUrls:
  - Host: "https://example.com/api/torznab/"
    Key: "your-api-key"
    Name: "jackett"
```

### Advanced Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_bin_prefix` | `TorrServer-linux` | Binary filename prefix. |
| `torrserver_repo_url` | `https://github.com/YouROK/TorrServer` | GitHub repository URL. Only used for remote installation. |
| `torrserver_api_url` | `https://api.github.com/repos/YouROK/TorrServer` | GitHub API URL. Only used for remote installation. |

## Dependencies

None.

## Example Playbook

### Basic Example

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

### Example with Local File Installation

```yaml
- hosts: all
  roles:
    - role: pavelpikta.torrserver
      vars:
        # Use local file instead of downloading from GitHub
        torrserver_local_file: "{{ playbook_dir }}/files/TorrServer-linux-amd64"
        torrserver_port: 8090
        torrserver_enable_auth: true
        torrserver_auth_users:
          admin: "your_secure_password"
```

**Note:** When using `torrserver_local_file`, the `torrserver_version` variable is ignored. The role will copy the specified file to the target host. Make sure the file exists on the Ansible control node and matches the target architecture.

### Example with BitTorr Settings

```yaml
- hosts: all
  roles:
    - role: pavelpikta.torrserver
      vars:
        torrserver_version: latest
        torrserver_port: 8090
        torrserver_bittorr_settings:
          CacheSize: 268435456  # 256 MB in bytes
          ConnectionsLimit: 50
          EnableTorznabSearch: true
          ForceEncrypt: true
          PeersListenPort: 32000
          TorznabUrls:
            - Host: "https://jackett.example.com/api/v2.0/indexers/all/results/torznab/"
              Key: "your-api-key"
              Name: "jackett"
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
