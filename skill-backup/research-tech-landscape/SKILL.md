---
name: research-tech-landscape
description: >
  Use this skill to map the current state of a technology ecosystem — maturity, adoption,
  community health, major players, and trajectory. Trigger when the user asks questions like
  "what's the current state of X?", "is X mature enough for production?", "who are the main
  players in X space?", "how widely adopted is X?", or any request to evaluate a technology
  before committing to it. Also trigger when comparing technology generations (e.g., REST vs
  GraphQL, SQL vs NoSQL) or assessing whether a tech is worth learning/adopting.
---

# Research: Technology Landscape

## Purpose
Map an ecosystem before the team commits to it. Answer: is this technology ready, who is using it, where is it going, and what are we betting on?

## Signals to investigate

### Adoption
- GitHub stars trajectory (not just total — look at star history rate)
- npm/PyPI/crates.io weekly download counts and trend direction
- Stack Overflow survey data for the relevant year
- Job postings mentioning the technology (proxy for enterprise adoption)
- Major companies publicly using it (check official adopters pages, case studies, tech blogs)

### Community Health
- GitHub: open issues vs closed ratio, PR merge velocity, time-to-first-response
- Discord/Slack/Forum activity — posts per week, response rate
- Number of active contributors (not just stars) — bus factor risk
- Last commit date and release cadence

### Maturity Signals
- Version number (pre-1.0 = unstable API, post-2.0 = likely breaking changes happened already)
- Changelog quality — is breaking change communication clear?
- Documented migration guides between major versions
- Production case studies from companies at relevant scale

### Trajectory
- Roadmap or RFC process (do they have one?)
- Sponsorship: backed by a company, foundation (CNCF, Apache, Linux Foundation), or purely community?
- Known deprecation risks or competing successors inside the same ecosystem

## Investigation steps

```bash
# Check npm download trend (for JS/TS packages)
curl "https://api.npmjs.org/downloads/range/last-year/<package>" | jq '.downloads[-4:] | .[].downloads'

# Check PyPI stats
pip index versions <package> 2>/dev/null | head -5

# Check GitHub metadata via CLI (if gh is available)
gh repo view <owner>/<repo> --json stargazerCount,forkCount,openIssues,updatedAt

# Check release cadence
gh release list -R <owner>/<repo> --limit 10
```

## Output format
Produce a landscape section in the research report with:

| Signal | Finding | Interpretation |
|--------|---------|----------------|
| Stars (total / 6mo trend) | ... | ... |
| Downloads / week | ... | ... |
| Last release | ... | ... |
| Backing | ... | ... |
| Enterprise adopters | ... | ... |
| Community size | ... | ... |

Follow with a **Maturity Verdict**: `Experimental` / `Growing` / `Stable` / `Mature` / `Legacy` and a one-paragraph justification.

## Constraints
- Always note the **date of data collection** — ecosystem signals age fast
- Do not confuse hype (social media attention) with adoption (production usage)
- Flag if the only adoption evidence is tutorials and blog posts — not the same as production case studies
