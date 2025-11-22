# Claude AI Assistant Configuration

**Version**: 1.0.0  
**Project**: Ansible Handshake DNS Infrastructure  
**Last Updated**: November 22, 2025

---

## Project Overview

This repository contains an Ansible playbook for deploying a decentralized DNS infrastructure combining:
- **Handshake (HNS)** blockchain for decentralized TLD resolution
- **Unbound** DNS server with DNS-over-TLS (DoT) and DNS-over-HTTPS (DoH)
- **Caddy** reverse proxy for HTTPS termination with Let's Encrypt automation
- **Quad9** secure upstream DNS resolution

---

## Primary Reference

**All AI agents should reference**: `@AGENTS.md` as the universal source of truth for project conventions, workflows, and standards.

This file contains Claude-specific context and optimizations that complement the universal agent guidelines.

---

## Claude-Specific Optimizations

### Conversation Management

**Context Window**: Claude has an extended context window suitable for:
- Processing multiple Ansible role files simultaneously
- Analyzing complex YAML playbooks with variable precedence
- Reviewing entire infrastructure configurations at once

**Recommended Context Loading Pattern**:
```
@AGENTS.md
@DEVELOPMENT_PLAN.md
@group_vars/all.yml
@inventory/production
@roles/[specific-role]/tasks/main.yml
```

### Ansible Best Practices for Claude

#### 1. YAML Analysis
- Validate indentation consistency (2 spaces standard)
- Check variable interpolation syntax `{{ variable }}`
- Verify Jinja2 template logic and filters
- Review task idempotency and conditionals

#### 2. Role Structure Review
When examining Ansible roles, analyze:
```yaml
roles/
├── role_name/
│   ├── tasks/main.yml          # Task definitions
│   ├── handlers/main.yml       # Event handlers
│   ├── templates/              # Jinja2 templates
│   ├── files/                  # Static files
│   ├── vars/main.yml           # Role variables
│   ├── defaults/main.yml       # Default variables
│   └── meta/main.yml           # Role metadata
```

#### 3. Docker Container Management
Focus on:
- Container networking configuration (`docker_network`)
- Volume mounting for persistence
- Environment variable injection
- Container health checks and restart policies
- Image version pinning vs. latest tags

#### 4. Security Considerations
Review and validate:
- TLS certificate management (Let's Encrypt automation)
- DNS query security (DoT/DoH implementation)
- Firewall rule configurations
- Secrets management (avoid plaintext credentials)
- Container user privileges (non-root execution)

### DNS Infrastructure Expertise

#### Handshake (HNS) Blockchain
```yaml
# Key configuration areas to understand:
hsd_rpc_port: 12037           # RPC interface port
hsd_ns_port: 5349             # Authoritative nameserver port
hsd_data_dir: /opt/hsd/data   # Blockchain data storage
```

**Claude should understand**:
- HNS blockchain synchronization process
- Decentralized TLD resolution mechanics
- HIP-5 protocol implementation (cross-protocol resolution)
- RPC API interaction patterns

#### Unbound DNS Server
```yaml
# Critical configuration patterns:
unbound_port: 53              # Standard DNS port
unbound_tls_port: 853         # DNS-over-TLS port
stub_zones:                   # HNS integration
  - name: "."
    stub_addr: "127.0.0.1@5349"
```

**Claude should validate**:
- Forward zone configurations
- DNSSEC validation settings
- Cache sizing and TTL strategies
- Access control lists (ACLs)
- Integration with Handshake stub zones

#### Caddy Reverse Proxy
```yaml
# DoH endpoint configuration:
caddy_doh_endpoint: /dns-query
caddy_tls_email: admin@example.com
```

**Claude should check**:
- Automatic HTTPS certificate provisioning
- Reverse proxy upstream configuration
- HTTP/2 and HTTP/3 support for DoH
- Health check endpoints
- Access logging and error handling

#### Quad9 Integration
```yaml
# Secure upstream DNS:
quad9_servers:
  - 9.9.9.9@853#dns.quad9.net
  - 149.112.112.112@853#dns.quad9.net
```

**Claude should verify**:
- TLS upstream connections
- Fallback server configuration
- DNS query filtering policies
- Privacy-preserving DNS practices

---

## Task-Specific Guidance

### When Asked to Review Ansible Playbooks

1. **Load Essential Context**:
   ```
   @playbooks/deploy.yml
   @group_vars/all.yml
   @inventory/production
   @ARCHITECTURE.md
   ```

2. **Validation Checklist**:
   - [ ] Playbook uses `hosts` and `become` correctly
   - [ ] Role dependencies properly declared
   - [ ] Variables follow precedence hierarchy
   - [ ] Tags applied for selective execution
   - [ ] Handlers triggered appropriately
   - [ ] Idempotency maintained throughout

3. **Suggest Improvements**:
   - Use `ansible-lint` output format for issues
   - Recommend role refactoring opportunities
   - Identify hard-coded values requiring variables
   - Suggest molecule testing scenarios

### When Asked to Create/Modify Roles

**Follow this structure**:

```yaml
# roles/example/tasks/main.yml
---
# Task file for example role

- name: Ensure Docker network exists
  community.docker.docker_network:
    name: "{{ docker_network }}"
    state: present
  tags: [setup, network]

- name: Deploy container with proper configuration
  community.docker.docker_container:
    name: "{{ container_name }}"
    image: "{{ container_image }}:{{ container_version }}"
    state: started
    restart_policy: unless-stopped
    networks:
      - name: "{{ docker_network }}"
    ports:
      - "{{ host_port }}:{{ container_port }}"
    volumes:
      - "{{ config_dir }}:/config:ro"
      - "{{ data_dir }}:/data:rw"
    env:
      CONTAINER_VAR: "{{ ansible_var }}"
  tags: [deploy, containers]

- name: Verify service health
  ansible.builtin.uri:
    url: "http://localhost:{{ health_check_port }}/health"
    status_code: 200
    timeout: 30
  retries: 5
  delay: 10
  tags: [verify]
```

**Key Principles**:
- Descriptive task names with clear intent
- Appropriate module usage (avoid `shell` when native modules exist)
- Proper variable interpolation
- Tag organization for workflow control
- Health checks and verification steps
- Meaningful retry/delay configurations

### When Asked to Debug Issues

1. **Gather Diagnostics**:
   ```bash
   # Check Ansible syntax
   ansible-playbook --syntax-check playbooks/deploy.yml
   
   # Run in check mode
   ansible-playbook --check playbooks/deploy.yml
   
   # Increase verbosity
   ansible-playbook -vvv playbooks/deploy.yml
   ```

2. **Common Issue Patterns**:
   - **Variable not defined**: Check precedence and scope
   - **Module errors**: Verify collection installation
   - **Connection failures**: Validate inventory and SSH access
   - **Idempotency failures**: Review task state management
   - **Docker issues**: Check daemon status and permissions

3. **Systematic Approach**:
   - Isolate failing task with tags
   - Review logs from target hosts
   - Validate variable substitution
   - Check ansible-lint warnings
   - Test with `--limit` for single host

### When Creating Documentation

**Generate documentation in this order**:

1. **README.md Updates**: High-level changes and user impact
2. **CHANGELOG.md**: Version-specific modifications
3. **Role Documentation**: In-role `README.md` or meta files
4. **Variable Documentation**: In `defaults/main.yml` comments
5. **Architecture Diagrams**: Text-based representations
6. **Troubleshooting Guides**: Common issues and solutions

**Format for variable documentation**:
```yaml
# defaults/main.yml

# Docker network name for DNS infrastructure
# All containers will be attached to this bridge network
# Default: dns_net
docker_network: "dns_net"

# Handshake node RPC port
# Used for blockchain interaction and TLD resolution
# Default: 12037
# Valid range: 1024-65535
hsd_rpc_port: 12037
```

---

## Integration with Multi-Agent Workflow

### Handoff to Architect Agent
```markdown
When architectural decisions are needed:
@architect Review infrastructure design for:
- Network topology and service interaction
- Scalability considerations for DNS load
- Security architecture for public DNS exposure
- Backup and disaster recovery strategy
```

### Handoff to Builder Agent
```markdown
For implementation tasks:
@builder Implement the following Ansible tasks:
- Role: [role_name]
- Objective: [specific goal]
- Acceptance criteria: [measurable outcomes]
- Reference: @DEVELOPMENT_PLAN.md (Phase X)
```

### Handoff to DevOps Agent
```markdown
For CI/CD and deployment:
@devops Configure automation for:
- Ansible playbook testing with Molecule
- GitHub Actions workflow for linting
- Integration testing in staging environment
- Blue-green deployment strategy
```

### Handoff to Validator Agent
```markdown
For testing and validation:
@validator Create test scenarios for:
- DNS resolution testing (HNS TLDs)
- DoT/DoH endpoint validation
- Load testing for query throughput
- Security scanning for exposed services
```

---

## Code Quality Standards

### Ansible Lint Configuration

Ensure `.ansible-lint` exists with:
```yaml
skip_list:
  - 'yaml[line-length]'  # Allow longer lines for readability

warn_list:
  - experimental  # Warn on experimental features

exclude_paths:
  - .cache/
  - .github/
  - test/

use_default_rules: true
```

### YAML Formatting

**Enforce these conventions**:
```yaml
# ✅ Correct: Dictionary format with colons
docker_container:
  name: hsd
  image: handshake/hsd:latest
  state: started

# ❌ Incorrect: Inline format for complex structures
docker_container: {name: hsd, image: handshake/hsd:latest}

# ✅ Correct: List with hyphens
packages:
  - docker
  - python3-docker
  - python3-pip

# ❌ Incorrect: Inline list format
packages: [docker, python3-docker, python3-pip]
```

### Variable Naming Conventions

```yaml
# ✅ Correct: snake_case with descriptive prefixes
hsd_rpc_port: 12037
unbound_cache_size: 100m
caddy_tls_email: admin@example.com

# ❌ Incorrect: Inconsistent naming
HSD_PORT: 12037
UnboundCacheSize: 100m
caddyEmail: admin@example.com
```

---

## Emergency Procedures

### When Playbook Execution Fails

1. **Immediate Actions**:
   - Note the failed task name and host
   - Check `--check` mode for safe preview
   - Review variable values with `-e` flag override

2. **Do NOT**:
   - Re-run without understanding failure cause
   - Modify production inventory without backup
   - Skip critical security tasks with tags
   - Ignore handler failures

3. **Recovery Pattern**:
   ```bash
   # 1. Isolate the issue
   ansible-playbook deploy.yml --limit failed-host --tags failing-task
   
   # 2. Increase verbosity
   ansible-playbook deploy.yml -vvv --limit failed-host
   
   # 3. Validate variables
   ansible all -m debug -a "var=problematic_variable"
   
   # 4. Test fix in check mode
   ansible-playbook deploy.yml --check --diff
   ```

### When Configuration Breaks DNS Resolution

**Critical Recovery**:
```yaml
# Rollback procedure:
- name: Restore previous configuration
  ansible.builtin.copy:
    src: /opt/backup/{{ item }}.conf
    dest: /opt/{{ item }}/{{ item }}.conf
    backup: yes
  loop:
    - unbound
    - caddy
  notify: restart dns services

- name: Restart services in safe order
  ansible.builtin.service:
    name: "{{ item }}"
    state: restarted
  loop:
    - unbound
    - caddy
    - hsd
```

---

## Performance Optimization Tips

### Ansible Execution Speed

1. **Use `strategy: free` for independent tasks**:
   ```yaml
   - hosts: dns_servers
     strategy: free  # Tasks run as soon as possible
     tasks: [...]
   ```

2. **Enable pipelining** in `ansible.cfg`:
   ```ini
   [ssh_connection]
   pipelining = True
   ```

3. **Limit facts gathering**:
   ```yaml
   - hosts: all
     gather_facts: no  # Skip if not needed
   ```

4. **Use async for long-running tasks**:
   ```yaml
   - name: Synchronize HNS blockchain
     ansible.builtin.command: hsd --daemon
     async: 3600  # 1 hour timeout
     poll: 0      # Fire and forget
     register: sync_job
   ```

---

## Documentation Standards

### Code Comments

**Use this format**:
```yaml
---
# Role: handshake
# Purpose: Deploy and configure Handshake (HNS) full node
# Dependencies: Docker, Python docker library
# Maintainer: DevOps Team

# ==============================================================================
# CONTAINER DEPLOYMENT
# ==============================================================================

- name: Deploy Handshake node container
  # Creates HNS full node with persistent blockchain storage
  # Exposes RPC (12037) and NS (5349) ports for Unbound integration
  community.docker.docker_container:
    name: hsd
    image: "{{ hsd_image }}:{{ hsd_version }}"
    # ... configuration ...
```

### README Updates

When modifying playbooks, update README with:
- **What changed**: Feature additions or modifications
- **Why it changed**: Business/technical justification
- **How to use**: Updated usage examples
- **Breaking changes**: Migration steps if applicable

---

## Learning Resources

**For Claude to reference when needed**:

- **Ansible Documentation**: https://docs.ansible.com/
- **Ansible Galaxy**: https://galaxy.ansible.com/ (for roles and collections)
- **Handshake Documentation**: https://hsd-dev.org/
- **Unbound Documentation**: https://unbound.docs.nlnetlabs.nl/
- **Caddy Documentation**: https://caddyserver.com/docs/
- **Docker Ansible Collection**: https://docs.ansible.com/ansible/latest/collections/community/docker/

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-22 | Initial Claude configuration for DNS infrastructure project |

---

## Feedback Loop

**Continuous Improvement**:
- Document Claude interaction patterns that work well
- Note areas requiring human clarification
- Suggest AGENTS.md updates for universal application
- Request additional context files as needed

---

**This document should be reviewed and updated quarterly or when significant project changes occur.**
Import command and agent standards from docs/claude/
