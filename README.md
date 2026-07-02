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
- Proxy, WebDAV, FUSE, Telegram bot, and HTTPS redirect support.
- Version-aware CLI and BitTorr configuration based on installed TorrServer release.
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
| `torrserver_ip` | `null` | Bind address for the web server (`--ip`). |
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
| `torrserver_max_size` | `null` | Maximum allowed stream size in bytes (`--maxsize`). |
| `torrserver_tg_token` | `null` | Telegram bot token (`--tgtoken`). |
| `torrserver_tg_config` | `{}` | Telegram bot settings for `tg.cfg`. See [Telegram configuration](#telegram-configuration). |
| `torrserver_fuse_path` | `null` | FUSE mount path (`--fusepath`). |
| `torrserver_webdav` | `false` | Enable WebDAV (`--webdav`). |
| `torrserver_ui` | `false` | Open TorrServer page in browser on start (`--ui`). |
| `torrserver_proxy_url` | `null` | Proxy URL for BitTorrent traffic (`--proxyurl`). Supports `http`, `socks4`, `socks5`, and `socks5h`. |
| `torrserver_proxy_mode` | `null` | Proxy mode (`--proxymode`): `tracker` (HTTP trackers only, default), `peers` (peer connections only), or `full` (all traffic). |
| `torrserver_force_https` | `false` | Redirect all HTTP requests to HTTPS (`--force-https`). Requires `torrserver_ssl_enable: true`. |
| `torrserver_enable_bbr` | `true` | Enable BBR congestion control for better streaming performance. |
| `torrserver_version_strict` | `false` | Fail when configured options require a newer TorrServer than the effective installed version. When `false`, unsupported options are skipped with a warning. |

### Version compatibility

The role detects the effective TorrServer version from the installed binary (or the install target during first run) and only applies CLI flags and BitTorr settings supported by that version. Unsupported options configured in your playbook are skipped by default; set `torrserver_version_strict: true` to fail instead.

| Minimum version | CLI flags | BitTorr settings |
|-----------------|-----------|------------------|
| MatriX.130 | `ssl`, `searchwa`, `maxsize` | — |
| MatriX.136 | `ip`, `tgtoken` | `ResponsiveMode` |
| MatriX.137 | `fusepath`, `webdav`, `proxyurl`, `proxymode` | `EnableTorznabSearch`, `TorznabUrls`, `ShowFSActiveTorr`, `StoreSettingsInJson`, `StoreViewedInJson` |
| MatriX.138 | — | `TMDBSettings`, `EnableProxy`, `ProxyHosts` |
| MatriX.141.10 | `force-https` | `EnableLPD`, `LPDIPv6`, `TrackTimecode` |

All other documented CLI flags and BitTorr settings are available on older MatriX releases. Version requirements are defined in `vars/main.yml`.

### BitTorr Settings

The role supports configuration of BitTorr settings via `settings.json`. Set only the keys you want to change in `torrserver_bittorr_settings`; the role merges them with built-in defaults.

Use `torrserver_bittorr_settings_host` in `host_vars` for per-host partial overrides. It is merged on top of `torrserver_bittorr_settings` without replacing the group dictionary. This is required because Ansible replaces whole dictionaries across inventory levels — without `_host`, a host would lose group settings.

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_bittorr_settings` | `{}` | BitTorr settings to override. See below for available settings. |
| `torrserver_bittorr_settings_host` | `{}` | Per-host partial overrides merged on top of `torrserver_bittorr_settings`. |

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
- `EnableLPD` (boolean): Enable Local Peer Discovery. Default: `true`
- `LPDIPv6` (boolean): Enable LPD over IPv6. Default: `false`
- `TrackTimecode` (boolean): Store playback position (timecode) in viewed data. Default: `false`
- `EnableProxy` (boolean): Enable P2P proxy for configured hosts. Default: `false`
- `ProxyHosts` (list): Host patterns routed through the P2P proxy. Default: `["*themoviedb.org", "*tmdb.org", "rutor.info"]`
- `TMDBSettings` (object): TMDB integration settings. Default: see below

**TMDBSettings format:**

```yaml
TMDBSettings:
  APIKey: "your-tmdb-api-key"
  APIURL: "https://api.themoviedb.org"
  ImageURL: "https://image.tmdb.org"
  ImageURLRu: "https://imagetmdb.com"
```

**TorznabUrls format:**

```yaml
TorznabUrls:
  - Host: "https://example.com/api/torznab/"
    Key: "your-api-key"
    Name: "jackett"
```

### Telegram configuration

When `torrserver_tg_token` is set, the role deploys `tg.cfg` in the TorrServer data directory. Set only the keys you want to override in `torrserver_tg_config`.

| Field | Default | Description |
|-------|---------|-------------|
| `HostTG` | `https://api.telegram.org` | Telegram Bot API base URL |
| `HostWeb` | `""` | Base URL for stream links (auto-detected if empty) |
| `WhiteIds` | `[]` | Allowed Telegram user IDs (empty = allow all) |
| `BlackIds` | `[]` | Blocked Telegram user IDs |
| `Socks5` | `""` | Optional SOCKS5 proxy for Telegram API access (e.g. `socks5://user:pass@host:port`) |

**Example:**

```yaml
torrserver_tg_token: "123456:ABC-DEF..."
torrserver_tg_config:
  HostWeb: "https://ts.devsecops.stream"
  WhiteIds:
    - 129836428
  Socks5: "socks5://user:pass@116.202.26.49:48130"
```

The `tg.cfg` file is written with mode `0600` because it may contain proxy credentials.

### Advanced Variables

| Variable | Default Value | Description |
|----------|---------------|-------------|
| `torrserver_bin_prefix` | `TorrServer-linux` | Binary filename prefix. |
| `torrserver_repo_url` | `https://github.com/YouROK/TorrServer` | GitHub repository URL. Only used for remote installation. |
| `torrserver_api_url` | `https://api.github.com/repos/YouROK/TorrServer` | GitHub API URL. Only used for remote installation. |

## Dependencies

None.

## Tags

The role supports selective execution with Ansible tags. Use the `torrserver` tag to run the full role.

| Tag | Description |
|-----|-------------|
| `torrserver` | Run all role tasks |
| `setup` | Environment preparation (packages, user, directories) |
| `dependencies` | Install system packages only |
| `user` | Create torrserver user, group, and install directory |
| `install` | Binary installation (local or remote) |
| `facts` | Detect installed TorrServer version before upgrade |
| `install_local` | Copy binary from `torrserver_local_file` |
| `install_remote` | Download binary from GitHub releases |
| `config` | All configuration tasks |
| `version` | Resolve version capabilities for CLI and BitTorr settings |
| `daemon_config` | Deploy `torrserver.config` (also runs `version`) |
| `auth` | Deploy `accs.db` authentication file |
| `settings` | Deploy `settings.json` BitTorr settings (also runs `version`) |
| `telegram` | Deploy `tg.cfg` Telegram bot configuration (requires `torrserver_tg_token`) |
| `tg_config` | Alias for `telegram` |
| `debug` | Show rendered configuration (slurp + debug) |
| `bbr` | Enable BBR congestion control |
| `service` | Manage systemd unit |
| `systemd` | Create and enable the systemd service |

Examples:

```bash
# Update authentication only
ansible-playbook playbooks/torrserver.yml --tags auth

# Reinstall binary from GitHub
ansible-playbook playbooks/torrserver.yml --tags install_remote

# Apply daemon CLI config and BitTorr settings
ansible-playbook playbooks/torrserver.yml --tags "daemon_config,settings"
```

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

### Example with Proxy and HTTPS

```yaml
- hosts: all
  roles:
    - role: pavelpikta.torrserver
      vars:
        torrserver_version: latest
        torrserver_port: 8090
        torrserver_ssl_enable: true
        torrserver_force_https: true
        torrserver_proxy_url: "socks5://127.0.0.1:1080"
        torrserver_proxy_mode: tracker
        torrserver_webdav: true
```

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
