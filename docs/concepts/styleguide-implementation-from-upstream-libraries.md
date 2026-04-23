Beyond **The Good Docs Project (TGDP)**, there are several other "libraries" and frameworks that provide standardized templates, especially for a **Senior Technical Writer** working in a **Product Engineering** environment.

In 2026, the trend has shifted from "static templates" to "documentation frameworks" that help you structure the entire information architecture. Here are the most worthy alternatives:

### 1. Diátaxis (The "Structural" Framework)
While TGDP gives you the *content* of a page, **Diátaxis** gives you the *logic* of your entire documentation site. It is currently the industry standard for high-level documentation architecture.
* **The Library:** It divides all technical content into four quadrants: **Tutorials, How-to Guides, Explanation, and Reference.**
* **Why it’s a great partner to TGDP:** You use Diátaxis to decide *what* to write, and TGDP templates to decide *how* to write it.
* **Where to find it:** [diataxis.fr](https://diataxis.fr/)

### 2. Cloud Native Computing Foundation (CNCF) Guide
If your organization works with Kubernetes, Docker, or Cloud-Native tech, the CNCF maintains a specialized library of documentation templates.
* **Focus:** It is much more technical than TGDP, focusing on **Architecture Diagrams, Installation flows, and CLI references.**
* **Best for:** Deep infrastructure or platform engineering docs.

### 3. Microsoft’s "Style Guide" & Templates
Microsoft has open-sourced their internal documentation standards (the same ones used for Azure and .NET). 
* **The Library:** They provide specific templates for **Release Notes, Quickstarts, and API Tutorials.**
* **The Value:** Since most developers are familiar with Microsoft docs, using these templates makes your documentation feel "familiar" and "professional" instantly.
* **Where to find it:** [Microsoft Style Guide (GitHub)](https://github.com/MicrosoftDocs/microsoft-style-guide)

### 4. Docsie’s Engineering Template Library (New for 2026)
Docsie has released an open-source collection of **89 specific engineering templates** across 15 categories.
* **What's inside:** It includes very specific "Product Engineering" needs like **System Architecture Overviews, Runbooks, and Incident Response procedures.**
* **Why use it:** It fills the gaps that TGDP sometimes misses, particularly regarding DevOps and "Internal-facing" documentation.

---

### Comparison of the "Big Three" Template Libraries

| Library | Best For... | Type of Content |
| :--- | :--- | :--- |
| **The Good Docs Project** | General Software Docs | High-quality, researched Markdown templates. |
| **Diátaxis** | Information Architecture | The "Strategy" of how to organize your site. |
| **Docsie Engineering** | DevOps & System Docs | Heavy technical docs like Runbooks and Specs. |

### 💡 Strategy for your new role:
Instead of picking just one, I recommend a **"Hybrid Library"** approach:
1.  Use **Diátaxis** to set the navigation/menu structure of your new portal.
2.  Use **TGDP** for your "User-facing" docs (How-to's and Concepts).
3.  Use the **Microsoft Style Guide** to set the tone and grammar rules for the whole team.

**Would you like me to find a specific "System Architecture" template for you? These are usually the hardest to write from scratch in a new engineering role.**



Since you are building a **`.github`** repository (which is a special "Profile" or "Organization-wide" metadata repo in GitHub), you have a unique opportunity to create a **"Single Source of Truth"** for your entire engineering team.

To bring in external libraries like **The Good Docs Project** or **Diátaxis** while keeping them as **Upstream**, the professional standard is to use **Git Submodules**.

### 1. The Strategy: Git Submodules
A submodule allows you to keep a different Git repository as a subdirectory of your own. This is perfect for your needs because:
* It keeps the external library's history separate from yours.
* You can "pull" the latest updates from the original authors at any time.
* Your **Custom Agents** can be trained to look specifically into these subdirectories for "Gold Standard" examples.

---

### 2. How to set it up (The Commands)
Open your terminal in your local `.github` repo and run:

```bash
# 1. Create a folder for external standards
mkdir -p internal/upstream-libraries

# 2. Add The Good Docs Project as a submodule
git submodule add https://github.com/thegooddocsproject/templates.git internal/upstream-libraries/good-docs

# 3. Add Diátaxis (if applicable) or others
git submodule add https://github.com/diataxis/diataxis.fr.git internal/upstream-libraries/diataxis

# 4. Commit the new tracking files
git add .
git commit -m "chore: add upstream documentation libraries as submodules"
git push origin main
```

---

### 3. How to "Sync" with Upstream
One month from now, if The Good Docs Project releases a new "API Reference" template, you don't need to manually copy it. Just run:

```bash
git submodule update --remote --merge
```
This will fetch the latest version of their templates into your `internal/upstream-libraries/good-docs` folder.

---

### 4. Integrating with your Custom Agents
Since you are building **Custom Agents** in `.github/skills` or `.github/agents`, you can now point them to these directories in their **System Instructions**.

**Example Agent Instruction:**
> *"When generating a new 'How-to' guide, first analyze the template structure found in `internal/upstream-libraries/good-docs/templates/how-to.md`. Ensure the output matches this structural hierarchy exactly, but apply our internal brand voice."*

### 5. Why the `.github` repo is the perfect place:
* **Visibility:** Any repository in your organization can reference these templates.
* **Automation:** You can set up a **GitHub Action** in this repo that runs once a week, checks for updates in the upstream submodules, and automatically creates a Pull Request if a "Template" has been updated. This ensures your organization's documentation standards are **self-healing** and always up-to-date.

**Since you're setting this up on your Mac M2, do you have a specific folder structure in mind for your `skills` and `agents` within the .github repo, or should I suggest a standard layout?**

---

## 6. FAQ: Removing Upstream Libraries Later

### **Q: What if in the future I want to do away with upstream-library? Will it create any problem?**

**A: No significant problems!** Submodules are completely **removable** and **non-intrusive**.

### **Clean Removal Path**

If you decide to drop submodules later:

```bash
# 1. Remove the submodule
git submodule deinit -f internal/upstream-libraries/good-docs
git rm -f internal/upstream-libraries/good-docs

# 2. Commit the removal
git add .gitmodules .
git commit -m "chore: remove upstream-libraries submodule"

# 3. Clean local reference
rm -rf .git/modules/internal/upstream-libraries/

# 4. Push
git push origin main
```

✅ **Your skills, prompts, templates remain untouched**

### **Potential Friction Points (Minimal)**

| Area | Impact | Mitigation |
|------|--------|-----------|
| **Agent Instructions** | References to `internal/upstream-libraries/` become broken paths | Update agent system prompts (5 min) |
| **Skill Scripts** | If any skills hardcode paths to submodules | Use relative paths with fallbacks |
| **Team Knowledge** | Team might expect submodules to exist | Update internal docs |

### **Smart Setup Strategy (Do This Now)**

To make future removal **painless**, structure your agent instructions with a **fallback mechanism**:

```yaml
# In your agent system prompts (use a VARIABLE, not hardcoded path)
REFERENCE_LIBRARY_PATH: "internal/upstream-libraries"  # Optional

When REFERENCE_LIBRARY_PATH exists:
  - Use upstream standards
When REFERENCE_LIBRARY_PATH does NOT exist:
  - Fall back to your own standards in .github/templates
```

This way:
- ✅ Submodules are **optional** by design
- ✅ Agents work **with or without** them
- ✅ Zero friction to remove later

### **Bottom Line**

✅ **No lock-in** - submodules are completely optional infrastructure  
✅ **Easy removal** - ~10 minute cleanup if needed  
✅ **Your work is safe** - custom skills/prompts/templates are independent  
✅ **Reversible** - can add them back anytime  

**Don't worry about being "locked in." This is clean, modular architecture.** Experiment with submodules now and remove them later with zero consequences if your team outgrows them or you develop better internal standards.

---

## 7. Alternative: Direct URL References (Even Simpler!)

### **Q: Can I refer to TGDP URLs directly for agents instead of creating upstream-library?**

**A: YES! And this is SIMPLER and often BETTER than submodules!** 🎯

You can have agents **fetch and reference external URLs directly**. No upstream-library complexity needed.

### **Three Approaches**

#### **Approach 1: Direct URL in Agent Instructions**

```yaml
# .github/agents/doc-writer/AGENT.md

## System Instructions

Reference these external standards when generating documentation:

**Structure Reference:**
- Diátaxis Framework: https://diataxis.fr/
- TGDP How-To Template: https://raw.githubusercontent.com/thegooddocsproject/templates/main/templates/how-to.md
- TGDP Explanation: https://raw.githubusercontent.com/thegooddocsproject/templates/main/templates/explanation.md

**Style Reference:**
- Microsoft Style Guide: https://github.com/MicrosoftDocs/microsoft-style-guide

When generating documentation:
1. Review the structure at the above URLs
2. Apply the patterns to your output
3. Maintain our internal brand voice
```

✅ **Pros:** No complexity, always fresh, lightweight  
❌ **Cons:** Requires internet, relies on URLs staying live

#### **Approach 2: Centralized Standards Registry (RECOMMENDED)**

Store all reference URLs in one place for easy updates:

```yaml
# .github/standards/references.yaml
documentation_standards:
  diataxis:
    url: "https://diataxis.fr/"
    description: "Information architecture framework"
    
  good_docs_project:
    howto: "https://raw.githubusercontent.com/thegooddocsproject/templates/main/templates/how-to.md"
    explanation: "https://raw.githubusercontent.com/thegooddocsproject/templates/main/templates/explanation.md"
    tutorial: "https://raw.githubusercontent.com/thegooddocsproject/templates/main/templates/tutorial.md"
    
  microsoft_style:
    url: "https://github.com/MicrosoftDocs/microsoft-style-guide"
    description: "Tone, voice, and style guidelines"
    
  docsie_engineering:
    url: "https://github.com/docsie/engineering-templates"
    description: "DevOps and system documentation templates"
```

Your agents reference this file:

```yaml
# .github/agents/doc-writer/AGENT.md

Load documentation standards from: .github/standards/references.yaml

When generating [doc_type]:
  1. Fetch the relevant URL from references.yaml
  2. Analyze structure and patterns
  3. Apply to output while maintaining brand voice
```

✅ **Pros:** Centralized, easy to update, clean, lightweight, always fresh  
✅ **Single source of truth for all standards**

#### **Approach 3: Runtime Fetch with Optional Caching**

Agents fetch URLs at runtime with optional local caching:

```typescript
// .github/skills/fetch-standard.ts
async function fetchDocumentationStandard(standardName: string) {
  const standards = require('./standards/references.yaml');
  const url = standards.documentation_standards[standardName].url;
  
  // Optional: Check local cache first
  const cached = readFromCache(standardName);
  if (cached && isFresh(cached)) return cached;
  
  // Fetch fresh content
  const content = await fetch(url);
  cacheLocally(standardName, content);
  return content;
}
```

### **Quick Comparison**

| Aspect | Submodules | Direct URLs | URL Registry |
|--------|-----------|------------|--------------|
| **Setup** | Medium | Minimal | Minimal |
| **Freshness** | Manual | Automatic | Automatic |
| **Repo Size** | Large | Tiny | Tiny |
| **Maintenance** | High | Low | Low |
| **Centralized Control** | No | Per-agent | **YES** |
| **Easy URL Updates** | No | Medium | **YES** |

### **Recommended Setup**

**Skip submodules entirely.** Use this structure instead:

```
.github/
├── standards/
│   └── references.yaml              # All external URLs in one place
├── agents/
│   ├── doc-writer/
│   │   └── AGENT.md                 # Ref: .github/standards/references.yaml
│   └── code-analyzer/
│       └── AGENT.md
└── skills/
    └── fetch-documentation.ts       # Utility to resolve URLs
```

### **Why This Wins**

✅ **No complexity** - just a YAML file with URLs  
✅ **Always fresh** - agents fetch latest standards on demand  
✅ **Lightweight** - your repo stays clean (no downloaded content)  
✅ **Easy to maintain** - change URLs in one place  
✅ **Team-friendly** - obvious what standards you're using  
✅ **Future-proof** - trivial to remove or swap standards  
✅ **No git commands needed** - just edit YAML  

**This is the modern approach (2026).** Hard-coding external dependencies into your repo is yesterday's pattern. Direct URL references are cleaner and more maintainable. 🚀

