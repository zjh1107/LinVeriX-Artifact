# LinVeriX experiment artifact

This package contains the benchmark and scheduler source, formal run protocol,
the six formal result sets, and scripts for reproducing analyses and figures.


## Experimental protocol

Unless an RQ specifies otherwise, runs use 300 two-second windows, 10 clean
warm-up windows, seeds 113--117, 256 segments, eight query threads, budget
$B=2$, and Top-$K$ $K=6$. Each dataset and seed uses the same read-only query
input across policies. Heterogeneous segment penalties are injected into the
foreground query path and reduced by maintenance. The reported residual is
the remaining penalty; P99 is measured independently from foreground queries.

| RQ | Experiment |
| --- | --- |
| RQ1 | LinVeriX, Greedy, Threshold, and Periodic on Books, Facebook, and Wiki using AIDEL. |
| RQ2 | LinVeriX residual and measured P99 trajectories under continuous and spaced six-segment recovery pulses on Books. |
| RQ3 | Books ablations: Full, No Avoidance, No Phase 2, and No Top-K. |
| RQ4 | Four proof-sampling rates, injected modification/omission/replay faults, and per-window budget reconciliation. |
| RQ5 | Books budget sweep $B\in\{1,2,4,6\}$ with $K=6$. |
| RQ6 | LinVeriX, Greedy, Threshold, and Periodic within the PGM backend on Books. |

## Directory map

| Path | Contents |
| --- | --- |
| `source/` | AIDEL benchmark, PGM adapter, controller, audit code, build files, and run scripts. |
| `formal_results/RQ1_AIDEL/` | Three-dataset four-policy aggregates, seed summaries, manifests, and raw window CSVs. |
| `formal_results/RQ2_Recovery/` | LinVeriX recovery traces, event records, and trajectory summaries. |
| `formal_results/RQ3_Ablation/` | Four Books ablation runs and per-seed records. |
| `formal_results/RQ4_Audit/` | Proof-sampling, injected-fault, and budget-reconciliation records. |
| `formal_results/RQ5_Budget/` | Books budget-sweep results and per-seed records. |
| `formal_results/RQ6_PGM/` | PGM Books comparison aggregates, seed summaries, manifests, and raw logs. |
| `analysis/make_figures.py` | Generates vector figures from recorded results; RQ1 and RQ6 show P99, P99-degradation AUC, and Top-6 residual. |
| `data_manifest/` | SHA-256 digests and portable metadata for prepared key samples; original SOSD datasets are not redistributed. |

RQ3 and RQ5 aggregate files use neutral `aggregate.csv` and `per_seed.csv`
names. Their manifests and raw logs record the formal 300-window, five-seed
runs.

## Reproduction

The benchmark runs under Ubuntu 24.04 with a C++17 compiler, Python 3, OpenSSL
development headers, and SOSD uint64 Books, Facebook, and Wiki source files.
From `source/`, build the general AIDEL benchmark and the two formal
experiment executables:

The optional document-ID workload reads `data/document-id.sorted.10M` relative
to `source/`; set `DOCUMENT_ID_DATA` to use a different input path.

```bash
bash build.sh
bash scripts/build_rq1_protocol.sh
bash scripts/build_pgm_rq1_protocol.sh
```

Prepare deterministic one-million-key samples for each dataset:

```bash
bash scripts/prepare_sosd_samples.sh /path/to/books_50M_uint64 books data/sosd_samples/books 1000000 113 114 115 116 117
bash scripts/prepare_sosd_samples.sh /path/to/facebook_200M_uint64 facebook data/sosd_samples/facebook 1000000 113 114 115 116 117
bash scripts/prepare_sosd_samples.sh /path/to/wiki_ts_200M_uint64 wiki data/sosd_samples/wiki 1000000 113 114 115 116 117
```

Check the prepared sample hashes against `data_manifest/`. Then run each
experiment from `source/`:

```bash
bash scripts/run_rq1_formal.sh
python3 scripts/run_rq2_original_design.py --windows 300 --warmup 10 --seeds 113 114 115 116 117 --allow-rebuilt-binary --out runs/rq2_formal
python3 scripts/run_followup_rqs.py rq3 --windows 300 --warmup 10 --seeds 113 114 115 116 117 --out runs/rq3_formal
python3 scripts/run_followup_rqs.py rq4 --windows 300 --warmup 10 --seeds 113 114 115 116 117 --out runs/rq4_formal
python3 scripts/run_followup_rqs.py rq5 --windows 300 --warmup 10 --seeds 113 114 115 116 117 --out runs/rq5_formal
python3 scripts/run_followup_rqs.py rq6 --windows 300 --warmup 10 --seeds 113 114 115 116 117 --out runs/rq6_pgm_formal
```

Each complete run is 600 seconds: 20 seconds of warm-up and 580 seconds of
evaluation. The complete matrix takes many hours. Keep each run directory
separate and retain the generated manifests; source or configuration hashes
identify which executable produced each record. If a sample is missing, set
`BOOKS_SOURCE` to the local SOSD Books file before running the RQ2--RQ6 scripts.

## Figures and audit

Install `reportlab` in a Python environment and run these commands from the
artifact root. Figures are written to `generated_figures/` by default, keeping
experiment outputs separate from the manuscript source.

```bash
python3 analysis/make_figures.py --out-dir generated_figures
python3 source/scripts/plot_rq2_recovery_publication.py --run-dir formal_results/RQ2_Recovery --out-dir generated_figures
python3 analysis/audit_budget.py
```

The RQ2 plot uses the archived event-aligned and per-window summary CSVs. The
budget audit checks count-budget, phase-budget, executed-action, and
used-versus-executed agreement. The audit implementation uses a 64-bit FNV-1a
combiner; the reported RQ4 detection rates characterize the tested implementation
and injected faults.

The manuscript package contains the EDBT A4 LaTeX source, template files, and
the figure copies used by the paper. Its README documents compilation and
submission checks.
