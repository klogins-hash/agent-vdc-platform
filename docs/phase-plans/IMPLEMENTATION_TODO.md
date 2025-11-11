# Multi-Cloud VDC Implementation Todo List

## Phase 1: Provider Setup & Prerequisites
- [ ] Create Exoscale account (or verify existing)
  - [ ] Retrieve API credentials (key + secret)
  - [ ] Verify ch-dk-2 (Zurich) zone availability
  - [ ] Set up billing alerts

- [ ] Create Hetzner Cloud account (or verify existing)
  - [ ] Retrieve API token
  - [ ] Verify nbg1 (Nuremberg) availability
  - [ ] Set up billing alerts

- [ ] Configure OpenNebula (91.99.13.109)
  - [ ] Add Exoscale as cloud provider/driver
  - [ ] Add Hetzner as cloud provider/driver
  - [ ] Configure credentials in OpenNebula

## Phase 2: Network & Security Setup
- [ ] Create security groups
  - [ ] Exoscale security group: Allow NFS (2049), iSCSI (3260, 860), SSH (22)
  - [ ] Hetzner security group: Allow SSH (22), NFS client, IPSec ports (500, 4500)

- [ ] Set up VPN between clouds (choose one option)
  - [ ] Option A: IPSec VPN with strongSwan (~1 hour setup)
    - [ ] Configure IPSec on Exoscale NFS server
    - [ ] Configure IPSec on Hetzner VM
    - [ ] Test tunnel connectivity
  - [ ] Option B: GRE Tunnel (~15 min setup)
    - [ ] Create GRE tunnel on both sides
    - [ ] Verify 10.63.0.0/24 network connectivity
  - [ ] Option C: Use S3 for backups (no VPN, slower)

## Phase 3: Storage Infrastructure (Exoscale)
- [ ] Provision block storage
  - [ ] Create 50GB block storage volume in ch-dk-2
  - [ ] Create or find NFS server VM
  - [ ] Attach block storage to NFS VM
  - [ ] Format and prepare NFS exports

- [ ] Set up NFS service
  - [ ] Install NFS Server on Exoscale VM
  - [ ] Create /exports directory structure
  - [ ] Configure /etc/exports for Hetzner VM
  - [ ] Start & enable NFS service

- [ ] Set up S3 backups
  - [ ] Create S3 bucket: vdc-backups
  - [ ] Configure bucket versioning (keep 10+ versions)
  - [ ] Generate S3 API credentials (separate key)

## Phase 4: Compute Infrastructure (Hetzner)
- [ ] Provision VM
  - [ ] Create CX21 (2 vCPU, 4GB, 40GB NVMe) in nbg1-dc3
  - [ ] Configure public IP
  - [ ] Set up SSH key access
  - [ ] Record VM IP: _______________

- [ ] Install base system
  - [ ] Update system packages
  - [ ] Install Docker & Docker Compose
  - [ ] Install VPN software (strongSwan or GRE tools)
  - [ ] Install NFS client utilities
  - [ ] Install monitoring tools (htop, iotop)

## Phase 5: Mount Exoscale Storage on Hetzner
- [ ] Configure VPN/tunnel connection
  - [ ] Test connectivity between Exoscale & Hetzner
  - [ ] Verify route visibility (traceroute checks)
  - [ ] Monitor latency (ping Exoscale NFS server)

- [ ] Mount NFS on Hetzner
  - [ ] Create mount point: /mnt/exoscale-storage
  - [ ] Mount Exoscale NFS: mount -t nfs 192.168.0.10:/exports/vdc
  - [ ] Verify mount accessibility
  - [ ] Add to /etc/fstab for persistent mount

## Phase 6: Database Stack Deployment
- [ ] Prepare docker-compose.yaml
  - [ ] Update PostgreSQL volume paths → /mnt/exoscale-storage/postgres
  - [ ] Update Valkey volume paths → /mnt/exoscale-storage/valkey
  - [ ] Configure environment variables
  - [ ] Add health checks

- [ ] Deploy services
  - [ ] Run: docker-compose up -d
  - [ ] Verify all services running: docker ps
  - [ ] Check logs for errors: docker-compose logs -f
  - [ ] Test PostgreSQL connectivity
  - [ ] Test Valkey connectivity

- [ ] Configure PostgreSQL
  - [ ] Create application databases
  - [ ] Set up users & passwords
  - [ ] Enable backups: pg_dump automation
  - [ ] Enable replication (optional)

## Phase 7: MCP Server Deployment
- [ ] Deploy Node.js MCP server
  - [ ] Create /opt/mcp-server directory
  - [ ] Copy MCP server code
  - [ ] Install npm dependencies
  - [ ] Create systemd service file
  - [ ] Enable & start service

- [ ] Test MCP endpoints
  - [ ] Verify WebSocket connectivity: ws://hetzner-ip:3000
  - [ ] Test tool availability
  - [ ] Test database tool execution
  - [ ] Test cache operations

## Phase 8: Backup & Replication
- [ ] Set up automated backups
  - [ ] Create backup script for PostgreSQL
  - [ ] Schedule daily backups via cron
  - [ ] Configure S3 upload: aws s3 cp (to Exoscale)
  - [ ] Verify backups appear in Exoscale S3

- [ ] Set up database replication (optional)
  - [ ] Create publication on Hetzner PostgreSQL
  - [ ] Create secondary subscription (on Exoscale NFS server)
  - [ ] Verify replication lag
  - [ ] Test failover scenarios

## Phase 9: OpenNebula Integration
- [ ] Create OpenNebula templates
  - [ ] Template: Exoscale block storage provisioning
  - [ ] Template: Hetzner VM provisioning
  - [ ] Template: VPC/networking config
  - [ ] Template: Auto-scaling rules

- [ ] Configure monitoring in OpenNebula
  - [ ] Add Hetzner VM monitoring
  - [ ] Add Exoscale storage monitoring
  - [ ] Set up alerts for resource thresholds
  - [ ] Create dashboard view

## Phase 10: Testing & Validation
- [ ] Load testing
  - [ ] Simulate database writes (pgbench)
  - [ ] Verify replication performance
  - [ ] Monitor NFS latency
  - [ ] Check backup throughput

- [ ] Failover testing
  - [ ] Test Hetzner VM restart
  - [ ] Test NFS remount
  - [ ] Test backup restore
  - [ ] Verify data consistency

- [ ] Security validation
  - [ ] Verify VPN encryption (tcpdump)
  - [ ] Check firewall rules
  - [ ] Validate SSH key-only access
  - [ ] Check S3 bucket permissions

## Phase 11: Documentation & Runbooks
- [ ] Create operational documentation
  - [ ] How to scale compute (increase CX21 → CX31)
  - [ ] How to scale storage (increase block storage)
  - [ ] How to restore from backup
  - [ ] How to handle infrastructure failures

- [ ] Create troubleshooting guides
  - [ ] NFS mount failures
  - [ ] VPN connectivity issues
  - [ ] Database replication lag
  - [ ] Backup/restore procedures

## Phase 12: Cost Tracking & Optimization
- [ ] Set up cost monitoring
  - [ ] Track Exoscale monthly spend
  - [ ] Track Hetzner monthly spend
  - [ ] Monitor data transfer costs
  - [ ] Set budget alerts

- [ ] Optimize costs
  - [ ] Clean up unused snapshots
  - [ ] Archive old backups
  - [ ] Review reserved capacity options
  - [ ] Evaluate commitment discounts

---

## Implementation Status
- [ ] **Phase 1:** Not started
- [ ] **Phase 2:** Not started
- [ ] **Phase 3:** Not started
- [ ] **Phase 4:** Not started
- [ ] **Phase 5:** Not started
- [ ] **Phase 6:** Not started
- [ ] **Phase 7:** Not started
- [ ] **Phase 8:** Not started
- [ ] **Phase 9:** Not started
- [ ] **Phase 10:** Not started
- [ ] **Phase 11:** Not started
- [ ] **Phase 12:** Not started

## Quick Reference: Key IPs & Credentials
```
OpenNebula Frontend: 91.99.13.109 (root access)
Exoscale Zone: ch-dk-2 (Zurich)
Hetzner Zone: nbg1-dc3 (Nuremberg)
NFS Server IP (Exoscale): _______________
Hetzner VM Public IP: _______________
Hetzner VM Private IP: _______________
VPN Tunnel Network: 10.63.0.0/24 (if using GRE)
```

## Notes
- Total estimated monthly cost: ~$16 (Hetzner $6 + Exoscale $5 + misc $5)
- Setup time estimate: 4-8 hours (first time)
- Maintenance time: ~1 hour/month
- Backup retention: Keep 10+ daily versions in S3
