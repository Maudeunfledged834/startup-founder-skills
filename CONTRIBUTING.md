# Contributing

Thanks for helping improve Startup Founder Skills! Here's how to add or improve skills.

## Skill File Structure

Each skill lives in its own directory under `skills/` and contains a `SKILL.md` file:

```
skills/
  your-skill-name/
    SKILL.md
```

## Skill Template

```markdown
---
name: your-skill-name
trigger: When to activate this skill (1-2 sentences)
related: [skill-a, skill-b]
reads: [startup-context]
---

# Skill Name

## When to Use
Describe the trigger conditions — what the user says or does that should activate this skill.

## Context Required
What information this skill needs from `startup-context` or the user before it can run.

## Workflow
Step-by-step process the agent should follow.

1. **Step one** — what to do first
2. **Step two** — what comes next
3. **Step three** — etc.

## Output Format
Describe what the final output should look like — structure, sections, length.

## Frameworks & Best Practices
Domain knowledge, mental models, and guardrails for the agent.

## Related Skills
- `skill-a` — when to chain to this skill
- `skill-b` — when to chain to this skill

## Examples
Show 1-2 example prompts and what good output looks like.
```

## Guidelines

1. **Be specific** — vague skills produce vague output. Include concrete frameworks and templates.
2. **Reference startup-context** — every skill should read from the shared context first.
3. **Cross-reference related skills** — help the agent know when to chain skills together.
4. **Include trigger conditions** — be precise about when the skill should activate.
5. **Test with real prompts** — try your skill with actual founder tasks before submitting.

## Submitting a PR

1. Fork the repo
2. Create your skill directory and `SKILL.md`
3. Add your skill to the table in `README.md` (alphabetical within its category)
4. Open a PR with a brief description of the skill and why it's useful

## Improving Existing Skills

Found a skill that could be better? Open a PR with:
- What the current behavior is
- What the improved behavior should be
- Why the change matters
