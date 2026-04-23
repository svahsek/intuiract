# Ideal Folder Structure for Global .github Repository

This document outlines the recommended folder structure and best practices for organizing resources in your global `.github` repository.

## Directory Structure

```
consumer-repo/.github/
├── workflows/              # Reusable GitHub Actions workflows
│   ├── ci.yml
│   ├── cd.yml
│   ├── security-scan.yml
│   └── lint.yml
│
├── actions/                # Custom GitHub Actions
│   ├── deploy/
│   │   ├── action.yml
│   │   └── index.js
│   └── notify/
│       ├── action.yml
│       └── index.js
│
├── agents/                 # Custom AI Agents (VS Code MCP)
│   ├── keshavs.agent.md
│   ├── development.agent.md
│   └── README.md
│
├── skills/                 # Custom Skills/SKILL definitions
│   ├── code-review.skill.md
│   ├── security-audit.skill.md
│   └── README.md
│
├── scripts/                # Reusable shell/automation scripts
│   ├── install-deps.sh
│   ├── deploy.sh
│   └── bump-version.sh
│
├── templates/              # Issue/PR templates
│   ├── bug_report.md
│   ├── feature_request.md
│   └── pull_request.md
│
├── configs/                # Shared configurations
│   ├── eslint.config.js
│   ├── prettier.config.js
│   ├── husky-hooks/
│   └── lint-staged.config.js
│
├── docs/                   # Documentation for agents/skills
│   ├── AGENTS.md
│   ├── SKILLS.md
│   ├── WORKFLOWS.md
│   └── SETUP.md
│
└── README.md               # Overview of what's available
```

## Key Practices

### 1. README Files

Put a `README.md` in each folder explaining what's available:

```markdown
# Agents

List of available agents:
- `keshavs.agent.md` - Your custom agent description
- Usage instructions...
```

### 2. Naming Conventions

```
workflows/     ✓ (GitHub standard)
actions/       ✓ (GitHub standard)
agents/        ✓ (custom - clearly named)
skills/        ✓ (custom - clearly named)
scripts/       ✓ (custom - reusable scripts)
templates/     ✓ (GitHub standard)
configs/       ✓ (shared configs)
docs/          ✓ (documentation)
```

### 3. Documentation Structure

```
docs/
├── AGENTS.md          # How to use agents
├── SKILLS.md          # How to use skills
├── WORKFLOWS.md       # How to reference workflows
├── GETTING-STARTED.md # Quick start guide
└── FAQ.md
```

### 4. Root README.md Template

```markdown
# Global GitHub Config

This `.github` repository contains:
- 📋 Reusable workflows
- 🔧 Custom actions
- 🤖 Custom agents
- 💡 Custom skills
- 🛠️ Shared scripts
- ⚙️ Shared configurations

## Quick Links
- [Agents Documentation](docs/AGENTS.md)
- [Skills Documentation](docs/SKILLS.md)
- [Workflows Documentation](docs/WORKFLOWS.md)

## Usage
See individual folders for specific documentation.
```

## Best Practices

| Practice | Why |
|----------|-----|
| **Separate folders** | Easy to find and reference resources |
| **README in each folder** | Self-documenting |
| **Consistent naming** | `resource-name.type.md` (e.g., `code-review.skill.md`) |
| **Central docs/ folder** | Single source of truth for documentation |
| **Metadata/version files** | Track versions of agents/skills |
| **Test folder (optional)** | Test agents/workflows before deploying |

## Optional: Add Version Control

```
agents/
├── keshavs.agent.md
└── _metadata.json    # Track versions, tags, etc.
```

Example `_metadata.json`:

```json
{
  "agents": {
    "keshavs": {
      "version": "1.0.0",
      "updated": "2026-02-12",
      "description": "Main custom agent"
    }
  }
}
```

## Implementation Steps

1. Create the folder structure as outlined above
2. Add README files to each folder
3. Document resources in the `docs/` folder
4. Update the root `README.md` with links to documentation
5. Keep metadata files updated as you add/update resources

This structure provides a solid, scalable foundation for managing your global GitHub configuration and extensions.
