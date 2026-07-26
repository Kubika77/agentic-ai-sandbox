# Plan: New notebook — Vectorless RAG with PageIndex

## Context

The repo's `README.md` links to the course's reference notebook,
`PageIndex_Vectorless_RAG_CrashCourse (1).ipynb`, and `CLAUDE.md` already lists "Vectorless RAG" as
a planned course topic. The existing `updatedlangchain/` notebooks (1–5) cover core LangChain
mechanics (agents, model integration, tools, messages, structured output) but nothing on retrieval —
this notebook is the natural "6th" entry, introducing **PageIndex**, a hosted document-indexing
service that replaces vector-similarity search with an LLM reasoning over a hierarchical
table-of-contents-like tree (no embeddings, no vector DB, no chunking).

Research into the `VectifyAI/PageIndex` library and its docs (`docs.pageindex.ai`) confirms the
Python SDK flow:
1. `PageIndexClient(api_key=...).submit_document(path)` → `doc_id`
2. poll `get_document(doc_id)["status"]` (or `is_retrieval_ready(doc_id)`) until `"completed"`
3. `get_tree(doc_id, node_summary=True)` → hierarchical tree of nodes (id, title, page range, summary)
4. an LLM reasons over that tree (not the full document) to pick relevant node(s) for a question
5. page content for the chosen node(s) is fetched and passed to a final LLM call that answers the
   question with citations back to the node/page

This maps well onto patterns already used in this repo: step 4 is a structured-output call
(`model.with_structured_output(Schema)`, exactly as in `5-structuredoutput.ipynb`) instead of raw
JSON-prompting, and step 5 is a normal `model.invoke(...)` call. Using LangChain's `ChatOpenAI` /
`init_chat_model` for both LLM steps (instead of the raw `openai` client used in reference examples
found online) keeps the notebook consistent with the rest of the course content.

## New/changed files

- **`updatedlangchain/6-vectorlessrag.ipynb`** (new) — the notebook itself.
- **`pyproject.toml`** — add `"pageindex"` to the `dependencies` list (installed afterward via
  `uv add pageindex`, run as a separate step, not baked into the notebook).
- **`.env`** (not edited by Claude — user action) — needs a new `PAGEINDEX_API_KEY` entry. The user
  must obtain this key from the PageIndex developer dashboard (`dashboard.pageindex.ai`) and add it
  themselves, since `.env` must never be created/printed by the assistant per this project's own
  CLAUDE.md rule.

## Notebook outline (`6-vectorlessrag.ipynb`)

Mirrors the markdown-header + code-cell rhythm used in notebooks 1–5.

1. **Markdown intro** — "Vectorless RAG with PageIndex": 2–3 sentences on what vectorless RAG is
   and how it differs from embedding-based RAG (no vector DB/chunking; LLM reasons over a document
   tree instead).
2. **Setup cell** — `load_dotenv()`, then
   `os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")` and
   `os.environ["PAGEINDEX_API_KEY"] = os.getenv("PAGEINDEX_API_KEY")`, matching the `os.environ[...] =
   os.getenv(...)` pattern used in notebooks 1/2/4/5. Include the same
   `os.environ.pop("SSLKEYLOGFILE", None)` guard used in notebook 1, for consistency.
3. **Markdown** — "Submitting a document to PageIndex".
4. **Download + submit cell** — download a short public PDF at runtime via `requests` (the
   well-known "Attention Is All You Need" arXiv paper, ~15 pages — short enough for PageIndex's tree
   build to finish quickly, and avoids committing a binary file to the repo), save to a temp path,
   then `PageIndexClient(api_key=...).submit_document(path)` → capture `doc_id`.
5. **Polling cell** — small loop calling `get_document(doc_id)["status"]` (or
   `is_retrieval_ready(doc_id)`) with a short `time.sleep`, printing status until `"completed"`.
6. **Markdown** — "Inspecting the PageIndex tree".
7. **Tree retrieval cell** — `get_tree(doc_id, node_summary=True)`, print the tree (titles + node
   ids + summaries) so the reader can see the "table of contents" the LLM will reason over.
8. **Markdown** — "Reasoning over the tree instead of vector search" — explain that instead of
   embeddings, an LLM is shown the (lightweight) tree and picks relevant node id(s) for a question.
9. **Structured node-selection cell** — define a small Pydantic model, e.g.
   ```python
   class NodeSelection(BaseModel):
       node_ids: list[str] = Field(description="ids of the tree nodes relevant to the question")
       reasoning: str = Field(description="brief justification for the chosen nodes")
   ```
   Build the model via `init_chat_model("gpt-4.1")` (consistent with notebook 2/5's use of OpenAI
   through the unified `init_chat_model` interface), bind it with `.with_structured_output(NodeSelection)`,
   and invoke it with a prompt containing the tree (titles/summaries only, no full page text) plus a
   sample question about the paper. This directly reuses the structured-output pattern from
   `5-structuredoutput.ipynb`.
10. **Markdown** — "Fetching content for the selected nodes and answering".
11. **Content + answer cell** — walk the tree structure returned in step 7 to pull the page/text
    range for each selected node id, concatenate into context, then call the plain chat model
    (`model.invoke(...)`) with a prompt asking it to answer the original question using only that
    context and to cite the node/page it drew from. Print the final answer.
12. **Markdown (closing)** — one-paragraph recap: no vector store, no embeddings, no chunking — just
    a document tree plus two LLM calls (select → answer).

## Dependency step

Add to `pyproject.toml`:
```toml
dependencies = [
    "ipykernel>=7.2.0",
    "langchain>=1.3.2",
    "langchain-community>=0.4.2",
    "langchain-google-genai>=4.2.4",
    "langchain-groq>=1.1.2",
    "langchain-openai>=1.2.2",
    "pageindex",
    "python-dotenv>=1.2.2",
]
```
then run `uv add pageindex` (or `uv sync` after the manual edit) so `uv.lock` is regenerated.

## Verification

- Confirm `PAGEINDEX_API_KEY` and `OPENAI_API_KEY` are present in `.env` (names only — never print
  values).
- Run `uv sync` to confirm `pageindex` installs cleanly under Python 3.14.
- Open `updatedlangchain/6-vectorlessrag.ipynb` in Jupyter/VS Code and run all cells top-to-bottom:
  - submission + polling cell reaches `"completed"` status without error
  - tree cell prints a non-empty hierarchical structure
  - structured node-selection cell returns a `NodeSelection` object with at least one valid node id
  - final cell prints a coherent answer that cites the retrieved node/page
- Sanity-check the answer against the source PDF manually (e.g., ask about the paper's core
  contribution and confirm the cited section matches).

---

**Status: implemented.** The notebook, dependency, and lockfile update described above have already
been created/applied in this repo (this file is kept as a record of the plan that was followed).

---

# Notes: Claude Code Agents (what they are / why use them)

## What they are

"Agents" in Claude Code is a different concept from the LangChain agents built in this repo's
notebooks (`1-langchainintro.ipynb`) — it's the CLI tool's own mechanism for spinning up isolated
Claude sessions to handle sub-tasks:

- **Subagents (the `Agent` tool)** — a worker Claude launched *from within* the current session to do
  a bounded task in its own context window, then reports back only a summary. Built-in types
  available in this repo's session: `code-reviewer`, `Explore` (read-only search), `general-purpose`,
  `Plan`, `claude-code-guide`, `statusline-setup`.
- **Custom agents** — defined as markdown files under `.claude/agents/*.md` with frontmatter
  (`name`, `description`, tool restrictions, optional model override) and a system prompt, letting a
  project define its own specialized agent types beyond the built-ins.

## Why use them

- **Protects the main context window.** Grepping across a repo, reading docs, or exploring an
  unfamiliar codebase produces a lot of noise. A subagent absorbs that noise and hands back only
  what matters, instead of filling up the primary conversation.
- **Parallelism.** Multiple independent subagents can run concurrently in one message (e.g. "review
  this PR" + "research this library" at the same time); `isolation: "worktree"` gives each one its
  own git worktree so file edits never collide.
- **Specialization + reduced blast radius.** Task-specific tool access — e.g. `Explore` only gets
  read-only tools — so a research task literally can't accidentally edit files.
- **Background execution.** Long tasks run in the background by default; you get a notification when
  done instead of the session blocking on them.

## Caveats

- A fresh subagent has **no memory** of the parent conversation — its prompt must be self-contained
  (file paths, context, what's already been tried), since it starts cold.
- Each agent has its own context window, so running several in parallel **multiplies token usage** —
  not worth it for trivial one-off lookups.
- There's spawn latency, so reserve agents for tasks actually worth isolating rather than every small
  step.

## How this applies to this project

Useful here for things like: reviewing a notebook for correctness (`code-reviewer` agent), searching
across all `updatedlangchain/*.ipynb` notebooks for a pattern (`Explore` agent), or running a
long-running polling/API task — such as the PageIndex submit-and-poll loop described in the
Vectorless RAG plan above — in the background while continuing other work in the main session.


## Built-in agent types available in claude code:
- claude — catch-all default agent
- claude-code-guide — answers questions about Claude Code, Agent SDK, Claude API
- code-reviewer — security/quality code review (Read, Grep, Glob only)
- Explore — fast read-only codebase search
- general-purpose — multi-step research/tasks, full tool access
- Plan — designs implementation plans
- statusline-setup — configures the status line