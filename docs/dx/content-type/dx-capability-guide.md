Is there a particular format for software product feature page which we release for developer community for adoption (I mean, say if the product is for developer adoption and is open source)
Has anyone identified this as a content type, like the content type is there for 'Release Notes',  End User Guide, Deployment Guide etc.

Like, say, My product supports SD JWT for Verifiable Credentials, How should I put this as a separate page under my docs?



# Refined

This summary is designed to be pasted directly into your internal **Documentation Style Guide** or a `CONTRIBUTING.md` file. It codifies the "Capability Guide" as a high-standard, adoption-driven content type.

---

# Content Type Guideline: Capability Guide

## 1. Definition & Purpose
A **Capability Guide** is a technical, outcome-oriented document designed to drive developer adoption of a specific product feature or standard. Unlike a generic feature description, it focuses on the **empowerment** of the developer—explaining not just what the feature *is*, but what the system is now *capable* of achieving and how to implement it.

**Primary Goal:** To move a developer from discovery to a successful "Test it Today" state.

---

## 2. Standardized Structure (Strict Schema)
To ensure consistency across the portal, every Capability Guide must include the following six sections in order:

1.  **Overview:** A concise technical definition of the feature and its alignment with industry standards (e.g., W3C, IETF).
2.  **Why This Matters (Value Prop):** A bulleted list of tangible benefits (e.g., security, interoperability, user experience).
3.  **Technical Mechanism:** A deep dive into the "under-the-hood" logic. Must mention specific libraries, APIs, or protocols involved.
4.  **Workflow / User Flow:** A step-by-step chronological sequence of how the data or user moves through the feature.
5.  **Constraints & Limitations:** A transparent list of technical boundaries, offline limitations, or pending support items to reduce support debt.
6.  **Test the Feature (Implementation):** A hands-on guide featuring configuration keys, code snippets, and "Negative Testing" steps.

---

## 3. Metadata Approach (Frontmatter)
All Capability Guides must include the following metadata for programmatic navigation and persona-based filtering:

```yaml
---
title: [Human Readable Title]
content_type: capability_guide
status: [experimental | beta | stable]
standard_alignment: [e.g., SD-JWT, W3C VC 2.0]
primary_library: [The specific SDK or Lib used]
dev_persona: [e.g., Backend, Security, Mobile]
last_reviewed: YYYY-MM-DD
---
```

---

## 4. Docs-as-Code Governance
* **Tracking:** Use GitHub Issues to request a Capability Guide, linking it to the corresponding software feature issue.
* **Validation:** GitHub Actions should lint the Markdown files to ensure all required metadata fields and the six core headings are present before a PR can be merged.
* **Ownership:** Subject Matter Experts (SMEs) provide the technical logic; Technical Writers refine the "Value Prop" and "User Flow" to ensure DX clarity.

---

## 5. Comparison: Why this isn't "Release Notes"
| Feature | Release Notes | Capability Guide |
| :--- | :--- | :--- |
| **Audience** | Existing users tracking changes. | New/Existing users looking to adopt a feature. |
| **Tone** | Historical / Chronological. | Instructional / Persuasive. |
| **Longevity** | Point-in-time. | Evergreen (updated as the capability evolves). |
| **Depth** | Surface level (what changed). | Architectural (how it works). |

---

### Pro-Tip for Approvers
When reviewing a **Capability Guide**, ask: *"If I were a developer with 10 minutes, could I understand the value, see the flow, and know exactly which config key to flip?"* If the answer is no, the document requires further refinement.




# Crude



Yes, this is absolutely a recognized content type. In the world of Developer Experience (DX), this is often referred to as a **Feature Spotlight** or a **Capability Guide**. 

Unlike a "Reference" page (which is dry and technical) or "Release Notes" (which are chronological), a Feature Page is **outcome-oriented**. Its job is to move a developer from "What is this?" to "I need to implement this right now."

For a specialized feature like **SD-JWT for Verifiable Credentials**, you should follow a format that balances high-level value with "get-your-hands-dirty" code.

---

## The Standard Format: The "Developer Adoption" Template

When creating a page for an open-source feature, follow this modular structure:

### 1. The Hero Section (The "Why")
* **Headline:** Not just "SD-JWT Support," but "Selective Disclosure with SD-JWT."
* **The Value Prop:** One sentence on why this matters (e.g., "Implement privacy-preserving credentials that allow users to share only the data they choose.")
* **Status Badge:** Clearly mark it as `Beta`, `Stable`, or `Experimental`.

### 2. Conceptual Overview (The "How it works")
Don't rewrite the whole spec. Explain how *your* product specifically implements it. 
* **Diagram:** A simple sequence diagram showing the Issuer, Holder, and Verifier flow using your SDK/Tool.
* **Prerequisites:** List what the developer needs before they can use this (e.g., specific version of your library, an existing DID).

### 3. The "Hello World" / Quick Start
This is the most critical part for adoption.
* **Code Snippet:** A concise, copy-pasteable example of creating an SD-JWT using your tool.
* **CLI Example:** If you have a CLI, show the one-liner command to generate a credential.

### 4. Deep Dive / Configuration
* **Feature Details:** Explain specific parameters (e.g., handling salt, formatting disclosures).
* **Security Considerations:** Since this is Verifiable Credentials, mention how signatures are handled or how to avoid common pitfalls.

### 5. Reference & Resources
* **API Link:** Link directly to the auto-generated API docs (Docs-as-code) for the classes/methods used.
* **Sample Repo:** Link to a standalone GitHub "Starter Kit" or demo app using SD-JWT.
* **Spec Link:** Link to the IETF SD-JWT draft or the W3C VC spec for those who want the raw theory.

## Why this is a distinct Content Type
In technical communication frameworks (like the **Diátaxis** framework), this falls under **Tutorials** or **Explanation**, but for adoption, we treat it as a **Marketing-Technical Hybrid**.

| Content Type | Purpose | Tone |
| :--- | :--- | :--- |
| **Release Notes** | What changed? | Brief, bulleted, chronological. |
| **Feature Page** | Why and how do I use *this*? | Persuasive, instructional, code-heavy. |
| **API Reference** | What does this parameter do? | Exhaustive, dry, structured. |

---

## Practical Application: "SD-JWT for Verifiable Credentials"
For your GitHub-based docs-as-code setup, you should structure the file like this:

`docs/features/selective-disclosure-jwt.md`

1.  **Intro:** "Our SDK now supports SD-JWT (Selective Disclosure JSON Web Tokens) allowing for minimal data disclosure..."
2.  **Code Block:** Show a `const credential = new SDJWTBuilder()...` example.
3.  **Visual:** Use a Mermaid.js diagram (which renders natively in GitHub) to show the disclosure flow.
4.  **CTA (Call to Action):** "Star the repo" or "Join our Discord to discuss VC implementation."

---

## Identifying this in your GitHub Project
Since you are using GitHub Issues/Projects, you should create a specific **Issue Template** for these pages called **"New Feature Documentation"**. This ensures that every time a developer builds a feature like SD-JWT, they are prompted to provide the "Quick Start" code and "Value Prop" immediately.

Would you like me to draft a Markdown template you can copy-paste into your GitHub repo as a starter for this page?