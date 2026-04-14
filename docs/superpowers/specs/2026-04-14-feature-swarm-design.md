# Feature Swarm Skill — Design Spec

**Date:** 2026-04-14
**Status:** Draft
**Approach:** Single Superpowers Skill (Approach 1)

---

## Overview

A custom Claude Code skill that takes an app description as input, spawns a swarm of parallel agents to discover and research features, lets the user select which features to keep, and saves structured research as markdown files for the next AI session to use.

Inspired by Ruflo's swarm feature — adapted to Claude Code's native Agent tool infrastructure.

## Architecture

```
User Input: "I want to build an app that does X and Y"
                        |
                        v
              +---------------------+
              |  PHASE 1: DISCOVER  |
              |  (3 parallel agents)|
              +----------+----------+
                         |
        +----------------+----------------+
        v                v                v
  +----------+    +----------+    +----------+
  |Competitor|    |User Journey|   |Technical |
  | Analyst  |    | Explorer  |    |Capability|
  |          |    |           |    | Scout    |
  +----+-----+    +-----+-----+   +----+-----+
       +----------------+----------------+
                        v
              +---------------------+
              |  MERGE & DEDUPE     |
              |  (main agent)       |
              +----------+----------+
                         v
              +---------------------+
              |  PHASE 2: SELECT    |
              |  Tiered checklist   |
              |  presented to user  |
              +----------+----------+
                         v
              +---------------------+
              |  PHASE 3: RESEARCH  |
              |  (1 agent per       |
              |   selected feature) |
              +----------+----------+
                         |
        +----+----+------+------+----+
        v    v    v      v      v    v
      Agent Agent Agent Agent Agent ...
        |    |    |      |      |    |
        +----+----+------+------+----+
                         v
              +---------------------+
              |  PHASE 4: OUTPUT    |
              |  Save research +    |
              |  master index +     |
              |  project brief      |
              +---------------------+
```

**Four distinct phases:**
1. **Discover** — 3 agents brainstorm features from different angles, in parallel
2. **Select** — Main agent merges results, presents tiered checklist, user picks
3. **Research** — 1 agent per selected feature does deep competitive + technical research, in parallel
4. **Output** — Main agent assembles all research into organized markdown files

---

## Phase 1: Discovery

Three agents are dispatched in parallel using the Agent tool. Each explores from a different angle.

### Agent 1: Competitor Analyst
- Searches the web for existing apps/products that solve similar problems
- Identifies what features they offer
- Notes pricing models, target audiences, differentiators
- **Output:** list of features found across competitors, with source context

### Agent 2: User Journey Explorer
- Thinks from the end-user's perspective
- Maps out user workflows: what would someone using this app need to do?
- Identifies features implied by the user journeys (onboarding, core actions, settings, notifications, etc.)
- **Output:** list of features derived from user needs

### Agent 3: Technical Capability Scout
- Explores what's technically possible with modern APIs, libraries, and services
- Identifies features enabled by available tech (e.g., "AI-powered search is easy with X API")
- Flags technical constraints or opportunities the user might not have considered
- **Output:** list of technically-enabled features with implementation notes

### Merge Step (main agent)
- Combines all three lists
- Deduplicates (same feature discovered by multiple agents)
- Assigns each feature to a tier:
  - **MVP / Must-Have** — core to the app description
  - **Nice-to-Have** — enhances the experience but not essential
  - **Future / Stretch** — ambitious or dependent on other features
- Tiering criteria: how central it is to the stated app purpose, how many agents independently surfaced it, and dependency complexity

---

## Phase 2: Feature Selection

The skill presents the user with a tiered, interactive checklist:

```
+==============================================================+
|  DISCOVERED FEATURES FOR: "Your app description"             |
+==============================================================+
|                                                              |
|  -- MVP / Must-Have ------------------------------------     |
|  [1] User Authentication          **  complexity | 3/3 agents |
|      Sign up, login, OAuth                                   |
|  [2] Dashboard                    **  complexity | 2/3 agents |
|      Central hub for user actions                            |
|  [3] ...                                                     |
|                                                              |
|  -- Nice-to-Have ---------------------------------------     |
|  [4] Push Notifications           *   complexity | 2/3 agents |
|      Real-time alerts for users                              |
|  [5] ...                                                     |
|                                                              |
|  -- Future / Stretch -----------------------------------     |
|  [6] AI-Powered Recommendations   *** complexity | 1/3 agents |
|      ML-based content suggestions                            |
|  [7] ...                                                     |
|                                                              |
+==============================================================+
|  Select: numbers (e.g. 1,2,4), "all", or "mvp"             |
+==============================================================+
```

**Per feature, the user sees:**
- Number for selection
- Feature name
- Complexity rating (1-3 stars)
- How many discovery agents independently surfaced it (consensus signal)
- One-line description

**Selection shortcuts:**
- Specific numbers: `1, 3, 5`
- `all` — select everything
- `mvp` — select all MVP/Must-Have tier features

Implemented via `AskUserQuestion` tool.

---

## Phase 3: Deep Research

For each selected feature, one agent is dispatched. All run in parallel.

### Agent Input
Each research agent receives:
- The original app description (for context)
- The specific feature name + description
- The tier it was assigned to
- Which discovery agents surfaced it (and any notes they had)

### Agent Output
Each research agent produces a structured report:

```markdown
# Feature: [Feature Name]
## Tier: [MVP / Nice-to-Have / Future]

## Competitive Analysis
- How 2-3 existing products implement this feature
- What works well / what users complain about
- Common UX patterns for this feature

## Technical Feasibility
- Recommended approach / architecture
- Libraries, APIs, or services to use
- Estimated complexity (simple / moderate / complex)
- Dependencies on other features (if any)

## Recommendations
- Suggested MVP scope (what to build first)
- What to skip or defer
- Potential pitfalls to watch out for
```

### Agent Constraints
- Each agent uses `WebSearch` to find real competitive data
- Agents do NOT coordinate with each other — fully independent
- If an agent can't find meaningful competitive data, it says so rather than fabricating
- Target output: ~300-500 words per feature

---

## Phase 4: Output & Handoff

After all research agents return, the main agent assembles the final output.

### File Structure

```
docs/research/
├── README.md                          # Master index
├── project-brief.md                   # Structured brief for next AI session
├── features/
│   ├── 01-user-authentication.md      # Individual research files
│   ├── 02-dashboard.md                # numbered by tier order
│   ├── 03-push-notifications.md
│   └── ...
```

### README.md — Master Index
- App description (the original input)
- Date of research
- Summary table: all selected features with tier, complexity, and link to research file
- Quick-reference for humans browsing the project

### project-brief.md — Structured Brief
Designed as direct input for a fresh Claude session or the `superpowers:writing-plans` skill:

```markdown
# Project Brief: [App Name/Description]
## Generated: [date]

## App Overview
[Original user description, expanded with discovery context]

## Selected Features (by tier)
### MVP / Must-Have
- Feature 1 — [one-line summary + complexity]
- Feature 2 — ...

### Nice-to-Have
- ...

### Future / Stretch
- ...

## Tech Stack Recommendations
[Aggregated from all research agents — common libraries/services across features]

## Feature Dependencies
[Build order suggestions — which features depend on others]

## Suggested Implementation Order
[Recommended sequence based on dependencies + tier]
```

### Handoff Message
The skill prints:
> "Research complete. To start building, open a new session and say: 'Read docs/research/project-brief.md and create an implementation plan'"

---

## Skill File Structure

```
~/.claude/skills/feature-swarm/
├── SKILL.md                           # Main skill file (entry point)
├── prompts/
│   ├── competitor-analyst.md          # Prompt template for discovery agent 1
│   ├── user-journey-explorer.md       # Prompt template for discovery agent 2
│   ├── technical-capability-scout.md  # Prompt template for discovery agent 3
│   └── feature-researcher.md         # Prompt template for Phase 3 agents
```

### Why Separate Prompt Files
- Each agent prompt is ~200-400 words with specific instructions
- Keeping them separate makes them individually testable and editable
- SKILL.md stays clean — it orchestrates, the prompts define agent behavior

### SKILL.md Responsibilities
1. Parse the user's app description input
2. Dispatch 3 discovery agents (reading their prompt templates, injecting the app description)
3. Merge/dedupe results into tiered feature list
4. Present checklist via `AskUserQuestion`
5. Parse user selection
6. Dispatch N research agents (one per feature, using the researcher prompt template)
7. Collect results, write output files
8. Print handoff message

### Invocation
The user types `/feature-swarm` followed by their app description, or just `/feature-swarm` and gets prompted for input.

---

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Research per feature | Competitive + Technical combined | Both angles give a complete picture per feature |
| Discovery strategy | Multi-agent (3 angles) | Surfaces features a single perspective would miss |
| Parallelism model | 1 agent per feature | Deep focus without context bleeding |
| Output format | Markdown in project dir | Readable by humans and future AI sessions |
| Handoff format | Index + structured brief | Quick reference for humans, optimized input for AI |
| Selection UX | Tiered checklist with shortcuts | Sorted by importance, fast to select |
| Skill architecture | Single skill + prompt templates | Simple to maintain, uses native Claude Code tools |
