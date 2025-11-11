# Single-Tenant Agent VDC: Per-Customer Isolated Instances

## 🎯 **The Model: One Bucket Per Customer**

```
Customer A          Customer B          Customer C
└─ Private VDC  │  └─ Private VDC   │  └─ Private VDC
   ├─ Data Bucket   │   ├─ Data Bucket  │   ├─ Data Bucket
   ├─ AI Agents     │   ├─ AI Agents    │   ├─ AI Agents
   └─ Knowledge Graph   └─ Knowledge Graph  └─ Knowledge Graph

(Completely isolated, zero data leakage)

Enterprise Tier: Add structured schema layer on top
Casual Tier: Just dump and let agents organize
```

---

## 💡 **Why Single-Tenant is Perfect**

### **Casual User ($10-20/mo)**
```
"Here's my stuff. Organize it however you want."

System:
1. Dump everything into /data bucket
2. AI agents run continuously
3. Auto-build knowledge graph
4. Auto-generate summaries
5. Auto-detect relationships
6. User queries: "Show me important things"
   → AI figures out what's important
```

### **Enterprise User ($50-100/mo)**
```
Same as above, PLUS:

+ Custom schema (if they want structure)
+ Role-based access control
+ Custom categorization logic
+ Audit trails
+ Data governance
+ Export in any format
+ But default: also just dump & organize
```

---

## 🏗️ **Single-Tenant VDC Architecture**

```
┌──────────────────────────────────────────────┐
│         Customer's Private VDC                │
│         (Hetzner CX21, $6/mo)                 │
├──────────────────────────────────────────────┤
│                                               │
│  ┌─────────────────────────────────────┐    │
│  │  Data Bucket (MinIO)                 │    │
│  │  ├─ Raw uploads                      │    │
│  │  ├─ Processing queue                 │    │
│  │  └─ Organized results                │    │
│  └─────────────────────────────────────┘    │
│                                               │
│  ┌─────────────────────────────────────┐    │
│  │  Qdrant (Vector DB)                  │    │
│  │  └─ Growing embeddings               │    │
│  └─────────────────────────────────────┘    │
│                                               │
│  ┌─────────────────────────────────────┐    │
│  │  Neo4j (Graph DB)                    │    │
│  │  └─ Auto-discovered relationships    │    │
│  └─────────────────────────────────────┘    │
│                                               │
│  ┌─────────────────────────────────────┐    │
│  │  Agent Workers (24/7))               │    │
│  │  ├─ Ingest Agent                     │    │
│  │  ├─ Processing Agents                │    │
│  │  ├─ Graph Builder                    │    │
│  │  └─ Learning Agent (improves over time)  │
│  └─────────────────────────────────────┘    │
│                                               │
│  ┌─────────────────────────────────────┐    │
│  │  Query API                           │    │
│  │  └─ User/Agent interface             │    │
│  └─────────────────────────────────────┘    │
│                                               │
└────────────┬───────────────────────────────┘
             │ NFS Mount
       Exoscale Storage ($2-5/mo per customer)
       └─ Backups, Archives, LongTerm Storage
```

---

## 💰 **Per-Customer Economics**

### **Casual Tier ($12/mo minimum)**
```
Hetzner CX21:           $6/mo
  └─ 2 vCPU, 4GB RAM

Exoscale Storage 10GB:  $2/mo
  └─ Backups, rarely accessed

Shared Infrastructure:  $4/mo
  └─ Auth, billing, monitoring
─────────────────────────────
TOTAL: $12/mo per customer
```

**Can host 5-10 casual customers on single Hetzner by sharing VMs**

### **Pro Tier ($25/mo)**
```
Hetzner CX21 (dedicated): $6/mo
Exoscale Storage 50GB:    $5/mo
Premium support:          $10/mo
Advanced features:        $4/mo
─────────────────────────────
TOTAL: $25/mo per customer
```

### **Enterprise Tier ($100/mo)**
```
Hetzner CX31:            $13/mo
Exoscale Storage 200GB:  $15/mo
Dedicated support:       $30/mo
Custom schema layer:     $20/mo
Audit/compliance:        $22/mo
─────────────────────────────
TOTAL: $100/mo per customer
```

---

## 🔄 **Background Agent Workers (24/7)**

### **Agent 1: Ingest Agent**
```python
async def ingest_agent():
    while True:
        # Check upload bucket for new files
        new_files = await minio.list_bucket('uploads', added_since=last_check)

        for file in new_files:
            # Extract text/metadata
            content = await extract_text(file)

            # Create initial embeddings
            embeddings = await embedding_api.embed(content)

            # Store in Qdrant
            await qdrant.upsert(
                vector=embeddings,
                metadata={
                    'source': file,
                    'file_type': file.type,
                    'timestamp': now(),
                    'raw_text': content[:500]  # snippet
                }
            )

            # Move to processed bucket
            await minio.move('uploads', 'processed', file)

        await asyncio.sleep(5)  # Check every 5 seconds
```

**What it does:**
- ✅ Watches upload bucket for new files
- ✅ Extracts text from PDFs, images, documents
- ✅ Creates embeddings immediately
- ✅ Stores in vector DB for fast search
- ✅ Runs continuously in background

---

### **Agent 2: Relationship Agent**
```python
async def relationship_agent():
    while True:
        # Find entities that don't have relationships yet
        unlinked_entities = await neo4j.query("""
            MATCH (n:Entity)
            WHERE NOT (n)-[:RELATED_TO]->()
            RETURN n LIMIT 10
        """)

        for entity in unlinked_entities:
            # Use LLM to find related entities
            similar = await qdrant.search(entity.name, top_k=5)

            for match in similar:
                # Infer relationship type
                relationship = await claude.infer_relationship(
                    entity.name,
                    match.entity
                )

                # Store in graph
                await neo4j.add_relationship(
                    entity,
                    relationship['type'],
                    match.entity,
                    {'confidence': relationship['confidence']}
                )

        await asyncio.sleep(3600)  # Run every hour
```

**What it does:**
- ✅ Analyzes disconnected entities
- ✅ Uses LLM to infer relationships
- ✅ Continuously improves knowledge graph
- ✅ Adds confidence scores
- ✅ Runs in background while user works

---

### **Agent 3: Learning Agent**
```python
async def learning_agent():
    """Continuously improves understanding of user's data"""
    while True:
        # Analyze what user is querying
        recent_queries = await query_log.get_last_100()

        # Find gaps in knowledge graph
        gaps = await identify_gaps(recent_queries)

        # Focus on what user cares about
        for gap in gaps:
            # Re-analyze data with focus on user's interest
            await deep_analyze(gap)

            # Create new entities/relationships
            new_relationships = await neo4j.expand_graph(gap)

        # Optimize embeddings based on usage
        await optimize_embeddings_for_user()

        await asyncio.sleep(7200)  # Run every 2 hours
```

**What it does:**
- ✅ Learns what user cares about
- ✅ Focuses processing on relevant data
- ✅ Continually expands knowledge graph
- ✅ Gets smarter over time
- ✅ Personalized to each customer

---

## 📥 **Data Ingestion Patterns**

### **Pattern 1: Dump Bucket (Simplest)**
```
User drops files into: /uploads/
System sees new file → Processes automatically
Result: File is searchable, embedded, graphed
Time: 30 seconds to 5 minutes depending on size
```

### **Pattern 2: Batch Upload (TBs)**
```
User: "Process my entire email archive"
System:
1. Upload to S3 multi-part
2. Queue for processing
3. Agents work 24/7 on it
4. Progress bar shows completion
5. Eventually: All 500k emails searchable
```

### **Pattern 3: Streaming (Real-time)**
```
API: POST /api/ingest/stream
Body: Continuous stream of data

System:
1. Chunks data
2. Embeds in real-time
3. Updates graph as you speak
4. Agent: "I found something related..."
```

### **Pattern 4: API Integration (Sync)**
```
Connect to: Email, Slack, Notion, Google Drive
System:
1. Syncs periodically
2. Only pulls new items
3. Auto-integrates into knowledge graph
4. Ready for queries
```

---

## 🔍 **Query Patterns**

### **Casual User Queries**
```
"Show me important stuff from this month"
System response:
  ├─ Top concepts (by frequency)
  ├─ Key people mentioned
  ├─ Major decisions made
  ├─ Timeline of events
  └─ "Agent notes: You interact with Bob 80% of the time"

"Tell me about [topic]"
System response:
  ├─ All mentions of topic
  ├─ Related people/places
  ├─ Timeline evolution
  ├─ Key quotes
  └─ Summaries from your own words
```

### **Enterprise User Queries**
```
Same as above, PLUS custom:

"Show me Q3 financial data"
System response:
  ├─ Revenue breakdown
  ├─ Expense analysis
  ├─ Variance to forecast
  ├─ Structured reports (via schema layer)
  └─ Audit trail of what data was used

"Export Q3 as XLSX"
System response:
  ├─ Pulls structured schema
  ├─ Fills with data from graphs
  ├─ Formats as requested
  └─ Ready for finance software
```

---

## 🚀 **Deploying Multiple Customer VDCs**

### **Architecture for 100s of Customers**

```
┌────────────────────────────────────────────────┐
│        Shared Control Plane                     │
│    (OpenNebula, 91.99.13.109)                  │
├────────────────────────────────────────────────┤
│                                                 │
│  Manages & monitors customer VDCs             │
│  - Billing                                     │
│  - User authentication                        │
│  - License management                         │
│  - Monitoring & alerts                        │
│  - Backups                                    │
│                                                 │
└───┬──────────────┬──────────────┬──────────────┘
    │              │              │
    ▼              ▼              ▼
Customer A    Customer B    Customer C
  VDC           VDC           VDC
Hetzner     Hetzner     Hetzner
CX21        CX21        CX21
$6/mo       $6/mo       $6/mo
```

### **Resource Pooling (Cost Sharing)**

**Option 1: Dedicated Per Customer (Simple)**
```
1 Hetzner CX21 = 1 Customer
Cost: $6/mo compute (customer pays)
Setup: 15 minutes per customer
Isolation: Complete
Scaling: Add new VMs as needed
```

**Option 2: Shared Resources (Cost Efficient)**
```
1 Hetzner CX41 ($29/mo) = 10-20 casual customers
Each customer gets isolated namespace (Docker)
But shares underlying hardware

Isolation: Namespace-level (Docker containers)
Cost per customer: $1.5-3/mo (much cheaper!)
Scaling: Add resources only when needed
Risk: Noisy neighbor problem
```

**Option 3: Hybrid (Recommended)**
```
Entry Tier: Shared CX41 ($1.50/mo per customer)
Pro Tier: Shared CX31 ($5/mo per customer)
Enterprise: Dedicated CX31 ($13/mo per customer)
```

---

## 📊 **Operating Model**

### **Customer Lifecycle**

```
1. SIGNUP
   └─ OpenNebula provisions VDC for them
   └─ Customer gets private dashboard

2. ONBOARDING
   └─ Upload guide
   └─ Sample data processor
   └─ First embeddings ready in minutes

3. USAGE
   └─ Agents work 24/7 in background
   └─ Customer queries anytime
   └─ Results get better daily
   └─ Optional: Subscribe to structured layer

4. GROWTH
   └─ Upgrade to larger VM if needed
   └─ More storage for longer-term data
   └─ Custom schema (enterprise tier)
   └─ Custom integrations

5. DEPARTURE
   └─ Data export (everything)
   └─ Clean shutdown & destroy VDC
   └─ Deletion from Exoscale
```

---

## ⚙️ **System Resilience**

### **Fault Tolerance Per Customer**

```
Customer's data in Exoscale NFS:
├─ Daily snapshots (7 day retention)
├─ Weekly archives (30 day retention)
└─ Monthly cold storage (1 year retention)

If Hetzner VM dies:
1. OpenNebula detects failure
2. Automatically provisions new VM
3. Mounts NFS from Exoscale
4. Services restart from state
5. Agent workers resume where they left
6. Time to recovery: ~5 minutes
```

---

## 🔐 **Security Model (Per Customer)**

```
Customer A Data:
├─ Stored in: /data/customer-a/
├─ Encrypted: At rest (AES-256)
├─ Encrypted: In transit (TLS)
├─ Isolated: Separate Docker namespace
├─ Networked: Private VPN (VPC-like)
└─ Access: API key + OAuth (their own auth)

Customer B Data:
├─ Completely separate
├─ Zero cross-contamination
├─ Can't see Customer A's anything
└─ Independent encryption keys
```

---

## 📈 **Scaling Scenarios**

### **Scenario 1: 10 Casual Users ($120/mo revenue)**
```
Infrastructure:
├─ 2x Hetzner CX21 (shared casual tier)
├─ 1x Hetzner CX21 (staging/dev)
├─ Exoscale storage: 500GB total
└─ Total cost: $35/mo

Margin: $120 - $35 = $85/mo (71% margin)
Per-user cost: $3.50
```

### **Scenario 2: 100 Mixed Users**
```
├─ 50 Casual ($12/mo each): $600
├─ 40 Pro ($25/mo each): $1,000
├─ 10 Enterprise ($100/mo each): $1,000
= $2,600/mo revenue

Infrastructure:
├─ 10x Casual VMs (shared):     $30/mo
├─ 5x Pro VMs (semi-dedicated):  $45/mo
├─ 10x Enterprise VMs:          $130/mo
├─ Exoscale storage:            $200/mo
├─ Control plane & monitoring:  $50/mo
= ~$455/mo cost

Margin: $2,600 - $455 = $2,145/mo (82% margin!)
```

### **Scenario 3: 1000 Users (Growth)**
```
Revenue: $20,000-30,000/mo
Infrastructure: $3,000-5,000/mo
Margin: $15,000-25,000/mo (75-83%)

At this scale:
├─ Managing with OpenNebula automation
├─ Kubernetes for orchestration (optional)
├─ CDN for data delivery
└─ Multiple cloud regions
```

---

## 🎯 **Implementation Roadmap**

### **Phase 1: Single VDC (Week 1)**
- [x] Design agent-optimized stack
- [ ] Build background agents (ingest, relationship, learning)
- [ ] Deploy to single Hetzner VM
- [ ] Test with casual user workflow
- [ ] Verify cost structure

### **Phase 2: Multi-Customer Control Plane (Week 2)**
- [ ] Update OpenNebula to provision VDCs
- [ ] Create customer dashboard
- [ ] Implement billing system
- [ ] Build user authentication
- [ ] Create support playbooks

### **Phase 3: Onboarding & UX (Week 3)**
- [ ] Design "dump bucket" UI
- [ ] Create tutorial process
- [ ] Build data import tools
- [ ] Test casual user experience
- [ ] Create documentation

### **Phase 4: Enterprise Features (Week 4)**
- [ ] Build structured schema layer
- [ ] Create role-based access
- [ ] Add audit trails
- [ ] Build export tools
- [ ] Create compliance docs

### **Phase 5: Production Hardening (Week 5)**
- [ ] Load testing
- [ ] Security audit
- [ ] Disaster recovery testing
- [ ] Performance tuning
- [ ] Monitoring setup

---

## 💬 **Customer Value Prop**

### **Casual User:**
> "I don't want to think about databases. I just want to dump everything and have AI figure it out. I ask questions, it answers. Done."

### **Enterprise User:**
> "Same as casual, but I need structure, audit trails, and export. I shouldn't have to hire a DBA or data engineer."

### **Your Value Prop:**
> "We built the database so you don't have to. Just dump data. Agents handle everything. Scale from gigabytes to terabytes without architecture changes."

---

## ✅ **Summary: Single-Tenant Agent VDC Model**

- **Per customer:** Own isolated VDC, own data, own agents
- **Cost effective:** $6-13 per customer compute
- **Scalable:** From 1 to 1000 customers trivially
- **AI-native:** Agents work 24/7 organizing data
- **Simple:** "Dump bucket" UX, no schema design
- **Flexible:** Casual → Pro → Enterprise on same tech
- **Profitable:** 75-83% margins at scale

**This is the SaaS model built for AI agents.**

Would you like me to:
1. Create the background agent service code?
2. Build the single VDC implementation guide?
3. Design the OpenNebula multi-customer provisioning?
4. Create the customer dashboard mockup?
