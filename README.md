# Decentralized DNS Infrastructure with Handshake, Unbound, and Caddy

![Ansible](https://img.shields.io/badge/Ansible-2.10+-green)
![Docker](https://img.shields.io/badge/Docker-20.10+-blue)

An Ansible playbook to deploy a secure, decentralized DNS infrastructure combining:
- **Handshake (HNS)** blockchain for decentralized TLD resolution
- **Unbound** DNS server with DNS-over-TLS (DoT) and DNS-over-HTTPS (DoH)
- **Caddy** reverse proxy for HTTPS termination
- **Quad9** for secure upstream DNS resolution

## Features

✅ **Handshake Full Node** - Authoritative resolution for HNS TLDs

✅ **Unbound DNS** - Caching resolver with DoT/DoH support

✅ **Caddy 2** - Modern DoH endpoint with Let's Encrypt automation

✅ **Docker Containers** - Isolated services with compose-free deployment

✅ **Quad9 Integration** - Secure DNS-over-TLS upstream

✅ **HIP-5 Protocol** - Cross-protocol resolution (ENS, IPFS, Tor)

✅ **Ansible Best Practices** - Role-based structure with linting compliance

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

# Deploy infrastructure
ansible-playbook -i inventory/production playbooks/deploy.yml
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

## Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/improvement`)
3. Commit changes (`git commit -am 'Add some feature'`)
4. Push to branch (`git push origin feature/improvement`)
5. Open Pull Request

## License

MIT License - See [LICENSE](LICENSE) for details
