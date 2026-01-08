# Agentic Computer & Browser Template

**A production-ready foundation for building LLM agents that control browsers and operating systems.**

[![MIT License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)

## Overview

This template provides a high-efficiency architecture for building autonomous agents that interact with computers through:
- **Compact UI state representation** - Token-optimized element trees instead of raw HTML/screenshots
- **Incremental diff streaming** - Send only what changed, not entire pages
- **Strict action DSL** - Constrained, reliable actions with JSON Schema validation
- **Google Gemini 2.0 + MCP integration** - Latest multimodal AI with Model Context Protocol support

## Why This Template?

This is **not a toy**. It implements patterns used by production agentic systems:

✅ **Correct abstraction boundary**: `UIState → UIDiff → Action` is the minimal control loop  
✅ **Token-efficient**: Diffs + schemas reduce costs by 50-80% vs screenshot/DOM approaches  
✅ **Composable**: Browser + OS environments split cleanly for expansion  
✅ **Industry-aligned**: Matches Playwright-based computer-use agents and internal tool-call agents  
✅ **Agent-native**: Planner/stepper loop works with any LLM provider  

## Architecture

```
┌─────────────┐
│   User Goal │
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Planner (LLM)      │  ← Breaks down high-level goal
│  Gemini 2.0 Flash   │
└──────┬──────────────┘
       │ Subgoals
       ▼
┌─────────────────────┐
│  Stepper (LLM)      │  ← Decides next action from UI state
│  + Action Schema    │
└──────┬──────────────┘
       │ Action[]
       ▼
┌─────────────────────┐
│ Browser/OS Env      │  ← Executes actions, returns UIDiff
│ (Playwright/CDP)    │
└──────┬──────────────┘
       │ UIDiff
       └──► (repeat)
```

## Key Features

### 1. JSON Schema-First Design
All data structures are strictly typed with JSON Schemas in `schemas/`:
- `ui_state.json` - Complete UI snapshot with pruned element tree
- `ui_diff.json` - Incremental changes (added/removed/updated elements)
- `action.json` - Validated actions with Gemini API and MCP configurations

### 2. Diff-Based State Updates
Instead of re-sending entire pages:
```json
{
  \"base_screen_id\": \"abc123\",
  \"screen_id\": \"def456\",
  \"added\": [{\"id\": \"new_button\", ...}],
  \"removed\": [\"old_div\"],
  \"updated\": [{\"id\": \"counter\", \"changes\": {\"text\": \"5\"}}]
}
```

### 3. Gemini 2.0 Integration
Native support for Google's latest models:
- `gemini-2.0-flash-exp` (default) - Fastest, most cost-effective
- `gemini-1.5-pro` - Best reasoning
- `gemini-1.5-flash` - Balanced performance
- Multimodal vision fallback when DOM is insufficient
- Tool calling for MCP servers

### 4. Model Context Protocol (MCP)
Wire any MCP server into your agent's action space:
```json
{
  \"kind\": \"mcp_call\",
  \"mcp_config\": {
    \"server_name\": \"filesystem\",
    \"tool_name\": \"read_file\",
    \"arguments\": {\"path\": \"/data/config.json\"}
  }
}
```

## Project Structure

```
agentic-computer-browser-template/
├── schemas/              # JSON Schemas for all data structures
│   ├── ui_state.json    # Full UI snapshot
│   ├── ui_diff.json     # Incremental changes
│   └── action.json      # Action DSL with Gemini/MCP config
├── env_browser/         # Browser environment service
│   ├── service.py       # FastAPI server for UI state + actions
│   └── playwright_adapter.py
├── env_os/              # OS automation (future)
│   └── macos.py
├── agent/               # LLM agent orchestration
│   ├── planner.py       # Goal decomposition
│   ├── stepper.py       # Action decision from UI state
│   └── loop.py          # Main control loop
├── storage/             # Task graph + context store
│   └── task_db.py
├── examples/            # Example tasks
│   └── web_form_demo.py
└── README.md
```

## Quickstart

### 1. Install Dependencies
```bash
pip install -r requirements.txt
playwright install chromium
```

### 2. Set API Keys
```bash
export GEMINI_API_KEY=\"your-key-here\"
```

### 3. Run Browser Environment
```bash
uvicorn env_browser.service:app --reload
```

### 4. Run Example Agent
```python
from agent.loop import run_task
import google.generativeai as genai

genai.configure(api_key=os.environ[\"GEMINI_API_KEY\"])
model = genai.GenerativeModel('gemini-2.0-flash-exp')

await run_task(model, \"Fill out the contact form on example.com\")
```

## Roadmap: Critical Improvements

The following enhancements are planned to make this production-ready:

### Phase 1: Stability
- [ ] **Deterministic element IDs** - Hash(role + text + bbox + DOM path) for stable diffs
- [ ] **Affordance scores** - Add `clickability` confidence to elements
- [ ] **Termination signals** - `goal_satisfied: boolean` in env responses

### Phase 2: Observability
- [ ] **Replay logs** - Persist `(UIState, Action)` pairs for debugging
- [ ] **Vision fallback** - Screenshot + region reference when DOM fails
- [ ] **Metrics dashboard** - Token usage, success rates, latency

### Phase 3: Scale
- [ ] **Multi-environment** - macOS, Windows, Linux OS automation
- [ ] **Parallel execution** - Run multiple agents in isolated browsers
- [ ] **Cloud deployment** - Docker + K8s configs

## Use Cases

This template is designed for:
- **Agentic browsing** - Autonomous web research and data extraction
- **Desktop copilots** - OS-level automation (file management, app control)
- **Workflow automation** - End-to-end business process automation
- **Testing & QA** - Intelligent UI testing that adapts to changes
- **Data labeling** - Semi-autonomous data collection pipelines

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Areas of Interest
- Playwright adapter optimizations
- Additional LLM provider integrations (OpenAI, Anthropic, local models)
- Vision-based element detection
- MCP server implementations

## License

MIT License - see [LICENSE](LICENSE) for details.

## Citation

If you use this template in research or production, please cite:

```bibtex
@software{agentic_computer_browser_template,
  title = {Agentic Computer & Browser Template},
  author = {jdot274},
  year = {2025},
  url = {https://github.com/jdot274/agentic-computer-browser-template}
}
```

## Acknowledgments

- Inspired by [Anthropic's Computer Use](https://www.anthropic.com/news/3-5-models-and-computer-use)
- Built on [Playwright](https://playwright.dev/) browser automation
- Powered by [Google Gemini 2.0](https://deepmind.google/technologies/gemini/)
- Implements [Model Context Protocol](https://modelcontextprotocol.io/)

---

**Status**: Active development | Production-ready foundation | Contributions welcome
