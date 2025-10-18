## ERPNext Service

Service contract for ERPNext v15 using the upstream Frappe nginx container. The defaults wire ERPNext to the shared MariaDB and Redis services, configure persistent volumes for sites/logs, and expose runtime-agnostic knobs so the same definition can render for any supported infrastructure backend.

### Runtime Coverage
- Proxmox LXC via the shared render/apply roles
- Docker Compose v2
- Podman Quadlet units (system scope)
- Kubernetes Deployment + Service + PVC + Secret
- Bare-metal systemd service

### Dependencies
- `mariadb` – provides the primary database (`DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`)
- `redis` – supplies cache/queue/socketio endpoints (`REDIS_CACHE`, `REDIS_QUEUE`, `REDIS_SOCKETIO`)

### Exports
```
ERP_SITE_URL={{ erpnext_site_url }}
ERP_SITE_NAME={{ erpnext_site_name }}
```

### Secrets
- `DB_PASSWORD` – MariaDB credential for the ERPNext database user
- `ADMIN_PASSWORD` – password for the ERPNext administrator bootstrap
- `SITE_ADMIN_PASSWORD` – password for the first ERPNext site

All secrets default to environment lookups (`ERP_DB_PASSWORD`, `ERP_ADMIN_PASSWORD`, `ERP_SITE_ADMIN_PASSWORD`) with placeholder fallbacks. Override in inventory or via Ansible Vault before deploying.

### Health Check
`curl -fsS http://127.0.0.1:{{ erpnext_service_port }}/api/method/ping` ensures the ERPNext HTTP endpoint responds. The same command feeds Compose healthchecks, Quadlet probes, Kubernetes readiness/liveness, and the post-deploy validation step.

### Key Overrides
| Variable | Default | Purpose |
| --- | --- | --- |
| `erpnext_service_port` | `8081` | Published HTTP port |
| `erpnext_site_name` | `erp.example.com` | Primary ERPNext site name |
| `erpnext_site_url` | `https://erp.example.com` | Public base URL used for headers |
| `erpnext_db_host` | `mariadb` | Database host rendered into the container |
| `erpnext_db_name` | `erpnext` | MariaDB schema created for ERPNext |
| `erpnext_sites_volume` | `erpnext-sites` | Persistent sites volume |
| `erpnext_logs_volume` | `erpnext-logs` | Persistent logs volume |
| `erpnext_container_ip` | `192.168.100.13` | Proxmox LXC address |
| `erpnext_container_cpu_cores` | `4` | CPU allocation across runtimes |
| `erpnext_container_memory_mb` | `6144` | Memory allocation across runtimes |
| `erpnext_kubernetes_namespace` | `business` | Namespace for K8s artifacts |

Adjust these in your inventory to match your environment. Additional runtime-specific keys can be appended to the defaults if the shared templates consume them (for example adding ingress annotations or extra volumes).

### Usage
```yaml
- hosts: erp_hosts
  roles:
    - role: svc-erpnext
      vars:
        runtime: docker
        erpnext_site_url: https://erp.example.com
        erpnext_site_name: erp.example.com
        erpnext_db_password: "{{ vault_erp_db_password }}"
        erpnext_admin_password: "{{ vault_erp_admin_password }}"
        erpnext_site_admin_password: "{{ vault_erp_site_admin_password }}"
```
