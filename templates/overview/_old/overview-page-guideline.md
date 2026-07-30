# Overview page guidelines

This template follows a standardized structure for Inji product overview pages.
Replace all placeholders (enclosed in curly braces {}) with actual content.

This structure is formalized machine-readably in `overview-schema.yaml` — that file is the
authority for required/optional fields and validation; this document is its plain-language
mirror for anyone hand-authoring a page instead of running the agent pipeline.

REQUIRED SECTIONS:
- Overview
- Architecture
- Deployment
- Documentation
- Contribution & Community

OPTIONAL SECTIONS (include if applicable):
- Standards, Specifications, and Compliance (flat feature-coverage table, or split into named
  categories — e.g. Credential Data Models, Verification Protocols, Credential Formats — when
  compliance spans several distinct groupings)
- Try It Out (sandbox/collab environment pointer)
- Plugin Support (for products with a plugin-based architecture the product itself loads)
- SDK Integration (for products that ship embeddable SDK/library components for OTHER
  applications to consume — distinct from Plugin Support)
- Configurations (properties-file blocks, or freeform tables/lists for non-properties config)
- Databases (for products with database dependencies)
- Upgrades (when migration guides exist)
- Upcoming Features (roadmap items not yet shipped)

PLACEHOLDERS LEGEND:
- {PRODUCT_NAME}: e.g., "Inji Mobile Wallet", "Inji Web Wallet", "Inji Verify"
- {TARGET_USERS}: e.g., "end users", "verifiers", "holders"
- {CORE_FUNCTIONALITY}: Brief description of what the product does
- Feature-coverage status: ✅ Available, ❌ Unsupported, 🟡 Partial, 🕓 Coming Soon

STYLE GUIDELINES:
- Keep descriptions concise and action-oriented
- Link to detailed documentation rather than duplicating content
- Use tables for feature comparisons
- Include code snippets for configuration examples
