# Detection Scripts Directory

## Overview

This directory contains breaking change patterns for Rails upgrades. These patterns are used for **direct, real-time analysis** using Cursor's native Grep tool.

---

## Directory Structure

```
detection-scripts/
├── patterns/                          # Breaking change patterns (YAML)
│   ├── rails-72-patterns.yml         # Rails 7.2 patterns
│   ├── rails-80-patterns.yml         # Rails 8.0 patterns
│   └── rails-81-patterns.yml         # Rails 8.1 patterns
├── templates/                         # Optional templates
│   └── detection-script-template.sh  # (Available if needed)
└── README.md                          # This file
```

---

## How Patterns Are Used

### Direct Analysis with Cursor Tools

The skill now performs **direct, real-time analysis** using Cursor's native tools:

1. **Load Pattern File**
   ```
   Read("detection-scripts/patterns/rails-81-patterns.yml")
   ```

2. **Convert Patterns to Grep Queries**
   ```yaml
   # YAML Pattern:
   - name: "cache_classes deprecation"
     file_pattern: "config/**/*.rb"
     search_pattern: "cache_classes"
     priority: "HIGH"

   # Converts to:
   Grep("cache_classes", glob: "config/**/*.rb", -C: 2)
   ```

3. **Execute in Parallel**
   ```
   Run ALL Grep operations simultaneously for 5-10x speed boost!

   Grep("cache_classes", glob: "config/**/*.rb", -C: 2)
   Grep("force_ssl", glob: "config/**/*.rb", -C: 2)
   Grep("show_exceptions", glob: "config/**/*.rb", -C: 2)
   ... (all patterns run in parallel)
   ```

4. **Return Results Immediately**
   ```
   Results include:
   - File path
   - Line number
   - Matched text
   - Context (lines before/after)
   - Pattern metadata (priority, component)

   Total time: < 10 seconds for typical Rails app
   ```

### Benefits of Direct Analysis

✅ **Fast** - Sub-second pattern detection with parallel Grep
✅ **Automatic** - Analysis happens immediately on request
✅ **Real-time** - Instant results with full code context
✅ **Interactive** - Agent can ask clarifying questions during analysis
✅ **Parallel** - All patterns searched simultaneously
✅ **Comprehensive** - Actual code from project files with file:line references
✅ **Offline-capable** - No external dependencies required

---

## Pattern File Format

### YAML Structure

```yaml
version: "8.1"
description: "Breaking change patterns for Rails 8.0 → 8.1"

breaking_changes:
  high_priority:
    - name: "Pattern name"
      pattern: 'regex or literal string'
      exclude: "exclusion pattern (optional)"
      search_paths:
        - "path/to/search"
      component: "Rails component"
      file_types:
        - rb
      description: "What changed"
      fix: "How to fix it"

  medium_priority:
    - name: "Another pattern"
      pattern: 'search pattern'
      search_paths:
        - "another/path"
      ...

  low_priority:
    - name: "Optional improvement"
      ...
```

### Key Fields

- **name**: Human-readable pattern name
- **pattern**: Regex or literal string to search for
- **search_paths**: Where to search (converted to glob patterns)
- **priority**: HIGH, MEDIUM, or LOW
- **component**: Rails component (ActionCable, ActiveRecord, etc.)
- **description**: What the breaking change is
- **fix**: How to remediate it

---

## Workflow Integration

### Primary Workflow: Direct Analysis

See `workflows/direct-analysis-workflow.md` for the complete workflow.

**Quick Summary:**
1. User requests upgrade analysis
2. Agent loads pattern file for target version
3. Agent converts all patterns to parallel Grep queries
4. Agent executes searches (< 5 seconds)
5. Agent aggregates findings with context
6. Agent generates comprehensive report with actual code

**Performance:** Sub-second pattern loading + ~5 seconds analysis = **< 10 seconds total**

### Optional: Custom Scripts

The YAML patterns can be used as a reference for creating custom detection scripts if needed for specific workflows.

---

## Adding New Patterns

When Rails releases new versions:

### 1. Create New Pattern File

```bash
cp detection-scripts/patterns/rails-81-patterns.yml \
   detection-scripts/patterns/rails-82-patterns.yml
```

### 2. Update Version Information

```yaml
version: "8.2"
description: "Breaking change patterns for Rails 8.1 → 8.2"
```

### 3. Add New Breaking Changes

Research official Rails CHANGELOGs and add patterns:

```yaml
breaking_changes:
  high_priority:
    - name: "New breaking change"
      pattern: 'new_deprecated_method'
      search_paths:
        - "app/**/*.rb"
      component: "ActiveRecord"
      description: "Method removed in Rails 8.2"
      fix: "Use new_method instead"
```

### 4. Test Patterns

Test that Grep queries work correctly:

```
Grep("new_deprecated_method", glob: "app/**/*.rb", -C: 2)
```

### 5. Update SKILL.md

Add reference to new pattern file in available resources section.

---

## Pattern Design Guidelines

### Good Patterns

✅ **Specific enough** to avoid false positives
```yaml
pattern: 'config\.cache_classes\s*='
```

✅ **Include context** about what changed
```yaml
description: "cache_classes removed in Rails 7.1, replaced with enable_reloading (inverted logic)"
```

✅ **Provide clear fix**
```yaml
fix: "Replace config.cache_classes = true with config.enable_reloading = false"
```

### Bad Patterns

❌ **Too broad** - matches everything
```yaml
pattern: 'config\.'  # Matches ALL config settings
```

❌ **No context** - user doesn't know what to do
```yaml
description: "This changed"
fix: "Update it"
```

❌ **Wrong search paths** - won't find matches
```yaml
search_paths:
  - "models/"  # Should be "app/models/"
```


---

## Performance Benchmarks

| Project Size | Pattern Count | Analysis Time | Total Time |
|--------------|---------------|---------------|------------|
| Small (50 files) | 10 patterns | 0.5s | < 5s |
| Medium (200 files) | 15 patterns | 1.5s | < 10s |
| Large (500 files) | 20 patterns | 3.0s | < 15s |

**Performance Advantages:**
- Parallel Grep execution
- Local file access (no network overhead)
- Immediate results
- Interactive clarification capability

---

## Common Use Cases

### Use Case 1: Full Upgrade Analysis
```
User: "Upgrade my Rails app to 8.1"

Agent:
1. Read("detection-scripts/patterns/rails-81-patterns.yml")
2. Convert to Grep queries
3. Execute parallel analysis
4. Generate comprehensive report
```

### Use Case 2: Breaking Changes Only
```
User: "Find breaking changes for Rails 8.0"

Agent:
1. Load patterns for Rails 8.0
2. Run direct analysis
3. Show summary of findings
4. Offer full report if needed
```

### Use Case 3: Specific Component Analysis
```
User: "Check my models for Rails 8.1 ActiveRecord changes"

Agent:
1. Filter patterns to ActiveRecord only
2. Grep("pattern", glob: "app/models/**/*.rb")
3. Show model-specific findings
```

---

## Troubleshooting

### No Patterns Found
**Issue:** Pattern file not loading

**Solution:**
- Verify file exists: `detection-scripts/patterns/rails-{VERSION}-patterns.yml`
- Check YAML syntax is valid
- Ensure version number is correct (72, 80, 81)

### No Matches Detected
**Issue:** Grep returns no results (but should find matches)

**Solution:**
- Test pattern manually: `Grep("pattern", glob: "**/*.rb")`
- Verify glob pattern is correct
- Check search_paths in YAML
- Try broader pattern first, then narrow down

### Too Many False Positives
**Issue:** Pattern matches non-breaking code

**Solution:**
- Make pattern more specific
- Add negative lookahead in regex
- Narrow search_paths
- Add exclude pattern in YAML

---

## Future Enhancements

Potential improvements for pattern files:

1. **Auto-detection of Rails version** from patterns
2. **Pattern testing suite** to validate accuracy
3. **Pattern statistics** (how often each matches)
4. **User-contributed patterns** via pull requests
5. **AI-assisted pattern generation** from CHANGELOGs

---

## Related Files

- **Workflows:** `workflows/direct-analysis-workflow.md`
- **Templates:** `templates/upgrade-report-template.md`
- **Examples:** `examples/direct-analysis-only.md`
- **Main Skill:** `SKILL.md`

---

**Last Updated:** January 7, 2026
**Pattern Format Version:** 2.0
**Approach:** Direct analysis using Cursor native tools
