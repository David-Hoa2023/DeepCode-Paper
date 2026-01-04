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

---

## Session: 2026-01-05

### Issue: CodeImplementationAgent Infinite Loop (2+ hours)

**Symptom**: Agent made tool calls continuously for ~2 hours without terminating
```
23:46:05 → 01:44:50 = ~2 hours of "Requesting tool call" logs
```

**Root Cause**: Path mismatch in `get_unimplemented_files()` method
- `all_files_list` contains paths like `project/src/model/file.py`
- `implemented_files` contains paths like `file.py` or `model/file.py`
- The fuzzy matching logic was too strict, never matching files as "implemented"
- Loop never detected completion → ran until 2-hour timeout

### Fixes Applied

| File | Change |
|------|--------|
| `workflows/agents/memory_agent_concise.py` | Improved `get_unimplemented_files()` path matching with 3 strategies: normalized path match, partial path suffix match, filename match |
| `workflows/code_implementation_workflow.py` | Added stuck-loop detection: exits after 50 iterations without new file implementations |

### Code Changes

1. **`memory_agent_concise.py` (lines 1929-1983)**: New `is_implemented()` with:
   - `normalize_path()`: Strips common prefixes like `generate_code/`, `code/`, `src/`
   - Strategy 1: Exact normalized path match
   - Strategy 2: Partial path suffix match with boundary check
   - Strategy 3: Same filename match (for unique filenames with extensions)

2. **`code_implementation_workflow.py` (lines 336-339, 478-498)**: Progress tracking:
   - Tracks `iterations_without_progress` counter
   - Exits with warning if no new files for 50 consecutive iterations
   - Logs remaining files for debugging

### Lessons Learned
1. Path normalization is critical when comparing file paths from different sources
2. Always add timeout/progress detection for long-running loops
3. The termination condition `get_unimplemented_files() == []` depends on accurate path matching

---

## Session Update: 2026-01-04 (continued)

### New Issue & Fix

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| File tree creation stuck | Full 15,735-char plan sent to DeepSeek for structure generation | Added `_extract_file_structure()` to extract only file_structure section (~4K chars max) |

### Code Changes
1. `workflows/code_implementation_workflow.py`: Added `_extract_file_structure()` (lines 94-113)
   - Reduces token usage by ~81% for file tree creation (12K → 2.4K chars)

2. `workflows/agent_orchestration_engine.py`: Added paper truncation (lines 675-686)
   - Truncates paper to 30K chars max for planning phase
   - Reduces planning time from ~5min to ~2-3min for large papers

3. `tools/command_executor.py`: Added Unix-to-Windows command translation (lines 116-140)
   - Translates `mkdir -p` → Windows `mkdir`
   - Translates `touch` → Windows `type nul >`
   - Fixes file tree creation failing on Windows

### Successful Run (23:38 - 23:43)
| Step | Time | Duration | Notes |
|------|------|----------|-------|
| Paper upload | 23:38:39 | - | PDF 1.89 MB, 18 pages |
| Paper truncation | 23:39:04 | - | 66K → 30K chars |
| AlgorithmAnalysisAgent | 23:41:04 | 2 min | Faster with truncation |
| CodePlannerAgent | 23:43:06 | 2 min | Score 1.00/1.0 |
| File tree creation | 23:43:58 | 30s | Windows commands working |
| Total planning | - | ~4 min | Down from ~5-6 min |

### Performance Summary
| Optimization | Impact |
|-------------|--------|
| Paper truncation (30K max) | ~30% faster planning |
| File structure extraction | ~80% fewer tokens for tree creation |
| Unix→Windows translation | File tree creation works on Windows |
| Disabled web searches | More reliable YAML output |
