# Gemini AI Assistant Configuration

**Version**: 1.0.0  
**Project**: Ansible Handshake DNS Infrastructure  
**Last Updated**: November 22, 2025

---

## Project Overview

This repository contains an Ansible playbook for deploying a decentralized DNS infrastructure combining Handshake (HNS) blockchain, Unbound DNS server, Caddy reverse proxy, and Quad9 secure DNS.

---

## Primary Reference

**All AI agents should reference**: `@AGENTS.md` as the universal source of truth for project conventions, workflows, and standards.

This file contains Gemini-specific configurations and optimizations that complement the universal agent guidelines.

---

## Gemini-Specific Optimizations

### Model Capabilities

Gemini excels at:
- **Multi-modal understanding**: Can process diagrams, architecture drawings, and documentation together
- **Code generation**: Strong Ansible YAML and Jinja2 template generation
- **Pattern recognition**: Identifying anti-patterns and suggesting improvements
- **Contextual reasoning**: Understanding infrastructure interdependencies

### Recommended Context Loading

```
@AGENTS.md
@DEVELOPMENT_PLAN.md
@README.md
@group_vars/all.yml
@roles/[component]/tasks/main.yml
@roles/[component]/templates/config.j2
```

---

## Ansible Development with Gemini

### 1. Playbook Generation

When asked to create Ansible playbooks, follow this structure:

```yaml
---
# Playbook: deploy.yml
# Purpose: Deploy decentralized DNS infrastructure
# Target: Ubuntu 20.04+ servers with Docker installed

- name: Deploy DNS Infrastructure
  hosts: dns_servers
  become: yes
  gather_facts: yes
  
  vars_files:
    - group_vars/all.yml
  
  pre_tasks:
    - name: Display deployment information
      ansible.builtin.debug:
        msg: "Deploying to {{ inventory_hostname }}"
    
    - name: Verify Docker is installed
      ansible.builtin.command: docker --version
      register: docker_check
      changed_when: false
      failed_when: docker_check.rc != 0
  
  roles:
    - role: common
      tags: [common, base]
    
    - role: handshake
      tags: [hsd, blockchain]
    
    - role: unbound
      tags: [unbound, dns]
    
    - role: caddy
      tags: [caddy, proxy]
  
  post_tasks:
    - name: Verify all services are running
      ansible.builtin.docker_container_info:
        name: "{{ item }}"
      loop:
        - hsd
        - unbound
        - caddy
      register: container_status
      failed_when: container_status.container.State.Status != 'running'
      tags: [verify]
    
    - name: Display deployment summary
      ansible.builtin.debug:
        msg: "Deployment complete. Services: {{ container_status.results | map(attribute='container.Name') | list }}"
```

**Key Gemini strengths to leverage**:
- Generate comprehensive task descriptions
- Create robust error handling
- Include verification steps
- Suggest meaningful tags

### 2. Role Development

When creating Ansible roles, generate all required files:

**tasks/main.yml**:
```yaml
---
# Role: handshake
# Deploys Handshake (HNS) full node for decentralized DNS

- name: Include installation tasks
  ansible.builtin.include_tasks: install.yml
  tags: [install]

- name: Include configuration tasks
  ansible.builtin.include_tasks: configure.yml
  tags: [config]

- name: Include verification tasks
  ansible.builtin.include_tasks: verify.yml
  tags: [verify]
```

**tasks/install.yml**:
```yaml
---
- name: Create Handshake data directory
  ansible.builtin.file:
    path: "{{ hsd_data_dir }}"
    state: directory
    owner: "{{ hsd_user }}"
    group: "{{ hsd_group }}"
    mode: '0750'
  tags: [directories]

- name: Pull Handshake Docker image
  community.docker.docker_image:
    name: "{{ hsd_image }}"
    tag: "{{ hsd_version }}"
    source: pull
  tags: [images]

- name: Deploy Handshake container
  community.docker.docker_container:
    name: hsd
    image: "{{ hsd_image }}:{{ hsd_version }}"
    state: started
    restart_policy: unless-stopped
    networks:
      - name: "{{ docker_network }}"
    ports:
      - "{{ hsd_rpc_port }}:12037"
      - "{{ hsd_ns_port }}:5349"
    volumes:
      - "{{ hsd_data_dir }}:/root/.hsd"
    env:
      HSD_NETWORK: "{{ hsd_network }}"
      HSD_LOG_LEVEL: "{{ hsd_log_level }}"
    healthcheck:
      test: ["CMD", "hsd-cli", "info"]
      interval: 30s
      timeout: 10s
      retries: 3
  tags: [deploy]
```

**handlers/main.yml**:
```yaml
---
- name: restart hsd
  community.docker.docker_container:
    name: hsd
    state: restarted

- name: reload hsd config
  community.docker.docker_container:
    name: hsd
    comparisons:
      env: strict
    state: started
    force_recreate: yes
```

**defaults/main.yml** (with comprehensive documentation):
```yaml
---
# Handshake Role Default Variables

# ============================================
# DOCKER CONFIGURATION
# ============================================

# Docker image for Handshake node
# Official image: https://hub.docker.com/r/handshake/hsd
hsd_image: "handshake/hsd"

# Image version/tag
# Use specific versions for production, 'latest' for development
# Check releases: https://github.com/handshake-org/hsd/releases
hsd_version: "latest"

# ============================================
# NETWORK CONFIGURATION
# ============================================

# Handshake network type
# Options: main, testnet, regtest, simnet
# Production should always use 'main'
hsd_network: "main"

# RPC port for API access
# Used by Unbound to query HNS blockchain
# Default: 12037
# Valid range: 1024-65535
hsd_rpc_port: 12037

# Authoritative nameserver port
# Port where HSD serves DNS responses for .hns TLDs
# Default: 5349 (non-standard to avoid conflicts)
hsd_ns_port: 5349

# ============================================
# STORAGE CONFIGURATION
# ============================================

# Data directory for blockchain storage
# Should have sufficient space (50GB+ recommended)
# This directory will be mounted into the container
hsd_data_dir: "/opt/hsd/data"

# User and group for file ownership
hsd_user: "hsd"
hsd_group: "hsd"

# ============================================
# LOGGING
# ============================================

# Log level for HSD
# Options: none, error, warning, info, debug, spam
# Production: info or warning
# Development: debug
hsd_log_level: "info"

# ============================================
# PERFORMANCE TUNING
# ============================================

# Maximum memory for HSD process (in MB)
# Adjust based on available system resources
hsd_max_memory: 4096

# Number of workers for parallel processing
# Recommended: number of CPU cores
hsd_workers: "{{ ansible_processor_vcpus }}"
```

### 3. Template Generation (Jinja2)

Gemini is excellent at generating complex Jinja2 templates. Example:

**templates/unbound.conf.j2**:
```jinja2
# Unbound DNS Server Configuration
# Generated by Ansible - DO NOT EDIT MANUALLY
# Role: unbound
# Date: {{ ansible_date_time.iso8601 }}

server:
  # Basic Configuration
  # ============================================
  
  # Server will run as this user
  username: "unbound"
  
  # Working directory
  directory: "/etc/unbound"
  
  # Logging
  verbosity: {{ unbound_verbosity | default(1) }}
  {% if unbound_log_queries %}
  log-queries: yes
  {% endif %}
  
  # Network Configuration
  # ============================================
  
  # Listen on these interfaces
  {% for interface in unbound_interfaces %}
  interface: {{ interface }}
  {% endfor %}
  
  # Listen on standard DNS port
  port: 53
  
  # Enable TCP for large responses
  do-tcp: yes
  
  # Enable UDP (standard DNS)
  do-udp: yes
  
  # DNS-over-TLS Configuration
  {% if unbound_enable_dot %}
  # ============================================
  tls-service-key: "{{ unbound_tls_key }}"
  tls-service-pem: "{{ unbound_tls_cert }}"
  tls-port: 853
  {% endif %}
  
  # Access Control
  # ============================================
  
  # Deny everything by default
  access-control: 0.0.0.0/0 refuse
  access-control: ::0/0 refuse
  
  # Allow specific networks
  {% for network in unbound_allowed_networks %}
  access-control: {{ network }} allow
  {% endfor %}
  
  # Cache Configuration
  # ============================================
  
  # Message cache size
  msg-cache-size: {{ unbound_msg_cache_size | default('50m') }}
  
  # RRset cache size (should be 2x message cache)
  rrset-cache-size: {{ unbound_rrset_cache_size | default('100m') }}
  
  # Cache TTL settings
  cache-min-ttl: {{ unbound_cache_min_ttl | default(300) }}
  cache-max-ttl: {{ unbound_cache_max_ttl | default(86400) }}
  
  # Prefetch frequently accessed records before expiry
  prefetch: {{ 'yes' if unbound_prefetch else 'no' }}
  
  # DNSSEC Configuration
  # ============================================
  
  {% if unbound_enable_dnssec %}
  # Enable DNSSEC validation
  auto-trust-anchor-file: "{{ unbound_trust_anchor }}"
  val-permissive-mode: no
  
  # Serve expired cache while updating
  serve-expired: yes
  serve-expired-ttl: 86400
  {% else %}
  # DNSSEC validation disabled
  val-permissive-mode: yes
  {% endif %}
  
  # Privacy Configuration
  # ============================================
  
  # Hide server version
  hide-identity: yes
  hide-version: yes
  
  # Minimize query information sent to upstream
  qname-minimisation: yes
  
  # Performance Tuning
  # ============================================
  
  # Number of threads (match CPU cores)
  num-threads: {{ ansible_processor_vcpus }}
  
  # Number of queries per thread
  num-queries-per-thread: 4096
  
  # Socket buffers for high query rate
  so-rcvbuf: 4m
  so-sndbuf: 4m
  
  # HIP-5 Protocol Support
  # ============================================
  
  {% if unbound_enable_hip5 %}
  # Enable plugins for cross-protocol resolution
  module-config: "validator iterator"
  
  # Custom plugin configuration
  {% for protocol in hip5_protocols %}
  local-zone: "{{ protocol }}." redirect
  local-data: "{{ protocol }}. IN A {{ hip5_resolvers[protocol] }}"
  {% endfor %}
  {% endif %}

# Stub Zone Configuration for Handshake
# ============================================

stub-zone:
  # Root zone for HNS resolution
  name: "."
  
  # Forward to Handshake authoritative nameserver
  stub-addr: 127.0.0.1@{{ hsd_ns_port }}
  
  # Don't validate DNSSEC for HNS (uses blockchain verification)
  stub-no-cache: no
  stub-prime: no

# Forward Zone Configuration for Traditional DNS
# ============================================

forward-zone:
  # Forward everything else to Quad9
  name: "."
  
  # Primary Quad9 servers with DNS-over-TLS
  {% for server in quad9_servers %}
  forward-addr: {{ server }}
  {% endfor %}
  
  # Use TLS for upstream queries
  forward-tls-upstream: yes
  
  # Don't cache negative responses as long
  forward-no-cache: no

{% if unbound_custom_zones | default([]) | length > 0 %}
# Custom Zones
# ============================================

{% for zone in unbound_custom_zones %}
local-zone: "{{ zone.name }}" {{ zone.type }}
{% for record in zone.records %}
local-data: "{{ record }}"
{% endfor %}

{% endfor %}
{% endif %}
```

**Gemini's template generation strengths**:
- Comprehensive conditional logic
- Clear section organization
- Inline documentation
- Variable validation

### 4. Variable Validation

Gemini can generate variable validation tasks:

```yaml
---
# tasks/validate.yml
# Validate role variables before execution

- name: Validate required variables are defined
  ansible.builtin.assert:
    that:
      - hsd_image is defined
      - hsd_version is defined
      - hsd_data_dir is defined
      - docker_network is defined
    fail_msg: "Required variables are not defined. Check role defaults and group_vars."
    success_msg: "All required variables are defined."

- name: Validate port numbers are in valid range
  ansible.builtin.assert:
    that:
      - hsd_rpc_port | int >= 1024
      - hsd_rpc_port | int <= 65535
      - hsd_ns_port | int >= 1024
      - hsd_ns_port | int <= 65535
    fail_msg: "Port numbers must be between 1024 and 65535."
    success_msg: "Port numbers are valid."

- name: Validate network type
  ansible.builtin.assert:
    that:
      - hsd_network in ['main', 'testnet', 'regtest', 'simnet']
    fail_msg: "Invalid network type. Must be main, testnet, regtest, or simnet."
    success_msg: "Network type is valid."

- name: Validate data directory is absolute path
  ansible.builtin.assert:
    that:
      - hsd_data_dir is match('^/')
    fail_msg: "Data directory must be an absolute path."
    success_msg: "Data directory path is valid."

- name: Check data directory has sufficient space
  ansible.builtin.shell: |
    df -BG {{ hsd_data_dir | dirname }} | awk 'NR==2 {print $4}' | sed 's/G//'
  register: available_space
  changed_when: false
  failed_when: available_space.stdout | int < 50
```

---

## Code Quality and Best Practices

### Lint Integration

Gemini should suggest running these checks:

```bash
# Ansible syntax check
ansible-playbook --syntax-check playbooks/deploy.yml

# Ansible lint with custom rules
ansible-lint -c .ansible-lint playbooks/ roles/

# YAML lint
yamllint -c .yamllint .

# Jinja2 template validation
ansible-playbook --syntax-check playbooks/deploy.yml
```

### Security Scanning

Suggest security improvements:

```yaml
# Example security enhancements Gemini might recommend

- name: Ensure containers run as non-root
  community.docker.docker_container:
    name: "{{ container_name }}"
    user: "1000:1000"  # Non-root UID:GID
    # ... other config ...

- name: Drop unnecessary Linux capabilities
  community.docker.docker_container:
    name: "{{ container_name }}"
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Only if needed for ports < 1024
    # ... other config ...

- name: Enable read-only root filesystem
  community.docker.docker_container:
    name: "{{ container_name }}"
    read_only: yes
    tmpfs:
      /tmp: "rw,noexec,nosuid,size=100m"  # Writable tmp
    # ... other config ...

- name: Set resource limits
  community.docker.docker_container:
    name: "{{ container_name }}"
    memory: "2g"
    memory_reservation: "1g"
    cpus: "2.0"
    # ... other config ...
```

---

## Testing and Verification

### Molecule Test Scenarios

Gemini can generate Molecule configurations:

**molecule/default/molecule.yml**:
```yaml
---
dependency:
  name: galaxy
  options:
    requirements-file: requirements.yml

driver:
  name: docker

platforms:
  - name: ubuntu-20-04
    image: ubuntu:20.04
    pre_build_image: true
    privileged: true
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    command: /sbin/init

provisioner:
  name: ansible
  config_options:
    defaults:
      callbacks_enabled: profile_tasks
  inventory:
    host_vars:
      ubuntu-20-04:
        docker_network: test_network
        hsd_data_dir: /tmp/hsd
        hsd_network: regtest

verifier:
  name: ansible

scenario:
  name: default
  test_sequence:
    - dependency
    - syntax
    - create
    - prepare
    - converge
    - idempotence
    - verify
    - cleanup
    - destroy
```

**molecule/default/verify.yml**:
```yaml
---
- name: Verify
  hosts: all
  gather_facts: false
  tasks:
    - name: Check HSD container is running
      community.docker.docker_container_info:
        name: hsd
      register: hsd_container
      failed_when: hsd_container.container.State.Status != 'running'

    - name: Verify HSD RPC is responding
      ansible.builtin.uri:
        url: "http://localhost:12037"
        method: POST
        body_format: json
        body:
          method: "getinfo"
          params: []
          id: 1
        status_code: 200
      register: rpc_response

    - name: Check blockchain sync status
      ansible.builtin.assert:
        that:
          - rpc_response.json.result is defined
          - rpc_response.json.result.chain is defined
        fail_msg: "HSD RPC returned unexpected response"

    - name: Verify data directory exists and has correct permissions
      ansible.builtin.stat:
        path: /tmp/hsd
      register: data_dir
      failed_when: not data_dir.stat.exists or data_dir.stat.mode != '0750'
```

---

## Documentation Generation

### Role README Template

Gemini can generate comprehensive role documentation:

```markdown
# Handshake Role

Ansible role for deploying and managing Handshake (HNS) blockchain full nodes.

## Requirements

- Docker 24.0+
- Python docker library: `pip install docker`
- Ansible collections:
  - `community.docker`
  - `community.general`

## Role Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `docker_network` | Docker network name | `dns_net` |
| `hsd_data_dir` | Blockchain data storage path | `/opt/hsd/data` |

### Optional Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `hsd_image` | `handshake/hsd` | Docker image |
| `hsd_version` | `latest` | Image tag |
| `hsd_network` | `main` | Network type (main/testnet) |
| `hsd_rpc_port` | `12037` | RPC API port |
| `hsd_ns_port` | `5349` | Authoritative NS port |
| `hsd_log_level` | `info` | Logging verbosity |

See [defaults/main.yml](defaults/main.yml) for complete list.

## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: dns_servers
  become: yes
  roles:
    - role: handshake
      hsd_network: main
      hsd_log_level: info
      hsd_data_dir: /opt/hsd/data
```

## Testing

```bash
# Syntax check
ansible-playbook --syntax-check tests/test.yml

# Molecule test
molecule test
```

## License

MIT

## Author

DevOps Team
```

---

## Multi-Modal Capabilities

### Architecture Diagram Understanding

Gemini can understand and reference architecture diagrams. Example prompt:

```
Given this infrastructure diagram [attach image], generate:
1. Ansible inventory file with correct groupings
2. Playbook that respects the deployment order shown
3. Group variables for each tier
```

### Log Analysis

Gemini excels at analyzing logs and suggesting fixes:

```
Analyze these Unbound logs and suggest Ansible tasks to fix:

[1669234567] unbound[1234:0] error: could not open autotrust file for writing, /var/lib/unbound/root.key: Permission denied
[1669234568] unbound[1234:0] error: unable to read /etc/unbound/unbound.conf: No such file or directory
```

**Gemini's likely response**:
```yaml
# Fix 1: Ensure trust anchor directory has correct permissions
- name: Create Unbound directory structure
  ansible.builtin.file:
    path: /var/lib/unbound
    state: directory
    owner: unbound
    group: unbound
    mode: '0755'

# Fix 2: Ensure config file exists before service starts
- name: Deploy Unbound configuration
  ansible.builtin.template:
    src: unbound.conf.j2
    dest: /etc/unbound/unbound.conf
    owner: root
    group: unbound
    mode: '0644'
    validate: 'unbound-checkconf %s'
  notify: restart unbound
```

---

## Performance Optimization

### Ansible Acceleration

Gemini can suggest performance improvements:

```yaml
# ansible.cfg optimizations
[defaults]
forks = 20                    # Increase parallelism
gathering = smart             # Smart fact caching
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_fact_cache
fact_caching_timeout = 3600

[ssh_connection]
pipelining = True             # Reduce SSH overhead
ssh_args = -o ControlMaster=auto -o ControlPersist=60s

# Strategy plugins for faster execution
- hosts: all
  strategy: free              # Tasks run as soon as possible
  # or
  strategy: mitogen_linear    # If using mitogen plugin
```

---

## Integration with Other Agents

### Handoff Patterns

**To @architect**:
```
@architect
I've implemented the Handshake role but need architectural review for:
- Multi-datacenter HNS node deployment
- Load balancing strategy for DNS queries
- Data replication approach for blockchain sync

Current implementation: @roles/handshake/
```

**To @devops**:
```
@devops
Handshake role is complete. Please create:
- GitHub Actions workflow for role testing
- Molecule scenarios for different OS versions
- Prometheus monitoring for HNS metrics

Reference: @roles/handshake/tasks/main.yml
```

**To @validator**:
```
@validator
Please create test suite for:
- HNS TLD resolution (.hns domains)
- RPC API functionality
- Blockchain synchronization
- Container restart resilience

Test data: @tests/fixtures/
```

---

## Common Gemini Prompts

### Generate Complete Role
```
Create a complete Ansible role for deploying Unbound DNS server with:
- Docker container deployment
- DNS-over-TLS configuration
- Integration with Handshake blockchain for stub zones
- Quad9 forwarding for traditional DNS
- DNSSEC validation
- Cache optimization
- Molecule tests
```

### Refactor Existing Code
```
Refactor this Ansible playbook to:
- Use proper role structure
- Implement handlers for service restarts
- Add variable validation
- Include idempotency checks
- Add molecule tests

[paste existing code]
```

### Generate Documentation
```
Generate comprehensive README.md for this role:
- Overview
- Requirements
- Variables table with defaults
- Example playbook
- Testing instructions
- Architecture diagram (text-based)

Context: @roles/handshake/
```

### Troubleshooting
```
Debug this Ansible error and provide fix:

TASK [Deploy HSD container] ****
fatal: [dns-01]: FAILED! => {
  "msg": "Error starting container: 409 Client Error: Conflict"
}

Current task: @roles/handshake/tasks/install.yml (line 42)
Variables: @group_vars/all.yml
```

---

## Emergency Procedures

### Quick Fixes

When deployment fails, Gemini can generate recovery playbooks:

```yaml
---
# playbooks/emergency_rollback.yml
# Quick rollback for failed deployment

- name: Emergency Rollback
  hosts: "{{ target_host | default('all') }}"
  become: yes
  gather_facts: no
  
  vars:
    backup_suffix: ".backup.{{ ansible_date_time.epoch }}"
  
  tasks:
    - name: Stop all containers
      community.docker.docker_container:
        name: "{{ item }}"
        state: stopped
      loop:
        - hsd
        - unbound
        - caddy
      ignore_errors: yes
    
    - name: Restore configuration from backup
      ansible.builtin.copy:
        src: "{{ item }}.conf{{ backup_suffix }}"
        dest: "{{ item }}.conf"
        remote_src: yes
      loop:
        - /etc/unbound/unbound
        - /etc/caddy/Caddyfile
      ignore_errors: yes
    
    - name: Restart containers with old config
      community.docker.docker_container:
        name: "{{ item }}"
        state: started
      loop:
        - hsd
        - unbound
        - caddy
      ignore_errors: yes
    
    - name: Verify services are running
      ansible.builtin.wait_for:
        host: localhost
        port: "{{ item }}"
        timeout: 30
      loop:
        - 53
        - 443
        - 853
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-22 | Initial Gemini configuration for DNS infrastructure project |

---

## Continuous Improvement

Gemini should:
- Learn from successful patterns in the codebase
- Suggest refactoring opportunities
- Identify technical debt
- Propose automation enhancements
- Document lessons learned

**Feedback Format**:
```markdown
## Gemini Observation

**Date**: 2025-11-22
**Context**: [What task was being performed]

### Pattern Identified
[Description of code pattern or issue]

### Recommendation
[Specific improvement suggestion]

### Impact
[Expected benefit of implementing]
```

---

**This document should be reviewed and updated when Gemini's capabilities evolve or project requirements change.**
