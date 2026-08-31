# OpenCausal — MVP Version (Gemini AI Agent · Dossier Generator) · CausalSentinel Team Tech Lead

> **This is the original pre-build proposal (July 2026). It describes what was planned, not
> what shipped — the R/DuckDB MR pre-computation and colocalization were deliberately not
> built. For what the system actually does, see [README.md](README.md).**

**Intern:** Shucheng (Bangli) Cao
**Level:** 5th-year PhD candidate (McGill, Quantitative Life Sciences), Advanced Python/R
**Timeline:** 2–3 weeks (~55 hours)
**Paradigm:** **Dossier Generator** — CLI tool that takes a protein + disease, runs an autonomous Gemini workflow over the deepest causal-genetics + safety stack in the cohort.
**Database count:** **8** (the maximum in the cohort, reflecting Shucheng's role as the strongest methodologist and tech lead of the CausalSentinel team. His project is the cohort's reference for variant-level rigor).

---

## The Agent

**What the agent does (autonomous workflow):** Given a protein + disease, the agent autonomously retrieves pre-computed MR results, runs colocalization context lookup, queries UniProt + Open Targets for protein and target-disease context, finds known modulators in ChEMBL, checks ClinVar for clinically significant variants, queries gnomAD for population frequency + LOF tolerance, cross-references the GWAS Catalog for additional genetic evidence, and pulls PharmGKB drug-gene interactions — writing a comprehensive causal evidence card.

**Input:** Protein + disease (CLI: `python agent.py --protein PNPLA3 --disease MASLD`).
**Output:** `cards/{protein}_{disease}_causal_card.md` + `.json` — MR result + coloc + protein dossier + target-disease evidence + known modulators + **clinically significant variants + population constraint + GWAS Catalog evidence + pharmacogenomic context** + integrated causal verdict.

**Tools (8 public databases):**

1. `get_mr_result(protein, disease)` — **IEU OpenGWAS** (pre-computed MR in DuckDB).
2. `get_uniprot_dossier(protein)` — **UniProt REST**.
3. `get_target_disease_evidence(target, disease_efo)` — **Open Targets GraphQL**.
4. `get_chembl_modulators(target_chembl_id)` — **ChEMBL REST**.
5. `get_clinvar_variants(gene_symbol)` — **ClinVar** via NCBI E-utilities: clinically significant variants and pathogenicity.
6. `get_gnomad_constraint(gene_symbol)` — **gnomAD API**: population allele frequencies, pLI / LOEUF metrics for target loss-of-function tolerance (essential for target-safety assessment).
7. `get_gwas_catalog(gene_or_trait)` — **GWAS Catalog API**: alternative / complementary genetic-evidence source to IEU OpenGWAS, especially for non-pQTL associations.
8. `get_pharmgkb_drug_gene(gene_symbol)` — **PharmGKB API**: drug-gene interactions and pharmacogenomic implications for repurposing.

**Why pre-compute MR?** TwoSampleMR + coloc are too slow for live workflows. Pre-compute the 5×1 grid (R + ieugwasr + TwoSampleMR + coloc) → DuckDB → fast agent tool lookup.

**Example runs (≥3):**

- `python agent.py --protein PNPLA3 --disease MASLD` → causal card: MR replicated, PP.H4=0.92, UniProt lipase domain context, 12 ChEMBL modulators, ClinVar I148M variant (NAFLD risk), gnomAD constraint shows tolerable for LOF, GWAS Catalog additional hits, PharmGKB lipid-pathway interactions.
- `python agent.py --protein MARC1 --disease MASLD` → card with MR estimate, dossier, only 2 known modulators (drug-development opportunity), gnomAD LOF-tolerant, ClinVar p.A165T protective variant.
- `python agent.py --protein PNPLA3 --protein HSD17B13 --disease MASLD --compare` → head-to-head causal verdict including PoLF tolerance comparison from gnomAD (HSD17B13 LOF appears protective).

---

## Week-by-Week

**Week 1 (~18h):** Run MR + coloc for 5 cis-pQTL proteins × MASLD in R; persist to DuckDB. **Hand off the engine sub-template.**
**Week 2 (~22h):** Build 8 Python tool functions (4 hit public APIs, 1 reads DuckDB, 3 wrap NCBI / gnomAD / GWAS Catalog / PharmGKB). Wire up Gemini. Build card formatter with embedded MR forest plot and gnomAD constraint plot.
**Week 3 (~15h):** Generate 5 causal cards + 2 head-to-heads; tune system prompt; README + demo.

## What's OUT

1,200-instrument cache, AlphaFold druggability scoring on 50+50 reference targets, RCSB PDB, full bidirectional mode, FAERS safety priors layer (the drug-safety component), 11-source target-dossier query layer.

## Stretch Goals

- 9th tool: `get_alphafold_pocket(uniprot_id)` for druggability scoring.
- Add deCODE as a second proteomic platform for MR replication.

## Realistic CV Entry

*Built OpenCausal, a working Gemini AI dossier-generator agent for causal target identification integrating 8 public databases spanning genetic causality, structure-function, chemistry, clinical variant interpretation, population genetics, and pharmacogenomics. Served as tech lead for the CausalSentinel team.*

- Ran two-sample MR with colocalization for 5 cis-pQTL proteins × MASLD in R, persisted to DuckDB for instant agent-tool retrieval.
- Wrapped 8 public databases (IEU OpenGWAS pre-computed, UniProt, Open Targets, ChEMBL, ClinVar, gnomAD, GWAS Catalog, PharmGKB) into a Gemini agent producing causal evidence cards with gnomAD-derived target-safety constraint metrics and PharmGKB-derived drug-gene context.
- Generated 5 causal cards + 2 head-to-head comparisons; ran the week-1 agent-setup hand-off for the shared starter template.

## Tech Stack

R (TwoSampleMR, coloc, ieugwasr) for pre-computation; Python, `google-generativeai`, DuckDB, requests, matplotlib for the agent layer; IEU OpenGWAS, UniProt REST, Open Targets GraphQL, ChEMBL REST, NCBI E-utilities (ClinVar), gnomAD API, GWAS Catalog API, PharmGKB API.

---

## Shared Agent Skeleton (three paradigms, one Gemini primitive)

Every intern's agent uses Gemini's automatic function calling, but the interface layer differs by paradigm. The cohort uses **one starter repo with three sub-templates** that interns clone in week 1:

- **Dossier-generator template** — CLI script: takes structured args, runs the agent workflow autonomously, writes `*.md` + `*.json` to disk.
- **Dashboard template** — Streamlit page with selectors and tables; the agent is invoked on button-click for specific synthesis tasks.
- **Computation-engine template** — Streamlit form (or CLI) that takes structured analytical inputs, runs the agent workflow, produces a downloadable analytical report with plots.

**Why no chat interfaces?** Scientists need reproducible, shareable artifacts. The agent dimension (Gemini-as-orchestrator, autonomous tool-calling across multiple public databases, synthesis across sources) is preserved in all three paradigms; only the deliverable shape changes.

The starter repo with all three sub-templates is owned by another project in the cohort. The shared repo should also include pre-built wrappers for the most heavily-used databases (ChEMBL, openFDA FAERS, Open Targets, ClinVar) so multiple interns don't redo the same boilerplate.

### Reference snippet — Gemini function calling (same across all three paradigms)

```python
import google.generativeai as genai
import os
genai.configure(api_key=os.environ["GEMINI_API_KEY"])

def my_tool(arg: str) -> dict:
    """One-line docstring Gemini uses to decide when to call this tool."""
    return {"result": ...}

model = genai.GenerativeModel(
    model_name="gemini-2.5-flash",
    tools=[my_tool, other_tool, ...],   # 4-8 tools per agent
    system_instruction=open("system_prompt.md").read(),
)
chat = model.start_chat(enable_automatic_function_calling=True)
response = chat.send_message("structured request — one shot, not a conversation")
```
