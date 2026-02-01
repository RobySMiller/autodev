# Autonomous Software Development Workflow

🤖 **AI-assisted software development with human oversight at key milestones**

This repository contains a workflow system that enables rapid software development using AI agents (Claude Code) with strategic human gates for approval and course correction.

## 🎯 Project Goals

- **Speed**: Reduce development time from weeks to hours/days
- **Quality**: Maintain high standards through automated testing and human review
- **Dogfooding**: Build this system using itself as the first test case
- **Reusability**: Create templates and patterns others can adopt

## 🏗️ Architecture

```
Human (Slack) ↔ Clawbot (Railway) ↔ Dev Machine (Claude Code) ↔ GitHub ↔ Railway Deploy
```

## 📋 Workflow Phases

| Phase | Description | Human Gate | Duration |
|-------|-------------|------------|----------|
| **0** | Setup & Tooling | None (one-time) | 1-2 hours |
| **1** | Idea → Approved Plan | **Plan Approval** | 1-2 hours |
| **2** | Detailed Architecture | **Architecture Approval** | 2-4 hours |
| **3** | Autonomous Build | **Visual/Functional Review** | 4-24 hours |
| **4** | Deploy Pipeline | **Production Approval** | 1-2 hours |
| **5** | Monitor & Learn | None | Ongoing |

## 🚀 Current Status

**Phase 1 - Step 1.1: Project Initialization** ✅

This project is being built using its own workflow system (dogfooding approach).

- [x] Repository created and initialized
- [x] Planning document imported
- [x] Progress tracking established
- [ ] Connect to GitHub (next step)

## 📚 Documentation

- [`docs/original-plan.md`](docs/original-plan.md) - Complete workflow specification
- [`claude-progress.md`](claude-progress.md) - Real-time project progress
- [`docs/prd.md`](docs/prd.md) - Product Requirements (generated in Phase 1.2)
- [`docs/stories.md`](docs/stories.md) - User Stories (generated in Phase 1.3)
- [`docs/stack.md`](docs/stack.md) - Tech Stack Decision (generated in Phase 1.4)

## 🔧 Key Innovation: Progress File Pattern

Each AI session reads/writes a `claude-progress.md` file to maintain context across sessions, solving the "AI memory loss" problem:

```markdown
# Project: [Name]
## Status: [Phase X - Step Y]
## Last Updated: [timestamp]

### Completed
- [x] Task 1 (commit: abc123)

### In Progress  
- [ ] Task 2

### Next Steps
1. [Next action]
```

## 🎪 First Test: Building Itself

This autonomous workflow system is its own first test case. Every step, decision, and iteration is being tracked as we build the tools to build software autonomously.

## 🤝 Contributing

This is currently an experimental dogfooding project. Watch this space as we iterate on the workflow and open it up for broader use.

## 📄 License

MIT (when we get to Phase 4)
