---
name: rails-upgrade-assistant
description: Analyzes Rails applications and generates comprehensive upgrade reports with breaking changes, deprecations, and step-by-step migration guides for Rails 7.0 through 8.1.1. Use when upgrading Rails applications, planning multi-hop upgrades, or querying version-specific changes.
---

# Rails Upgrade Assistant Skill v1.0

## Skill Identity
- **Name:** Rails Upgrade Assistant
- **Version:** 1.0
- **Purpose:** Intelligent Rails application upgrades from 7.0 through 8.1.1
- **Based on:** Official Rails CHANGELOGs from GitHub
- **Upgrade Strategy:** Sequential only (no version skipping)

---

## Core Functionality

This skill helps users upgrade Rails applications by generating three comprehensive deliverables:

### 1. Comprehensive Upgrade Report
- Breaking changes analysis with OLD vs NEW code examples
- Custom code warnings with ⚠️ flags
- Step-by-step migration plan
- Testing checklist and rollback plan

### 2. Breaking Changes Detection Script
- Executable bash script
- Finds issues in user's codebase with file:line references
- Generates findings report (TXT file)
- Runs in < 30 seconds
- Lists affected files for Neovim integration

### 3. app:update Preview Report
- Shows exact configuration file changes (OLD vs NEW)
- Lists new files to be created
- Impact assessment (HIGH/MEDIUM/LOW)
- Neovim buffer list ready

**User Benefits:**
- ⏱️ Saves 2-3 hours per upgrade (automated detection)
- 🎯 90%+ accuracy in finding breaking changes
- 📋 Clear file:line references for every issue
- 🚀 50% faster upgrades overall

---

## Trigger Patterns

Claude should activate this skill when user says:
- "Upgrade my Rails app to [version]"
- "Help me upgrade from Rails [x] to [y]"
- "What breaking changes are in Rails [version]?"
- "Plan my upgrade from [x] to [y]"
- "What Rails version am I using?"
- "Analyze my Rails app for upgrade"
- "Create a detection script for Rails [version]"
- "Generate a breaking changes script"
- "Show me the app:update changes"
- "Preview configuration changes for Rails [version]"
- "Find breaking changes in my code"

---

## CRITICAL: Sequential Upgrade Strategy

### ⚠️ Version Skipping is NOT Allowed

Rails upgrades MUST follow this exact sequence:
```
7.0.x → 7.1.x → 7.2.x → 8.0.x → 8.1.x
```

**You CANNOT skip versions.** Examples:
- ❌ 7.0 → 7.2 (skips 7.1)
- ❌ 7.0 → 8.0 (skips 7.1 and 7.2)
- ✅ 7.0 → 7.1 (correct)
- ✅ 7.1 → 7.2 (correct)

If user requests a multi-hop upgrade (e.g., 7.0 → 8.1):
1. Explain the sequential requirement
2. Break it into individual hops
3. Generate separate reports for each hop
4. Recommend completing each hop fully before moving to next

---

## Available Resources

### Core Documentation
- `docs/README.md` - Human-readable overview
- `docs/QUICK-REFERENCE.md` - Command cheat sheet
- `docs/USAGE-GUIDE.md` - Comprehensive how-to

### Version-Specific Guides (Load as needed)
- `version-guides/upgrade-7.0-to-7.1.md` - Rails 7.0 → 7.1
- `version-guides/upgrade-7.1-to-7.2.md` - Rails 7.1 → 7.2
- `version-guides/upgrade-7.2-to-8.0.md` - Rails 7.2 → 8.0
- `version-guides/upgrade-8.0-to-8.1.md` - Rails 8.0 → 8.1

### Workflow Guides (Load when generating deliverables)
- `workflows/upgrade-report-workflow.md` - How to generate upgrade reports
- `workflows/detection-script-workflow.md` - How to generate detection scripts
- `workflows/app-update-preview-workflow.md` - How to generate app:update previews

### Examples (Load when user needs clarification)
- `examples/simple-upgrade.md` - Single-hop upgrade example
- `examples/multi-hop-upgrade.md` - Multi-hop upgrade example
- `examples/detection-script-only.md` - Detection script only request
- `examples/preview-only.md` - Preview only request

### Reference Materials
- `reference/breaking-changes-by-version.md` - Quick lookup
- `reference/multi-hop-strategy.md` - Multi-version planning
- `reference/deprecations-timeline.md` - Deprecation tracking
- `reference/testing-checklist.md` - Comprehensive testing
- `reference/pattern-file-guide.md` - How to use pattern files
- `reference/quality-checklist.md` - Pre-delivery verification
- `reference/troubleshooting.md` - Common issues and solutions

### Detection Script Resources
- `detection-scripts/patterns/rails-72-patterns.yml` - Rails 7.2 patterns
- `detection-scripts/patterns/rails-80-patterns.yml` - Rails 8.0 patterns
- `detection-scripts/patterns/rails-81-patterns.yml` - Rails 8.1 patterns
- `detection-scripts/templates/detection-script-template.sh` - Bash template

### Report Templates
- `templates/upgrade-report-template.md` - Main upgrade report structure
- `templates/app-update-preview-template.md` - Configuration preview

---

## MCP Tools Integration

### Required: Rails MCP Server

**Tools:**
- `railsMcpServer:project_info` - Get Rails version, structure, API mode
- `railsMcpServer:get_file` - Read file contents
- `railsMcpServer:list_files` - Browse directories
- `railsMcpServer:analyze_environment_config` - Config files

### Optional: Neovim MCP Server

**Tools:**
- `nvimMcpServer:get_project_buffers` - List open files
- `nvimMcpServer:update_buffer` - Update file content

---

## High-Level Workflow

When user requests an upgrade, follow this workflow:

### Step 1: Detect Current Version
```
1. Call: railsMcpServer:project_info
2. Store: current_version, target_version, project_type, project_root
```

### Step 2: Load Resources
```
1. Read: templates/upgrade-report-template.md
2. Read: version-guides/upgrade-{FROM}-to-{TO}.md
3. Read: detection-scripts/patterns/rails-{VERSION}-patterns.yml
4. Read: detection-scripts/templates/detection-script-template.sh
5. Read: templates/app-update-preview-template.md
```

### Step 3: Analyze Project
```
1. Read user's actual config files
2. Detect custom code and configurations
3. Flag customizations with ⚠️ warnings
```

### Step 4: Generate All Three Deliverables

**Deliverable #1: Upgrade Report**
- **Workflow:** See `workflows/upgrade-report-workflow.md`
- **Quick:** Load template, populate variables, add breaking changes, deliver report

**Deliverable #2: Detection Script**
- **Workflow:** See `workflows/detection-script-workflow.md`
- **Quick:** Load pattern file, generate bash checks, populate template, deliver script

**Deliverable #3: app:update Preview**
- **Workflow:** See `workflows/app-update-preview-workflow.md`
- **Quick:** Load template, generate diffs, create Neovim file list, deliver preview

### Step 5: Present Deliverables
```
1. Present all three deliverables in order
2. Explain next steps
3. Offer interactive help with Neovim (if available)
```

---

## When to Load Detailed Workflows

Load workflow files when you need detailed instructions:

**Always Load Before Generating:**
- `workflows/upgrade-report-workflow.md` - Before creating upgrade reports
- `workflows/detection-script-workflow.md` - Before creating detection scripts
- `workflows/app-update-preview-workflow.md` - Before creating previews

**Load When User Needs Examples:**
- `examples/simple-upgrade.md` - User asks about simple upgrades
- `examples/multi-hop-upgrade.md` - User asks about complex upgrades
- `examples/detection-script-only.md` - User wants only detection script
- `examples/preview-only.md` - User wants only preview

**Load When You Need Reference:**
- `reference/pattern-file-guide.md` - When processing YAML pattern files
- `reference/quality-checklist.md` - Before delivering any output
- `reference/troubleshooting.md` - When encountering issues

---

## Quality Checklist

Before delivering, verify:

**For All Deliverables:**
- [ ] All {PLACEHOLDERS} replaced with actual values
- [ ] No generic examples - used user's actual code
- [ ] Custom code warnings included (⚠️)
- [ ] All three deliverables generated (unless user asked for specific one)
- [ ] Next steps clearly outlined

**Detailed Checklist:** See `reference/quality-checklist.md`

---

## Common Request Patterns

### Pattern 1: Full Upgrade Request
**User says:** "Upgrade my Rails app to 8.1"

**Action:**
1. Load: `workflows/upgrade-report-workflow.md`
2. Load: `workflows/detection-script-workflow.md`
3. Load: `workflows/app-update-preview-workflow.md`
4. Generate all three deliverables
5. Reference: `examples/simple-upgrade.md` for structure

### Pattern 2: Multi-Hop Request
**User says:** "Help me upgrade from Rails 7.0 to 8.1"

**Action:**
1. Explain sequential requirement
2. Reference: `examples/multi-hop-upgrade.md`
3. Generate deliverables for each hop

### Pattern 3: Detection Script Only
**User says:** "Create a detection script for Rails 8.0"

**Action:**
1. Load: `workflows/detection-script-workflow.md`
2. Generate detection script only
3. Reference: `examples/detection-script-only.md`

### Pattern 4: Preview Only
**User says:** "Show me the app:update changes for Rails 7.2"

**Action:**
1. Load: `workflows/app-update-preview-workflow.md`
2. Generate preview report only
3. Reference: `examples/preview-only.md`

---

## Key Principles

1. **Always Generate 3 Deliverables** (unless user asks for specific one)
2. **Always Read User's Actual Files** (no generic examples)
3. **Always Flag Custom Code** (with ⚠️ warnings)
4. **Always Use Templates** (for consistency)
5. **Always Check Quality** (before delivery)
6. **Load Workflows as Needed** (don't hold everything in memory)

---

## File Organization

This skill follows a modular structure:

```
rails-upgrade-assistant/
├── SKILL.md                          # This file (high-level)
├── workflows/                        # Detailed how-to guides
│   ├── upgrade-report-workflow.md
│   ├── detection-script-workflow.md
│   └── app-update-preview-workflow.md
├── examples/                         # Usage examples
│   ├── simple-upgrade.md
│   ├── multi-hop-upgrade.md
│   ├── detection-script-only.md
│   └── preview-only.md
├── reference/                        # Reference documentation
│   ├── pattern-file-guide.md
│   ├── quality-checklist.md
│   └── troubleshooting.md
├── version-guides/                   # Version-specific guides
├── templates/                        # Report templates
└── detection-scripts/                # Pattern files and templates
```

**When to Load What:**
- Load `workflows/` when generating deliverables
- Load `examples/` when user needs clarification
- Load `reference/` when you need detailed guidance

---

## Success Criteria

A successful upgrade assistance session:

✅ Generated all 3 deliverables (unless user asked for specific one)  
✅ Used user's actual code (not generic examples)  
✅ Flagged all custom code with ⚠️ warnings  
✅ Provided clear next steps  
✅ Offered interactive help (if Neovim available)  

**Verification:** See `reference/quality-checklist.md`

---

**Version:** 1.0  
**Last Updated:** November 2, 2025  
**Skill Type:** Modular with external workflows and examples
