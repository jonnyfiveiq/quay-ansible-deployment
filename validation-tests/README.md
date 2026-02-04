# Quay Registry Validation Test Suite

Ansible playbook suite to execute validation tests against a Quay registry as defined in the AAP/Quay Integration Proposal Section 5.

## Prerequisites

- Ansible 2.14+ with `containers.podman` collection installed
- Podman installed on the test runner
- Network access to the Quay registry
- Quay API credentials with repository create/delete permissions
- (Optional) SSH access to Quay container host for failure/recovery tests

## Installation

```bash
# Install required Ansible collection
ansible-galaxy collection install containers.podman

# Clone or copy this directory to your test runner
```

## Configuration

1. Copy and edit the variables file:

```bash
cp vars/test_config.yml vars/my_config.yml
# Edit vars/my_config.yml with your Quay server details
```

2. Set the Quay password as an environment variable:

```bash
export QUAY_PASSWORD='your-password-or-token'
```

## Usage

### Run All Tests

```bash
ansible-playbook quay_validation_tests.yml \
  -e "quay_host=quay.example.com" \
  -e "quay_username=testuser" \
  -e "test_namespace=aap-validation"
```

### Run Specific Test Sections

```bash
# Concurrent operation tests only (CO-001 to CO-004)
ansible-playbook quay_validation_tests.yml --tags concurrent

# Scale and capacity tests only (SC-001 to SC-004)
ansible-playbook quay_validation_tests.yml --tags scale

# Operational lifecycle tests only (OL-001 to OL-003)
ansible-playbook quay_validation_tests.yml --tags lifecycle

# Failure and recovery tests only (FR-001 to FR-002)
ansible-playbook quay_validation_tests.yml --tags failure \
  -e "run_failure_tests=true"
```

### Run Individual Tests

```bash
# Run only CO-001
ansible-playbook quay_validation_tests.yml --tags co001

# Run only SC-004
ansible-playbook quay_validation_tests.yml --tags sc004
```

### Run Extended Stability Test (48 hours)

```bash
ansible-playbook quay_validation_tests.yml --tags ol003 \
  -e "run_extended_tests=true"
```

## Test Sections

### 5.3.1 Concurrent Operation Limits

| Test ID | Description |
|---------|-------------|
| CO-001 | Concurrent image pulls (identical image) - 5/10/20 simultaneous pulls |
| CO-002 | Concurrent image pulls (distinct images) - 10 different images |
| CO-003 | Concurrent image pushes - 5/10 simultaneous pushes |
| CO-004 | Mixed concurrent operations - pulls + pushes + tag operations |

### 5.3.2 Scale and Capacity Limits

| Test ID | Description |
|---------|-------------|
| SC-001 | Maximum repository count - 10/25/50/100 repositories |
| SC-002 | Maximum images per repository - 50/100/200/500 tagged images |
| SC-003 | Maximum total image count - 50/100/500/1000 total images |
| SC-004 | Large image handling - 1GB/2GB/3GB image push/pull throughput |

### 5.3.3 Operational Lifecycle

| Test ID | Description |
|---------|-------------|
| OL-001 | Garbage collection under load |
| OL-002 | Garbage collection at scale |
| OL-003 | Extended operation stability (48-hour test) |

### 5.3.4 Failure and Recovery

| Test ID | Description |
|---------|-------------|
| FR-001 | Interrupted push recovery |
| FR-002 | Container restart under load |

## Output

Test results are written to the `reports/` directory:

- `quay_validation_report_YYYY-MM-DD.md` - Human-readable Markdown report
- `quay_validation_results_YYYY-MM-DD.json` - Machine-readable JSON results

## Test Environment Specification

As per the proposal, tests should be run against:

| Component | Specification |
|-----------|---------------|
| Quay Deployment | Single-node containerized (Podman on RHEL 9) |
| Storage Backend | Local filesystem (no object storage) |
| Database | PostgreSQL 15 (shared instance) |
| Cache | Single Redis container |
| Compute Resources | 2 vCPU, 16GB RAM |
| Storage | 500GB local SSD |

## Customising Test Parameters

Edit `vars/test_config.yml` to adjust:

- Concurrent operation counts
- Repository/image scale limits
- Image sizes
- Success criteria thresholds
- Extended test duration

## Notes

- GC tests rely on Quay's automatic garbage collection; there is no manual trigger
- Failure/recovery tests require SSH access to the Quay host
- Extended stability test (OL-003) runs for 48 hours by default
- Tests create artifacts in the test namespace which are cleaned up afterwards

## Troubleshooting

**TLS certificate errors:**
```bash
-e "validate_certs=false"
```

**API authentication failures:**
Ensure your user has admin permissions on the test namespace.

**Timeouts on large image tests:**
Increase async timeouts in the playbook if your network is slow.
