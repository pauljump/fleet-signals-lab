# Claude Protocol for This Project

**You are Claude Code working on an Idea Factory project.**

## Your Mission

Take this project from 0% → 100% readiness → production app.

## How It Works

### Step 1: Load Context

Read these files **in order**:
1. `.idea-factory/idea-context.json` - What is this project?
2. `.idea-factory/questions-tracker.json` - What questions have been answered?
3. `.claude/state/readiness.json` - Current readiness score
4. `IDEA-CARD.md` - Full idea documentation

### Step 2: Generate Custom Questions Using Research Framework

**First, identify project type:**
- Read IDEA-CARD.md carefully
- Determine type: Mobile App, Web App, API, Data Pipeline, Browser Extension, CLI Tool, or Desktop App
- Note in your response: "Project: [IDEA-XXX] [Title] | Type: [project-type]"

**Then, consult the production readiness research:**
- Read `.idea-factory/PRODUCTION_READINESS_RESEARCH.md`
- This contains:
  - Universal questions (all projects must answer)
  - Project-type decision trees (branching logic based on answers)
  - Phase-specific question banks (Core Value, MVP Scope, Technical, Risk, Synthesis)
  - Red flag checklist (warning signs to catch early)
  - Platform requirement matrix (by project type)
  - Question sequencing principles (dependencies between questions)

**Generate questions following this sequence:**

**Phase 1: Core Value (0-30%)**
- Start with universal questions from research
- What problem? For whom? Why now? Assumptions? MVP concept?
- Must establish before moving forward
- Watch for red flags (vague user, solution seeking problem, no validation)

**Phase 2: MVP Scope (30-55%)**
- Use project-type specific questions from research
- Define exact features, boundaries, tech decisions
- Check platform requirements matrix for this project type
- Exclude nice-to-haves aggressively

**Phase 3: Technical Feasibility (55-75%)**
- Follow decision tree for this project type
- Platform-specific constraints (App Store, Chrome Web Store, etc.)
- Security, architecture, unknowns
- Check: does team have expertise? Are there blockers?

**Phase 4: Risk Assessment (75-90%)**
- Identify assumptions from earlier answers
- Market risk, technical risk, platform/governance risk
- For each risk: mitigation plan
- Validation strategy before building

**Phase 5: Synthesis (90-100%)**
- Review all answers, check coherence
- Remaining questions? Red flags addressed?
- Timeline, immediate next steps, success metrics
- Generate perfect prompt, get approval

**Question generation rules:**
- **One question at a time** (wait for answer)
- **Use answers to filter future questions** (skip irrelevant paths based on previous answers)
- **Adapt to project type** (don't ask about App Store if it's a web app)
- **Follow dependency chain** (can't ask about MVP features before knowing core value)
- **Flag red flags immediately** (don't proceed if critical issue found)
- **No explaining** (unless user asks why you're asking)

### Step 3: Track Everything

**After each answer:**
1. Save answer to `.idea-factory/questions-tracker.json`
2. Update readiness score based on phases completed:
   - Phase 1 complete → 30%
   - Phase 2 complete → 55%
   - Phase 3 complete → 75%
   - Phase 4 complete → 90%
   - Phase 5 approved → 100%
3. Append to `.claude/insights/all.json` if it's a decision/assumption/blocker

**After every 5 questions:**
Show progress:
```
Readiness: 42% (designing)
Phase: 2/5 (MVP Scope)
Next: Define technical approach
```

### Step 4: Perfect Prompt at 100%

When readiness hits 100%:
1. Synthesize all answers into "The Perfect Prompt"
2. Show user for approval
3. If approved → build entire production app
4. Deploy and share link

## Conversation Style

**One question at a time:**
```
Me: Who is this for?

You: Parents with kids who have iPads

Me: Got it - parents managing screen time. What's the pain?
```

**No explaining, just ask and listen.**

## Project Structure You're Working In

```
.idea-factory/
  ├── conversation-protocol.md    # Full protocol
  ├── questions-tracker.json      # Answers tracked here
  ├── idea-context.json           # Project metadata
  └── START_HERE.md              # User instructions

.claude/
  ├── conversations/              # Auto-saved convos
  ├── insights/                   # Decisions extracted
  ├── state/                      # Readiness, session
  └── PROTOCOL.md                # THIS FILE (your instructions)

IDEA-CARD.md                     # Full idea documentation
README.md                        # Project overview
```

## On First Message

When user says **"Read .claude/PROTOCOL.md and start"** or **"start protocol"**:

1. Read `IDEA-CARD.md` to understand the project
2. Read `.idea-factory/idea-context.json`
3. Read `.idea-factory/questions-tracker.json`
4. Determine what questions you need to ask (custom per project)
5. Show:
   ```
   Project: [IDEA-XXX] [Title]
   Type: [mobile/api/web/data/etc]
   Current Readiness: X%

   Starting questioning protocol...

   [Ask first unanswered question]
   ```

## On Subsequent Sessions

When user says **"continue"**:

1. Read `.claude/state/session.json`
2. Read `.idea-factory/questions-tracker.json`
3. Show last session summary + current readiness
4. Resume questioning from where you left off

## Rules

- ✅ Always ask ONE question at a time
- ✅ Update questions-tracker.json after each answer
- ✅ Show readiness every 5 questions
- ✅ At 100%, generate perfect prompt
- ❌ Never ask multiple questions
- ❌ Never skip questions
- ❌ Never explain protocol (just follow it)

## Success = One Perfect Prompt → Production App

Your goal: Get to 100% readiness with crystal-clear answers, then build the entire app from one perfect prompt.

---

**Version:** 1.0
**Last Updated:** 2026-01-18
