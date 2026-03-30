---
name: create-skill
description: Create reusable agent skills through structured interview. Use when
  formalizing a workflow, building a capability, turning a conversation into a
  reusable skill, or asked to "make a skill". Use this even if the user just
  says "skill" or "slash command" or "automate this".
argument-hint: "[workflow description]"
allowed-tools: AskUserQuestion, Write, Read
---

# /create-skill

Build skills through interview, not scaffolding.

## Philosophy

- **Interview-driven**: Questions reveal requirements better than templates
- **Capture, don't invent**: Check if the current conversation already contains a workflow worth capturing — extract steps, tools used, and corrections before asking fresh questions
- **3 examples minimum**: Specificity before abstraction
- **Explain the why**: Instructions that explain reasoning outperform rigid MUSTs. The agent is smart — give it context to make judgment calls, not just rules to follow
- **Progressive disclosure**: Keep SKILL.md lean, split heavy content to reference files
- **Lean over rigid**: Remove instructions that aren't pulling their weight. If test runs show the agent ignoring a section, cut it rather than adding enforcement

## Workflow

Guide the user through 5 phases using AskUserQuestion. Use multiple choice
where possible. Ask one question at a time. Go deep.

**Before starting**: Check if the current conversation already contains a workflow the user wants to capture. If so, extract the tools used, the sequence of steps, corrections the user made, and input/output formats observed. Present this as a starting point — the user fills gaps and confirms.

### Phase 1: Shape

Understand the problem space:

```
Q: What type of skill is this?
  1. Discipline (enforces a practice — TDD, review, verification)
  2. Process (guides multi-step workflows — brainstorming, planning, deployment)
  3. Technique (teaches patterns/recipes — code generation, analysis)
  4. Reference (provides information — API docs, templates, specs)
  5. Always-on (runs every session — runbooks, session state)
```

The skill type determines which hardening techniques apply later in Phase 5.
See [references/techniques.md](references/techniques.md) for the full matrix.

Ask:
- What does this skill do? (1 sentence)
- When should it trigger? (specific scenarios, natural phrasings a user would say)
- When should it NOT trigger? (near-misses — adjacent tasks that seem similar but aren't)
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

Extract common patterns and variations. Look for repeated work across examples — if all three involve the same helper logic, that's a signal to bundle a script.

### Phase 3: Detail

Nail down specifics:

```
Q: Does this skill need external tools?
  1. No, just built-in tools
  2. Yes, CLI tools (specify which)
  3. Yes, APIs (specify which)
  4. Not sure yet
```

Ask:
- What tools does it need? (Read, Write, Bash, Task, etc.)
- External dependencies? (CLIs, APIs)
- What's deterministic vs needs LLM judgment? (deterministic operations → bundle as scripts)
- What must it NEVER do? (hard constraints — but explain why, not just the rule)
- Does it need dynamic context at invocation time? (current branch, project state, etc.)
- Does it bundle supporting files? (scripts, templates, reference docs)

```
Q: How much reasoning effort does this skill need?
  1. Low — simple, procedural steps
  2. High — complex orchestration, judgment calls
  3. Max — deep analysis, architecture decisions
```

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
- Should this persist state or be ephemeral?
- What should NOT be in this skill?
- Should the host agent auto-invoke this, or manual-only? (side effects = manual-only)

### Phase 5: Validate

Before generating, define success:

- "Give me 3 test prompts that SHOULD trigger this skill"
- "Give me 2 prompts that should NOT trigger it"
- "For the first test prompt, what does good output look like?"

Guide the user toward **realistic** test prompts — concrete, with file paths, personal context, casual phrasing, maybe typos. Not abstract ("format this data") but specific ("ok so I have this CSV in ~/Downloads called q4-sales.xlsx and I need to...").

Anti-trigger prompts should be **near-misses**: queries that share keywords with the skill but actually need something different. Obvious irrelevance ("write a fibonacci function") doesn't test anything.

**Hardening (based on skill type from Phase 1):**

For discipline and process skills, also ask:
- "What corners would an agent cut if it were in a rush?" → seeds the anti-rationalization table
- "What does failure look like if the skill is ignored?" → defines what baseline testing should catch

See [references/techniques.md](references/techniques.md) for the full hardening guide. Apply techniques matched to the skill type — discipline skills get the full stack (anti-rationalization, pressure testing, persuasion), technique/reference skills get light validation only.

### Generate

Create SKILL.md from interview answers. Follow these rules:

**Description writing** (most important field — the host agent uses it alone to decide whether to load):
- Write in third person ("Processes files..." not "I can help...")
- Include trigger words matching how users naturally phrase requests
- State WHAT it does AND WHEN to use it
- Be deliberately "pushy" — agents tend to under-trigger. Add phrases like "Use this whenever the user mentions X, even if they don't explicitly ask for it"
- Under 200 characters, but prioritize trigger coverage over brevity

**Instruction language**:
- Use imperative commands, not questions ("Extract the data" not "Can you extract?")
- **Explain the why** behind instructions rather than piling on rigid MUSTs. If the agent understands the reasoning, it handles edge cases better than if it's following rote rules
- Be concrete ("Use UTF-8 encoding") not abstract ("Use appropriate encoding")
- Number workflow steps explicitly
- Include 3-5 input/output examples for non-obvious behavior

**Template**:

```yaml
---
name: skill-name
description: [third-person, trigger-word-rich, WHAT + WHEN, deliberately pushy, under 200 chars]
argument-hint: "[expected arguments]"
allowed-tools: [tools identified in Phase 3]
# effort: high          # uncomment for complex orchestration/reasoning tasks
# disable-model-invocation: true  # uncomment for skills with side effects
---

# /skill-name

[One sentence: what this skill does]

## When to Use

- [Trigger condition 1 from Phase 1]
- [Trigger condition 2]

## When NOT to Use

- [Near-miss anti-pattern 1 from Phase 1]
- [Near-miss anti-pattern 2]

## Workflow

### Step 1: [From Phase 2 examples]
[Imperative instructions with reasoning — explain WHY, not just WHAT]

### Step 2: [...]
[...]

## Examples

[3-5 concrete input/output pairs from Phase 2]

## Error Handling

[From Phase 4 completeness questions]
```

**Dynamic context** — if Phase 3 identified runtime state needs, use shell injection:

```markdown
Current branch: !`git branch --show-current`
Changed files: !`git diff --name-only`
```

**Bundled files** — if Phase 3 identified supporting files or repeated helper logic:

```markdown
See [reference.md](references/reference.md) for details.
Run: ./scripts/validate.sh
```

Bundle scripts when the agent would otherwise regenerate the same helper every invocation. Scripts execute without loading code into context — big token savings.

Target: under 300 lines in SKILL.md. Split to reference files early.

### Post-Generate

After writing the skill:

1. **Review description against test prompts** — would the Phase 5 trigger prompts activate this skill based on the description alone? If not, the description needs more trigger words
2. **Test immediately** — invoke the skill on the first test prompt
3. **Verify anti-triggers** — confirm near-miss prompts don't activate it
4. **Read the transcript** — did the agent follow the skill or drift? If it ignored a section, that section needs rewriting (explain the why) or cutting (it wasn't pulling its weight)
5. Optionally: `/magi "review this skill for clarity and completeness"`

**For discipline/process skills (additional steps):**

6. **Baseline test** — run the scenario without the skill loaded. Note every shortcut, skip, or rationalization. If the agent already does the right thing without the skill, the skill isn't needed
7. **Populate anti-rationalization table** — from baseline observations, add each excuse and counter to the skill
8. **Pressure test** — combine 3+ pressures (time + sunk cost + authority) in a test prompt. If the agent shortcuts under pressure, tighten the gates or add to the rationalization table
9. **Define measurable criteria** — 3-6 binary pass/fail questions for the skill's output. This makes the skill autoresearch-ready for future optimization

## Output Location

Write to the target skill directory's `SKILL.md` (for example `skills/{skill-name}/SKILL.md` or the host agent's skills folder).

Only create reference files if content exceeds 300 lines or has distinct reference material.

## What NOT to Include

- README.md (SKILL.md is the readme)
- CHANGELOG.md (use git)
- LICENSE (inherit from parent)
- Separate config files (use frontmatter)
