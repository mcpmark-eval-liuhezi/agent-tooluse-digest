# Literature Digest: Tool Use, Function Calling, and Planning in LLM Agents

**Prepared for:** Thursday group meeting — literature overview for the new project on tool use and planning in LLM agents
**Scope:** Recent arXiv papers on tool use, function calling, and planning with LLM-based agents
**Method:** Full texts read (not just abstracts). arXiv IDs, titles, one-paragraph method summaries, and headline results below. At least one paper was cross-checked against the wider web (see §External Reception).

This digest covers six papers selected as the most relevant from an arXiv literature sweep. The corpus was **not** thin — the search surfaced far more relevant papers than could be covered in one session, so the shortfall to note is one of *breadth*, not availability (see §Corpus Notes). The six were chosen to span the three methodological pillars of the field as it stands: (1) **inference-time planning / search**, (2) **training and distillation of tool-using agents**, and (3) **benchmarks and critical re-evaluation of accepted ideas**.

---

## 1. ToolTree: Efficient LLM Agent Tool Planning via Dual-Feedback Monte Carlo Tree Search and Bidirectional Pruning

- **arXiv ID:** [2603.12740](https://arxiv.org/abs/2603.12740)
- **Authors:** Shuo Yang, Soyeon Caren Han, Yihao Ding, Shuhe Wang, Eduard Hovy

**Method.** ToolTree frames tool planning as a Monte Carlo Tree Search (MCTS) problem guided by two complementary, training-free LLM signals: a *pre-execution* prior r_pre(s,a) that forecasts a candidate tool's usefulness from the current context, tool card, and a schema-valid argument draft — used both to gate tree expansion (pre-pruning, keeping only the top-K branches above a threshold) and to bias the UCT exploration bonus — and a *post-execution* score r_post(s,a) that grades the actual, grounded output of an executed tool call, which drives backward propagation of Q-values and marks low-utility edges non-expandable (post-pruning). Together these yield **bidirectional pruning** that keeps the tree compact under a fixed rollout budget, so the agent allocates compute adaptively and can recover from early missteps without any task-specific retraining. Deterministic (a,args)→o caching avoids duplicate calls, and typed error tokens are attached to persistent failures so downstream scoring handles failures explicitly.

**Headline result.** ~10% average gain over the prior state-of-the-art planning paradigm across four benchmarks, spanning both closed-set (GTA: 66.95 F1; m&m) and open-set (ToolBench: 69.04 pass rate; RestBench) tool planning, while keeping the highest efficiency. Robustness checks in the appendix are notable: performance drops <2% when distractor tools inflate the library from 14 to 10,014 tools, and a restoration analysis shows the search tolerates noisy LLM judges — even with a 25.8% (GPT-4o) / 39.4% (GPT-4o-mini) judge error rate, the gap to a perfect oracle judge is only +2.8 / +4.1 points.

---

## 2. ToolOPD: Credit-Aware On-Policy Distillation for Tool-Use Agents

- **arXiv ID:** [2610.25693](https://arxiv.org/abs/2610.25693)
- **Authors:** Chu Zhao, Lei Tang, Yanming Wang, Jianzhe Zhao, Guibing Guo, Zhengzong Chen, Yuanyuan Zhao, Fei Huang

**Method.** The paper opens with empirical findings that undermine standard on-policy distillation (OPD): on ALFWorld diagnostic trajectories, the top 20% of interaction turns carry 74.2% of the credit, and outcome-reversal counterfactuals show that flipping only the final outcome label given to the teacher changes the credit assigned to 55.2% (ALFWorld), 25.0% (WebShop), and 50.0% (SearchQA) of turns — so uniform per-action teacher supervision is badly miscalibrated. ToolOPD then fixes this with an outcome-aware OPD framework: for each student-generated trajectory, the frozen teacher is conditioned on the full trajectory plus its outcome to induce a concise, reusable **hindsight skill** (a single imperative rule). ToolOPD compares the teacher's token probabilities on the student's actions with and without that skill; the resulting log-prob shift Δ is mapped, asymmetrically by outcome (reinforce actions the skill raises for successes; suppress actions the skill lowers for failures), into per-token credit weights H_i,t, which reweight a clipped, importance-sampled KL distillation loss applied only to executable action tokens. This loss is jointly optimized with GRPO, where GRPO supplies exploration and environmental reward while ToolOPD acts as an auxiliary outcome-conditioned regularizer. The student deploys alone at inference with no teacher and no skill generation.

**Headline result.** Substantial gains over RL and OPD baselines on ALFWorld, SearchQA, and WebShop — e.g., Qwen3-0.6B on WebShop jumps from 10.9% (GRPO) to 58.6% success, a ~5x improvement — with consistent gains across model scales and improved long-context robustness and optimization stability.

---

## 3. Small Reasoning Models are Instruction Followers in Function Calling

- **arXiv ID:** [2608.22472](https://arxiv.org/abs/2608.22472)
- **Authors:** Yalda Taheri, Mohammad Hassan Heydari, Erfan Naaman, Afsaneh Fatemi

**Method.** This paper argues, contrarily to prevailing practice, that small reasoning models (SRMs) do *worse* in native function-calling (NFC) mode — where tools are injected via structured schema tokens — and *better* when tool use is framed as ordinary instruction following. The authors introduce **Instruction-Followed Function Calling (IFFC)**: the function-calling logic is fully decoupled from the primary QA LLM and delegated to a dedicated small model that operates inside a normal user–assistant conversational context, with tools described in the system prompt as "behavioral guidelines" and executed through a ReAct-style loop. Because the SRM's context is entirely separate from the QA model's, it avoids context pollution; and by treating tool definitions as natural-language instructions rather than rigid schemas, the model spends its reasoning capacity on argument generation and tool selection instead of fighting syntactic constraints.

**Headline result.** IFFC consistently beats NFC and prompt-based function calling (PFC) across the Qwen-3 series (0.6B–8B), Phi-4 Mini, and Gemma-3 family on BFCL V3 (Live and Non-Live). A 4B open-weights model under IFFC matches or exceeds frontier proprietary models in NFC mode — e.g., Qwen-3 4B (Think) IFFC achieves 96.0% (Simple) / 97.5% (Multiple) on Non-Live vs. GPT-5.2's 72.9% / 88.0% and Claude 4.5 Sonnet's 72.6% / 95.5%. Qwen-3 0.6B's Live accuracy roughly doubles with "Think" mode enabled (22.9% → 57.7%), and the reasoning trace confers striking quantization robustness: Qwen-3 4B (Think) holds 93.5% accuracy at Q4KM vs. 94.1% at FP16, whereas standard models collapse (Gemma-3 1B falls 23.9% → 8.7%) — supporting viable on-device agentic deployment.

---

## 4. Rethinking the Role of Tool Graph Memory in Multi-Turn Tool Use

- **arXiv ID:** [2610.21571](https://arxiv.org/abs/2610.21571)
- **Authors:** Uiji Hwang, Richeng Xuan, Taeuk Kim

**Method.** A rigorous negative-result / re-evaluation study of **tool graph memory** (as in ToolNet and H-EPM), which stores past experience as weighted tool transitions and uses them to suggest the next tool. The authors evaluate it on 9 open-weight models (2B–120B; Qwen, Gemma, Mistral, gpt-oss families) across ToolSandbox and the multi-turn subset of BFCL-v4, with three repeated runs and paired significance tests. They then dissect any gains via controlled ablations: quantifying extra LLM-call/token cost; testing whether the memory's specific suggestions matter by substituting **random tools** or **no suggestions**; and building an **oracle** condition where the correct next tool is always supplied. From the same episodic experience (H-EPM's summaries, including failures), they construct a lightweight **insight memory** — cluster summaries with K-means, distill "what to do / what to avoid" per cluster, inject the relevant insights into the prompt — as a direct comparator.

**Headline result.** Tool graph memory rarely yields significant gains over the memory-free base agent, and where H-EPM does gain, the gains cost over 2× the LLM calls. The memory's suggestions are often wrong, and crucially the gain mostly does **not** come from the memory itself: random or no suggestions retain most of the improvement, and a simple instruction to "keep calling tools" achieves comparable gains. Even the oracle next-tool suggestion leaves most failures standing (tool-selection errors drop to 4% of failures, but necessity, dependency, and argument failures persist). Insight memory instead beats tool graph memory in most settings with almost no extra LLM calls, and combining the two is worse than insight alone on 7 of 9 models (at 2.68× the calls). The takeaway: memory works better as a *soft prior about the situation* than as a *prescription of the next tool*.

---

## 5. PlanBench-XL: Evaluating Long-Horizon Planning of LLM Tool-Use Agents in Large-Scale Tool Ecosystems

- **arXiv ID:** [2606.22388](https://arxiv.org/abs/2606.22388)
- **Authors:** Jiayu Liu, Qihan Lin, Cheng Qian, Rui Wang, Emre Can Acikgoz, Xiaocheng Yang, Jiateng Liu, Zhenhailong Wang, Xiusi Chen, Heng Ji, Dilek Hakkani-Tür

**Method.** A benchmark paper addressing a realistic blind spot: agents operating in *large* tool ecosystems cannot see all tools at once, and must iteratively **retrieve** tools during problem solving. PlanBench-XL is an interactive retail-domain benchmark of **327 tasks over 1,665 tools**, where each instance is a multi-step workflow (~25 turns on average). Construction is systematic: a generator LLM proposes typed domain datatypes and candidate tools over input/output datatype pairs, a filter LLM removes unreasonable/redundant ones, paired **noisy tools** are added that disclose their unreliability in descriptions, and a backend database instantiates non-trivial values so answers genuinely require tool-derived evidence. Tasks are specified internally as datatype transforms with code-computed ground-truth tool-call sequences, and only instances needing ≥5 distinct tool calls are kept. Agents take three actions (retrieve, tool-call, answer) in a loop, with an environment-maintained state of discovered tools and obtained datatypes. A key design is **bi-directional anticipation** — searching forward from accumulated evidence or backward from the desired goal — plus an optional **retrieval-time blocking** mechanism that replaces path-critical tools with explicit-failure, implicit-failure, or semantically-misleading alternatives, forcing agents to detect disrupted paths and re-plan at runtime.

**Headline result.** Across ten leading LLMs, massive-tool planning remains hard: GPT-5.4 reaches 51.90% accuracy in block-free settings but **collapses to 11.36% under the most severe blocking condition**. Failure analysis is rich: implicit (silent) failures are the most damaging because they return superficially plausible values that agents treat as grounded evidence (42.2% of implicit-failure follow-ups lead to value reuse vs. 0% for explicit errors, up to 75.5% for Llama-3.3-70B); models show distinct "termination policies" — GPT surrenders (77.3% of default failures, rising to 80.6% under blocking), DeepSeek and Llama commit hallucinated values (58.8%/81.7% of default failures), and Gemini keeps searching — and models can reject semantically misleading tools (≤3% of invoked alternatives) but struggle with executable-looking silent failures.

---

## 6. TURTLE: Evolving Self-tuning Benchmarks for Long-Horizon Tool Use

- **arXiv ID:** [2610.27307](https://arxiv.org/abs/2610.27307)
- **Authors:** Hyunji Alex Nam, Michael Dennis, Natasha Jaques, Minsuk Chang

**Method.** Rather than a new planning method, TURTLE attacks the scaling problem of tool-use evaluation itself: existing benchmarks are domain-specific, human-supervised, or built from scratch with no compatibility with prior harnesses. TURTLE (Tool Use Related Tasks with Long-Horizon Evolution) applies **Unsupervised Environment Design (UED)** via evolutionary search — implemented on top of AlphaEvolve, so even closed-source frontier models can act as the proposer without weight access. Three roles: a **proposer** that mutates a parent task (given its score, summaries, and five sampled archive tasks) to maximize a multi-objective fitness; a **solver** whose empirical performance defines difficulty (each task attempted N times, with a target ~50% completion rate, following "sampling for learnability"); and a **feedback generator** that summarizes solver failure traces in natural language for the proposer to exploit. Crucially, the authors deliberately avoid zero-sum adversarial proposer/solver dynamics, which they show leads to reward hacking (ambiguous or partially-solvable tasks); instead they verify feasibility by requiring the ground-truth solution to execute cleanly in simulation and the task to be perfectly solved in at least one attempt. Fitness combines target-difficulty (bell curve peaked at c=0.5), learnability c(1-c), and trajectory-length bonuses conditioned on relaxed/strict difficulty bands, with trajectory length measured from the solver's successful runs (not the ground truth) to prevent trivial length inflation. Results are reported with Gemma 4 31B-it as solver, plus ablations with Gemini 3.6 Flash and Gemini 3.1 Flash Lite, on AppWorld, τ-telecom, τ-banking, and AutomationBench.

**Headline result.** On AppWorld, evolved tasks raise the proportion of hard tasks by **165%**, increase desirable task features (logical filtering and reasoning) by 29–212%, and broaden diagnostic coverage of (task feature × error type) pairs from 49 to 54 — producing self-tuning test suites that adapt to the solver and elicit a broader range of failure modes than the original tasks, with no human supervision.

---

## External Reception

**ToolTree ([2603.12740](https://arxiv.org/abs/2603.12740))** was cross-checked against the wider web. The official GitHub repository ([SYang2000/ICLR_2026_ToolTree](https://github.com/SYang2000/ICLR_2026_ToolTree), MIT-licensed, Python/Shell) confirms the work is **accepted at ICLR 2026** and hosts the full released implementation — the MCTS main loop, dual (pre/post) evaluation, bidirectional pruning, real tool-execution mode, and a 32-test behavior suite — with news entries noting the complete official code release (2026-07-05). The repository shows modest but real community traction (8 stars, 8 watchers, 1 open issue, 16 contributions from the lead author), and the authors' citation block includes an OpenReview forum ID (`Ef5O9gNNLE`) for the ICLR 2026 submission. Reception is consistent with the paper's claims: external coverage describes the same architecture (pre-evaluation filtering schema/slot-incompatible calls, post-evaluation pruning via real feedback, answer predictor over the best MCTS trajectory) and the same headline (~10% average gain over prior SOTA across 4 benchmarks). The independent confirmation of peer-reviewed acceptance at a top venue, plus working public code, is the strongest external validation any of these papers received; no contradicting criticism or replication dispute surfaced in the search.

The remaining five papers were not web cross-checked. ToolOPD lists a code repository in the paper (github.com/MagicAgent-Search/ToolOPD) which is available on the arXiv HTML page, but no independent third-party discussion was found, so its external reception remains unverified.

---

## Corpus Notes

- **Corpus size:** The arXiv searches returned far more relevant papers than the six covered here, so the "at least three" requirement is comfortably exceeded; there is no shortfall in availability. Notable highly-relevant papers that were surfaced but not covered for brevity include: OpenVLHarness (2610.26860, visual agentic tool-use with multimodal memory), NIOCIS (2610.25985, routing layer for tool-use agents), "Agents' Last Doubt" (2610.24194, when to act vs. step back), TPS-Bench (2511.01527, tool planning & scheduling), and PlanBench-XL's own related-work table cites many more.
- **Search coverage:** Searches were run for tool use + LLM agents, function calling, and planning with LLM agents, across cs.AI/cs.CL/cs.LG, sorted by both date and relevance. Two of the initial search queries were malformed and returned an essentially random recent-paper feed (irrelevant robotics/CV/materials papers); they were discarded and replaced with corrected queries, which is why the selection below reflects only well-formed search results.
- **Selection criteria:** Six papers spanning (a) planning/search methods, (b) training/distillation, (c) benchmarks for planning and for evolution, and (d) critical re-evaluation of an accepted idea. Selection favors recency (mostly 2026 submissions) and diversity of method type over exhaustive coverage of any one pillar.

## Discussion Hooks for Thursday

1. **Planning vs. training.** ToolTree (search at inference, no training) vs. ToolOPD (training with outcome-aware distillation). Are these complementary — could ToolOPD-trained agents be run under ToolTree-style search?
2. **The interface debate.** IFFC's claim that instruction-following beats native function-calling for small reasoning models challenges how everyone builds tool use today. Worth testing in our own stack.
3. **What memory should store.** The tool-graph-memory re-evaluation suggests situation-level insights beat next-tool prescriptions — a caution for anyone building agent memory systems.
4. **Benchmarks are the bottleneck.** PlanBench-XL's 51.90% → 11.36% collapse under tool blocking, and TURTLE's 165% increase in hard tasks, both say current tool-use evaluation severely underestimates difficulty — and both supply diagnostics to fix it.
