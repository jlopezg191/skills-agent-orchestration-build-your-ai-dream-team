# Agent team

For Mona's Project Pulse dashboard, I will use a four-agent custom team orchestrated through GitHub Copilot CLI in a Codespace. The definitions live under `.github/agents/` in the repository and each agent has a specific role in the workflow.

- Planner — Model: `Claude Opus 4.7 (copilot)` — Researches the repo, reads relevant docs and dependencies, and produces an implementation plan with file assignments, sequencing, risks, and validation expectations. Definition: `.github/agents/planner.agent.md`
- Orchestrator — Model: `Claude Opus 4.7 (copilot)` — Breaks the plan into phases, delegates work to the specialist agents, keeps file ownership clear, and verifies the final integration before reporting back to the user. Definition: `.github/agents/orchestrator.agent.md`
- Designer — Model: `Gemini 3.1 Pro (copilot)` — Focuses on the dashboard UX and visual design, including usability, accessibility, interaction flow, information hierarchy, and polished project-card styling for Project Pulse. Definition: `.github/agents/designer.agent.md`
- Coder — Model: `GPT-5.5 (copilot)` — Implements the code and logic in the assigned scope, fixes bugs, and validates behavior in the app or files delegated by the Orchestrator. Definition: `.github/agents/coder.agent.md`

This team is being coordinated with GitHub Copilot CLI in a Codespace so the Planner, Orchestrator, Designer, and Coder can work together in a structured, multi-agent workflow to build the Project Pulse dashboard.
