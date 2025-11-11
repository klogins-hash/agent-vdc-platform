# Sleep-time Compute Integration for Agent VDC Platform

## 📖 **Reference: "Sleep-time Compute: Beyond Inference Scaling at Test-time"**
- Research: Letta AI + UC Berkeley
- Key Finding: 5-10x test-time compute reduction through offline pre-computation
- Multi-query amortization: 2.5x cost savings when multiple queries share context
- URL: https://arxiv.org/abs/2504.13171

---

## 🎯 **Executive Summary**

Sleep-time compute fundamentally changes how your Agent VDC platform should think about query processing:

**Traditional Approach (Test-time Only):**
```
User Query → Request arrives → Full processing → Response (slow, expensive)
```

**Sleep-time Compute Approach:**
```
Document uploaded → Background processing (sleep-time) → Pre-computed context c'
User Query → Retrieve c' → Lightweight final step → Response (fast, cheap)
```

**Your platform naturally enables this** because customers have idle periods between queries. You can pre-compute analysis while "sleeping" and serve results instantly when users query.

---

## 🏗️ **How Sleep-Time Compute Aligns with Your Architecture**

### **Current Architecture (Already Partially Implements This)**

Your background agents are already doing sleep-time compute:

```
Per-Customer VDC:
├─ Sleep-Time Phase (Background Agents - 24/7)
│  ├─ Ingest Agent (every 5 sec)
│  │  ├─ Extracts text from new files
│  │  ├─ Generates embeddings
│  │  └─ Stores in Qdrant (vector DB)
│  │
│  ├─ Relationship Agent (every hour)
│  │  ├─ Analyzes unlinked entities
│  │  ├─ Infers relationships with LLM
│  │  ├─ Builds knowledge graph
│  │  └─ Stores in Neo4j
│  │
│  └─ Learning Agent (every 2 hours)
│     ├─ Analyzes recent queries
│     ├─ Identifies gaps
│     ├─ Re-analyzes data with LLM
│     └─ Optimizes embeddings
│
└─ Test-Time Phase (Query API)
   ├─ Receives user query
   ├─ Uses pre-computed embeddings + graphs
   ├─ Lightweight LLM call for final answer
   └─ Returns instantly (milliseconds)
```

**You're already doing roughly what the paper describes!** The key is optimizing it.

---

## 💻 **Implementation: Sleep-Time Compute Pipeline**

### **Layer 1: Background Processing (Sleep-Time)**

```python
# File: services/agents/sleep_time_agent.py

from typing import Optional
from datetime import datetime
import asyncio

class SleepTimeComputeAgent:
    """
    Processes customer data during idle periods.
    Generates optimized context c' for fast test-time queries.
    """

    def __init__(self, vdc_id: str, vector_db, graph_db, s3_client):
        self.vdc_id = vdc_id
        self.vector_db = vector_db  # Qdrant
        self.graph_db = graph_db    # Neo4j
        self.s3 = s3_client         # MinIO
        self.llm = AsyncLLMClient()  # Claude/GPT-4o (expensive models OK)

    async def process_document(
        self,
        doc_id: str,
        raw_content: str,
        doc_type: str = "general"
    ) -> dict:
        """
        Heavy processing during sleep-time.
        Converts raw document into optimized c'.
        """

        start_time = datetime.now()

        # 1. Extract entities & relationships (expensive LLM call)
        entities, relationships = await self.extract_structured_data(
            raw_content,
            doc_type
        )

        # 2. Generate embeddings for all entities
        embeddings = await self.generate_embeddings(entities)

        # 3. Build knowledge graph relationships
        for rel in relationships:
            await self.store_relationship(rel)

        # 4. Generate summaries for different query types
        summaries = await self.generate_summaries(raw_content, doc_type)

        # 5. Predict likely queries (what will users ask?)
        predicted_queries = await self.predict_queries(
            raw_content,
            doc_type,
            entities
        )

        # 6. Pre-compute answers for predicted queries
        precomputed_answers = await self.precompute_answers(
            raw_content,
            predicted_queries
        )

        # Store optimized context
        optimized_context = {
            'doc_id': doc_id,
            'embeddings': embeddings,
            'entities': entities,
            'relationships': relationships,
            'summaries': summaries,
            'predicted_queries': predicted_queries,
            'precomputed_answers': precomputed_answers,
            'processed_at': datetime.now().isoformat(),
            'processing_time_ms': (datetime.now() - start_time).total_seconds() * 1000
        }

        # Store in persistent storage
        await self.store_context(optimized_context)

        return optimized_context

    async def extract_structured_data(
        self,
        content: str,
        doc_type: str
    ) -> tuple:
        """
        Use expensive LLM (Claude Opus) to extract entities/relationships.
        OK to use expensive model since this runs offline.
        """

        prompt = self._build_extraction_prompt(content, doc_type)

        response = await self.llm.invoke(
            model="claude-opus",  # Expensive, but OK for sleep-time
            prompt=prompt,
            max_tokens=4000,
            temperature=0
        )

        entities = self._parse_entities(response)
        relationships = self._parse_relationships(response)

        return entities, relationships

    async def generate_embeddings(self, entities: list) -> dict:
        """
        Generate embeddings for all entities using Claude API.
        Store in Qdrant for fast retrieval.
        """
        embeddings = {}

        for entity in entities:
            embedding = await self.llm.embed(entity['text'])

            await self.vector_db.upsert(
                vector_id=entity['id'],
                vector=embedding,
                metadata={
                    'type': entity['type'],
                    'vdc_id': self.vdc_id,
                    'source_doc': entity['source'],
                    'added_at': datetime.now().isoformat()
                }
            )

            embeddings[entity['id']] = embedding

        return embeddings

    async def predict_queries(
        self,
        content: str,
        doc_type: str,
        entities: list
    ) -> list:
        """
        Predict what questions users will ask about this document.
        Pre-compute answers for these predictions.

        From paper: "sleep-time compute is most effective when
        query is predictable from context"
        """

        # Use fast LLM for predictions (can be less costly)
        prompt = f"""
Given this {doc_type} document with these key entities: {[e['name'] for e in entities]}

What are the 5-10 most likely questions a user would ask about this document?

Return as JSON array of questions.
"""

        response = await self.llm.invoke(
            model="gpt-4o-mini",  # Cheaper for this step
            prompt=prompt,
            max_tokens=1000
        )

        predicted_queries = self._parse_json_list(response)

        return predicted_queries

    async def precompute_answers(
        self,
        content: str,
        queries: list
    ) -> dict:
        """
        Pre-compute answers to predicted queries.
        Store cache so test-time just retrieves.
        """

        precomputed = {}

        for query in queries:
            # Use Claude Opus (expensive, but sleep-time)
            answer = await self.llm.invoke(
                model="claude-opus",
                prompt=f"Context:\n{content}\n\nQuestion:\n{query}",
                max_tokens=2000
            )

            # Cache the answer
            precomputed[query] = {
                'answer': answer,
                'computed_at': datetime.now().isoformat()
            }

        return precomputed

    async def store_context(self, optimized_context: dict):
        """Store optimized context for test-time retrieval."""
        await self.s3.put(
            bucket=f"vdc-{self.vdc_id}-context",
            key=f"context-{optimized_context['doc_id']}.json",
            data=json.dumps(optimized_context)
        )
```

---

### **Layer 2: Query-Time Retrieval (Test-Time)**

```python
# File: services/apis/query_api.py

class QueryAPI:
    """
    Fast test-time query interface.
    Uses pre-computed context c' from sleep-time.
    """

    async def query(self, customer_id: str, query: str) -> dict:
        """
        Respond to user query using pre-computed context.
        Should be milliseconds-level latency.
        """

        start_time = datetime.now()

        # 1. Search pre-computed embeddings (fast!)
        query_embedding = await self.llm.embed(query)
        similar_entities = await self.vector_db.search(
            vector=query_embedding,
            top_k=10,
            namespace=customer_id
        )

        # 2. Check precomputed answers cache
        exact_match = await self.check_precomputed_cache(
            customer_id,
            query
        )

        if exact_match:
            # Cache hit! Return instantly
            return {
                'answer': exact_match['answer'],
                'type': 'cached',
                'latency_ms': (datetime.now() - start_time).total_seconds() * 1000
            }

        # 3. Build context from similar entities + graph
        context = await self.build_context_from_precomputed(
            similar_entities
        )

        # 4. Use cheap LLM for final answer (gpt-4o-mini or Claude Sonnet)
        final_answer = await self.llm.invoke(
            model="gpt-4o-mini",  # Fast, cheap
            prompt=f"Context:\n{context}\n\nQuestion:\n{query}",
            max_tokens=500
        )

        latency_ms = (datetime.now() - start_time).total_seconds() * 1000

        # Log for learning
        await self.log_query(customer_id, query, final_answer, latency_ms)

        return {
            'answer': final_answer,
            'type': 'computed',
            'latency_ms': latency_ms,
            'sources': [e['metadata']['source_doc'] for e in similar_entities]
        }

    async def check_precomputed_cache(
        self,
        customer_id: str,
        query: str
    ) -> Optional[dict]:
        """Check if answer was precomputed during sleep-time."""

        # Simple string matching first
        try:
            cached = await self.s3.get(
                bucket=f"vdc-{customer_id}-context",
                key=f"precomputed-answers.json"
            )
            answers = json.loads(cached)

            # Fuzzy match on queries
            for precomputed_q, answer_obj in answers.items():
                similarity = self.fuzzy_match(query, precomputed_q)
                if similarity > 0.85:  # High confidence match
                    return answer_obj
        except:
            pass

        return None
```

---

## 📊 **Sleep-Time vs Test-Time Compute Budget**

### **Per-Customer Resource Allocation**

```
CASUAL TIER ($12/mo - 2 vCPU, 4GB VM):

Sleep-Time Budget (Continuous, background):
├─ Ingest Agent: ~100ms per file (5-10 files/hour typical)
├─ Relationship Agent: ~1 hour/day @ high effort
├─ Learning Agent: ~30 min every 2 hours
└─ Total: ~$0.50/day in compute costs

Test-Time Budget (On-demand, fast):
├─ Query API: Lightweight only
├─ Max tokens: 500 per query
├─ Model: gpt-4o-mini only
└─ Total: ~$0.01-0.05 per query

Result:
- If customer makes 10 queries/day: ~$1/day total
- Cost per query: $0.10 (amortized)
- Response time: <500ms
- vs Traditional: 5x cheaper, 10x faster
```

---

## 🎯 **Query Predictability Analysis**

From research paper: **Effectiveness depends on how predictable queries are from context**

### **High Predictability Cases (Where Sleep-Time Compute Shines)**

```
Document Type: Financial Report
Context: Contains revenue, expenses, metrics
User likely asks:
├─ "What was Q3 revenue?" ✓ Highly predictable
├─ "What's our profit margin?" ✓ Highly predictable
├─ "Compare to Q2?" ✓ Highly predictable
└─ "Who is the CEO?" ✗ Not predictable from financials

Sleep-Time Pre-Compute:
├─ Extract all financial metrics
├─ Calculate all common ratios
├─ Generate comparisons
└─ Result: 10+ queries answered from same pre-computation
```

**Benefit: 2.5x cost savings per query (from paper)**

### **Low Predictability Cases (Use Standard Test-Time)**

```
Document Type: Creative Writing
Context: Novel excerpt
User might ask:
├─ "What happens next?" - Unpredictable
├─ "Who is the antagonist?" - Sometimes predictable
├─ "Analyze the symbolism of X" - Very unpredictable

Decision:
- Skip sleep-time pre-computation
- Use standard test-time compute
- Save resources for predictable queries
```

---

## 🔄 **Multi-Query Amortization: The SaaS Advantage**

This is where your platform gets HUGE leverage:

### **Scenario: Financial Analysis Customer**

```
Customer uploads: Q3 2024 Earnings Report

Sleep-Time Compute (Once):
├─ Extract all financial data: $2.00 (Claude Opus)
├─ Build analysis models: $1.50
├─ Pre-compute common reports: $1.50
└─ Total Sleep-Time Cost: $5.00 (one-time)

Test-Time Queries (Multiple):
├─ Query 1: "Revenue breakdown?" → $0.01 (cached + lightweight)
├─ Query 2: "Profit margins by segment?" → $0.01
├─ Query 3: "Compare to last year?" → $0.01
├─ Query 4-10: Similar → $0.01 each
└─ Total Test-Time Cost: $0.10 (for 10 queries)

TOTAL COST FOR USER: $5.10 for 10 queries = $0.51/query
VS TRADITIONAL: $1-2 per query = 60-75% SAVINGS
```

**Paper shows: 2.5x savings at 10 queries per context**

---

## 🚀 **Integration Roadmap**

### **Phase 1: Formalize Sleep-Time Pipeline (Week 1)**

```
[ ] Define document type taxonomy (11 types: financial, legal, medical, etc.)
[ ] Build entity extraction prompt templates per type
[ ] Create query prediction models per document type
[ ] Test LLM-based predictions for accuracy
```

### **Phase 2: Implement Precomputation (Week 2-3)**

```
[ ] Implement SleepTimeComputeAgent class
[ ] Add precomputed context storage to S3
[ ] Build query cache layer
[ ] Implement fuzzy matching for cache hits
```

### **Phase 3: Optimize Query API (Week 3-4)**

```
[ ] Migrate query API to use precomputed context
[ ] Add latency tracking
[ ] Implement cache hit/miss metrics
[ ] A/B test against baseline
```

### **Phase 4: Monitoring & Optimization (Ongoing)**

```
[ ] Track sleep-time vs test-time compute usage per customer
[ ] Monitor query predictability scores
[ ] Adjust pre-computation priorities based on actual queries
[ ] Report cost savings to sales/marketing
```

---

## 📈 **Expected Improvements**

Based on research paper + your architecture:

```
Metric                          Before      After       Improvement
─────────────────────────────────────────────────────────────────
Query Latency                   2-5 sec     200-500ms   10-20x faster
Cost per Query                  $0.50       $0.10       5x cheaper
Test-Time Compute Required      100%        20%         80% reduction
Customer Satisfaction           Baseline    +30%        Faster answers
```

---

## 🎓 **Key Insights from Research**

1. **Pre-computation is most effective when queries are predictable** from context
   - Financial docs: Very predictable → High benefit
   - Creative content: Unpredictable → Lower benefit

2. **Multi-query amortization is your superpower**
   - Platform customers naturally ask multiple related questions
   - Same pre-computation can serve 5-10+ queries
   - 2.5x cost reduction at 10 queries per context

3. **"Sleep-time compute" is representation learning**
   - Converting raw context into optimized c'
   - Like caching, but with LLM reasoning
   - Trades offline cost for online speed

4. **Works best in stateful scenarios**
   - Document Q&A: Perfect ✓
   - Conversational AI: Perfect ✓
   - Code repositories: Perfect ✓
   - Your VDC model: Perfect ✓

---

## ✅ **Architecture Alignment Checklist**

- [x] Background agents already doing sleep-time work
- [x] Per-customer isolation enables multi-query amortization
- [x] Qdrant + Neo4j are perfect for caching pre-computed results
- [x] E2B can run heavy sleep-time processing
- [x] Query API naturally becomes test-time interface
- [x] Margins improve: 5x cheaper to serve queries
- [x] This directly addresses cost-per-query problem

---

## 🎯 **Recommendation**

**Integrate sleep-time compute principles as core optimization:**

1. Formalize document type taxonomy → predict query patterns
2. Implement pre-computation pipeline → background agents
3. Build intelligent caching → query API optimization
4. Monitor & report savings → customer value proposition

**This transforms your platform from "infrastructure" to "AI data intelligence" with superior economics.**

---

**Next: Create `SLEEP_TIME_COMPUTE_IMPLEMENTATION_GUIDE.md` with exact code examples for background agents?**
