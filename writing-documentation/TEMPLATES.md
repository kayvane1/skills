# Documentation Templates

Pick the right doc type, then use the matching template. Apply the principles in [SKILL.md](SKILL.md) as you fill it in.

## Choosing a doc type

| Need | Use |
|---|---|
| Project intro + quick start + nav hub | **README.md** |
| 4+ recurring issues, debugging procedures | **TROUBLESHOOTING.md** |
| Multi-step deployment / bootstrap | **DEPLOYMENT_GUIDE.md** |
| Operational procedure (incident, maintenance) | **runbooks/<operation>.md** |
| REST / library API reference | **API_REFERENCE.md** |
| Architectural decision with trade-offs | **docs/adr/NNNN-title.md** |
| System design at a higher level | **ARCHITECTURE.md** |

**Default:** start with README.md. Split out specialised docs only when the README grows past ~300 lines or a section needs more than ~100 lines of detail.

## README.md

```markdown
# <Project name>

<One-sentence description.>

## What it does

<2–3 sentences: the problem this solves and the rough approach.>

## Quick start

### Prerequisites

- <Tool / version>

### Install

\`\`\`bash
<commands>
\`\`\`

### Use

\`\`\`bash
<minimal working example>
\`\`\`

## How it works

<High-level architecture: components and how they connect. Diagram if useful.>

## Development

\`\`\`bash
<run, test, lint commands>
\`\`\`

## Configuration

| Variable | Description | Default |
|---|---|---|
| `<VAR>` | <what it controls> | `<default>` |

## Links

- <Deeper docs, runbooks, ADRs as relevant>
```

## TROUBLESHOOTING.md

```markdown
# Troubleshooting

## <Issue title — state the symptom>

**Symptoms**
- <What the user sees>
- <Error message verbatim>

**Cause**
<Why it happens — one or two sentences.>

**Fix**
1. <Step>
2. <Step>
3. <Verification>

**Prevent**
<How to avoid recurrence.>

---

## <Next issue>
…
```

## DEPLOYMENT_GUIDE.md

```markdown
# Deployment Guide — <Target>

## Overview

<What this deployment accomplishes. When to use it (vs. routine deploys).>

## Prerequisites

- [ ] <Permission / access>
- [ ] <Tool / version>

## Pre-flight checks

- [ ] <Verify state X>
- [ ] <Backup Y>
- [ ] <Notify stakeholders>

## Steps

### 1. <Action>

**Why:** <one sentence>

\`\`\`bash
<exact command>
\`\`\`

**Expect:**
\`\`\`
<expected output>
\`\`\`

**Verify:** \`<verification command>\`

### 2. <Action>
…

## Post-deploy verification

1. <Check>
2. <Check>

## Rollback

If anything in the verification fails:

1. <Rollback step>
2. <Verification of rollback>
```

## runbooks/<operation>.md

```markdown
# Runbook: <Operation>

**Purpose:** <What this achieves.>
**When:** <Scenarios that trigger this runbook.>
**Duration:** <Estimate.>
**Risk:** Low / Medium / High

## Prerequisites

- <Permissions required>
- <Access to systems / tools>

\`\`\`bash
<pre-check command>
\`\`\`

## Procedure

### 1. <Action>

\`\`\`bash
<command>
\`\`\`

**Expect:** <expected output>

**If it fails:** <recovery steps>

### 2. <Action>
…

## Verification

- [ ] <Check>
- [ ] <Check>

## Rollback

<Steps if procedure fails partway.>
```

## API_REFERENCE.md

```markdown
# API Reference

## Auth

<How to authenticate. Include header or bearer-token format.>

## Base URL

\`https://api.example.com/v1\`

## Endpoints

### \`GET /resource\`

<One-sentence purpose.>

**Parameters**

| Name | In | Type | Required | Description |
|---|---|---|---|---|
| `id` | path | string | yes | <…> |
| `limit` | query | int | no | <…> |

**Example**

\`\`\`bash
curl https://api.example.com/v1/resource/123
\`\`\`

**Response 200**

\`\`\`json
{ "status": "success", "data": { … } }
\`\`\`

**Status codes**

| Code | Meaning |
|---|---|
| 200 | OK |
| 400 | Bad request |
| 401 | Unauthorized |
| 429 | Rate-limited |

## Errors

<Error response shape. Common cases and how to recover.>

## Rate limits

<Policy, headers returned, recommended client behaviour.>
```

## docs/adr/NNNN-title.md (Architectural Decision Record)

```markdown
# ADR-NNNN: <Title stating the decision>

**Status:** Proposed | Accepted | Superseded by ADR-XXXX
**Date:** YYYY-MM-DD

## Context

<The problem, constraints, and forces in play. Why a decision is needed now.>

## Decision

<The chosen option, stated as an active sentence: "We will use X for Y.">

## Consequences

**Positive**
- <…>

**Negative / trade-offs**
- <…>

**Follow-ups**
- <Actions this decision enables or requires.>

## Alternatives considered

### <Option A>
<Why rejected.>

### <Option B>
<Why rejected.>
```

## ARCHITECTURE.md

```markdown
# Architecture

## What this system does

<One paragraph. The conclusion, not the journey.>

## Components

<Diagram or list with one-line purposes.>

| Component | Purpose | Tech |
|---|---|---|
| <name> | <…> | <…> |

## Request flow

<For a typical operation, what happens step by step. Sequence diagram if useful.>

## Data

<Storage choices, shapes, retention.>

## Cross-cutting concerns

- **Auth:** <…>
- **Observability:** <logging, metrics, tracing>
- **Failure modes:** <…>

## See also

- <ADRs that shaped this>
- <Runbooks for ops>
```
