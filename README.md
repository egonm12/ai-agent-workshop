# AI Agents Workshop – From Idea to Working Prototype

> From single prompts to a coordinated AI delivery loop.

## 0. Audience & Prerequisites [Core]

Designed for software engineers comfortable with Git, Node (≥18), and basic LLM usage. You'll evolve a minimal multi-agent system (opencode + MCP) into a richer autonomous setup producing reusable artifacts (research summaries, specs, starter code).

**You'll achieve (≈60–90 min):**
- Add a research sub-agent (bounded role)
- Integrate a documentation MCP server
- Generate a product/feature spec + prototype seed

**Required:** Node ≥18, GitHub Copilot access configured for opencode, Git.
**Optional:** GitHub PAT for private repo interactions.

---

## 1. Workshop Vision [Core]

If you've used ChatGPT/Copilot before, this extends that experience into a coordinated team of AI helpers that:
- Pursue goals until completion (not just one reply)
- Delegate to specialized sub‑agents
- Leverage external context/tools (MCP servers)
- Produce concrete, reusable outputs: specs, decisions, starter code

**Outcomes:**
- Conceptual understanding (when to use agents vs. plain LLM calls)
- Working multi‑agent setup with your additions
- Research sub‑agent + documentation MCP server
- Generated product plan + prototype seed

---

## 2. Quick Start (TL;DR) [Core]

1. Clone & enter: `git clone <REPO_URL> ai-agent-workshop && cd ai-agent-workshop`
2. Launch: `opencode`
3. Initialize (first run): `/init`
4. List available (sub)agents: `/agents`
5. Design research sub-agent:
   - plan → propose spec
   - build → JSON snippet
   - Add to `opencode.json`, restart
6. Test: "Use research to summarize 3 competitors in <your domain>."
7. (Optional) Add Context7 MCP server; restart; fetch live docs.
8. Baseline comparison: Same feature spec via (a) agent system vs (b) single LLM.

**Common First Errors:**
- Forgot restart after config edit
- Missing `"mode": "subagent"`
- Overly vague description (no triggers)
- Trailing commas in JSON

---

## 3. Core Concepts [Core]

| Term | Short definition |
|---|---|
| Agent | Autonomous actor with a single, clear responsibility (primary or helper). |
| Sub‑agent | Lightweight agent invoked by a primary agent for a focused subtask. |
| MCP Server | External process that hosts tools/agents and exposes an API for opencode. |
| Tool | A callable capability provided to an agent (e.g., read/write, shell, webfetch). |
| Model Call | Invocation of an LLM model with inputs, temperature, and token budget. |

---

## 4. Built-In Agents [Core]

| Agent | Mode | Purpose | Avoid |
|-------|------|---------|-------|
| plan | primary | Decompose goals; propose specs/sequences | Full code generation |
| build | primary | Turn approved specs into config/code | Market analysis / ideation |
| simple-tasks | primary (utility) | Fast, bounded tasks (summaries, minor edits) | Deep multi-step planning |

---

## 5. Why Multi‑Agent vs Single LLM [Core]

**Benefits of agent systems:**
- **Goal persistence:** Pursue objectives until completion, not just one reply
- **Specialized delegation:** Different sub‑agents handle research, planning, coding, validation
- **Tool integration:** Leverage external context seamlessly
- **Reusable artifacts:** Intermediate outputs (specs, decisions) become building blocks

**Before / After:**
- Single LLM: Generic, context-light; manual glue between steps
- Multi-agent chain: Structured (research → plan → build); reusable artifacts; less re-prompt overhead

---

## 6. Repository Overview [Reference]

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

## 7. Core Assignments [Core]

### Assignment 1: Add a Research Sub-Agent

Goal: Introduce a specialized capability using existing `plan` + `build` agents to co-design the spec.

**Design workflow:**
1. Prompt the `plan` agent:
   "I want to add a new sub-agent focused on market/domain research. Propose: name, concise mission statement, ideal model choice, boundaries (what NOT to do), interaction patterns with existing agents, and an example delegation prompt."
2. Review and refine the proposal. Ask `plan` to iterate if unclear.
3. Once satisfied, ask the `build` agent:
   "Generate the JSON snippet for `opencode.json` under the agent section for the proposed sub-agent. Make sure it uses mode=subagent and include a polished description embedding boundaries."
4. Insert the snippet into `opencode.json`.
5. **Restart opencode** (required for config changes).
6. Test via primary agent prompt referencing the new role.

**Example refined snippet (for illustration):**
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
- [ ] New agent appears in introspection
- [ ] Delegation evidence (agent references sub-agent)
- [ ] Output is more structured or faster than doing it manually

### Assignment 2: Add Context7 MCP Server

Goal: Add Context7 MCP server to provide up-to-date code documentation and examples directly in your prompts.

**What is Context7?**
Context7 provides up-to-date, version-specific documentation and code examples from popular libraries and frameworks. Instead of relying on potentially outdated training data, it fetches current documentation directly from source repositories.

**Design workflow:**
1. Use `plan` agent: "Analyze the value proposition of adding Context7 MCP server to our workshop environment. What coding scenarios would benefit most from up-to-date documentation access?"
2. Use `build` agent: "Generate the JSON config block for adding Context7 MCP server to our opencode.json. Use the npm package '@upstash/context7-mcp' and include usage hints."
3. Add the configuration to `opencode.json` under the `mcp` section.
4. **Restart opencode** to register the new server.
5. Test the integration with specific prompts.

**Configuration to add:**
```jsonc
"context7": {
  "type": "local",
  "command": ["npx", "-y", "@upstash/context7-mcp"]
}
```

**Test prompts (try these after setup):**
1. **Explicit test:** "Use Context7 to get current Next.js documentation for creating middleware. Show me how to implement JWT authentication in middleware."
2. **Implicit test:** "Create a React component using the latest Chakra UI patterns for a user profile card." (Observe if Context7 is automatically invoked)
3. **Library-specific test:** "Get documentation for Supabase authentication setup and show me how to implement social login with Google."

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

**Common issues:**
- If installation fails, try `bunx` instead of `npx` in the command array
- Ensure Node.js >= v18.0.0 for compatibility
- Check that the server appears in logs during opencode startup
- API rate limits apply; consider getting a free API key from context7.com for higher limits

---

## 8. File Referencing & Tool Usage [Core]

### File Referencing with @
When working with agents, reference specific files using the `@` syntax for precise context:

**Examples:**
- `@opencode.json` - Reference the main configuration file
- `@README.md` - Include the entire README content
- `@src/components/Header.js` - Reference a specific source file

**Best practices:**
- Use `@filename` when asking agents to analyze, modify, or understand specific files
- Combine with delegation: "Have the build agent update @package.json with the new dependency"
- Reference multiple files: "Review @opencode.json and @AGENTS.md to ensure consistency"

### Tool Trigger Strategies
- **Implicit (semantic) trigger**: Agent decides relevance from higher-level instruction (e.g., "Break this big task into smaller logical steps" may cause use of `sequential-thinking`).
- **Explicit trigger**: Name the tool or describe unique action (e.g., "Use playwright to navigate..." or "Invoke GitHub tool to list issues.")
- **Best practice**: For deterministic workshop outcomes, be explicit first; later experiment with implicit prompting.

### Current MCP Servers [Reference]

| MCP Server | Type | Purpose | Typical Trigger |
|------------|------|---------|-----------------|
| `sequential-thinking` | Local reasoning enhancer | Structured multi-step thought scaffolding | Auto: multi-part objectives; Explicit: "Use sequential thinking" |
| `playwright` | Local browser automation | Scripted browsing, DOM inspection, screenshots | Usually explicit: "Use playwright tool to..." |
| `github` | Remote GitHub context | Repository info, PR diffs, issues, code search | Auto: repo/PR references; Explicit: "Use GitHub tool" |

---

## 9. Essential Prompt & Design Foundations [Reference]

### Meta‑prompting: Use an Agent to Draft Prompts

Use a helper agent to design, refine, and sanity‑check prompts before sending them to the executor agent. Benefits: faster iteration, fewer failed runs, clearer intent.

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

## 10. Troubleshooting [Reference]

**Triage Order:**
1. Restart opencode
2. Validate JSON (quick online validator)
3. Run MCP command manually (`npx -y <package>`)
4. Check Node version
5. Reinstall dependency
6. Add explicit invocation phrasing

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| New agent ignored | Not referenced / no delegation explicit | Prompt main agent to use it; update description to clarify niche |
| MCP server missing | Command fails | Run manually to test; ensure package installed |
| JSON parse error | Trailing comma / invalid syntax | Use a JSON linter |
| Tool returns empty | Needs auth / env var | Add variable to `.env` and restart |
| Tool never auto-triggers | Description too vague | Add explicit mention in prompt; enrich description with verbs |
| Sub-agent overreaches (does coding) | Missing boundary clause | Refine description: state "Avoid code; delegate to build" |

---

## 11. Parallel Delegation [Advanced]

You can delegate to multiple (sub)agents concurrently: "Run agents A,B,C in parallel and collect outputs."

**Caveats:**
- **Cost & latency:** parallel model calls multiply token usage and wall time
- **Race conditions:** concurrent edits to the same files can conflict. Prefer unique targets
- **Conflicting outputs:** implement a deterministic merge/validation step (e.g., a reconciler agent)

---

## 12. Extensions & Next Steps [Optional]

**After core assignments:**
- Add a "decision-record" agent producing lightweight ADRs
- Introduce planning refinement loop: `plan` -> `research` -> `plan`
- Add a code quality MCP server (linting / static analysis)
- Benchmark: Same prompt via plain single LLM vs. orchestrated agent system

**Stretch ideas:**
- Look at more configuration options in the [docs](https://opencode.ai/docs/agents/#configure)
- Add a `qa` agent and have `build` ask it to review generated code
- Compare responses before/after Context7 integration for the same coding question

**MCP servers (curated list):**
Community‑curated list of ready‑to‑use MCP servers: https://github.com/punkpeye/awesome-mcp-servers

---

## 13. Validation & Quality [Reference]

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

## 14. Prompt Patterns [Reference]

**Good delegation prompts:**
- "Break this task into sub-tasks and assign each to the best sub-agent. Execute sequentially. Show a final integrated summary."
- "Before coding, have the plan agent validate scope; only then let build generate code."

**Tool-conscious prompt templates:**
- **Sequential Thinking (explicit):** "Use a structured multi-step reasoning process (sequential thinking tool) to decompose this requirement before proposing a plan."
- **Playwright (explicit):** "Use the playwright tool to open https://example.com, extract the main heading text, and return a screenshot."
- **GitHub (explicit):** "Use the GitHub tool to list open issues labeled 'bug' and summarize recurring themes."

**Escalation pattern:**
1. High-level objective
2. Ask for decomposition (may trigger sequential-thinking)
3. Delegate research/passive data gathering (research sub-agent + MCP context server)
4. Synthesize via plan
5. Implement via build
6. (Optional) Validate via QA / simple-tasks

**Evaluation prompt (baseline comparison):**
> Provide a market validation + architecture + first feature spec for a SaaS idea that automates <X>. (Ask once with agent system, once with plain LLM.)

---

## 15. Advanced Design & Maintenance [Advanced]

### Updating AGENTS.md

**What to add:**
- New agent specs with rationales (why it exists, intended inputs/outputs)
- Clear boundaries: what the agent must NOT do
- Decision records: short notes for choices (model, tools, triggers)
- Validation prompts: tests to confirm expected behaviour

**How it helps:**
- Increases consistency across agents and reduces accidental overlap
- Improves accuracy of automatic triggers and auto‑routing
- Speeds onboarding and reduces review friction

### Model Selection Rationale (Expanded)

Reference: [Copilot model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison)

**Detailed tradeoffs:**
| Goal | Preferred model class | Why | When to upgrade | When to downgrade |
|---|---|---|---|---|
| Complex planning, long coherence | higher‑capability models | better reasoning, costlier | Subagent fails validation consistently | Cost/latency dominates |
| Short, high‑throughput tasks | smaller/fast models | cheaper, lower latency | Need better multi‑step reasoning | Outputs remain acceptable |
| Safety‑critical / deterministic | conservative models + lower temp | fewer hallucinations | Error rate too high | Speed requirements increase |

### Writing Agent Descriptions — Examples

**Complete checklist:**
- Purpose: one sentence goal
- Inputs: explicit required inputs
- Outputs: exact artefacts produced
- Boundaries: what it must NOT do
- Triggers: how it is invoked (manual/auto/event)
- Model/tool hints: preferred model class and tools
- Validation: 1–3 prompts to check correct behaviour

**Good vs bad examples:**
- **Good (concise, precise):** "build:primary — Prepare and run CI for pull requests. Inputs: repo, branch. Outputs: build log, artifacts. Tools: shell, read, write. NOT: deploy."
- **Good (focus on boundary):** "doc:updater — Auto‑update API docs from comments. Inputs: repo, commit range. Outputs: PR with docs. NOT: change API code or minor refactors."
- **Bad (vague):** "helper — helps with repo tasks, use as needed." (too broad, no inputs/outputs)

---

## 16. Resources [Reference]

### Installation & Setup
- **Installation:** https://opencode.ai/docs/#install
- **GitHub Copilot setup:** https://opencode.ai/docs/providers/#github-copilot
- **Agent usage:** https://opencode.ai/docs/agents/#usage

### MCP Servers & Examples
- **Curated list:** https://github.com/punkpeye/awesome-mcp-servers
- **Agent examples:** https://github.com/wshobson/agents
- **Model comparison:** https://docs.github.com/en/copilot/reference/ai-models/model-comparison

---

## 17. Quick Action Summary [Reference]

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

## 18. Appendix: Fast Design Prompt Templates [Reference]

**Sub-agent spec (to `plan`):**
> You are designing a new sub-agent for <focus>. Provide: name, 1-line mission, responsibilities list, explicit non-responsibilities, ideal model choice (from existing), collaboration notes with existing agents, and 2 example delegation prompts.

**JSON generation (to `build`):**
> Convert the approved spec into a valid JSON object to insert under the agent section. Use mode=subagent. Keep description under 500 chars, embed boundaries.

**MCP server addition (to `plan`):**
> Propose integration details for a new MCP server giving <data type>. Include purpose, risk mitigation, and explicit invocation verbs.

---

## 19. License / Usage [Reference]

Internal workshop educational material. Adapt as needed for experimentation.

---

Happy building—compose your digital team and let it work end‑to‑end!
