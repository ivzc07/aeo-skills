# AEO - Autonomous Engineering Organization

A layered autonomous development system for Claude Code that provides confidence-based decision making, automated QA, spec validation, and intelligent human escalation.

## What is AEO?

AEO is a set of Vercel Skills that enables Claude Code to execute development tasks autonomously with built-in safeguards:

- **Confidence scoring** - Knows when to proceed vs when to ask
- **Spec validation** - Ensures tasks are well-defined before starting
- **Automated QA** - Internal code reviewer with veto power
- **Failure patterns** - Recognizes and auto-fixes common errors
- **Smart escalation** - Human-AI interface with clear options
- **Cost tracking** - Token usage and budget management
- **Architecture protection** - Detects circular deps and layer violations

## Installation

```bash
# Prerequisites: PAI and Ralph must be installed first
npx skills add snarktank/ralph
npx skills add ivzc07/aeo-skills
```

## Quick Start

```bash
# In your project
claude

# Activate autonomous mode
/aeo

# Describe your task
"Add user authentication with email verification"

# AEO will:
# 1. Validate the spec
# 2. Calculate confidence
# 3. Execute or escalate
# 4. Run QA review
# 5. Learn from outcome
```

## Architecture

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

## Skills

### Essential Skills (auto-installed)

1. **aeo-core** - Confidence engine and autonomy decision making
2. **aeo-spec-validator** - Task definition validation
3. **aeo-qa-agent** - Internal code reviewer with veto power
4. **aeo-failure-patterns** - Error pattern recognition and auto-fix
5. **aeo-escalation** - Human-AI interface

### Optional Skills

6. **aeo-cost-governor** - Token usage and budget tracking
7. **aeo-architecture** - Architecture analysis and protection

## Usage Examples

```bash
# Small task - AEO executes autonomously
"Add email validation to the signup form"
→ Confidence: 0.92 → Autonomous execution

# Medium task - AEO generates mini-PRD
"Create a dashboard showing user metrics"
→ Confidence: 0.78 → Advisory mode → Executes

# Large task - AEO escalates for planning
"Build a complete project management system"
→ Confidence: 0.45 → Escalates → Generates PRD → Ralph executes

# Risky task - AEO blocks
"Refactor the authentication system"
→ Confidence: 0.35 → Refuses → Requests detailed spec
```

## Configuration

Optional configuration files in `$PAI_DIR/USER/`:

```json
// ~/.claude/USER/aeo-budget.json
{
  "daily_budget_usd": 10.00,
  "per_task_budget_usd": 2.00,
  "alert_threshold_percent": 80
}
```

```yaml
# ~/.claude/USER/aeo-layers.yaml
layers:
  - name: "presentation"
    may_import: ["domain", "application"]
  - name: "domain"
    may_import: []
  - name: "application"
    may_import: ["domain"]
```

## Memory and Learning

AEO stores data in `$PAI_DIR/MEMORY/`:

- `aeo-signals.jsonl` - Task outcomes for confidence learning
- `aeo-escalations.jsonl` - Escalation decisions and patterns
- `aeo-failure-patterns.json` - Learned error patterns
- `aeo-costs.jsonl` - Token usage tracking

## Development

```bash
# Fork and clone
git clone https://github.com/YOUR_USERNAME/aeo-skills.git
cd aeo-skills

# Make changes
vim skills/aeo-core/SKILL.md

# Test locally
npx skills link .

# Publish
git push origin main
# Verify at https://skills.sh
```

## License

MIT

## References

- [Vercel Skills](https://github.com/vercel-labs/agent-skills)
- [Ralph](https://github.com/snarktank/ralph)
- [PAI](https://github.com/danielmiessler/Personal_AI_Infrastructure)
- [skills.sh](https://skills.sh)
