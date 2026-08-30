# OpenCausal

[![CABS: ds4cabs](https://img.shields.io/badge/CABS-ds4cabs-1f4b99?logo=github)](https://github.com/ds4cabs)
[![GitHub Pages: live](https://img.shields.io/badge/GitHub_Pages-live-brightgreen?logo=github)](https://ds4cabs.github.io/CausalSentinel/)
![CABS: 2026](https://img.shields.io/badge/CABS-2026-6f42c1)
![status: MVP in progress](https://img.shields.io/badge/status-MVP_in_progress-f1c40f)
![type: Dossier Generator](https://img.shields.io/badge/type-Dossier_Generator-1f6feb)
![domain: Causal Inference](https://img.shields.io/badge/domain-Causal_Inference-0aa)

**Author:** Shucheng (Bangli) Cao

**This is the standalone repository for OpenCausal** — the causal-evidence agent below, split out on
its own so it can be cited and developed independently. It is one of two components of
**CausalSentinel**, the CABS 2026 Data Science Summer Internship project: OpenCausal
(this repo, causal evidence for drug-target prioritization) and **OpenSentinel**
(drug-safety comparison, built by fellow intern Natalie Huang). The combined project,
with both components together, lives at
[github.com/ds4cabs/CausalSentinel](https://github.com/ds4cabs/CausalSentinel).

**Project Type:** Dossier Generator

## Overview
Give it a protein and a disease. It decides which public databases to query, queries them,
and writes one **target evidence card**: a short, sourced argument ending in a go / no-go,
where every number is traceable to the tool that returned it.

The hard part is not fetching from eight databases. It is making the output something a
reviewer can *falsify* — so the design puts the model where it can do least damage:

| | Written by |
|---|---|
| Evidence table, caveats, sources, provenance | **rendered mechanically from tool output** — the model never touches them |
| Verdict line + reasoning paragraph | the model — then **checked against tool output**, and the run fails if a claim has no source |

## Try it in ten seconds — no install, no key

- **[Browse the 991 pre-generated protein dossiers](https://ds4cabs.github.io/CausalSentinel/dossiers/)** —
  one page per protein: its published MR estimates (retrieved, never computed here),
  pQTL instruments, GWAS signal, and a phenome map. 101,543 protein-outcome estimates
  indexed in total. Pure static markdown, no language model anywhere in that path
  (also browsable [in-repo](dossiers/README.md)).
- **[Open the card viewer](https://ds4cabs.github.io/CausalSentinel/viewer/)** — ten worked
  protein × disease cards, one panel per tool, every value read from each card's own
  ledger; second tab shows genetics-to-clinic timelines.

## Run the web app — type your own pair (two minutes)

```bash
git clone https://github.com/ds4cabs/CausalSentinel.git
cd CausalSentinel
pip install -r requirements.txt
streamlit run app.py
```

Your browser opens a page: type any **protein** and **disease**, press *Build the
evidence card*, and ~20 seconds later the full card is on screen — built from live
queries to the nine public databases, all keyless and free. **No API key is needed**,
because the card is rendered by code from tool output; the model only ever writes a
one-line verdict and one paragraph. Paste your own free
[Gemini key](https://aistudio.google.com/apikey) in the sidebar (used for that run,
never stored) to add those two sentences — and watch the validator check them on the
page. A **Databases** picker in the sidebar lets you query only the sources you
need — skipped sources render honestly as “tool not called in this run”.

One app, three tabs — build your own card, browse the ten worked cards, and open
any of the 991 protein dossiers:

| ![A freshly built card](figs/app_02_card.png) | ![The two live figures under the card](figs/app_03_figures.png) |
|---|---|
| *a freshly built card — verdict, a code-written reading of the evidence, then one panel per tool* | *under every card: the forest of retrieved MR estimates and the gnomAD constraint figure, drawn live* |

| ![The ten worked cards](figs/app_04_viewer.png) | ![Genetics-to-clinic timelines](figs/app_05_timeline.png) |
|---|---|
| *the ten worked cards, same panel-per-tool engine* | *hand-verified genetics → clinic timelines — genetic evidence precedes the clinic's verdict by a decade* |

| ![The 991-protein gallery](figs/app_06_gallery.png) | ![Pick your databases](figs/app_07_pick_sources.png) |
|---|---|
| *the 991-protein gallery: filterable index, dossiers rendered in-app* | *pick your databases — query only what you need; skipped sources say so instead of pretending* |

The figures are drawn from tool output too — nothing on these axes is typed in by hand:

| ![IL6R retrieved MR estimates](figs/IL6R_mr_forest.png) | ![HMGCR population constraint](figs/HMGCR_constraint.png) |
|---|---|
| *retrieved MR estimates for IL6R — top 15 of 133 outcomes; the truncation is printed on the figure* | *where HMGCR sits on the knock-out-tolerance scale; the verdict comes from the tool, not the figure* |

## What it does and does not do

- **It retrieves published Mendelian randomization estimates** (EpiGraphDB pQTL resource,
  Zheng et al. *Nat Genet* 2020) — beta, se, p, method, cis/trans instrument, Steiger
  direction, colocalization probability where available.
- **It does not compute MR or colocalization.** No instrument selection, no harmonisation.
  Every estimate is labelled `computed_here: false`.
- **Coverage is partial and the card says so.** Proteins with no plasma pQTL instrument
  return "not available" — which means *no estimate*, not *no effect*.
- **Where no GWAS was ever run, no agent can help.** That data still has to be generated
  by a human. This is the line between a tool and the science.

## Core tools (9, all live)

| Tool | Source | Answers |
|---|---|---|
| `get_mr_result` | EpiGraphDB pQTL MR | is there a *published* causal estimate for this protein → disease? |
| `get_clinical_evidence` | Open Targets (ChEMBL + trial registries) | has the clinic already tried this target — which drugs, what stage, why did trials stop? |
| `get_target_disease_evidence` | Open Targets | how strongly is this target associated with this disease? |
| `get_uniprot_dossier` | UniProt | what is this protein and where does it act? |
| `get_chembl_modulators` | ChEMBL | is it already druggable, and by what? |
| `get_clinvar_variants` | ClinVar (NCBI) | are there clinically classified variants? |
| `get_gnomad_constraint` | gnomAD | is it LoF-intolerant — i.e. a *safety warning*? |
| `get_gwas_catalog` | GWAS Catalog | how much genetic signal maps to the locus? |
| `get_pharmgkb_drug_gene` | PharmGKB / ClinPGx | any pharmacogenomic relationships? |

Each tool reports its `source_release` (UniProt release, ClinVar build, ChEMBL version,
Open Targets data release, EpiGraphDB build), so a card is reproducible rather than merely
timestamped.

## What a card actually looks like

Not a description of one — the real thing, `cards/PNPLA3_MASLD_evidence_card.md`, trimmed:

```markdown
**Verdict:** GO — Strong genetic and literature association with MASLD supports its pursuit.

> **You asked about "MASLD". This card scored MONDO_0013209 — metabolic dysfunction-
> associated steatotic liver disease.** If those are not the same thing, every number
> below answers a different question.

| Evidence | Tool | Result |
|---|---|---|
| Causal effect (MR) — retrieved, not computed | `get_mr_result` | **not available** — no pQTL MR
  estimate for this protein in the resource (absence of an estimate is not evidence of no effect) |
| Clinical variants | `get_clinvar_variants` | 216 ClinVar records; 0 pathogenic in a sample of 30 |
| Population constraint / LoF tolerance | `get_gnomad_constraint` | pLI=1.6e-14, LOEUF=1.26 → LoF-tolerant |
| Extra genetic evidence | `get_gwas_catalog` | 114 unique SNPs from 256/256 association rows |
| Pharmacogenomics | `get_pharmgkb_drug_gene` | 2 clinical annotation(s) over 6 drug(s):
  asparaginase, cyclophosphamide, daunorubicin, ethanol +2 more — ClinPGx evidence level 3
  (scale 1A strongest to 4 weakest) — e.g. rs738409 (PNPLA3); ethanol; Alcoholism (level 3 Toxicity) |

## Caveats declared by the tools
- **`get_clinvar_variants`** — Pathogenic count is over the 30 record(s) retrieved, NOT over
  all 216 ClinVar records for this gene; it is a sample, not a rate.
```

Four things to notice, because they are the design:

1. **The substitution is shown on both sides.** You typed "MASLD"; the card says what it
   actually scored. You cannot audit a resolution you cannot see.
2. **"not available" is never "no effect."** An absent estimate is an absent estimate.
3. **Denominators travel with counts.** `0 pathogenic in a sample of 30` out of 216, and the
   caveat block says outright that this is a sample and not a rate.
4. **Every row names the tool that produced it.** The `.json` beside the card carries the full
   ledger — every call, its arguments, its verbatim return — so any card can be re-derived
   and diffed.

Older versions of this same card are frozen in [`cards/archive/`](cards/archive/), one folder
per version, each with a README saying what that version got wrong and where it was fixed.

## Tech Stack
Python, **`google-genai`** (Gemini SDK; the older `google-generativeai` is deprecated),
requests, python-dotenv.

## Getting Started — full CLI

**Prerequisites:** Python 3.10+. A free Gemini API key is needed only for `agent.py`
(the model's two sentences); put it in `../.env` as `GEMINI_API_KEY` (see
`.env.example` — the `.env` file lives one level up and is never committed). The
individual tools, the validator tests and the web app's no-key mode need no key at all.

```bash
# 1) create and activate an isolated environment
python -m venv .venv
.venv\Scripts\activate            # Windows  (macOS/Linux: source .venv/bin/activate)

# 2) install dependencies
pip install -r requirements.txt

# 3) test a single tool on its own (no Gemini key needed)
python tools\uniprot.py
python tools\mr.py                 # PCSK9, IL6R, PNPLA3 — including an honest "no estimate"

# 4) run the agent on one pair -> writes a card to cards/
python agent.py --protein PCSK9 --disease "high cholesterol"

# 5) or run the benchmark set (10 pairs chosen to exercise different branches)
python agent.py --batch pairs_benchmark.txt

# 6) validator regression tests (60 cases, no network, no key)
python test_validator.py
```

Output: `cards/PCSK9_high-cholesterol_evidence_card.md` (+ `.json`). The `.json` carries
the **full tool ledger** — every call, its arguments and its verbatim return — so any card
can be re-derived and diffed. *The JSON is the record; the markdown is the readable view.*

### Architecture

```
agent.py          orchestrates: wrap tools -> model calls them -> render -> validate
  ledger.py       captures every tool call's arguments and verbatim return value
  render.py       builds table + caveats + sources + provenance FROM the ledger
  validate_card.py  fails the run if the model's prose outruns the ledger
  tools/*.py      one database wrapper each; one public function returning a dict
  system_prompt.md  the agent's rules
```

Gemini's automatic function calling normally executes tools *inside* the SDK, so the caller
never sees what came back and the card is whatever the model remembers. `ledger.py` wraps
each tool so the return values survive — which is what makes deterministic rendering and
validation possible at all.

### Proteome sweep — the whole searchable universe, no LLM, no key

`proteome_sweep.py` turns the same tool layer into a lookup resource: one **MR-feasibility
dossier per protein**, generated mechanically (no language model anywhere in this path).

```bash
python proteome_sweep.py --pilot        # 8 proteins covering all three tiers
python proteome_sweep.py --all         # all 991 proteins (985 Tier A)
```

The output ships in this repo: **[browse the dossier index](dossiers/README.md)**.

Each dossier answers, in order: **(1)** which published MR estimates exist (retrieved);
**(2)** whether pQTL instruments exist even where no MR was run — *Tier B: the un-run
analyses*; **(3)** the actual GWAS Catalog results at the locus (trait, best p, lead SNP,
study); **(4)** a phenome map of genetically-associated diseases, each overlaid with its
MR status — rows with genetic signal and no MR estimate are labelled `candidate analysis`,
which is the research-opportunity / comorbidity-hypothesis space; **(5)** druggability and
safety annotation. `dossiers/master_index.csv` is the cross-check table over all proteins.

| Tier | Meaning |
|---|---|
| A | published pQTL-MR estimates exist (Zheng et al. 2020, via EpiGraphDB) — shown |
| B | a pQTL GWAS exists but no MR estimate here — instruments derivable, analysis un-run |
| C | no plasma pQTL found — gene-level genetic evidence only, as an honest preview |

### What the validator catches

Fabricated numbers (compared **numerically**, tolerant of honest rounding and of "over N" /
"nearly N" bounds), fabricated rsIDs and accessions, unsupported qualitative claims
("FDA-approved", "monoclonal antibody", "small-molecule"), **causal language when the MR
tool returned no estimate**, and any claim that this agent performed MR itself.

On the 10-pair benchmark it currently rejects 2 cards — each for a real defect, not a false
alarm. Run `test_validator.py` after any change to it.

## Notes
This project is the cohort's causal evidence reference implementation with strong
variant-level rigor. Built in rounds (each ships); Round 1 = a 3-tool card end to end.

**License:** [MIT](LICENSE) — Copyright (c) 2026 Chinese American Biopharmaceutical Society (CABS) / ds4cabs.

## Found a problem?

That is exactly the feedback this project wants — a card that reads wrong, a dossier
number that does not match its source, anything hard to use.
**[Open an issue](https://github.com/ds4cabs/CausalSentinel/issues)** — a one-line
complaint is enough.
