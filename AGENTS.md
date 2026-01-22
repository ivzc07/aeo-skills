# AEO - Agent Instructions

This document contains instructions for AI agents working on the AEO codebase.

## Repository Purpose

AEO is a Vercel Skills repository that provides autonomous engineering capabilities to Claude Code. Each skill is a markdown file (SKILL.md) containing instructions for LLMs, not executable code.

## File Structure

```
aeo-skills/
├── README.md                 # User-facing documentation
├── AGENTS.md                 # This file - agent instructions
├── CLAUDE.md                 # Claude Code specific configuration
└── skills/
    ├── aeo-core/SKILL.md             # Main confidence engine
    ├── aeo-spec-validator/SKILL.md   # Spec validation
    ├── aeo-qa-agent/SKILL.md         # Code review and QA
    ├── aeo-failure-patterns/SKILL.md # Error pattern recognition
    ├── aeo-escalation/SKILL.md       # Human escalation
    ├── aeo-cost-governor/SKILL.md    # Cost tracking (optional)
    └── aeo-architecture/SKILL.md     # Architecture analysis (optional)
```

## How Skills Work

**IMPORTANT:** Skills are NOT code. They are markdown instructions that tell the LLM what to do.

When a user types `/aeo`:
1. Claude loads `skills/aeo-core/SKILL.md`
2. The SKILL.md contains instructions like "Calculate confidence score..."
3. Claude follows those instructions
4. Other skills are invoked as needed via the Skill tool

## Development Rules

1. **Skills are instructions, not code** - Write in imperative mood: "Calculate X", not "function calculateX()"
2. **Reference other skills** - Use exact skill names when invoking
3. **Specify file paths** - Always use `$PAI_DIR/MEMORY/` for PAI integration
4. **Include examples** - Show expected input/output formats
5. **Define triggers** - When should this skill activate?
6. **Keep it self-contained** - Each skill should work independently
7. **Test locally** - Use `npx skills link .` before publishing

## PAI Integration Points

All skills integrate with PAI infrastructure:

### Memory
- **Location:** `$PAI_DIR/MEMORY/`
- **Files:** `aeo-signals.jsonl`, `aeo-escalations.jsonl`, `aeo-failure-patterns.json`, `aeo-costs.jsonl`
- **Access:** Use Read tool to read, Write tool to append

### Hooks
- **SessionStart:** Load AEO state
- **SessionStop:** Save signals and learnings
- **PreToolUse:** Track costs (if cost-governor enabled)

### TELOS
- **Location:** `$PAI_DIR/USER/TELOS/`
- **Purpose:** User goals and context
- **Access:** Read to understand user's objectives

## Ralph Integration

Ralph is the execution loop that AEO integrates with:

### Trigger Points
- **Pre-execution:** AEO validates spec and calculates confidence
- **During execution:** On-demand calls for escalation
- **Post-execution:** AEO QA review before commit

### PRD Format
Ralph uses `prd.json` - AEO should validate PRD completeness before execution begins

### Progress Tracking
Use Ralph's `progress.txt` for persisting state between iterations

## Testing Skills

Since skills are instructions, testing means:

1. **Manual testing:** Install skill locally, test with real tasks
2. **Prompt testing:** Try skill instructions with various inputs
3. **Integration testing:** Verify PAI memory and Ralph integration
4. **Edge cases:** Test with ambiguous specs, risky tasks, etc.

## Publishing

1. Update version in README.md
2. Commit all changes
3. Push to GitHub
4. Verify at https://skills.sh
5. Test installation: `npx skills add YOUR_USERNAME/aeo-skills`

## Common Tasks

### Add a new failure pattern
Edit `skills/aeo-failure-patterns/SKILL.md`:
- Add pattern to "Core Patterns" section
- Include error message signature
- Specify fix steps
- Set confidence threshold (0.85 for auto-fix)

### Adjust confidence thresholds
Edit `skills/aeo-core/SKILL.md`:
- Modify "Decision Thresholds" section
- Adjust base confidence calculation
- Update rule-based factors

### Add new QA check
Edit `skills/aeo-qa-agent/SKILL.md`:
- Add to appropriate category (Security, Code Smells, etc.)
- Specify whether VETO or auto-fix
- Include examples

## Code Style

Even though skills are markdown, follow these conventions:

- **Headings:** Use `##` for main sections
- **Code blocks:** Use ```bash for commands, ```javascript for data structures
- **Lists:** Use `-` for items, numbered lists for sequences
- **Emphasis:** Use **bold** for critical points, `code` for file paths and commands
- **Examples:** Show concrete input/output
- **References:** Link to related skills and documentation

## Getting Help

- **Vercel Skills:** https://github.com/vercel-labs/agent-skills
- **Skills CLI:** https://skills.sh/docs
- **PAI:** Check `$PAI_DIR/skills/CORE/SKILL.md`
- **Ralph:** https://github.com/snarktank/ralph
