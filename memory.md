# DeepCode Project Memory

## Session: 2026-01-04

### Configuration
- **LLM Provider**: DeepSeek (via OpenAI-compatible API)
- **API Key**: `sk-cfbcf72a54af43aeb9365044bf238603`
- **Base URL**: `https://api.deepseek.com/v1`
- **Search**: Brave API configured

### Issues & Fixes

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| Unicode encoding error on startup | Windows console can't print banner | Run via `python -m streamlit run ui/streamlit_app.py` |
| Missing `mcp_agent` module | Package not installed | `pip install mcp-agent` |
| NumPy version conflict | numpy 2.x incompatible | `pip install "numpy<2"` |
| Missing `google.genai` | Required by mcp-agent | `pip install google-genai` |
| DeepSeek message format error | API expects string, got array | Patched `augmented_llm_openai.py` lines 650-661 |
| Invalid max_tokens (40000) | DeepSeek limit is 8192 | Set `base_max_tokens: 8192` in config |
| Planning step stuck/slow | Agents doing web searches instead of YAML output | Disabled web searches for planning agents |

### Key File Changes

1. **`mcp_agent.config.yaml`**
   - `llm_provider: "openai"` (for DeepSeek compatibility)
   - `base_max_tokens: 8192`, `retry_max_tokens: 8192`
   - Added Brave API key

2. **`mcp_agent.secrets.yaml`**
   - Added DeepSeek credentials under `openai:` section

3. **`C:\ProgramData\anaconda3\Lib\site-packages\mcp_agent\workflows\llm\augmented_llm_openai.py`**
   - Lines 650-661: Convert tool message content from array to string for DeepSeek

4. **`workflows/agent_orchestration_engine.py`**
   - Added progress callbacks with timing
   - Disabled web searches during planning (lines 692-698)

5. **`workflows/code_implementation_workflow.py`**
   - Added iteration progress logging

### What Works
- App runs at http://localhost:8501
- PDF upload and conversion
- Paper analysis with DeepSeek
- Code plan generation (score 1.00/1.0)
- File structure creation
- Progress tracking in logs

### Lessons Learned
1. DeepSeek has strict 8192 token limit - must configure explicitly
2. DeepSeek requires string content in tool messages (not arrays like OpenAI)
3. Disabling web searches during planning improves reliability and speed
4. Large papers (~66K chars) work but take ~5 min for planning phase

### Performance
- Paper analysis: ~2-3 min per agent
- Total planning: ~5 min for 66K char document
- Code generation: In progress
