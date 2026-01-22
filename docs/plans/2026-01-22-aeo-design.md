# AEO - Autonomous Engineering Organization Design Document

**Date:** 2026-01-22
**Author:** Design created via brainstorming session
**Status:** Approved for Implementation

---

## Overview

AEO is a layered autonomous development system for Claude Code that provides confidence-based decision making, automated QA, spec validation, and intelligent human escalation. Built as Vercel Skills on top of PAI infrastructure with Ralph integration.

## Architecture Philosophy

**Trust but Verify:** Calculate confidence, execute autonomously when high, involve human when low, continuously learn from outcomes.

## System Layers

1. **Core Layer (aeo-core)**: Brain that calculates confidence scores and makes autonomy decisions
2. **Essential Skills** (auto-installed): spec-validator, qa-agent, escalation, failure-patterns
3. **Optional Skills**: cost-governor, architecture

## Activation Model

**Opt-in via slash command:**
- `/aeo` → Enter autonomous mode (sets AEO_ENABLED=true)
- `/aeo off` → Exit autonomous mode
- `/aeo status` → Show current state, costs, patterns

## Integration Points

### With PAI
- Uses `$PAI_DIR/MEMORY/` for signals, escalations, patterns, costs, ADRs
- Extends PAI hooks to capture AEO events
- TELOS context for user goals

### With Ralph
- On-demand invocation at decision points
- Compatible with prd.json format
- Pre-execution validation, post-execution review

---

## Skill Specifications

### 1. aeo-core (Confidence Engine)

**Purpose:** Calculate confidence scores (0-1) and decide execution path

**Confidence Calculation (Learning System):**

```javascript
// Phase 0: Spec Validation
spec_score = aeo_spec_validator.validate(task)
if spec_score < 40:
    return REFUSE("Spec too unclear")

// Phase 1: Rule-Based Score
base_confidence = clamp(0.50 + clarity + context - risk, 0.0, 1.0)

// Phase 2: Spec Score Adjustment
if spec_score >= 80: base_confidence += 0.10
elif spec_score < 60: base_confidence -= 0.10

// Phase 3: Security Multipliers
if task.touches_payments: base_confidence *= 0.5
elif task.touches_auth: base_confidence *= 0.7

// Phase 4: LLM Adjustment (±0.10)
final_confidence = clamp(base_confidence + llm_adjustment, 0.0, 1.0)
```

**Decision Thresholds:**
- **≥ 0.85**: Autonomous - execute without asking
- **≥ 0.70**: Advisory - note risk, offer to pause
- **≥ 0.50**: Blocking - explain concern, wait for confirmation
- **< 0.50**: Refuse - explain why, request clarification

**Learning from Outcomes:**
- Write signals to `$PAI_DIR/MEMORY/aeo-signals.jsonl`
- Adjust confidence based on actual difficulty
- Rolling 100-task window

**Rule-Based Factors:**

**Clarity (+0.15 each):**
- Explicit acceptance criteria
- Tech stack specified
- Dependencies listed
- Test requirements defined

**Context (+0.10 each):**
- Similar successful task in memory
- Familiar codebase area
- Recent successful commits

**Risk (-0.10 each):**
- Touching authentication/security
- Modifying core infrastructure
- Large scope (>5 files, >500 LOC)
- Unclear dependencies

---

### 2. aeo-spec-validator

**Purpose:** Validate tasks are sufficiently defined before execution

**Scoring (0-100):**

**Clarity Indicators (0-50):**
- Objective clarity: 15 pts (explicit vs vague)
- Acceptance criteria: 15 pts (specific vs none)
- Context provided: 10 pts (full vs missing)
- Dependencies identified: 10 pts (full vs none)

**Quality Indicators (0-30):**
- Tech stack specified: 10 pts
- Test requirements: 10 pts
- Performance/security: 10 pts

**Risk Assessment (0-20):**
- Scope: 10 pts (small vs large)
- Complexity: 10 pts (CRUD vs multi-system)

**Ambiguity Detection:**
- Subjective terms: fast, good, simple, clean
- Vague verbs: improve, optimize, enhance (without specifics)
- Missing constraints: no performance/error requirements

**Actions by Score:**
- **80-100**: Proceed - well-defined
- **60-79**: Minor gaps - note assumptions
- **40-59**: Major gaps - ask questions
- **< 40**: Unacceptable - refuse

---

### 3. aeo-qa-agent

**Purpose:** Internal code reviewer with veto power

**Review Categories:**

**Security Issues (VETO):**
- SQL injection, XSS, CSRF
- Hardcoded credentials
- Missing input validation
- Insecure cryptography
- Auth bypasses

**Code Smells (Auto-fix):**
- `console.log`, `debugger` - remove
- TODO/FIXME without tickets - flag
- Unused imports/variables - remove
- Inconsistent naming - fix
- Magic numbers - extract constants
- Duplicate code - extract function

**Test Coverage:**
- New features without tests - flag
- Edge cases not covered - flag

**Architecture Violations:**
- Circular dependencies
- Layer violations
- Breaking encapsulation

**Review Timing:**
- After completing logical work unit
- Before suggesting completion
- Before git commit (final gate)

**Veto Process:**
1. Stop execution immediately
2. Output issue with file:line
3. Show current code vs required fix
4. Block until resolved
5. Re-review after fix

**Auto-fix Rules:**
- Simple fixes (remove console.log): apply without asking
- Complex fixes (refactoring): flag for review

---

### 4. aeo-escalation

**Purpose:** Human-AI interface for when to interrupt and involve humans

**Escalation Triggers:**
1. Low confidence (< 0.70)
2. QA veto
3. Failure pattern match (2+ failures)
4. Cost limit approaching (80% warning, 100% stop)
5. Architecture violation

**Presentation Format:**
```
⚠️ ESCALATION REQUIRED

Task: [description]
Confidence: 0.62

Concerns:
• [Spec] Missing validation rules
• [Risk] Touching authentication
• [Context] No integration mentioned

Options:
1. 📝 Clarify spec first
2. ⚡ Proceed with assumptions
3. 🛑 Pause and escalate
4. 🔄 Break into smaller task

Your choice (1-4):
```

**Decision Recording:**
- Write to `$PAI_DIR/MEMORY/aeo-escalations.jsonl`
- Track task_id, confidence, trigger, user_choice

**Learning from Escalations:**
- Consistent "proceed" → Increase base confidence by 0.05
- Consistent "clarify" → Flag as "needs better specs"
- Custom direction → Extract pattern

**Escalation Fatigue Prevention:**
- 3 escalations in 10 min → Suggest break or task redefinition
- Same issue twice → Ask to adjust threshold or skip checks

---

### 5. aeo-failure-patterns

**Purpose:** Recognize common errors and apply known fixes automatically

**Hybrid Architecture:**

**Core Patterns (in SKILL.md):**
20-30 curated patterns with high-confidence fixes:
- Dependency issues (module not found, version conflicts)
- Type errors (property missing, type mismatch)
- Build issues (stack overflow, timeout)
- Test failures (async callback, timeout)
- File system (permission denied)
- Configuration (config not found)
- Git issues (merge conflicts, unrelated histories)

**Project-Specific Learnings (in PAI memory):**
`$PAI_DIR/MEMORY/aeo-failure-patterns.json`
- Pattern + context + fix
- Success count and confidence
- Learned from human resolutions

**Pattern Matching Flow:**
1. Error occurs → Extract message + stack + context
2. Match core patterns → If confidence ≥ 0.85 → Auto-fix
3. Match project patterns → If confidence ≥ 0.80 → Auto-fix
4. Fuzzy match → If confidence ≥ 0.70 → Propose, ask confirmation
5. No match → Escalate

**Max Retry Limit:**
- Same error, same fix → Max 2 attempts
- After 2 failures → Escalate
- Prevents infinite loops

**Learning New Patterns:**
- Human resolution → Capture with confidence 0.50
- 3+ successes → Promote to auto-fix (≥ 0.80)
- Failures → Reduce confidence by 0.20
- Below 0.50 → Remove from active

---

### 6. aeo-cost-governor (Optional)

**Purpose:** Track token usage and budget

**Pricing Data:**
```yaml
claude-opus-4-5: input $15/M, output $75/M
claude-sonnet-4-5: input $3/M, output $15/M
claude-haiku-4-5: input $0.80/M, output $4/M
```

**Budget Configuration:**
`$PAI_DIR/USER/aeo-budget.json`
```json
{
  "daily_budget_usd": 10.00,
  "per_task_budget_usd": 2.00,
  "alert_threshold_percent": 80
}
```

**Tracking:**
`$PAI_DIR/MEMORY/aeo-costs.jsonl`
- timestamp, task_id, model, tokens, cost

**Enforcement:**
- 80% daily: Warning
- 100% daily: Hard stop
- Per-task: Stop if exceeds

**Model Selection Guidance:**
- Haiku: Simple formatting, basic refactors
- Sonnet: Feature implementation, bug fixes
- Opus: Architecture, complex debugging, security

---

### 7. aeo-architecture (Optional)

**Purpose:** Analyze and protect code architecture

**Architecture Analysis:**
1. Build dependency graph
2. Detect circular dependencies
3. Check layer violations
4. Find hidden coupling

**Layer Protection:**
`$PAI_DIR/USER/aeo-layers.yaml`
Define allowed imports between layers

**Architecture Decision Records (ADRs):**
`$PAI_DIR/MEMORY/ADRS/YYYY-MM-DD-ADR-XXX-title.md`
- Context, Decision, Consequences
- Auto-generate on architectural changes

**Health Score (0-100):**
```
100 - (circular_deps * 10)
     - (layer_violations * 5)
     - (duplicate_code * 2)
     - (missing_tests * 1)
```

**Integration with QA:**
- QA detects potential issue → Delegate to aeo-architecture
- aeo-architecture confirms → Returns to QA for veto

---

## Data Flow

```
User Request → /aeo activates
            ↓
aeo-core calculates confidence
            ↓
aeo-spec-validator scores spec
            ↓
If confidence < 0.70 → aeo-escalation
If confidence ≥ 0.70 → Execute autonomously
            ↓
Ralph execution loop (on-demand calls AEO)
            ↓
aeo-qa-agent reviews (before commit)
            ↓
If QA veto → Fix → Re-review
If QA passes → Commit
            ↓
aeo-failure-patterns handles errors
            ↓
Record outcome → PAI memory
            ↓
Update confidence model
```

---

## Repository Structure

```
aeo-skills/
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── skills/
│   ├── aeo-core/SKILL.md
│   ├── aeo-spec-validator/SKILL.md
│   ├── aeo-qa-agent/SKILL.md
│   ├── aeo-failure-patterns/SKILL.md
│   ├── aeo-escalation/SKILL.md
│   ├── aeo-cost-governor/SKILL.md
│   └── aeo-architecture/SKILL.md
└── .github/workflows/skill-validation.yml
```

---

## Installation Flow

```bash
# Step 1: Install PAI
git clone https://github.com/danielmiessler/Personal_AI_Infrastructure.git ~/.claude

# Step 2: Install Ralph
npx skills add snarktank/ralph

# Step 3: Install AEO
npx skills add laurence/aeo-skills

# Step 4: Configure (optional)
mkdir -p ~/.claude/USER
cat > ~/.claude/USER/aeo-budget.json << EOF
{
  "daily_budget_usd": 10.00,
  "per_task_budget_usd": 2.00
}
EOF
```

---

## Key Design Decisions

1. **Learning System for Confidence** - Starts rule-based, improves from outcomes
2. **Hybrid Failure Patterns** - Core portable + project-specific learnings
3. **Opt-in Activation** - /aeo command, not auto-load
4. **Batch QA Review** - Before commit, not after every keystroke
5. **Security Multipliers** - Auth/payments never reach full autonomy
6. **Max Retry Limit** - Prevents infinite fix loops
7. **Two-Tier Architecture** - Essential (4 skills) + Optional (2 skills)

---

## Success Criteria

- Users can install with `npx skills add laurence/aeo-skills`
- `/aeo` activates autonomous mode successfully
- Confidence scores accurately predict task difficulty
- QA catches security issues before commit
- Failure patterns resolve common errors autonomously
- Escalations present clear, actionable options
- Integration with PAI memory and Ralph works end-to-end

---

## References

- [Vercel Skills](https://github.com/vercel-labs/agent-skills)
- [Ralph](https://github.com/snarktank/ralph)
- [PAI](https://github.com/danielmiessler/Personal_AI_Infrastructure)
- [skills.sh](https://skills.sh)
