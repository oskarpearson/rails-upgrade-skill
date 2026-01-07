# Direct Analysis Workflow

**Purpose:** Directly analyze Rails projects for breaking changes using Cursor's native tools

**When to use:** Every upgrade request for immediate, real-time analysis

**Key Features:**
- ⚡ Real-time analysis with immediate results
- 💬 Interactive clarification (agent can ask questions during analysis)
- 🔄 Parallel Grep execution (5-10x faster than sequential searches)
- 📊 Comprehensive findings with actual code context
- 🎯 Sub-second pattern detection for typical Rails apps

---

## Prerequisites

Before starting:
- User has requested upgrade analysis
- Target Rails version is known
- Project is accessible in Cursor workspace

---

## Step-by-Step Workflow

### Step 1: Detect Rails Version and Project Structure

Use Cursor's native tools to understand the project:

```
1. Read("Gemfile")
   Extract Rails version: Grep("gem ['\"]rails['\"]", path: "Gemfile")
   Result: gem 'rails', '~> 8.0.0'

2. Read("config/application.rb")
   Check: API-only mode, load_defaults version, custom configs

3. LS("/")
   Verify Rails structure: app/, config/, db/, lib/, etc.

4. Glob("app/models/**/*.rb")
   Count models for complexity estimate
```

**Store:**
- `current_version`: e.g., "8.0.0"
- `target_version`: e.g., "8.1.1"
- `project_type`: "Full Stack" or "API-only"
- `model_count`: Number of models (for time estimate)

---

### Step 2: Load Breaking Change Patterns

Read the YAML pattern file for target version:

```
Read("detection-scripts/patterns/rails-{VERSION}-patterns.yml")

Example for Rails 8.1:
- patterns:
    - name: "SSL Configuration Change"
      file_pattern: "config/environments/production.rb"
      search_pattern: "force_ssl"
      priority: "HIGH"
      component: "ActionDispatch"
```

**Parse YAML to extract:**
- Pattern name
- File glob pattern
- Search regex pattern
- Priority level
- Component name
- Remediation guidance

---

### Step 3: Convert Patterns to Grep Queries

For each pattern in YAML, create a Grep operation:

```ruby
# Pattern from YAML:
{
  name: "cache_classes deprecation",
  file_pattern: "config/**/*.rb",
  search_pattern: "cache_classes",
  priority: "HIGH"
}

# Convert to Grep:
Grep("cache_classes", glob: "config/**/*.rb", -C: 3)

# Result:
config/environments/production.rb:10:  # Configure cache classes
config/environments/production.rb:11:  config.cache_classes = true
config/environments/production.rb:12:  # Comment explaining...
```

---

### Step 4: Execute Parallel Analysis

**Key Advantage:** Run ALL Grep operations in parallel for maximum speed!

```
# Example: Rails 7.0 → 7.1 analysis (5 patterns)
Run simultaneously:
1. Grep("cache_classes", glob: "config/**/*.rb", -C: 2)
2. Grep("force_ssl", glob: "config/**/*.rb", -C: 2)
3. Grep("lib/.*class", glob: "config/application.rb", -C: 2)
4. Grep("show_exceptions.*=.*(true|false)", glob: "config/**/*.rb", -C: 2)
5. Grep("database.*storage", glob: "config/database.yml", -C: 2)

Performance: ~50ms total (vs ~500ms sequential) = 10x faster!
```

**Collect Results:**
- File path
- Line number
- Matched text
- Context (lines before and after)
- Pattern metadata (name, priority, component)

---

### Step 5: Read Affected Files for Full Context

For each file with matches, read the full file:

```
If cache_classes found in config/environments/production.rb:

Read("config/environments/production.rb")

Extract:
- Full configuration context
- Custom middleware or settings
- Related configurations
- User's actual code to include in report
```

---

### Step 6: Detect Custom Code Patterns

Run additional Grep searches for common customizations:

```
Parallel Grep operations:
1. Grep("middleware.use|middleware.insert", glob: "config/**/*.rb", -C: 2)
   → Custom middleware

2. Grep("Redis.new|Redis.current", glob: "config/**/*.rb", -C: 2)
   → Manual Redis configuration

3. Grep("autoload_paths|eager_load_paths", glob: "config/**/*.rb", -C: 2)
   → Custom load paths

4. Grep("Sprockets|assets.compile", glob: "config/**/*.rb", -C: 2)
   → Asset pipeline customizations

5. Grep("config\\..*=.*ENV", glob: "config/**/*.rb", -C: 2)
   → Environment-based configurations
```

**Flag each finding with ⚠️ warning for user review**

---

### Step 7: Aggregate and Organize Findings

Organize all findings by:

**1. Priority Level:**
- 🔴 HIGH: Will break application (e.g., cache_classes removed)
- 🟡 MEDIUM: Should address (e.g., deprecation warnings)
- 🟢 LOW: Optional improvements (e.g., new features available)

**2. Component:**
- ActionCable
- ActionController
- ActionMailer
- ActionView
- ActiveRecord
- ActiveStorage
- ActiveSupport
- Railties
- Configuration

**3. File Location:**
- Group by file for easier navigation
- Include file:line:context for each finding

---

### Step 8: Generate Findings Summary

Create structured summary:

```markdown
# Rails {FROM} → {TO} Upgrade Analysis

## Summary
- 🔴 HIGH Priority: 3 issues found
- 🟡 MEDIUM Priority: 5 issues found
- 🟢 LOW Priority: 2 issues found
- ⚠️ Custom Code: 4 warnings

## Breaking Changes Detected

### 🔴 HIGH Priority

#### 1. cache_classes Configuration Deprecated
**File:** config/environments/production.rb:11
**Component:** Railties

**Your Code:**
\```ruby
config.cache_classes = true
\```

**Issue:**
The cache_classes configuration has been replaced with enable_reloading
(inverted logic) in Rails 7.1.

**Action Required:**
Replace with: config.enable_reloading = false

---

(Continue for all findings...)

## Custom Code Warnings

### ⚠️ Custom Middleware Detected
**File:** config/application.rb:25

**Your Code:**
\```ruby
config.middleware.use CustomAuthMiddleware
\```

**Review Needed:**
Custom middleware may need updates for Rails {TARGET_VERSION} changes.
Test thoroughly after upgrade.

---

(Continue for all custom code...)

## Files Affected

- config/environments/production.rb (3 changes)
- config/application.rb (2 changes)
- Gemfile (1 change)
- config/database.yml (1 change)

Total: 7 changes across 4 files
```

---

### Step 9: Provide Actionable Next Steps

Include clear guidance:

```markdown
## 📋 Immediate Actions

1. **Review HIGH Priority Issues (3 found)**
   - These will break your application
   - Must be fixed before deploying to production

2. **Address Custom Code Warnings (4 found)**
   - Review each customization
   - Test compatibility with new Rails version

3. **Update Configuration**
   - See detailed instructions in full upgrade report
   - Apply changes incrementally
   - Test after each change

## 🚀 Ready for Full Upgrade Report?

I can now generate a comprehensive upgrade report with:
- Detailed OLD vs NEW code examples
- Step-by-step migration guide
- Testing checklist
- Rollback procedures

Would you like me to generate the full report?
```

---

## Performance Benchmarks

**Typical Performance (Rails app with ~50 files):**

| Operation | Time | Notes |
|-----------|------|-------|
| Detect version | ~20ms | Read Gemfile + parse |
| Load patterns | ~10ms | Read YAML file |
| Parallel Grep (10 patterns) | ~100ms | All patterns simultaneously |
| Read affected files (5 files) | ~30ms | Full context |
| Custom code detection (5 patterns) | ~50ms | Parallel Grep |
| **Total Analysis** | **~210ms** | **Sub-second results!** |

**Why So Fast:**
- Local file access (no network calls)
- Parallel Grep operations
- Ripgrep optimization
- Direct analysis with immediate results

---

## Quality Checklist

Before delivering findings, verify:

- [ ] Rails version detected correctly
- [ ] All patterns from YAML executed
- [ ] File:line references accurate
- [ ] Code context included (before/after lines)
- [ ] Custom code warnings identified
- [ ] Findings organized by priority
- [ ] Actionable next steps provided
- [ ] Analysis completed in < 10 seconds

---

## Error Handling

**Common Issues and Solutions:**

### Issue: Gemfile not found
**Solution:**
```
Grep("rails", glob: "**/Gemfile", -i: true)
# Search recursively in case project has non-standard structure
```

### Issue: No matches found
**Solution:**
```
This is good! Either:
1. No breaking changes affect this project (rare)
2. Project already updated (check load_defaults version)

Inform user: "No breaking changes detected in your codebase!"
```

### Issue: Too many matches (100+)
**Solution:**
```
1. Group by file
2. Show summary with counts
3. Offer to focus on HIGH priority only
4. Ask: "Would you like me to focus on specific files or patterns?"
```

### Issue: Permission denied reading files
**Solution:**
```
Report to user: "Unable to read {file}. Please check file permissions."
Continue with remaining analysis.
```

---

## Interactive Features

**The agent can ask clarifying questions during analysis:**

**Example 1:**
```
Agent: "I found 15 uses of config.cache_classes across multiple files.
        Would you like me to analyze all of them, or focus on the main
        configuration files only?"

User: "Focus on config files"

Agent: [Refines analysis to config/**/*.rb only]
```

**Example 2:**
```
Agent: "I detected custom middleware in your application. Should I do
        a deeper analysis of your middleware stack for compatibility?"

User: "Yes, please"

Agent: [Runs additional Grep patterns for middleware-related issues]
```

**Example 3:**
```
Agent: "Your app appears to be API-only. Should I skip view-related
        breaking changes?"

User: "Yes, skip them"

Agent: [Filters out ActionView patterns]
```

---

## Integration with Full Report

After analysis, seamlessly transition to full report generation:

```
Analysis complete! I found:
- 3 HIGH priority breaking changes
- 5 MEDIUM priority issues
- 4 custom code warnings

I can now generate a comprehensive upgrade report with:
✓ Detailed code examples from YOUR project
✓ Step-by-step migration instructions
✓ Testing checklist
✓ Rollback procedures

Generate full report now? (yes/no)
```


---

## Advanced Features

### 1. Incremental Analysis

If project is large (500+ files):
```
1. Analyze config files first (fastest)
2. Show preliminary findings
3. Ask: "Continue with full codebase analysis?"
4. If yes: Analyze app/, lib/, etc.
```

### 2. Focused Analysis

User can request specific analysis:
```
"Check my models for ActiveRecord changes"
  ↓
Grep(activerecord_patterns, glob: "app/models/**/*.rb")

"Find Redis configuration issues"
  ↓
Grep(redis_patterns, glob: "config/**/*.rb", "lib/**/*.rb")
```

### 3. Diff Analysis

Compare two versions of same file:
```
1. Read current file: Read("config/application.rb")
2. Show what needs to change
3. Offer to apply changes: "Should I update this file?"
```

---

## Success Criteria

Analysis is successful when:

✅ Rails version detected accurately
✅ All breaking change patterns executed
✅ Real code examples from user's project
✅ File:line references for every finding
✅ Custom code warnings identified
✅ Analysis completed in < 10 seconds
✅ Clear next steps provided
✅ User can request full report

---

**Version:** 2.0 (Direct Analysis)
**Last Updated:** January 7, 2026
**Performance:** Sub-second analysis for typical Rails apps
