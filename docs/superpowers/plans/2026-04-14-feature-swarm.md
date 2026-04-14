# Feature Swarm Skill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a custom Claude Code skill that spawns a swarm of parallel agents to discover, research, and document features for a new app or project.

**Architecture:** A single SKILL.md orchestrates four phases — discovery (3 parallel agents), selection (tiered checklist), research (1 agent per feature in parallel), and output (markdown files). Prompt templates live in separate files for editability.

**Tech Stack:** Claude Code skills framework (markdown), Agent tool for parallelism, AskUserQuestion for selection, WebSearch for competitive research, Write/Read tools for file I/O.

---

## File Structure

```
~/.claude/skills/feature-swarm/
├── SKILL.md                           # Main skill file — orchestration logic
├── prompts/
│   ├── competitor-analyst.md          # Discovery agent 1 prompt template
│   ├── user-journey-explorer.md       # Discovery agent 2 prompt template
│   ├── technical-capability-scout.md  # Discovery agent 3 prompt template
│   └── feature-researcher.md         # Phase 3 research agent prompt template
```

All files are markdown — this is a skill (guidance document), not application code. "Testing" means running the skill end-to-end and verifying it produces correct output.

---

### Task 1: Create directory structure

**Files:**
- Create: `~/.claude/skills/feature-swarm/prompts/` (directory)

- [ ] **Step 1: Create the skill directory and prompts subdirectory**

Run:
```bash
mkdir -p ~/.claude/skills/feature-swarm/prompts
```

- [ ] **Step 2: Verify structure**

Run:
```bash
ls -la ~/.claude/skills/feature-swarm/
```
Expected: empty directory with `prompts/` subdirectory

- [ ] **Step 3: Commit**

```bash
cd ~/.claude/skills/feature-swarm
git init
git add .
git commit -m "chore: initialize feature-swarm skill directory structure"
```

---

### Task 2: Write the Competitor Analyst prompt template

**Files:**
- Create: `~/.claude/skills/feature-swarm/prompts/competitor-analyst.md`

- [ ] **Step 1: Write the prompt template**

Write the following to `~/.claude/skills/feature-swarm/prompts/competitor-analyst.md`:

```markdown
# Competitor Analyst Agent

You are researching competing products for a new app. Your goal is to find existing apps and products that solve similar problems and identify what features they offer.

## App Description

{{APP_DESCRIPTION}}

## Instructions

1. Use WebSearch to find 3-5 existing products or apps that solve a similar problem to the one described above
2. For each product, identify the key features they offer
3. Note pricing models, target audiences, and differentiators where available
4. Compile a unified list of unique features found across all competitors

## Output Format

Return your findings as a structured list. Use this exact format for each feature:

FEATURE: [Feature Name]
DESCRIPTION: [One sentence describing what this feature does for the user]
FOUND_IN: [Which competitor products offer this feature]
NOTES: [Relevant context such as UX patterns, pricing implications, or user reception]

List ALL distinct features you find across all competitors. Do not filter or prioritize — that happens in a later phase.

## Constraints

- You MUST use WebSearch to find real products. Do not invent or fabricate competitor names.
- If you cannot find relevant competitors for this specific niche, say so explicitly, then brainstorm likely features based on the app description and adjacent products.
- Keep each field to one line. Be concise.
- Aim for 8-15 features total across all competitors.
```

- [ ] **Step 2: Verify the file was written correctly**

Run:
```bash
cat ~/.claude/skills/feature-swarm/prompts/competitor-analyst.md | head -5
```
Expected: first 5 lines match the template header

- [ ] **Step 3: Commit**

```bash
git -C ~/.claude/skills/feature-swarm add prompts/competitor-analyst.md
git -C ~/.claude/skills/feature-swarm commit -m "feat: add competitor analyst prompt template"
```

---

### Task 3: Write the User Journey Explorer prompt template

**Files:**
- Create: `~/.claude/skills/feature-swarm/prompts/user-journey-explorer.md`

- [ ] **Step 1: Write the prompt template**

Write the following to `~/.claude/skills/feature-swarm/prompts/user-journey-explorer.md`:

```markdown
# User Journey Explorer Agent

You are mapping user journeys for a new app. Your goal is to think from the end-user's perspective and identify every feature implied by their workflows.

## App Description

{{APP_DESCRIPTION}}

## Instructions

1. Identify 2-4 primary user personas who would use this app (e.g., new user, power user, admin)
2. For each persona, map their key workflows across these stages:
   - **Onboarding:** First-time experience, sign-up, setup, initial configuration
   - **Core actions:** The primary things they do daily or weekly
   - **Management:** Settings, preferences, account, data management
   - **Edge cases:** Error handling, recovery, help, support, offline behavior
3. From these workflows, extract every feature that would be needed to support them

## Output Format

Return your findings as a structured list. Use this exact format for each feature:

FEATURE: [Feature Name]
DESCRIPTION: [One sentence describing what this feature does for the user]
PERSONA: [Which persona(s) need this — e.g., "all users", "admin only", "new users"]
JOURNEY: [Which stage this belongs to — onboarding, core, management, or edge case]

List ALL features implied by the user journeys. Include both obvious features (login, dashboard) and non-obvious ones (forgot password, empty states, loading indicators). Do not filter or prioritize — that happens in a later phase.

## Constraints

- Think from the user's perspective, not the developer's. Ask "what would the user need?" not "what's easy to build?"
- Include features for error states and unhappy paths, not just the golden path
- Keep each field to one line. Be concise.
- Aim for 10-20 features total across all personas and journeys.
```

- [ ] **Step 2: Verify the file was written correctly**

Run:
```bash
cat ~/.claude/skills/feature-swarm/prompts/user-journey-explorer.md | head -5
```
Expected: first 5 lines match the template header

- [ ] **Step 3: Commit**

```bash
git -C ~/.claude/skills/feature-swarm add prompts/user-journey-explorer.md
git -C ~/.claude/skills/feature-swarm commit -m "feat: add user journey explorer prompt template"
```

---

### Task 4: Write the Technical Capability Scout prompt template

**Files:**
- Create: `~/.claude/skills/feature-swarm/prompts/technical-capability-scout.md`

- [ ] **Step 1: Write the prompt template**

Write the following to `~/.claude/skills/feature-swarm/prompts/technical-capability-scout.md`:

```markdown
# Technical Capability Scout Agent

You are exploring what modern technology makes possible for a new app. Your goal is to identify features that are enabled or made easy by current APIs, libraries, and services — features the user might not have considered.

## App Description

{{APP_DESCRIPTION}}

## Instructions

1. Use WebSearch to research relevant APIs, libraries, and cloud services for this type of app
2. Identify features that are made easy or newly possible by available technology (e.g., "AI-powered search is straightforward with Algolia or Typesense", "real-time collaboration is easy with Yjs or Liveblocks")
3. Flag any technical constraints the user should know about (rate limits, costs, scaling concerns)
4. Note which specific technologies enable each feature

## Output Format

Return your findings as a structured list. Use this exact format for each feature:

FEATURE: [Feature Name]
DESCRIPTION: [One sentence describing what this feature does for the user]
ENABLED_BY: [Specific API, library, or service that makes this possible — use real names]
COMPLEXITY: [simple / moderate / complex]
NOTES: [Why this is worth considering — cost estimate, performance characteristics, or user value]

List ALL technically-enabled features you discover. Focus on features that technology makes easy or that the user might not know are feasible. Do not filter or prioritize — that happens in a later phase.

## Constraints

- You MUST use WebSearch to find real, current technologies. Do not fabricate library names or APIs.
- Focus on features ENABLED by technology, not basic CRUD operations every app has.
- If a feature is technically possible but impractical (too expensive, too slow, too complex), include it but note why in NOTES.
- Keep each field to one line. Be concise.
- Aim for 8-15 features.
```

- [ ] **Step 2: Verify the file was written correctly**

Run:
```bash
cat ~/.claude/skills/feature-swarm/prompts/technical-capability-scout.md | head -5
```
Expected: first 5 lines match the template header

- [ ] **Step 3: Commit**

```bash
git -C ~/.claude/skills/feature-swarm add prompts/technical-capability-scout.md
git -C ~/.claude/skills/feature-swarm commit -m "feat: add technical capability scout prompt template"
```

---

### Task 5: Write the Feature Researcher prompt template

**Files:**
- Create: `~/.claude/skills/feature-swarm/prompts/feature-researcher.md`

- [ ] **Step 1: Write the prompt template**

Write the following to `~/.claude/skills/feature-swarm/prompts/feature-researcher.md`:

```markdown
# Feature Research Agent

You are conducting deep research on one specific feature for a new app. Your job is to produce both competitive analysis (how others do it) and technical feasibility assessment (how to build it).

## App Description

{{APP_DESCRIPTION}}

## Feature to Research

- **Name:** {{FEATURE_NAME}}
- **Description:** {{FEATURE_DESCRIPTION}}
- **Tier:** {{FEATURE_TIER}}
- **Discovery Notes:** {{DISCOVERY_NOTES}}

## Instructions

### Part 1: Competitive Analysis

1. Use WebSearch to find 2-3 existing products that implement this specific feature
2. For each product, note: how they implement it, what works well, what users complain about
3. Identify common UX patterns users expect for this type of feature

### Part 2: Technical Feasibility

1. Research the recommended approach or architecture for building this feature
2. Identify specific libraries, APIs, or services to use (with real names)
3. Assess overall complexity: simple, moderate, or complex
4. Note dependencies on other features if any

### Part 3: Recommendations

1. Define the MVP scope — the minimum version of this feature worth building
2. Identify what to explicitly skip or defer to a later version
3. Flag potential pitfalls or common mistakes

## Output Format

Return your research as a markdown document using this exact structure:

# Feature: [Feature Name]
## Tier: [MVP / Nice-to-Have / Future]

## Competitive Analysis
- **[Product 1]:** [How they implement it. What works, what doesn't.]
- **[Product 2]:** [How they implement it. What works, what doesn't.]
- **Common UX patterns:** [What users expect from this type of feature]

## Technical Feasibility
- **Recommended approach:** [Architecture or implementation strategy]
- **Libraries/APIs:** [Specific packages or services, with version info if relevant]
- **Complexity:** [simple / moderate / complex] — [one sentence justification]
- **Dependencies:** [Other features this depends on, or "None"]

## Recommendations
- **MVP scope:** [The minimum viable version of this feature]
- **Defer:** [What to skip in v1]
- **Pitfalls:** [Common mistakes or things to watch out for]

## Constraints

- Use WebSearch for real competitive data. Do not fabricate product names or reviews.
- If you cannot find competitive data for this specific feature, say "No direct competitors found" and focus on technical feasibility.
- Be specific: name actual libraries, actual products, actual UX patterns.
- Keep total output to 300-500 words. Be dense, not verbose.
```

- [ ] **Step 2: Verify the file was written correctly**

Run:
```bash
cat ~/.claude/skills/feature-swarm/prompts/feature-researcher.md | head -5
```
Expected: first 5 lines match the template header

- [ ] **Step 3: Commit**

```bash
git -C ~/.claude/skills/feature-swarm add prompts/feature-researcher.md
git -C ~/.claude/skills/feature-swarm commit -m "feat: add feature researcher prompt template"
```

---

### Task 6: Write the main SKILL.md

This is the orchestration file — the brain of the skill. It tells Claude exactly what to do at each phase.

**Files:**
- Create: `~/.claude/skills/feature-swarm/SKILL.md`

- [ ] **Step 1: Write the SKILL.md file**

Write the following to `~/.claude/skills/feature-swarm/SKILL.md`:

````markdown
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
  docs/research/features/           — Individual feature research files

To start building, open a new session and say:
  "Read docs/research/project-brief.md and create an implementation plan"
```

## Error Handling

- If a discovery agent returns no features: proceed with results from the other agents. If ALL three return nothing, tell the user and ask them to provide a more detailed app description.
- If a research agent fails or returns empty: write a placeholder file noting the failure and continue with other features. Do not retry or block the entire output.
- If the user provides an ambiguous selection: ask for clarification. Do not guess.
````

- [ ] **Step 2: Verify the SKILL.md frontmatter is correct**

Run:
```bash
head -4 ~/.claude/skills/feature-swarm/SKILL.md
```
Expected:
```
---
name: feature-swarm
description: Use when the user wants to discover and research features for a new app or project by spawning parallel research agents
---
```

- [ ] **Step 3: Commit**

```bash
git -C ~/.claude/skills/feature-swarm add SKILL.md
git -C ~/.claude/skills/feature-swarm commit -m "feat: add main SKILL.md orchestration file"
```

---

### Task 7: End-to-end smoke test

**Files:**
- None (testing only)

- [ ] **Step 1: Verify all files exist**

Run:
```bash
find ~/.claude/skills/feature-swarm -type f | sort
```
Expected output:
```
/Users/lucatescari/.claude/skills/feature-swarm/SKILL.md
/Users/lucatescari/.claude/skills/feature-swarm/prompts/competitor-analyst.md
/Users/lucatescari/.claude/skills/feature-swarm/prompts/feature-researcher.md
/Users/lucatescari/.claude/skills/feature-swarm/prompts/technical-capability-scout.md
/Users/lucatescari/.claude/skills/feature-swarm/prompts/user-journey-explorer.md
```

- [ ] **Step 2: Verify skill is discoverable**

Start a new Claude Code session and check if `/feature-swarm` appears in the available skills. If it does not, check these potential issues:
- Custom skills may need to be in the project's `.claude/skills/` directory instead of `~/.claude/skills/`
- The skill may need a different registration mechanism
- Try moving to `.claude/skills/feature-swarm/SKILL.md` (project-level) if user-level is not discovered

- [ ] **Step 3: Run a smoke test**

Invoke the skill with a simple test input:
```
/feature-swarm I want to build a simple habit tracking app where users can create daily habits, mark them complete, and see their streaks
```

Verify each phase executes:
1. Three discovery agents are dispatched in parallel
2. A merged, tiered feature list is presented as a checklist
3. After selection, research agents are dispatched (one per feature)
4. Output files are written to `docs/research/`
5. Handoff message is printed

- [ ] **Step 4: Verify output structure**

After the smoke test completes, check:
```bash
find docs/research -type f | sort
```
Expected: `README.md`, `project-brief.md`, and numbered feature files in `features/`

- [ ] **Step 5: Commit the test output (optional)**

If the test output is worth keeping as an example:
```bash
git add docs/research/
git commit -m "docs: add smoke test research output as example"
```

Otherwise discard:
```bash
rm -rf docs/research/
```
