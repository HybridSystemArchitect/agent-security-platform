# Agent Security Platform — Core Security Kernel v10

> Enterprise-grade cryptographic security kernel for AI agent governance.  
> **No code. No trust. No exceptions.**

---

## What problem does this solve?

In 2026, enterprises deploy AI agents that make decisions, call external systems, and operate autonomously. The question every CISO is asking:

> *"What did this agent do, on whose behalf, and can you prove it?"*

Traditional security tools weren't built for this. WAFs don't understand agent reasoning chains. DLP doesn't inspect LLM context windows. CASB doesn't attribute tool calls to agent identity.

**Agent Security Platform answers all three questions — cryptographically.**

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  CoreSecurityKernelV10                      │
│                  (Single Entry Point)                       │
├──────────────┬──────────────┬──────────────┬───────────────┤
│    Agent     │   Capability │     Goal     │   Unit of     │
│   Identity   │   Firewall   │  Integrity   │     Work      │
│  (Ed25519)   │ (Zero-Trust) │   Engine     │   Engine      │
│              │              │  (Cosine     │  (Atomic      │
│              │              │   Drift)     │   Commit)     │
├──────────────┴──────────────┴──────────────┴───────────────┤
│                    Audit Ledger                             │
│         (SHA-256 Merkle Chain · SQLite WAL · ACID)         │
├─────────────────────────────────────────────────────────────┤
│                  Provenance Graph                           │
│         (Ed25519-signed DAG · Cycle Detection)             │
├─────────────────────────────────────────────────────────────┤
│               EU AI Act Reporter                           │
│         (Article 12 · Article 15 · JSON Export)           │
└─────────────────────────────────────────────────────────────┘
```

---

## Security Properties

| Property | Implementation |
|---|---|
| **Tamper-evident audit trail** | SHA-256 Merkle chain — any past modification breaks verification |
| **Per-agent cryptographic identity** | Ed25519 key pair per agent — first-class security principal |
| **Zero-trust authorization** | Explicit capability whitelist — anything not listed is denied |
| **Salami-slice detection** | Time-windowed operation counting — incremental abuse intercepted |
| **Mission drift detection** | Cosine-distance fingerprint — goal divergence triggers intercept |
| **Fail-closed design** | Any unresolvable security state raises `KernelPanicException` |
| **Atomic commits** | All-or-nothing — no partial state corruption possible |
| **Persistent audit log** | SQLite WAL mode — survives process restart, crash-safe |

---

## OWASP LLM Top 10 Coverage (2025)

| OWASP ID | Threat | Mitigation |
|---|---|---|
| **LLM01:2025** | Prompt Injection | Provenance lineage validation floor |
| **LLM06:2025** | Excessive Agency | Capability Firewall + risk tier ceiling |
| **LLM08:2025** | Excessive Permissions | AgentScope explicit whitelist |
| **LLM09:2025** | Misinformation | Goal Integrity cosine drift detection |
| **LLM10:2025** | Unbounded Consumption | Salami-slice time-window detection |

---

## EU AI Act Compliance

| Article | Requirement | Implementation |
|---|---|---|
| **Article 12** | Automatic record-keeping of all AI decisions | Full structured JSON audit log, every operation |
| **Article 15** | Cryptographic audit trail for high-risk AI | SHA-256 Merkle chain, Ed25519 signatures |

### Example: Article 12 Report Output

```json
{
  "report_type": "EU_AI_ACT_ARTICLE_12",
  "version": "10.0.0",
  "generated_at": "2026-06-21T14:32:01.000000Z",
  "integrity_verified": true,
  "integrity_violations": [],
  "total_records": 1847,
  "chain_root_hash": "a3f8c2d1e4b7...",
  "records": [
    {
      "global_sequence": 1847,
      "stream_id": "agent:f3a2b1c4/order:99821",
      "actor_id": "agent:f3a2b1c4d5e6f7a8",
      "audit_event_type": "ENTITY_UPDATED",
      "event_hash": "7d4e2f1a3b8c...",
      "ledger_hash": "9c1a4e7f2b3d...",
      "timestamp": "2026-06-21T14:31:58.441Z"
    }
  ]
}
```

---

## Test Results

```
========================= 33 passed in 0.84s ==========================

PASSED tests/test_kernel.py::test_canonical_serializer_determinism
PASSED tests/test_kernel.py::test_canonical_serializer_order_independence
PASSED tests/test_kernel.py::test_canonical_serializer_enum
PASSED tests/test_kernel.py::test_canonical_serializer_datetime
PASSED tests/test_kernel.py::test_merkle_link_chaining
PASSED tests/test_kernel.py::test_agent_identity_creation
PASSED tests/test_kernel.py::test_agent_signature_verify
PASSED tests/test_kernel.py::test_agent_signature_reject_wrong_message
PASSED tests/test_kernel.py::test_agent_scope_expiry
PASSED tests/test_kernel.py::test_entity_immutability
PASSED tests/test_kernel.py::test_entity_versioning
PASSED tests/test_kernel.py::test_event_hash_determinism
PASSED tests/test_kernel.py::test_ledger_genesis_entry
PASSED tests/test_kernel.py::test_ledger_commit_and_retrieve
PASSED tests/test_kernel.py::test_ledger_chain_integrity_clean
PASSED tests/test_kernel.py::test_ledger_tampering_detection
PASSED tests/test_kernel.py::test_actor_trail
PASSED tests/test_kernel.py::test_provenance_dag_cycle_detection
PASSED tests/test_kernel.py::test_provenance_lineage_floor
PASSED tests/test_kernel.py::test_provenance_signed_edge_valid
PASSED tests/test_kernel.py::test_provenance_signed_edge_invalid
PASSED tests/test_kernel.py::test_firewall_allow_valid_capability
PASSED tests/test_kernel.py::test_firewall_deny_missing_capability
PASSED tests/test_kernel.py::test_firewall_deny_risk_tier_exceeded
PASSED tests/test_kernel.py::test_firewall_salami_detection
PASSED tests/test_kernel.py::test_goal_integrity_nominal
PASSED tests/test_kernel.py::test_goal_integrity_drift_critical
PASSED tests/test_kernel.py::test_goal_integrity_cumulative_drift
PASSED tests/test_kernel.py::test_kernel_full_operation_cycle
PASSED tests/test_kernel.py::test_kernel_status_structure
PASSED tests/test_kernel.py::test_eu_article_12_report_structure
PASSED tests/test_kernel.py::test_eu_article_15_report_structure
PASSED tests/test_kernel.py::test_eu_report_json_serializable
```

---

## REST API — 12 Endpoints

```
POST   /agents                     Register agent identity (Ed25519 key pair)
GET    /agents                     List all registered agents
GET    /agents/{agent_id}          Agent details + scope

POST   /entities                   Create entity (committed to audit ledger)
GET    /entities/{entity_id}       Full version history

POST   /operations                 Execute security-gated operation
                                   (firewall + provenance + drift + atomic commit)

GET    /ledger                     Paginated audit ledger access
GET    /ledger/stats               Total entries, last hash
GET    /ledger/verify              Full Merkle chain integrity verification
GET    /ledger/actor/{actor_id}    Complete actor audit trail

GET    /compliance/article-12      EU AI Act Article 12 report
GET    /compliance/article-15      EU AI Act Article 15 report
```

---

## Example: Tamper Detection

```python
# Commit a legitimate operation
receipt = kernel.execute_operation(...)
print(receipt.ledger_hash)
# → "a3f8c2d1e4b7f9a2c5d8e1f4b7a2c5d8..."

# Attacker modifies a past record directly in the database
db.execute("UPDATE ledger SET event_hash='deadbeef' WHERE global_sequence=47")

# Integrity check catches it immediately
result = kernel.verify_integrity()
print(result["verified"])        # → False
print(result["violations"])      # → [{"global_sequence": 47, "type": "HASH_MISMATCH"}]
```

---

## Example: Capability Firewall

```python
# Agent scope: only database reads allowed
scope = AgentScope(
    allowed_capabilities=frozenset({"db:read"}),
    max_risk_tier="read"
)

# Attempt to execute a delete operation → blocked
kernel.execute_operation(
    capability="db:delete",
    risk_tier="delete",
    ...
)
# → CapabilityDeniedException:
#   "Agent 'agent:f3a2b1c4' does not have capability 'db:delete' in its scope."
```

---

## Example: Mission Drift Detection

```python
# Mission: data processing (cosine fingerprint)
mission = MissionFingerprint(
    core_intent_vector=(0.6, 0.8, 0.0),
    max_allowed_drift=0.25
)

# Agent starts pursuing unrelated objectives
result = kernel.goal_engine.evaluate(
    observed_vectors=[(-0.6, -0.8, 0.0)]  # opposing direction
)
print(result["status"])     # → "MISSION_DRIFT_CRITICAL"
print(result["max_drift"])  # → 0.9999
# → Operation intercepted. KernelPanicException raised.
```

---

## Technical Stack

- **Language:** Python 3.10+
- **Cryptography:** Ed25519 (via `cryptography` library)
- **Persistence:** SQLite WAL mode (zero external dependencies)
- **API Layer:** FastAPI + Uvicorn
- **Tests:** pytest (33 tests, 100% pass rate)
- **External dependencies:** `cryptography`, `fastapi`, `uvicorn` — nothing else

---

## Licensing & Acquisition

This is a **commercial source code license**.

- Non-exclusive perpetual license for internal enterprise use
- Full source code delivery via Escrow
- Integration support available

**For licensing inquiries:**  
GitHub: [@HybridSystemArchitect](https://github.com/HybridSystemArchitect)

---

## Part of the Agent Security Platform Suite

| Module | Description | Status |
|---|---|---|
| **Core Security Kernel v10** | Cryptographic agent governance | ✅ Available |
| **PromptShield v3.0** | Multi-model prompt injection detection ensemble | ✅ Available |
| **AlignmentLens Suite** | 12-module AI alignment diagnostics | ✅ Available |
| **HyperbolicGraphRAG** | Poincaré-ball knowledge graph security | ✅ Available |
| **Memory Security Platform** | Agent memory integrity + audit ledger | ✅ Available |
| **AEGIS Enterprise** | Full unified platform | ✅ Available |

---

*Built by HybridSystemArchitect · June 2026*
