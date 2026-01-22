# PAI Integration for AEO Skills

This document describes how AEO skills integrate with PAI (Personal AI Infrastructure) memory and hooks systems.

## Memory Files

AEO stores data in `$PAI_DIR/MEMORY/` (usually `~/.claude/MEMORY/`):

### Active Memory Files

| File | Purpose | Format | Access Pattern |
|------|---------|--------|----------------|
| `aeo-signals.jsonl` | Task outcomes for confidence learning | JSONL (newline-delimited JSON) | Append after task completion |
| `aeo-escalations.jsonl` | Escalation decisions and patterns | JSONL | Append after escalation |
| `aeo-failure-patterns.json` | Learned error patterns | JSON array | Read/write for pattern matching |
| `aeo-costs.jsonl` | Token usage tracking | JSONL | Append after each operation |
| `aeo-task-cost.json` | Current task cost tracking | JSON | Overwrite during task |

## File Formats

### aeo-signals.jsonl

```json
{"timestamp":"2026-01-22T10:30:00Z","task_id":"task-123","task_description":"Add user authentication","predicted_confidence":0.85,"actual_difficulty":"medium","success":true,"adjustment":0.00}
{"timestamp":"2026-01-22T11:15:00Z","task_id":"task-124","task_description":"Fix login bug","predicted_confidence":0.92,"actual_difficulty":"easy","success":true,"adjustment":0.05}
{"timestamp":"2026-01-22T12:00:00Z","task_id":"task-125","task_description":"Refactor database","predicted_confidence":0.65,"actual_difficulty":"hard","success":true,"adjustment":-0.05}
```

**Reading signals:**
```bash
# Get last 100 signals for rolling confidence
tail -100 ~/.claude/MEMORY/aeo-signals.jsonl | \
  jq -s 'map(.adjustment) | add / length'
# Output: 0.0167 (average adjustment)

# Get successful tasks only
jq -s 'map(select(.success == true))' ~/.claude/MEMORY/aeo-signals.jsonl

# Count by difficulty
jq -s 'group_by(.actual_difficulty) | map({difficulty: .[0].actual_difficulty, count: length})' \
  ~/.claude/MEMORY/aeo-signals.jsonl
```

### aeo-escalations.jsonl

```json
{"timestamp":"2026-01-22T10:30:00Z","task_id":"task-123","escalation_type":"low_confidence","confidence_before":0.62,"option_presented":[1,2,3],"option_chosen":2,"human_input":"User provided email service details","resolution":"Improved spec, confidence increased to 0.85","success":true,"learning":"Context details improve confidence by ~0.20"}
{"timestamp":"2026-01-22T11:15:00Z","task_id":"task-124","escalation_type":"qa_veto","confidence_before":0.78,"option_presented":[1,2,3],"option_chosen":1,"human_input":"Fix immediately","resolution":"Security vulnerability fixed, re-review passed","success":true,"learning":"Security vetos should auto-fix immediately"}
```

**Analyzing escalations:**
```bash
# Most common escalation types
jq -s 'group_by(.escalation_type) | map({type: .[0].escalation_type, count: length}) | sort_by(-.count)' \
  ~/.claude/MEMORY/aeo-escalations.jsonl

# Success rate by escalation type
jq -s 'group_by(.escalation_type) | map({
    type: .[0].escalation_type,
    success_rate: (map(select(.success == true)) | length / length * 100)
  })' ~/.claude/MEMORY/aeo-escalations.jsonl
```

### aeo-failure-patterns.json

```json
{
  "project_patterns": [
    {
      "error_signature": "TypeError: Cannot read properties of undefined (reading 'data')",
      "context": {"language": "javascript", "framework": "react"},
      "fix": "Add optional chaining: response?.data instead of response.data",
      "confidence": 0.85,
      "success_count": 5,
      "failure_count": 0,
      "last_seen": "2026-01-22T10:00:00Z"
    },
    {
      "error_signature": "ENOSPC: no space left on device",
      "context": {"language": "bash", "operation": "docker build"},
      "fix": "Run docker system prune -a to clear unused images and containers",
      "confidence": 0.90,
      "success_count": 3,
      "failure_count": 0,
      "last_seen": "2026-01-21T15:30:00Z"
    }
  ]
}
```

**Reading and matching patterns:**
```bash
# Find patterns matching error message
jq -r '.project_patterns[] | select(.error_signature | contains("undefined"))' \
  ~/.claude/MEMORY/aeo-failure-patterns.json

# Get high-confidence patterns (≥0.80)
jq -r '.project_patterns[] | select(.confidence >= 0.80)' \
  ~/.claude/MEMORY/aeo-failure-patterns.json
```

**Updating pattern confidence:**
```bash
# Increment success count
jq '(.project_patterns[] | select(.error_signature == "specific error")).success_count += 1' \
  ~/.claude/MEMORY/aeo-failure-patterns.json > /tmp/patterns.json
mv /tmp/patterns.json ~/.claude/MEMORY/aeo-failure-patterns.json

# Add new pattern
jq '.project_patterns += [{
  "error_signature": "new error message",
  "context": {"language": "typescript"},
  "fix": "steps to fix",
  "confidence": 0.50,
  "success_count": 0,
  "failure_count": 0,
  "last_seen": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
}]' ~/.claude/MEMORY/aeo-failure-patterns.json > /tmp/patterns.json
mv /tmp/patterns.json ~/.claude/MEMORY/aeo-failure-patterns.json
```

### aeo-costs.jsonl

```json
{"timestamp":"2026-01-22T10:30:00Z","date":"2026-01-22","task_id":"task-123","model":"claude-sonnet-4.5","input_tokens":1250,"output_tokens":850,"cost_usd":0.15}
{"timestamp":"2026-01-22T11:00:00Z","date":"2026-01-22","task_id":"task-124","model":"claude-haiku-4","input_tokens":500,"output_tokens":300,"cost_usd":0.01}
```

**Daily cost summary:**
```bash
# Today's total
today=$(date -u +%Y-%m-%d)
jq -s "select(.date == \"$today\") | map(.cost_usd) | add" ~/.claude/MEMORY/aeo-costs.jsonl

# By model for today
jq -s "select(.date == \"$today\") | group_by(.model) | map({
    model: .[0].model,
    cost: map(.cost_usd) | add,
    tasks: length
  })" ~/.claude/MEMORY/aeo-costs.jsonl
```

### aeo-task-cost.json

```json
{
  "task_id": "task-123",
  "task_description": "Add user authentication",
  "start_time": "2026-01-22T10:30:00Z",
  "model": "claude-sonnet-4.5",
  "budget_usd": 2.00,
  "usage": {
    "input_tokens": 1250,
    "output_tokens": 850,
    "cost_usd": 0.15
  }
}
```

**Updating task cost:**
```bash
# Add cost to current task
jq '.usage.cost_usd += 0.25' ~/.claude/MEMORY/aeo-task-cost.json > /tmp/task-cost.json
mv /tmp/task-cost.json ~/.claude/MEMORY/aeo-task-cost.json
```

## Hook Integration

### SessionStart Hook

```javascript
// ~/.claude/hooks/SessionStart/aeo-session.js

import { readFileSync } from 'fs';
import { join } from 'path';

export async function execute({ pai_dir }) {
  // Read recent signals to calibrate confidence
  const signalsFile = join(pai_dir, 'MEMORY/aeo-signals.jsonl');

  try {
    const signals = readFileSync(signalsFile, 'utf8')
      .split('\n')
      .filter(line => line.trim())
      .slice(-100); // Last 100 signals

    const adjustments = signals
      .map(line => {
        try {
          return JSON.parse(line).adjustment;
        } catch {
          return 0;
        }
      });

    const avgAdjustment = adjustments.reduce((a, b) => a + b, 0) / adjustments.length;

    console.log(`AEO: Loaded ${signals.length} signals, avg adjustment: ${avgAdjustment.toFixed(4)}`);

    return {
      confidence_offset: avgAdjustment,
      signals_loaded: signals.length
    };
  } catch (error) {
    // First time using AEO - no signals yet
    console.log('AEO: No previous signals found (first run)');
    return {
      confidence_offset: 0,
      signals_loaded: 0
    };
  }
}
```

### PreToolUse Hook

```javascript
// ~/.claude/hooks/PreToolUse/aeo-cost-tracking.js

import { appendFileSync, readFileSync, writeFileSync } from 'fs';
import { join } from 'path';

// Pricing per 1M tokens (Jan 2025)
const PRICING = {
  'claude-opus-4': { input: 15.00, output: 75.00 },
  'claude-sonnet-4.5': { input: 3.00, output: 15.00 },
  'claude-haiku-4': { input: 0.80, output: 4.00 }
};

export async function execute({ tool_name, tool_input, model, pai_dir }) {
  // Skip if cost tracking disabled
  const budgetFile = join(pai_dir, 'USER/aeo-budget.json');
  try {
    const budget = JSON.parse(readFileSync(budgetFile, 'utf8'));
    if (!budget.enable_tracking) return;
  } catch {
    // Use defaults
  }

  // Estimate cost (simplified - actual costs vary)
  const inputSize = JSON.stringify(tool_input).length;
  const estimatedTokens = Math.ceil(inputSize / 4); // Rough estimate
  const modelKey = model.includes('opus') ? 'claude-opus-4' :
                    model.includes('haiku') ? 'claude-haiku-4' :
                    'claude-sonnet-4.5';

  const pricing = PRICING[modelKey];
  const estimatedCost = (estimatedTokens / 1_000_000) * (pricing.input + pricing.output) * 10; // Rough estimate

  // Update task cost
  const taskCostFile = join(pai_dir, 'MEMORY/aeo-task-cost.json');
  try {
    const taskCost = JSON.parse(readFileSync(taskCostFile, 'utf8'));
    taskCost.usage.cost_usd += estimatedCost;
    taskCost.usage.input_tokens += estimatedTokens;
    writeFileSync(taskCostFile, JSON.stringify(taskCost, null, 2));

    // Check if over budget
    if (taskCost.usage.cost_usd > taskCost.budget_usd) {
      console.warn(`AEO: Task budget exceeded (${taskCost.usage.cost_usd.toFixed(2)} of ${taskCost.budget_usd})`);
      // Would invoke aeo-escalation here
    }
  } catch (error) {
    // Task cost file doesn't exist yet
  }

  return { cost_tracked: true, estimated_cost: estimatedCost };
}
```

### SessionStop Hook

```javascript
// ~/.claude/hooks/SessionStop/aeo-session-summary.js

import { appendFileSync } from 'fs';
import { join } from 'path';

export async function execute({ pai_dir, session_id }) {
  // Write session summary
  const summary = {
    timestamp: new Date().toISOString(),
    session_id: session_id,
    type: 'aeo-session'
  };

  // This would be populated with actual session data
  // For now, just record that session ended

  console.log('AEO: Session ended, memory persisted');
  return { success: true };
}
```

## Configuration Files

### aeo-budget.json

Create at `$PAI_DIR/USER/aeo-budget.json`:

```json
{
  "daily_budget_usd": 10.00,
  "per_task_budget_usd": 2.00,
  "alert_threshold_percent": 80,
  "hard_limit_percent": 100,
  "enable_tracking": true
}
```

### aeo-layers.yaml

Create at `$PAI_DIR/USER/aeo-layers.yaml`:

```yaml
layers:
  - name: "presentation"
    path: "src/components/"
    may_import: ["domain", "application"]
    may_not_import: ["infrastructure", "presentation"]

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
```

## Memory Access Patterns

### Reading from Memory

```bash
# Read last 50 signals
tail -50 ~/.claude/MEMORY/aeo-signals.jsonl

# Find escalations by type
jq -s 'map(select(.escalation_type == "low_confidence"))' ~/.claude/MEMORY/aeo-escalations.jsonl

# Get high-confidence failure patterns
jq '.project_patterns[] | select(.confidence >= 0.80)' ~/.claude/MEMORY/aeo-failure-patterns.json

# Today's costs
today=$(date -u +%Y-%m-%d)
jq -s "select(.date == \"$today\")" ~/.claude/MEMORY/aeo-costs.jsonl
```

### Writing to Memory

```bash
# Append signal
echo '{"timestamp":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","task_id":"xxx","success":true,"adjustment":0.05}' >> ~/.claude/MEMORY/aeo-signals.jsonl

# Append escalation
echo '{"timestamp":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","escalation_type":"low_confidence","success":true}' >> ~/.claude/MEMORY/aeo-escalations.jsonl

# Append cost
echo '{"timestamp":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","date":"'$(date -u +%Y-%m-%d)'","cost_usd":0.15}' >> ~/.claude/MEMORY/aeo-costs.jsonl

# Update failure patterns (using jq)
jq '.project_patterns += [new_pattern]' ~/.claude/MEMORY/aeo-failure-patterns.json > /tmp/patterns.json
mv /tmp/patterns.json ~/.claude/MEMORY/aeo-failure-patterns.json
```

## Cleanup and Maintenance

### Archive Old Signals

```bash
# Keep last 1000 signals, archive older
head -n -1000 ~/.claude/MEMORY/aeo-signals.jsonl > ~/.claude/MEMORY/aeo-signals.archive.jsonl
tail -1000 ~/.claude/MEMORY/aeo-signals.jsonl > /tmp/signals.jsonl
mv /tmp/signals.jsonl ~/.claude/MEMORY/aeo-signals.jsonl
```

### Memory Size Monitoring

```bash
# Check memory file sizes
ls -lh ~/.claude/MEMORY/aeo-*.jsonl ~/.claude/MEMORY/aeo-*.json

# Count records
wc -l ~/.claude/MEMORY/aeo-signals.jsonl
wc -l ~/.claude/MEMORY/aeo-escalations.jsonl
wc -l ~/.claude/MEMORY/aeo-costs.jsonl
```

## Integration Checklist

- [ ] Memory directory exists: `$PAI_DIR/MEMORY/`
- [ ] Configuration files created:
  - [ ] `$PAI_DIR/USER/aeo-budget.json` (optional)
  - [ ] `$PAI_DIR/USER/aeo-layers.yaml` (optional)
- [ ] Hooks installed:
  - [ ] `SessionStart/aeo-session.js`
  - [ ] `PreToolUse/aeo-cost-tracking.js` (optional)
  - [ ] `SessionStop/aeo-session-summary.js`
- [ ] Memory files initialized (first run will create them)
- [ ] Permissions set correctly (memory files writable)

## Troubleshooting

### Memory files not created

**Issue:** AEO can't write to memory

**Solution:**
```bash
# Create MEMORY directory if missing
mkdir -p ~/.claude/MEMORY

# Check permissions
ls -la ~/.claude/MEMORY/

# Make sure files are writable
touch ~/.claude/MEMORY/aeo-signals.jsonl
chmod 644 ~/.claude/MEMORY/aeo-*.jsonl
```

### Hooks not executing

**Issue:** PAI hooks not running

**Solution:**
```bash
# Check hooks are in correct location
ls -la ~/.claude/hooks/SessionStart/
ls -la ~/.claude/hooks/PreToolUse/
ls -la ~/.claude/hooks/SessionStop/

# Verify hook format (must export execute())
cat ~/.claude/hooks/SessionStart/aeo-session.js
```

### jq not available

**Issue:** Can't process JSON files

**Solution:**
```bash
# Install jq
# macOS
brew install jq

# Linux
sudo apt-get install jq

# Or use Node.js JSON processing instead
node -e "console.log(JSON.parse(require('fs').readFileSync('file.json')))"
```
