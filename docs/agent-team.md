# Agent team

The custom agent team for building Mona's Project Pulse dashboard is coordinated through GitHub Copilot CLI in a Codespace:

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 4.7 (copilot) | Breaks the dashboard request into phases, delegates work to the specialist agents, manages dependencies and file ownership, and verifies that the integrated result works together. | [`.github/agents/orchestrator.agent.md`](../.github/agents/orchestrator.agent.md) |
| **Planner** | Claude Opus 4.7 (copilot) | Researches the repository and relevant documentation, identifies requirements, risks, edge cases, dependencies, and validation needs, then produces an actionable implementation plan. | [`.github/agents/planner.agent.md`](../.github/agents/planner.agent.md) |
| **Coder** | GPT-5.5 (copilot) | Implements the dashboard code, fixes bugs, creates assigned runnable-app support files, follows repository patterns, and validates deterministic, testable behavior. | [`.github/agents/coder.agent.md`](../.github/agents/coder.agent.md) |
| **Designer** | Gemini 3.1 Pro (copilot) | Shapes the Project Pulse user experience through information hierarchy, accessibility, responsive behavior, visual clarity, polished project cards, status badges, and priority treatment. | [`.github/agents/designer.agent.md`](../.github/agents/designer.agent.md) |

The Orchestrator uses the Planner's research to assign non-overlapping work to the Coder and Designer, then coordinates integration and verification. GitHub Copilot CLI in the Codespace is the control point for this workflow; the agents do not stage, commit, or push changes.
