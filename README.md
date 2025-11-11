# OpenNebula Cloud Infrastructure

A production-grade two-server OpenNebula cloud infrastructure deployed on Hetzner Cloud with:
- **Lightweight Control Plane**: Debian 11 (3 cores, 4GB RAM)
- **Robust Compute Node**: Ubuntu 22.04 (8 cores, 16GB RAM)

## Quick Access

| Component | Details |
|-----------|---------|
| **Frontend** | 91.99.13.109 (oneadmin / 1X7srkMITq) |
| **Compute** | 46.224.2.60 |
| **Web UI** | http://91.99.13.109 |
| **SSH Key** | `~/.ssh/opennebula_cloud` |

## Repository Structure

```
.
├── README.md                      # This file
├── DEPLOYMENT.md                  # Deployment details and timeline
├── SERVER_DETAILS.md              # Server credentials and specifications
├── SSH_KEYS.md                    # SSH key information (public key reference)
├── ARCHITECTURE.md                # System architecture diagrams
├── USAGE_GUIDE.md                 # How to use the infrastructure
├── scripts/
│   ├── deploy.sh                  # Deployment automation script
│   ├── monitor.sh                 # Host monitoring script
│   └── backup.sh                  # Backup configuration
├── terraform/
│   ├── main.tf                    # Hetzner infrastructure definition
│   ├── variables.tf               # Terraform variables
│   └── outputs.tf                 # Terraform outputs
├── docs/
│   ├── SCALING.md                 # How to add more compute nodes
│   ├── TROUBLESHOOTING.md         # Common issues and solutions
│   └── MAINTENANCE.md             # Regular maintenance procedures
├── .gitignore                     # Git ignore rules
└── data/
    └── deployment-log.txt         # Deployment log and timeline

```

## Key Features

✅ **Two-Server Architecture**
- Lightweight frontend for management
- Robust compute node for workloads
- Both Debian and Ubuntu officially supported by OpenNebula 7.0

✅ **Security**
- ED25519 SSH key-based authentication
- No password-based SSH access
- Secure communication between servers

✅ **Scalability**
- Additional compute nodes can be added
- Distributed resource management
- Support for multiple hypervisors (KVM, LXC, QEMU)

✅ **Management**
- Sunstone Web UI for visual management
- OpenNebula CLI for advanced users
- Real-time monitoring and resource allocation

## Getting Started

### Prerequisites
- SSH key: `~/.ssh/opennebula_cloud` (stored locally)
- Basic familiarity with OpenNebula or Cloud 101 knowledge

### Accessing the Infrastructure

```bash
# SSH into frontend as oneadmin
ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109

# SSH into compute node
ssh -i ~/.ssh/opennebula_cloud root@46.224.2.60

# Access Web UI
# Open browser: http://91.99.13.109
# Login: oneadmin / 1X7srkMITq
```

## Documentation

- **[DEPLOYMENT.md](./DEPLOYMENT.md)** - Detailed deployment information
- **[SERVER_DETAILS.md](./SERVER_DETAILS.md)** - Server specifications and access
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - System architecture and design
- **[USAGE_GUIDE.md](./USAGE_GUIDE.md)** - How to deploy VMs and manage resources
- **[docs/SCALING.md](./docs/SCALING.md)** - How to add more compute nodes
- **[docs/TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md)** - Common issues and fixes

## Deployment Details

- **Deployment Date**: November 11, 2025
- **OpenNebula Version**: 7.0
- **Frontend OS**: Debian 11 (Bullseye)
- **Compute OS**: Ubuntu 22.04 (Jammy)
- **Hypervisor**: KVM with libvirt

## Quick Operations

```bash
# List all hosts
ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109 'onehost list'

# List VM templates
ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109 'onetemplate list'

# List running VMs
ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109 'onevm list'

# View compute node details
ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109 'onehost show 1'
```

## Support & Resources

- **OpenNebula Documentation**: https://docs.opennebula.io/
- **Community Forum**: https://forum.opennebula.org/
- **GitHub**: https://github.com/OpenNebula/one

## Security Notes

⚠️ **IMPORTANT**:
- Store SSH private key in secure vault
- Rotate credentials periodically
- Keep OpenNebula updated
- Monitor logs regularly
- Use firewall rules to restrict access

## License

This infrastructure documentation is provided as-is for internal use.

## Contact

For questions or issues regarding this deployment, refer to the troubleshooting documentation or contact your cloud administrator.
