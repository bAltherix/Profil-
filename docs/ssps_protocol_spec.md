# SSPS Protocol Specification (v1.0)
### Session State Persistence System Protocol  
### Used by PaxCore to Serialize BECIA v4 State
---

## Abstract

The Session State Persistence System (SSPS) is a deterministic, portable
snapshot protocol used to serialize and reconstruct the internal state of
BECIA‑based agents.

SSPS defines:

- a canonical snapshot structure  
- serialization rules  
- validation rules  
- reconstruction rules  
- versioning semantics  
- compatibility guarantees  

SSPS is model‑agnostic, runtime‑agnostic, and storage‑agnostic.  
It is implemented by PaxCore and consumed by BECIA v4.

---

## 1. Purpose & Scope

SSPS provides a unified, minimal, deterministic representation of:

- identity state  
- relational state (L3.5)  
- cognitive arc (L4)  
- adaptive modulation (L4.1)  
- emotional baseline (L0)  
- safety metadata  
- session metadata  

SSPS does **not** store:

- raw conversation logs  
- verbatim messages  
- transcripts  
- embeddings  
- tool traces  

It stores only the minimal structured state vector required to reconstruct
the agent’s identity and behavioural continuity.

---

## 2. Design Principles

- **Deterministic**  
  Same snapshot → same reconstructed state.
- **Minimal**  
  300–800 tokens typical size.
- **Portable**  
  Works across runtimes, models, and deployments.
- **Model‑agnostic**  
  SSPS does not depend on any specific LLM.
- **Privacy‑aligned**  
  No raw logs, no PII beyond user_id.
- **Schema‑versioned**  
  Supports multiple profiles (BECIA v4, generic, custom).
- **Immutable**  
  Snapshots are never modified in place.

---

## 3. Snapshot Model

An SSPS snapshot is a structured JSON document containing:

- metadata (schema version, timestamps, confidence)  
- session metadata  
- emotional baseline (L0)  
- relational state (L3.5)  
- cognitive arc (L4)  
- adaptive modulation (L4.1)  

The schema is intentionally minimal and does not include raw conversational
content or model‑specific data.

The default profile for BECIA v4 is:
```
schema_version: ssps-becia-v4.0
```

Profiles define which sections are required and how they map to runtime state.

---

## 4. Validation Rules

A snapshot is valid if:

1. `schema_version` is supported  
2. `user_id` is present  
3. `expires_at >= saved_at`  
4. `confidence ∈ [0.0, 1.0]`  
5. all required sections are present  
6. JSON is structurally valid  
7. no raw logs or transcripts are included  

Validation is performed by PaxCore before storage.

---

## 5. Reconstruction Rules

Given a valid snapshot:

- L0 baseline is restored  
- L3.5 relational state is restored  
- L4 cognitive arc is restored  
- L4.1 adaptive modulation is restored  
- L5 safety metadata is restored (if present)  

Reconstruction is deterministic:
```
snapshot → core_state → BECIA runtime
```

SSPS defines the structure; BECIA defines how the structure is interpreted.

---

## 6. Schema Versioning & Migration

SSPS uses semantic versioning.

### Versioning Rules

- **Backward compatibility**  
  Newer runtimes must read older snapshots of the same MAJOR version.

- **Forward compatibility (best effort)**  
  Unknown fields may be safely ignored by older runtimes.

- **Non-destructive migration**  
  Migration occurs at load time; stored snapshots remain immutable.

- **Profile isolation**  
  Each schema profile is stored independently.

These rules ensure stable evolution of the protocol without breaking existing
deployments.

---

## 7. Determinism Boundary

SSPS guarantees deterministic **state reconstruction**,  
not deterministic LLM output.

The underlying model remains probabilistic;  
SSPS ensures that the *conditions* under which it operates are stable,
reproducible, and auditable.

---

## 8. Security & Privacy

SSPS is designed for privacy‑aligned deployments.

- no raw logs  
- no verbatim content  
- minimal PII (user_id only)  
- snapshot size intentionally small  
- compatible with GDPR‑aligned retention policies  

Deployments may strengthen security through:

- encryption‑at‑rest (backend‑provided)  
- encryption‑in‑transit (TLS)  
- access control at the storage layer  
- key management via platform‑native systems (KMS, Vault)  

---

## 9. Operational Considerations

Although SSPS does not define storage behaviour, deployments may expose
operational metrics such as:

- snapshot writes  
- snapshot loads  
- expired snapshots  
- schema version distribution  

These metrics support monitoring and observability without storing or exposing
user content.

---

## 10. Summary

SSPS provides a portable, deterministic, minimal protocol for representing
agent state across sessions and runtimes.  
It is the serialization layer of the BECIA → SSPS → PaxCore continuity pipeline.

---

© 2026 b.AItherix — All Rights Reserved



