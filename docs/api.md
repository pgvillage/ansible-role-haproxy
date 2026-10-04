# API documentation

This document lists every variable this role uses. All variables and their defaults are in
[`defaults/main.yml`](../defaults/main.yml).

## Overview

| Variable | Default | Description |
|---|---|---|
| [`haproxy_mode`](#haproxy_mode) | `tcp` | Proxy mode for the `defaults` section |
| [`haproxy_stats_port`](#haproxy_stats_port) | `8404` | Port for the statistics page |
| [`haproxy_package_state`](#haproxy_package_state) | `present` | State of the HAProxy packages |
| [`haproxy_local_package_names`](#haproxy_local_package_names) | `[]` | Local package files to copy and install |
| [`haproxy_package_names`](#haproxy_package_names) | `["haproxy"]` | Packages to install from repositories |
| [`haproxy_maxconn`](#haproxy_maxconn) | `1000` | Maximum concurrent connections and session rate |
| [`haproxy_defaults_timeouts`](#haproxy_defaults_timeouts) | see below | Timeouts for the `defaults` section |
| [`haproxy_socket`](#haproxy_socket) | `/var/lib/haproxy/stats` | Path to the admin stats socket |
| [`haproxy_user`](#haproxy_user--haproxy_group) | `haproxy` | OS user HAProxy runs as |
| [`haproxy_group`](#haproxy_user--haproxy_group) | `haproxy` | OS group HAProxy runs as |
| [`haproxy_frontends`](#haproxy_frontends) | `[]` | List of frontends |
| [`haproxy_backends`](#haproxy_backends) | `[]` | List of backends |
| [`haproxy_global_vars`](#haproxy_global_vars) | `[]` | Extra lines for the `global` section |

## Installation

### `haproxy_package_state`

State of the HAProxy packages. Accepts any value that `ansible.builtin.package` supports
(`present`, `latest`, `absent`).

```yaml
haproxy_package_state: present
```

### `haproxy_package_names`

List of packages to install from the configured package repositories.

```yaml
haproxy_package_names:
  - haproxy
```

### `haproxy_local_package_names`

List of local package files to copy to `/tmp/` on the target host and install from there.
Ansible looks the files up in the role's `files` directory or on the Ansible search path.

```yaml
haproxy_local_package_names:
  - haproxy-2.8.3-1.el9.x86_64.rpm
```

Default: `[]`

## Global settings

### `haproxy_maxconn`

Maximum number of concurrent connections (`maxconn`) for the `global` and `defaults` sections.
The role also uses this value as the maximum session rate (`maxsessrate`) in the `global` section.

Default: `1000`

### `haproxy_socket`

Path to the HAProxy admin stats socket (`stats socket <path> level admin`).

Default: `/var/lib/haproxy/stats`

### `haproxy_user` / `haproxy_group`

OS user and group HAProxy runs as.

Default: `haproxy` / `haproxy`

### `haproxy_global_vars`

List of extra raw configuration lines added to the `global` section, one per line.

```yaml
haproxy_global_vars:
  - "nbthread 4"
  - "tune.ssl.default-dh-param 2048"
```

Default: `[]`

## Defaults section

### `haproxy_mode`

Proxy mode used in the `defaults` section: `tcp` or `http`. Use `tcp` for PostgreSQL traffic.

Default: `tcp`

### `haproxy_defaults_timeouts`

List of timeouts for the `defaults` section. Each entry is rendered as `timeout <entry>`, so it
needs to contain both the timeout name and its value.

```yaml
haproxy_defaults_timeouts:
  - "client     31m"
  - "connect    4s"
  - "check      5s"
```

## Statistics

### `haproxy_stats_port`

Port for the HAProxy statistics frontend. The statistics page is served at
`http://<host>:<port>/stats` and refreshes every 15 seconds.

Default: `8404`

## Frontends and backends

### `haproxy_frontends`

List of frontends. Every item renders a `frontend` section in `haproxy.cfg`.

| Key | Required | Default | Description |
|---|---|---|---|
| `name` | yes | | Name of the frontend |
| `address` | yes | | Address to bind to, e.g. `"*"` |
| `port` | yes | | Port to bind to |
| `bind_params` | no | `''` | Extra parameters appended to the `bind` line |
| `mode` | no | `http` | `tcp` or `http` |
| `backend` | no | | Name of the default backend (`default_backend`) |
| `options` | no | `[]` | List of `option` lines |
| `params` | no | `[]` | List of raw configuration lines |
| `timeout_client` | no | | Client timeout, e.g. `"10800s"` |

Example:

```yaml
haproxy_frontends:
  - name: "PostgresReadWrite-frontend"
    address: "*"
    port: 5432
    mode: tcp
    backend: "PostgresReadWrite-backend"
    timeout_client: "10800s"
```

Default: `[]`

### `haproxy_backends`

List of backends. Every item renders a `backend` section in `haproxy.cfg`.

| Key | Required | Default | Description |
|---|---|---|---|
| `name` | yes | | Name of the backend |
| `servers` | yes | | List of servers (see below) |
| `mode` | no | `http` | `tcp` or `http` |
| `balance_method` | no | `leastconn` | Load balancing algorithm |
| `options` | no | `[]` | List of `option` lines |
| `params` | no | `[]` | List of raw configuration lines |

Every item in `servers` supports these keys:

| Key | Required | Description |
|---|---|---|
| `name` | yes | Server name |
| `address` | yes | Server address |
| `port` | yes | Server port |
| `checkport` | no | Port used for health checks. If you omit it, HAProxy checks `port` |

Every server gets these fixed check settings:
`inter 2s downinter 5s rise 3 fall 2 slowstart 60s maxconn 1000 maxqueue 128 weight 100`.
Every backend gets `timeout server 10800s`.

Example:

```yaml
haproxy_backends:
  - name: "PostgresReadWrite-backend"
    mode: tcp
    balance_method: "leastconn"
    options:
      - "external-check"
    params:
      - "external-check command /opt/pgroute66/checkpgprimary.sh"
    servers:
      - name: "dbserver1.example.org"
        address: "192.168.17.18"
        port: 5432
        checkport: "9201"
      - name: "dbserver2.example.org"
        address: "192.168.17.19"
        port: 5432
        checkport: "9201"
```

Default: `[]`
