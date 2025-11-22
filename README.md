# Decentralized DNS Infrastructure with Handshake, Unbound, and Caddy

![Ansible](https://img.shields.io/badge/Ansible-2.10+-green)
![Docker](https://img.shields.io/badge/Docker-20.10+-blue)

An Ansible playbook to deploy a secure, decentralized DNS infrastructure combining:
- **Handshake (HNS)** blockchain for decentralized TLD resolution
- **Unbound** DNS server with DNS-over-TLS (DoT) and DNS-over-HTTPS (DoH)
- **Caddy** reverse proxy for HTTPS termination
- **Quad9** for secure upstream DNS resolution

## Features

### Core DNS Services
✅ **Handshake Full Node** - Authoritative resolution for HNS TLDs

✅ **Unbound DNS** - Caching resolver with DoT/DoH support

✅ **Caddy 2** - Modern DoH endpoint with Let's Encrypt automation

✅ **Quad9 Integration** - Secure DNS-over-TLS upstream

✅ **HIP-5 Protocol** - Cross-protocol resolution (ENS, IPFS, Tor)

### Security Features
🔒 **DNSSEC Validation** - Full DNSSEC support with auto-trust anchors

🔒 **Security Hardening** - Glue protection, referral path validation, algorithm downgrade protection

🔒 **Firewall Configuration** - UFW-based firewall with rate limiting

🔒 **Rate Limiting** - DDoS protection for both DNS and DoH endpoints

🔒 **Container Health Checks** - Automated health monitoring for all services

🔒 **Privacy Features** - Query minimization, identity hiding

### Operational Excellence
📊 **Automated Backups** - Daily backups of wallets, configs, and certificates

📊 **CI/CD Pipeline** - GitHub Actions for linting, syntax checking, and security scanning

📊 **Comprehensive Logging** - Structured logging with performance profiling

📊 **Ansible Best Practices** - Tags, pre/post-tasks, validation, and error handling

## Software Versions

This playbook uses the following stable software versions:

| Component | Version | Release Date | Notes |
|-----------|---------|--------------|-------|
| **Handshake (HSD)** | v8.0.0 | Aug 2024 | Requires database migration |
| **Unbound** | 1.21.0 | Aug 2024 | Latest stable from NLnet Labs |
| **Caddy** | 2.10.2 | Nov 2024 | Official v2 with auto HTTPS |
| **community.docker** | >=5.0.2 | Nov 2024 | Requires ansible-core >=2.17.0 |
| **community.crypto** | >=3.0.5 | Oct 2024 | For certificate management |

### HSD v8.0.0 Migration Requirements

**⚠️ IMPORTANT**: If upgrading from a previous HSD version:

1. **Backup your wallet first**:
   ```bash
   docker exec hsd hsd-cli wallet backup --name=backup-$(date +%Y%m%d)
   ```

2. **First run requires migration flags**:
   ```bash
   # The playbook will handle this, but if running manually:
   hsd --chain-migrate=4 --wallet-migrate=7
   ```

3. **Migration is automatic** when deploying with this playbook, but allow extra time for the first deployment after upgrade.

## Architecture

```
[Client Devices]
    |
    | DoH/DoT
    ↓
[Caddy (443/tcp)]  ← Let's Encrypt Certificates
    | ↑
    | | Decrypted DNS
    ↓ |
[Unbound (53/udp, 853/tcp)]
    ├─→ [Handshake Full Node (.hns TLDs)]
    └─→ [Quad9 (9.9.9.9:853) - All other TLDs]
```

## Prerequisites

- Ansible 2.10+
- Target system:
  - Ubuntu 20.04+/Debian 11+
  - 2 vCPU, 4GB RAM, 50GB storage
  - Open ports: 53/udp, 53/tcp, 443/tcp, 853/tcp

## Installation

```
# Clone repository
git clone https://github.com/yourusername/ansible-handshake-dns.git
cd ansible-handshake-dns

# Configure inventory
cp inventory/production.example inventory/production
nano inventory/production  # Add your servers

# Edit group variables
nano group_vars/all.yml

# Install dependencies
ansible-galaxy install -r requirements.yml

# Deploy full infrastructure
ansible-playbook -i inventory/production playbooks/deploy_dns.yml

# Deploy specific components using tags
ansible-playbook -i inventory/production playbooks/deploy_dns.yml --tags "firewall,docker,hsd"

# Run only configuration changes
ansible-playbook -i inventory/production playbooks/deploy_dns.yml --tags "configure"
```

### Selective Deployment with Tags

The playbook supports granular control through tags:

```bash
# Available tags:
# - install: Install packages and dependencies
# - configure: Configure services
# - deploy: Deploy containers
# - verify: Verify deployment
# - firewall: Firewall configuration only
# - docker: Docker setup only
# - hsd, unbound, caddy: Individual services
# - security: Security-related tasks

# Examples:
# Deploy only DNS services (skip firewall)
ansible-playbook -i inventory/production playbooks/deploy_dns.yml --skip-tags "firewall"

# Update configurations without reinstalling
ansible-playbook -i inventory/production playbooks/deploy_dns.yml --tags "configure"

# Verify deployment health
ansible-playbook -i inventory/production playbooks/deploy_dns.yml --tags "verify"
```

## Configuration

### Key Variables (`group_vars/all.yml`)
```
# Network
docker_network: "dns_net"

# Quad9 DNS
quad9_servers:
  - 9.9.9.9@853#dns.quad9.net
  - 149.112.112.112@853#dns.quad9.net

# Certificates
acme_email: "admin@yourdomain.hns"

# HIP-5 Protocols
hip5_protocols: ["_eth", "_ipfs", "_tor"]
hip5_resolvers:
  eth: "https://ethresolver.yourdomain.hns"
  ipfs: "https://ipfsgateway.yourdomain.hns"
```

**Note**: Replace placeholder values (`yourusername`, `yourdomain.hns`, etc.) with your actual information before use.

### Optional: Backup Configuration

To enable automated backups, add a backup role to your playbook:

```yaml
# In playbooks/deploy_dns.yml, add:
- role: backup
  tags: [backup, optional]
```

Configure backup settings in `group_vars/dns_servers.yml`:

```yaml
# Backup configuration
backup_dir: /opt/backups/dns
backup_retention_days: 30
backup_hsd_wallet: true
backup_configurations: true
backup_certificates: true

# Optional remote backup
backup_remote_enabled: false
backup_remote_type: "s3"  # or "rsync"
backup_remote_destination: "s3://my-bucket/dns-backups"
```

Run manual backup:
```bash
/usr/local/bin/dns-backup.sh
```

### Security Configuration

#### Firewall Rules

The firewall role automatically configures UFW with:
- Allow: 22/tcp (SSH, rate-limited), 53/udp+tcp (DNS), 443/tcp (HTTPS), 853/tcp (DoT)
- Deny: All other incoming traffic
- Rate limiting on SSH to prevent brute force attacks

#### DNSSEC Validation

DNSSEC is enabled by default in Unbound with:
- Auto-trust anchor updates
- Strict validation for signed zones
- Protection against algorithm downgrades
- Glue record validation

Verify DNSSEC is working:
```bash
dig @your-server dnssec.works +dnssec
# Look for "ad" flag in the response
```

#### Rate Limiting

Protection against DDoS attacks:
- **Unbound**: 1000 queries/sec per IP, 100 total IP connections
- **Caddy**: 100 requests/minute per IP for DoH endpoint
- **UFW**: Connection rate limiting on SSH

## Verification

```
# Test Handshake resolution
dig @your-server +short icann.

# Test standard DNS over DoT
kdig @your-server -p 853 google.com. +tls

# Test DoH endpoint
curl -H 'accept: application/dns-json' \
  'https://your-server/dns-query?name=example.com&type=A'
```

## Security Considerations

1. **Certificate Management**
   - Uses Let's Encrypt with auto-renewal
   - Certificates stored in `/opt/certs`
2. **Network Isolation**
   - Dedicated Docker bridge network
   - Firewall rules recommended for public exposure
3. **Regular Updates**
   - Container versions are pinned for reproducibility
   - Review [Software Versions](#software-versions) section before upgrading
   - Test in non-production environment first
   ```bash
   # Deploy with updated versions
   ansible-playbook -i inventory/production playbooks/deploy.yml
   ```

## Troubleshooting

**Common Issues**:
- **Port Conflicts**: Ensure host ports 53/udp and 443/tcp are free
- **Certificate Errors**: Verify ACME email in `group_vars/all.yml`
- **HIP-5 Resolution**: Check `docker logs hsd`

**Diagnostic Commands**:
```
# Check container status
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# View Unbound logs
docker logs unbound | grep -i error

# Test DoH directly
curl -v -H 'accept: application/dns-message' \
  --data-binary @query.bin https://your-server/dns-query
```

## Development & CI/CD

### Continuous Integration

This project includes a GitHub Actions CI/CD pipeline that automatically:
- Lints all Ansible playbooks with `ansible-lint`
- Validates YAML syntax with `yamllint`
- Performs syntax checking on playbooks
- Runs security scans with Trivy
- Executes dry-run deployments in check mode

The CI pipeline runs on:
- All pushes to `main` and `develop` branches
- All pull requests
- Manual workflow dispatch

### Local Development

Before committing, run local checks:

```bash
# Install development dependencies
pip install ansible ansible-lint yamllint

# Run linting
ansible-lint playbooks/
yamllint .

# Syntax check
ansible-playbook playbooks/deploy_dns.yml --syntax-check

# Dry run (check mode)
ansible-playbook playbooks/deploy_dns.yml --check --diff
```

### Container Health Monitoring

All containers include health checks that run every 30 seconds:

```bash
# Check container health status
docker ps --format "table {{.Names}}\t{{.Status}}"

# View health check logs
docker inspect hsd --format='{{json .State.Health}}' | jq
docker inspect unbound --format='{{json .State.Health}}' | jq
docker inspect caddy --format='{{json .State.Health}}' | jq
```

### Logging

Ansible execution logs are stored in `ansible.log` with:
- Task timing information (profile_tasks callback)
- Total execution time (timer callback)
- All task results and changes

Container logs:
```bash
# View container logs
docker logs -f hsd
docker logs -f unbound
docker logs -f caddy

# Follow all logs
docker logs -f --tail=100 hsd unbound caddy
```

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/improvement`)
3. Run local linting and tests
4. Commit changes (`git commit -am 'Add some feature'`)
5. Push to branch (`git push origin feature/improvement`)
6. Open Pull Request (CI will run automatically)

All contributions must pass CI checks before merging.

## License

MIT License - See [LICENSE](LICENSE) for details
