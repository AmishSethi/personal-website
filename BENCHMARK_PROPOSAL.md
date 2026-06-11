# OnCallBench: Can a Model Hold the Pager?

**A benchmark for autonomous production operations — measuring whether an AI agent can keep a live, degrading software system inside its service-level objectives over a sustained on-call shift.**

Status: v0.4 proposal · June 2026 · revised after three-track adversarial review (eval methodology, frontier-lab adoption, SRE domain + prior-art audit)

> Naming note: v0.3 was "PagerBench," renamed after review — one letter from OpenAI's PaperBench, trademark-adjacent to PagerDuty in the same product category, and "SREBench" is already taken twice (Parity; srebench.com). "OnCallBench" verified free as of June 2026; full USPTO/EUIPO clearance before any public release. The tagline survives the rename.

---

## 1. One-paragraph thesis

Every frontier benchmark that labs report today measures **building** things (SWE-bench Pro, FrontierCode, Terminal-Bench) or **producing** things (GDPval, GDP.pdf) — one-shot deliverables in healthy environments. Nothing that frontier labs report measures **operating** things: holding responsibility for a live system, under traffic, while it degrades in unscripted ways, where doing nothing has a cost-per-minute, doing the wrong thing makes it worse, and many alerts should be ignored. OnCallBench puts an agent on-call for a simulated multi-day shift over realistic production stacks with injected faults, live closed-loop synthetic traffic, and concurrent routine work, and scores it on a single, mechanical, judge-free headline number derived from **Error-Budget Retention (EBR)** — how much of the system's SLO error budget the agent preserves relative to a calibrated reference remediation — with reliability enforced through a verification gate rather than buried in a footnote. The pitch to labs in one line: *"Your model writes production code. Can it carry the pager?"*

## 2. Why this gap, and why now

### 2.1 The reporting landscape (June 2026)

Anthropic's Claude Fable 5 announcement (June 9, 2026) reports: SWE-bench Verified/Pro, Terminal-Bench 2.1, FrontierCode, OSWorld-Verified, HLE, GDPval-AA, MMMU-Pro, CharXiv; the system card adds Vending-Bench 2, BrowseComp, OfficeQA Pro. OpenAI's GPT-5.5 and Google's Gemini 3.1 Pro report near-identical sets. Three structural facts:

1. **The static-exam era is formally over.** Anthropic dropped GPQA Diamond and AIME as saturated; OpenAI formally deprecated SWE-bench Verified. The benchmarks labs still feature are agentic, economically framed, and score in the 20–50% range (FrontierCode Diamond 29.3%, Agents' Last Exam 24%, GDPval ~48% win-rate).
2. **Every featured benchmark is greenfield/one-shot.** SWE-bench-family: fix an issue in a healthy, well-tested repo, then the episode ends. Terminal-Bench: discrete tasks with a binary check. GDPval: produce a deliverable, get graded once. τ²-bench: a bounded customer conversation. Even Vending-Bench — the closest thing to sustained responsibility — is a turn-based business sim with LLM-simulated counterparties and author-acknowledged run-to-run variance.
3. **Long-horizon measurement is hitting a ceiling.** METR's 50%-time-horizon for Claude Mythos is ≥16 hours — at the edge of their suite (only 5 of 228 tasks ≥16h), and METR's own limitations note says measurement above ~16 hours is unreliable. The regime between "16 hours" and "weeks of responsibility" is the acknowledged open frontier. Epoch × METR's in-development long-horizon benchmark (MirrorCode) targets software *development* — operations remains unclaimed.

### 2.2 The gap

The 2026 gap-map evidence (Epoch benchmarking-hub stated gaps, METR limitations note, TheAgentCompany's findings, the "Measuring what Matters" construct-validity survey) converges on several under-measured areas. OnCallBench deliberately sits at the intersection of four:

- **Maintenance and operations, not greenfield.** SWE benchmarks resolve issues in healthy OSS repos. No adopted benchmark measures keeping a degrading production system alive: triage, mitigation, root-cause, toil, postmortems. The ops-adjacent efforts that exist (§8) are all snapshot- or discrete-task-format.
- **Sustained responsibility beyond the METR ceiling.** A multi-day shift with concurrent, interleaved events measures coherence-over-time that one-shot tasks cannot.
- **Reliability as a first-class property.** τ-bench introduced pass^k (90% pass@1 → ~57% pass^8) but labs still headline pass@1 — so OnCallBench does not ask labs to volunteer their worst number; it makes reliability a *verification gate* a neutral steward enforces (§4.2).
- **Calibrated inaction.** AbstentionBench showed reasoning training *degrades* abstention. No agentic benchmark scores knowing when **not** to act. On-call is the natural habitat: false alarms, self-healing blips, and "watch it, don't touch it" situations are embedded in every shift, and wrong interventions mechanically burn error budget.

### 2.3 Why labs would adopt it (the demand side)

- **It is the enterprise buyer's actual question for 2026–27.** Labs sell coding agents today; the next market is agents that *run* what they build. Every major observability/incident vendor shipped "AI SRE" products in 2025–26 (Datadog Bits AI SRE GA, Traversal, Cleric, incident.io) with unaudited marketing claims ("70–90% faster resolution") — which strengthens, not weakens, the case for one neutral measure.
- **The one-number narrative is legible far beyond ML.** Uptime, SLOs, and error budgets are existing industry vocabulary (Google SRE book). "Can AI hold the pager?" needs no explanation. On comparison-table surfaces, the score is branded simply "**OnCallBench: 31**" — EBR is the methodology name, not the label, exactly as "SWE-bench Pro: 63%" doesn't print its grading semantics in the cell.
- **Headroom (stated as a hypothesis, not a fact).** We *hypothesize* frontier agents land at 15–35% of the headline metric at launch, based on the difficulty stack (long-horizon + concurrency + irreversibility vs. 29.3% on FrontierCode Diamond for mere mergeability). Phase 2 exists to test this; if pilot scores cluster too high or too low to discriminate, difficulty is retuned before launch, not after.
- **Objective, judge-free headline.** After the HLE label-noise scandal (~18–30% contested answers), the GDPval grader-agreement problem (66% vs 71% human-human), and the FrontierMath conflict-of-interest scandal, labs and third parties visibly prefer mechanical ground truth. OnCallBench's headline is measured, not judged: the harness injected the fault, and SLO compliance is read off traffic outcomes. (Scoped claim: the *headline* is judge-free; two secondary metrics use rubric-like checks, disclosed as such in §4.3.)
- **It fits the adoption playbook** (per the SWE-bench / Terminal-Bench / GDPval-AA track record): automatic contamination-resistant verification ✓, launch headroom ✓ (pending pilot), neutral steward as a *precondition not an aspiration* ✓ (§9), reproducible harness with an official fast split ✓ (§6), memorable metric ✓, expert-sourced realism ✓ (faults curated with practicing SREs from postmortem corpora).
- **Year-one honesty:** at launch, all models cluster low and the story is the human-baseline gap; the score-vs-competitor value arrives in year two when someone first crosses ~50% of the human anchor and claims "the first model that can hold a pager." That is the SWE-bench adoption arc, and quarterly fault packs keep the benchmark news in the interim.

## 3. What OnCallBench is

### 3.1 The episode

An **episode** is an on-call shift: the agent holds operational responsibility for one **world** (a fully containerized production stack, §3.2) for a compressed simulated duty period (7 simulated days). During the shift:

- **Live closed-loop traffic runs continuously.** A deterministic, seeded, *closed-loop* traffic generator (scripted clients with explicit retry/timeout/backoff policies that react to responses — so retry storms, thundering herds, and backpressure collapse emerge from real amplification dynamics, not scripted theater) exercises user journeys against the stack. Its measurements feed the score. No LLM simulation anywhere in the loop.
- **Scheduled events fire from a hidden fault schedule** (12–20 per episode): infrastructure faults, bad deploys landing through CI, dependency brownouts, data-poisoning events, cert expiries, disk-fill slow burns, cascading combinations — interleaved with **false alarms, transient self-healing blips, and red herrings** whose correct response is calibrated inaction.
- **Alert volume is decoupled from event count.** Each real event fans out realistically (a cascade can emit 10–50 alerts); a chronic drizzle of known-noisy alerts runs throughout. The agent must maintain a noise model, not just answer a doorbell — this is what makes triage-under-volume measurable.
- **Routine toil arrives via a ticket queue** — and toil is wired into world physics: a substantial fraction of toil items (≥30%) have downstream SLO consequences inside later schedule windows (the ignored cert-renewal ticket *is* Thursday's outage). Prioritization is thus measured by consequence, not claimed by assertion.
- **The agent has a real operator's toolkit**, nothing more: a terminal in a bastion container (kubectl/ssh/psql etc.), metrics API (Prometheus-compatible), logs, traces, the alerting feed, the ticket system, a runbook wiki (containing some stale and some wrong runbooks — blind document-following is punished by outcomes), and the deploy pipeline (ship fixes, roll back, scale, failover).
- **Observability is realistically incomplete.** Telemetry gaps are a first-class fault-template dimension: services without tracing, cardinality-limited metrics, a lying dashboard, and monitoring-of-monitoring faults (Prometheus itself down mid-incident). W2 and W6 have structurally poor coverage by design.
- **State-restore primitives are enumerated per world and priced.** Where snapshot/PITR legitimately exists, restores cost realistic simulated time and data loss — a tool, not a save-scumming exploit. No out-of-band reset path exists.

### 3.2 The time economy (normative spec, not an implementation detail)

Review identified this as the design's load-bearing wall, so it is specified normatively:

- Simulated time advances from three sources only: (a) **tool invocations**, charged from a pinned public cost table reflecting realistic execution durations (a `kubectl get pods` costs seconds; a database restore costs its realistic duration); (b) **token generation**, charged at a published tokens-per-sim-second exchange rate — thinking is not free, so the agent cannot stop the world to deliberate; (c) an explicit **`wait(t)`** primitive for deliberate watching.
- **Background world processes consume sim-time on their own schedules** (rollouts progress, disks fill, the fault scheduler fires) — the world never halts, and concurrent incidents genuinely compete for the agent's time budget.
- Wall-clock is fully decoupled: host CPU contention and model latency cannot leak into scores (the Terminal-Bench ±6pt lesson), while time pressure — the core construct of on-call — is preserved through the token charge.
- **Pre-registered ablation (Phase 2): rankings must be stable under ±2× perturbation of the tick-cost table.** If they are not, EBR differences are artifacts of the time economy and the design returns to the shop. A secondary wall-clock-coupled variant is reported for transparency.
- The human-baseline condition charges sim-time from wall-clock seconds at a separately calibrated exchange rate (humans cannot ingest telemetry at token speed; §4.4), with pause protocols for multi-hour shifts.

### 3.3 Worlds

v1.0 ships **six worlds**, each a distinct, self-hosted stack with realistic telemetry:

| World | Stack archetype | What it stresses |
|---|---|---|
| W1 | Kubernetes microservices e-commerce (~15 services, mixed languages) | Cascading failures, distributed tracing, partial degradation |
| W2 | Legacy LAMP monolith + cron jobs, no tests, sparse docs, poor telemetry | Archaeology, blast-radius fear, tribal-knowledge runbooks |
| W3 | Data platform (queue + stream processor + warehouse + orchestrator) | SLA-bound pipelines, poison messages, backfill tradeoffs |
| W4 | Multi-tenant SaaS API behind a gateway | Noisy neighbors, rate limiting, per-tenant SLOs in tension |
| W5 | CI/CD + artifact registry serving simulated developer traffic | Infra-for-engineers, cache corruption, supply-chain hygiene |
| W6 | On-prem-style VM fleet (no orchestrator): DNS, mail relay, file store | No `kubectl rollout undo`; snowflake servers; thin telemetry |

Worlds are built fresh or hard-forked-and-refactored (renamed services, restructured topology) so memorized OSS demo apps provide no answer key. Each world carries 40–80 **fault templates**; an episode instantiates a schedule with randomized parameters, orderings, timings, and identifiers. Four of six worlds and all sample fault packs are public (dev split); two worlds and all eval schedules are private (§5).

### 3.4 Fault templates and ground truth

Every fault template mechanically defines:

- the **injection** (e.g., a deploy introducing an N+1 query behind a feature flag that ramps at hour 3),
- its **oracle class** — see §4.1: *oracle-clean* (a unique dominant remediation exists) or *oracle-contested* (legitimate strategy tradeoffs exist, e.g., rollback-vs-fix-forward where rollback loses data; each world publishes its RPO/RTO and cost policies so tradeoffs are decidable in principle),
- **trap dynamics** where applicable (an unclean restart corrupts the write-ahead log, converting a 5-minute incident into a 4-hour one — cargo-cult remediation is punished by world physics, not by a judge),
- the **root-cause key set**: a small closed set of acceptable keys drawn from a published multi-level causal taxonomy (component / mechanism / trigger), against which the agent's mandatory structured postmortem field is matched — one submission, no multi-guess; key-matching is validated against blinded human-SRE postmortems in the pilot, because "root cause" in cascades legitimately admits multiple framings and we refuse to import HLE-style label noise through the back door.

**Schedule-level oracle.** Because episodes contain 12–20 interacting events, the reference is defined per *schedule*, not summed per template: a composite scripted remediation timeline executed by the harness under the same time economy, handling the events as an expert would interleave them. Phase 2 validates every schedule oracle against the best human-SRE run on that schedule; any schedule where a human beats the oracle gets its oracle revised before that schedule is used for scoring. Per-schedule oracle-vs-best-human gaps are published as a standing calibration appendix.

## 4. Scoring

### 4.1 The core quantity: Error-Budget Retention

Each world defines SLOs over the synthetic traffic (availability and latency, per journey). For each episode:

```
EBR = (B_agent − B_nothing) / (B_oracle − B_nothing)
```

where `B_x` is error budget remaining at shift end under the agent / do-nothing / schedule-oracle policies on the identical seed and schedule.

- **The harm tail is not censored.** EBR is *unclipped below zero* (winsorized at −1 for aggregation stability): an agent that makes things worse than doing nothing scores negative, and **Harm Rate** — the % of episodes with `B_agent < B_nothing` — is a mandatory companion statistic on every leaderboard surface. (v0.3 clipped at 0; review correctly showed that censoring rewards late-shift desperation gambles and erases exactly the blast-radius signal the benchmark claims to price in.)
- The upper clip at 1 remains, with **% of episodes at ceiling** published as a saturation/oracle-weakness diagnostic.
- **Denominator admission rule:** a schedule is scorable only if `(B_oracle − B_nothing)` ≥ 25% of the episode's total error budget — preventing near-zero denominators from amplifying noise. False alarms, blips, and watch-don't-touch windows are *embedded within* scorable schedules (where wrong actions burn real budget) rather than scored as standalone 0/0 episodes; pure-abstention skill is additionally read out by the False-Action Rate (§4.3).
- **Aggregation (full formula):** episode EBRs → mean per (world, schedule) cell → equal-weighted mean across cells = the model's EBR. Budget-weighted aggregation is also published; the pre-registration commits to both before any model is scored.

### 4.2 Headline and the reliability gate

- **Headline: one number.** Median cell-aggregated EBR, branded "OnCallBench: NN". No compound notation on any surface above the methods section.
- **Reliability is a gate, not a co-headline.** A leaderboard score is marked **Verified-Reliable** only if (a) the P10 of the episode-level EBR distribution (estimated from k = 10 seeds per cell on a reduced schedule grid at equal compute) is ≥ 0.5 × the median, and (b) Harm Rate ≤ 5%. Labs never have to print their worst number — the steward's gate does the work pass@1 marketing has dodged since τ-bench. (v0.3's median/min-of-5 co-headline is gone: review showed min-of-5 is an extreme order statistic that flips rankings by luck, and labs would simply omit it.)
- **Statistical discipline:** bootstrap CIs accompany every published delta; rank claims are made only where CIs separate; the headline is reported on oracle-clean templates, with oracle-contested templates scored against a published *strategy frontier* (each acceptable strategy gets its own calibrated trajectory; the agent is credited against the best-matching one) and reported as a separate column until the frontier methodology has a pilot behind it.

### 4.3 Secondary metrics (reported, not blended)

- **Harm Rate** (mandatory, leaderboard-level — see §4.1)
- **MTTR** distribution vs oracle, per fault class
- **Root-cause accuracy** (closed-key match, §3.4)
- **False-Action Rate**: state-mutating actions during false-alarm/healthy windows
- **Toil consequence score**: SLO damage attributable to dropped toil (mechanical, via §3.1 toil-physics)
- **Cost**: $ per shift, ARC-Prize-style, next to every score
- **Survival curve**: % of episodes retaining >0 budget at simulated hour 24/72/168 — the long-horizon coherence readout

Disclosure: toil-ticket completion checks and postmortem-quality checks are rubric-like; they are secondary, never blended into the headline, and labeled as such.

### 4.4 Human baseline — two conditions, honestly framed

Practicing SREs (≥5 yrs, contracted) run the same shifts through the same tool interface under the calibrated human time economy (§3.2):

- **Cold**: first contact with the world — symmetric with the agent's situation and framed *explicitly* as "cold human," never as "a human SRE."
- **Warm**: the same operator's third-plus shift on that world after onboarding with the dev fault pack and runbooks — approximating real on-call familiarity.

Both anchors are published with pre-registered n and CIs; the launch narrative uses whichever comparison the data supports, stated as "agents vs cold humans: X%; vs familiarized humans: Y%." (Review correctly flagged that a cold-contractor-only baseline systematically flatters AI, and practicing SREs would say so loudly and publicly.)

## 5. Contamination, gaming, and validity

- **Probe defense (new section — review identified the headline's attack surface).** The agent has root; the score is traffic outcomes; therefore: (a) probe traffic is statistically indistinguishable from background load at the network level (shared source pools, jittered patterns, no fingerprintable user-agents); (b) **hidden canary journeys**, distinct from the scoring probes, are checked post-hoc for divergence — serving the prober while real journeys die is detected; (c) probe-targeted manipulation (whitelisting, caching probe paths, prioritizing probe tenants) is an automatic disqualification, screened mechanically and audited by the steward; (d) the probe/canary design is red-teamed with scaffold developers before launch, with bounties.
- **Procedural instantiation + behavioral mutation.** Parameter randomization changes identifiers; review noted telemetry *signatures* are the memorizable part. Private-split templates are therefore **behaviorally mutated** (same root-cause class, different symptom signature), not just re-parameterized, and the **dev-vs-private score gap per model is published every quarter** as a standing overfit diagnostic.
- **Postmortem-corpus contamination, framed precisely.** Sourcing faults from public postmortems puts symptom→cause *knowledge* in pretraining — which is fine and even desirable (human SREs read postmortems too; that's competence, not leakage). The contamination boundary OnCallBench enforces is the mapping from *this world's* signals to *this schedule's* instantiation, which procedural generation plus behavioral mutation keeps novel, and which the quarterly gap-audit measures rather than asserts.
- **Quarterly fault packs, honestly versioned.** Fresh template packs each quarter keep the benchmark news and contamination-resistant. v0.3 claimed cross-pack comparability "by construction"; review showed that contradicts oracle re-anchoring. Corrected position: **scores are versioned per oracle generation (EBR-v1, EBR-v2, …)**, longitudinal claims are made only within a version, and each pack ships a calibration appendix (oracle-vs-human gap per pack) so consumers can see where normalization is soft. This is the SWE-bench → SWE-bench-Verified precedent, adopted deliberately rather than suffered accidentally.
- **Anti-shortcut physics.** The score is an integral over system state, not a terminal check; trap faults punish cargo-cult remediation; runbook errors punish blind document-following; restore primitives are priced (§3.1). Eval-awareness pressure is blunted because behaving like a careful operator *is* the optimal policy — there is no separately detectable "test-mode" answer pattern (contrast BrowseComp, where Anthropic documented Opus 4.6 recognizing the eval).
- **Harness noise floor, demonstrated not asserted.** Coherently virtualizing time across a 15-service stack is the hardest engineering claim here, and done badly it would *introduce* nondeterminism. Phase 1 therefore includes a **replay-determinism study**: identical seed + scripted agent, N reruns, published EBR dispersion = the harness noise floor; **launch is gated on the noise floor sitting well below observed model deltas.** (This is the direct answer to Anthropic's infrastructure-noise finding that VM resourcing alone moves Terminal-Bench ~6 points.)
- **Scaffold policy (the settled 2026 convention, in writing):** the steward runs a pinned reference scaffold for the leaderboard; bring-your-own-scaffold scores live in a separate, self-reported column.
- **Construct validity, scoped.** OnCallBench measures *operational reliability of software systems under degradation* — not "general agency." The construct has an external criterion: v1.1 includes a criterion-validity study correlating EBR with practitioner ratings on replayed real incidents, and the root-cause taxonomy is validated against human postmortems in the pilot (§3.4).

## 6. Harness, cost, and the fast split

- **Open-source, Dockerized, Harbor-interoperable at the container/agent-interface layer** — stated plainly: the world engine (traffic generator, fault scheduler, time economy, ticket queue) is a bespoke orchestration layer *on top of* Harbor-style plumbing, not a Terminal-Bench task pack. Integration cost for a lab running it themselves is real (est. 2–4 engineer-weeks) — which is why the adoption model is steward-run, lab-cited (§9).
- **Full eval:** 6 worlds × 3 admitted schedules × 10 seeds = **180 episodes**, parallelized, <48h wall-clock. Cost: provisionally $8–25K/model all-in (review judged v0.3's $3–8K optimistic by 2–3× given telemetry-dense contexts; the pilot publishes token actuals and this number gets replaced by measurement).
- **Official fast split (new — the RL-checkpoint loop requirement):** 2 public worlds × 1 schedule × 3 seeds, compressed shifts, ~4 hours end-to-end, **with a pilot-validated ≥0.9 Spearman correlation to the full score as a release criterion.** Benchmarks that can't sit inside a training loop get run once per launch and stop being news; the fast split is built deliberately, not discovered accidentally.
- Public dev split: 4 worlds, sample fault packs, full harness, reference scaffold — anyone can reproduce, build, and self-report.

## 7. Build plan and feasibility

| Phase | Duration | Output |
|---|---|---|
| 0. **Steward + design partners** | 8 wks | **Signed steward with funded 2-year operations (precondition — see §9)**; 4–6 practicing SREs + 2 eval scientists; fault taxonomy v1; naming clearance |
| 1. W1 + W2 vertical slice | 3 mo | 2 worlds, 60 templates, closed-loop traffic, time-economy implementation, EBR pipeline, **replay-determinism study (noise-floor gate)** |
| 2. Frontier pilot | 8 wks | 5 frontier models + both human-baseline conditions; **pre-registered**: variance decomposition (world/schedule/seed/model-side), tick-cost ±2× ablation, power analysis that *sets* the final seed/schedule budget, test-retest reliability, oracle-vs-human validation, token-cost actuals, fast-split correlation |
| 3. Full v1.0 | 4 mo | 6 worlds, 300+ templates, paper, dev split, leaderboard with verified runs and cost column — **all five frontier scores published simultaneously with the paper** |

Team: ~4 infra-heavy engineers, 1 eval scientist, SRE contractors. Build cost: **$1.5–2.5M through v1.0, plus funded steward operations through year 2** (quarterly packs, oracle recalibration, container upkeep across a large rot surface — the line item whose absence killed comparable benchmarks). Largest technical risks, in order: (1) time-economy implementation correctness (gated by the replay study), (2) oracle calibration (gated by human validation + the contested-template frontier), (3) pilot discrimination — if five frontier models don't separate beyond CIs, the benchmark does not launch on schedule; it gets retuned. A benchmark whose paper contains no test-retest reliability figure does not deserve adoption, and this one will not ship without it.

## 8. Prior art and differentiation (updated after independent audit)

| Existing work | What it is | Why OnCallBench is not it |
|---|---|---|
| **ITBench / ITBench-AA** (IBM; **now run by Artificial Analysis**, Kaggle leaderboards Dec 2025, HF Enterprise Agents Jan 2026) | SRE/FinOps/CISO scenarios; ITBench-AA gives agents *offline snapshots* of K8s incidents and grades structured RCA JSON; "frontier models score below 50%" | Snapshot-and-grade, not sustained duty; no live traffic, no SLO integral, no time economy, no inaction pricing. The most-adopted neighbor — and the strongest proof the *category* has third-party demand |
| **AIOpsLab** (Microsoft Research) | Discrete detect/localize/diagnose/mitigate episodes on one live microservice app | Short single-incident episodes; no duty period, no SLO-integral, no concurrent events |
| **Cloud-OpsBench** (Feb 2026) | 452 fault cases, deterministic "digital-twin" snapshots, positioned as an RL environment | Snapshot paradigm again; per-case grading |
| **DevOps-Gym** (2026) / **OpenRCA** (ICLR'25) / RCAEval line | DevOps task gym; offline-telemetry root-cause analysis (335 failures) | Discrete tasks / offline RCA; no responsibility over time |
| **SREBench** (Parity; also srebench.com) / **SRE-skills-bench** (Rootly) | K8s root-causing challenges; skills quizzes | Single-incident RCA; also occupy the naming space — one reason for the rename |
| **Terminal-Bench 2.x** | One-shot terminal tasks incl. sysadmin | Binary checks; no live system, traffic, or time dimension |
| **SWE-bench family / FrontierCode** | Repair/build in healthy repos | Greenfield; episode ends at the patch |
| **Vending-Bench 2** | Long-horizon business sim | LLM-simulated counterparties; business coherence, not technical operations; author-acknowledged variance |
| **τ²-bench** | Customer-service dual-control dialogs | Conversation-scale; LLM user simulator; its pass^k lesson is adopted here as a gate |
| **TheAgentCompany** | Simulated-company task breadth | Checkpoint grading; no SLO physics or degradation |
| **Epoch × METR MirrorCode** (in development) | Long-horizon software *development* | Building, not operating — complementary, and confirms long-horizon appetite while leaving ops open |
| **ControlTheory's AI-SRE eval methodology post** | How-to for live test environments with fault injection | A methodology blog, not a benchmark; closest in spirit, confirms practitioner demand |

Five separate 2025–26 efforts circle this niche; **none attempts sustained duty or SLO-integral scoring.** That is the verified open ground — the same shape SWE coding had in early 2023 and customer service had before τ-bench. ITBench-AA's existence cuts one way that matters: it proves third parties want an ops benchmark, and it does not occupy the duty-format slot.

## 9. Governance and the adoption path

The 2025–26 lesson is unambiguous: **third-party stewards are the adoption kingmakers** (GDPval became cross-lab only via Artificial Analysis; Terminal-Bench spread via Laude/Stanford; FrontierMath's lab funding is a permanent asterisk), and the graveyard (AIOpsLab, ITBench-v1, WorkBench) is full of benchmarks that *existed* but had no one continuously running frontier models on them and turning scores into news. Therefore:

1. **A signed steward with funded year-2 operations is a Phase 0 precondition, not a launch-playbook step.** No steward, no build. Candidate order, post-audit: **Epoch AI / METR** (their 2026 roadmap is dev-side long-horizon — ops is the complementary slot, and the METR-ceiling framing in §2.1 is their own stated problem); **Laude Institute** (Terminal-Bench operational pedigree); **Artificial Analysis** (pitched honestly as the *duty-format upgrade* alongside their snapshot-format ITBench-AA, not as filling an empty slot). The steward holds the private split and runs all leaderboard evaluations; **labs never self-run for the leaderboard — they cite verified scores**, the FrontierCode-in-one-day mechanism.
2. **Trust architecture for the private split** (the FrontierMath lesson, fully applied): no frontier lab funds the private split or receives privileged access; **retired quarterly packs are fully disclosed** — schedules, oracle scripts, world configs, and complete trajectories of all verified runs become public when a pack rotates out; a **standing oracle-adjudication channel** lets any lab challenge a specific episode's oracle and obtain re-scoring under a revised oracle; per-pack oracle-vs-human calibration is published (§5). A trailing lab can therefore audit *why* it lost one quarter later — which is precisely what converts a bad score from "attack the benchmark" into "fix the model."
3. **Comms risk handled ex ante:** all five frontier launch scores publish simultaneously (no one is singled out); trajectory publication for current packs follows a disclosure policy agreed with participating labs before the pilot (catastrophic-episode anecdotes — "the model bricked production" — are exactly the screenshot risk that would otherwise make every lab decline); the steward, not labs and not us, controls release timing.
4. **Safety note on the fault library:** generic fault injection adds nothing beyond Chaos Mesh/Gremlin and public postmortems, and the taxonomy is published openly. The one artifact with modest sabotage-uplift value — the curated trap-dynamics + oracle-timeline corpus ("which remediations convert small outages into catastrophes") — stays in the private split as a safety measure as well as an anti-contamination one, with a responsible-release note in the paper.

## 10. Honest limitations

- **Sim-to-real gap**: six worlds cannot capture organizational reality (humans to page, escalation politics, vendor support calls). v1 scopes to *solo technical* on-call deliberately; a human-coordination extension is future work, not a promise.
- **The oracle is the residual judge.** Judge-free scoring relocates judgment into oracle construction; the clean/contested split, human validation, adjudication channel, and retired-pack disclosure manage this — they do not eliminate it. We say so plainly rather than claiming the problem away.
- **Six worlds → finite archetype diversity**; a model could overfit world archetypes via the dev split. Behavioral mutation, the quarterly gap-audit, private worlds, and world-count growth are the countermeasures — and the gap-audit makes overfit *visible* rather than deniable.
- **EBR compresses a rich trajectory into one number**; Harm Rate is mandatory at leaderboard level, secondary metrics are published, and verified-run trajectories disclose fully at pack retirement so the headline cannot hide pathologies indefinitely.
- **Cost numbers are estimates until the pilot publishes actuals**; the human-baseline budget (senior SREs × enough shifts for usable CIs × two conditions) is the dominant uncertainty and is costed in Phase 2, not hand-waved.

---

## Appendix A: Review provenance

v0.4 incorporates three independent adversarial reviews of v0.3: (1) **eval-methodology** (verdict: major revisions — led to: unclipped EBR + Harm Rate, normative time economy + tick ablation, probe-defense section, schedule-level oracle, denominator admission rule, P10@k=10 reliability gate replacing min-of-5, human time-parity, oracle versioning replacing the "comparable by construction" claim, replay-determinism gate, restore-primitive pricing, toil physics, multi-key root-cause taxonomy, cost restatement); (2) **frontier-lab adoption** (verdict: would adopt with changes — led to: steward-as-precondition, single-number headline + steward-enforced gate, simultaneous five-model launch + disclosure policy, pre-registered power/test-retest requirements, retired-pack disclosure + adjudication channel, official fast split, pinned reference scaffold, year-one/year-two framing); (3) **SRE domain + prior-art audit** (verdict: major revisions — led to: the rename, the corrected prior-art table incl. ITBench-AA/Cloud-OpsBench/DevOps-Gym/SREBench/SRE-skills-bench/OpenRCA, the re-sequenced steward strategy, closed-loop traffic, alert fan-out, observability-gap faults, cold/warm human baselines, contested-oracle strategy frontier, contamination-boundary framing, trap-library gating).

## Appendix B: The research base (June 2026)

Compiled from: Anthropic Claude Fable 5 / Mythos 5 announcement and system-card coverage; OpenAI GPT-5.5 announcement/system-card coverage; Google Gemini 3.1 Pro model card; Epoch AI Benchmarking Hub; METR Time Horizon 1.1 + limitations note; Artificial Analysis (GDPval-AA, ITBench-AA, Intelligence Index, Terminal-Bench Hard); per-benchmark dossiers (SWE-bench/Pro, Terminal-Bench 2.x, OSWorld-Verified, τ²-bench, GPQA, AIME/MathArena, MMMU-Pro, HLE + FutureHouse audit, ARC-AGI-2/3, FrontierMath + funding controversy, BrowseComp + eval-awareness finding, Vending-Bench 1/2, GDPval + grader-agreement critique, SWE-Lancer, MRCR v2, LiveCodeBench, MCP-Atlas, FrontierCode, Agents' Last Exam, Remote Labor Index, Mercor APEX, HealthBench, PaperBench, MLE-bench, Cybench, SpreadsheetBench, AbstentionBench, LoCoMo/LongMemEval); ops-adjacent prior art (AIOpsLab, ITBench/ITBench-AA, OpenRCA, Cloud-OpsBench, DevOps-Gym, SREBench, SRE-skills-bench, RCAEval, ControlTheory); "Measuring what Matters" construct-validity survey (NeurIPS 2025); Anthropic engineering posts on infrastructure noise and eval awareness; adoption-pattern analysis across SWE-bench, τ-bench, ARC Prize, HLE, GDPval, Terminal-Bench.
