# Agent team

These custom agents will work together to build Mona's Project Pulse dashboard:

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Coordinates the Planner, Coder, and Designer, assigns scoped tasks and phases, and checks that the integrated dashboard meets the goal. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repository and relevant dependencies, then creates an implementation plan with file assignments, dependencies, risks, edge cases, and validation expectations. Does not write code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements dashboard logic and runnable-app support within the assigned scope, following repository patterns and validating the changes. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Shapes the dashboard's usability, accessibility, information hierarchy, responsive behavior, and visual design, including clear project cards and status and priority treatments. | `.github/agents/designer.agent.md` |

I am using GitHub Copilot CLI in a Codespace to orchestrate this team as it builds Project Pulse.
