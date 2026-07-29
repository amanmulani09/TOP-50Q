# 🏢 Company: [Company Name]

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
- [ ] Pass 1 — read all answers
- [ ] Pass 2 — answer aloud without looking
- [ ] Pass 3 — mock with only the question list, self-score weak ones

---

## 📋 Questions & Answers

**-------------------------- RAG & Vector Databases ----------------------------------------**

**Q1. Explain the RAG pipeline and its steps:**

ingestion → chunking → embeddings → indexing → retrieval → re-ranking → synthesis

Strong answer: Ingestion pulls and normalizes source docs; chunking splits into retrieval-sized units; embedding model converts chunks to vectors; indexing stores them in a vector DB (Pinecone) with metadata. At query time: embed the query → retrieve top-k → re-rank with a cross-encoder → LLM synthesizes a grounded answer with citations. Anchor it in the THG knowledge assistant — you built exactly this.

---

**Q2. How do you decide chunk size and chunking strategy?**

Strong answer: It's a recall/precision trade-off — too small loses context, too large dilutes the embedding and wastes tokens. Default to structure-aware chunking (headings/paragraphs), ~300–800 tokens, 10–15% overlap, metadata attached. Senior signal: "I tune it by measuring retrieval hit-rate on an eval set, not by guessing."

Red flag if you say: one fixed chunk size works for everything.

---

**Q3. What are embeddings and how did you choose an embedding model?**

Strong answer: Embeddings map text to dense vectors where semantic similarity = geometric closeness (cosine/dot product). Selection is a trade-off: retrieval quality on your domain (MTEB as a starting point, then verify on your own labeled query→doc set), dimensionality (storage + latency), max input length, price. Mention benchmarking candidates on THG's actual documentation before committing.

---

**Q4. What happens inside vector indexing — why not brute-force search?**

Strong answer: Exact kNN is O(n) per query and doesn't scale. ANN indexes — HNSW (graph-based) or IVF (clustering) — trade a tiny recall loss for sub-linear query time. Pinecone manages this, but knowing the recall/latency knobs exist (e.g., HNSW ef_search) shows depth beyond "the DB handles it."

---

**Q5. What is hybrid search and when do you need it?**

Strong answer: Combine dense (semantic) retrieval with sparse/keyword (BM25), merged via reciprocal rank fusion. Dense search fails on exact identifiers — SKUs, error codes, acronyms — which keyword search nails. Reach for hybrid whenever queries contain exact-match terms, e.g., internal docs full of product codes.

---

**Q6. Why re-rank after retrieval — isn't the vector DB already ranking?**

Strong answer: Bi-encoders score query and doc independently — fast but coarse. A cross-encoder reads query+chunk together for far more accurate relevance: retrieve top-50 cheap, re-rank to top-5 precise. Honest cost: 100–300ms added latency, so apply only when precision beats speed, and cache.

---

**Q7. How do you evaluate whether retrieval is actually working?**

Strong answer: Evaluate retrieval and generation separately. Retrieval: golden query→chunk pairs, measure recall@k, MRR, precision@k. Generation: faithfulness/groundedness + answer relevance via LLM-as-judge (Deepeval), tracked per-trace in Langfuse. End-to-end-only evaluation hides whether the failure is retrieval or synthesis.

---

**Q8. Why Pinecone? Compare vs. pgvector / FAISS.**

Strong answer: Pinecone = managed scaling, metadata filtering, namespaces, zero ops — right for a small team shipping fast. pgvector = modest data volume + already running Postgres (one less system). FAISS = a library not a service, fastest for local/batch but you own persistence/sharding/serving. Frame it as ops burden vs. control vs. cost, not "which is best."

---

**Q9. How do you handle multi-tenancy / access control in RAG?**

Strong answer: Filter at retrieval time, never post-hoc — attach tenant/ACL metadata to every chunk and apply metadata filters (or namespaces) in the query so restricted content never reaches the prompt. Post-retrieval filtering leaks content into context and breaks top-k. THG's platform is multi-tenant, so this was non-negotiable.

---

**Q10. What are the common failure modes of RAG?**

Strong answer: Retrieval misses (bad chunking, vocabulary mismatch, stale index), context poisoning (irrelevant chunks), model ignoring context and answering from parametric memory, lost-in-the-middle. Each has a distinct fix — hybrid search, re-ranking, groundedness prompting + evals, context ordering. Diagnose per stage, not RAG as one black box.

---

**-------------------------- Hallucinations, Evals & Guardrails ----------------------------------------**

**Q11. How do you reduce hallucinations?**

Strong answer: Layered — (1) ground with RAG and instruct answer-only-from-context / say "I don't know"; (2) require citations; (3) schema-constrain structured outputs; (4) measure faithfulness with LLM-as-judge on an eval set so every change shows a before/after score; (5) lower temperature for factual tasks. You engineer hallucination down and detect it — you don't eliminate it.

---

**Q12. How do you actually measure groundedness / faithfulness?**

Strong answer: Decompose the answer into atomic claims, check each against retrieved context with an LLM-as-judge — faithfulness = supported claims / total claims (Deepeval implements this). Log scores per trace in Langfuse so regressions surface on prompt/model changes. Senior signal: the judge itself is spot-checked against human labels.

---

**Q13. What guardrails would you put on an LLM app?**

Strong answer: Input: prompt-injection detection, scope filters, PII scrubbing before model or logs. Output: Pydantic schema validation, moderation checks, groundedness thresholds triggering fallbacks, allow-listed tools with validated arguments. System: token/cost limits per request, rate limiting, human-in-the-loop for irreversible actions.

---

**Q14. How do you do observability for LLM applications?**

Strong answer: Trace every request end-to-end — prompt version, retrieved chunks, tool calls, tokens, per-step latency, cost, eval scores (Langfuse for LLM traces; Grafana/Sentry for infra). Difference from normal APM: you're debugging quality, not just errors. Cite Razorpay: structured logging + alerting cut incident resolution ~35%; same discipline applied to LLM pipelines.

---

**Q15. How do you safely ship a prompt change?**

Strong answer: Treat prompts like code — version them, run the candidate against a regression eval suite (golden Q&A + faithfulness/relevance metrics), compare vs. current, roll out behind a flag with online monitoring.

Red flag: editing prompts by vibes with no regression suite.

---

**-------------------------- AI Agents & Agentic Workflows ----------------------------------------**

**Q16. What's the difference between an agent and an LLM call with tools?**

Strong answer: Tool-augmented call = one decision (model picks tool, you execute, model answers). Agent = a loop: plan → act → observe → decide next step, with state across iterations. The distinguishing feature is autonomy over control flow — the model, not your code, decides what happens next and when to stop.

---

**Q17. When does an agentic approach actually beat a single LLM call with tools? Be honest about the cost.**

Strong answer: Only when the task needs iteration or branching on intermediate results — diagnostics where the next check depends on the last result, multi-source research. Honest costs: multiplied latency and tokens, compounding per-step error rates, harder debugging. Most "agent" tasks are really one well-designed tool-augmented call. Cite: Codo loops because analysis branches per finding; the THG assistant's product-lookup path is deliberately a single call.

Reject signal (per their sheet): "agents are always better." Never say this.

---

**Q18. How do AI agents communicate (multi-agent)?**

Strong answer: Supervisor/orchestrator routing to specialist agents (LangGraph's typical shape), shared state/blackboard, message-passing with structured payloads; across frameworks, A2A standardizes it with agent cards + task-based messaging. Practical rule: pass structured, minimal context between agents — dumping full transcripts balloons cost and degrades quality.

---

**Q19. How do you implement agent memory?**

Strong answer: Short-term = conversation/task state in context, summarized/truncated as it grows. Long-term = facts extracted to a vector or KV store, retrieved semantically — memory is basically RAG over the agent's own history. Cite: session memory in the THG shopping assistant (preferences, prior products) keeping recommendations coherent across turns.

---

**Q20. Explain ReAct vs. plan-and-execute.**

Strong answer: ReAct interleaves reasoning and action per step — flexible, adapts to observations, but can wander and burns tokens. Plan-and-execute drafts the full plan up front, executes (often with cheaper models), re-plans on failure — predictable and cost-efficient for well-structured tasks. Choose based on how much the path depends on intermediate results.

---

**Q21. How do you make agents reliable in production?**

Strong answer: Bound everything — max iterations, per-step timeouts, token/cost budgets. Schema-validate tool arguments, make tools idempotent, retry with backoff, human-in-the-loop for irreversible or low-confidence actions, trace every step. An agent you can't observe is an agent you can't ship.

---

**Q22. Describe an AI agent you built in production.**

Strong answer: Codo — GitHub App triggers on PRs, agent fetches the diff, iteratively analyzes files for correctness/security issues via LangChain tool calling, Redis for state/caching, posts structured review comments. Production lessons: bound the loop by PR size, deduplicate findings, tune for precision — a noisy reviewer gets ignored.

---

**-------------------------- LangGraph / CrewAI ----------------------------------------**

**Q23. Why LangGraph over plain LangChain chains/agents?**

Strong answer: Chains are linear and legacy agents hide the control loop. LangGraph makes the workflow an explicit state machine — typed shared state, nodes as steps, conditional edges — giving cycles, parallelism, persistence, and human-in-the-loop as first-class features. Production win = control and debuggability: you can see and test every path.

---

**Q24. Explain LangGraph's core concepts.**

Strong answer: State — typed schema (TypedDict/Pydantic) shared across the graph, updated via reducers. Nodes — functions reading state and returning updates. Edges — fixed or conditional routing, enabling loops. Checkpointers persist state per thread → resumability, time-travel debugging, interrupts. The persistence layer is what makes it production-grade.

---

**Q25. How do you implement human-in-the-loop in LangGraph?**

Strong answer: Interrupts — the graph pauses at a designated point (e.g., before a sensitive tool call), the checkpointer persists state, execution resumes with the human's input injected (approve/edit/reject). Persistence per thread means the pause can last seconds or days. Right pattern for irreversible actions.

---

**Q26. CrewAI vs. LangGraph — when would you use each?**

Strong answer: CrewAI is higher-level — role-based agents with task delegation, fast to prototype collaborative workflows, less control over execution. LangGraph is explicit and lower-level — you own the state machine, which matters for determinism, evals, and HITL. Honest position: CrewAI for quick prototypes, LangGraph for production reliability.

---

**-------------------------- Prompt Engineering & Function Calling ----------------------------------------**

**Q27. Which prompting techniques actually matter in production?**

Strong answer: Clear role + task framing, few-shot examples for format-sensitive tasks, explicit output schemas, chain-of-thought for reasoning-heavy steps, delimiting untrusted content (user input, retrieved docs) from instructions. Equally important is what you cut — bloated prompts cost money and degrade instruction-following. Every prompt versioned and backed by an eval.

---

**Q28. How does function/tool calling actually work?**

Strong answer: You send tool definitions (name, description, JSON Schema params); the model emits a structured tool-call object instead of prose when a tool fits; your code executes it (the model never runs anything), returns the result as a tool message, model continues. Quality lives in descriptions and schemas — vague descriptions are the #1 cause of wrong tool selection.

---

**Q29. How do you enforce structured outputs?**

Strong answer: Prefer native structured-output/JSON-schema modes where the provider constrains decoding; otherwise prompt JSON-only and validate with Pydantic, retrying with the validation error fed back. Define the schema once in Pydantic and reuse it for the API contract and LLM output validation — one source of truth. Never regex-parse free text.

---

**Q30. What is context engineering?**

Strong answer: Deciding what enters the window each call — instructions, retrieved chunks, tool results, memory summaries — selected, ordered, budgeted. Models degrade on long noisy context (lost-in-the-middle) and tokens are money. Senior signal: "I spend more time on retrieval quality, ordering, and summarization policies than on prompt phrasing — that's where accuracy and cost actually move."

---

**Q31. Fine-tuning vs. RAG vs. prompt engineering — when would you reach for each?**

Strong answer: Prompting for behavior/format — zero data cost, instant iteration. RAG for injecting fresh/proprietary knowledge without retraining, with attribution — knowledge changes daily, weights shouldn't. Fine-tuning only for consistent style/format or narrow domain behavior when you have labeled data and prompting has plateaued.

Red flag (explicit in their sheet): saying fine-tuning is how you "add knowledge." State the opposite unprompted: fine-tuning shapes behavior; facts belong in retrieval.

---

**-------------------------- MCP & A2A ----------------------------------------**

**Q32. Explain MCP — what problem does it solve?**

Strong answer: Standardizes how LLM apps connect to external tools/data — instead of N×M custom integrations, servers expose tools/resources/prompts over a common JSON-RPC protocol and any client can use any server. Architecture: host → client (one per server) → server, over stdio or HTTP transport. Cite hands-on: you've built servers and clients with FastMCP.

---

**Q33. MCP vs. plain function calling — what's actually different?**

Strong answer: Function calling is the model capability (emit structured tool calls); MCP is the integration layer making tools discoverable and reusable across apps. Plain function calling = hand-wire every tool into your app; MCP = write a server once, any client discovers it at runtime. MCP still uses function calling under the hood.

---

**Q34. What is A2A and how does it differ from MCP?**

Strong answer: Agent-to-Agent protocol standardizes communication between agents across frameworks/vendors — capability cards, task lifecycle states, streamed results — so an agent on one stack can delegate to another. Rule of thumb: MCP connects an agent to tools/data (vertical); A2A connects agents to peer agents (horizontal). Complementary, not competing.

---

**-------------------------- Back End (Python + FastAPI) ----------------------------------------**

**Q35. How do you design a scalable Python API for ML/LLM inference?**

Strong answer: FastAPI async endpoints + Pydantic schemas, Uvicorn workers behind a load balancer; async I/O with pooling and per-call timeouts for LLM/DB calls; long tasks to a queue (Celery/SQS) with separate workers; SSE streaming for tokens; Redis caching for responses and embeddings; stateless app tier for horizontal scale; observability from day one. This is essentially Chitra.ai's architecture — say so.

---

**Q36. Sync vs. async in Python — when does async help?**

Strong answer: Async helps I/O-bound work — LLM API calls, DB queries, HTTP — by multiplexing waits on one event loop, exactly the LLM-app profile. Does nothing for CPU-bound work (multiprocessing / offload). Name the classic bug: a blocking call (requests, sync DB driver) inside an async endpoint stalls the entire event loop.

---

**Q37. What role does Pydantic play in an LLM application?**

Strong answer: One schema, three layers — API request/response validation, LLM structured-output validation (parse model JSON into typed models, retry on failure), and tool-argument validation before execution. It turns the least reliable component — model output — into typed, checked data at the boundary; that's most of what "guardrails" means in practice.

---

**Q38. How do you stream LLM tokens from FastAPI?**

Strong answer: StreamingResponse with text/event-stream, async-iterating the provider's token stream and yielding SSE-formatted events plus a done event. Handle client disconnects to cancel the upstream call (stop paying for unseen tokens); disable proxy buffering so tokens flush. Concrete because you stream the THG assistant's responses this way.

---

**Q39. How do you handle long-running jobs (e.g., video analysis) in an API?**

Strong answer: Never in the request cycle — validate, enqueue (Celery/SQS), return 202 + job ID; workers process independently; clients poll a status endpoint or get webhooks/SSE. Idempotency keys, retries with dead-letter queues, per-job timeouts. Cite Chitra.ai: minutes-long video processing made decoupling ingestion from processing the core design decision.

---

**Q40. How do you structure a production FastAPI app?**

Strong answer: Routers per domain, service layer for business logic, dependency injection for DB sessions/clients/config, Pydantic settings for env config, middleware for auth/request IDs/structured logging. LLM/agent logic lives in services, not endpoints — thin endpoints keep the logic unit-testable without HTTP.

---

**-------------------------- Front End (React & Real-Time) ----------------------------------------**

**Q41. WebSockets vs. SSE vs. polling in React — and how do you handle a dropped connection mid-stream?**

Strong answer: SSE for one-way LLM token streaming (plain HTTP, auto-reconnect with Last-Event-ID built in); WebSockets for bidirectional real-time (chat presence, collaborative state); polling as lowest-common-denominator fallback. Drops: reconnect with exponential backoff + jitter, send token/message offsets so the server resumes from the last acknowledged event, keep partial text rendered with a reconnecting state, de-duplicate replayed tokens by ID. Their sheet says this must be concrete since you stream copilot responses — it is.

---

**Q42. How do you optimize React performance?**

Strong answer: Profile first with React DevTools, then: useMemo/useCallback/React.memo for the actual re-render cause, virtualize long lists (react-window), code-split with lazy loading, debounce expensive inputs, TanStack Query as caching data layer. Senior signal is the ordering — measure, find the cause, fix that; blanket memoization adds complexity for nothing.

---

**Q43. How do you render a streaming chat response efficiently?**

Strong answer: Append tokens to state for the active message but batch updates (flush per animation frame) rather than re-rendering per token; virtualize the message list; memoize completed messages so only the streaming bubble re-renders; persist the transcript in a store/query cache so reconnects and navigation don't lose state.

---

**Q44. You built an embeddable chat widget — key considerations?**

Strong answer: Isolation and size — Lit with Shadow DOM so host-site CSS and the widget can't break each other; minimal bundle, lazy-loaded after page load to protect the storefront's Core Web Vitals; strict CORS + short-lived tokens for cross-origin API calls; fail silently — a broken assistant must never break checkout.

---

**-------------------------- Docker, Kubernetes & Cost/Latency ----------------------------------------**

**Q45. How do you containerize a FastAPI LLM service properly?**

Strong answer: Multi-stage build (slim runtime image), pinned dependencies, non-root user, secrets via env/secret manager — never in the image. Health/readiness endpoints, Uvicorn worker count matched to container CPU, small images for fast cold starts and rollbacks. CI builds and scans; deploys are a tag change — this is Chitra.ai's release pipeline.

---

**Q46. How would you deploy and scale this on Kubernetes?**

Strong answer: Deployment with liveness/readiness probes and resource requests/limits, Service + Ingress, HPA on custom metrics (queue depth / in-flight requests beat CPU for I/O-bound LLM services), ConfigMaps/Secrets, rolling updates for zero downtime, PodDisruptionBudgets. Honest hedge if pushed: hands-on depth is stronger in Docker/CI-CD; you know the primitives and are deepening operational K8s experience — don't bluff past this.

---

**Q47. How do you control cost in an LLM app on AWS?**

Strong answer: Biggest levers first — cache responses and embeddings (Redis); model cascade routing simple queries to cheaper models; trim context via better retrieval instead of stuffing; max-token limits; batch offline work. Then visibility: per-request cost in traces (Langfuse), cost allocation tags + CloudWatch alarms, provisioned vs. on-demand matched to traffic.

---

**Q48. How do you optimize latency end-to-end?**

Strong answer: Measure per stage first (retrieval, re-rank, TTFT, total). Then: stream tokens so perceived latency ≈ time-to-first-token; parallelize independent calls; smaller/faster models where quality allows; cache every layer — semantic caching for near-duplicate queries is often the single biggest win; cut prompt size (fewer input tokens = faster prefill); connection pooling/keep-alive.

---

**Q49. What's a model cascade / router and when is it worth it?**

Strong answer: A classifier or cheap model routes each request to the smallest model that handles it, escalating hard cases — cuts cost significantly while holding quality on the easy majority. Worth it once volume makes routing overhead trivial. Prerequisite: an eval suite, or you can't know what the cascade silently degrades.

---

**-------------------------- System Design & Production Experience ----------------------------------------**

**Q50. Design an AI chatbot end-to-end (LLD).**

Strong answer: Client (widget, SSE streaming) → API gateway (auth, rate limiting) → FastAPI orchestration → intent/routing → direct LLM path or RAG path (embed → Pinecone retrieve with tenant filters → re-rank → grounded generation) → tool-calling layer with schema-validated allow-listed tools → stream back. Cross-cutting: Redis session memory + cache, Postgres transcripts, Langfuse tracing with eval scores, input/output guardrails, feedback loop feeding the eval set. This mirrors the THG shopping assistant — draw it from memory.

---

**Q51. Describe an AI project you took to production and what you'd do differently.**

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

