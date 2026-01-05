# Quay Registry Ansible Deployment

Automated deployment and testing of Quay container registry using Ansible and Podman.

## Playbooks

### quay-setup.yaml
Deploys an all-in-one Quay registry with:
- Auto-generated secrets
- Pre-seeded admin account
- Self-signed TLS certificates
- Proper storage permissions
- No anonymous access
- No self-signup

**Usage:**
```bash
# Default password
ansible-playbook quay-setup.yaml

# Custom password
ansible-playbook quay-setup.yaml -e quay_admin_password='YourPassword'

# Custom server
ansible-playbook quay-setup.yaml -e quay_server_hostname='10.0.0.100'

# Clean reinstall
ansible-playbook quay-setup.yaml -e purge_data=true
```

### quay-test.yaml
End-to-end testing of Quay registry:
- Login/authentication
- Push images
- Pull images
- Multiple tags
- Running containers
- Repository accessibility

**Usage:**
```bash
# Default server (192.168.0.53)
ansible-playbook quay-test.yaml -e quay_admin_password='YourPassword'

# Custom server
ansible-playbook quay-test.yaml \
  -e quay_server_hostname='10.0.0.100' \
  -e quay_admin_password='YourPassword'
```

## Requirements

- RHEL/Fedora/CentOS
- Ansible 2.9+
- Podman
- Python 3

## Quick Start

1. Deploy Quay:
```bash
   ansible-playbook quay-setup.yaml -e quay_admin_password='MySecurePassword'
```

2. Test deployment:
```bash
   ansible-playbook quay-test.yaml -e quay_admin_password='MySecurePassword'
```

3. Access Quay:
   - URL: https://192.168.0.53/
   - Username: admin
   - Password: (whatever you set)

## Features

- ✅ Fully automated deployment
- ✅ Auto-generated secrets
- ✅ Self-signed TLS certificates
- ✅ Pre-configured admin user
- ✅ Comprehensive end-to-end testing
- ✅ No manual configuration required
- ✅ Idempotent playbooks

## Troubleshooting

**View logs:**
```bash
podman logs --tail 200 quay
```

**Restart Quay:**
```bash
podman restart quay
```

**Stop all services:**
```bash
podman pod stop quay-pod
```

**Start all services:**
```bash
podman pod start quay-pod
```

## Architecture

- **Quay**: Container registry (port 443)
- **PostgreSQL 15**: Database backend
- **Redis 7**: Caching and session storage
- **Podman Pod**: All services in a single pod

## License

MIT
