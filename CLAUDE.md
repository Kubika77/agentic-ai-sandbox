# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A learning sandbox for agentic AI development with LangChain (v1), following the
"Complete Agentic AI Course" (LangChain, LangGraph, RAG, Vectorless RAG, Guardrails, Evals) —
see README.md for course links. Content is primarily Jupyter notebooks under `updatedlangchain/`,
each covering one LangChain concept:

- `1-langchainintro.ipynb` — LangChain v1 agents, running an agent, tool responses
- `2-modelintegration.ipynb` — model integration across OpenAI, Google Gemini, and GROQ; streaming vs. batch
- `3-tools.ipynb` — tool schemas and tool execution loops
- `4-messages.ipynb` — message types (system/human/ai) vs. plain text prompts
- `5-structuredoutput.ipynb` — structured output via Pydantic and dataclasses, including nested structures

`main.py` is a placeholder entry point (not the focus of this project — the notebooks are).

## Environment & commands

- Package manager: **uv** (`pyproject.toml` + `uv.lock`). Requires Python >= 3.14 (`.python-version` pins 3.14).
- Install dependencies: `uv sync`
- Run the placeholder script: `uv run main.py`
- Run notebooks: open under `updatedlangchain/` via Jupyter/VS Code (ipykernel is a project dependency).
- No test suite, linter, or build step is currently configured.

## API keys

Model providers are configured via a `.env` file (gitignored, not committed) with:
`GROQ_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`. Notebooks load these with `python-dotenv`.
Never commit `.env` or print its contents.

## Key dependencies

`langchain`, `langchain-community`, `langchain-openai`, `langchain-google-genai`, `langchain-groq` —
notebooks intentionally show both the `langchain.chat_models` unified interface and each provider's
direct integration side by side, so don't collapse these into a single pattern when editing.
