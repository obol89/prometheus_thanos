# Ansible Role: Prometheus + Thanos

An Ansible role for deploying and managing a [Prometheus](https://prometheus.io/) monitoring stack with [Thanos](https://thanos.io/) for long-term storage, high availability, and multi-region querying.

## Architecture

This role deploys the following components based on the `prometheus_role` variable:

**Sidecar nodes** (`prometheus_role: sidecar`):
- Prometheus server
- Thanos Sidecar
- Alertmanager (clustered)

**Store nodes** (`prometheus_role: store`):
- Prometheus server
- Thanos Sidecar
- Thanos Store Gateway
- Thanos Query (with HAProxy frontend)
- Thanos Compact
- Thanos Rule

```
┌─────────────────────┐     ┌─────────────────────┐
│   Sidecar Node(s)   │     │     Store Node       │
│                     │     │                     │
│  Prometheus ←─ Sidecar ──→ Store Gateway        │
│  Alertmanager       │     │  Query ← HAProxy    │
│                     │     │  Compact            │
│                     │     │  Rule               │
└─────────────────────┘     └─────────────────────┘
```

## Requirements

- Ansible 2.9+
- Target OS: Linux (systemd-based)
- Ports listed in [defaults/main.yml](defaults/main.yml) must be available

## Role Variables

### Versions

| Variable | Default | Description |
|----------|---------|-------------|
| `prometheus_version` | `2.53.4` | Prometheus release version |
| `thanos_version` | `0.38.0` | Thanos release version |
| `prometheus_alertmanager_version` | `0.28.1` | Alertmanager release version |

### Key Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `prometheus_tsdb_retention_time` | `15d` | Local Prometheus data retention |
| `prometheus_region` | — | Region label (e.g., `us`, `eu`) |
| `prometheus_role` | — | Node role: `sidecar` or `store` |
| `thanos_query_endpoints` | `[]` | List of Thanos sidecar/store endpoints |
| `thanos_rule_alertmanager` | `[]` | Alertmanager endpoints for Thanos Rule |
| `prometheus_alertmanager_cluster_peers` | `[]` | Alertmanager cluster peer hostnames |

### Secrets (must be overridden)

These variables have no defaults and **must** be set in your inventory, group_vars, or host_vars:

| Variable | Description |
|----------|-------------|
| `netbox_api_token` | API token for NetBox HTTP service discovery |
| `alertmanager_smtp_smarthost` | SMTP relay host and port |
| `alertmanager_smtp_from` | Sender address for alert emails |
| `alertmanager_email_recipient` | Recipient address for alert emails |
| `alertmanager_webhook_url` | Webhook URL for alert notifications (e.g., iLert, PagerDuty) |
| `snmp_auth_profile` | SNMP v3 authentication profile name |

### Network Ports

| Variable | Default | Description |
|----------|---------|-------------|
| `prometheus_http_port` | `9090` | Prometheus HTTP |
| `thanos_query_http_port` | `10902` | Thanos Query HTTP |
| `thanos_query_grpc_port` | `10901` | Thanos Query gRPC |
| `thanos_sidecar_http_port` | `10904` | Thanos Sidecar HTTP |
| `thanos_sidecar_grpc_port` | `10903` | Thanos Sidecar gRPC |
| `thanos_store_http_port` | `10906` | Thanos Store HTTP |
| `thanos_store_grpc_port` | `10905` | Thanos Store gRPC |
| `thanos_compact_http_port` | `10907` | Thanos Compact HTTP |
| `thanos_rule_grpc_port` | `10908` | Thanos Rule gRPC |
| `thanos_rule_http_port` | `10909` | Thanos Rule HTTP |

See [defaults/main.yml](defaults/main.yml) for the full list of variables.

## Tags

| Tag | Description |
|-----|-------------|
| `prometheus_thanos` | Full deployment of all components |
| `prometheus_reload` | Reload Prometheus, Alertmanager, and Thanos Rule configs only |

## Example Playbook

```yaml
- hosts: prometheus_servers
  roles:
    - role: prometheus_thanos
      vars:
        prometheus_region: us
        prometheus_role: sidecar
        netbox_api_token: "{{ vault_netbox_api_token }}"
        alertmanager_smtp_smarthost: "smtp.example.com:25"
        alertmanager_smtp_from: "alertmanager@example.com"
        alertmanager_email_recipient: "alerts@example.com"
        alertmanager_webhook_url: "{{ vault_alertmanager_webhook_url }}"
        snmp_auth_profile: "my_snmp_v3"
```

## File Structure

```
prometheus_thanos/
├── defaults/main.yml              # Default variables
├── tasks/
│   ├── main.yml                   # Task orchestration
│   ├── common.yml                 # Binary downloads and user setup
│   ├── prometheus.yml             # Prometheus deployment
│   ├── prometheus_reload.yml      # Config reload handler
│   ├── prometheus_alertmanager.yml # Alertmanager deployment
│   ├── thanos_sidecar.yml         # Thanos Sidecar
│   ├── thanos_store.yml           # Thanos Store Gateway
│   ├── thanos_query.yml           # Thanos Query
│   ├── thanos_compact.yml         # Thanos Compact
│   ├── thanos_rule.yml            # Thanos Rule
│   └── haproxy.yml                # HAProxy for Thanos Query
├── templates/                     # Jinja2 config templates
└── files/                         # Static config files and alerting rules
```

## License

MIT
