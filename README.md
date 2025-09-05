# AI Agents Workshop – From Idea to Working Prototype

> From single prompts to a coordinated AI delivery loop.

**This is not a "vibe coding" workshop.** You'll learn production-ready patterns for building quality software with AI assistance in your real projects. Focus is on systematic workflows, proper delegation, and maintainable outcomes.

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

### Assignment 3: Research Task Management Solutions

Goal: Use your research sub-agent to analyze existing task management solutions and identify opportunities for a dependency-aware approach.

**Workshop Challenge:** You'll be building a **Dependency-Aware Task Manager** - a todo app where tasks can depend on other tasks with these tricky requirements:
- Circular dependencies must be detected and blocked
- Tasks can't be marked as "done" until dependencies are complete
- User can see dependency trees, perhaps visualize them as graphs

**Step 1: Use PLAN agent to scope the research**
```
"Plan focused research on task management applications that handle task dependencies or project workflows. Identify 5 competitors and analyze how they handle task relationships, dependencies, or project structures."
```

**Step 2: Use BUILD agent to create research structure**
```
"Create research/ directory with competitive-analysis.md and dependency-pain-points.md. Include templates for competitor comparison focusing on dependency handling and user pain points around task relationships."
```

**Step 3: Execute targeted research**
- "Use research to analyze 5 task management tools (like Asana, Monday.com, Todoist, Linear, etc.). Fill in @research/competitive-analysis.md focusing on how they handle task dependencies, if at all."
- "Use research to identify user pain points around task dependencies and project management complexity. Complete @research/dependency-pain-points.md with evidence of what frustrates users about current tools."

**Expected outputs:**
- `research/competitive-analysis.md` - 5 competitor analysis focused on dependency features
- `research/dependency-pain-points.md` - User frustrations with current dependency/project tools

**Agent pattern:** PLAN → BUILD → RESEARCH → RESEARCH

### Assignment 4: Define Core Dependency Features

Goal: Define exactly 2 core features for the Dependency-Aware Task Manager based on research findings.

**Step 1: Use PLAN agent to identify core features**
```
"Review @research/ files and propose 2 core features for a Dependency-Aware Task Manager that address the pain points found in research. Focus on: 1) Basic task dependency creation/management and 2) Dependency validation/cycle detection. Keep scope small - features that can be built in 2-3 days each."
```

**Step 2: Use BUILD agent to create dependency-focused specs**
```
"Create specs/mvp-features.md with detailed specifications for the 2 dependency management features. Include user stories, acceptance criteria, and technical approach considering graph algorithms and state management challenges."
```

**Suggested core features:**
- **Feature 1: Create Task Dependencies** - Users can link tasks together in dependency relationships
- **Feature 2: Dependency Validation** - System detects circular dependencies and prevents invalid task completion

**Feature template:**
```markdown
## Feature: [Name]
**User Story:** As a [user], I want [goal] so that [benefit]
**Acceptance Criteria:**
- [ ] Specific testable requirement
- [ ] Another measurable outcome
**Technical Approach:** Brief implementation notes (consider graph algorithms)
**Graph Challenges:** Specific dependency-related complexities to address
```

**Expected output:**
- `specs/mvp-features.md` - 2 dependency management features ready for implementation

**Agent pattern:** PLAN → BUILD

### Assignment 5: Establish Development Guidelines

Goal: Extend the existing @AGENTS.md file with project-specific development standards that will guide all AI agents during implementation.

**Understanding AGENTS.md:**
The `@AGENTS.md` file serves as a "constitution" for AI agents in your project. It provides guidelines, guardrails, and standards that agents automatically reference when working on your code. This ensures consistent quality, prevents common mistakes, and enforces your preferred development practices across all agent interactions.

**Step 1: Use SIMPLE-TASKS agent to review current guidelines**
```
"Review @AGENTS.md and summarize the current guidelines. Identify what development practices, coding standards, or technical approaches are missing for building quality software."
```

**Step 2: Use PLAN agent to design additional guidelines**
```
"Based on the 2 features in @specs/mvp-features.md and modern software development practices, propose additions to @AGENTS.md. Include: coding standards, testing approach (TDD/testing strategy), error handling patterns, security considerations, and technology-specific best practices."
```

**Step 3: Use BUILD agent to extend AGENTS.md**
```
"Add the proposed guidelines to @AGENTS.md. Create new sections for: Development Standards, Testing Strategy, Code Quality Rules, and Technology Guidelines. Ensure agents understand when and how to apply these standards."
```

**Example additions to consider:**
- **Testing Strategy:** "All features require unit tests. Use TDD approach - write failing test first, implement feature, refactor."
- **Code Quality:** "Follow [language] conventions. Use meaningful variable names. Add JSDoc/docstrings for public functions."
- **Error Handling:** "Implement proper error boundaries. Log errors with context. Provide user-friendly error messages."
- **Security:** "Sanitize all user inputs. Use environment variables for secrets. Validate all API responses."

**Expected outcome:**
- Enhanced `@AGENTS.md` with comprehensive development guidelines
- Clear standards that agents will follow during feature implementation

**Agent pattern:** SIMPLE-TASKS → PLAN → BUILD

### Assignment 6: Build Dependency-Aware Task Manager

Goal: Implement the 2 dependency management features using systematic agent delegation and established guidelines.

**Step 1: Use BUILD agent to set up task manager project**
```
"Review @specs/mvp-features.md and @AGENTS.md. Set up a [React/Node.js/Python] project for a Dependency-Aware Task Manager following development standards. Include testing framework setup and consider graph algorithm libraries (like vis.js for React visualization or networkx for Python)."
```
*Consider: CLI version (Node.js/Python) or web version (React) based on your preferred tech stack*

**Step 2: Build task dependency creation feature**
```
"Implement the task dependency creation feature from @specs/mvp-features.md. Focus on data structures to represent task graphs and basic CRUD operations for tasks and their relationships. Follow @AGENTS.md standards and implement with graph algorithms in mind."
```

**Step 3: Use SIMPLE-TASKS agent to validate dependency creation**
```
"Validate the task dependency creation feature. Test that users can create tasks, link them as dependencies, and that the underlying graph data structure correctly represents relationships. Check against acceptance criteria and coding standards."
```

**Step 4: Build dependency validation and cycle detection**
```
"Implement the dependency validation feature from @specs/mvp-features.md. Include cycle detection algorithms (like DFS-based cycle detection) and business logic to prevent task completion when dependencies aren't met. Ensure integration with the existing task system."
```

**Step 5: Use SIMPLE-TASKS agent for comprehensive validation**
```
"Perform full validation of the Dependency-Aware Task Manager. Test cycle detection works correctly, tasks can't be marked done with incomplete dependencies, and the system handles complex dependency graphs properly. Verify all acceptance criteria are met."
```

**Technical challenges to address:**
- Graph data structure design and storage
- Cycle detection algorithm implementation
- Task completion validation logic
- Visualization/display of dependency trees

**Expected outcome:**
- Working Dependency-Aware Task Manager with both core features
- Proper cycle detection preventing circular dependencies
- Task completion validation based on dependency status
- Clean implementation following established development standards

**Agent pattern:** BUILD → SIMPLE-TASKS → BUILD → SIMPLE-TASKS

**Technology flexibility:** The agents can work with any tech stack - specify your preferences in prompts:
- **Frontend:** React, Vue, Angular, Svelte + CSS frameworks (Tailwind, Bootstrap, Material-UI)
- **Backend:** Node.js, Python (FastAPI/Django), Go, Java (Spring Boot)
- **Database:** PostgreSQL, MongoDB, SQLite
- **Deployment:** Docker, Vercel, Railway, traditional servers

**💡 Context7 tip:** If you added Context7 in Assignment 2, explicitly mention it in your prompts: "Use Context7 to get current [React/FastAPI/etc.] documentation and implement using the latest patterns." This ensures agents use up-to-date APIs and best practices instead of potentially outdated training data.

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
