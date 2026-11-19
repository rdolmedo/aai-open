# AAI-open versions (#481)

A version pins everything that can move an AAI-open score: items, harness and protocol, sampling, graders and judges,
the aggregate, serving numerics and the code. A change to any of these makes a new version; it never shifts the
scores of an existing one silently.

| Version | What it is | Code | Status |
|---|---|---|---|
| **v0** | The benchmark exactly as used for the Snowball scaling work: the M5 ladder evals of 2026-11-15..18 (campaign `m5l`) and the numbers behind marin#11563 | tags `aai-open-v0-evals` (aa-evals `e3dc54f9`), `aai-open-v0-serving` (aa-serving `89d85472`) | frozen |
| **v0.1** | The benchmark as it runs from `main` after the 11-18/19 unification (#480) and run-time work (#451, #531, #478) | tag `aai-open-v0.1` (the `main` commit that holds this file) | current |

**A run's version.** From v0.1 on, every new run records `aai_open: {version, deviations, source}` in its `config.json`
(pi_eval and the HLE, Omniscience, GDP.pdf and SciCode clients), and their summaries carry `aai_open_version`; a
collected GDPval / CritPt / AutomationBench dir answers for its Pi run. Read it with
`aa_evals.common.aai_open.of_run(run_dir)` / `of_config(cfg)` (the stable public readers),
`python -m aa_evals.common.aai_open show RUN_DIR ...`, `grade.py status`, or `python -m aa_evals.campaign.score CONFIG`
(slot scores, aggregate, versions).
- A run that deviates from the spec in a way the launcher or the client can check records `v0.1+custom`. The client's
  own deviations are listed in the run config; the campaign-level ones are in the launcher's `show` output and in its
  launch-log `suite` events (`aai_open_deviations`).
- `source: launcher` means the campaign launcher checked the item plan and the campaign pins. `source: client` (a run
  started by hand) means only the client's own settings were checked: the label then says nothing about the items.
- A malformed `$AAI_OPEN_VERSION` never fails a run: it becomes `v0.1+custom` (a run started directly names the bad
  value in its deviations; the single-turn submitters forward plain `v0.1+custom` into their jobs).
- v0 runs predate the field (they read `unrecorded`). Their membership in v0 is the run manifest
  [`v0_runs.json`](v0_runs.json), built from the runs' own records by [`collect_runs.py`](collect_runs.py).

**Pending changes that will each bump the version** (#481 body): #406 finish-notice compliance (parked), #340 CritPt
references, #381 easier TB-Sci-lite, #479 GDPval rubric scoring, a new item plan (#181).

---

## v0: the Snowball scaling benchmark

### Scope

- **Runs.** Campaign `m5l` (config `/data/aai/m5/ladder/eval/m5l.json`, launch log
  `/data/aai/aa_evals/dispatch/m5l-launch/events.jsonl`). 145 full-stage runs, of which **111 are scored**. Of
  the other 34, 28 are superseded (`superseded_by` in the manifest) and 6 are suites dropped from the lean set
  (`unused`). The scored runs:

  | Model key | Model | Suites | Notes |
  |---|---|---|---|
  | `super` | Nemotron 3.5 Super VL | 9 | baseline |
  | `qwen35-4b` | Qwen3.5-4B | 9 | baseline |
  | `18k-1p0b` + `18k-1p0b-r2` | Snowball step-18000 + M5 SFT 1B | 5 + 4 | r2 re-ran TB2.1, CritPt, SciCode and HLE at the 12-h caps |
  | `18k-3p0b` + `18k-3p0b-r2` | ... 3B | 3 + 6 | r2 re-ran TB2.1, TB-Sci-lite, GDPval, CritPt, SciCode and HLE |
  | `18k-9p0b` | ... 9B | 9 | |
  | `sb-full` | Snowball + M5 SFT 10.73B | 9 | the pre-registered target of marin#11563 |
  | `sf-d512/d768/d1024-full-r2` | Snowflake SFTs | 9 each | the first `sf-*` runs were retired (server crash) |
  | `sb2-0p33b/1p0b/2p2b/4p8b` | Snowball + M5 SFT v2 (decontaminated) 0.33/1.0/2.19/4.8B | 6 each (lean) | 0p33b and 1p0b also started the 3 dropped suites; those runs were stopped and never used |
  | `marin-lr1` | Marin Snowball LongLoop R1 (comparison) | 6 (lean) | Datakit format; eval code `41eb04a9`/`d4f91de8`, see Code |

- **Leaderboard reference rows** used next to the ladder were scored on campaign c30 (#375; plan `cheap_full_v1`, the
  uncapped protocol below):
  - marin#11563's open-model table (DeepSeek 41.6, Qwen3.8 Xhigh 31.2, Super VL 30.6, Medium 26.9, Qwen3.6 19.2,
    Gemma 17.1, Lightning 14.1, Qwen3.5-4B 6.3) comes from the **c30 original runs** (no finish notice anywhere):
    `/data/aai/aa_evals/audits/c30_r2_20261113/out/table.json` (sha256 `297ccae7…`).
  - #51's c30 table (DeepSeek 44.1, …) is **c30n**: c30 with CritPt and GDPval re-run under the v3 notice
    (`out_c30n/table.json`, sha256 `8d66b75e…`).
  - The ladder figures' Lightning line (16.66) and marin#11563's "Qwen3.5-4B scores 8.2" use raw scores: c30-lightning and `m5l-qwen35-4b` respectively
    (`m5/prereg_aai_open/scores.json`).
- **Not v0:** `uval-sb10b-t06` (T=0.6 from `main`) and `ucab-sb10b-t10` (code-only arm from `main`), the #480
  validation runs.

### Items

Plan **`/data/aai/aa_evals/plans/c30/cheap_full_v1.json`**, sha256
`b3f0a1fc86cc7f16b2afcd1a38eebe24708ca01eae890336da48c1bc774ec943` (#181, 0.30 work row; list builders: 8 open
reference models + GLM). The items × repeats are identical in all 111 scored runs (checked from the run configs):

| Suite | AA v5.0 slot (weight) | Task set | Items × repeats | Selection | Retained / universe (c30 rescale) |
|---|---|---|---|---|---|
| TB2.1 | Terminal-Bench 4.0 (10) | bundle `tb21_6h` (89) | 84 × 3 | drop items no list-builder solved | 84 / 89 |
| TB-Sci-lite | Terminal-Bench-Science (15) | in-house pool v2.1 headline (135) | 32 × 2 | drop unsolved | 32 / 135 |
| AutomationBench | AutomationBench (5) | public set (600) | 446 × 1 | drop unsolved, then a random share | 553 / 600 |
| CritPt | CritPt (10) | all 70 | 70 × 3 | every item | 70 / 70 |
| SciCode | FrontierCode (15) | verified-v2 (64 problems) | 60 × 6 | drop unsolved | 60 / 64 |
| HLE | HLE-Diamond (10) | `text-only-v1` (728 of 1,000) | 157 × 2 | random subset | 728 / 728 |
| Omniscience | AA-Omniscience (10 acc. + 5 NHR) | public v1.1 (300) | 300 × 3 | every item | 300 / 300 |
| GDP.pdf | GDP.pdf (5) | `text-only-v1` (75 of 100) | 37 × 2 | drop unsolved, minus text-only exclusions | 37 / 75 |
| GDPval | AA-Briefcase (15) | `text-only-v1` (207) | 185 × 1 | random share | (Elo, no rescale) |

**Lean suite set** (`sb2-*`, `marin-lr1`; user 11-17): TB2.1, AutomationBench, GDPval, SciCode, Omniscience and GDP.pdf,
the same items. CritPt, HLE and TB-Sci-lite were dropped: they stayed flat from step-18000 1B to the 10.73B SFT and cost
34% of its eval GPU-h. Their slots are handled in the aggregate (below).

### Protocol

v0 has two modes: the **capped protocol** for SFT evals and their baselines (every `m5l` run) and the **uncapped
leaderboard protocol** (the c30 reference rows).

| | Capped (`m5l`) | Uncapped (c30) |
|---|---|---|
| Agentic harness | Pi 1.2.0, agent `pi-cont16k`, harness label `pi-1.2.0-pi180`, tools bash/read/write/edit, no compaction, 16,384 max tokens per response with continuation, bash default timeout 180 s, no agent internet, text-only model input | same |
| Context window | 131,072 (Pi and single-turn) | 131,072 |
| Finish notice (Pi) | v4 sticky, `submit-at-context-limit-v4-sticky-notice`: warn at 40,000 tokens left, reserve 16,384, final at 24,576, margin 256, final slack 2,048, text sha256 `e8147af470746c79df7a707ff1ed7dc01cbde38d13f9e31a2c2dfe44e067fd15` (all 60 scored Pi runs) | c30 main runs: none; c30n CritPt/GDPval: v3 |
| Response budgets | `capped-sft-eval-v1`: `/data/aai/m5/ladder/eval/capped_protocol.json`, sha256 `403ad91eb3582e718556a9d0070be42044971aaf6857e10dbff2c9645f399f96` (byte-identical copy: `aa_evals/campaign/capped_protocol_v1.json`). Warn = Super's c30 test p95; allowance = measured finishing allowance; counted in Super tokens for every model; whichever of the budget, the v4 context notice (Pi) or the near-window warning (single-turn) triggers first binds; an open tool call at the warning or at the v4 clamp is completed | none |
| Near-window warning | single-turn suites: warn at min(warn, window − prompt − allowance) | none |
| Time caps | Pi environment clock 7,200 s (model time excluded; protocol agent timeout 21,600). Model-time caps (Pi wall clock, SciCode chain cap, HLE/GDP.pdf request caps, HLE/Omniscience request timeouts) 43,200 s, a hang backstop (#455-#457, user 11-16). Exception: the first-wave runs (Super, Qwen3.5-4B, the 1B/3B first runs) ran at 7,200 s. Suites where that cap ended any attempt were re-run in full as `-r2`. Super/Qwen were kept; their cap hits bound the change at ≤0.8 / 0.3 / 0.05 pp. SciCode per-request timeout 14,400 s | 7,200 s everywhere (#344) |
| Infra and retries | same-seed retries of infrastructure failures, up to 6 per attempt (`eval_q35/common.py INFRA_RETRIES`). Infra attempts are not scored. Infra includes: verifier ran but produced no reward (`verifier_no_reward`, which also covers sandbox-start failures, e.g. "pod user namespace did not appear"); `verifier_bootstrap_timeout`; `stale_claim`; `worker_stopped`. The tail rule cut is not infra: the attempt is scored by its verifier. TB2.1 is scored resolved-only | same |
| Concurrency | one dispatcher per model, fixed budget 128 agents per GPU (`--no-auto-budget` from 11-16; the 3B ran with the auto budget until fixed at 256 on 2 GPUs), longest-predicted-first claims (`durations_v2.json` sha256 `855a0bbf…`) | c30's own launcher (#344 dispatcher) |
| TB2.1 environment | bundle `bundle-aptfix.tar` sha256 `9b0b4dbf5b1ac92cb0469d2eecf91a402afe1b1ca34f0f77905a28f4c7781ddc` (M3.6 apt repair). Pi runtime cache v6 (`d9bcbf42…`; v5 `d7fcf56b…` for TB-Sci-lite, GDPval, CritPt, AutomationBench). memcap 1.5 (TB2.1, TB-Sci-lite) / 1 (others). No disk cap. **Online verifier bootstrap** | same |

Response budgets of `capped-sft-eval-v1` (Super-native tokens; warn + allowance = total):

| Suite | Kind | Warn | Allowance | Total | Notes |
|---|---|---:|---:|---:|---|
| AutomationBench | Pi | 36,623 | 7,000 | 43,623 | |
| GDPval | Pi | 75,609 | 17,000 | 92,609 | |
| CritPt | Pi | 124,142 | 23,000 | 147,142 | |
| TB2.1 | Pi | 111,905 | 17,000 | 128,905 | |
| TB-Sci-lite | Pi | 112,698 | 23,000 | 135,698 | |
| GDP.pdf | single | 26,610 | 5,000 | 31,610 | near-window |
| Omniscience | single | 20,192 | 10,000 | 30,192 | near-window; notice sentence "If you are not confident in an answer, say that you do not know: that is a valid final answer, and it is better than a wrong one." |
| HLE | single | 130,850 | 32,000 | 162,850 | near-window (~98k effective) |
| SciCode | single, per step | 25,775 | 7,000 | 32,775 | near-window |

Known protocol quirks of v0, as run:
- **HLE near-window on Qwen3.5-4B.** Its server has max_model_len equal to the window and clamps requests, so its HLE
  ran without the near-window rule. At most 4 of 314 items could be affected (its 4 length stops). #452/#454
  (`fa25f746`/`cbea1c17`) fixed the room fitting for the Snowball servers before any scored SFT HLE run.
- **Super's GDP.pdf at T 0.6.** It ran at T 0.6 / top_p 0.95: the GDP.pdf client's built-in preset for the Super
  served name (AA's reasoning default). Every other request in v0 is T 1.0 (next section). The preset is unchanged in
  v0.1.

### Sampling (per model family, from the run configs)

| Family (spec) | Pi suites | Single-turn | Server generation config |
|---|---|---|---|
| Snowball / Snowflake SFTs (`snowball-eval`, `snowflake-eval`) | T 1.0, top_p 0.95, effort high, `clear_thinking: false`, pi_thinking max | same | `{"temperature":1.0,"top_p":0.95}` |
| Super VL (`nemotron35-super-mm-text`) | T 1.0, top_p 0.95, `enable_thinking`, `medium_effort: false`, `truncate_history_thinking: false`, pi_thinking high | same (GDP.pdf: T 0.6, see above) | `{"temperature":1.0,"top_p":0.95}` |
| Qwen3.5-4B (`qwen35-4b-eval-text`) | T 1.0, top_p 0.95, `enable_thinking`, pi_thinking high | same | T 1.0, top_p 0.95, top_k 20, min_p 0, presence_penalty 1.5 |
| Marin Datakit (`datakit-eval`) | T 1.0, top_p 0.95, effort high, `enable_thinking` | same | `{"temperature":1.0,"top_p":0.95}` |

SFT evals keep T 1.0 / top_p 0.95 (user, 11-19; #481 comment 7017537906): T=0.6 on the 10.73B SFT gave GDPval −68 Elo [−121, −15], the rest within noise.

### Graders and judges

- **Judge**: Qwen3.8-27B, arm `qwen38-27b-medium-pin`, served as `aa-judge-qwen38-27b` (teacher
  `qwen38-27b-eval-fast`, 1×B200, 262k), for HLE, Omniscience, GDP.pdf and the GDPval pairwise comparisons (all 37
  graded single-turn runs record this arm).
- **HLE**: `cais/hle-diamond` rev `04eeb7ef…` (sha256 `2b342190…`), text-only-v1. The official HLE judge prompt with a
  strict JSON schema (ExtractedAnswer). Score = accuracy.
- **Omniscience**: `ArtificialAnalysis/AA-Omniscience-Public-v1.1` rev `1f9cbcec…` (sha256 `f4fbdd1d…`). The AA answer
  prompt and grader template, verbatim. An empty answer is NOT_ATTEMPTED without a judge call. Accuracy = C/n;
  non-hallucination rate = 1 − I/(I+P+A) (`aa_evals/omniscience/metrics.py`).
- **GDP.pdf**: `tolo6474/GDP.pdf` rev `90feac14…` (parquet sha256 `2ba18b4f…`). LiteParse text alone (vision off), one
  turn, a judge call per criterion. Headline = all-pass (a task passes when every criterion passes); mean pass is
  reported.
- **GDPval**: each attempt is judged once against each of two fixed reference runs, the M3.6 deliverables of Qwen3.8
  Xhigh (`m36-qwen38-xhigh-gdpval-full`, 1426.7) and Lightning (`m36-lightning-gdpval-full`, 731.5). One order per
  pair (seed 731). The pool is `runs/gdpval/_pools/<campaign>-refs-<model>`, plus the runs' vs-expert matches.
  Bradley–Terry MLE with the human expert pinned at 968 (`aa_evals/gdpval/elo.py`, #183) and a 300-draw task
  bootstrap. In the aggregate, the Elo goes through AA's clamp (below).
- **CritPt**: a proxy, agreement with the frozen DeepSeek V4.1 Flash Max reference answers (reference `ds41-max`,
  runs `ref-ds41-max`, created 2026-11-05). Metric `critpt_v2_all70_agreement@ds41-max` on all 70 problems; no
  comparable answer, or a cutoff, counts 0.
- **SciCode**: dataset `verified-aa-v3`: base verified-v2 (64 problems, 287 scored steps, `flyingwagner/scicode-verified`
  0b40bd4) plus patch v3 (patch list sha256 `405d98d4…`). Background off. Template `background_comment_template.txt`
  (sha `9b407bdd44937f13`). Graded in two SIFs (np126 sha256 `f9f3834c…`, np22 `b0545593…`); a step passes if either
  passes; 1,800 s timeout, 16 GiB. Main-problem accuracy (headline) = share of (problem, repeat) with every step
  passing. Sub-step accuracy = mean over step rows.
- **TB2.1 / TB-Sci-lite**: the task's own verifier (TB2.1: bundle-aptfix tests, online `apt`/`uv` bootstrap).
  `score_resolved_only` (attempts still pending grading are left out; the 1-attempt bound is reported per run).
  TB-Sci-lite: pool v2.1, `runtime cache v5`.
- **AutomationBench**: the public set's own checker, graded by the collector.

### Aggregate

`aggregate = Σ_slot weight/100 × slot value`, raw % per slot. The one transform is AA's Elo clamp for GDPval in the
Briefcase slot, `100 × clamp((Elo − 500) / 2000, 0, 1)`. Slots: TB2.1 10, TB-Sci-lite 15, AutomationBench 5,
Omniscience accuracy 10, Omniscience NHR 5, CritPt 10, HLE 10, GDP.pdf all-pass 5, SciCode main 15, GDPval 15.

Two conventions were in use in v0:
- **Raw**: the M5 ladder and every SFT number on marin#11563 (pre-registration, links, post-mortem). The slot value is
  the run's score on the plan's items.
- **Universe**: the c30 leaderboard rows, including marin#11563's open-model table. The slot value is the score ×
  retained / universe (table above): items no list-builder solved count 0.

The same model differs a lot between the two: DeepSeek c30 is 41.63 universe vs 48.24 raw; Qwen3.5-4B is 6.3 vs 6.7
(c30) / 8.15 (m5l, capped).

Missing slots (lean evals):
- **65-point score**: the 7 slots the lean evals ran, out of 65.
- **Full index**: TB-Sci-lite, CritPt and HLE take `m5l-sb-full`'s values (1.56, 1.43, 4.46) (user 11-18).

Code: `aa-evals-coord/status/m36_aggregate.py` (sha256 `c7bece87…`), `/data/aai/m5/ladder/eval/scores_sb2/sb2_score.py`
(sha256 `aae46833…`), `m5/prereg_aai_open/scores.py`, `ladder65_figure.py`, `ladder_index_figure.py`. On `main`,
`aa_evals.common.aai_open.aggregate` and `python -m aa_evals.campaign.score` reproduce them from the stored runs.

| Model | Score | Recorded value |
|---|---|---|
| m5l-sb-full (raw) | 16.05 | 16.05 |
| sb2-0p33b / 1p0b / 2p2b / 4p8b, 65-point | 11.76 / 13.48 / 13.66 / 15.21 | same |
| sb2-*, full index (filled) | 12.58 / 14.30 / 14.48 / 16.04 | 12.6 / 14.3 / 14.5 / 16.0 |
| Super / Qwen3.5-4B (m5l) | 33.84 / 8.15 | same |
| marin-lr1, 65-point | 4.89 (`sb2_score.py`); `score.py` gives 4.09 of 55: its TB2.1 run dir has no `summary.json` | 4.89 |
| c30 universe (marin#11563 table) | 41.63 / 31.17 / 30.62 / 26.88 / 19.25 / 17.09 / 14.08 / 6.35 | same |
| c30n universe (#51) | 44.07 / 32.10 / … | same |

### Serving numerics (per model family)

The per-job record is `<serving registry>/numerics.<job>.0.json`: argv, env, weights, and `numerics_sha256`, which
includes the checkpoint files. All jobs are listed per run in `v0_runs.json`.

| Family | Spec | vLLM | Weights / KV | Serving code | Jobs (numerics sha256) |
|---|---|---|---|---|---|
| Super VL | `nemotron35-super-mm-text` | 0.32.0rc1.dev190 (vllm032) | NVFP4 (modelopt mixed) | 89d85472 | 17951506 (`0e01ff37`) |
| Qwen3.5-4B | `qwen35-4b-eval-text` | 0.32.0rc1.dev190 | bf16 | 89d85472 | 17951507 (`d3372c60`) |
| Snowball step-18000 SFTs, 10.73B | `snowball-eval` | 0.20.1 + GrugMoe plugin | block-FP8 weights, FP8 KV, seeded sampler | 89d85472 | 1B 17951682/17956700 (`e13294e2`), 3B 17955352/17956699 (`3d18012a`), 9B 17956767 (`b0d3d804`), 10.73B 17957225 (`ca5796e0`) |
| Snowball v2 SFTs | `snowball-eval` | 0.20.1 | same | 2381a72b (= 89d85472 + #458-#460) | 0p33b 17965373 (`a48f54a5`), 1p0b 17966383 (`b5fbaa45`), 2p2b 17968043 → 17968577 (`3a55eea6`, same), 4p8b 17968459 (`b40e33c7`) |
| Snowflake SFTs | `snowflake-eval` | 0.20.1, Triton fused MoE | bf16 weights and KV | 79a92cfd for the first job; 2381a72b at 3 engines per GPU after 15:25 11-16 | d512 17958479 → 17958758, d768 17958447 → 17958604, d1024 17958505 → 17958759 |
| Marin Datakit | `datakit-eval` | 0.20.1, fp32 router bias | FP8 | 35191a4d (local) | 17968886 (`8580797e`) |
| Judge | `qwen38-27b-eval-fast` (`aa-judge-qwen38-27b`) | vllm032 | | aa-serving at the time | shared server |

### Code

- **Eval (aa-evals)**: `e3dc54f90054e41bb853bad4c023428f88dc3653` (#457) for 79 of the 111 scored runs, including
  every run of `sb-full`, `18k-9p0b`, the r2 re-runs, the Snowflakes and `sb2-*`. **Tag `aai-open-v0-evals`.** Earlier
  commits on the same linear history (`dc603b65` → `fa25f746` #452 → `38427c2a` #453 → `cbea1c17` #454 → `3c161ac3` #455 →
  `b67f2aaf` #456 → `e3dc54f9`) ran the first wave:
  - `dc603b65`: 23 runs. Super and Qwen3.5-4B (all 9 suites each) and the 1B's first-run TB-Sci-lite, GDPval,
    AutomationBench, Omniscience and GDP.pdf. Qwen3.5-4B's HLE was graded at `38427c2a`.
  - `cbea1c17`: 3 runs, the 3B's first-run AutomationBench, Omniscience and GDP.pdf.
  - `41eb04a9` + `d4f91de8` (local branch `m5l-marin-eval-harness`, on top of `e3dc54f9`): `marin-lr1`, 6 runs.
    These add Datakit-format parsing on the capped raw-text paths and a collector that records an overwritten Pi
    session line instead of crashing. **Tag `aai-open-v0-evals-marin`** (`d4f91de8`). Both are ported to `main` by #523.

  The differences between these commits are BAD_NODES (#453), the HLE near-window room fitting (#452/#454; the
  affected 1B HLE was re-run) and the model-time caps (#455-#457; first-wave suites where the cap bound were re-run as
  r2).
- **Serving (aa-serving)**: `89d85472b6b9ae01a9687313735d7b710a669f79` for Super, Qwen3.5-4B, the step-18000 ladder and
  `sb-full`. **Tag `aai-open-v0-serving`.** Later commits on the same history:
  - `79a92cfd`: #458 `snowflake-eval` and #459 Triton MoE.
  - `2381a72b`: #460 engines_per_gpu; used by `sb2-*` and the 3-engine Snowflake servers.
  - `35191a4d` (local): `datakit-eval`. **Tag `aai-open-v0-serving-marin`.**
- Item plan, capped protocol and bundles: pinned by sha256 above. Run manifest: `docs/aai_open/v0_runs.json`
  (regenerate with `python docs/aai_open/collect_runs.py m5l`).

---

## v0.1: the benchmark on `main`

**Code: tag `aai-open-v0.1`** (the `main` commit that contains this file; aa-evals and serving are one tree since
#480). Pins in code: `aa_evals/common/aai_open.py`. The campaign launcher labels every suite and records the label;
a deviation from the pins below makes a run `v0.1+custom`.

How to run it: `python -m aa_evals.campaign.launch CONFIG ...` with a config copied from
`aa_evals/campaign/example_c30.json` (the uncapped leaderboard protocol), plus `"capped":
"aa_evals/campaign/capped_protocol_v1.json"` for SFT evals (the capped protocol). Keep the default sampling.

### What v0.1 keeps from v0

Unchanged from v0: the items (same plan and sha), the response budgets (same `capped-sft-eval-v1` file), the finish
notice (v4, same text sha), the near-window warning, the Pi harness and its flags, the context window, the sampling
(T 1.0 / top_p 0.95 from the specs' generation config, #562), the judges and graders, the GDPval references and the
Elo clamp, the aggregate (both conventions, the same code paths), and the serving specs of every family except
Super VL and DeepSeek.

### Pins checked by the launcher (`version()`) and clients

| Pin | v0.1 value |
|---|---|
| Plan | `cheap_full_v1.json` sha256 `b3f0a1fc…` |
| `suites.context_window` / `cap_sec` | 131,072 / 7,200 |
| Model-time caps (`suites.wall_cap_sec`) | uncapped: 7,200 (#344); capped: 43,200 (#539 default) |
| TB2.1 bundle / offline verifier toolchain | `bundle-aptfix.tar` `9b0b4dbf…` / `["tb21_6h", 9b0b4dbf…]` (#543) |
| Pi runtime caches | v6 `d9bcbf42…` (TB2.1), v5 `d7fcf56b…` (TB-Sci-lite) |
| SciCode dataset / TB-Sci-lite pool | `verified-aa-v3`, background off / `v2.1` |
| Capped protocol (when on) | sha256 `403ad91e…` (`capped_protocol_v1.json`) |
| Sampling | no `request_overrides`, `--temperature`, `--top-p`, `--request-overrides` or `--request-preset` |
| Pi finish notice / input | v4 40000:16384, text sha `e8147af4…` / text only |
| Judge arm (campaign `grading.judge.arm`, client grade stages) / GDPval references (`grading.gdpval_refs`) | `qwen38-27b-medium-pin` / `m36-qwen38-xhigh-gdpval-full`, `m36-lightning-gdpval-full` |
| Hand-given request flags (single-turn) | none of `--extra-body`, `--chat-template-kwargs`, `--max-tokens`, `--preset`, `--no-endpoint-defaults`, GDP.pdf `--vision auto` |

Per-model settings that are part of a model's own row are not deviations: registry and spec, effort, checkpoint,
budget, `worker_profile` (recorded per run). A lean `suites` list is not a deviation either: the aggregate reports
the missing slots.

### Serving numerics

`tbshared/numerics_record.json` → `pipelines["aa-serving"].specs["<spec>"]` at the tag, guarded by
`tbshared/test_numerics.py`.

| Family | Spec | vLLM | Changed since v0 |
|---|---|---|---|
| Snowball / Snowflake SFTs | `snowball-eval`, `snowflake-eval` | 0.20.1 | no; CPU shape 4c/64G per GPU and omp 4 (#535/#547) are numerics-neutral |
| Snowball on vLLM main | `snowball-eval-main` | 0.34.0rc1.dev815+g8f3c672dc | new spec, used only when named (#550); `snowball-eval` stays the default |
| Marin Datakit | `datakit-eval` | 0.20.1 | ported from `35191a4d` (#523) |
| Super VL | `nemotron35-super-mm(-text)` | **0.34.0rc1.dev815** | **yes**: vLLM main + supersede 3.0 (#552) and eot-checkpoint 5.0 (#557) |
| DeepSeek V4.1 Flash 1017 | `deepseek-v41-flash-1017` | **0.34.0rc1.dev815** | **yes**: vLLM main, #9 sampling-bias fix (#528) |
| Qwen3.5-4B, Qwen3.6, Qwen3.8, Gemma 4, Lightning, GLM, judge | `*-eval-text`, `qwen38-27b-eval-fast` | 0.32.0rc1.dev190 | no |

### v0 → v0.1 changelog

All protocol-relevant PRs merged into `main` after `e3dc54f9` / `89d85472`. The full list of 64 eval/serving PRs and their classification is in the PR that adds
this file; PRs not listed here are scheduling, cost, ops, docs, tests, or another pipeline (eval_q35-only,
task-synthesis-only, SFT-only).

**Code-effect evidence.** Two checks compared `main` `e678b535` with v0's `e3dc54f9` (#480 comment 7015700314, val-codeab):
- **CPU replay.** The original sb-full model responses, replayed through both versions of the sandbox and harness:
  same tool results, first requests, rewards and tool time.
- **Code-only arm.** At T=1.0 on `main`, against the original: TB2.1 +2.1 pp [−3.3, +7.6], AutomationBench −1.9
  [−5.4, +1.5], GDPval +23 Elo [−32, +80].

That check covers #485, #488, #508, #510, #519 (in `e678b535`), but not the later PRs.

| PR | Change for a scored run | Can move scores? | Evidence |
|---|---|---|---|
| #485 | harbor_ns superset: /dev nodes are char devices (a `grep -r /` no longer reads /dev forever); new image builds retry proxy/apt failures and use archive.debian.org for old releases | yes, rarely (only commands that touched /dev files; only new builds) | oracle canary 8/8 in all arms; covered by the code-effect check |
| #488 | Pi proxy splits SSE on LF only (a reply with NEL/U+2028/U+2029 no longer loses an event) | yes, rarely | frequency not measured; covered by the code-effect check |
| #508 | image eviction at every trial start + ENOSPC recovery | fewer infra-lost attempts (up, through fewer exhausted retries) | A/B 110 images on 219 GB: 110/110 vs 74/110 builds; covered |
| #519 | per-sandbox disk cap on by default (`--diskcap auto`: TB2.1 per-task 32/40/90 GiB, else max(3× storage, 30 GiB); `ulimit -f unlimited` fails) | yes, only for tasks writing past the cap | oracle cap on/off TB2.1 85/89 vs 82/88, TB-Sci-lite 139/141 vs 140/141, 0 kills; val run 0 kills, largest file 3.0 GB; covered |
| #510 | AA: infra reason renamed `verifier_no_reward` → `userns_setup` (sandbox-start failures were already infra in v0); eval_q35: scored 0 → infra | no (AA) | v0 `pi_eval/worker.py` already classified them infra |
| #523 | Datakit parsing; Omniscience strips Datakit think tags from answers; an agent-overwritten Pi session line is recorded, not fatal to collection; GrugMoe router bias fp32 | Datakit models only; collection no longer crashes on overwritten session lines | Snowball/Super text paths unchanged (tests); routing unchanged on 53/54 checkpoints |
| #543 | TB2.1 offline verifier toolchain (`suites.tb_toolchain_bundle`, set in `example_c30.json`): 60 of 89 tasks verify offline | up: fewer `verifier_bootstrap_timeout` retries and tasks lost on slow-network days | paired oracle gate 88/88 conclusive verdicts identical; sb-full had 13 bootstrap timeouts; val-codeab: timeouts track the proxy download rate (20 vs 20 frozen vs new at 210/195 kB/s) |
| #548 | hung-session watchdog on by default: guard accounting failure, stalled request (3,660 s), idle Pi (600 s, tool time not idle) → infra + same-seed retry | which attempts are infra: a hang ends as infra instead of running to the wall cap and scoring | 16,104 attempts: 490/490 t2 hangs + 3 eval caught, 0 healthy attempts ended |
| #549 | capped protocol: response-budget guard retries /detokenize once; its own 502 goes straight to Pi (infra `response_budget_accounting`) instead of a 30-min retry loop | capped runs: a guard failure is infra instead of a hang (11-15 hang root cause) | real Pi probe ended in 16 s with a classified error |
| #528 | DeepSeek eval server on vLLM main, #9 sampling bias fixed | yes, DeepSeek rows (new servers) | #9 reproducer P(token2=0) 0.4174 → 0.5006; greedy/top-20 within each build's prefix-hit floor |
| #552 | Super eval specs on vLLM main + supersede 3.0 | yes, Super rows (numerics; new servers) | cross-build TV median 0.006 vs own cache-hit 0.003/0.001, argmax 16/16; T=1 MTP3 calibrated (KS p 0.65/0.18/0.96); +15.7% tok/s |
| #557 | Super eval specs: eot-checkpoint 5.0 (end-of-turn KV reuse) | yes, Super rows (numerics; new servers) | accepted: TV vs fresh 0.045 (eot) vs 0.067 (stock), Wilcoxon p 0.79, end states within the prefix-hit floor |
| #539 | capped protocol: model-time caps default 12 h (v0 set them in the campaign config) | no: same values as v0's later runs | uncapped commands byte-identical |
| #524 | launcher `request_overrides` (off; makes a run `+custom`) | opt-in only | unset: every command byte-identical |
| #553 | `worker_profile: "slow"` (TB2.1 workers 32 slots / 6 CPU; for slow-served models, recorded per run) | no measurable effect | A/B 252 attempts per arm: pass +2.4 pp [−1.6, +6.4], tool time +7% [−11%, +30%] |
| #550 | `snowball-eval-main` spec (opt-in by name) | only if used | BPB 0.279431 vs 0.279455; 2,714/2,714 identical prompt ids; parsers 0 mismatches |
| #534, #511/#522/#529/#533/#540, #538, #545 | fixed dispatcher budgets by default (as m5l); lazy Pi workers and dispatcher growth/release; claim packing; deterministic tied-retry order | no (scheduling); under the uncapped 2-h wall cap, server load can change how many attempts the cap ends | attempts and seeds unchanged (#538, #545 tests) |
| #562 | docs: keep T 1.0 / top_p 0.95 for SFT evals | no (the sampling pin is unchanged) | #480 comments 6996073899, 7015700314 |

Net expectation for a re-run at v0.1:
- **Snowball / Snowflake SFT rows.** Same serving numerics. Scores move only through the environment and infra
  rows above: fewer TB2.1 tasks lost to verifier bootstrap timeouts (#543), hangs retried instead of scored (#548/#549),
  and disk-cap or /dev effects measured at zero in the code-effect check.
- **Super and DeepSeek rows.** New server numerics as well (#552/#557, #528), validated within each build's own
  noise floor.
