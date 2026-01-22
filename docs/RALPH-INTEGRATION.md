# Ralph Integration for AEO Skills

This document describes how AEO skills integrate with Ralph execution loop.

## Ralph Overview

Ralph is an execution loop that:
- Takes PRD (Product Requirement Document) as input
- Breaks down into stories and tasks
- Executes tasks with fresh context each iteration
- Uses `progress.txt` for persistence

**Repository:** https://github.com/snarktank/ralph

## Integration Points

### 1. Pre-Execution: Spec Validation

Ralph breaks down PRD into stories → AEO validates each story before execution:

```
Ralph Story:
├── Story: "Add user authentication with email verification"
└── Tasks:
    ├── Task 1: "Create auth service with JWT tokens"
    ├── Task 2: "Add email verification with Resend"
    └── Task 3: "Create login UI components"

AEO validates each task:
├── aeo-spec-validator scores task (0-100)
├── aeo-core calculates confidence (0-1)
├── If confidence ≥ 0.70 → Execute
└── If confidence < 0.70 → Escalate to human
```

**Example Integration:**

```javascript
// In Ralph's task executor
import { validateSpec } from 'aeo-spec-validator';
import { calculateConfidence } from 'aeo-core';

async function executeTask(task) {
  // Step 1: Validate spec
  const specScore = validateSpec(task.description);

  if (specScore < 40) {
    console.log('❌ Spec insufficient - AEO blocking execution');
    console.log('Score:', specScore, '/ 100');

    // AEO would invoke escalation here
    return { blocked: true, reason: 'insufficient_spec' };
  }

  // Step 2: Calculate confidence
  const confidence = calculateConfidence({
    task: task.description,
    specScore: specScore,
    context: getTaskContext()
  });

  console.log(`AEO Confidence: ${confidence.toFixed(2)}`);

  if (confidence < 0.70) {
    console.log('⚠️ Confidence below threshold - human approval required');

    // AEO escalation presents options
    const decision = await escalate({
      type: 'low_confidence',
      confidence: confidence,
      task: task
    });

    if (!decision.approved) {
      return { blocked: true, reason: 'confidence_too_low' };
    }
  }

  // Step 3: Execute task
  console.log('✅ Confidence acceptable - proceeding autonomously');
  return await executeTaskImplementation(task);
}
```

### 2. During Execution: On-Demand Escalation

Ralph executes tasks → AEO monitors for issues and escalates when needed:

```
Ralph Execution:
├── Implementing task...
├── Error occurs → AEO failure patterns tries auto-fix
├── Auto-fix fails → AEO escalates to human
└── Human provides guidance → Continue execution
```

**Example Integration:**

```javascript
// In Ralph's error handler
import { handleFailure } from 'aeo-failure-patterns';
import { escalate } from 'aeo-escalation';

async function executeWithErrorHandling(task) {
  try {
    return await implementTask(task);
  } catch (error) {
    console.log('Error during execution:', error.message);

    // AEO tries to auto-fix
    const fixResult = await handleFailure(error, {
      context: getExecutionContext()
    });

    if (fixResult.autoFixed) {
      console.log('✅ AEO auto-fixed error:', fixResult.fixDescription);
      return await implementTask(task); // Retry
    }

    if (fixResult.escalationRequired) {
      console.log('❌ Cannot auto-fix - escalating to human');

      const decision = await escalate({
        type: 'pattern_unknown',
        error: error,
        attemptedFixes: fixResult.attemptedFixes
      });

      if (decision.humanProvidedFix) {
        // Apply human's fix
        await applyFix(decision.fix);
        return await implementTask(task); // Retry
      }

      return { failed: true, error: error.message };
    }
  }
}
```

### 3. Post-Execution: QA Review

Ralph completes task → AEO QA reviews before committing:

```
Ralph completes implementation:
├── Code written
├── Tests pass
└── Ready to commit?

AEO QA review:
├── Scan for security issues (VETO POWER)
├── Auto-fix code smells
├── Check test coverage
├── Check architecture
└── If all pass → Approve commit
```

**Example Integration:**

```javascript
// In Ralph's commit handler
import { reviewChanges } from 'aeo-qa-agent';

async function commitChanges(task, changedFiles) {
  console.log('Running AEO QA review...');

  const review = await reviewChanges(changedFiles, {
    context: getExecutionContext()
  });

  if (review.veto) {
    console.log('🚨 QA VETO - Security issue detected');
    console.log('Issue:', review.vetoReason);
    console.log('File:', review.vetoFile);

    // Block commit
    return { committed: false, blocked: 'qa_veto' };
  }

  if (review.autoFixesApplied > 0) {
    console.log(`🔧 Applied ${review.autoFixesApplied} auto-fixes`);
    console.log('Fixes:', review.fixDescriptions);
  }

  if (review.testCoverageIssues) {
    console.log('⚠️ Test coverage issues detected:');
    review.testCoverageIssues.forEach(issue => {
      console.log('  -', issue);
    });

    // Could block or warn based on severity
    return { committed: false, blocked: 'missing_tests' };
  }

  console.log('✅ QA review passed - safe to commit');
  return await gitCommit(changedFiles);
}
```

## PRD Format Compatibility

Ralph uses `prd.json` - AEO integrates with this format:

```json
{
  "title": "User Authentication System",
  "description": "Implement JWT-based authentication with email verification",
  "stories": [
    {
      "id": "story-1",
      "title": "Create Authentication Service",
      "description": "Build service that generates and validates JWT tokens",
      "acceptanceCriteria": [
        "Service generates JWT tokens on successful login",
        "Tokens expire after 15 minutes",
        "Refresh tokens stored in Redis"
      ],
      "tasks": [
        {
          "id": "task-1",
          "description": "Implement JWT token generation using jose library",
          "context": {
            "techStack": ["Node.js", "jose", "Redis"],
            "dependencies": ["User model", "Redis client"],
            "files": ["src/services/AuthService.js"]
          }
        }
      ]
    }
  ]
}
```

**AEO reads from PRD:**

```javascript
// Extract task from Ralph's PRD
function extractTaskFromPRD(prd, taskId) {
  for (const story of prd.stories) {
    for (const task of story.tasks) {
      if (task.id === taskId) {
        return {
          description: task.description,
          acceptanceCriteria: story.acceptanceCriteria,
          techStack: task.context.techStack,
          dependencies: task.context.dependencies,
          files: task.context.files
        };
      }
    }
  }
  return null;
}

// Use in AEO spec validation
const task = extractTaskFromPRD(prd, currentTaskId);
const specScore = validateSpec(task);
```

## Progress Tracking

Ralph uses `progress.txt` for state persistence - AEO contributes signals:

```
Ralph progress.txt:
├── Current story: story-1
├── Current task: task-1
├── Status: implementing
└── Timestamp: 2026-01-22T10:30:00Z

AEO adds to progress:
├── Confidence: 0.85 (autonomous)
├── Spec score: 85/100 (well-defined)
├── QA status: pending
└── Signals recorded to memory
```

**Example Integration:**

```javascript
// In Ralph's progress updater
import { recordSignal } from 'aeo-core';

async function updateProgress(prd, taskId, updates) {
  // Ralph's standard progress update
  const progress = {
    ...loadProgress(),
    ...updates,
    timestamp: new Date().toISOString()
  };

  // AEO adds confidence tracking
  if (updates.confidence) {
    progress.aeo = {
      confidence: updates.confidence,
      specScore: updates.specScore,
      qaStatus: updates.qaStatus
    };
  }

  writeProgress('progress.txt', JSON.stringify(progress, null, 2));

  // AEO records to memory
  if (updates.completed) {
    await recordSignal({
      task_id: taskId,
      predicted_confidence: updates.confidence,
      actual_difficulty: updates.difficulty,
      success: updates.success,
      adjustment: updates.adjustment
    });
  }
}
```

## End-to-End Flow

### Complete Ralph + AEO Session

```bash
# 1. Ralph starts with PRD
ralph execute prd.json

# 2. Ralph breaks down into stories/tasks
ralph> Breaking down into 3 stories with 8 tasks...

# 3. For each task, AEO validates
ralph> [Task: Implement JWT tokens]
aeo> Spec score: 82/100 - well-defined
aeo> Confidence: 0.87 - AUTONOMOUS
ralph> Proceeding with implementation...

# 4. Ralph executes task
ralph> Writing src/services/AuthService.js...
ralph> Running tests...

# 5. AEO QA reviews
aeo> Scanning files for issues...
aeo> ✅ QA review passed
ralph> Committing changes...

# 6. Record outcome
aeo> Recording signal to memory...

# 7. Next task
ralph> [Task: Add email verification]
aeo> Spec score: 45/100 - NEEDS CLARIFICATION
aeo> ⚠️ CONFIDENCE BELOW THRESHOLD
aeo> Escalating to human...

human> We're using Resend for emails
aeo> Confidence updated: 0.75 - ADVISORY
ralph> Proceeding with assumptions...
```

## Configuration

### Ralph Config with AEO

Create `.ralph/config.json`:

```json
{
  "aeo": {
    "enabled": true,
    "confidenceThreshold": 0.70,
    "specThreshold": 40,
    "autoQA": true,
    "escalation": true
  },
  "prd": {
    "path": "prd.json",
    "backupPath": "prd.backup.json"
  },
  "progress": {
    "path": "progress.txt",
    "persistent": true
  }
}
```

### Enable AEO in Ralph

```bash
# Install AEO skills
npx skills add laurence/aeo-skills

# Enable AEO in Ralph config
cat > .ralph/config.json << 'EOF'
{
  "aeo": {
    "enabled": true,
    "confidenceThreshold": 0.70
  }
}
EOF

# Run Ralph with AEO
ralph execute prd.json
```

## Error Handling

### AEO Signals to Ralph

```javascript
// AEO provides signals to Ralph's execution loop
const SIGNALS = {
  SPEC_ACCEPTED: 'spec_accepted',
  SPEC_REFUSED: 'spec_refused',
  CONFIDENCE_HIGH: 'confidence_high', // ≥0.85
  CONFIDENCE_MEDIUM: 'confidence_medium', // 0.70-0.84
  CONFIDENCE_LOW: 'confidence_low', // <0.70
  QA_PASSED: 'qa_passed',
  QA_VETOED: 'qa_veto',
  ESCALATION_RESOLVED: 'escalation_resolved'
};

async function handleAEOSignal(signal, data) {
  switch (signal) {
    case SIGNALS.SPEC_REFUSED:
      console.log('⚠️ Task spec refused by AEO');
      console.log('Spec score:', data.score);
      console.log('Reason:', data.reason);
      // Ralph pauses and asks for better spec
      break;

    case SIGNALS.CONFIDENCE_LOW:
      console.log('⚠️ Low confidence - human input needed');
      console.log('Confidence:', data.confidence);
      // Ralph triggers escalation
      break;

    case SIGNALS.QA_VETOED:
      console.log('🚨 QA veto - blocking commit');
      console.log('Reason:', data.reason);
      // Ralph blocks commit, waits for fix
      break;

    case SIGNALS.ESCALATION_RESOLVED:
      console.log('✅ Escalation resolved - continuing');
      // Ralph resumes execution
      break;

    default:
      console.log('Unknown AEO signal:', signal);
  }
}
```

## Best Practices

### 1. Provide Context in PRD

```json
{
  "tasks": [
    {
      "id": "task-1",
      "description": "Add JWT authentication",
      "context": {
        "techStack": ["jose", "bcrypt"],
        "files": ["src/services/AuthService.js"],
        "dependencies": ["User model"],
        "acceptanceCriteria": [
          "Tokens expire in 15 minutes",
          "Refresh tokens in Redis"
        ]
      }
    }
  ]
}
```

**Better AEO confidence** with context vs without:
- Without context: 0.52 (BLOCKING)
- With context: 0.87 (AUTONOMOUS)

### 2. Let AEO Handle Errors

```javascript
// Don't catch all errors in Ralph
// Let AEO's failure patterns handle them

// ❌ Wrong: Blocking all errors
try {
  await implementTask(task);
} catch (error) {
  console.log('Task failed:', error);
  // Ralph stops
}

// ✅ Right: Let AEO try to fix
await implementTask(task); // AEO failure patterns will auto-fix if possible
```

### 3. Respect QA Vetoes

```javascript
// Always check QA result before committing
const review = await reviewChanges(changedFiles);

if (review.veto) {
  console.log('Blocked by QA - cannot commit');
  // Do NOT commit - let human fix the issue
  return;
}

// Only commit if QA passes
await gitCommit(changedFiles);
```

## Testing Integration

### Test AEO + Ralph Together

```bash
# Create test PRD
cat > test-prd.json << 'EOF'
{
  "title": "Test Feature",
  "stories": [
    {
      "id": "story-1",
      "title": "Simple Feature",
      "description": "Add a simple utility function",
      "acceptanceCriteria": ["Function works as expected"],
      "tasks": [
        {
          "id": "task-1",
          "description": "Create utility function to format dates",
          "context": {
            "techStack": ["JavaScript", "date-fns"],
            "files": ["src/utils/dateFormatter.js"]
          }
        }
      ]
    }
  ]
}
EOF

# Run Ralph with AEO
ralph execute test-prd.json

# Expected flow:
# 1. Ralph reads PRD
# 2. AEO validates spec (score should be high)
# 3. AEO calculates confidence (should be autonomous)
# 4. Ralph executes task
# 5. AEO QA reviews
# 6. Changes committed
# 7. Signal recorded to memory
```

## Troubleshooting

### AEO blocking too much

**Issue:** AEO keeps escalating even for simple tasks

**Solution:**
```json
// Adjust Ralph config to lower threshold
{
  "aeo": {
    "confidenceThreshold": 0.50  // Lower from 0.70
  }
}
```

### QA rejecting good code

**Issue:** QA vetoing safe changes

**Solution:**
```javascript
// Check QA configuration
// May need to adjust security rules or add exclusions
// Create ~/.claude/USER/aeo-exclusions.json:
{
  "exclusions": [
    "src/tests/*",  // Test files exempt from some rules
    "src/mocks/*"   // Mock files exempt
  ]
}
```

### Memory not persisting

**Issue:** Signals not being recorded

**Solution:**
```bash
# Check PAI memory directory
ls -la ~/.claude/MEMORY/

# Verify AEO can write
echo '{"test":"data"}' > ~/.claude/MEMORY/aeo-test.jsonl

# Check Ralph progress.txt
cat progress.txt
```
