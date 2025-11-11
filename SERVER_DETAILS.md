# Server Details & Credentials

## Frontend Server (Control Plane)

### Basic Information
- **Name**: opennebula-frontend
- **IP Address**: 91.99.13.109
- **IPv6**: 2a01:4f8:c012:1e1f::/64
- **Operating System**: Debian 11 (Bullseye)
- **Type**: Hetzner cpx21
- **Location**: fsn1 (Falkenstein, Germany)

### Hardware Specifications
```
CPU Cores:    3
CPU Type:     x86_64
RAM:          4 GB
Disk Space:   80 GB
Disk Type:    SSD
```

### Credentials
```
Root Username:     root
Root Password:     veiMv4jppVkv
OpenNebula User:   oneadmin
OpenNebula Pass:   1X7srkMITq
```

### Access Methods

#### SSH Access (with key)
```bash
ssh -i ~/.ssh/opennebula_cloud root@91.99.13.109
ssh -i ~/.ssh/opennebula_cloud oneadmin@91.99.13.109
```

#### SSH Access (with password)
```bash
ssh root@91.99.13.109
# Password: veiMv4jppVkv
```

#### Web UI (Sunstone)
```
URL:      http://91.99.13.109
Port:     80
Username: oneadmin
Password: 1X7srkMITq
```

### Services & Ports

| Service | Port | Status |
|---------|------|--------|
| OpenNebula Daemon | 2633 | ✅ Running |
| Sunstone Web UI | 80 | ✅ Running |
| SSH | 22 | ✅ Running |
| libvirtd | 16509 | ✅ Running |

### Important Directories

```
OpenNebula Home:      /var/lib/one/
OpenNebula Config:    /etc/one/
OpenNebula Logs:      /var/log/one/
VM Templates:         /var/lib/one/templates/
Images Directory:     /var/lib/one/images/
SSH Keys (oneadmin):  /var/lib/one/.ssh/
```

---

## Compute Node (Workload Manager)

### Basic Information
- **Name**: opennebula-compute
- **IP Address**: 46.224.2.60
- **IPv6**: 2a01:4f8:c012:416d::/64
- **Operating System**: Ubuntu 22.04 (Jammy)
- **Type**: Hetzner cpx41
- **Location**: fsn1 (Falkenstein, Germany)

### Hardware Specifications
```
CPU Cores:    8
CPU Type:     x86_64
RAM:          16 GB
Disk Space:   240 GB
Disk Type:    SSD
Network:      1 Gbps
```

### Credentials
```
Root Username: root
Root Password: rNJeCRR3tTCP
oneadmin User: oneadmin
```

### Access Methods

#### SSH Access (with key)
```bash
ssh -i ~/.ssh/opennebula_cloud root@46.224.2.60
ssh -i ~/.ssh/opennebula_cloud oneadmin@46.224.2.60
```

#### SSH Access (with password)
```bash
ssh root@46.224.2.60
# Password: rNJeCRR3tTCP
```

### Services & Ports

| Service | Port | Status |
|---------|------|--------|
| libvirtd | 16509 | ✅ Running |
| QEMU | Various | ✅ Running |
| SSH | 22 | ✅ Running |
| NFS Client | 2049 | ✅ Ready |

### Important Directories

```
libvirt Config:     /etc/libvirt/
libvirt Storage:    /var/lib/libvirt/
QEMU Images:        /var/lib/libvirt/images/
SSH Keys (oneadmin): /var/lib/one/.ssh/
VMs Directory:      /var/lib/one/
```

---

## SSH Key Information

### Local Machine

**Available SSH Keys for This Infrastructure:**

```
Private Key: ~/.ssh/opennebula_cloud
Public Key:  ~/.ssh/opennebula_cloud.pub
Key Type:    ED25519 (256-bit)
Comment:     opennebula-cloud-access
```

### Authorized Key Installation

The public key has been installed in:
- **Frontend**:
  - `/root/.ssh/authorized_keys`
  - `/var/lib/one/.ssh/authorized_keys`

- **Compute Node**:
  - `/root/.ssh/authorized_keys`
  - `/var/lib/one/.ssh/authorized_keys`

---

## Network Configuration

### Frontend
```
Interface:    eth0
IP Address:   91.99.13.109
Netmask:      255.255.255.0
Gateway:      91.99.13.1
DNS:          8.8.8.8, 8.8.4.4
IPv6:         2a01:4f8:c012:1e1f::1/64
```

### Compute Node
```
Interface:    eth0
IP Address:   46.224.2.60
Netmask:      255.255.255.0
Gateway:      46.224.2.1
DNS:          8.8.8.8, 8.8.4.4
IPv6:         2a01:4f8:c012:416d::1/64
```

---

## OpenNebula Configuration

### API Endpoint
```
URL:     http://91.99.13.109:2633/RPC2
Port:    2633
Protocol: XML-RPC
Auth:    oneadmin:1X7srkMITq
```

### Registered Hosts

**Host 0 (Frontend - Local)**
```
ID:         0
Name:       localhost
Cluster:    default
Hypervisor: qemu (local)
IM_MAD:     qemu
VM_MAD:     qemu
Status:     ONLINE
```

**Host 1 (Compute Node)**
```
ID:         1
Name:       opennebula-compute
Cluster:    default
Hypervisor: kvm
IM_MAD:     kvm
VM_MAD:     kvm
Status:     Monitoring
CPU Cores:  8
Memory:     16 GB
```

---

## Backup & Recovery

### Important System Files to Backup
```
/etc/one/                  # OpenNebula configuration
/var/lib/one/.ssh/         # SSH keys for inter-server communication
/var/lib/one/templates/    # VM templates
/var/lib/one/datastores/   # VM storage
```

### Backup Location (Local)
```
None configured yet - Recommended: Use automated backup solution
```

---

## Monitoring & Logs

### OpenNebula Logs
```
Main Log:      /var/log/one/oned.log
Error Log:     /var/log/one/oned.error.log
Scheduler Log: /var/log/one/sched.log
```

### System Monitoring
```
Command: systemctl status [service]

Available services:
  - opennebula
  - libvirtd
  - sshd
```

---

## Security Notes

⚠️ **IMPORTANT SECURITY CONSIDERATIONS:**

1. ✅ Use SSH keys instead of passwords
2. ✅ Never share private keys
3. ⚠️ Rotate credentials periodically
4. ⚠️ Use firewall rules to restrict access
5. ⚠️ Enable monitoring for suspicious activity
6. ⚠️ Keep systems updated
7. ⚠️ Use strong passwords for oneadmin user
8. ⚠️ Restrict API access to trusted networks

---

## Emergency Access

### If SSH key access is lost:

1. Contact Hetzner support for recovery console
2. Reset root password via Hetzner dashboard
3. Use password authentication temporarily
4. Regenerate SSH keys
5. Update authorized_keys files

### Recovery Command (if needed)
```bash
# On recovery console as root
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKPIiWgjZ1UqonNKv+GKRZuEw5YVoEadn2Rbdy4NWKUV opennebula-cloud-access" >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```
