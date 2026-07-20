You are helping refactor an internal TCA (Transaction Cost Analysis) AI query tool built in Python for use in JupyterHub. Before making any changes, you need to fully understand the existing codebase.

## Phase 1 — Read and Understand (do this first, make no changes)

Read every file in this project. For each file, note:
- Its purpose and what it is responsible for
- How it connects to other files (imports, calls, data it passes)
- Any logging it currently does and where logs are written
- Any error handling it currently does
- Any hardcoded values, credentials, or config that should be externalised

Once you have read everything, produce a written summary covering:

1. **Current architecture** — a description of all components and how data flows between them, from the user's query in the ipywidgets UI through to the rendered output
2. **Current logging** — exactly what is logged today, where, and in what format
3. **Current error handling** — how each failure mode is currently handled (or not handled)
4. **Structural issues** — anything that is hardcoded, poorly separated, or that would make the codebase hard for a 40-person team to work on
5. **What is working well** — things to preserve in the refactor

Do not suggest changes yet. Do not write any code yet. Just produce this summary and wait.

---

## Phase 2 — Propose Changes (only after I confirm the summary looks right)

Once I have confirmed the summary is accurate, propose the refactored structure as text only. No code yet.

The target architecture should follow these principles, which come from our internal design documentation:

**Module separation**
The codebase should be split into these logical modules, each with a single clear responsibility:
- `session_manager.py` — conversation state, obfuscation map, turn history, in-memory DataFrame store
- `obfuscation.py` — obfuscate and deobfuscate sensitive values against the client config; validate all labels; raise typed errors on failure
- `skill_loader.py` — select and load skill markdown files based on query content
- `prompt_builder.py` — assemble the full prompt from system instructions, skills, client config, conversation history, and current query
- `claude_gateway.py` — send to and receive from the Claude endpoint (currently Mattermost webhook); parse and validate the JSON envelope response; retry on malformed output
- `execution_engine.py` — construct the API payload, call `utils.sendtcarequest()`, run post-processing code in a restricted environment (pandas and numpy only)
- `renderer.py` — render DataFrames as tables and generate Matplotlib charts
- `logger.py` — all logging, nothing else
- `tca_notebook.ipynb` — UI only, no logic; thin ipywidgets layer over the modules above

**Logging**
Every turn should produce a folder of structured logs written in real time (not buffered). The structure should be:

```
~/tca_tool_logs/sessions/{date}{session_id}/turn{n}/
query_raw.txt
query_obfuscated.txt
mattermost_sent.json
mattermost_received.json
claude_parsed.json
api_payload.json
api_response_meta.json
postprocessing_code.py
output_chart.png
execution_log.txt
```
A `session_summary.md` should be maintained at the session level as a human-readable running log. Log retention should be configurable, defaulting to 7 days for turn logs and 30 days for charts.

**Error handling**
The following four failure types should each be handled explicitly with a clear typed exception and a user-facing message:
- Parse failure — Claude returned malformed or non-JSON output
- Deobfuscation failure — Claude used an unknown entity label not in the obfuscation map
- API failure — `sendtcarequest` returned an error; message should quote the API error and suggest the likely correct field name from the config
- Post-processing failure — generated Python code threw an exception; show the exception and the code that caused it

**Credentials**
User credentials (`email`, `api_token`) must be read from `~/.tca_config.json` and never appear in notebook code or any committed file.

**Config**
Client config should be loaded from `config/clients/{client_moniker}_config.json`. The config structure should include: available endpoints, field dictionary (API name → human meaning → obfuscated label), sensitive value registry (all known broker names, PM names, benchmark names → obfuscated equivalents), and client conventions.

**Claude response format**
The gateway should expect and validate this JSON envelope from Claude:
```json
{
  "response_type": "api_call | code | text | clarification",
  "explanation": "...",
  "api_payload": { ... },
  "post_processing_code": "...",
  "output_format": "table | chart | text",
  "chart_spec": { "type": "...", "x": "...", "y": "...", "title": "..." }
}

For the proposed changes, describe:
Which new files need to be created and what goes in each
Which existing files need to be split, merged, or significantly restructured
What the new logging structure looks like and how it differs from current
What changes are needed to the notebook itself
Any migration considerations — things that could break and need care
Present this as a clear written plan. No code yet. Wait for my confirmation before proceeding.
Phase 3 — Implement (only after I confirm the plan)
Once I have confirmed the proposed changes, implement them. Follow these rules:
Make changes incrementally, one module at a time, starting with logger.py and working outward
After each module, confirm it is complete before moving to the next
Do not modify the notebook until all backend modules are complete
After all modules are written, update the notebook to remove logic and replace it with calls to the new modules
Where existing behaviour is unclear or ambiguous, ask rather than assume
If you encounter something in the existing code that conflicts with the target architecture described above, flag it explicitly and ask how to resolve it before proceeding
Do not rename or restructure the config files or credentials without explicit confirmation
Preserve all existing functionality — this is a refactor, not a rewrite of behaviour
