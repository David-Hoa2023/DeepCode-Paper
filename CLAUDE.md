# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepCode is an AI-powered multi-agent system that transforms research papers and natural language descriptions into production-ready code. It supports three main workflows: Paper2Code (academic paper implementation), Text2Web (frontend development), and Text2Backend (backend development).

## Build and Run Commands

### Installation
```bash
# Install package directly
pip install deepcode-hku

# Or install from source
pip install -r requirements.txt
```

### Running the Application
```bash
# Web interface (recommended) - launches Streamlit on localhost:8503
python deepcode.py

# Or directly with streamlit
streamlit run ui/streamlit_app.py

# CLI interface
python cli/main_cli.py

# Paper test mode
python deepcode.py test <paper_name>
python deepcode.py test <paper_name> --fast
```

### Configuration
- `mcp_agent.config.yaml` - Main configuration (LLM providers, MCP servers, search settings)
- `mcp_agent.secrets.yaml` - API keys (OpenAI, Anthropic, Google)

## Architecture Overview

### Core Components

**Entry Points:**
- `deepcode.py` - Main launcher (web UI via Streamlit or paper test mode)
- `cli/main_cli.py` - CLI interface entry point
- `ui/streamlit_app.py` - Web interface entry point

**Multi-Agent Orchestration:**
- `workflows/agent_orchestration_engine.py` - Central orchestrator that coordinates all agents and manages workflow execution
- `workflows/code_implementation_workflow.py` - Main workflow for code generation tasks
- `workflows/codebase_index_workflow.py` - Workflow for indexing and analyzing codebases

**Specialized Agents** (in `workflows/agents/`):
- `memory_agent_concise.py` - Memory management and context engineering across sessions
- `code_implementation_agent.py` - Code synthesis and generation
- `requirement_analysis_agent.py` - Parses user requirements into actionable specs
- `document_segmentation_agent.py` - Handles large document processing

**MCP Tool Servers** (in `tools/`):
- `code_implementation_server.py` - File operations, code execution, testing
- `code_reference_indexer.py` - Code search and indexing
- `document_segmentation_server.py` - Document analysis for large papers
- `command_executor.py` - Shell command execution
- `git_command.py` - GitHub repository operations
- `pdf_downloader.py` - Document download and conversion
- `bocha_search_server.py` - Alternative web search

**UI Layer:**
- `ui/handlers.py` - Event handlers and business logic for web UI
- `ui/components.py` - Reusable UI components
- `ui/styles.py` - CSS styling

**Utilities:**
- `utils/file_processor.py` - File handling and format conversion
- `utils/dialogue_logger.py` - Conversation logging
- `utils/llm_utils.py` - LLM provider abstraction

### Data Flow

1. User input (paper/URL/text) → Orchestration Engine
2. Orchestration Engine → Requirement Analysis Agent (parse intent)
3. Orchestration Engine → Document Segmentation Agent (if large docs)
4. Orchestration Engine → Memory Agent (context management)
5. Orchestration Engine → Code Implementation Agent (generate code)
6. Code Implementation Agent ↔ MCP Servers (file ops, execution, search)
7. Output → User (code + tests + documentation)

### LLM Provider Configuration

The system supports multiple LLM providers configured in `mcp_agent.config.yaml`:
- `llm_provider` field selects active provider: "google", "anthropic", or "openai"
- Falls back to first available provider if primary unavailable
- Supports OpenAI-compatible endpoints via base_url configuration

## Key Patterns

- All MCP servers use the Model Context Protocol standard for tool integration
- Async execution throughout using `asyncio` (configured via `execution_engine: asyncio`)
- Hierarchical memory system for managing large codebases and long sessions
- CodeRAG pattern: retrieve relevant code examples before generation
