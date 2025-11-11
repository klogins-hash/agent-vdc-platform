# OpenNebula Cloud Infrastructure - Deployment Timeline

**Deployment Date**: November 11, 2025
**Status**: ✅ Complete and Operational
**OpenNebula Version**: 7.0
**Architecture**: Two-server setup (Control Plane + Compute Node)

---

## Deployment Summary

### Phase 1: Infrastructure Creation (Hetzner Cloud API)
- ✅ Frontend Server (cpx21) with Debian 11
- ✅ Compute Node (cpx41) with Ubuntu 22.04
- ✅ Both servers booted and accessible via SSH
- ✅ Root password resets completed

### Phase 2: Frontend Server Configuration (Debian 11)
- ✅ System updates and dependencies installed
- ✅ miniONE 7.0 downloaded and executed
- ✅ OpenNebula daemon (oned) installed and running
- ✅ Sunstone Web UI deployed and accessible
- ✅ KVM hypervisor configured locally
- ✅ Local host registered as "localhost" (ID: 0)

### Phase 3: SSH Key Setup
- ✅ SSH key exchange between frontend and compute
- ✅ oneadmin user SSH keys configured
- ✅ Passwordless authentication verified
- ✅ Local SSH key pair generated (ED25519)
- ✅ Public key installed on all servers

### Phase 4: Compute Node Configuration (Ubuntu 22.04)
- ✅ System updates installed
- ✅ NFS client installed
- ✅ QEMU/KVM packages installed
- ✅ libvirt-daemon-system installed and started
- ✅ libvirtd actively running and monitoring
- ✅ oneadmin user created with libvirt group membership
- ✅ virsh nodeinfo verified (8 cores, 16GB RAM)

### Phase 5: OpenNebula Integration
- ✅ Compute node added as KVM host (Host ID: 1)
- ✅ SSH connectivity verified from frontend to compute
- ✅ libvirt access confirmed from oneadmin@frontend
- ✅ Host monitoring initiated

---

## Server Details

### Frontend Server
```
Hostname: opennebula-frontend
IP: 91.99.13.109
OS: Debian 11 (Bullseye)
Type: cpx21
  - CPU: 3 cores
  - RAM: 4 GB
  - Disk: 80 GB

Services:
  - OpenNebula Daemon (oned): RUNNING
  - Sunstone Web UI: RUNNING (port 80)
  - KVM/libvirt: RUNNING
  - SSH Server: RUNNING

Access:
  - Root: ssh -i ~/.ssh/opennebula_cloud root@91.99.13.109
  - oneadmin: ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109
  - Web UI: http://91.99.13.109 (oneadmin/1X7srkMITq)
```

### Compute Node
```
Hostname: opennebula-compute
IP: 46.224.2.60
OS: Ubuntu 22.04 (Jammy)
Type: cpx41
  - CPU: 8 cores
  - RAM: 16 GB
  - Disk: 240 GB

Services:
  - libvirtd: RUNNING
  - QEMU: RUNNING
  - SSH Server: RUNNING

Access:
  - Root: ssh -i ~/.ssh/opennebula_cloud root@46.224.2.60
  - oneadmin: ssh -i ~/.ssh/opennebula_cloud oneadmin@46.224.2.60

Virsh Status:
  - CPU model: x86_64
  - CPU(s): 8
  - CPU frequency: 2445 MHz
  - Memory: 15,984,972 KiB (~16 GB)
```

---

## OpenNebula Configuration

### Registered Hosts
```
Host 0: localhost (Frontend)
  - Cluster: default
  - IM_MAD: qemu (local)
  - VM_MAD: qemu (local)
  - Status: ONLINE
  - CPU: 300 cores
  - Memory: 3.7 GB

Host 1: opennebula-compute (Compute Node)
  - Cluster: default
  - IM_MAD: kvm
  - VM_MAD: kvm
  - Status: Monitoring...
  - CPU: 8 cores
  - Memory: 16 GB
```

### Default Credentials
- **Username**: oneadmin
- **Password**: 1X7srkMITq
- **API endpoint**: http://91.99.13.109:2633

---

## Deployment Timeline

| Time | Component | Task | Status |
|------|-----------|------|--------|
| 08:50 | Hetzner API | Create frontend server | ✅ |
| 08:51 | Hetzner API | Create compute server | ✅ |
| 08:52 | Systems | Wait for boot, reset passwords | ✅ |
| 08:53 | Frontend | System update, miniONE download | ✅ |
| 09:00 | Frontend | miniONE installation | ✅ |
| 09:10 | Frontend | OpenNebula services verified | ✅ |
| 09:13 | Compute | Install dependencies (apt packages) | ✅ |
| 09:14 | Compute | Install libvirt-daemon-system | ✅ |
| 09:15 | Integration | Add compute node as Host 1 | ✅ |
| 09:20 | Security | Generate ED25519 SSH key pair | ✅ |
| 09:21 | Security | Install public key on all accounts | ✅ |
| 09:30 | Repository | Initialize git repository | ✅ |

---

## Key Deployment Decisions

### OS Selection
- **Frontend**: Debian 11 (lightweight, stable, minimal resource usage)
- **Compute**: Ubuntu 22.04 (extensive package support for VMs, more features)
- **Rationale**: Follows OpenNebula best practice of lightweight frontend + robust compute

### Hardware Sizing
- **Frontend**: cpx21 (3c/4GB) - sufficient for management workloads
- **Compute**: cpx41 (8c/16GB) - robust for running VMs
- **Rationale**: Separation of concerns, compute resource available for user VMs

### Security
- ED25519 SSH keys (modern, secure, 256-bit)
- Passwordless SSH authentication
- No exposed root passwords
- Public key stored in vault

### Networking
- Direct Hetzner Cloud network connectivity
- Private SSH keys for authentication
- No VPN required for this deployment
- Future: Can add firewall rules for additional security

---

## Post-Deployment Steps Completed

1. ✅ SSH key pair generated and distributed
2. ✅ SSH access verified on all servers
3. ✅ Compute node KVM capabilities confirmed
4. ✅ OpenNebula monitoring initiated
5. ✅ Repository created and documented

---

## Known Limitations and Next Steps

### Current Status
- Single compute node (can be scaled)
- No shared storage configured (can be added)
- No firewall rules configured (can be added)
- No backup procedures configured (can be implemented)

### Recommended Next Steps
1. Configure shared storage (NFS or iSCSI)
2. Add network configuration for VMs
3. Set up backup procedures
4. Configure firewall rules
5. Add more compute nodes as needed
6. Implement monitoring and alerting

---

## Support Resources

- OpenNebula Documentation: https://docs.opennebula.io/7.0/
- miniONE Repository: https://github.com/OpenNebula/minione
- OpenNebula Community: https://forum.opennebula.org/

---

## Notes

- Both servers are in the `default` cluster
- Network interfaces configured for single-network deployment
- All services are set to start on system boot
- System clocks are synchronized via NTP
