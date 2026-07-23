Complete Agentic AI Course- Langchain, Langgraph, RAG,Vectorless RAG, Guardrails,Evals:
https://www.youtube.com/watch?v=rV3HJ4LEZ7k&t=3439s

https://github.com/krishnaik06/Langchain-V1-Crash-Course
https://github.com/krishnaik06/Agentic-LanggraphCrash-course
https://github.com/krishnaik06/RAG-Tutorials
https://github.com/krishnaik06/RAG-Tutorials/blob/main/PageIndex_Vectorless_RAG_CrashCourse%20(1).ipynb

## Claude Code tools available in this session

**Always-loaded tools**
- **Bash** / **PowerShell** — shell execution (Bash for POSIX/git-bash syntax, PowerShell for native Windows)
- **Read** / **Write** / **Edit** — file operations
- **Glob** / **Grep** — file pattern matching and content search
- **Agent** — launch subagents for research or multi-step tasks
- **Artifact** — publish HTML/Markdown as a shareable web page
- **AskUserQuestion** — ask a clarifying question with structured options
- **ScheduleWakeup** — schedule a resumption for `/loop` dynamic mode
- **Skill** — invoke a packaged skill/slash-command
- **ToolSearch** — fetch schemas for deferred tools (below)
- **ReportFindings** — structured output for code-review results

**Deferred tools** (need `ToolSearch` to load a schema before use)
- **WebFetch**, **WebSearch** — fetch URLs / search the web
- **NotebookEdit** — edit Jupyter notebook cells
- **EnterPlanMode** / **ExitPlanMode** — planning mode workflow
- **EnterWorktree** / **ExitWorktree** — git worktree isolation
- **Monitor** — stream events from a background process
- **SendMessage** — message/resume a previously spawned agent
- **CronCreate** / **CronList** / **CronDelete** — scheduled cloud agent management
- **TaskOutput**, **TaskStop** — background task control
- **RemoteTrigger**, **DesignSync**, **PushNotification** — misc integrations
- **mcp__ide__executeCode**, **mcp__ide__getDiagnostics** — run code / get diagnostics via IDE connection
- **EndConversation** — reserved for ending a session after sustained abuse

**Subagent types** (via `Agent` tool): `claude`, `claude-code-guide`, `code-reviewer` (project custom agent), `Explore`, `general-purpose`, `Plan`, `statusline-setup`

**Skills** (via `Skill` tool): dataviz, artifact-design, artifact-capabilities, update-config, keybindings-help, simplify, fewer-permission-prompts, loop, schedule, claude-api, claude-in-chrome, run, init, review, security-review
