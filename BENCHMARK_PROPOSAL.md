# PagerBench: Can a Model Hold the Pager?

**A benchmark for autonomous production operations — measuring whether an AI agent can keep a live, degrading software system inside its service-level objectives over a sustained on-call shift.**

Status: v0.3 proposal · June 2026

---

## 1. One-paragraph thesis

Every frontier benchmark that labs report today measures **building** things (SWE-bench Pro, FrontierCode, Terminal-Bench) or **producing** things (GDPval, GDP.pdf) — one-shot deliverables in healthy environments. Nothing that frontier labs report measures **operating** things: holding responsibility for a live system, under traffic, while it degrades in unscripted ways, where doing nothing has a cost-per-minute, doing the wrong thing makes it worse, and many alerts should be ignored. PagerBench puts an agent on-call for a simulated multi-day shift over realistic production stacks with injected faults, live synthetic traffic, and concurrent routine work, and scores it on a single, fully automatic, judge-free number: **Error-Budget Retention (EBR)** — the fraction of the system's SLO error budget the agent preserves relative to an oracle remediation, with reliability across seeds (EBR@k) as a first-class headline. The pitch to labs in one line: *"Your model writes production code. Can it carry the pager?"*

## 2. Why this gap, and why now

### 2.1 The reporting landscape (June 2026)

Anthropic's Claude Fable 5 announcement (June 9, 2026) reports: SWE-bench Verified/Pro, Terminal-Bench 2.1, FrontierCode, OSWorld-Verified, HLE, GDPval-AA, MMMU-Pro, CharXiv; system card adds Vending-Bench 2, BrowseComp, OfficeQA Pro. OpenAI's GPT-5.5 and Google's Gemini 3.1 Pro report near-identical sets. Three structural facts about this landscape:

1. **The static-exam era is formally over.** Anthropic dropped GPQA Diamond and AIME as saturated; OpenAI formally deprecated SWE-bench Verified. The benchmarks labs still feature are agentic, economically framed, and score in the 20–50% range (FrontierCode Diamond 29.3%, Agents' Last Exam 24%, GDPval ~48% win-rate).
2. **Every featured benchmark is greenfield/one-shot.** SWE-bench-family: fix an issue in a healthy, well-tested repo, then the episode ends. Terminal-Bench: discrete tasks with a binary check. GDPval: produce a deliverable, get graded once. τ²-bench: a bounded customer conversation. Even Vending-Bench — the closest thing to sustained responsibility — is a turn-based business sim with LLM-simulated counterparties and run-to-run variance its own authors call out.
3. **Long-horizon measurement is hitting a ceiling.** METR's 50%-time-horizon for Claude Mythos is ≥16 hours — at the edge of their task suite (only 5 of 228 tasks ≥16h), and METR's own limitations note says measurement above ~16 hours is unreliable. The regime between "16 hours" and "weeks of sustained responsibility" is the acknowledged open frontier.

### 2.2 The gap

The 2026 gap-map evidence (Epoch benchmarking-hub gaps, METR limitations note, TheAgentCompany's findings, the "Measuring what Matters" construct-validity survey) converges on several under-measured areas. PagerBench deliberately sits at the intersection of four of them:

- **Maintenance and operations, not greenfield.** SWE benchmarks resolve issues in healthy OSS repos. Nothing adopted measures keeping a degrading production system alive: triage, mitigation, root-cause, toil, postmortems. (Academic one-offs exist — §8 — with near-zero lab adoption.)
- **Sustained responsibility beyond the METR ceiling.** A multi-day shift with concurrent, interleaved events measures exactly the coherence-over-time that one-shot tasks cannot.
- **Reliability as the headline, not a footnote.** τ-bench introduced pass^k (90% pass@1 → ~57% pass^8) but labs still headline pass@1. An ops benchmark makes reliability *constitutive*: an SRE who succeeds 80% of the time is not 80% as valuable — they are unemployable.
- **Calibrated inaction.** AbstentionBench showed reasoning training *degrades* abstention. No agentic benchmark scores knowing when **not** to act. On-call work is the natural habitat for this: false alarms, self-healing blips, and "watch it, don't touch it" situations are scored automatically in PagerBench because wrong interventions burn error budget.

### 2.3 Why labs would adopt it (the demand side)

- **It is the enterprise buyer's actual question for 2026–27.** Labs sell coding agents today; the explicit next market is agents that *run* what they build (every major incident-management and observability vendor shipped "AI SRE" products in 2025–26). A lab that scores well has a number its sales team can put in front of a CTO: "preserved 71% of the error budget across a quarter of production incidents."
- **The one-number narrative is legible far beyond ML.** Uptime, SLOs, and error budgets are existing industry language (Google SRE book vocabulary). "Can AI hold the pager?" needs no explanation.
- **Massive headroom by construction.** Frontier agents will plausibly land at 15–35% EBR at launch (long-horizon + concurrency + irreversibility is much harder than one-shot repair; cf. FrontierCode Diamond at 29.3% for mere mergeability).
- **Objective, judge-free scoring.** After the HLE label-noise scandal (~18–30% contested answers), the GDPval grader-agreement problem (66% vs 71% human-human), and the FrontierMath conflict-of-interest scandal, labs and third parties visibly prefer benchmarks with mechanical ground truth. PagerBench's ground truth is *known by construction*: the harness injected the fault, and SLO compliance is measured, not judged.
- **It fits the adoption playbook.** Per the 2025–26 track record (SWE-bench, Terminal-Bench, GDPval-AA), benchmarks spread when they have: automatic contamination-resistant verification ✓, frontier <50% at launch ✓, a neutral steward ✓ (§9), a cheap reproducible harness ✓ (Harbor-compatible, §6), a memorable metric ✓, and expert-sourced realism ✓ (faults curated from real postmortem corpora with practicing SREs).

## 3. What PagerBench is

### 3.1 The episode

An **episode** is an on-call shift: the agent is handed operational responsibility for one **world** (a fully containerized production stack, §3.2) for a compressed simulated duty period (7 simulated days; roughly 2–6 wall-clock hours depending on agent speed). During the shift:

- **Live traffic runs continuously.** A deterministic, scripted (non-LLM) traffic generator exercises user journeys against the stack; its success/latency measurements *are* the score. No LLM simulation anywhere in the loop — avoiding τ²-bench's simulator confound.
- **Scheduled events fire from a hidden fault schedule** (12–20 per episode): infrastructure faults, bad deploys landing through the CI pipeline, dependency brownouts, data-poisoning events, cert expiries, disk-fill slow burns, cascading combinations — plus **false alarms, transient self-healing blips, and red-herring alerts** that the correct response to is calibrated inaction.
- **Routine toil arrives via a ticket queue**: log-rotation, a routine dependency upgrade, a request to provision a new service account, a postmortem due for yesterday's incident. Toil competes for attention with incidents — measuring prioritization under interruption, which no current benchmark touches.
- **The agent has a real operator's toolkit**, nothing more: a terminal in a bastion container (kubectl/ssh/psql etc.), the metrics API (Prometheus-compatible), logs (Loki-style), traces, the alerting feed, the ticket system, a runbook wiki (deliberately containing some stale and some wrong runbooks — trusting documentation blindly is a scored failure mode), and the deploy pipeline (it can ship code fixes, roll back, scale, failover).

### 3.2 Worlds

v1.0 ships **six worlds**, each a distinct, fully self-hosted stack with realistic telemetry:

| World | Stack archetype | What it stresses |
|---|---|---|
| W1 | Kubernetes microservices e-commerce (~15 services, mixed languages) | Cascading failures, distributed tracing, partial degradation |
| W2 | Legacy LAMP monolith + cron jobs, no tests, sparse docs | Archaeology, fear of blast radius, tribal-knowledge runbooks |
| W3 | Data platform (queue + stream processor + warehouse + orchestrator) | SLA-bound pipelines, poison messages, backfill decisions |
| W4 | Multi-tenant SaaS API behind a gateway | Noisy neighbors, rate limiting, per-tenant SLOs in tension |
| W5 | CI/CD + artifact registry serving (simulated) developer traffic | Infra-for-engineers, lockfile/cache corruption, supply-chain hygiene |
| W6 | On-prem-style VM fleet (no orchestrator): DNS, mail relay, file store | When there is no `kubectl rollout undo`; snowflake servers |

Worlds are built from scratch or hard-forked-and-refactored (renamed services, restructured topology) so that memorized OSS demo apps (Online Boutique, DeathStarBench) provide no direct answer key. Each world has 40–80 **fault templates**; an episode instantiates a schedule from templates with randomized parameters, orderings, timings, and identifiers.

### 3.3 What an event looks like (ground truth by construction)

Every fault template defines, mechanically:

- the **injection** (e.g., a deploy that introduces an N+1 query under a feature flag that ramps at hour 3),
- the **oracle remediation timeline**: a scripted reference fix executed by the harness in calibration runs, establishing the best-achievable SLO trajectory for that fault,
- the **do-nothing trajectory**: the SLO damage if untouched,
- **trap dynamics** where applicable (e.g., an unclean restart corrupts a write-ahead log, converting a 5-minute incident into a 4-hour one — naive "turn it off and on again" strategies are punished by the world's physics, not by a judge),
- the **root-cause key** (a structured identifier) against which the agent's mandatory machine-readable postmortem field is string-matched.

## 4. Scoring: Error-Budget Retention

### 4.1 Primary metric

Each world defines SLOs over the synthetic traffic (availability and latency, per journey). For each episode:

```
EBR = clip( (B_agent − B_nothing) / (B_oracle − B_nothing), 0, 1 )
```

where `B_x` is the error budget remaining at shift end under the agent / do-nothing / oracle-remediation policies, on the **identical** fault schedule and seed. EBR is therefore:

- **0** = the agent was worthless (or net-harmful — actions that burn budget on healthy systems push toward 0, so blast-radius discipline and false-alarm abstention are priced in automatically, with no separate judge or rubric),
- **1** = the agent matched scripted-expert remediation,
- fully **mechanical**: computed from traffic-probe measurements, no LLM judge, no rubric grader anywhere in the headline number.

### 4.2 Reliability is the headline

The reported headline is **EBR@5: the median and the minimum across 5 paired-seed episodes per (world, schedule) cell.** Publishing min-of-5 alongside median makes the τ-bench pass^k lesson — reliability collapse hidden by pass@1 — impossible to bury. A model that scores median 0.60 / min 0.55 is a different product than median 0.60 / min 0.05, and on-call is precisely the job where the second model is useless.

### 4.3 Secondary metrics (reported, not blended)

- **MTTR** distribution vs oracle (per fault class)
- **Root-cause accuracy**: structured postmortem field vs injection key (mechanical)
- **False-action rate**: state-mutating actions taken during false-alarm/healthy windows
- **Toil completion** under incident pressure
- **Cost**: $ of model spend per shift — published ARC-Prize-style next to every score
- **Survival curve**: % of episodes where the agent retains >0 budget at hour 24/72/168 of simulated time — the long-horizon coherence readout

### 4.4 Human baseline

Practicing SREs (contracted, ≥5 yrs experience) run the same shifts through the *same* tool interface (terminal + dashboards, no special access), establishing: (a) the human EBR distribution, (b) human MTTR per fault class, (c) the human-cost anchor ($/shift). The narrative number at launch: *"frontier agents preserve X% of the error budget a human SRE preserves, at Y% of the cost."*

## 5. Contamination, gaming, and validity

- **Procedural instantiation.** Fault templates × parameter randomization × renamed topologies mean no fixed answer key exists to memorize. The *eval* fault schedules and two of six worlds are private (semi-private split, ARC/FrontierMath-style — but with the conflict-of-interest lesson applied: no lab funds or gets privileged access to the private split, enforced by the governance structure in §9).
- **Quarterly fault packs.** Fresh templates each quarter (sourced from sanitized real postmortems — a large public corpus exists and accumulates as a renewable resource), so the benchmark refreshes like LiveCodeBench/MathArena but with stable scoring semantics: EBR's oracle normalization makes scores comparable across packs by construction.
- **Anti-shortcut physics.** The score is an integral over system state, not a terminal check, so "find the magic command" strategies that plague terminal benchmarks don't transfer; trap faults punish cargo-cult remediation; runbook errors punish blind document-following.
- **Eval-awareness pressure is blunted** because behaving like a careful operator *is* the optimal policy; there is no separately detectable "test mode" answer pattern to switch into (contrast BrowseComp, where Anthropic documented Opus 4.6 recognizing the eval).
- **Infrastructure-noise discipline** (Anthropic showed VM resourcing alone moves Terminal-Bench ~6 pts): PagerBench runs on **simulated time** — the world clock advances on harness ticks, decoupled from wall-clock, so host CPU contention and model latency don't leak into scores; resource specs are pinned in the harness; all randomness is seeded; paired-seed design gives every model the identical schedule.
- **Construct validity.** The claim is deliberately narrow: PagerBench measures *operational reliability of software systems under degradation*, not "general agency." The target construct has an external criterion to validate against — incident outcomes and MTTR distributions in real organizations — and v1.1 includes a criterion-validity study correlating EBR with practitioner ratings on replayed real incidents.

## 6. Harness and cost

- **Harbor-compatible** (the Terminal-Bench harness labs already integrate with), open-source, Dockerized; one world ≈ 8–16 containers, runnable on a single large VM.
- A full eval = 6 worlds × 4 schedules × 5 seeds = **120 episodes**. At 2–6 wall-clock hours each, parallelized, a full run completes in <48 hours on commodity cloud for roughly **$3–8K of compute + model spend per model** — comparable to GDPval-AA-class agentic evals, an order cheaper than PaperBench.
- A public **dev split** (2 worlds, sample fault packs, full harness) lets anyone reproduce, build scaffolds, and self-report; the steward runs the private split.

## 7. Build plan and feasibility

| Phase | Duration | Output |
|---|---|---|
| 0. Design partners | 6 wks | 4–6 practicing SREs + 2 eval scientists; fault taxonomy v1 from public postmortem corpora |
| 1. W1 + W2 vertical slice | 3 mo | 2 worlds, 60 fault templates, traffic gen, EBR pipeline, oracle calibration |
| 2. Frontier pilot | 6 wks | 5 frontier models + human baseline on the slice; variance/discrimination report; pre-registered analysis |
| 3. Full v1.0 | 4 mo | 6 worlds, 300+ templates, paper, public dev split, leaderboard with verified runs |

Team: ~4 engineers (infra-heavy), 1 eval scientist, SRE contractors. Estimated build cost: **$1.5–2.5M** through v1.0 — in line with what Scale, Mercor, Andon, and Laude spent on comparable agentic benchmarks. The single largest technical risk is **oracle calibration drift** (the scripted reference fix must be genuinely near-optimal or EBR>1 truncation distorts rankings); mitigation: oracle timelines are re-validated against the best human-SRE runs in Phase 2, and EBR is re-anchorable without changing semantics.

## 8. Prior art and differentiation

| Existing work | What it is | Why PagerBench is not it |
|---|---|---|
| **AIOpsLab** (Microsoft Research) | Single-incident detect/localize/resolve tasks on one microservice app | Discrete short tasks, one app, no sustained duty, no SLO-integral scoring, no traffic-measured ground truth; research adoption only |
| **ITBench** (IBM, 2025) | SRE/FinOps/CISO scenario suite | Same: isolated scenarios, rubric/check-based, no continuous responsibility, near-zero lab adoption |
| **Terminal-Bench 2.x** | One-shot terminal tasks incl. some sysadmin | Binary terminal tasks; no live system, no traffic, no time dimension, no inaction scoring |
| **SWE-bench family / FrontierCode** | Repair/build in healthy repos | Greenfield; episode ends at the patch; no operational consequence model |
| **Vending-Bench 2** | Long-horizon business sim | LLM-simulated counterparties, dollar metric with author-acknowledged huge run variance; measures business coherence, not technical operations |
| **τ²-bench** | Customer-service dual-control dialogs | Conversation-scale, LLM user simulator; pass^k lesson adopted, domain not |
| **TheAgentCompany** | Simulated-company task breadth | Breadth over depth; no SLO physics, no degradation, checkpoint-based grading |
| **Epoch × METR long-horizon SWE** (in development) | Long-horizon software *development* | Building, again — not operating. Complementary, not competing; their existence confirms the long-horizon appetite while leaving operations unclaimed |

The operations niche has academic toeholds but **no benchmark with frontier-lab adoption, no SLO-integral scoring, no sustained-duty format, and no judge-free long-horizon metric**. That is the same shape SWE coding was in early 2023 (HumanEval existed; SWE-bench made labs report it) and customer service was in mid-2024 (before τ-bench).

## 9. Governance and the adoption path

The 2025–26 lesson is unambiguous: **third-party stewards are the adoption kingmakers** (GDPval only became cross-lab via Artificial Analysis; Terminal-Bench spread via Laude/Stanford neutrality; FrontierMath's lab funding became a permanent asterisk). Therefore:

1. **Independent steward from day one** — a nonprofit or university-affiliated entity holds the private split and runs verified evaluations. No frontier lab may fund the private split or receive privileged access; corporate sponsorship is accepted only for the open dev split, disclosed.
2. **Launch playbook**: pre-registered pilot → arXiv paper with 5 frontier scores + human baseline + variance analysis → public leaderboard with cost-per-shift column → offer Artificial Analysis / Epoch operational partnership (they have stated gaps in exactly this area) → quarterly fault-pack releases keep it news.
3. **The wedge into model cards**: launch scores of 15–35% EBR against a ~70–85% human-SRE baseline give labs 2+ years of reportable progress on a metric their enterprise customers ask about unprompted. The first lab to feature it gets the "we measure what enterprises actually need" story — the same dynamic that made Anthropic feature FrontierCode within one day of its release.

## 10. Honest limitations

- **Sim-to-real gap**: six worlds cannot capture organizational reality (humans to coordinate with, political constraints, vendor support calls). v1 scopes to *solo technical* on-call deliberately; a human-coordination extension is future work, not a launch promise.
- **World construction is expensive and judgment-laden**; fault realism depends on the SRE design partners. Mitigated by sourcing from real postmortems and publishing the taxonomy for critique.
- **EBR compresses a rich trajectory into one number**; the secondary metrics exist precisely so the headline can't hide pathologies, and raw trajectories are published for verified runs.
- **Six worlds → finite diversity**: a model could overfit world archetypes via the dev split. The private worlds, procedural instantiation, and quarterly packs are the countermeasure; world count grows over time.

---

## Appendix A: The research base (June 2026)

Compiled from: Anthropic Claude Fable 5 / Mythos 5 announcement and system card coverage; OpenAI GPT-5.5 announcement/system card coverage; Google Gemini 3.1 Pro model card; Epoch AI Benchmarking Hub; METR Time Horizon 1.1 + limitations note; Artificial Analysis (GDPval-AA, Intelligence Index, Terminal-Bench Hard); per-benchmark dossiers (SWE-bench/Pro, Terminal-Bench 2.x, OSWorld-Verified, τ²-bench, GPQA, AIME/MathArena, MMMU-Pro, HLE + FutureHouse audit, ARC-AGI-2/3, FrontierMath + funding controversy, BrowseComp + eval-awareness finding, Vending-Bench 1/2, GDPval + grader-agreement critique, SWE-Lancer, MRCR v2, LiveCodeBench, MCP-Atlas, FrontierCode, Agents' Last Exam, Remote Labor Index, Mercor APEX, HealthBench, PaperBench, MLE-bench, Cybench, SpreadsheetBench, AbstentionBench, LoCoMo/LongMemEval); "Measuring what Matters" construct-validity survey (NeurIPS 2025); Anthropic engineering posts on infrastructure noise and eval awareness; adoption-pattern analysis across SWE-bench, τ-bench, ARC Prize, HLE, GDPval, Terminal-Bench.
