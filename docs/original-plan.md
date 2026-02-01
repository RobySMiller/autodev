# Autonomous Software Development Workflow

## Project Overview

**Goal:** Build tooling that enables rapid, AI-assisted software development with human oversight at key milestones.

**First Project:** This workflow system itself (dogfooding).

**Architecture Decisions Made:**
- **Orchestration:** Claude Code / Agent SDK on dedicated machine (uses Max subscription)
- **Interface:** Clawbot on Railway → Slack for human communication and approvals
- **Deployments:** Railway (existing dev/prod environments)
- **Memory:** Progress file pattern + Memory Tool for context persistence
- **Model Routing:** Claude Code for complex work; future optimization with LiteLLM for high-volume tasks

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                           HUMAN (You)                           │
│                        Slack / iMessage                         │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                       CLAWBOT (Railway)                         │
│                  - Receives messages                            │
│                  - Routes to appropriate handler                │
│                  - Sends approval requests                      │
│                  - Relays results back to human                 │
│                  - Maintains conversation memory (Memory Tool)  │
└─────────────────────────┬───────────────────────────────────────┘
                          │ SSH / API
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DEDICATED DEV MACHINE                        │
│                - Runs Claude Code (uses Max subscription)       │
│                - Executes agent workflows                       │
│                - Has filesystem access for code generation      │
│                - Pushes to GitHub                              │
│                - Writes progress files for session continuity   │
└─────────────────────────┬───────────────────────────────────────┘
                          │ git push
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                          GITHUB                                 │
│                  - Feature branches per task                    │
│                  - PRs trigger CI                              │
│                  - Protected main branch                        │
└─────────────────────────┬───────────────────────────────────────┘
                          │ webhook
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                          RAILWAY                                │
│              - Dev environment (auto-deploy from dev branch)    │
│              - Prod environment (manual deploy from main)       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Workflow Phases Summary

| Phase | Name | Human Gates | Key Outputs |
|-------|------|-------------|-------------|
| 0 | Setup & Tooling | None (one-time) | Infrastructure ready |
| 1 | Idea → Approved Plan | **Plan Approval** | PRD, stories, stack decision |
| 2 | Detailed Architecture | **Architecture Approval** | System design, task breakdown |
| 3 | Autonomous Build | **Visual/Functional Review** | Working code on dev branch |
| 4 | Deploy Pipeline | **Production Approval** | Code on prod |
| 5 | Monitor & Learn | None | Retrospective, template updates |

---

## Progress File Pattern (Memory Across Sessions)

Every agent session reads and writes to a progress file. This solves context loss.

**Location:** `PROJECT_ROOT/claude-progress.md`

**Structure:**
```markdown
# Project: [Name]
## Status: [Phase X - Step Y]
## Last Updated: [timestamp]

### Completed
- [x] Task 1 description (commit: abc123)
- [x] Task 2 description (commit: def456)

### In Progress
- [ ] Task 3 description
- Started: [timestamp]
- Notes: [any context needed]

### Blocked / Needs Human
- [ ] Task 4 description
- Reason: [why blocked]
- Question for human: [specific question]

### Decisions Made
- Decision 1: [what was decided and why]
- Decision 2: [what was decided and why]

### Next Steps
1. [Next immediate action]
2. [Following action]
```

**Rules:**
1. Agent reads this file at session start
2. Agent updates this file before session end
3. Agent updates after each significant action
4. Human can edit this file to course-correct

---

## Phase 0: Setup & Tooling (One-Time)

### 0.1 Dedicated Dev Machine Setup

**Requirements:**
- Mac Mini, Linux box, or cloud VM (Hetzner ~$20/month)
- Tailscale installed for secure remote access
- Claude Code CLI installed and authenticated with Max subscription
- Git configured with GitHub SSH access
- Node.js and Python installed

**Verification:**
```bash
claude --version # Claude Code installed
git remote -v # GitHub access works
tailscale status # VPN connected
```

### 0.2 Clawbot Updates

**Add to Clawbot:**
1. Memory Tool integration (for conversation persistence)
2. SSH command to trigger work on dedicated machine
3. Slack approval flow (buttons for approve/reject)
4. Progress file reader (to report status)

### 0.3 GitHub Repository Setup

**Create repo:** `autonomous-workflow` (or similar)

**Branch protection on `main`:**
- Require PR reviews
- Require status checks to pass
- No direct pushes

**Branch strategy:**
```
main (protected)
└── dev (auto-deploys to Railway dev)
    └── feature/[project]-[phase]
        └── task/[task-id]-[description]
```

### 0.4 Railway Configuration

**Existing setup (confirm):**
- Dev environment connected to `dev` branch
- Prod environment connected to `main` branch
- Auto-deploy enabled for dev
- Manual deploy for prod

---

# PHASE 1: Idea → Approved Plan (DETAILED)

## Overview

**Input:** Problem statement or product idea (from human via Slack)
**Output:** Approved PRD with user stories and tech stack
**Human Gate:** Plan must be approved before Phase 2
**Estimated Time:** 1-2 hours (mostly human review)

---

## Phase 1 Workflow

```
Human sends idea via Slack
                │
                ▼
     Clawbot receives message
                │
                ▼
Clawbot triggers Phase 1 on dedicated machine
                │
                ▼
┌───────────────────────────────────────┐
│         STEP 1.1: Initialize Project │
│           - Create project directory  │
│           - Initialize git repo       │
│           - Create claude-progress.md │
│           - Create initial structure  │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│         STEP 1.2: Generate PRD       │
│           - Analyze problem statement │
│           - Research if needed (web)  │
│           - Write structured PRD      │
│           - Save to docs/prd.md       │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│      STEP 1.3: Generate User Stories │
│           - Break PRD into stories    │
│           - Prioritize (MoSCoW)       │
│           - Add acceptance criteria   │
│           - Save to docs/stories.md   │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│      STEP 1.4: Recommend Tech Stack  │
│           - Analyze requirements      │
│           - Consider existing templates│
│           - Justify choices           │
│           - Save to docs/stack.md     │
└───────────────────┬───────────────────┘
                    │
                    ▼
┌───────────────────────────────────────┐
│       STEP 1.5: Compile Plan Summary │
│           - Create executive summary  │
│           - List key risks/assumptions│
│           - Update claude-progress.md │
│           - Commit all docs           │
└───────────────────┬───────────────────┘
                    │
                    ▼
      Clawbot sends plan summary to Slack
                    │
                    ▼
              ┌────────────┐
              │   HUMAN    │
              │    GATE    │
              └─────┬──────┘
                    │
              ┌────┴────┐
              ▼         ▼
           Approve   Reject
              │         │
              ▼         ▼
           Phase 2   Human edits and resubmits
```

---

## Step 1.1: Initialize Project

**Trigger:** Clawbot receives new project request

**Actions:**
```bash
# On dedicated machine
cd ~/projects
mkdir [project-name]
cd [project-name]

# Initialize git
git init
git remote add origin git@github.com:[user]/[project-name].git

# Create directory structure
mkdir -p docs
mkdir -p src  
mkdir -p tests

# Create progress file
touch claude-progress.md

# Create .gitignore
cat > .gitignore << 'EOF'
node_modules/
.env
.env.local
*.log
.DS_Store
EOF

# Initial commit
git add .
git commit -m "Initial project setup"
git push -u origin main
git checkout -b dev
git push -u origin dev
```

**Progress File Update:**
```markdown
# Project: [Project Name]
## Status: Phase 1 - Step 1.1 Complete
## Last Updated: [timestamp]

### Completed
- [x] Project directory created
- [x] Git repository initialized  
- [x] Connected to GitHub
- [x] Branch structure created (main, dev)

### In Progress
- [ ] PRD generation

### Next Steps
1. Analyze problem statement
2. Generate PRD
```

**Output:** Empty project with git setup, progress file initialized

---

## Step 1.2: Generate PRD

**Input:** Original problem statement from human

**PRD Template:** `docs/prd.md`

```markdown
# Product Requirements Document: [Project Name]

## 1. Problem Statement

### 1.1 Background
[Context about the problem space]

### 1.2 Problem  
[Specific problem being solved]

### 1.3 Target Users
[Who will use this product]

### 1.4 Current Solutions
[How users solve this today, if applicable]

---

## 2. Proposed Solution

### 2.1 Overview
[High-level description of the solution]

### 2.2 Core Value Proposition
[Why users will choose this over alternatives]

---

## 3. Features

### 3.1 Must Have (P0)
| Feature | Description | Success Criteria |
|---------|-------------|------------------|
|         |             |                  |

### 3.2 Should Have (P1)
| Feature | Description | Success Criteria |
|---------|-------------|------------------|
|         |             |                  |

### 3.3 Could Have (P2)
| Feature | Description | Success Criteria |
|---------|-------------|------------------|
|         |             |                  |

### 3.4 Won't Have (Out of Scope)
- [Feature explicitly excluded]

---

## 4. Success Metrics

| Metric | Target | How Measured |
|--------|--------|--------------|
|        |        |              |

---

## 5. Constraints & Assumptions

### 5.1 Technical Constraints
- [Constraint 1]

### 5.2 Business Constraints  
- [Timeline, budget, etc.]

### 5.3 Assumptions
- [Assumption 1]

---

## 6. Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
|      |            |        |            |

---

## 7. Open Questions
- [ ] [Question that needs human input]
```

**Agent Instructions:**
1. Read the problem statement carefully
2. If problem is vague, list clarifying questions in "Open Questions"
3. Research competitors/existing solutions if relevant (use web search)
4. Fill in all sections with reasonable depth
5. Be opinionated on prioritization (MoSCoW)
6. Flag assumptions explicitly

**Progress File Update:**
```markdown
### Completed
- [x] Project directory created
- [x] Git repository initialized
- [x] PRD generated (docs/prd.md)

### In Progress  
- [ ] User stories generation

### Decisions Made
- [Any decisions made during PRD, e.g., "Targeting web-first, mobile later"]
```

---

## Step 1.3: Generate User Stories

**Input:** Completed PRD

**Stories Template:** `docs/stories.md`

```markdown
# User Stories: [Project Name]

## Epic 1: [Epic Name]

### Story 1.1: [Story Title]
**As a** [user type]
**I want to** [action]  
**So that** [benefit]

**Priority:** P0/P1/P2

**Acceptance Criteria:**
- [ ] [Criterion 1]
- [ ] [Criterion 2] 
- [ ] [Criterion 3]

**Technical Notes:**
- [Any implementation hints]

**Dependencies:**
- [Other stories this depends on]

---

### Story 1.2: [Story Title]
...

---

## Epic 2: [Epic Name]
...
```

**Agent Instructions:**
1. Read PRD features section
2. Convert each feature into one or more user stories
3. Group stories into logical epics
4. Add acceptance criteria that are testable
5. Note dependencies between stories
6. Maintain priority from PRD

**Validation:**
- Every P0 feature has at least one story
- Every story has at least 2 acceptance criteria
- No circular dependencies

---

## Step 1.4: Recommend Tech Stack

**Input:** PRD and user stories

**Stack Template:** `docs/stack.md`

```markdown
# Technical Stack Recommendation: [Project Name]

## 1. Recommended Stack

| Layer | Technology | Version | Justification |
|-------|------------|---------|---------------|
| Frontend |          |         |               |
| Backend  |          |         |               |  
| Database |          |         |               |
| Hosting  |          |         |               |
| CI/CD    |          |         |               |

## 2. Architecture Pattern
[e.g., Monolith, Microservices, Serverless]

**Justification:** [Why this pattern fits]

## 3. Key Libraries/Dependencies

| Purpose | Library | Why |
|---------|---------|-----|
|         |         |     |

## 4. Alternatives Considered

| Layer | Alternative | Why Not Chosen |
|-------|-------------|----------------|
|       |             |                |

## 5. Template/Starter to Use
[Link to GitHub template or "Build from scratch"]

**Why:** [Justification]

## 6. Development Environment

| Tool | Purpose |
|------|---------|
|      |         |

## 7. Estimated Complexity
- **Overall:** [Low/Medium/High]
- **Frontend:** [Low/Medium/High] 
- **Backend:** [Low/Medium/High]
- **Infrastructure:** [Low/Medium/High]
```

**Agent Instructions:**
1. Consider project requirements from PRD
2. Favor technologies Roby already uses (Next.js, Railway, etc.)
3. Prefer existing templates when suitable
4. Keep stack simple for MVP
5. Justify every choice

**Known Preferences:**
- Hosting: Railway (already set up)
- Frontend: React/Next.js preferred
- Avoid over-engineering for MVPs

---

## Step 1.5: Compile Plan Summary

**Actions:**
1. Create executive summary combining PRD, stories, stack
2. List all open questions requiring human input
3. Update progress file
4. Commit all documents
5. Prepare Slack message for approval

**Summary Template:** `docs/plan-summary.md`

```markdown
# Plan Summary: [Project Name]

## Executive Summary
[2-3 paragraphs summarizing what we're building and why]

## Scope
- **In Scope:** [Bullet list]
- **Out of Scope:** [Bullet list]

## Key Deliverables
1. [Deliverable 1]
2. [Deliverable 2]

## Tech Stack (High-Level)
- Frontend: [X]
- Backend: [Y] 
- Database: [Z]
- Hosting: Railway

## Timeline Estimate
- Phase 2 (Architecture): [X hours]
- Phase 3 (Build): [X hours/days]
- Phase 4 (Deploy): [X hours]

## Risks & Concerns
1. [Risk 1]
2. [Risk 2]

## Open Questions (Need Human Input)
1. [ ] [Question 1]
2. [ ] [Question 2]

## Recommendation
[Go/No-Go recommendation with reasoning]
```

**Git Commit:**
```bash
git add docs/
git commit -m "Phase 1 complete: PRD, stories, and stack recommendation"
git push origin dev
```

**Slack Message to Human:**
```
📋 *Plan Ready for Review: [Project Name]*

*Summary:* [1-2 sentence description]
*Scope:* [X] features in P0, [Y] in P1  
*Stack:* [Frontend] + [Backend] on Railway
*Estimated Build Time:* [X] hours

*Open Questions:*
• [Question 1]
• [Question 2]

📄 *Full docs:* [GitHub link to docs/ folder]

*Actions:*
• Reply `approve` to proceed to architecture
• Reply `reject [reason]` to revise  
• Reply with answers to open questions
```

---

## Phase 1 Completion Criteria

**Checklist before requesting approval:**
- [ ] `docs/prd.md` exists and is complete
- [ ] `docs/stories.md` exists with all P0 features covered
- [ ] `docs/stack.md` exists with justified recommendations
- [ ] `docs/plan-summary.md` exists
- [ ] `claude-progress.md` is up to date
- [ ] All docs committed to `dev` branch
- [ ] No unaddressed errors or warnings
- [ ] Open questions are clearly listed

**Human Approval Criteria:**
- PRD accurately captures the problem and solution
- Scope is appropriate for MVP
- Tech stack makes sense
- Timeline estimate is reasonable
- Open questions are answered or deferred

---

## Phase 1 Error Handling

| Error | Recovery |
|-------|----------|
| Problem statement too vague | List clarifying questions, send to human before continuing |
| Cannot determine tech stack | Default to Next.js + Railway, note assumption |
| Git push fails | Retry 3x, then alert human |
| Session timeout mid-phase | Progress file allows resume from last completed step |

---

## Clawbot Commands for Phase 1

**Trigger Phase 1:**
```
@clawbot new project: [problem statement]
```

**Check Status:**
```
@clawbot status
```
Returns current phase, step, and any blockers.

**Approve Plan:**
```
@clawbot approve
```
Or use Slack button.

**Reject Plan:**
```
@clawbot reject [reason]
```

**Answer Question:**
```
@clawbot answer: [question number] [answer]
```

---

## Transition to Phase 2

**On Approval:**
1. Clawbot updates progress file with approval
2. Clawbot triggers Phase 2 on dedicated machine
3. Phase 2 begins with architecture generation

**Progress File After Approval:**
```markdown
# Project: [Project Name]
## Status: Phase 2 - Starting
## Last Updated: [timestamp]

### Completed
- [x] Phase 1: Planning (Approved by human on [date])
  - PRD: docs/prd.md
  - Stories: docs/stories.md  
  - Stack: docs/stack.md

### In Progress
- [ ] Phase 2: Architecture

### Decisions Made
- Plan approved as-is
- [Any answers to open questions]

### Next Steps
1. Generate system architecture
2. Create component breakdown
3. Detail task list
```

---

# Phases 2-5: Summary (Detail in Separate Docs)

## Phase 2: Detailed Architecture
- System architecture diagram (Mermaid)
- API contracts
- Data models
- Component breakdown  
- Task list with estimates
- **Human Gate:** Architecture approval

## Phase 3: Autonomous Build
- Create feature branch per task
- Implement code
- Write tests
- Run tests
- Debug failures (escalate if needed)
- Checkpoint every 15 minutes
- **Human Gate:** Visual/functional review at dev URL

## Phase 4: Deploy Pipeline
- Security scan (SAST, deps, secrets)
- Deploy to staging (auto)
- Integration tests
- **Human Gate:** Production approval
- Deploy to prod

## Phase 5: Monitor & Learn
- Monitor for errors (24-48 hours)
- Auto-rollback if error spike
- Generate retrospective
- Update templates and routing rules
- Adjust autonomy thresholds

---

# Appendix: Clawbot Integration Checklist

Before running Phase 1, ensure Clawbot has:
- [ ] Memory Tool enabled for conversation persistence
- [ ] SSH access to dedicated dev machine (via Tailscale)
- [ ] Slack bot with approval buttons
- [ ] Command parser for: `new project`, `status`, `approve`, `reject`, `answer`
- [ ] Ability to read `claude-progress.md` from remote machine
- [ ] Ability to send formatted Slack messages with links

---

# Appendix: File Structure After Phase 1

```
[project-name]/
├── .git/
├── .gitignore
├── claude-progress.md
├── docs/
│   ├── prd.md
│   ├── stories.md
│   ├── stack.md
│   └── plan-summary.md
├── src/ (empty, ready for Phase 3)
└── tests/ (empty, ready for Phase 3)
```