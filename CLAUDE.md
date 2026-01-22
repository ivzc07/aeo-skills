# AEO - Claude Code Configuration

This file contains Claude Code specific configuration for AEO skills.

## Skill Activation

### Primary Activation
User types: `/aeo`
Action: Load `skills/aeo-core/SKILL.md`

### Secondary Activations
Other skills activate via instructions within aeo-core:
- aeo-spec-validator - "Invoke Skill tool with skill='aeo-spec-validator'"
- aeo-qa-agent - "Invoke Skill tool with skill='aeo-qa-agent'"
- aeo-failure-patterns - "Invoke Skill tool with skill='aeo-failure-patterns'"
- aeo-escalation - "Invoke Skill tool with skill='aeo-escalation'"

## Tool Usage

### Skill Tool
Always use the Skill tool to invoke other AEO skills:

```typescript
// Correct
Skill({ skill: "aeo-spec-validator" })

// Wrong - don't use Read tool on skill files
Read({ file_path: "skills/aeo-spec-validator/SKILL.md" })
```

### Read/Write Tools
For PAI memory integration:

```typescript
// Read existing signals
Read({ file_path: `${process.env.PAI_DIR}/MEMORY/aeo-signals.jsonl` })

// Append new signal
Bash({ command: `echo '${JSON.stringify(signal)}' >> ~/.claude/MEMORY/aeo-signals.jsonl` })
```

## Environment Variables

Skills should reference:
- `$PAI_DIR` - PAI installation directory (usually `~/.claude`)
- `$PAI_HOME` - Same as $PAI_DIR
- `USER` - User's home directory

## Error Handling

When skills fail:
1. **aeo-failure-patterns** - Try to match and auto-fix
2. **aeo-escalation** - If can't fix, escalate to human
3. **aeo-core** - Adjust confidence based on outcome

## Memory Cleanup

Periodically (every 100 tasks):
- Keep last 100 signals in `aeo-signals.jsonl`
- Archive older signals to `aeo-signals.archive.jsonl`
- Same for escalations, costs, patterns

## Debugging

To debug AEO skills:

```bash
# Enable verbose logging
export AEO_DEBUG=true

# Check memory files
cat ~/.claude/MEMORY/aeo-signals.jsonl | tail -20

# Test skill loading
npx skills link .
claude
> /aeo
```

## Session Context

Each Claude session:
1. **On start:** aeo-core reads `$PAI_DIR/MEMORY/aeo-signals.jsonl` to calculate rolling confidence
2. **During execution:** Skills write signals to memory
3. **On stop:** PAI's SessionStop hook captures AEO state
