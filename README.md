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
ansible-playbook quay-setup.yaml -K

# Custom password
ansible-playbook quay-setup.yaml -e quay_admin_password='YourPassword' -K

# Custom server
ansible-playbook quay-setup.yaml -e quay_server_hostname='10.0.0.100' -K

# Clean reinstall
ansible-playbook quay-setup.yaml -e purge_data=true -K
```

**Note:** The `-K` flag prompts for your sudo password (required for system operations).

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

## Installation

Install required Ansible collections:
```bash
ansible-galaxy collection install ansible.posix
```

## Quick Start

1. Deploy Quay:
```bash
   ansible-playbook quay-setup.yaml -e quay_admin_password='MySecurePassword' -K
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
- ✅ OCI artifact support (tarballs, etc.)

## Working with OCI Artifacts (Tarballs)

Quay 3.8.1+ supports storing arbitrary files as OCI artifacts using oras CLI.

**Prerequisites:**
- Install [oras CLI](https://oras.land/docs/installation)
- Login to registry: `oras login <registry> --insecure` (or `podman login` for shared credentials)

**Push a tarball:**
```bash
# Create empty config file (required)
echo '{}' > config.json

# Push with relative path (important!)
oras push <registry>/org/repo:tag \
  --config config.json:application/vnd.unknown.config.v1+json \
  myfile.tar.gz:application/tar+gzip \
  --insecure
```

**Pull a tarball:**
```bash
# Pull (extracts to current directory)
oras pull <registry>/org/repo:tag --insecure

# Verify contents
tar tvzf myfile.tar.gz
```

**Example:**
```bash
# Create tarball
tar czf test.tar.gz files/

# Create config
echo '{}' > config.json

# Push
oras push 192.168.0.53/myorg/artifacts:v1 \
  --config config.json:application/vnd.unknown.config.v1+json \
  test.tar.gz:application/tar+gzip \
  --insecure

# Pull
oras pull 192.168.0.53/myorg/artifacts:v1 --insecure
```

**Important Notes:**
- Use **relative paths** when pushing (not absolute paths like `~/file.tar.gz`)
- Media type must be `application/tar+gzip` for `.tar.gz` files
- Config media type must be `application/vnd.unknown.config.v1+json`
- The `--insecure` flag is needed for self-signed certificates

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
