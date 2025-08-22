# AI Agents Workshop – From Idea to Working Prototype

> Let AI do the work, not just the talking.

This repository supports the hands‑on part of a workshop where you learn to build autonomous AI agents with **opencode** and the **Model Context Protocol (MCP)**. You start with a minimal but realistic multi‑agent + tool configuration and extend it step by step (adding new agents, adding new context/tool servers, and eventually producing real deliverables).

---

## 1. Workshop Vision

You already know what a single LLM chat can do. The next level is an **Agent System** that:
- Pursues goals until completion (not just one reply)
- Delegates to specialized sub‑agents
- Leverages external context/tools (MCP servers)
- Produces concrete outputs: validation notes, market insights, architecture, roadmap, and code

You will leave with:
- Conceptual understanding (main agent vs. sub‑agents; when to use an agent vs. a plain LLM call)
- A working mini multi‑agent setup
- An added core capability
- A specialized sub‑agent
- An additional MCP server for richer context
- A generated product plan + starter prototype code

---

## 2. Repository Overview

| File | Purpose |
|------|---------|
| `opencode.json` | Core configuration (agents + MCP servers) |
| `.gitignore` | (Currently empty placeholder) |

Current MCP servers and their intent:

| MCP Server | Type | Purpose | Typical Trigger Style |
|------------|------|---------|------------------------|
| `sequential-thinking` | Local reasoning enhancer | Provides structured multi-step thought scaffolding (explicit chain-of-thought style planning without leaking internal reasoning unless summarized) | Often auto-invoked when the agent detects a multi-part objective OR explicitly: "Use sequential thinking to break this down." |
| `playwright` | Local browser automation / UI exploration | Enables scripted browsing, DOM inspection, screenshots, interaction for tasks like "open the page and extract X" or "take a screenshot" | Usually explicit: "Use the playwright tool to visit https://example.com and capture a screenshot." |
| `github` | Remote (Copilot) GitHub context / code operations | Supplies repository information, PR diffs, issues, code search, commit data | Auto when referencing repo/PR specifics (e.g. "List open PRs"), or explicit: "Use the GitHub tool to fetch the diff of PR #12." |

Trigger strategy explanation:
- **Implicit (semantic) trigger**: The agent decides a tool is relevant from your higher-level instruction (e.g., "Break this big task into smaller logical steps" may cause use of `sequential-thinking`).
- **Explicit trigger**: You name the tool or describe an action uniquely tied to it (e.g., "Use playwright to navigate…" or "Invoke the GitHub MCP tool to list issues.")
- **Best practice**: For deterministic workshop outcomes, be explicit first; later experiment with implicit prompting to observe autonomy.

Current (truncated) `opencode.json`:
```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "github-copilot/claude-sonnet-4",
  "small_model": "github-copilot/gpt-5-mini",
  "agent": {
    "build": {
      "model": "github-copilot/claude-sonnet-4",
      "description": "Building and implementing features with detailed code generation",
      "mode": "primary"
    },
    "plan": {
      "model": "github-copilot/gpt-5",
      "description": "Strategic planning and architectural decisions",
      "mode": "primary"
    },
    "simple-tasks": {
      "model": "github-copilot/gpt-5-mini",
      "description": "Quick assistance with simple or repetitive tasks",
      "mode": "subagent"
    }
  },
  "mcp": {
    "sequential-thinking": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "playwright": {
      "type": "local",
      "command": ["npx", "-y", "@playwright/mcp@latest"]
    },
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

---

## 3. Prerequisites

- Node.js (LTS 18+)
- Git + a GitHub account (Copilot access for listed models)
- (Optional) API keys for other model providers if you extend beyond defaults
- `npm` / `npx` available locally

Optional environment file (`.env`) if you later introduce additional model/tool credentials:
```
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
```

---

## 4. Getting Started

Clone and enter the project:

```bash
git clone <REPO_URL> ai-agent-workshop
cd ai-agent-workshop
```

Run opencode (examples, depends on your global/local install):

```bash
npx opencode chat
# or
opencode chat
```

Verify:
- Agents visible: `build`, `plan`, `simple-tasks`
- MCP servers registered: `sequential-thinking`, `playwright`, `github`

---

## 5. How You Will Work

Iterative loop:
1. Inspect current config
2. Add or modify an agent / MCP server
3. Ask the main agent to leverage the new capability
4. Compare results vs. a single "raw" LLM call (optional baseline)
5. Refine

---

## 6. Assignments (Initial Set)

### Assignment 1: Add a (Sub)Agent (with Planning Support)

Goal: Introduce a specialized capability AND learn to co-design its instruction set using existing `plan` + `build` agents.

**Design workflow (recommended):**
1. Prompt the `plan` agent:  
   "I want to add a new sub-agent focused on market/domain research. Propose: name, concise mission statement, ideal model choice, boundaries (what NOT to do), interaction patterns with existing agents, and an example delegation prompt."
2. Review and refine the proposal. Ask `plan` to iterate if unclear.
3. Once satisfied, ask the `build` agent:  
   "Generate the JSON snippet for `opencode.json` under the agent section for the proposed sub-agent. Make sure it uses mode=subagent and include a polished description embedding boundaries."
4. Insert the snippet into `opencode.json`.
5. Test via primary agent prompt referencing the new role.

Example refined snippet (for illustration):
```jsonc
"research": {
  "model": "github-copilot/gpt-5",
  "description": "Market & domain analysis specialist. Produces concise factual scans, competitor grids, pains/opportunities. Avoids coding, architectural decisions, or implementation-level detail. Escalate architectural topics to 'plan'; delegate code generation to 'build'.",
  "mode": "subagent"
}
```

**Validation prompts:**
- "Use the research sub-agent to summarize top 5 competitive differentiators in warehouse robotics SaaS."
- "Have research produce a competitor table; then have plan integrate it into a positioning statement."

**Checklist:**
- New agent appears in introspection
- Delegation evidence (agent references sub-agent)
- Output is more structured or faster than doing it manually

**Stretch ideas:**
- Add a `constraints` / style instruction in description
- Create a complementary `qa` agent and have `build` ask it to review generated code

**Common pitfalls:**
- Typos in JSON (validate)
- Wrong `mode` (should be `"subagent"` for a non-primary helper)
- **New Best Practice**: Always co-create agent specs using `plan` (ideation) then `build` (structuring/formatting).

---

### Assignment 2: Add a New MCP Server (Context Enrichment)

Goal: Expand external context surface (e.g. `context7` or another structured knowledge provider) AND learn to design its integration using existing agents.

**Design workflow:**
1. Use `plan` agent: "Propose value statement, risks, and instruction pattern for integrating a new MCP server providing <domain/context>."
2. Use `build` agent: "Generate the JSON config block for an MCP server named 'context7' that runs via npx … Include brief inline comment with usage hint."
3. Add block under `mcp`.
4. **Explicit test prompt:**  
   "Use the context7 tool to retrieve high-level drivers for <domain>. Summarize them, cite tool output sections, and propose follow-up research questions."
5. **Implicit test prompt:**  
   "Identify major trends shaping <domain> and show which external data source(s) you consulted." (Observe if the agent autonomously calls the server.)

Generic pattern to add a local MCP server:

```jsonc
"my-context": {
  "type": "local",
  "command": ["npx", "-y", "<mcp-package-name>"]
}
```

If `context7` is a custom or third-party server:
1. Identify install / run command (`npx -y context7-mcp` or similar).
2. Add block under `mcp` in `opencode.json`.
3. (If private) `npm install <package>` first, or point to a script.
4. Restart opencode; confirm the tools (list or help output).

**Validation:**
- Tools become listed
- Invocation returns structured data
- Agent integrates that data into reasoning (not hallucinated)

**Stretch:**
- Add input guardrails (ask agent first to outline what data it will request)
- Combine with sequential-thinking tool to refine raw MCP results

**Pitfalls:**
- Forgetting install, causing command not found
- Mismatch in command array (each token separate)
- Using remote type without correct `url`

---

## 7. After These Two Assignments

Next likely steps (not part of the initial mandatory list, but recommended):
- Add a "decision-record" agent producing lightweight ADRs
- Introduce a planning refinement loop: `plan` -> `research` -> `plan`
- Add a code quality MCP server (linting / static analysis)
- Benchmark: Same prompt via a plain single LLM vs. orchestrated agent system

---

## 8. Prompt Patterns

**Good delegation prompts:**
- "Break this task into sub-tasks and assign each to the best sub-agent. Execute sequentially. Show a final integrated summary."
- "Before coding, have the plan agent validate scope; only then let build generate code."

**Tool-conscious prompt templates:**
- **Sequential Thinking (explicit):**  
  "Use a structured multi-step reasoning process (sequential thinking tool) to decompose this requirement before proposing a plan."
- **Playwright (explicit):**  
  "Use the playwright tool to open https://example.com, extract the main heading text, and return a screenshot."
- **GitHub (explicit):**  
  "Use the GitHub tool to list open issues labeled 'bug' and summarize recurring themes."
- **Mixed (implicit + explicit):**  
  "Gather current competitive insights (research sub-agent) and if repository context is relevant, pull recent commit messages (GitHub tool) to infer active focus areas."

**Escalation pattern:**
1. High-level objective
2. Ask for decomposition (may trigger sequential-thinking)
3. Delegate research/passive data gathering (research sub-agent + MCP context server)
4. Synthesize via plan
5. Implement via build
6. (Optional) Validate via QA / simple-tasks

**Evaluation prompt (baseline comparison):**
> Provide a market validation + architecture + first feature spec for a SaaS idea that automates <X>. (Ask once with agent system, once with plain LLM.)

Compare:
- Structure
- Depth
- Consistency
- Reusability of intermediate artifacts (tasks, decisions)

---

## 9. Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| New agent ignored | Not referenced / no delegation explicit | Prompt main agent to use it; update description to clarify niche |
| MCP server missing | Command fails | Run manually to test; ensure package installed |
| JSON parse error | Trailing comma / invalid syntax | Use a JSON linter |
| Tool returns empty | Needs auth / env var | Add variable to `.env` and restart |
| Tool never auto-triggers | Description too vague / agent not recognizing pattern | Add explicit mention in prompt; enrich server description with verbs ("analyze", "fetch", "inspect") |
| Sub-agent overreaches (does coding) | Missing boundary clause | Refine description: state "Avoid code; delegate to build" |
| Repeated tool invocation loops | Ambiguous goal phrasing | Provide termination criteria: "Stop after first structured summary unless contradictions found." |

---

## 10. Recommended Mindset

Think in systems:
- Each sub-agent should have a single, high-signal responsibility
- Add tools only when they materially reduce hallucination or manual steps
- Measure difference vs. baseline (ask: did autonomy reduce your cognitive load?)

---

## 11. License / Usage

Internal workshop educational material. Adapt as needed for experimentation.

---

## 12. Quick Action Summary

| Action | Command / Edit |
|--------|----------------|
| Start chat | `npx opencode chat` |
| Add sub-agent | Edit `opencode.json` under `agent` |
| Add MCP server | Edit `opencode.json` under `mcp` |
| Reload | Restart the opencode CLI |
| Ask plan to draft sub-agent spec | Prompt inside chat |
| Ask build to format JSON snippet | Prompt inside chat |
| Force explicit tool invocation | Mention tool name + action |
| Test implicit triggering | Omit tool name; give semantic goal |

---

## Appendix: Fast Design Prompt Templates

**Sub-agent spec (to `plan`):**
> You are designing a new sub-agent for <focus>. Provide: name, 1-line mission, responsibilities list, explicit non-responsibilities, ideal model choice (from existing), collaboration notes with existing agents, and 2 example delegation prompts.

**JSON generation (to `build`):**
> Convert the approved spec into a valid JSON object to insert under the agent section. Use mode=subagent. Keep description under 500 chars, embed boundaries.

**MCP server addition (to `plan`):**
> Propose integration details for a new MCP server giving <data type>. Include purpose, risk mitigation, and explicit invocation verbs.

---

Happy building—compose your digital team and let it work end‑to‑end!