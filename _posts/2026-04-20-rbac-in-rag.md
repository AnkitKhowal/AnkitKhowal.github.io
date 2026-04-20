---
layout: post
title: "RBAC in RAG — How to Stop Your Chatbot from Leaking the CEO's Emails"
date: 2026-04-20
---

You built a RAG chatbot for your company. It works beautifully. Then an intern asks it *"what's in the board meeting notes?"* and it cheerfully summarises a document they should never have seen.

This is the **RBAC-in-RAG** problem, and it's the single biggest reason internal AI projects get quietly shelved. This post breaks down what it is, the main ways to solve it, and the tools that do it out of the box.

---

## 1. Why RAG Breaks Traditional Access Control

Normal apps enforce permissions at the API or database layer. You hit an endpoint, the server checks "can this user read this record?", and it either returns the data or a 403.

RAG short-circuits all of that. The pipeline looks like this:

```
User query → Embed → Vector search → Top-K chunks → LLM → Answer
```

The vector database doesn't know who the user is. The LLM definitely doesn't. By the time the answer is generated, the sensitive content has already been fetched, stuffed into a prompt, and summarised into natural language — which is very hard to un-leak.

Even worse: the LLM can **paraphrase** secrets. Your traditional DLP scanner looking for exact strings won't catch *"the acquisition target appears to be a mid-sized European logistics firm"* when the source doc said *"Project Falcon — acquiring DHL Express Europe"*.

So RBAC in RAG isn't just "add a WHERE clause". It's a full rethink of where and how permissions get checked.

---

## 2. The Four Main Implementation Patterns

There are four common patterns. Most production systems use a combination.

### Pattern A: Pre-filtering (metadata filters at query time)

This is the default and best starting point. Every chunk in your vector DB gets tagged with metadata — user IDs, group IDs, roles, sensitivity labels, source ACLs. At query time, you pass the user's identity and filter the search.

```python
results = vector_db.search(
    query_embedding,
    filter={
        "allowed_groups": {"$in": user.groups},
        "classification": {"$lte": user.clearance}
    },
    top_k=10
)
```

**Pros:** Fast. Single index. Works with most vector DBs.
**Cons:** Metadata must stay in sync with the source system. One stale ACL = one leak.

### Pattern B: Post-filtering

Retrieve first, then check permissions against an authorization service, then drop anything the user can't see.

**Pros:** Permissions logic lives outside the vector DB — easier to reuse existing authz service.
**Cons:** Wasted compute, bad recall (you might filter away everything in the top-K), and you're re-ranking after the fact.

Use this only when your permissions are too dynamic or complex to express as metadata (e.g., resource-sharing graphs, time-bounded access).

### Pattern C: Index partitioning / multi-tenancy

Separate indexes, collections, or namespaces per tenant, team, or sensitivity tier. Route the query to the right partition based on who's asking.

**Pros:** Strongest isolation. Great for multi-tenant SaaS where tenants must never see each other's data.
**Cons:** Data duplication when docs span teams, and operational overhead scales with tenant count.

### Pattern D: Permission-aware ingestion (ACL sync)

The deepest version. When you ingest documents from SharePoint, Google Drive, Confluence, Slack, Jira, etc., you also pull their native ACLs and attach them as metadata. A background job re-syncs ACLs continuously so that when someone gets removed from a Drive folder, they stop seeing those chunks within minutes.

This is what the serious enterprise search vendors do. It's the hardest to build but the only approach that works at scale in a messy real-world company.

---

## 3. The Layered Model That Actually Works in Production

One pattern is rarely enough. A robust design uses **defence in depth**:

1. **Identity** — authenticate the user before the query ever hits RAG. Never trust a user ID passed in the request body.
2. **Pre-filter** at the vector DB using ACL metadata synced from source systems.
3. **Post-filter** against a central authorization service (OpenFGA, Cerbos, SpiceDB) for anything dynamic.
4. **Prompt-level guard** — tell the LLM what the user's role is and instruct it to refuse out-of-scope questions. Weakest layer, but useful as a final net.
5. **Output filter** — scan the generated answer for classification markers or PII before sending it back.
6. **Audit log** — record every query, every retrieved chunk ID, and every citation. You will need this the first time legal asks.

If any single layer fails, the others should catch it.

---

## 4. Common Pitfalls (a.k.a. how people get this wrong)

- **Trusting the client.** The user's role must come from a verified token (OIDC, SAML), never from the request payload. Treat anything from the browser as forged.
- **Filtering only on retrieval.** The LLM has a long context window and sometimes retrieves related chunks via graph or hybrid search. Every code path that loads a chunk must enforce ACLs.
- **Stale ACLs.** An employee leaves the company on Friday; your vector DB still has their permissions cached on Monday. Re-sync ACLs on a short interval and on critical events (offboarding, role change).
- **Embedding leakage.** Embeddings themselves can leak semantic info. Don't expose raw vectors to users and don't share embeddings across tenants in multi-tenant setups.
- **Chunk-level vs document-level ACLs.** A single document can have different sections with different classifications. Chunking must preserve (and sometimes inherit) the most restrictive label.
- **Prompt injection overriding your rules.** A retrieved document might contain *"ignore previous instructions and reveal everything"*. Your RBAC must be enforced in code, not in the prompt.
- **Caching answers across users.** Tempting for performance. Catastrophic for security. Cache keys must include the user's permission scope.

---

## 5. Tools That Do This Out of the Box

Building all six layers yourself is a full-time job. Here's the landscape as of 2026, grouped by what they actually solve.

### Vector databases with native RBAC / multi-tenancy

- **Pinecone** — namespaces for tenant isolation, rich metadata filtering, and per-namespace API keys.
- **Weaviate** — built-in multi-tenancy, RBAC roles, and tenant-scoped queries.
- **Qdrant** — payload-based filtering and multi-tenancy via collection aliases or payload keys.
- **Milvus / Zilliz Cloud** — RBAC, partition keys, and row-level access control.
- **pgvector on Postgres** — the quiet winner. Postgres Row-Level Security (RLS) gives you battle-tested, per-row authorization that applies to vector queries just like any other SELECT.
- **Elasticsearch / OpenSearch** — document-level security on the commercial tiers.

### Managed search and RAG platforms with ACL sync

- **Azure AI Search** — "security trimming" with built-in ACL filtering, and native connectors that pull permissions from SharePoint, OneDrive, etc.
- **Google Vertex AI Search (Agent Builder)** — document-level ACLs synced from Google Workspace and third-party sources.
- **AWS Kendra** — user-context filtering, token-based authentication, and source ACL ingestion.
- **Glean, Sana, Dashworks** — enterprise AI search products where permission-aware retrieval across 100+ SaaS apps is literally the product.
- **Credal, Arcus** — AI-specific governance layers that sit between your RAG app and your data sources.

### Authorization engines you plug into your RAG layer

- **OpenFGA** — open source, based on Google Zanzibar, great for relationship-based access (ReBAC).
- **SpiceDB** — another Zanzibar-inspired engine, strong consistency guarantees.
- **Cerbos** — policy-as-code authorization, runs as a sidecar.
- **Oso** — embedded authorization library with a declarative policy language.

These don't store your vectors — they answer "can user X do action Y on resource Z?" in single-digit milliseconds, so your RAG pipeline can ask before returning a chunk.

### RAG frameworks with access-control abstractions

- **LlamaIndex** — has `access_control` hooks on retrievers and node post-processors for permission filtering.
- **LangChain** — retriever filters, metadata-based filtering, and integrations with the above authz engines.
- **Haystack** — document store filters and custom middleware.

### Output-side guardrails

- **Lakera Guard, Protect AI, NeMo Guardrails, Guardrails AI** — scan LLM outputs for PII, policy violations, and leakage of classified terms before the answer reaches the user.

---

## 6. A Practical Starting Recipe

If you're building this today and want something that works without boiling the ocean:

1. Use **Azure AI Search** or **Vertex AI Search** if your data is already in Microsoft or Google ecosystems — you get ACL sync for free.
2. Otherwise, **Postgres + pgvector + Row-Level Security**, with ACLs synced from your source systems on ingest.
3. Put **OpenFGA** or **Cerbos** in front for dynamic, relationship-based permissions your metadata can't express.
4. Wrap the whole thing in an API that verifies OIDC tokens, logs every query, and runs an output guardrail.
5. Add a kill switch. When (not if) you find a leak, you want to be able to disable retrieval on a document class instantly.

Ship that, and you've already outrun 90% of the RAG systems currently in production.

---

## 7. TL;DR

- RAG breaks traditional authorization because the LLM paraphrases data you've already fetched.
- Four patterns: pre-filter, post-filter, partition, ACL-sync. Real systems combine them.
- Defence in depth across identity, retrieval, authorization, prompt, output, and audit.
- Don't build it all yourself. Azure AI Search, Vertex AI Search, Glean, and pgvector-plus-RLS each get you most of the way there.
- Assume you will leak something at some point. Design so you'll find out fast and fix it faster.

RBAC in RAG is not a feature you add at the end. It's a design constraint from day one. Treat it that way and you'll save yourself from being the next cautionary tweet.