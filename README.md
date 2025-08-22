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

## 2. Core Concepts

| Term | Short definition |
|---|---|
| Agent | Autonomous actor with a single, clear responsibility (primary or helper). |
| Sub‑agent | Lightweight agent invoked by a primary agent for a focused subtask. |
| MCP Server | External process that hosts tools/agents and exposes an API for opencode. |
| Tool | A callable capability provided to an agent (e.g., read/write, shell, webfetch). |
| Model Call | Invocation of an LLM model with inputs, temperature, and token budget. |

---

## 3. Why Multi‑Agent vs Single LLM

**Benefits of agent systems:**
- **Goal persistence:** Agents pursue objectives until completion, not just one reply
- **Specialized delegation:** Different sub‑agents handle research, planning, coding, validation
- **Tool integration:** Leverage external context (GitHub, browser, databases) seamlessly
- **Incremental refinement:** Plan → research → validate → implement → review cycles
- **Reusable artifacts:** Intermediate outputs (specs, decisions) become building blocks

**Before/after comparison:**
```
Single LLM: "Build a SaaS for inventory management"
→ Generic advice, no context, requires manual follow‑up

Agent system: Same prompt
→ research agent gathers market data
→ plan agent creates architecture + roadmap
→ build agent generates starter code
→ Produces: competitor analysis, technical spec, working prototype
```

---

## 4. Repository Overview

| File | Purpose |
|------|---------|
| `opencode.json` | Core configuration (agents + MCP servers) |
| `.gitignore` | (Currently empty placeholder) |

Current MCP servers and their intent:

| MCP Server | Type | Purpose | Typical Trigger Style |
|------------|------|---------|------------------------|
| `sequential-thinking` | Local reasoning enhancer | Provides structured multi-step thought scaffolding (explicit chain-of-thought style planning without leaking internal reasoning unless summarized) | Often auto-invoked when the agent detects a multi-part objective OR explicitly: "Use sequential thinking to break this down." |
| `playwright` | Local browser automation / UI exploration | Enables scripted browsing, DOM inspection, screenshots, interaction for tasks like "open the page and extract X" or "take a screenshot" | Usually explicit: "Use the playwright tool to test out the contact form on https://example.com/contact." |
| `github` | Remote (Copilot) GitHub context / code operations | Supplies repository information, PR diffs, issues, code search, commit data | Auto when referencing repo/PR specifics (e.g. "List open PRs"), or explicit: "Use the GitHub tool to fetch the diff of PR #12." |

Trigger strategy explanation:
- **Implicit (semantic) trigger**: The agent decides a tool is relevant from your higher-level instruction (e.g., "Break this big task into smaller logical steps" may cause use of `sequential-thinking`).
- **Explicit trigger**: You name the tool or describe an action uniquely tied to it (e.g., "Use playwright to navigate…" or "Invoke the GitHub MCP tool to list issues.")
- **Best practice**: For deterministic workshop outcomes, be explicit first; later experiment with implicit prompting to observe autonomy.

---

## 5. Quick Start

### Prerequisites

- A modern terminal
- Install opencode following the instructions at https://opencode.ai/docs/#install
- Once installed use your GitHub Copilot subscription with opencode following the instructions here: https://opencode.ai/docs/providers/#github-copilot

### First Run

Clone and enter the project:

```bash
git clone <REPO_URL> ai-agent-workshop
cd ai-agent-workshop
```

Run opencode:

```bash
opencode
```

**Important:** After making any changes to `opencode.json`, you must exit opencode and restart it for changes to take effect. Use `opencode --continue` to resume your previous session.

**First-time setup:** Initialize agent guidelines by running:

```
/init
```

This command creates an `AGENTS.md` file with coding guidelines, build commands, and best practices for AI agents working in this repository. The file serves as a reference for agents to maintain consistency and follow project conventions.

### Verify Setup

- Agents visible: `build`, `plan`, `simple-tasks` - see the [usage instructions](https://opencode.ai/docs/agents/#usage) to learn how to switch between them.
- MCP servers registered: `sequential-thinking`, `playwright`, `github`
- `AGENTS.md` file created with project guidelines

---

## 6. Working with Files and Context

### File Referencing with @
When working with agents, you can reference specific files using the `@` syntax for precise context:

**Examples:**
- `@opencode.json` - Reference the main configuration file
- `@README.md` - Include the entire README content
- `@src/components/Header.js` - Reference a specific source file

**Best practices:**
- Use `@filename` when asking agents to analyze, modify, or understand specific files
- Combine with delegation: "Have the build agent update @package.json with the new dependency"
- Reference multiple files: "Review @opencode.json and @AGENTS.md to ensure consistency"

### Iterative Workflow
1. Reference current config with `@opencode.json`
2. Add or modify an agent / MCP server
3. Ask the main agent to leverage the new capability
4. Use file references for precise context: "@opencode.json shows we have X agents..."
5. Compare results vs. a single "raw" LLM call (optional baseline)
6. Refine using targeted file updates

---

## 7. Essential Prompt & Design Foundations

### Meta‑prompting: use an agent to draft and refine prompts

Use a helper agent to design, refine, and sanity‑check prompts before sending them to the executor agent. Benefits: faster iteration, fewer failed runs, clearer intent, and easier reproducibility.

**Quick pattern:**
1. Create a small "prompt‑engineer" agent with task: "Draft a 2‑step prompt for <target agent> to achieve X."
2. Ask it to supply: (a) short prompt, (b) fallback prompt, (c) 2 validation checks.
3. Run the target agent with the short prompt; if checks fail, run fallback.

### Agent Description Essentials

**Basic checklist:**
- Purpose: one sentence goal
- Boundaries: what it must NOT do
- Model hint: preferred model class
- Validation: 1–2 test prompts

### Model Selection Quick Reference

| Goal | Preferred model | Why |
|---|---|---|
| Complex planning | higher‑capability models | better reasoning |
| Simple tasks | smaller/fast models | cheaper, lower latency |
| Safety‑critical | conservative models | fewer hallucinations |

Reference: [Copilot model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)

---

## 8. Assignment 1: Add a (Sub)Agent (with Planning Support)

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
- Look at more configuration options in the [docs](https://opencode.ai/docs/agents/#configure).
- Add a `constraints` / style instruction in description
- Create a complementary `qa` agent and have `build` ask it to review generated code

**Common pitfalls:**
- Typos in JSON (validate)
- Wrong `mode` (should be `"subagent"` for a non-primary helper)
- **New Best Practice**: Always co-create agent specs using `plan` (ideation) then `build` (structuring/formatting).

### Assignment 2: Add a New MCP Server (Context Enrichment)

Goal: Add Context7 MCP server to provide up-to-date code documentation and examples directly in your prompts. This demonstrates how external MCP servers can dramatically enhance agent capabilities with real-world data.

**What is Context7?**
Context7 is an MCP server that provides up-to-date, version-specific documentation and code examples from popular libraries and frameworks. Instead of relying on potentially outdated training data, it fetches current documentation directly from source repositories.

**MCP servers (curated list):**
A community‑curated list of ready‑to‑use MCP servers and adapters: https://github.com/punkpeye/awesome-mcp-servers

**Design workflow:**
1. Use `plan` agent: "Analyze the value proposition of adding Context7 MCP server to our workshop environment. What coding scenarios would benefit most from up-to-date documentation access?"
2. Use `build` agent: "Generate the JSON config block for adding Context7 MCP server to our opencode.json. Use the npm package '@upstash/context7-mcp' and include usage hints."
3. Add the configuration to `opencode.json` under the `mcp` section.
4. Restart opencode to register the new server.
5. **Test the integration** with specific prompts.

**Configuration to add:**

```jsonc
"context7": {
  "type": "local", 
  "command": ["npx", "-y", "@upstash/context7-mcp"]
}
```

**Test prompts (try these after setup):**
1. **Explicit test:** 
   "Use Context7 to get current Next.js documentation for creating middleware. Show me how to implement JWT authentication in middleware."
   
2. **Implicit test:** 
   "Create a React component using the latest Chakra UI patterns for a user profile card." (Observe if Context7 is automatically invoked)

3. **Library-specific test:**
   "Get documentation for Supabase authentication setup and show me how to implement social login with Google."

**Validation checklist:**
- [ ] Context7 appears in MCP server list (check opencode startup logs)
- [ ] Tools `get-library-docs` and `resolve-library-id` are available
- [ ] Explicit invocation returns current documentation (not generic/outdated info)
- [ ] Agent can cite specific Context7 sources in responses
- [ ] Code examples use current API patterns and syntax

**Expected benefits:**
- **No more hallucinated APIs:** Real, current documentation instead of made-up methods
- **Version-specific examples:** Code that actually works with current library versions  
- **Reduced context switching:** Documentation flows directly into your development workflow
- **Up-to-date patterns:** Latest best practices instead of outdated tutorials

**Stretch goals:**
- Ask Context7 for documentation on libraries you're actively using in projects
- Compare responses before/after Context7 integration for the same coding question
- Try implicit triggering by asking coding questions without mentioning Context7 explicitly

**Common issues:**
- If installation fails, try `bunx` instead of `npx` in the command array
- Ensure Node.js >= v18.0.0 for compatibility  
- Check that the server appears in logs during opencode startup
- API rate limits apply; consider getting a free API key from context7.com for higher limits

---

## 9. Parallel Delegation Essentials

You can delegate the same task to multiple (sub)agents concurrently by instructing opencode to run them in parallel. Use explicit language: "Run agents A,B,C in parallel and collect outputs."

**Caveats and best practices:**
- **Cost & latency:** parallel model calls multiply token usage and wall time.
- **Race conditions:** concurrent edits to the same files can conflict. Prefer unique targets or coordination files.
- **Serialized critical sections:** for file writes or DB updates, use a "lock" agent or instruct a single agent to perform merges/commits.
- **Conflicting outputs:** implement a deterministic merge/validation step (e.g., a reconciler agent) to choose or combine outputs.

---

## 10. Extended Design & Maintenance

### Updating AGENTS.md — what to add and why

**What to add:**
- New agent specs with rationales (why it exists, intended inputs/outputs).
- Clear boundaries: what the agent must NOT do.
- Decision records: short notes for choices (model, tools, triggers).
- Validation prompts: a small set of tests/prompts to confirm expected behaviour.

**How it helps:**
- Increases consistency across agents and reduces accidental overlap.
- Improves accuracy of automatic triggers and auto‑routing.
- Speeds onboarding and reduces review friction.

### Model Selection Rationale (Expanded)

Reference: [Copilot model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)

**Detailed tradeoffs:**
| Goal | Preferred model class | Why | When to upgrade | When to downgrade |
|---|---|---|---|---|
| Complex planning, long coherence | higher‑capability models | better reasoning, costlier | Subagent fails validation consistently | Cost/latency dominates |
| Short, high‑throughput tasks | smaller/fast models | cheaper, lower latency | Need better multi‑step reasoning | Outputs remain acceptable |
| Safety‑critical / deterministic | conservative models + lower temp | fewer hallucinations | Error rate too high | Speed requirements increase |

### Writing Agent Descriptions — Full Examples

**Complete checklist:**
- Purpose: one sentence goal
- Inputs: explicit required inputs
- Outputs: exact artefacts produced
- Boundaries: what it must NOT do
- Triggers: how it is invoked (manual/auto/event)
- Model/tool hints: preferred model class and tools
- Validation: 1–3 prompts to check correct behaviour

Example repo for reference: https://github.com/wshobson/agents

**Good vs bad examples:**
- **Good (concise, precise):**
  - "build:primary — Prepare and run CI for pull requests. Inputs: repo, branch. Outputs: build log, artifacts. Tools: shell, read, write. NOT: deploy."
- **Good (focus on boundary):**
  - "doc:updater — Auto‑update API docs from comments. Inputs: repo, commit range. Outputs: PR with docs. NOT: change API code or minor refactors."
- **Bad (vague):**
  - "helper — helps with repo tasks, use as needed." (too broad, no inputs/outputs)

---

## 11. Validation & Quality

### Quality Rubric (5-point scale)

Rate agent outputs on:
1. **Structure:** Clear organization and logical flow
2. **Boundary adherence:** Stays within defined role limits
3. **Evidence use:** Cites tool outputs and external sources
4. **Factual grounding:** Avoids hallucination, verifiable claims
5. **Delegation clarity:** Appropriate hand-offs to other agents

### Golden Prompt Pattern

Create baseline comparison by running the same high-level prompt:
1. Once with your agent system (orchestrated)
2. Once with a single LLM call (baseline)

Compare: structure, depth, consistency, reusability of intermediate artifacts.

---

## 12. After the Initial Assignments

Next likely steps (not part of the initial mandatory list, but recommended):
- Add a "decision-record" agent producing lightweight ADRs
- Introduce a planning refinement loop: `plan` -> `research` -> `plan`
- Add a code quality MCP server (linting / static analysis)
- Benchmark: Same prompt via a plain single LLM vs. orchestrated agent system

---

## 13. Prompt Patterns

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

## 14. Troubleshooting

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

## 15. Recommended Mindset

Think in systems:
- Each sub-agent should have a single, high-signal responsibility
- Add tools only when they materially reduce hallucination or manual steps
- Measure difference vs. baseline (ask: did autonomy reduce your cognitive load?)

---

## 16. Resources

### MCP Servers
- **Curated list:** https://github.com/punkpeye/awesome-mcp-servers
- **Agent examples:** https://github.com/wshobson/agents
- **Model comparison:** https://docs.github.com/en/copilot/reference/ai-models/model-comparison

### OpenCode Documentation
- **Installation:** https://opencode.ai/docs/#install
- **GitHub Copilot setup:** https://opencode.ai/docs/providers/#github-copilot
- **Agent usage:** https://opencode.ai/docs/agents/#usage

---

## 17. License / Usage

Internal workshop educational material. Adapt as needed for experimentation.

---

## 18. Quick Action Summary

| Action | Command / Edit |
|--------|----------------|
| Start chat | `npx opencode chat` |
| Continue session | `opencode --continue` |
| Initialize guidelines | `/init` |
| Add sub-agent | Edit `opencode.json` under `agent` |
| Add MCP server | Edit `opencode.json` under `mcp` |
| Reload | **Exit and restart opencode** (required after `opencode.json` changes) |
| Ask plan to draft sub-agent spec | Prompt inside chat |
| Ask build to format JSON snippet | Prompt inside chat |
| Force explicit tool invocation | Mention tool name + action |
| Test implicit triggering | Omit tool name; give semantic goal |
| Use meta-prompting | Create prompt-engineer subagent first |
| Run agents in parallel | "Run agents A,B,C in parallel and collect outputs" |

---

## 19. Appendix: Fast Design Prompt Templates

**Sub-agent spec (to `plan`):**
> You are designing a new sub-agent for <focus>. Provide: name, 1-line mission, responsibilities list, explicit non-responsibilities, ideal model choice (from existing), collaboration notes with existing agents, and 2 example delegation prompts.

**JSON generation (to `build`):**
> Convert the approved spec into a valid JSON object to insert under the agent section. Use mode=subagent. Keep description under 500 chars, embed boundaries.

**MCP server addition (to `plan`):**
> Propose integration details for a new MCP server giving <data type>. Include purpose, risk mitigation, and explicit invocation verbs.

---

Happy building—compose your digital team and let it work end‑to‑end!
