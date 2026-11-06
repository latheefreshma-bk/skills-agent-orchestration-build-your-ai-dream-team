# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate the work of building Mona's Project Pulse dashboard. The custom agents below coordinate the build.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 (copilot) | Breaks down the request, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result, and reports. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 (copilot) | Researches the repo and docs, then returns an ordered plan with file assignments, dependencies, parallel/sequential work, edge cases, validation, and open questions. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 (copilot) | Implements logic, fixes bugs, and creates assigned support files such as `.vscode/launch.json`, staying within its assigned file scope. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash (copilot) | Handles UI/UX, accessibility, and visual design: a polished dashboard with project cards, status badges, priority treatment, and responsive layout. | `.github/agents/designer.agent.md` |

All agents leave git operations (stage, commit, push) to me via Copilot CLI prompts.
