# Universal Agent Instructions

**Version**: 1.0.0  
**Project**: Ansible Handshake DNS Infrastructure  
**Last Updated**: November 22, 2025

---

## About This File

`AGENTS.md` is the **authoritative source of truth** for all AI coding assistants working on this project. Every AI agent (Claude, GitHub Copilot, Gemini, Cursor, Windsurf, Cline, etc.) should reference this file before making contributions.

**Tool-specific instructions** (CLAUDE.md, GEMINI.md, COPILOT.md) should enhance but never contradict the guidelines established here.

---

## Project Mission

Deploy a secure, decentralized DNS infrastructure that combines:
- **Handshake (HNS)** blockchain for censorship-resistant TLD resolution
- **Unbound** DNS server with modern privacy protocols (DoT/DoH)
- **Caddy** web server for automated HTTPS certificate management
- **Quad9** integration for secure traditional DNS fallback

**Target Audience**: System administrators and DevOps engineers deploying privacy-focused DNS infrastructure.

---

## Repository Structure

```
ansible-handshake-dns/
├── .github/
│   └── workflows/              # CI/CD automation
│       └── ansible-lint.yml    # Ansible quality checks
├── inventory/
│   ├── production              # Production servers
│   └── staging                 # Staging environment
├── group_vars/
│   └── all.yml                 # Global variables
├── host_vars/                  # Host-specific variables
├── roles/
│   ├── handshake/              # HNS blockchain node
│   │   ├── tasks/
│   │   ├── templates/
│   │   ├── defaults/
│   │   └── handlers/
│   ├── unbound/                # DNS resolver
│   ├── caddy/                  # Reverse proxy
│   └── common/                 # Shared configuration
├── playbooks/
│   ├── deploy.yml              # Main deployment
│   ├── update.yml              # Update existing deployment
│   └── rollback.yml            # Emergency rollback
├── AGENTS.md                   # This file (universal rules)
├── CLAUDE.md                   # Claude-specific instructions
├── GEMINI.md                   # Gemini-specific instructions
├── COPILOT.md                  # GitHub Copilot instructions
├── DEVELOPMENT_PLAN.md         # Roadmap and architecture
├── README.md                   # User-facing documentation
├── CONTRIBUTING.md             # Contribution guidelines
├── SECURITY.md                 # Security policies
├── ansible.cfg                 # Ansible configuration
└── requirements.yml            # Ansible Galaxy dependencies
```

---

## Core Principles

### 1. Infrastructure as Code (IaC)
- All infrastructure changes MUST be codified in Ansible
- NO manual server configuration without playbook update
- Version control ALL configuration changes
- Document WHY decisions were made, not just WHAT

### 2. Idempotency
- Every Ansible task MUST be idempotent (safe to run multiple times)
- Use appropriate module states (`present`, `absent`, `started`, `stopped`)
- Avoid `shell`/`command` modules unless absolutely necessary
- When using `shell`/`command`, implement `creates` or `changed_when`

### 3. Security First
- NO plaintext secrets in version control
- Use Ansible Vault for sensitive data
- Principle of least privilege for all services
- Regular security updates for all components
- Non-root container execution where possible

### 4. Testing Before Production
- Test in staging environment first
- Use `--check` mode for dry runs
- Validate with `--syntax-check` before commit
- Run `ansible-lint` to catch common issues

### 5. Documentation
- Update README.md for user-facing changes
- Comment complex Jinja2 logic in templates
- Explain variable purposes in `defaults/main.yml`
- Maintain CHANGELOG.md for releases

---

## Technology Stack

### Core Components

| Component | Version | Purpose | Port(s) |
|-----------|---------|---------|---------|
| **Handshake (HSD)** | Latest stable | Decentralized blockchain DNS | 12037 (RPC), 5349 (NS) |
| **Unbound** | 1.17+ | Recursive DNS resolver | 53 (DNS), 853 (DoT) |
| **Caddy** | 2.7+ | Reverse proxy & DoH endpoint | 443 (HTTPS), 80 (HTTP redirect) |
| **Docker** | 24.0+ | Container runtime | - |
| **Python** | 3.9+ | Ansible requirement | - |

### Ansible Dependencies

```yaml
# requirements.yml
collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: community.general
    version: ">=7.0.0"
```

**Installation**:
```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Development Workflow

### Standard Git Workflow

```mermaid
graph LR
    A[Create Feature Branch] --> B[Make Changes]
    B --> C[Test Locally]
    C --> D[Commit & Push]
    D --> E[Create Pull Request]
    E --> F[CI/CD Checks]
    F --> G[Code Review]
    G --> H[Merge to Main]
```

### Branch Naming Convention

```
feature/add-monitoring      # New functionality
bugfix/fix-dns-resolution   # Bug fixes
hotfix/security-patch       # Urgent production fixes
docs/update-readme          # Documentation only
refactor/role-structure     # Code improvements
```

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Example**:
```
feat(unbound): add DNS-over-HTTPS support

Implement DoH using Caddy as reverse proxy to Unbound.
Configure automatic HTTPS certificates via Let's Encrypt.

Closes #42
```

---

## Code Standards

### YAML Formatting

**Ansible Playbooks and Roles**:
```yaml
---
# ✅ CORRECT: Descriptive name, proper indentation
- name: Ensure Handshake container is running
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
  tags: [deploy, hsd]

# ❌ INCORRECT: Missing name, poor formatting
- docker_container: {name: hsd, image: handshake/hsd, state: started}
```

**Key Rules**:
- Use 2-space indentation (NO tabs)
- One task per block (no inline multi-task definitions)
- Always include descriptive `name:` field
- Use `{{ variable }}` for all variable references
- Quote variables in YAML values: `"{{ var }}"`
- Apply meaningful tags for selective execution

### Variable Naming

```yaml
# ✅ CORRECT: snake_case, clear scope prefix
hsd_rpc_port: 12037
unbound_cache_size: "100m"
caddy_tls_email: "admin@example.com"
docker_network: "dns_net"

# ❌ INCORRECT: Mixed case, unclear names
HSD_PORT: 12037
cacheSize: 100
email: admin@example.com
network: dns
```

**Naming Conventions**:
- `<component>_<purpose>_<detail>` format
- Use snake_case for all variables
- Boolean variables: `enable_<feature>`, `use_<option>`
- Lists: Plural nouns (`packages`, `services`, `ports`)
- Dictionaries: Singular nouns (`config`, `credentials`)

### Jinja2 Templates

```jinja2
{# ✅ CORRECT: Clear logic, proper spacing #}
{% if unbound_enable_dnssec %}
# DNSSEC validation enabled
auto-trust-anchor-file: "{{ unbound_trust_anchor }}"
val-permissive-mode: no
{% else %}
# DNSSEC validation disabled
val-permissive-mode: yes
{% endif %}

{# Use filters for transformations #}
server:
  interface: {{ unbound_interfaces | join('\n  interface: ') }}

{# ❌ INCORRECT: Poor readability #}
{%if unbound_enable_dnssec%}auto-trust-anchor-file:"{{unbound_trust_anchor}}"{%endif%}
```

### Directory Organization

**Role Structure** (standard Ansible layout):
```
roles/component_name/
├── tasks/
│   ├── main.yml          # Entry point, imports other task files
│   ├── install.yml       # Installation tasks
│   ├── configure.yml     # Configuration tasks
│   └── verify.yml        # Verification tasks
├── handlers/
│   └── main.yml          # Service restart handlers
├── templates/
│   └── config.j2         # Jinja2 configuration templates
├── files/
│   └── static_file       # Static files to copy
├── defaults/
│   └── main.yml          # Default variables (lowest precedence)
├── vars/
│   └── main.yml          # Role variables (higher precedence)
├── meta/
│   └── main.yml          # Role dependencies and metadata
└── README.md             # Role documentation
```

---

## Testing Requirements

### Pre-Commit Checks

**ALWAYS run before committing**:
```bash
# 1. Syntax validation
ansible-playbook --syntax-check playbooks/deploy.yml

# 2. Ansible lint
ansible-lint playbooks/ roles/

# 3. YAML lint (optional but recommended)
yamllint -c .yamllint .

# 4. Dry run (check mode)
ansible-playbook --check playbooks/deploy.yml -i inventory/staging
```

### Molecule Testing (Recommended)

For complex roles, use Molecule:
```bash
# Install molecule
pip install molecule molecule-docker ansible-lint

# Test role
cd roles/handshake
molecule test
```

### Integration Testing

**Verify deployed infrastructure**:
```bash
# Test DNS resolution (traditional)
dig @your-server.com example.com

# Test HNS TLD resolution
dig @your-server.com icann.

# Test DNS-over-TLS
kdig -d @your-server.com +tls example.com

# Test DNS-over-HTTPS
curl -H 'accept: application/dns-json' \
  'https://your-server.com/dns-query?name=example.com&type=A'
```

---

## Security Guidelines

### Secrets Management

**Use Ansible Vault for ALL sensitive data**:
```yaml
# group_vars/all.yml (encrypted with ansible-vault)
---
vault_acme_email: "admin@secure-domain.com"
vault_hsd_api_key: "super-secret-key-here"
vault_monitoring_token: "another-secret-token"
```

**Encrypt/Decrypt Commands**:
```bash
# Encrypt file
ansible-vault encrypt group_vars/all.yml

# Decrypt for editing
ansible-vault edit group_vars/all.yml

# Decrypt to view
ansible-vault view group_vars/all.yml

# Run playbook with vault password
ansible-playbook --ask-vault-pass playbooks/deploy.yml
```

### Firewall Configuration

**Restrict access to essential ports only**:
```yaml
# roles/common/tasks/firewall.yml
- name: Allow DNS traffic (UDP/TCP 53)
  ansible.builtin.ufw:
    rule: allow
    port: "53"
    proto: "{{ item }}"
  loop: [tcp, udp]

- name: Allow DNS-over-TLS (TCP 853)
  ansible.builtin.ufw:
    rule: allow
    port: "853"
    proto: tcp

- name: Allow HTTPS for DoH (TCP 443)
  ansible.builtin.ufw:
    rule: allow
    port: "443"
    proto: tcp

- name: Deny all other incoming by default
  ansible.builtin.ufw:
    state: enabled
    policy: deny
    direction: incoming
```

### Container Security

```yaml
# ✅ CORRECT: Non-root user, read-only filesystem where possible
- name: Deploy secure container
  community.docker.docker_container:
    name: unbound
    image: "{{ unbound_image }}"
    user: "{{ unbound_uid }}:{{ unbound_gid }}"  # Non-root
    read_only: yes                                # Read-only root FS
    security_opts:
      - no-new-privileges:true                    # Prevent privilege escalation
    cap_drop:
      - ALL                                       # Drop all capabilities
    cap_add:
      - NET_BIND_SERVICE                          # Only add needed caps

# ❌ INCORRECT: Running as root with unnecessary privileges
- name: Insecure container
  community.docker.docker_container:
    name: container
    privileged: yes  # NEVER do this unless absolutely necessary
```

---

## Commands Reference

### Common Ansible Commands

```bash
# ============================================
# DEPLOYMENT
# ============================================

# Full deployment to production
ansible-playbook -i inventory/production playbooks/deploy.yml

# Deploy specific role with tags
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --tags "unbound"

# Deploy to specific host
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --limit "dns-01.example.com"

# Dry run (no changes made)
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --check --diff

# ============================================
# UPDATES
# ============================================

# Update specific component
ansible-playbook -i inventory/production playbooks/update.yml \
  --tags "hsd" --extra-vars "hsd_version=v6.0.0"

# Rolling update (one host at a time)
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --serial 1

# ============================================
# DEBUGGING
# ============================================

# High verbosity
ansible-playbook -i inventory/production playbooks/deploy.yml -vvv

# Step-by-step execution (confirm each task)
ansible-playbook -i inventory/production playbooks/deploy.yml --step

# Start at specific task
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --start-at-task="Configure Unbound"

# ============================================
# INFORMATION GATHERING
# ============================================

# List all hosts
ansible all -i inventory/production --list-hosts

# Check connectivity
ansible all -i inventory/production -m ping

# Get system facts
ansible all -i inventory/production -m setup

# View variable values
ansible all -i inventory/production -m debug \
  -a "var=hsd_rpc_port"
```

### Docker Management Commands

```bash
# ============================================
# CONTAINER OPERATIONS
# ============================================

# List running containers
docker ps

# View container logs
docker logs -f hsd

# Execute command in container
docker exec -it hsd /bin/sh

# Inspect container configuration
docker inspect hsd

# ============================================
# MAINTENANCE
# ============================================

# Remove unused containers and images
docker system prune -a

# View resource usage
docker stats

# Restart specific container
docker restart unbound
```

### DNS Testing Commands

```bash
# ============================================
# STANDARD DNS QUERIES
# ============================================

# Basic DNS query
dig @localhost example.com

# Query specific record type
dig @localhost example.com MX

# Query HNS TLD
dig @localhost icann.

# Trace DNS resolution path
dig @localhost +trace example.com

# ============================================
# SECURE DNS PROTOCOLS
# ============================================

# DNS-over-TLS (using kdig from knot-dnsutils)
kdig -d @dns-server.com +tls example.com

# DNS-over-HTTPS (JSON API)
curl -H 'accept: application/dns-json' \
  'https://dns-server.com/dns-query?name=example.com&type=A'

# DNS-over-HTTPS (binary format)
curl -H 'accept: application/dns-message' \
  --data-binary @query.bin \
  https://dns-server.com/dns-query

# ============================================
# PERFORMANCE TESTING
# ============================================

# Benchmark DNS server
dnsperf -d queryfile -s dns-server.com -p 53

# Check response time
time dig @dns-server.com example.com
```

---

## Troubleshooting Guide

### Common Issues and Solutions

#### Issue: Ansible "Unreachable" errors

**Symptoms**: `UNREACHABLE! => {"changed": false, "msg": "Failed to connect"}`

**Solutions**:
```bash
# 1. Verify SSH connectivity
ssh -i ~/.ssh/id_rsa user@target-host

# 2. Check inventory configuration
ansible-inventory -i inventory/production --list

# 3. Test with ping module
ansible all -i inventory/production -m ping

# 4. Use verbose mode to see connection details
ansible-playbook playbooks/deploy.yml -vvv
```

#### Issue: Docker container fails to start

**Symptoms**: Container repeatedly crashes or exits immediately

**Solutions**:
```bash
# 1. Check container logs
docker logs hsd

# 2. Inspect container configuration
docker inspect hsd

# 3. Verify volume mounts exist and have correct permissions
ls -la /opt/hsd/data

# 4. Try running container manually for debugging
docker run -it --entrypoint /bin/sh handshake/hsd

# 5. Check Docker network
docker network inspect dns_net
```

#### Issue: DNS queries not resolving

**Symptoms**: `dig` commands timeout or return SERVFAIL

**Solutions**:
```bash
# 1. Check if Unbound is running
docker ps | grep unbound

# 2. View Unbound logs
docker logs unbound

# 3. Test Unbound configuration
docker exec unbound unbound-checkconf

# 4. Verify port bindings
netstat -tuln | grep -E '53|853'

# 5. Check firewall rules
sudo ufw status

# 6. Test upstream connectivity
docker exec unbound ping -c 3 9.9.9.9
```

#### Issue: Let's Encrypt certificate provisioning fails

**Symptoms**: Caddy cannot obtain HTTPS certificates

**Solutions**:
```bash
# 1. Check Caddy logs
docker logs caddy

# 2. Verify DNS points to your server
dig +short your-domain.com

# 3. Ensure port 80 and 443 are accessible
curl -I http://your-domain.com
curl -I https://your-domain.com

# 4. Check ACME email is valid in configuration
grep acme_email group_vars/all.yml

# 5. Review Caddy configuration
docker exec caddy cat /etc/caddy/Caddyfile
```

#### Issue: HNS blockchain sync is slow or stuck

**Symptoms**: Handshake node not syncing or taking very long

**Solutions**:
```bash
# 1. Check HNS sync status
docker exec hsd hsd-cli info

# 2. View HNS logs for errors
docker logs hsd | grep -i error

# 3. Verify RPC connection
curl -u x:api-key -X POST http://localhost:12037 \
  -d '{"method":"getinfo","params":[],"id":1}'

# 4. Check peer connections
docker exec hsd hsd-cli getpeerinfo

# 5. Consider using a snapshot for faster sync
# (Download trusted blockchain snapshot and mount as volume)
```

---

## Emergency Procedures

### Rollback Procedure

**If deployment causes issues**:
```bash
# 1. Immediate rollback to previous state
ansible-playbook -i inventory/production playbooks/rollback.yml

# 2. If playbook unavailable, manual rollback:
# Stop new containers
ansible all -i inventory/production -m docker_container \
  -a "name=hsd state=stopped"

# Start backup containers
ansible all -i inventory/production -m docker_container \
  -a "name=hsd-backup state=started"

# 3. Restore configuration from backup
ansible all -i inventory/production -m copy \
  -a "src=/opt/backup/unbound.conf dest=/opt/unbound/unbound.conf backup=yes"

# 4. Restart affected services
ansible all -i inventory/production -m service \
  -a "name=unbound state=restarted"
```

### Data Recovery

**Backup locations**:
```yaml
backup_directories:
  - /opt/hsd/data          # Blockchain data
  - /opt/unbound/conf      # Unbound configuration
  - /opt/caddy/config      # Caddy configuration
  - /opt/certs             # TLS certificates
```

**Restore procedure**:
```bash
# 1. Stop services
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --tags "stop"

# 2. Restore from backup
ansible all -i inventory/production -m synchronize \
  -a "src=/backup/hsd-data/ dest=/opt/hsd/data/"

# 3. Verify permissions
ansible all -i inventory/production -m file \
  -a "path=/opt/hsd/data owner=hsd group=hsd recurse=yes"

# 4. Restart services
ansible-playbook -i inventory/production playbooks/deploy.yml \
  --tags "start"
```

---

## Performance Optimization

### Ansible Performance Tuning

**ansible.cfg optimizations**:
```ini
[defaults]
# Increase parallelism (default: 5)
forks = 20

# Use faster connection method
transport = ssh

# Enable SSH pipelining (reduces SSH connections)
pipelining = True

# Disable fact gathering if not needed
gathering = explicit

# Use callback plugins for better output
stdout_callback = yaml

[ssh_connection]
# Reuse SSH connections
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

### DNS Server Performance

**Unbound tuning** (in templates):
```yaml
# templates/unbound.conf.j2

server:
  # Number of threads (match CPU cores)
  num-threads: {{ ansible_processor_vcpus }}
  
  # Increase cache sizes
  msg-cache-size: {{ unbound_msg_cache_size | default('100m') }}
  rrset-cache-size: {{ unbound_rrset_cache_size | default('200m') }}
  
  # Prefetch frequently accessed records
  prefetch: yes
  prefetch-key: yes
  
  # Optimize for high query rate
  so-rcvbuf: 4m
  so-sndbuf: 4m
  
  # TCP connection limits
  incoming-num-tcp: 1000
  outgoing-num-tcp: 10000
```

---

## Multi-Agent Collaboration

### Agent Roles and Handoffs

This project supports multiple AI agents with specialized roles:

#### @architect
**When to invoke**: Architecture decisions, infrastructure design, scalability planning

**Example handoff**:
```
@architect Review infrastructure design for:
- Multi-region deployment strategy
- Load balancing for DNS queries
- High availability failover configuration
- Database/storage architecture for HNS blockchain
```

#### @builder
**When to invoke**: Implementation of new features, role development, code writing

**Example handoff**:
```
@builder Implement the following:
- Role: monitoring
- Objective: Add Prometheus metrics collection for Unbound
- Acceptance criteria:
  - Expose metrics endpoint on port 9167
  - Include query rate, cache hit ratio, response time
  - Integrate with existing Docker network
- Reference: @DEVELOPMENT_PLAN.md (Phase 3)
```

#### @validator
**When to invoke**: Testing, quality assurance, security audits

**Example handoff**:
```
@validator Create test suite for:
- DNS resolution for HNS TLDs (.hns domains)
- DoT/DoH endpoint functionality
- Certificate renewal automation
- Load testing for 10,000 qps
- Security scanning for exposed services
```

#### @devops
**When to invoke**: CI/CD pipeline, deployment automation, monitoring setup

**Example handoff**:
```
@devops Configure automation for:
- GitHub Actions workflow for ansible-lint
- Automated staging deployment on PR merge
- Blue-green production deployment
- Prometheus + Grafana monitoring stack
- Automated backup scheduling
```

#### @scribe
**When to invoke**: Documentation updates, README improvements, changelog maintenance

**Example handoff**:
```
@scribe Document the following changes:
- New DoH endpoint configuration
- Updated deployment procedure for v2.0
- Troubleshooting guide for common DNS issues
- Update CHANGELOG.md with release notes
```

---

## Continuous Improvement

### Agent Feedback Loop

**After completing major tasks, agents should**:
1. Document any challenges encountered
2. Suggest improvements to this AGENTS.md file
3. Identify missing documentation
4. Propose automation opportunities

**Format for suggestions**:
```markdown
# Improvement Suggestion

**Date**: 2025-11-22
**Agent**: Claude/Gemini/Copilot
**Area**: [Deployment/Testing/Documentation]

## Current State
[What exists now]

## Proposed Improvement
[What should change]

## Rationale
[Why this would help]

## Implementation Effort
[Low/Medium/High]
```

### Project Metrics to Track

- Deployment success rate
- Time to deploy (baseline and improvements)
- Ansible lint warnings/errors trend
- Test coverage percentage
- Documentation completeness

---

## Questions and Clarifications

**If you're unsure about something**:

1. **Check existing documentation**:
   - README.md for user guidance
   - DEVELOPMENT_PLAN.md for architecture
   - Role-specific README files

2. **Search codebase**:
   ```bash
   # Find variable usage
   grep -r "hsd_rpc_port" .
   
   # Find task examples
   grep -r "docker_container" roles/
   ```

3. **Consult authoritative sources**:
   - Ansible documentation: https://docs.ansible.com/
   - Component docs (linked in README)

4. **Ask for human clarification**:
   ```
   I need clarification on:
   - [Specific question]
   - Why: [Context]
   - Impact: [What depends on this]
   ```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-11-22 | Initial universal agent guidelines for DNS infrastructure project |

---

## Maintenance

This file should be reviewed and updated:
- When major architectural changes occur
- Quarterly as part of technical debt review
- When new AI agents are introduced to the project
- When team workflows significantly change

**Last Review**: 2025-11-22  
**Next Review**: 2026-02-22  
**Maintained By**: DevOps Team
