# 🏢 Company: [Data Accenture]

| Field | Details |
|---|---|
| **Role** | Senior AI Engineer |
| **Round** | AI / Technical Pre-Screening |
| **Source** | HR-shared question sheet (topics + red flags) |
| **Interview Date** | TBD |
| **Status** | 🟡 Preparing |
| **Result** | — |

**Their stated priority topics:** Python + FastAPI · AI Agents & Agentic Workflow · Prompt Engineering & Function Calling · LangGraph / CrewAI · MCP & A2A · React.js · Vector Databases · Docker & Kubernetes

**Their explicit reject signals (from the HR sheet):**
- ❌ Saying "agents are always better" → see Q17
- ❌ Saying fine-tuning is how you "add knowledge" → see Q31

---

## ✅ Prep Checklist

### Topic coverage
- [ ] RAG & Vector Databases — Q1–Q10
- [ ] Hallucinations, Evals & Guardrails — Q11–Q15
- [ ] AI Agents & Agentic Workflows — Q16–Q22
- [ ] LangGraph / CrewAI — Q23–Q26
- [ ] Prompt Engineering & Function Calling — Q27–Q31
- [ ] MCP & A2A — Q32–Q34
- [ ] Back End: Python + FastAPI — Q35–Q40
- [ ] Front End: React & Real-Time — Q41–Q44
- [ ] Docker, Kubernetes & Cost/Latency — Q45–Q49
- [ ] System Design & Production Experience — Q50–Q51

### Must-do before the call
- [ ] Memorize Q17 framing (agentic vs. single call — their #1 reject filter)
- [ ] Memorize Q31 framing (fine-tuning ≠ knowledge — their #2 reject filter)
- [ ] One sentence per resume metric on how it was measured (~40% discovery time, ~60% triage effort, ~35% incident resolution)
- [ ] Run one CrewAI example hands-on (on their list, not on resume)
- [ ] Review K8s primitives — probes, HPA, requests/limits (Q46 hedge is the ceiling; don't improvise past it)
- [ ] Whiteboard the THG shopping assistant architecture from memory (Q50)
- [ ] Whiteboard the knowledge-assistant RAG pipeline from memory (Q1 + Q51)
- [ ] Dry-run answers out loud — skeletons, not scripts

### Practice passes
- [ ] Pass 1 — read all answers + mental models
- [ ] Pass 2 — answer aloud from the mental model only, without reading the answer
- [ ] Pass 3 — mock with only the question list, self-score weak ones

---

## 🧠 Master Mental Model — how to answer ANY question in this round

Every strong answer here follows the same 4-step shape. If you blank on a question, fall back to this:

**1. DIRECT** — answer the actual question in the first sentence. No warm-up.
**2. TRADE-OFF** — name the cost, limit, or failure mode of your answer. This is the senior signal.
**3. ANCHOR** — tie it to a real project (THG assistant, knowledge assistant, Codo, Chitra.ai, Razorpay).
**4. MEASURE** — say how you'd verify it (eval set, metric, trace) or what you'd watch in production.

Per-question "Mental model" lines below tell you which shape that specific question wants — read the mental model, then check the strong answer against it. In the interview, recall the model, not the paragraph.

---

## 📋 Questions & Answers

**-------------------------- RAG & Vector Databases ----------------------------------------**

**Q1. Explain the RAG pipeline and its steps:**

ingestion → chunking → embeddings → indexing → retrieval → re-ranking → synthesis

Mental model: Pipeline walk — name all 7 stages in order first (shows the map), then zoom into 1–2 stages with a trade-off each, then anchor: "I built this at THG."

Strong answer: Ingestion pulls and normalizes source docs; chunking splits into retrieval-sized units; embedding model converts chunks to vectors; indexing stores them in a vector DB (Pinecone) with metadata. At query time: embed the query → retrieve top-k → re-rank with a cross-encoder → LLM synthesizes a grounded answer with citations. Anchor it in the THG knowledge assistant — you built exactly this.

---

**Q2. How do you decide chunk size and chunking strategy?**

Mental model: Trade-off axis — name the axis (too small ↔ too large), give your default with numbers, then kill the question with "I tune it by measurement, not guessing."

Strong answer: It's a recall/precision trade-off — too small loses context, too large dilutes the embedding and wastes tokens. Default to structure-aware chunking (headings/paragraphs), ~300–800 tokens, 10–15% overlap, metadata attached. Senior signal: "I tune it by measuring retrieval hit-rate on an eval set, not by guessing."

Red flag if you say: one fixed chunk size works for everything.

---

**Q3. What are embeddings and how did you choose an embedding model?**

Mental model: Define in one line → selection = 4 criteria (quality, dimensions, input length, price) → anchor with "I benchmarked on our own data."

Strong answer: Embeddings map text to dense vectors where semantic similarity = geometric closeness (cosine/dot product). Selection is a trade-off: retrieval quality on your domain (MTEB as a starting point, then verify on your own labeled query→doc set), dimensionality (storage + latency), max input length, price. Mention benchmarking candidates on THG's actual documentation before committing.

---

**Q4. What happens inside vector indexing — why not brute-force search?**

Mental model: Why-not-naive — state why brute force fails (O(n)), name the two ANN families (HNSW, IVF), state the trade they make (recall vs. speed).

Strong answer: Exact kNN is O(n) per query and doesn't scale. ANN indexes — HNSW (graph-based) or IVF (clustering) — trade a tiny recall loss for sub-linear query time. Pinecone manages this, but knowing the recall/latency knobs exist (e.g., HNSW ef_search) shows depth beyond "the DB handles it."

---

**Q5. What is hybrid search and when do you need it?**

Mental model: Definition → the failure case that motivates it (dense search misses exact identifiers) → concrete trigger ("whenever queries contain SKUs/codes").

Strong answer: Combine dense (semantic) retrieval with sparse/keyword (BM25), merged via reciprocal rank fusion. Dense search fails on exact identifiers — SKUs, error codes, acronyms — which keyword search nails. Reach for hybrid whenever queries contain exact-match terms, e.g., internal docs full of product codes.

---

**Q6. Why re-rank after retrieval — isn't the vector DB already ranking?**

Mental model: Contrast pair — bi-encoder (fast/coarse) vs. cross-encoder (slow/precise) → the retrieve-wide-then-rerank-narrow pattern → admit the latency cost.

Strong answer: Bi-encoders score query and doc independently — fast but coarse. A cross-encoder reads query+chunk together for far more accurate relevance: retrieve top-50 cheap, re-rank to top-5 precise. Honest cost: 100–300ms added latency, so apply only when precision beats speed, and cache.

---

**Q7. How do you evaluate whether retrieval is actually working?**

Mental model: Split the system — "I evaluate retrieval and generation separately" is the whole answer; then one metric set per half, plus the tools (Deepeval, Langfuse).

Strong answer: Evaluate retrieval and generation separately. Retrieval: golden query→chunk pairs, measure recall@k, MRR, precision@k. Generation: faithfulness/groundedness + answer relevance via LLM-as-judge (Deepeval), tracked per-trace in Langfuse. End-to-end-only evaluation hides whether the failure is retrieval or synthesis.

---

**Q8. Why Pinecone? Compare vs. pgvector / FAISS.**

Mental model: Reframe — refuse "which is best," answer on the real axis: ops burden vs. control vs. cost. One sentence per option, then justify YOUR context.

Strong answer: Pinecone = managed scaling, metadata filtering, namespaces, zero ops — right for a small team shipping fast. pgvector = modest data volume + already running Postgres (one less system). FAISS = a library not a service, fastest for local/batch but you own persistence/sharding/serving. Frame it as ops burden vs. control vs. cost, not "which is best."

---

**Q9. How do you handle multi-tenancy / access control in RAG?**

Mental model: One principle answers everything — "filter at retrieval time, never post-hoc" — then explain the two ways post-hoc fails (leak into context, breaks top-k).

Strong answer: Filter at retrieval time, never post-hoc — attach tenant/ACL metadata to every chunk and apply metadata filters (or namespaces) in the query so restricted content never reaches the prompt. Post-retrieval filtering leaks content into context and breaks top-k. THG's platform is multi-tenant, so this was non-negotiable.

---

**Q10. What are the common failure modes of RAG?**

Mental model: Failure taxonomy — list 4 failure modes, pair each with its fix. Close with the meta-point: diagnose per stage, not RAG as a black box.

Strong answer: Retrieval misses (bad chunking, vocabulary mismatch, stale index), context poisoning (irrelevant chunks), model ignoring context and answering from parametric memory, lost-in-the-middle. Each has a distinct fix — hybrid search, re-ranking, groundedness prompting + evals, context ordering. Diagnose per stage, not RAG as one black box.

---

**-------------------------- Hallucinations, Evals & Guardrails ----------------------------------------**

**Q11. How do you reduce hallucinations?**

Mental model: Layered defense — number the layers (ground → cite → constrain → measure → temperature). End with humility: "you engineer it down, you don't eliminate it."

Strong answer: Layered — (1) ground with RAG and instruct answer-only-from-context / say "I don't know"; (2) require citations; (3) schema-constrain structured outputs; (4) measure faithfulness with LLM-as-judge on an eval set so every change shows a before/after score; (5) lower temperature for factual tasks. You engineer hallucination down and detect it — you don't eliminate it.

---

**Q12. How do you actually measure groundedness / faithfulness?**

Mental model: Give the formula — claims decomposed → checked against context → supported/total. Then the tooling, then the senior twist: "the judge itself is calibrated."

Strong answer: Decompose the answer into atomic claims, check each against retrieved context with an LLM-as-judge — faithfulness = supported claims / total claims (Deepeval implements this). Log scores per trace in Langfuse so regressions surface on prompt/model changes. Senior signal: the judge itself is spot-checked against human labels.

---

**Q13. What guardrails would you put on an LLM app?**

Mental model: Three boundaries — input side, output side, system level. One breath each; don't dump an unordered list.

Strong answer: Input: prompt-injection detection, scope filters, PII scrubbing before model or logs. Output: Pydantic schema validation, moderation checks, groundedness thresholds triggering fallbacks, allow-listed tools with validated arguments. System: token/cost limits per request, rate limiting, human-in-the-loop for irreversible actions.

---

**Q14. How do you do observability for LLM applications?**

Mental model: Differentiate from normal APM — "you're debugging quality, not just errors" — then list what a trace must capture, then anchor with the Razorpay ~35% number.

Strong answer: Trace every request end-to-end — prompt version, retrieved chunks, tool calls, tokens, per-step latency, cost, eval scores (Langfuse for LLM traces; Grafana/Sentry for infra). Difference from normal APM: you're debugging quality, not just errors. Cite Razorpay: structured logging + alerting cut incident resolution ~35%; same discipline applied to LLM pipelines.

---

**Q15. How do you safely ship a prompt change?**

Mental model: Analogy carries it — "treat prompts like code": version → regression suite → compare → flag-gated rollout. The analogy IS the answer.

Strong answer: Treat prompts like code — version them, run the candidate against a regression eval suite (golden Q&A + faithfulness/relevance metrics), compare vs. current, roll out behind a flag with online monitoring.

Red flag: editing prompts by vibes with no regression suite.

---

**-------------------------- AI Agents & Agentic Workflows ----------------------------------------**

**Q16. What's the difference between an agent and an LLM call with tools?**

Mental model: One distinguishing feature — "autonomy over control flow." Everything else (loop, state, stopping) hangs off that single phrase.

Strong answer: Tool-augmented call = one decision (model picks tool, you execute, model answers). Agent = a loop: plan → act → observe → decide next step, with state across iterations. The distinguishing feature is autonomy over control flow — the model, not your code, decides what happens next and when to stop.

---

**Q17. When does an agentic approach actually beat a single LLM call with tools? Be honest about the cost.**

Mental model: Skeptic's frame — lead with "only when," itemize the real costs unprompted, then the kill line: "most agent tasks are really one tool-augmented call." Prove it with one project that loops (Codo) and one that deliberately doesn't (THG lookup path).

Strong answer: Only when the task needs iteration or branching on intermediate results — diagnostics where the next check depends on the last result, multi-source research. Honest costs: multiplied latency and tokens, compounding per-step error rates, harder debugging. Most "agent" tasks are really one well-designed tool-augmented call. Cite: Codo loops because analysis branches per finding; the THG assistant's product-lookup path is deliberately a single call.

Reject signal (per their sheet): "agents are always better." Never say this.

---

**Q18. How do AI agents communicate (multi-agent)?**

Mental model: Patterns then protocol — 3 patterns (supervisor, shared state, message passing), then A2A as the standard, then the practical rule about minimal context.

Strong answer: Supervisor/orchestrator routing to specialist agents (LangGraph's typical shape), shared state/blackboard, message-passing with structured payloads; across frameworks, A2A standardizes it with agent cards + task-based messaging. Practical rule: pass structured, minimal context between agents — dumping full transcripts balloons cost and degrades quality.

---

**Q19. How do you implement agent memory?**

Mental model: Two-tier split — short-term (in context) vs. long-term (external store), then the unlock phrase: "memory is RAG over the agent's own history." Anchor with THG session memory.

Strong answer: Short-term = conversation/task state in context, summarized/truncated as it grows. Long-term = facts extracted to a vector or KV store, retrieved semantically — memory is basically RAG over the agent's own history. Cite: session memory in the THG shopping assistant (preferences, prior products) keeping recommendations coherent across turns.

---

**Q20. Explain ReAct vs. plan-and-execute.**

Mental model: Contrast pair + decision rule — one sentence per pattern with its cost, then: "I choose based on how much the path depends on intermediate results."

Strong answer: ReAct interleaves reasoning and action per step — flexible, adapts to observations, but can wander and burns tokens. Plan-and-execute drafts the full plan up front, executes (often with cheaper models), re-plans on failure — predictable and cost-efficient for well-structured tasks. Choose based on how much the path depends on intermediate results.

---

**Q21. How do you make agents reliable in production?**

Mental model: "Bound everything" is the headline — budgets, timeouts, iterations. Then validation, retries, HITL, and close with the quotable: "an agent you can't observe is an agent you can't ship."

Strong answer: Bound everything — max iterations, per-step timeouts, token/cost budgets. Schema-validate tool arguments, make tools idempotent, retry with backoff, human-in-the-loop for irreversible or low-confidence actions, trace every step. An agent you can't observe is an agent you can't ship.

---

**Q22. Describe an AI agent you built in production.**

Mental model: STAR compressed — trigger → loop → tools/state → output, then ONE production lesson (precision over coverage). The lesson is what makes it senior; the description alone is junior.

Strong answer: Codo — GitHub App triggers on PRs, agent fetches the diff, iteratively analyzes files for correctness/security issues via LangChain tool calling, Redis for state/caching, posts structured review comments. Production lessons: bound the loop by PR size, deduplicate findings, tune for precision — a noisy reviewer gets ignored.

---

**-------------------------- LangGraph / CrewAI ----------------------------------------**

**Q23. Why LangGraph over plain LangChain chains/agents?**

Mental model: Name what's wrong with the old thing (linear chains, hidden loops) → name the fix (explicit state machine) → name the production payoff (see and test every path).

Strong answer: Chains are linear and legacy agents hide the control loop. LangGraph makes the workflow an explicit state machine — typed shared state, nodes as steps, conditional edges — giving cycles, parallelism, persistence, and human-in-the-loop as first-class features. Production win = control and debuggability: you can see and test every path.

---

**Q24. Explain LangGraph's core concepts.**

Mental model: Four nouns in order — State, Nodes, Edges, Checkpointers — one sentence each. Land on checkpointers: "the persistence layer is what makes it production-grade."

Strong answer: State — typed schema (TypedDict/Pydantic) shared across the graph, updated via reducers. Nodes — functions reading state and returning updates. Edges — fixed or conditional routing, enabling loops. Checkpointers persist state per thread → resumability, time-travel debugging, interrupts. The persistence layer is what makes it production-grade.

---

**Q25. How do you implement human-in-the-loop in LangGraph?**

Mental model: Mechanism → why it works → when to use. Interrupt → checkpointer persistence (pause can last days) → irreversible actions.

Strong answer: Interrupts — the graph pauses at a designated point (e.g., before a sensitive tool call), the checkpointer persists state, execution resumes with the human's input injected (approve/edit/reject). Persistence per thread means the pause can last seconds or days. Right pattern for irreversible actions.

---

**Q26. CrewAI vs. LangGraph — when would you use each?**

Mental model: Abstraction-level frame — high-level/fast/less control vs. low-level/explicit/production. Take a position: prototypes vs. reliability. Fence-sitting reads junior.

Strong answer: CrewAI is higher-level — role-based agents with task delegation, fast to prototype collaborative workflows, less control over execution. LangGraph is explicit and lower-level — you own the state machine, which matters for determinism, evals, and HITL. Honest position: CrewAI for quick prototypes, LangGraph for production reliability.

---

**-------------------------- Prompt Engineering & Function Calling ----------------------------------------**

**Q27. Which prompting techniques actually matter in production?**

Mental model: List 5 fast, then flip the question — "equally important is what you cut" — then close on discipline: versioned + eval-backed.

Strong answer: Clear role + task framing, few-shot examples for format-sensitive tasks, explicit output schemas, chain-of-thought for reasoning-heavy steps, delimiting untrusted content (user input, retrieved docs) from instructions. Equally important is what you cut — bloated prompts cost money and degrade instruction-following. Every prompt versioned and backed by an eval.

---

**Q28. How does function/tool calling actually work?**

Mental model: Walk the wire — definitions sent → model emits structured call → YOUR code executes → result returned → model continues. Emphasize "the model never runs anything," then where quality lives (descriptions).

Strong answer: You send tool definitions (name, description, JSON Schema params); the model emits a structured tool-call object instead of prose when a tool fits; your code executes it (the model never runs anything), returns the result as a tool message, model continues. Quality lives in descriptions and schemas — vague descriptions are the #1 cause of wrong tool selection.

---

**Q29. How do you enforce structured outputs?**

Mental model: Preference ladder — native structured-output mode first, Pydantic validate-and-retry second, never regex. Bonus: "one schema, reused everywhere."

Strong answer: Prefer native structured-output/JSON-schema modes where the provider constrains decoding; otherwise prompt JSON-only and validate with Pydantic, retrying with the validation error fed back. Define the schema once in Pydantic and reuse it for the API contract and LLM output validation — one source of truth. Never regex-parse free text.

---

**Q30. What is context engineering?**

Mental model: Definition (what enters the window: selected, ordered, budgeted) → why it beats prompt wording (degradation + cost) → the senior confession: "I spend more time here than on phrasing."

Strong answer: Deciding what enters the window each call — instructions, retrieved chunks, tool results, memory summaries — selected, ordered, budgeted. Models degrade on long noisy context (lost-in-the-middle) and tokens are money. Senior signal: "I spend more time on retrieval quality, ordering, and summarization policies than on prompt phrasing — that's where accuracy and cost actually move."

---

**Q31. Fine-tuning vs. RAG vs. prompt engineering — when would you reach for each?**

Mental model: Escalation ladder — prompting first (cheapest), RAG for knowledge, fine-tuning last and only for behavior. Then say the red-flag antidote UNPROMPTED: "fine-tuning shapes behavior; facts belong in retrieval."

Strong answer: Prompting for behavior/format — zero data cost, instant iteration. RAG for injecting fresh/proprietary knowledge without retraining, with attribution — knowledge changes daily, weights shouldn't. Fine-tuning only for consistent style/format or narrow domain behavior when you have labeled data and prompting has plateaued.

Red flag (explicit in their sheet): saying fine-tuning is how you "add knowledge." State the opposite unprompted: fine-tuning shapes behavior; facts belong in retrieval.

---

**-------------------------- MCP & A2A ----------------------------------------**

**Q32. Explain MCP — what problem does it solve?**

Mental model: Problem before protocol — N×M integration mess → one standard → architecture in one line (host → client → server) → hands-on proof (FastMCP).

Strong answer: Standardizes how LLM apps connect to external tools/data — instead of N×M custom integrations, servers expose tools/resources/prompts over a common JSON-RPC protocol and any client can use any server. Architecture: host → client (one per server) → server, over stdio or HTTP transport. Cite hands-on: you've built servers and clients with FastMCP.

---

**Q33. MCP vs. plain function calling — what's actually different?**

Mental model: Layer separation — capability (function calling) vs. integration layer (MCP). One concrete contrast: hand-wire every tool vs. write a server once. Close: "MCP uses function calling under the hood."

Strong answer: Function calling is the model capability (emit structured tool calls); MCP is the integration layer making tools discoverable and reusable across apps. Plain function calling = hand-wire every tool into your app; MCP = write a server once, any client discovers it at runtime. MCP still uses function calling under the hood.

---

**Q34. What is A2A and how does it differ from MCP?**

Mental model: One rule of thumb does all the work — MCP = agent↔tools (vertical), A2A = agent↔agent (horizontal). Say "complementary, not competing" so it doesn't sound like a versus.

Strong answer: Agent-to-Agent protocol standardizes communication between agents across frameworks/vendors — capability cards, task lifecycle states, streamed results — so an agent on one stack can delegate to another. Rule of thumb: MCP connects an agent to tools/data (vertical); A2A connects agents to peer agents (horizontal). Complementary, not competing.

---

**-------------------------- Back End (Python + FastAPI) ----------------------------------------**

**Q35. How do you design a scalable Python API for ML/LLM inference?**

Mental model: Follow the request — entry (FastAPI/Pydantic) → concurrency (async I/O) → escape hatch for long work (queue) → delivery (SSE) → speed (cache) → scale (stateless). Anchor: "this is Chitra.ai."

Strong answer: FastAPI async endpoints + Pydantic schemas, Uvicorn workers behind a load balancer; async I/O with pooling and per-call timeouts for LLM/DB calls; long tasks to a queue (Celery/SQS) with separate workers; SSE streaming for tokens; Redis caching for responses and embeddings; stateless app tier for horizontal scale; observability from day one. This is essentially Chitra.ai's architecture — say so.

---

**Q36. Sync vs. async in Python — when does async help?**

Mental model: One rule + one bug — helps I/O-bound, does nothing for CPU-bound; then name the classic mistake (blocking call stalls the event loop). The bug is what proves hands-on experience.

Strong answer: Async helps I/O-bound work — LLM API calls, DB queries, HTTP — by multiplexing waits on one event loop, exactly the LLM-app profile. Does nothing for CPU-bound work (multiprocessing / offload). Name the classic bug: a blocking call (requests, sync DB driver) inside an async endpoint stalls the entire event loop.

---

**Q37. What role does Pydantic play in an LLM application?**

Mental model: One schema, three layers — API validation, LLM output validation, tool-arg validation. Punchline: "that's most of what guardrails means in practice."

Strong answer: One schema, three layers — API request/response validation, LLM structured-output validation (parse model JSON into typed models, retry on failure), and tool-argument validation before execution. It turns the least reliable component — model output — into typed, checked data at the boundary; that's most of what "guardrails" means in practice.

---

**Q38. How do you stream LLM tokens from FastAPI?**

Mental model: Happy path + two gotchas — StreamingResponse/SSE yield loop, then disconnect handling (stop paying for unseen tokens) and proxy buffering. The gotchas are the senior half.

Strong answer: StreamingResponse with text/event-stream, async-iterating the provider's token stream and yielding SSE-formatted events plus a done event. Handle client disconnects to cancel the upstream call (stop paying for unseen tokens); disable proxy buffering so tokens flush. Concrete because you stream the THG assistant's responses this way.

---

**Q39. How do you handle long-running jobs (e.g., video analysis) in an API?**

Mental model: Open with the principle — "never in the request cycle" — then the 202+job-ID pattern, then reliability trimmings (idempotency, DLQ, timeouts). Anchor: Chitra.ai's core design decision.

Strong answer: Never in the request cycle — validate, enqueue (Celery/SQS), return 202 + job ID; workers process independently; clients poll a status endpoint or get webhooks/SSE. Idempotency keys, retries with dead-letter queues, per-job timeouts. Cite Chitra.ai: minutes-long video processing made decoupling ingestion from processing the core design decision.

---

**Q40. How do you structure a production FastAPI app?**

Mental model: Layers with a why — routers / services / DI / config / middleware, then the reason that matters: thin endpoints keep logic unit-testable without HTTP.

Strong answer: Routers per domain, service layer for business logic, dependency injection for DB sessions/clients/config, Pydantic settings for env config, middleware for auth/request IDs/structured logging. LLM/agent logic lives in services, not endpoints — thin endpoints keep the logic unit-testable without HTTP.

---

**-------------------------- Front End (React & Real-Time) ----------------------------------------**

**Q41. WebSockets vs. SSE vs. polling in React — and how do you handle a dropped connection mid-stream?**

Mental model: Match tool to traffic shape — one-way stream = SSE, bidirectional = WS, fallback = polling. Then the drop recipe: backoff+jitter → resume from last acked offset → keep partial UI → dedupe by ID. They flagged this must be concrete — give the recipe, not vibes.

Strong answer: SSE for one-way LLM token streaming (plain HTTP, auto-reconnect with Last-Event-ID built in); WebSockets for bidirectional real-time (chat presence, collaborative state); polling as lowest-common-denominator fallback. Drops: reconnect with exponential backoff + jitter, send token/message offsets so the server resumes from the last acknowledged event, keep partial text rendered with a reconnecting state, de-duplicate replayed tokens by ID. Their sheet says this must be concrete since you stream copilot responses — it is.

---

**Q42. How do you optimize React performance?**

Mental model: "Profile first" is the answer's spine — measure, find the cause, fix that. The techniques (memo, virtualize, code-split) are examples, not the point. Blanket memoization = the trap to name.

Strong answer: Profile first with React DevTools, then: useMemo/useCallback/React.memo for the actual re-render cause, virtualize long lists (react-window), code-split with lazy loading, debounce expensive inputs, TanStack Query as caching data layer. Senior signal is the ordering — measure, find the cause, fix that; blanket memoization adds complexity for nothing.

---

**Q43. How do you render a streaming chat response efficiently?**

Mental model: Contain the churn — only the streaming bubble should re-render. Everything (batched flushes, virtualization, memoized completed messages) serves that one goal.

Strong answer: Append tokens to state for the active message but batch updates (flush per animation frame) rather than re-rendering per token; virtualize the message list; memoize completed messages so only the streaming bubble re-renders; persist the transcript in a store/query cache so reconnects and navigation don't lose state.

---

**Q44. You built an embeddable chat widget — key considerations?**

Mental model: Guest-in-someone's-house frame — isolate (Shadow DOM), stay light (bundle, lazy load, CWV), be secure (CORS, short-lived tokens), and never break the host: "a broken assistant must never break checkout."

Strong answer: Isolation and size — Lit with Shadow DOM so host-site CSS and the widget can't break each other; minimal bundle, lazy-loaded after page load to protect the storefront's Core Web Vitals; strict CORS + short-lived tokens for cross-origin API calls; fail silently — a broken assistant must never break checkout.

---

**-------------------------- Docker, Kubernetes & Cost/Latency ----------------------------------------**

**Q45. How do you containerize a FastAPI LLM service properly?**

Mental model: Build → run → ship — build hygiene (multi-stage, pinned, non-root, no secrets), runtime hygiene (probes, worker sizing), release hygiene (CI scan, tag deploys). Anchor: Chitra.ai's pipeline.

Strong answer: Multi-stage build (slim runtime image), pinned dependencies, non-root user, secrets via env/secret manager — never in the image. Health/readiness endpoints, Uvicorn worker count matched to container CPU, small images for fast cold starts and rollbacks. CI builds and scans; deploys are a tag change — this is Chitra.ai's release pipeline.

---

**Q46. How would you deploy and scale this on Kubernetes?**

Mental model: Primitives in deployment order — Deployment+probes → Service+Ingress → HPA (custom metrics beat CPU for I/O-bound) → config/secrets → rollout safety. Then the honest hedge — and STOP there.

Strong answer: Deployment with liveness/readiness probes and resource requests/limits, Service + Ingress, HPA on custom metrics (queue depth / in-flight requests beat CPU for I/O-bound LLM services), ConfigMaps/Secrets, rolling updates for zero downtime, PodDisruptionBudgets. Honest hedge if pushed: hands-on depth is stronger in Docker/CI-CD; you know the primitives and are deepening operational K8s experience — don't bluff past this.

---

**Q47. How do you control cost in an LLM app on AWS?**

Mental model: Biggest levers first, in order — cache → cascade → trim context → cap tokens → batch. Then the second half nobody says: visibility (per-request cost in traces, tags, alarms). Levers without measurement is half an answer.

Strong answer: Biggest levers first — cache responses and embeddings (Redis); model cascade routing simple queries to cheaper models; trim context via better retrieval instead of stuffing; max-token limits; batch offline work. Then visibility: per-request cost in traces (Langfuse), cost allocation tags + CloudWatch alarms, provisioned vs. on-demand matched to traffic.

---

**Q48. How do you optimize latency end-to-end?**

Mental model: Measure per stage first — then attack in impact order: perceived latency via streaming (TTFT), parallelize, smaller models, cache (semantic cache = biggest single win), shrink prompts, warm connections.

Strong answer: Measure per stage first (retrieval, re-rank, TTFT, total). Then: stream tokens so perceived latency ≈ time-to-first-token; parallelize independent calls; smaller/faster models where quality allows; cache every layer — semantic caching for near-duplicate queries is often the single biggest win; cut prompt size (fewer input tokens = faster prefill); connection pooling/keep-alive.

---

**Q49. What's a model cascade / router and when is it worth it?**

Mental model: Define → when worth it (volume) → the prerequisite everyone forgets: an eval suite, or the cascade silently degrades quality. The prerequisite is the senior half.

Strong answer: A classifier or cheap model routes each request to the smallest model that handles it, escalating hard cases — cuts cost significantly while holding quality on the easy majority. Worth it once volume makes routing overhead trivial. Prerequisite: an eval suite, or you can't know what the cascade silently degrades.

---

**-------------------------- System Design & Production Experience ----------------------------------------**

**Q50. Design an AI chatbot end-to-end (LLD).**

Mental model: Draw the request path left to right (client → gateway → orchestration → RAG/tools → stream back), THEN layer cross-cutting concerns underneath (memory, storage, tracing, guardrails, feedback loop). Path first, layers second — mixing them mid-drawing reads chaotic. Anchor: "this mirrors what I built at THG."

Strong answer: Client (widget, SSE streaming) → API gateway (auth, rate limiting) → FastAPI orchestration → intent/routing → direct LLM path or RAG path (embed → Pinecone retrieve with tenant filters → re-rank → grounded generation) → tool-calling layer with schema-validated allow-listed tools → stream back. Cross-cutting: Redis session memory + cache, Postgres transcripts, Langfuse tracing with eval scores, input/output guardrails, feedback loop feeding the eval set. This mirrors the THG shopping assistant — draw it from memory.

---

**Q51. Describe an AI project you took to production and what you'd do differently.**

Mental model: Ownership → outcome → self-critique. The "do differently" half is the real question — name a genuine mistake (tuned by anecdote before building evals) and the lesson as a rule you now follow. A flawless story reads as either junior or dishonest.

Strong answer: THG knowledge assistant — RAG over internal docs with Pinecone; owned embedding-model selection, chunking/retrieval trade-offs, accuracy/latency/cost balance; cut information-discovery time ~40%. What you'd do differently: build the retrieval eval set before tuning, not after — early chunking iteration was anecdotal; once recall@k and faithfulness were measured, decisions got faster and better. Close with: "no pipeline changes without a before/after metric."

---

**Additional points to be ready for:**

- Explain your end-to-end RAG architecture (Q1 + Q50 combined, drawn on a whiteboard).
- How would you build a multi-agent workflow? (Q18 + Q23–Q26.)
- Every resume number (~40%, ~60%, ~35%) → one sentence each on how it was measured. An unexplainable number is worse than none.
- CrewAI and A2A are on their list but not on your resume → run one CrewAI example before the call so "I've evaluated it" is true.
- Kubernetes is priority #8 but absent from your resume → Q46's hedge is the ceiling; don't improvise beyond it.

**Highest Priority Topics (their list — weight your prep accordingly)**

1. Python + FastAPI
2. AI Agents & Agentic Workflow
3. Prompt Engineering & Function Calling
4. LangGraph / CrewAI
5. MCP & A2A
6. React.js
7. Vector Databases
8. Docker & Kubernetes