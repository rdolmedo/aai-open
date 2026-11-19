# aai-open

Open evaluation harness for **AAI-open**, an open stand-in for the Artificial Analysis Intelligence Index
(Terminal-Bench 2.1, Terminal-Bench-Science lite, AutomationBench, AA-Omniscience, CritPt, HLE-Diamond, GDP.pdf,
GDPval, SciCode). It is the code behind the evaluations reported with our synthetic-data results; the data
generation code is in the companion repository `synth-data`.

Status: dry-run export, under review. Generated from private trunk commit `43d7f73b07a3` (see `CORE_VERSION`).

## What is here

| Path | What |
|---|---|
| `aa_evals/` | one package per benchmark (`aa_evals/<bench>/run.py`), graders and judges, the campaign launcher (`aa_evals/campaign/`), the Pi and single-turn runners (`aa_evals/common/pi_eval`, `aa_evals/common/harbor_mswea`) |
| `harbor_ns/` | rootless container sandbox for task environments (Linux namespaces; no Docker daemon needed) |
| `tb_agents/` | agent harness glue (Pi coding agent, terminus-2), response budgets and finish notices |
| `inference/`, `task_synthesis/serve/`, `synth/teachers/` | model serving launcher (vLLM) and the serving specs we evaluated with |
| `tbshared/`, `tbsched/` | shared utilities, the serving numerics record, the session scheduler |

`synth/`, `task_synthesis/` and `eval_q35/` are present only as far as the harness imports them (shared core).

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
