# Database Stack VDC - Complete Setup Guide

## 📋 Overview

This is a complete, reusable infrastructure-as-code setup for deploying a **single virtual machine** containing all database services, accessible via **MCP (Model Context Protocol)** and **WebSocket API** for AI agent integration.

**Services included in single VM:**
- PostgreSQL (Primary SQL Database)
- Valkey/Redis (In-Memory Cache)
- RabbitMQ (Message Queue)
- MCP Server + WebSocket (AI Agent API)

---

## 🚀 Quick Start

### Option 1: Full Automation (Recommended)

```bash
# 1. Provision the VM on Exoscale
/tmp/provision_single_vm.sh

# After VM is running, SSH into it:
ssh root@<VM_IP>

# 2. Inside the VM, run automated setup (uses cloud-init or direct install)
bash /tmp/install_all_services.sh

# 3. Verify all services are running
curl http://localhost:3000/health
```

### Option 2: Cloud-Init Automation

```bash
# Modify provision script to use cloud-init:
exo compute instance create database-stack-vm-1 \
  --template ad4d213f-8e6c-4817-8579-201a7b29ed0b \
  --security-group db-stack-sg \
  --instance-type standard.medium \
  --disk-size 100 \
  --cloud-init /tmp/cloud-init.yaml
```

---

## 📦 Files Included

### Provisioning Scripts

**`/tmp/provision_single_vm.sh`**
- Creates security group with proper firewall rules
- Launches single Debian 12 VM on Exoscale
- Outputs VM IP for next steps
- Size: standard.medium (2 CPU, 4GB RAM)

**`/tmp/install_all_services.sh`**
- Installs all database services
- Sets up MCP server with systemd service
- Creates Node.js application
- Ready to run on VM

**`/tmp/cloud-init.yaml`**
- Cloud-init configuration for automated setup
- Can be used with `--cloud-init` flag
- Provisions entire environment on VM boot
- Includes all services + MCP server

### Cleanup & Management

**`/tmp/cleanup_auto_shutdown.sh`**
- Removes all Exoscale resources
- Stops all running services
- Deletes VM and security group
- One-command cleanup

---

## 🔧 Architecture

```
┌─────────────────────────────────────────────┐
│     Single VM Instance (Exoscale)           │
│  database-stack-vm-1 (standard.medium)      │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ PostgreSQL (port 5432)               │  │
│  │ - Primary relational database        │  │
│  │ - Connection pool management         │  │
│  └──────────────────────────────────────┘  │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ Valkey/Redis (port 6379)             │  │
│  │ - Session & data caching             │  │
│  │ - Real-time data structures          │  │
│  └──────────────────────────────────────┘  │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ RabbitMQ (ports 5672, 15672)         │  │
│  │ - Message queueing                   │  │
│  │ - Async job processing               │  │
│  │ - Management UI on 15672              │  │
│  └──────────────────────────────────────┘  │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │ MCP Server (port 3000)               │  │
│  │ - WebSocket API for AI agents        │  │
│  │ - Tool interface for database ops    │  │
│  │ - Health check endpoint              │  │
│  └──────────────────────────────────────┘  │
│                                             │
└─────────────────────────────────────────────┘
         ↓
   Security Group: db-stack-sg
   - Allow SSH (port 22)
   - Allow PostgreSQL (5432)
   - Allow Valkey (6379)
   - Allow RabbitMQ (5672, 15672)
   - Allow MCP/WebSocket (3000, 8080)
```

---

## 📊 Service Details

### PostgreSQL
```
- Admin user: postgres
- Password: postgres
- Port: 5432
- Connection string: postgresql://postgres:postgres@localhost:5432/postgres
```

### Valkey (Redis)
```
- Port: 6379
- No authentication by default
- Commands: GET, SET, DEL, MGET, MSET, etc.
```

### RabbitMQ
```
- AMQP Port: 5672
- Management UI: http://localhost:15672
- Username: admin
- Password: password
```

### MCP Server
```
- Protocol: WebSocket
- Port: 3000
- URL: ws://localhost:3000
- Health Check: http://localhost:3000/health
```

---

## 🤖 MCP Server Tools

The MCP server exposes three main tools for AI agents:

### 1. **query_postgres**
Execute SQL queries on PostgreSQL

```json
{
  "tool": "query_postgres",
  "query": "SELECT * FROM table_name",
  "database": "postgres"
}
```

**Response:**
```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "example" }
  ]
}
```

### 2. **cache_operation**
Interact with Valkey/Redis cache

```json
{
  "tool": "cache_operation",
  "operation": "set",
  "key": "user:123",
  "value": "John Doe"
}
```

**Supported Operations:** `get`, `set`, `del`

### 3. **queue_message**
Send/receive messages via RabbitMQ

```json
{
  "tool": "queue_message",
  "operation": "send",
  "queue": "tasks",
  "message": "Process order #123"
}
```

**Supported Operations:** `send`, `receive`

---

## 🔌 WebSocket Connection Example

```python
import asyncio
import json
import websockets

async def send_query():
    async with websockets.connect('ws://localhost:3000') as websocket:
        # Execute PostgreSQL query
        request = {
            "tool": "query_postgres",
            "query": "SELECT COUNT(*) as count FROM table_name"
        }

        await websocket.send(json.dumps(request))
        response = await websocket.recv()
        print("Response:", json.loads(response))

asyncio.run(send_query())
```

---

## 📈 Scaling & Cost Management

### Instance Types
- **nano**: 1 CPU, 512 MB RAM (minimum, $3/month)
- **small**: 2 CPU, 2 GB RAM (test, $8/month)
- **medium**: 2 CPU, 4 GB RAM (production, $12/month)
- **large**: 4 CPU, 8 GB RAM (high-load, $20/month)

Current setup uses: **standard.medium**

### Cost Estimate (monthly)
- VM (standard.medium): ~$12/month
- Data transfer: Variable (usually <$5/month for test)
- Total: ~$17/month

### To reduce costs:
```bash
# Modify provision script to use nano instance:
# --instance-type standard.nano \
```

---

## ⚠️ Production Considerations

For production deployments, consider:

1. **Database Backups**
   - Configure automated PostgreSQL backups
   - Use point-in-time recovery
   - Test restore procedures regularly

2. **High Availability**
   - Deploy multiple instances behind load balancer
   - Use RabbitMQ clustering
   - Configure PostgreSQL replication

3. **Security**
   - Change default RabbitMQ password
   - Configure PostgreSQL user permissions
   - Use VPN/private networks
   - Enable encryption in transit

4. **Monitoring**
   - Set up Prometheus/Grafana
   - Monitor service health
   - Configure alerts
   - Log aggregation

5. **Capacity Planning**
   - Monitor CPU/Memory usage
   - Right-size instances for workload
   - Plan for growth
   - Test scalability

---

## 🛠️ Troubleshooting

### Services Not Starting

```bash
# Check service status
systemctl status postgresql
systemctl status rabbitmq-server
systemctl status valkey-server
systemctl status mcp-server

# View logs
journalctl -u postgresql -n 50
journalctl -u mcp-server -n 50
```

### Connection Issues

```bash
# Test PostgreSQL connection
psql -U postgres -h localhost

# Test Valkey connection
redis-cli -h localhost ping

# Test RabbitMQ connection
curl http://localhost:15672/api/connections

# Test MCP Server
curl http://localhost:3000/health
```

### Memory Issues

```bash
# Check available memory
free -h

# Check service memory usage
ps aux | grep postgres
ps aux | grep rabbitmq
ps aux | grep node
```

---

## 📝 Usage Examples

### Create Database and Table

```python
import psycopg2

conn = psycopg2.connect("dbname=postgres user=postgres password=postgres host=localhost")
cur = conn.cursor()
cur.execute("CREATE TABLE users (id SERIAL, name VARCHAR(100))")
cur.execute("INSERT INTO users (name) VALUES ('Alice')")
conn.commit()
```

### Cache Data in Valkey

```python
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)
r.set('user:1', 'Alice')
value = r.get('user:1')
print(value)  # Output: Alice
```

### Queue Messages in RabbitMQ

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='tasks')
channel.basic_publish(exchange='', routing_key='tasks', body='Task data')
connection.close()
```

---

## 🗑️ Cleanup When Done

```bash
# Run cleanup script to remove all resources
/tmp/cleanup_auto_shutdown.sh

# Or manually:
exo compute instance delete database-stack-vm-1 --force
exo compute security-group delete db-stack-sg
```

---

## 📞 Support & Documentation

- **OpenNebula**: https://docs.opennebula.io/
- **Exoscale**: https://www.exoscale.com/
- **PostgreSQL**: https://www.postgresql.org/docs/
- **RabbitMQ**: https://www.rabbitmq.com/documentation.html
- **Valkey**: https://valkey.io/
- **MCP Protocol**: https://spec.modelcontextprotocol.io/

---

## ✅ Next Steps

1. **Deploy**: Run `/tmp/provision_single_vm.sh`
2. **Install**: SSH and run `/tmp/install_all_services.sh`
3. **Verify**: Check health with `curl http://localhost:3000/health`
4. **Integrate**: Connect your AI agent to WebSocket endpoint
5. **Done**: Services are ready for use!

---

**Last Updated**: November 11, 2025
**Creator**: Cline AI Assistant
**Template**: Database Stack VDC v1.0
