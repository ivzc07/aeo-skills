# AEO Installation and Testing Guide

Complete guide for installing, testing, and publishing AEO skills.

## Prerequisites

- **Required:**
  - [ ] PAI (Personal AI Infrastructure) installed
  - [ ] Node.js 18+ and npm
  - [ ] Git
  - [ ] GitHub account

- **Optional but Recommended:**
  - [ ] Ralph execution loop
  - [ ] jq (for JSON processing)

## Installation

### Step 1: Verify PAI Installation

```bash
# Check if PAI is installed
ls ~/.claude/

# Should see:
# MEMORY/
# hooks/
# skills/
# CORE/
# Commands/
# etc.
```

**If PAI is not installed:**
```bash
# Clone PAI
git clone https://github.com/danielmiessler/Personal_AI_Infrastructure.git ~/.claude
cd ~/.claude

# Run installation
bun run install.ts --setup
```

### Step 2: Install AEO Skills

```bash
# Navigate to your project
cd /path/to/your/project

# Option 1: Install from GitHub (when published)
npx skills add laurence/aeo-skills

# Option 2: Install locally (during development)
cd /path/to/aeo-skills
npx skills link .
```

**Verify installation:**
```bash
# Check skills are available
ls skills/

# Should see:
# aeo-core/
# aeo-spec-validator/
# aeo-qa-agent/
# aeo-failure-patterns/
# aeo-escalation/
# aeo-cost-governor/
# aeo-architecture/
```

### Step 3: Configure Optional Components

**Enable cost tracking (optional):**
```bash
# Create budget config
mkdir -p ~/.claude/USER
cat > ~/.claude/USER/aeo-budget.json << 'EOF'
{
  "daily_budget_usd": 10.00,
  "per_task_budget_usd": 2.00,
  "alert_threshold_percent": 80,
  "hard_limit_percent": 100,
  "enable_tracking": true
}
EOF
```

**Configure architecture layers (optional):**
```bash
# Create layer config
cat > ~/.claude/USER/aeo-layers.yaml << 'EOF'
layers:
  - name: "presentation"
    path: "src/components/"
    may_import: ["domain", "application"]
    may_not_import: ["infrastructure"]

  - name: "domain"
    path: "src/domain/"
    may_import: []
    may_not_import: ["presentation", "application", "infrastructure"]

  - name: "application"
    path: "src/services/"
    may_import: ["domain"]
    may_not_import: ["presentation", "infrastructure"]

  - name: "infrastructure"
    path: "src/infrastructure/"
    may_import: ["domain"]
    may_not_import: ["presentation", "application"]
EOF
```

## Local Testing

### Test 1: Skill Activation

```bash
# Start Claude Code in your project
cd /path/to/your/project
claude

# Activate AEO
/aeo

# Should see:
# "AEO Core - Confidence Engine loaded"
# "Ready for task input"
```

**Expected behavior:**
- AEO loads aeo-core skill
- Other skills available on demand
- No errors in activation

### Test 2: Spec Validation

```
User: /aeo
User: Implement user login

AEO should respond with spec validation result...
```

**Expected output:**
```
AEO-Spec-Validator: Analyzing task...
                   Spec Score: 35/100 - UNACCEPTABLE

                   Missing:
                   • No acceptance criteria
                   • No tech stack specified
                   • No dependencies listed
```

### Test 3: Confidence Calculation

```
User: /aeo
User: Add console.log to debug the login function

AEO should calculate confidence...
```

**Expected output:**
```
AEO-Spec-Validator: Spec Score: 72/100 - MINOR GAPS
AEO-Core: Confidence: 0.65 - BLOCKING

          ⚠️ CONFIDENCE BELOW THRESHOLD
          (Escalation should present options)
```

### Test 4: Autonomous Execution

```
User: /aeo
User: Add comment to the top of README.md explaining what this project does

AEO should:
- Validate spec (should be high score)
- Calculate confidence (should be high)
- Execute autonomously
- Run QA review
- Commit changes
```

**Expected behavior:**
- No escalation needed
- Task completes autonomously
- One auto-fix applied (removing console.log if present)
- Commit successful

### Test 5: Security Veto

```
# Create test file with vulnerability
cat > test-security.js << 'EOF'
const userId = req.params.id
const query = `SELECT * FROM users WHERE id = ${userId}`
EOF

# In Claude Code:
User: /aeo
User: Add test-security.js to the project

AEO should:
- Execute task
- QA should detect SQL injection
- Veto the commit
- Present fix
- Apply fix
- Re-review
- Commit safe version
```

**Expected behavior:**
- QA veto triggered
- Auto-fix applied (parameterized query)
- Commit only after fix verified

## Validation Checklist

### Core Skills Validation

- [ ] **aeo-core**: Loads and calculates confidence
- [ ] **aeo-spec-validator**: Scores task specs (0-100)
- [ ] **aeo-qa-agent**: Reviews code and vetoes security issues
- [ ] **aeo-failure-patterns**: Recognizes and auto-fixes errors
- [ ] **aeo-escalation**: Presents options when confidence low
- [ ] **aeo-cost-governor**: Tracks token usage (optional)
- [ ] **aeo-architecture**: Detects circular deps (optional)

### Integration Validation

- [ ] **PAI memory**: Signals written to `~/.claude/MEMORY/`
- [ ] **Hooks**: SessionStart/Stop hooks execute (if configured)
- [ ] **Ralph**: Can execute from Ralph PRD (if installed)

### Format Validation

```bash
# Check all skills have required frontmatter
for skill in skills/*/SKILL.md; do
  echo "Checking $skill..."
  grep -q "^name:" "$skill" && echo "  ✅ Name found"
  grep -q "^description:" "$skill" && echo "  ✅ Description found"
done
```

**Expected output:**
```
Checking skills/aeo-core/SKILL.md
  ✅ Name found
  ✅ Description found
Checking skills/aeo-spec-validator/SKILL.md
  ✅ Name found
  ✅ Description found
[... all 7 skills ...]
```

## Troubleshooting

### Issue: Skills not activating

**Symptoms:**
- `/aeo` command does nothing
- Skills not loading

**Solutions:**

1. **Check skills are installed:**
```bash
ls skills/aeo-*/SKILL.md

# If missing, reinstall
npx skills link .  # If testing locally
```

2. **Check skill format:**
```bash
# Verify frontmatter
head -5 skills/aeo-core/SKILL.md

# Should see:
# ---
# name: aeo-core
# description: ...
# ---
```

3. **Check Claude Code config:**
```bash
# Verify skills directory configured
cat CLAUDE.md | grep skills
```

### Issue: Memory files not being created

**Symptoms:**
- No files in `~/.claude/MEMORY/`
- Signals not being recorded

**Solutions:**

1. **Create MEMORY directory:**
```bash
mkdir -p ~/.claude/MEMORY
touch ~/.claude/MEMORY/aeo-signals.jsonl
```

2. **Check permissions:**
```bash
ls -la ~/.claude/MEMORY/

# Should be writable
chmod 644 ~/.claude/MEMORY/aeo-*.jsonl
```

3. **Test write:**
```bash
echo '{"test":"data"}' >> ~/.claude/MEMORY/aeo-signals.jsonl
cat ~/.claude/MEMORY/aeo-signals.jsonl
```

### Issue: Cost tracking not working

**Symptoms:**
- No budget warnings
- Costs not being tracked

**Solutions:**

1. **Check budget config:**
```bash
cat ~/.claude/USER/aeo-budget.json

# Should have "enable_tracking": true
```

2. **Verify jq is installed:**
```bash
which jq
# If missing:
# macOS: brew install jq
# Linux: sudo apt-get install jq
```

### Issue: QA too strict

**Symptoms:**
- QA blocking on minor issues
- Too many auto-fixes

**Solutions:**

1. **Adjust QA strictness:** Edit `skills/aeo-qa-agent/SKILL.md` to relax rules
2. **Add exclusions:** Create `~/.claude/USER/aeo-exclusions.json`
3. **Provide better specs:** More detailed specs reduce QA issues

## Pre-Publication Checklist

### Code Quality

- [ ] All 7 skills have proper frontmatter (name, description)
- [ ] All skills pass markdown validation
- [ ] No broken skill references (skills reference each other correctly)
- [ ] Documentation consistent across skills

### Documentation

- [ ] README.md is clear and comprehensive
- [ ] AGENTS.md provides agent guidelines
- [ ] CLAUDE.md documents Claude-specific config
- [ ] Integration docs complete (PAI, Ralph)
- [ ] Examples cover all major scenarios

### Testing

- [ ] Local testing passes all validation checks
- [ ] Test scenarios execute successfully
- [ ] Memory files created and written correctly
- [ ] Hooks integrate properly with PAI

### Repository

- [ ] `.github/workflows/skill-validation.yml` is present
- [ ] All committed files pass validation
- [ ] Repository is clean (no unnecessary files)
- [ ] Commit messages follow conventional format

## Publishing

### Step 1: Prepare Repository

```bash
# Ensure you're on main branch
git checkout main

# Pull latest changes
git pull origin main

# Verify all skills present
ls skills/aeo-*/SKILL.md
```

### Step 2: Update Version (if needed)

```bash
# Update version in README.md if making a release
# For now, using development version
```

### Step 3: Commit Any Final Changes

```bash
git add .
git commit -m "chore: prepare for publication

- All 7 skills implemented and tested
- Integration documentation complete
- Examples cover all major scenarios
- Validation workflows in place"
```

### Step 4: Push to GitHub

```bash
# Push to main branch
git push origin main
```

### Step 5: Verify on skills.sh

```bash
# Visit https://skills.sh
# Search for "aeo-skills" or your username
# Verify skills appear in directory

# Test installation from GitHub
npx skills add YOUR_USERNAME/aeo-skills
```

### Step 6: Post-Publication Testing

```bash
# In a fresh project:
npx skills add YOUR_USERNAME/aeo-skills

# Test activation
/aeo

# Run simple task
User: /aeo
User: Add a comment to README.md

# Should work exactly like local testing
```

## Support

### Getting Help

- **Issues:** Report at https://github.com/YOUR_USERNAME/aeo-skills/issues
- **Questions:** Check docs/ directory or PAI community
- **PAI Help:** Check `~/.claude/skills/CORE/SKILL.md`

### Common Installation Errors

**Error: "Skills not found"**
```bash
# Solution: Verify installation path
pwd
ls skills/

# Should be in project root with skills/ subdirectory
```

**Error: "Permission denied writing to memory"**
```bash
# Solution: Fix permissions
mkdir -p ~/.claude/MEMORY
chmod 755 ~/.claude/MEMORY
```

**Error: "jq command not found"**
```bash
# Solution: Install jq
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# Or use Node.js JSON processing instead
```

## Next Steps

After successful installation and testing:

1. **Start with simple tasks** - Build confidence with autonomous execution
2. **Review memory signals** - See how AEO learns over time
3. **Adjust thresholds** - Tune confidence thresholds based on your preferences
4. **Add custom patterns** - Teach AEO your project-specific error patterns
5. **Configure layers** - Set up architecture protection for your codebase
6. **Enable cost tracking** - If cost-conscious, enable budget tracking

## Advanced Configuration

### Custom Confidence Thresholds

Edit `skills/aeo-core/SKILL.md`:

```markdown
## Decision Thresholds

### **≥ 0.90: AUTONOMOUS**  # Raised from 0.85
### **≥ 0.75: ADVISORY**  # Raised from 0.70
### **≥ 0.60: BLOCKING**   # Raised from 0.50
```

### Custom Spec Thresholds

Edit `skills/eo-spec-validator/SKILL.md`:

```markdown
### **≥ 85-100: PROCEED**  # Raised from 80
### **≥ 65-79: MINOR GAPS**  # Raised from 60
```

### Add Project-Specific Patterns

Edit `~/.claude/MEMORY/aeo-failure-patterns.json`:

```bash
jq '.project_patterns += [{
  "error_signature": "Your specific error",
  "context": {"language": "typescript"},
  "fix": "Steps to fix",
  "confidence": 0.85,
  "success_count": 0,
  "failure_count": 0,
  "last_seen": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
}]' ~/.claude/MEMORY/aeo-failure-patterns.json > /tmp/patterns.json
mv /tmp/patterns.json ~/.claude/MEMORY/aeo-failure-patterns.json
```
