# aai-open

| AAI-open benchmark | Weight | Stands in for (AA Intelligence Index v5.0) | Why | How it differs from AA's version |
|---|---:|---|---|---|
| Terminal-Bench 2.1 | 10 | Terminal-Bench 4.0 | TB 4.0 is near zero for open models (Qwen3.8 27B: 5.6 on TB 4.0 vs 79.8 on TB 2.1), so it gives no signal | 84 of the 89 tasks (tasks no reference model solved are dropped), 3 repeats; Pi agent harness instead of Terminus 2 |
| Terminal-Bench-Science lite | 15 | Terminal-Bench-Science | TB-Science 0.1 is also near zero for open models (Qwen3.8 27B Xhigh: 0.5) | an in-house pool of the upstream tasks labelled `lite` (well built, parked as too easy for frontier models): 32 of 135 tasks, 2 repeats; AA has no number for it |
| AutomationBench | 5 | AutomationBench | the same benchmark | the public set, 446 of 600 tasks (unsolved tasks dropped, then a random share), 1 repeat; Pi agent harness |
| AA-Omniscience | 10 + 5 | AA-Omniscience (accuracy 10, non-hallucination rate 5) | the same benchmark | the 300 public v1.1 questions instead of all 3,000 (2,700 are private; AA reports a −0.2 pt public−private gap), 3 repeats; self-hosted judge (Qwen3.8-27B) |
| CritPt | 10 | CritPt | the answers are hidden; only the authors' server grades them | scored as agreement with frozen reference answers from DeepSeek V4.1 Flash (Max) on all 70 problems, 3 repeats (a proxy, not the official grade); Pi agent harness instead of Stirrup |
| HLE-Diamond (text) | 10 | HLE-Diamond | the same benchmark | the 728 text-only questions instead of all 1,000 (272 have images): a random 157, 2 repeats; self-hosted judge (Qwen3.8-27B) instead of GPT-5.6 Luna |
| GDP.pdf (text) | 5 | GDP.pdf | the same benchmark | document text only (no page images), so 75 of the 100 tasks qualify: 37 of them (unsolved dropped), 2 repeats; self-hosted judge |
| SciCode-Verified | 15 | FrontierCode 2.0 | FrontierCode 2.0 is private | SciCode-Verified v2 plus our fix pass (patch list v3): 60 of 64 problems, 6 repeats, background off; main-problem accuracy |
| GDPval-AA (text) | 15 | AA-Briefcase | Briefcase is private; AA ranks models on GDPval-AA as on Briefcase (ρ = 0.97 over 137 models) | the public GDPval gold tasks, text-only subset (185 of 207), 1 repeat; deliverables judged pairwise against two fixed reference models by a self-hosted judge (Qwen3.8-27B), Bradley–Terry Elo with the human expert at 968, mapped to the slot with AA's clamp |

Weights sum to 100; the aggregate is the weighted mean of the raw slot scores. Common to all benchmarks: a
131,072-token context for every model, the agentic suites (Terminal-Bench 2.1, TB-Science lite, AutomationBench,
CritPt, GDPval) run in the Pi coding-agent harness without compaction, the item subsets (about 30% of the full work)
were chosen on open reference models, and judges are self-hosted open models. Exact pins: `docs/aai_open/VERSIONS.md`.

Open evaluation harness for **AAI-open**, an open stand-in for the Artificial Analysis Intelligence Index v5.0.

## Running

Every benchmark has one entry point:

    python -m aa_evals.<bench>.run --model url:<base_url>@<served_name> --run-id <id> [--limit N] [--repeats K]

`url:` points at any OpenAI-compatible endpoint (for example a local `vllm serve`). Each benchmark's README lists
its data download, protocol and deviations from AA's methodology.

**Cluster code.** The job runners submit to HTCondor with Apptainer images, as on the cluster where these results
were produced; that path is kept for reproducibility and documented as internal. Single-turn benchmarks run locally
against the `url:` endpoint. Internal default paths were rewritten at export to neutral ones (`/data/aai` for data
and runs, `/opt/aai` for code checkouts); set the documented environment variables (e.g. `TASKSYNTH_SERVING_ROOT`)
or create those directories. A pluggable local executor is planned.

## Versions

AAI-open is versioned: a version pins item sets, protocol, graders, weights and the code commit. v0 is the protocol
of our Snowball scaling evaluations; v0.1 is the current code. See `docs/aai_open/VERSIONS.md`; every run records
its `aai_open_version` (`aa_evals/common/aai_open.py`).

## Evaluating an SFT model

`aa_evals/campaign/SFT_EVAL.md` is the runbook we use (with `example_sft_snowball.json` and the expected throughput
profile in `aa_evals/campaign/expected/`); it describes our cluster setup.

## Data and benchmarks

No benchmark items are included. Download scripts fetch each benchmark from its official source under its own
license (HLE questions, for example, may not be redistributed).

## License

MIT (`LICENSE`), except third-party components listed in `NOTICE`, which keep their own licenses.
