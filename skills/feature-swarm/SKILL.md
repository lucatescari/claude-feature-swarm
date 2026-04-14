---
name: feature-swarm
description: Use when the user wants to discover and research features for a new app or project by spawning parallel research agents
---

# Feature Swarm

Spawn a swarm of parallel agents to discover, research, and document features for a new app or project.

**Announce at start:** "Using Feature Swarm to discover and research features for your project."

## Input

The user provides an app description either as an argument to `/feature-swarm` or when prompted.

If no app description was provided as an argument, ask the user using AskUserQuestion:
> "What app or project do you want to research features for? Describe it in a sentence or two."

Store the app description — it is used in every phase.

## Phase 1: Discovery (3 Parallel Agents)

Read the three discovery prompt templates from this skill's `prompts/` directory:
- `prompts/competitor-analyst.md`
- `prompts/user-journey-explorer.md`
- `prompts/technical-capability-scout.md`

For each template, replace `{{APP_DESCRIPTION}}` with the user's app description.

Dispatch all three agents **in a single message** using the Agent tool so they run in parallel:

```
Agent({
  description: "Competitor analysis for feature discovery",
  prompt: [contents of competitor-analyst.md with {{APP_DESCRIPTION}} replaced]
})

Agent({
  description: "User journey mapping for feature discovery",
  prompt: [contents of user-journey-explorer.md with {{APP_DESCRIPTION}} replaced]
})

Agent({
  description: "Technical capability scouting for feature discovery",
  prompt: [contents of technical-capability-scout.md with {{APP_DESCRIPTION}} replaced]
})
```

**CRITICAL:** All three MUST be dispatched in a single message for parallel execution. Do NOT dispatch them sequentially.

Wait for all three agents to return before proceeding.

## Phase 2: Merge, Tier, and Select

### Step 2a: Merge and Deduplicate

Collect all features from all three agents. Group features that describe the same capability even if named differently (e.g., "User Login" from the competitor analyst and "Authentication" from the user journey explorer are the same feature).

For each unique feature, record:
- **Canonical name** — pick the clearest, most descriptive name
- **Description** — one-line summary combining the best description from all agents
- **Agent count** — how many of the 3 agents independently surfaced this (1/3, 2/3, or 3/3)
- **Notes** — any relevant context from the discovery agents (combine the most useful notes)

### Step 2b: Assign Tiers

Categorize each feature into one of three tiers:

**MVP / Must-Have:**
- Core to the stated app purpose — the app doesn't work without it
- Surfaced by 2-3 agents (high consensus)
- Low-to-moderate complexity

**Nice-to-Have:**
- Enhances the experience but the app functions without it
- May have been surfaced by 1-2 agents
- Any complexity level

**Future / Stretch:**
- Ambitious, requires significant infrastructure, or depends heavily on other features
- Often surfaced by only 1 agent (low consensus)
- Typically high complexity

### Step 2c: Estimate Complexity

Rate each feature:
- `*` — **Simple:** Standard patterns, off-the-shelf solutions, well-documented approaches
- `**` — **Moderate:** Requires integration work, some custom logic, multiple components
- `***` — **Complex:** Requires ML/AI, complex infrastructure, significant custom engineering, or novel approaches

### Step 2d: Present Checklist

Use AskUserQuestion to present the tiered feature list. Format it exactly like this:

```
DISCOVERED FEATURES FOR: "[app description]"

-- MVP / Must-Have --
[1] [Feature Name]              [* / ** / ***] complexity | [N]/3 agents
    [one-line description]
[2] [Feature Name]              [* / ** / ***] complexity | [N]/3 agents
    [one-line description]

-- Nice-to-Have --
[3] [Feature Name]              [* / ** / ***] complexity | [N]/3 agents
    [one-line description]

-- Future / Stretch --
[4] [Feature Name]              [* / ** / ***] complexity | [N]/3 agents
    [one-line description]

Select features: type numbers (e.g. "1,3,5"), "all", or "mvp"
```

Numbers are sequential across all tiers (MVP features first, then Nice-to-Have, then Future).

### Step 2e: Parse Selection

Handle the user's response:
- **Comma-separated numbers** (e.g., "1,3,5"): Select those specific features
- **"all"**: Select every feature
- **"mvp"**: Select all features in the MVP / Must-Have tier
- **Anything else**: Ask for clarification — repeat the selection prompt

## Phase 3: Deep Research (1 Agent Per Feature)

Read the research prompt template from `prompts/feature-researcher.md`.

For each selected feature, prepare an agent prompt by replacing ALL placeholders in the template:
- `{{APP_DESCRIPTION}}` — the user's original app description
- `{{FEATURE_NAME}}` — the canonical feature name
- `{{FEATURE_DESCRIPTION}}` — the one-line description
- `{{FEATURE_TIER}}` — the assigned tier (MVP / Nice-to-Have / Future)
- `{{DISCOVERY_NOTES}}` — combined notes from all discovery agents that surfaced this feature

Dispatch all research agents **in a single message** for parallel execution:

```
Agent({
  description: "Research: [Feature Name 1]",
  prompt: [feature-researcher.md with all placeholders replaced for feature 1]
})

Agent({
  description: "Research: [Feature Name 2]",
  prompt: [feature-researcher.md with all placeholders replaced for feature 2]
})

// ... one agent per selected feature
```

**Batching rule:** If more than 8 features are selected, dispatch in batches of 8. Wait for each batch to complete before dispatching the next. Print a progress update between batches:
> "Batch 1/N complete. Researching next batch..."

## Phase 4: Output

After all research agents return, create the output files.

### Step 4a: Create Directory

```bash
mkdir -p docs/research/features
```

### Step 4b: Write Individual Feature Files

For each researched feature, write the agent's research output to `docs/research/features/NN-feature-name.md`.

Numbering rules:
- Two-digit prefix: `01-`, `02-`, etc.
- Ordered by tier: all MVP features first, then Nice-to-Have, then Future
- Within a tier, order by agent consensus (3/3 before 2/3 before 1/3)
- Convert feature name to kebab-case for the filename (e.g., "User Authentication" becomes `01-user-authentication.md`)

### Step 4c: Write README.md (Master Index)

Write to `docs/research/README.md`:

```markdown
# Feature Research: [App Description]

**Generated:** [YYYY-MM-DD]
**Features researched:** [N]

## Features

| # | Feature | Tier | Complexity | Consensus | Details |
|---|---------|------|------------|-----------|---------|
| 1 | [Name] | MVP | ** | 3/3 | [View](features/01-name.md) |
| 2 | [Name] | MVP | * | 2/3 | [View](features/02-name.md) |
| ... | ... | ... | ... | ... | ... |

## How to Use This Research

- Browse individual feature files in `features/` for detailed competitive analysis and technical feasibility
- See [project-brief.md](project-brief.md) for a structured summary optimized for AI-assisted planning
- To start building: open a new Claude session and say "Read docs/research/project-brief.md and create an implementation plan"
```

### Step 4d: Write project-brief.md (Structured Brief)

Write to `docs/research/project-brief.md`:

```markdown
# Project Brief: [App Description]

**Generated:** [YYYY-MM-DD]

## App Overview

[The user's original app description, expanded with context discovered during the research phase. 2-3 sentences.]

## Selected Features

### MVP / Must-Have
- **[Feature 1]** — [one-line summary] | Complexity: [* / ** / ***]
- **[Feature 2]** — [one-line summary] | Complexity: [* / ** / ***]

### Nice-to-Have
- **[Feature N]** — [one-line summary] | Complexity: [* / ** / ***]

### Future / Stretch
- **[Feature N]** — [one-line summary] | Complexity: [* / ** / ***]

## Tech Stack Recommendations

[Aggregate the libraries, APIs, and services recommended across all feature research. Group by category (e.g., "Authentication", "Database", "Real-time", "AI/ML"). Only include technologies that were recommended by research agents — do not add your own suggestions.]

## Feature Dependencies

[List features that depend on other features being built first. Format as: "Feature X requires Feature Y because [reason]". Only include real dependencies identified in the research, not speculative ones.]

## Suggested Implementation Order

[Based on tier priority and dependencies, recommend a build sequence:]
1. [Feature] — [why first: no dependencies, foundational, etc.]
2. [Feature] — [why next: depends on #1, high priority, etc.]
3. ...
```

### Step 4e: Print Handoff Message

Print this message to the user:

```
Research complete! [N] features researched and saved to docs/research/.

Files created:
  docs/research/README.md          — Master index with links to all features
  docs/research/project-brief.md   — Structured brief for AI-assisted planning
  docs/research/features/          — Individual feature research files

To start building, open a new session and say:
  "Read docs/research/project-brief.md and create an implementation plan"
```

## Error Handling

- If a discovery agent returns no features: proceed with results from the other agents. If ALL three return nothing, tell the user and ask them to provide a more detailed app description.
- If a research agent fails or returns empty: write a placeholder file noting the failure and continue with other features. Do not retry or block the entire output.
- If the user provides an ambiguous selection: ask for clarification. Do not guess.
