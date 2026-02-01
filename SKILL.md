---
name: create-skill
description: Create Claude Code skills through structured interview. Use when
  formalizing a workflow, building a capability, or asked to "make a skill".
allowed-tools: [AskUserQuestion, Write, Read]
---

# /create-skill

Build skills through interview, not scaffolding.

## Philosophy

- **Interview-driven**: Questions reveal requirements better than templates
- **Non-obvious questions**: Ask about failure modes, not obvious inputs
- **3 examples minimum**: Specificity before abstraction
- **Progressive disclosure**: Keep SKILL.md lean, split heavy content to references/

## Workflow

Guide the user through 4 phases using AskUserQuestion. Use multiple choice
where possible. Ask one question at a time. Go deep.

### Phase 1: Shape

Understand the problem space:

```
Q: What type of skill is this?
  1. Workflow automation (multi-step process)
  2. Code generation (templates, scaffolding)
  3. Analysis/review (examine and report)
  4. Integration (connect external tools)
```

Ask:
- What does this skill do? (1 sentence)
- When should it trigger? (specific scenarios)
- Who uses it? (personal, team, public)

### Phase 2: Flow

Walk through 3 concrete examples:

```
Q: In example 1, what's the first thing you do?
  1. Read a file
  2. Run a command
  3. Ask the user something
  4. Search the codebase
```

- "Describe example 1 step by step"
- "Now a different scenario (example 2)"
- "What's an edge case? (example 3)"

Extract common patterns and variations across all examples.

### Phase 3: Detail

Nail down specifics:

```
Q: Does this skill need external tools?
  1. No, just Claude Code built-ins
  2. Yes, CLI tools (specify which)
  3. Yes, APIs (specify which)
  4. Not sure yet
```

Ask:
- What tools does it need? (Read, Write, Bash, Task, etc.)
- External dependencies? (CLIs, APIs)
- What's deterministic vs needs LLM judgment?
- What must it NEVER do? (constraints)

### Phase 4: Completeness

Cover edge cases:

```
Q: If the skill fails, what should happen?
  1. Stop and show error
  2. Ask user how to proceed
  3. Try alternative approach
  4. Fail silently (non-critical)
```

Ask:
- What if [likely failure] happens?
- Should this persist or be ephemeral?
- What should NOT be in this skill?

### Generate

Create SKILL.md from interview answers:

```yaml
---
name: skill-name
description: What it does and when to trigger (under 200 chars)
allowed-tools: [tools identified in Phase 3]
# Optional fields from interview:
# requires: [binaries, APIs]
# constraints: [what it must NOT do]
# ephemeral: true  # if one-off
---

# /skill-name

[One sentence: what this skill does]

## When to Use

- [Trigger condition 1 from Phase 1]
- [Trigger condition 2]

## When NOT to Use

- [Anti-pattern from Phase 4]

## Workflow

### Step 1: [From Phase 2 examples]
[What to do, derived from common patterns across 3 examples]

### Step 2: [...]
[...]

## Examples

[Condensed versions of the 3 examples from Phase 2]

## Error Handling

[From Phase 4 completeness questions]
```

Target: under 500 lines, no TODOs.

### Validate (Optional)

- `/magi "review this skill for clarity and completeness"`
- Test immediately on a real task
- Iterate based on usage

## Output Location

Write to: `~/.claude/skills/{skill-name}/SKILL.md`

Only create references/ directory if content exceeds 500 lines.

## What NOT to Include

- README.md (SKILL.md is the readme)
- CHANGELOG.md (use git)
- LICENSE (inherit)
- Separate config (use frontmatter)
- Scripts (unless truly necessary)
