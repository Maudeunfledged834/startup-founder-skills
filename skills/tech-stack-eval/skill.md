---
name: tech-stack-eval
trigger: When the user needs to choose between technologies, frameworks, or tools — or says "which framework should I use", "compare X vs Y", "should we migrate from X to Y", "what database should I use".
related: [architecture-design, cicd-setup]
reads: [startup-context]
---

# Tech Stack Evaluation

## When to Use

- The user is choosing a framework, language, database, or cloud provider
- They want to compare technologies or evaluate migration from one to another
- They need a total cost of ownership (TCO) estimate for infrastructure decisions

## Context Required

From `startup-context`: product type, team skills, tech stack, stage, scale, budget. Also ask:
- What problem are you solving? (push back on solution-first thinking)
- Non-negotiable requirements (performance, compliance, team familiarity)
- Team experience with the options and timeline (tight deadlines favor familiar tools)

## Workflow

1. **Clarify the decision** — What exactly is being decided and what are the real requirements? Push back if the user picks tech before defining the problem.
2. **Identify candidates** — List 2-4 realistic options. Exclude clearly wrong choices.
3. **Define evaluation criteria** — Select 6-8 weighted criteria from the master list below.
4. **Score each candidate** — Rate 1-5 on each criterion with justification.
5. **Assess ecosystem health** — Maintenance activity, community size, trajectory.
6. **Estimate TCO** — For infra decisions, project 12-month cost including engineering time.
7. **Analyze migration path** — If migrating, estimate effort, risk, and phased approach.
8. **Deliver recommendation** — Clear winner with rationale. No "it depends" without follow-up.

## Output Format

```markdown
# Tech Stack Evaluation: [Decision Title]

## Decision Context
What we're choosing and why it matters.

## Candidates
| Technology | Version | License | One-Liner |
|-----------|---------|---------|-----------|

## Evaluation Criteria
| Criterion | Weight | Why It Matters |
|-----------|--------|----------------|

## Scoring Matrix
| Criterion (weight) | Option A | Option B | Option C |
|---------------------|----------|----------|----------|
| ... | 4/5 | 3/5 | 5/5 |
| **Weighted Total** | **X.X** | **X.X** | **X.X** |

## Ecosystem Health
| Metric | Option A | Option B |
|--------|----------|----------|
(GitHub stars, weekly downloads, last release, open issues, major users)

## TCO Estimate (12 months)
| Cost Category | Option A | Option B |
|---------------|----------|----------|

## Recommendation
Clear winner and why, with caveats. Migration path if applicable.
```

## Frameworks & Best Practices

### Master Evaluation Criteria

Select 6-8 of these and assign weights (total = 100%):

- **Performance** — throughput, latency, resource efficiency
- **Developer Experience** — tooling, debugging, documentation
- **Learning Curve / Team Familiarity** — time to productivity
- **Ecosystem & Libraries** — packages, integrations, support
- **Maintenance & Longevity** — release cadence, backing, bus factor
- **Hiring Pool** — developer availability
- **Scalability** — handle 10-100x growth without rewrite
- **Cost / Vendor Lock-in** — TCO and switching cost

### Ecosystem Health Scoring

Rate ecosystem health on a 3-tier scale:

- **Thriving** — regular releases (< 3 months), growing adoption, multiple corporate sponsors, active community
- **Stable** — regular releases (< 6 months), steady adoption, established community, no signs of decline
- **At Risk** — infrequent releases (> 12 months), declining adoption, key maintainers leaving, few contributors

### TCO Calculation Framework

Estimate over 12 months: compute, storage, bandwidth, licensing, engineering time (setup + ongoing maintenance hours x loaded cost), and operational overhead (monitoring, on-call). **Engineering time is usually the largest cost for startups** -- a technology saving $200/month on hosting but costing 40 extra engineering hours to operate is a net loss.

### Migration Risk Assessment

| Risk Level | Criteria |
|------------|----------|
| **Low** | Additive change, no data migration, can run in parallel, < 2 weeks |
| **Medium** | Requires data migration or API changes, 2-8 weeks, can be phased |
| **High** | Core system replacement, > 8 weeks, requires downtime or big-bang cutover |

Use the strangler fig pattern for migrations: route new traffic to the new system, migrate old incrementally. Always maintain rollback capability. Set a concrete cut-off date -- half-migrated systems are the worst outcome.

### Common Decision Anti-Patterns

- **Resume-Driven Development** — choosing tech for resumes, not fit
- **Hype Cycle Trap** — adopting at peak hype before proven stability
- **Premature Optimization** — distributed systems when a single Postgres handles the load
- **Sunk Cost Fallacy** — refusing to migrate because of prior investment
- **Ignoring Team Skills** — picking the "best" tool nobody on the team knows

## Related Skills

- `architecture-design` — chain when the tech stack decision feeds into a broader system design
- `cicd-setup` — chain to configure CI/CD for the chosen technology

## Examples

**Example prompt:** "Should we use PostgreSQL or MongoDB for our B2B SaaS app?"

**Good output snippet:**
```
## Recommendation: PostgreSQL

For B2B SaaS with relational data (users, orgs, permissions), PostgreSQL
is the clear choice. Your data is inherently relational — modeling in
MongoDB requires denormalization that creates update anomalies. JSONB
gives schema flexibility without giving up transactions. Ecosystem and
hiring pool are 3x larger than MongoDB.
```

**Example prompt:** "We're on Heroku at $2,400/mo. Should we migrate to AWS?"

**Good output snippet:**
```
## TCO Estimate (12 months)
| Category | Heroku | AWS |
|----------|--------|-----|
| Compute | $1,200/mo | $480/mo (ECS Fargate) |
| Database | $800/mo | $350/mo (RDS) |
| Add-ons | $400/mo | $120/mo |
| Engineering (setup) | $0 | $12,000 one-time |
| Engineering (ongoing) | 2 hrs/mo | 8 hrs/mo |
| **Annual Total** | **$28,800** | **$18,000** |

Break-even at month 14. At Series A with team of 6, wait until Heroku
hits $4,000/mo — engineering hours are better spent on product now.
```
