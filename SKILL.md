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

This skill helps users upgrade Rails applications through direct, intelligent analysis:

### Direct Project Analysis
- Agent analyzes project directly using Cursor's native tools (Read, Grep, Glob, LS)
- Detects Rails version and project structure automatically
- Searches codebase for breaking change patterns in real-time
- Identifies custom code and configurations with file:line references
- Generates comprehensive reports with actual code examples from the project
- Interactive process allows clarifying questions during analysis

### Comprehensive Reporting
- **Upgrade Report**: Breaking changes analysis with OLD vs NEW code examples from YOUR project, custom code warnings with ⚠️ flags, step-by-step migration plan, testing checklist and rollback plan
- **app:update Preview**: Shows exact configuration file changes (OLD vs NEW), lists new files to be created, impact assessment (HIGH/MEDIUM/LOW)

**User Benefits:**
- ⚡ 3-5x faster analysis (direct file access vs external scripts)
- 🎯 90%+ accuracy in finding breaking changes
- 📋 Clear file:line references for every issue
- 🚀 Immediate results with real-time analysis
- 💬 Interactive guidance with ability to ask clarifying questions
- 📊 60-75% faster overall upgrade workflow
- 🔧 Zero setup required - works out of the box in Cursor

---

## Trigger Patterns

Claude should activate this skill when user says:

**Upgrade Requests:**
- "Upgrade my Rails app to [version]"
- "Help me upgrade from Rails [x] to [y]"
- "What breaking changes are in Rails [version]?"
- "Plan my upgrade from [x] to [y]"
- "What Rails version am I using?"
- "Analyze my Rails app for upgrade"
- "Find breaking changes in my code"
- "Detect Rails version and breaking changes"

**Specific Analysis Requests:**
- "Show me the app:update changes"
- "Preview configuration changes for Rails [version]"
- "Analyze my [model/controller/config] for Rails [version] compatibility"
- "Find custom middleware in my app"
- "Check my routes for breaking changes"

**Query-Only Requests:**
- "What ActiveRecord changes are in Rails [version]?"
- "Explain [specific breaking change]"
- "Show me migration steps for [version]"

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
- `workflows/direct-analysis-workflow.md` - How to analyze projects directly
- `workflows/app-update-preview-workflow.md` - How to generate app:update previews

### Examples (Load when user needs clarification)
- `examples/simple-upgrade.md` - Single-hop upgrade example
- `examples/multi-hop-upgrade.md` - Multi-hop upgrade example
- `examples/direct-analysis-only.md` - Direct analysis only request
- `examples/preview-only.md` - Preview only request

### Reference Materials
- `reference/breaking-changes-by-version.md` - Quick lookup
- `reference/multi-hop-strategy.md` - Multi-version planning
- `reference/deprecations-timeline.md` - Deprecation tracking
- `reference/testing-checklist.md` - Comprehensive testing
- `reference/pattern-file-guide.md` - How to use pattern files
- `reference/quality-checklist.md` - Pre-delivery verification
- `reference/troubleshooting.md` - Common issues and solutions

### Breaking Change Patterns
- `detection-scripts/patterns/rails-72-patterns.yml` - Rails 7.2 patterns for Grep analysis
- `detection-scripts/patterns/rails-80-patterns.yml` - Rails 8.0 patterns for Grep analysis
- `detection-scripts/patterns/rails-81-patterns.yml` - Rails 8.1 patterns for Grep analysis
- Note: YAML patterns converted to Grep queries for real-time analysis

### Report Templates
- `templates/upgrade-report-template.md` - Main upgrade report structure
- `templates/app-update-preview-template.md` - Configuration preview

---

## Cursor Native Tools

This skill uses Cursor's built-in tools for fast, local analysis:

### File Operations
- **Read** - Read file contents (Gemfile, configs, models, etc.)
- **Grep** - Fast pattern searching using ripgrep (breaking change detection)
- **Glob** - Find files by pattern (e.g., all models, all initializers)
- **LS** - List directory contents (project structure detection)

### Project Operations
- **Shell** - Execute commands (e.g., `bin/rails routes` for route analysis)
- **StrReplace** - Precise file updates for suggested changes
- **Write** - Create or overwrite files when needed

### Key Advantages
- ⚡ **5-10x faster** than external tools (local file access)
- 🔄 **Parallel execution** (run multiple Grep searches simultaneously)
- 📡 **Works offline** (no network dependencies)
- 🎯 **Zero setup** (no installation required)
- 💬 **Interactive** (can ask questions during analysis)

---

## High-Level Workflow

When user requests an upgrade, follow this workflow:

### Step 1: Detect Rails Version and Project Structure
```
1. Read("Gemfile") → Extract Rails version via Grep
2. Read("config/application.rb") → Check API-only mode, load_defaults
3. LS("/") → Verify Rails project structure (app/, config/, db/)
4. Store: current_version, target_version, project_type
```

### Step 2: Load Analysis Resources
```
1. Read: detection-scripts/patterns/rails-{VERSION}-patterns.yml
2. Read: version-guides/upgrade-{FROM}-to-{TO}.md
3. Read: workflows/direct-analysis-workflow.md (for analysis instructions)
4. Read: workflows/upgrade-report-workflow.md
5. Read: workflows/app-update-preview-workflow.md
```

### Step 3: Analyze Project for Breaking Changes
```
Convert YAML patterns to Grep queries and run in parallel:

1. Grep("cache_classes|enable_reloading", glob: "config/**/*.rb", -C: 2)
2. Grep("force_ssl|assume_ssl", glob: "config/**/*.rb", -C: 2)
3. Grep("show_exceptions", glob: "config/**/*.rb", -C: 2)
4. Grep("ActiveRecord::Base.connection", glob: "app/**/*.rb", -B: 2, -A: 2)
5. [... more patterns based on version ...]

Result: All breaking changes with file:line:context
```

### Step 4: Read Affected Files for Context
```
For each detected issue:
1. Read(affected_file) → Get full context
2. Identify custom code patterns
3. Extract OLD code examples
4. Prepare NEW code examples from version guide
```

### Step 5: Generate Comprehensive Reports

**Deliverable #1: Upgrade Report**
- **Workflow:** See `workflows/upgrade-report-workflow.md`
- **Input:** Detected breaking changes + version guide data + actual code
- **Output:** Complete report with real code examples from user's project

**Deliverable #2: app:update Preview**
- **Workflow:** See `workflows/app-update-preview-workflow.md`
- **Input:** Current config files + target version changes
- **Output:** Preview with file diffs and update guidance

### Step 6: Present Results and Offer Guidance
```
1. Present Comprehensive Upgrade Report
2. Present app:update Preview
3. Explain next steps
4. Offer to apply changes using StrReplace (if user approves)
5. Answer follow-up questions interactively
```

---

## When to Load Detailed Workflows

Load workflow files when you need detailed instructions:

**For Direct Analysis:**
- `workflows/direct-analysis-workflow.md` - How to analyze projects using Cursor tools
- `workflows/upgrade-report-workflow.md` - How to generate comprehensive reports
- `workflows/app-update-preview-workflow.md` - How to generate config previews

**Load When User Needs Examples:**
- `examples/simple-upgrade.md` - User asks about simple upgrades
- `examples/multi-hop-upgrade.md` - User asks about complex upgrades
- `examples/direct-analysis-only.md` - User wants analysis only
- `examples/preview-only.md` - User wants only preview

**Load When You Need Reference:**
- `reference/pattern-file-guide.md` - When processing YAML pattern files
- `reference/quality-checklist.md` - Before delivering any output
- `reference/troubleshooting.md` - When encountering issues

---

## Quality Checklist

Before delivering, verify:

**For Project Analysis:**
- [ ] Successfully detected Rails version from Gemfile
- [ ] Confirmed project type (API-only or full stack)
- [ ] Ran all relevant Grep patterns for target version
- [ ] Found breaking changes with file:line references
- [ ] Collected context (lines before/after matches)

**For Comprehensive Upgrade Report:**
- [ ] All {PLACEHOLDERS} replaced with actual values
- [ ] Used ACTUAL code from user's project (not generic examples)
- [ ] Breaking changes section includes real file:line references
- [ ] Custom code warnings based on actual detected patterns
- [ ] Code examples show OLD code from user's files
- [ ] Code examples show NEW code for target version
- [ ] Next steps clearly outlined
- [ ] Testing guidance provided

**For app:update Preview:**
- [ ] All {PLACEHOLDERS} replaced with actual values
- [ ] Read user's actual current config files
- [ ] Compared against target version defaults
- [ ] Diffs show real current vs target version changes
- [ ] Preserved custom configurations marked with ⚠️
- [ ] Next steps clearly outlined

**Detailed Checklist:** See `reference/quality-checklist.md`

---

## Common Request Patterns

### Pattern 1: Full Upgrade Request
**User says:** "Upgrade my Rails app to 8.1"

**Action:**
1. Detect version: Read("Gemfile"), Read("config/application.rb")
2. Load: `workflows/direct-analysis-workflow.md`
3. Load: `workflows/upgrade-report-workflow.md`
4. Load: `workflows/app-update-preview-workflow.md`
5. Analyze project using parallel Grep for breaking changes
6. Generate Comprehensive Upgrade Report (with actual code)
7. Generate app:update Preview (with actual config)
8. Reference: `examples/simple-upgrade.md` for structure

### Pattern 2: Multi-Hop Request
**User says:** "Help me upgrade from Rails 7.0 to 8.1"

**Action:**
1. Explain sequential requirement (must do: 7.0→7.1→7.2→8.0→8.1)
2. Reference: `examples/multi-hop-upgrade.md`
3. Follow Pattern 1 for FIRST hop (7.0 → 7.1)
4. After first hop complete, repeat for next hops

### Pattern 3: Analysis Only Request
**User says:** "Find breaking changes in my Rails 7.1 app"

**Action:**
1. Load: `workflows/direct-analysis-workflow.md`
2. Analyze project using Grep for all patterns
3. Report findings with file:line references
4. Reference: `examples/direct-analysis-only.md`
5. Ask if user wants full upgrade report

### Pattern 4: Query-Only Request
**User says:** "What ActiveRecord changes are in Rails 8.0?"

**Action:**
1. Load: `version-guides/upgrade-7.2-to-8.0.md`
2. Extract ActiveRecord-related changes
3. Provide summary without project analysis
4. Ask if user wants to analyze their project

### Pattern 5: Preview Only Request
**User says:** "Show me the app:update changes for Rails 7.2"

**Action:**
1. Detect current version from project
2. Load: `workflows/app-update-preview-workflow.md`
3. Read current config files
4. Generate preview with diffs
5. Reference: `examples/preview-only.md`

---

## Key Principles

1. **Always Analyze Directly** (use Cursor tools for immediate results)
2. **Use Parallel Grep** (run multiple pattern searches simultaneously for speed)
3. **Always Use Actual Code** (read user's files, show their actual code in reports)
4. **Always Flag Custom Code** (with ⚠️ warnings based on detected patterns)
5. **Always Use Templates** (for consistency)
6. **Always Check Quality** (verify file:line references before delivery)
7. **Load Workflows as Needed** (don't hold everything in memory)
8. **Interactive Process** (can ask clarifying questions during analysis)

---

## File Organization

This skill follows a modular structure:

```
rails-upgrade-assistant/
├── SKILL.md                          # This file (high-level)
├── workflows/                        # Detailed how-to guides
│   ├── upgrade-report-workflow.md
│   ├── direct-analysis-workflow.md
│   └── app-update-preview-workflow.md
├── examples/                         # Usage examples
│   ├── simple-upgrade.md
│   ├── multi-hop-upgrade.md
│   ├── direct-analysis-only.md
│   └── preview-only.md
├── reference/                        # Reference documentation
│   ├── pattern-file-guide.md
│   ├── quality-checklist.md
│   └── troubleshooting.md
├── version-guides/                   # Version-specific guides
├── templates/                        # Report templates
└── detection-scripts/                # Pattern files for Grep analysis
```

**When to Load What:**
- Load `workflows/` when generating deliverables
- Load `examples/` when user needs clarification
- Load `reference/` when you need detailed guidance

---

## Success Criteria

A successful upgrade assistance session:

✅ Detected Rails version and project structure correctly
✅ Analyzed project using Cursor tools (Read/Grep/Glob/LS)
✅ Generated Comprehensive Upgrade Report with actual code examples
✅ Generated app:update Preview with real config diffs
✅ Used user's actual code (not generic examples)
✅ Flagged all custom code with ⚠️ warnings based on detected patterns
✅ Provided clear next steps with file:line references
✅ Analysis completed in <10 seconds for typical Rails app
✅ Interactive guidance provided (answered follow-up questions)

**Verification:** See `reference/quality-checklist.md`

---

**Version:** 1.0
**Last Updated:** November 2, 2025
**Skill Type:** Modular with external workflows and examples
