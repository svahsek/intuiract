# Framework Concepts — Q&A Reference

Answers to foundational questions about the patterns used in the Release Notes Automation Framework.

---

## 1. What is Registry/Manifest Architecture — and is it a general software engineering practice?

### Core Definition

A registry/manifest architecture is a two-layer indirection pattern:

- **Registry** — "what exists?" — a central catalog of things you can discover
- **Manifest** — "what's in this thing and where are its parts?" — a pointer file, not the content itself

### It Is a General, Well-Established Pattern

| Where | Registry | Manifest |
|---|---|---|
| Package managers | npm, PyPI, Cargo | `package.json`, `Cargo.toml` |
| Containers | Docker Hub / OCI registries | `Dockerfile`, image manifest |
| Kubernetes | Helm chart repo | `Chart.yaml`, YAML manifests |
| Mobile apps | App Store metadata | `AndroidManifest.xml`, `Info.plist` |
| Browsers | Extension store | `manifest.json` |
| Microservices | Consul / Eureka service registry | Service descriptor |
| Java JARs | Maven Central | `MANIFEST.MF` |

### How It Is Used in This Framework

The same pattern is applied, but the **consumers are AI agents**, not humans or build tools:

```
standards/content-types.yaml              ← Registry: "what content types exist?"
  ↓
standards/content-types/release-notes.yaml  ← Manifest: "where are the schema, rules, skill?"
  ↓
templates/, rules/, skills/                  ← Actual assets
```

The key agentic motivation: an agent should **never hardcode a file path**. Instead it asks the registry, resolves the manifest, and finds everything dynamically — so if files move, only the manifest changes and all agents stay intact.

### Verdict

This is a proven general software engineering pattern applied with a specific agentic purpose: **runtime discovery by agents** rather than by build tools or humans. The pattern is proven; the application to agent orchestration is the newer, more niche part.

---

## 2. What is Reconciliation — and is it a common practice?

### Core Definition

Reconciliation is the process of taking the **same piece of information from multiple sources**, comparing them, resolving conflicts, and producing **one authoritative version**.

The word comes from accounting — you reconcile your bank statement against your own records to find where they agree and where they differ, then settle on the truth.

### It Is a Common Practice — Everywhere

| Domain | What Gets Reconciled |
|---|---|
| Banking / Finance | Your records vs. bank ledger → single balance |
| Databases | Replicated nodes that diverged → consistent state |
| Git merge | Two branches with conflicting changes → one codebase |
| ETL / Data pipelines | Same field arriving from multiple data sources → one canonical value |
| Healthcare records | Patient data from multiple hospitals → unified record |
| Kubernetes | Desired state (YAML) vs. actual cluster state → controller reconciles them |
| E-commerce inventory | Warehouse system vs. order system vs. returns → one stock count |

### What It Means in This Framework Specifically

In the release-notes skill, the same release item can appear across **four different sources**, each telling a slightly different story:

```
GitHub Issue        → says: "this is a bug fix, milestone v2.1"
PR label            → says: "this is a feature"
PR title prefix     → says: "feat: ..."
Commit message      → says: "fix: ..."
```

Reconciliation answers: **which one do I trust?**

`standards/source-precedence.yaml` defines the authority chain — e.g., for `change_type`:

```
GitHub issue label → PR label → PR title prefix → commit prefix
```

The reconciler picks the highest-trust source that has a value, discards the rest, records the provenance (where the value came from), and outputs one clean `release-evidence.yaml`.

### Key Idea

Reconciliation = **conflict resolution with explicit rules**, not guesswork. Without it, you'd either pick arbitrarily or show all four conflicting values and ask a human to decide — which defeats the purpose of automation.

In this framework, it is the layer that makes output **deterministic and auditable** rather than prompt-dependent.

---

## 3. What is the Schema-Based Manifest Built Here — and Is It Modern Practice?

### What Has Been Built — Three Interlocking Layers

**Layer 1 — Registry (`standards/content-types.yaml`)**

A single YAML file that catalogs what content types exist. Currently one entry: `release-notes`. Consumers read this first to know what is available.

**Layer 2 — Manifest (`standards/content-types/release-notes.yaml`)**

A pointer file for the `release-notes` content type. It does not contain actual rules — it contains **paths to where everything lives**: schema, rendering rules, skill, examples, validation rules. Consumers fetch this and resolve everything else from it.

**Layer 3 — Schema (`templates/release-notes/release-notes-schema.yaml`)**

A formal JSON Schema (draft-07) definition. It defines exactly what fields are required, what values are valid (e.g., `releaseType` must be one of `Major / Minor / Patch / Hotfix / LTS`), and what patterns strings must match (semver regex, ISO date format). Agents validate against this before rendering.

### Is This Modern Practice? Very Much So.

These patterns are standard in modern platform and infrastructure engineering:

| Pattern in This Framework | Industry Equivalent |
|---|---|
| Central registry YAML | Helm chart index, Backstage software catalog, OCI index manifest |
| Manifest with entrypoints | Kubernetes CRD, OpenAPI `$ref` resolution, Helm `Chart.yaml` |
| JSON Schema for validation | OpenAPI specs, JSON Schema in CI pipelines, AsyncAPI |
| Validate before render/deploy | Kubernetes admission webhooks, Terraform plan before apply |
| Consumer fetches manifest, not files | CDN-served manifests, package registries, Terraform module registries |

Notably, **Backstage** (built by Spotify, now a CNCF project) does almost exactly this — a `catalog-info.yaml` per service that points to all its entrypoints, and a central catalog that discovers them. This framework applies the same concept to documentation content types.

### Should You Carry On With This Practice? Yes.

**1. Agents need stable contracts, not stable paths.**
If a file path is hardcoded into an agent prompt and the file moves, every agent breaks. With a manifest, you move the file, update one pointer, and all agents stay intact.

**2. Multiple consumers can adopt without copying standards.**
A consumer repo fetches the manifest and gets the current schema. No copy-paste drift. This is how package registries work — consumers do not vendor npm's registry, they query it.

**3. Schema-first = automation-safe.**
Without a schema, agents generate whatever looks right. With a schema, there is a machine-checkable contract. This is the difference between "we generate docs" and "we guarantee docs are correct before publishing."

**4. It scales to more content types naturally.**
When `feature-pages` or `api-docs` are added as content types, you add a registry entry and a new manifest. The pattern does not change.

### One Honest Caveat

The overhead of maintaining registry + manifest + schema is only worth it if you have **multiple consumers or multiple content types**. For a single internal tool with one content type it can feel like overengineering. But this is explicitly built as a **reusable standards repository that other repos fetch from** — so the architecture matches the ambition exactly.

---

*Generated from Q&A session — 2026-06-14*
