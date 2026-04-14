# Feature Swarm

A Claude Code plugin that spawns a swarm of parallel AI agents to discover, research, and document features for your next app or project.

Inspired by Ruflo's swarm feature, adapted for Claude Code's native Agent infrastructure.

## What It Does

You describe your app idea. Feature Swarm handles the rest:

1. **Discover** -- 3 agents research features in parallel from different angles:
   - **Competitor Analyst** -- searches the web for similar products and extracts their feature sets
   - **User Journey Explorer** -- maps user workflows and identifies implied features
   - **Technical Capability Scout** -- researches what modern APIs/libraries make possible

2. **Select** -- presents a tiered checklist (MVP / Nice-to-Have / Future) with complexity ratings and agent consensus scores. Pick specific features, select all, or just the MVP tier.

3. **Research** -- spawns 1 agent per selected feature for deep competitive analysis + technical feasibility, all running in parallel.

4. **Output** -- saves structured research as markdown files, ready for the next AI session to build from.

## Example

```
/feature-swarm I want to build a habit tracking app where users create daily habits, mark them complete, and see their streaks
```

Output:

```
docs/research/
├── README.md              # Master index with feature table
├── project-brief.md       # Structured brief for AI-assisted planning
├── features/
│   ├── 01-user-auth.md    # Competitive + technical research
│   ├── 02-streak-tracking.md
│   ├── 03-notifications.md
│   └── ...
```

To continue building after research:

```
Read docs/research/project-brief.md and create an implementation plan
```

## Installation

### Option 1: As a Claude Code Plugin (Recommended)

Add the GitHub repo as a marketplace, then install:

```bash
/plugin marketplace add lucatescari/claude-feature-swarm
/plugin install feature-swarm@lucatescari/claude-feature-swarm
```

The skill becomes available as `/feature-swarm` in all your Claude Code sessions.

### Option 2: Manual Installation

Copy the skill files directly to your Claude Code skills directory:

```bash
git clone https://github.com/lucatescari/claude-feature-swarm.git
mkdir -p ~/.claude/skills/feature-swarm
cp -r claude-feature-swarm/skills/feature-swarm/* ~/.claude/skills/feature-swarm/
```

The skill will be auto-discovered by Claude Code -- no restart needed.

## How It Works

Feature Swarm is a Claude Code skill (a markdown guidance document, not application code). When invoked, it instructs Claude to:

- Read prompt templates from the `prompts/` directory
- Dispatch parallel agents using Claude Code's built-in `Agent` tool
- Merge and deduplicate results using Claude's reasoning
- Present an interactive selection UI via `AskUserQuestion`
- Write structured output files to your project

No external dependencies. No API keys beyond your existing Claude Code setup. No servers to run.

## File Structure

```
skills/feature-swarm/
├── SKILL.md                           # Main orchestration logic
├── prompts/
│   ├── competitor-analyst.md          # Discovery agent: market research
│   ├── user-journey-explorer.md       # Discovery agent: user perspective
│   ├── technical-capability-scout.md  # Discovery agent: tech possibilities
│   └── feature-researcher.md         # Deep research agent (1 per feature)
```

## Output Format

### Feature Selection Checklist

After discovery, you see a tiered list like:

```
-- MVP / Must-Have --
[1] User Authentication          * complexity | 2/3 agents
    Registration, login, password reset
[2] Streak Tracking              * complexity | 3/3 agents
    Consecutive day counts with visual display

-- Nice-to-Have --
[3] Offline Support              ** complexity | 3/3 agents
    Works without internet, syncs when connected

-- Future / Stretch --
[4] AI Habit Coaching            *** complexity | 1/3 agents
    Personalized insights and streak predictions

Select: numbers (1,3), "all", or "mvp"
```

### Per-Feature Research

Each feature gets a detailed research file:

```markdown
# Feature: Streak Tracking
## Tier: MVP

## Competitive Analysis
- **Streaks app:** Uses "don't break the chain" as core UX...
- **Loop Habit Tracker:** Forgiving "habit strength" formula...

## Technical Feasibility
- **Recommended approach:** Simple date-based counter with...
- **Libraries/APIs:** date-fns for date math, D3.js for...
- **Complexity:** simple

## Recommendations
- **MVP scope:** Basic consecutive-day counter with number display
- **Defer:** Heatmap visualization, streak freezes
- **Pitfalls:** Timezone handling across devices
```

### Project Brief

A structured document designed as input for AI-assisted planning:

- App overview
- Selected features by tier with complexity ratings
- Tech stack recommendations (aggregated from all research)
- Feature dependencies and suggested build order

## Configuration

No configuration needed. The skill uses Claude Code's built-in tools:

- `Agent` for parallel agent dispatch
- `WebSearch` for competitive and technical research
- `AskUserQuestion` for feature selection
- `Write` for saving output files

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI, desktop app, or IDE extension
- A Claude API key with access to web search

## License

MIT
