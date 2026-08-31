# Adversarial audit record — 2026-08-07

Recovered 2026-08-30 from the workflow transcript that produced it (`wf_73dd8598-8cc`, 41 agent records, 2026-08-07 06:19–06:37). Until then the only surviving statement of these numbers was one sentence in `CHANGELOG.md`. This file restores the underlying record so the manuscript's counts can be checked rather than taken on trust.

## How it was run

The audit was orchestrated as a multi-agent workflow: four attack agents, each given a
different reviewing lens and the complete bundle of ten generated cards together with the
verbatim tool returns they were written from; then one skeptic agent per allegation,
instructed that its default answer was that the allegation is wrong and that it must name
the specific tool field making the statement unearned before upholding; then one synthesis
agent. Every agent was an instance of a large language model. Attack and skeptic agents
returned structured objects against a fixed schema, which is why the record can be
reconstructed exactly rather than paraphrased.

The four lenses were: statistical genetics and MR methodology; drug discovery and target
validation; evidence provenance and misattribution; and pretrained-knowledge leakage.

## The funnel

| stage | n | how |
|---|---|---|
| alleged defects | **86** | 4 critic lenses, each judging all 10 generated cards |
| sent for adjudication | **36** | **capped** — the harness verified `defects[:36]` |
| never adjudicated | **50** | the remainder of the 86; no verdict exists for these |
| upheld by a skeptic | **27** | each checked by an independent agent instructed to refute |
| distinct defects after deduplication | **18** | reported in `CHANGELOG.md` v0.2 |

**The cap matters and must be disclosed.** Fifty of the eighty-six allegations were never checked, so 18 is a lower bound on the defects this audit could have confirmed, not a total. The validator's recall of 0/18 is unaffected: it flagged none of the eighteen.

## Allegations per lens

| lens | alleged |
|---|---|
| ? | 86 |

## Upheld defects

**27 upheld, 9 refuted**, out of 36 adjudicated.

| severity | n |
|---|---|
| major | 19 |
| critical | 6 |
| minor | 2 |

| card | upheld defects |
|---|---|
| TREM2 x Alzheimer disease | 5 |
| PNPLA3 x MASLD | 4 |
| IL6R x coronary heart disease | 4 |
| SCN2A x epilepsy | 4 |
| VKORC1 x venous thrombosis | 3 |
| PCSK9 x high cholesterol | 2 |
| HMGCR x high cholesterol | 2 |
| LPA x coronary heart disease | 1 |
| OR51E2 x epilepsy | 1 |
| IL6R x Alzheimer disease | 1 |

### Categories

Fourteen of the 27 upheld defects are pretrained-knowledge leakage — a sentence true of the biology but not derivable from that run's ledger. Counting the explicit `leaked-*` categories together with `pretrained-leakage` gives **16**.

| category | n |
|---|---|
| `pretrained-leakage` | 4 |
| `wrong-tool-attribution` | 3 |
| `leaked-rsid` | 1 |
| `leaked-mechanism` | 1 |
| `leaked-annotation-text` | 1 |
| `leaked-phenotype-attribution` | 1 |
| `leaked-indication` | 1 |
| `leaked-annotation-category` | 1 |
| `leaked-modality-and-approval` | 1 |
| `leaked-approval-status` | 1 |
| `leaked-development-stage` | 1 |
| `leaked-efficacy-evidence` | 1 |
| `leaked-modality` | 1 |
| `leaked-function-text` | 1 |
| `direction-error` | 1 |
| `caveat-contradiction` | 1 |
| `sampled-count-as-rate` | 1 |
| `verdict-not-supported` | 1 |
| `truncated-lookup-as-negative` | 1 |
| `wrong-field-attribution` | 1 |
| `ignored-tool-result` | 1 |
| `misused-metric` | 1 |

## The six critical defects, in full

Each survived an independent skeptic whose default instruction was to refute the allegation.

### 1. LPA x coronary heart disease — `leaked-development-stage`

> GO — Robust genetic and causal evidence demonstrates that elevated LPA levels strongly increase coronary heart disease risk, supported by clinical-stage antisense and RNAi inhibitors.

**Why it is wrong.** 'clinical-stage' is not in the ledger, and it recurs in the reasoning as 'ChEMBL identifies active clinical modulators'. get_chembl_modulators returned two entries against target 'LPA mRNA' with actions RNAI INHIBITOR and ANTISENSE INHIBITOR and no phase, status, or development-stage field of any kind. The model knows pelacarsen and olpasiran are in late-phase trials and promoted two bare ChEMBL IDs to clinical assets. For 'clinical-stage' to be earned, get_chembl_modulators would have had to return max_phase or a development-status field.

**Why the validator missed it.** The blacklisted phrase is 'clinical trial'; 'clinical-stage' and 'active clinical modulators' are near-synonyms the regex does not match, and ChEMBL found:true with two modulators, so the existence half of the claim checks out. Card scored ok:true.

**Suggested fix.** Say 'ChEMBL lists two nucleic-acid inhibitors against LPA mRNA; development stage was not retrieved.' Add max_phase to the ChEMBL return and gate every stage/approval word on it.

**Skeptic.** UPHELD. The quote is verbatim from the card's verdict field. The offending field is get_mr_result.matched_disease_estimates[0].beta = -0.0441899892357374, with protein = \"IL6R\" and the tool being EpiGraphDB pQTL MR, i.e. the exposure is plasma IL6R protein level (Wald ratio, n_snp = 1, cis instrument rs4129267). A negative beta on a protein-level exposure says genetically HIGHER IL6R protein is associated with LOWER CHD. The ledger therefore supports \"raising IL6R protein is protective\"; the verdict asserts the opposite direction of pharmacological modulation, \"a causal, protective role of IL6R INHIBITION.\" The auditor's internal-consistency proof holds: on the LPA card in the same bundle, against the same outcome row (outcome_gwas_id \"7\", Coronary heart disease), beta = +0.252303585657371 was rendered as \"elevated LPA levels strongly increase coronary heart disease risk\" — the same convention applied to IL6R's beta = -0.044 gives higher-protein-lower-risk. The sign was flipped on IL6R only, to land on a remembered conclusion. Nothing in this run's ledger bridges the gap: get_chembl_modulators shows INHIBITOR/ANTAGONIST actions (what the drugs do, not that inhibition benefits CHD); get_target_disease_evidence.datatype_scores.clinical = 0.0061 for MONDO_0005010 (essentially no clinical evidence for this indication); get_pharmgkb_drug_gene lists tocilizumab with no outcome or direction. The Asp358Ala/rs2228145 receptor-shedding mechanism that legitimately reverses the sign in the published literature appears in no tool field of this run — this is pretrained-knowledge leakage carrying the load. Aggravating: get_mr_result.note instructs \"Check cis_or_trans, steiger_direction_ok, and coloc_prob before treating this as causal,\" and this row has steiger_direction_ok = \"NA\" and coloc_prob = null (unlike the LPA and PCSK9 rows, which carry \"TRUE\"), so the card asserts \"causal\" and a direction on an estimate whose direction check the ledger marks unavailable. Partial mitigation, insufficient to refute: the reasoning body hedges with the directionless word \"modulation\" (\"genetically proxied IL6R modulation\"), but the verdict does not hedge — it names inhibition, and the verdict is the actionable headline. Severity stays critical: this is a sign reversal on a GO recommendation, and it belongs to the leakage class the system most fears, since the sentence is biologically true and so no reviewer will challenge it while the pipeline established the opposite direction. Suggested fix as written is appropriate: state the estimate in ledger terms (each SD higher genetically-predicted plasma IL6R associated with lower CHD, beta -0.044, se 0.0085, p 2.2e-07, single cis instrument rs4129267, Steiger NA, coloc unavailable), then either drop \"inhibition\" or flag explicitly that translating this estimate into IL6R blockade requires the Asp358Ala shedding mechanism, which this run did not retrieve.

### 2. IL6R x coronary heart disease — `direction-error`

> GO — genetic and Mendelian randomization evidence supports a causal, protective role of IL6R inhibition in coronary heart disease.

**Why it is wrong.** The retrieved estimate is a pQTL MR: exposure = plasma IL6R protein level, outcome = CHD, beta = -0.0442. Read off the ledger, the negative sign means HIGHER measured IL6R protein maps to LOWER CHD risk. The card inverts this into 'IL6R inhibition' being protective. The reconciliation that actually makes the card's sentence true (rs4129267/Asp358Ala raises soluble IL6R while blunting classic IL-6 signalling, so blockade mimics the protective allele) is pure outside knowledge — nothing in get_mr_result, get_uniprot_dossier or get_chembl_modulators states what the pQTL exposure means mechanistically or that lowering the protein reproduces the sign. As written, the ledger's sign points the opposite way to the recommendation.

**Why the validator missed it.** beta = -0.044 is reproduced verbatim and matches the ledger numerically, so the number check passes; MR was found=true so the causal-language rule is satisfied. The validator has no model of what the exposure is or of which direction of perturbation the sign implies.

**Suggested fix.** State the estimate in its own terms and stop: 'higher genetically-proxied plasma IL6R associates with lower CHD (beta = -0.044, p = 2.2e-07, single cis SNP rs4129267)'. Do not translate a protein-level beta into a direction of pharmacological modulation unless a tool in the ledger establishes that mapping.

**Skeptic.** Verified in bundle_trim.json. The quote is verbatim in the TREM2 card's VERDICT line: "GO — Strong genetic and biological evidence supports TREM2 as a causal target for Alzheimer disease, through microglial activation and amyloid-beta clearance." Scanning all 10 cards field-by-field, the strings microglia/microglial, amyloid, clearance and activation occur ONLY in this card's model-written verdict and reasoning — zero occurrences in any tool_results anywhere in the bundle. The specific missing field: get_uniprot_dossier returns an identical fixed key set in all 10 cards (accession, diseases, found, gene_names, gene_symbol, protein_name, source_release, subcellular_location, url) — there is no function/annotation field at all, so mechanism is not merely un-retrieved here, the tool surface cannot produce it. No other tool carries mechanism either: OT returns only datatype_scores, ClinVar counts, gnomAD constraint metrics, GWAS rsIDs, ChEMBL n_modulators:0, PharmGKB found:false. The only lexical neighbour in the ledger is protein_name "Triggering receptor expressed on myeloid cells 2"; "myeloid cells" -> "microglial activation" requires outside knowledge, and "amyloid-beta clearance" has no anchor whatsoever. The clause is load-bearing, not decorative: get_mr_result returned found=false with the explicit note "TREM2 HAS pQTL MR estimates in this resource, but NONE matched the requested disease 'Alzheimer disease' ... do not present them as evidence about 'Alzheimer disease'", so the invented mechanism is the only thing in the verdict line supporting the word "causal". Its asserted direction (TREM2 -> amyloid-beta clearance) is itself an unsettled question in the field, so it is not even a safe "correct but unearned" sentence. The same failure recurs in the reasoning, where "explicitly annotated in UniProt as playing a causal role in Alzheimer disease 17 via amyloid-beta and lipoprotein binding" falsely attributes a mechanism to a tool whose diseases field is the bare two-item list ["Polycystic lipomembranous osteodysplasia with sclerosing leukoencephalopathy 2", "Alzheimer disease 17"]. The validator-miss account also checks out: it logged only {kind: qualitative-claim, token: small-molecule} with checked=2, and the mechanism clause contains no number, rsID, accession, or listed token for a regex to bind to. The suggested fix is appropriate.

### 3. SCN2A x epilepsy — `sampled-count-as-rate`

> ClinVar lists thousands of records for the gene, including numerous pathogenic variants

**Why it is wrong.** The tool returned total_records 3108 and pathogenic_in_sample 11 with sample_size 30, plus the explicit warning: 'Pathogenic count is over the 30 record(s) retrieved, NOT over all 3108 ClinVar records for this gene; it is a sample, not a rate.' The sentence welds the 3108 total to the sampled pathogenic count so that 'thousands of records, including numerous pathogenic variants' reads as thousands of records of which many are pathogenic. Eleven of thirty sampled records is the only fact retrieved, and it licenses no statement about the other 3078. The verdict repeats the inflation as 'extensive clinical variant data'.

**Why the validator missed it.** Neither 3108 nor 11 is quoted, so there is nothing to compare numerically; 'thousands' and 'numerous' are vague quantifiers that a value-matching regex cannot evaluate, and the sampling caveat lives in a tool note the checker does not reconcile against prose.

**Suggested fix.** 'ClinVar holds 3108 records for SCN2A; of the 30 records sampled in this run, 11 are classified pathogenic — a sample, not a rate over the full record set.'

**Skeptic.** UPHELD. Quote verified verbatim: the IL6R x coronary heart disease entry's "verdict" field reads exactly "GO — genetic and Mendelian randomization evidence supports a causal, protective role of IL6R inhibition in coronary heart disease."

The specific tool field that makes it unearned: get_mr_result.matched_disease_estimates[0] returns beta = -0.0441899892357374 for protein "IL6R" on outcome "Coronary heart disease" (outcome_gwas_id "7", instrument_rsid "rs4129267", method "Wald ratio", n_snp 1), under source_release "EpiGraphDB pQTL MR (Zheng et al., Nat Genet 2020)". In that resource the exposure is the plasma protein level, so the estimate says: a genetically-proxied INCREASE in circulating IL6R is associated with LOWER CHD risk. The card converts that into an endorsement of receptor INHIBITION — the opposite direction of modulation of the measured exposure — and attributes the conversion to the MR ("Retrieved published Mendelian randomization estimates suggest a significant protective effect of genetically proxied IL6R modulation ... beta = -0.044").

I searched the whole ledger for anything that could bridge protein abundance to drug direction and found nothing. get_uniprot_dossier gives only protein_name "Interleukin-6 receptor subunit alpha" and subcellular_location ["Cell membrane","Secreted","Secreted"] — no soluble-vs-membrane receptor annotation, no classical-vs-trans signalling note, no direction-of-effect field. get_chembl_modulators lists INHIBITOR/ANTAGONIST entries, but that establishes only that the available chemical matter blocks the receptor, not that blockade is the protective direction. get_mr_result has no exposure-direction annotation and no drug-target MR. So the bridging premise (that the sIL6R-raising allele actually attenuates IL-6 signalling and mimics blockade) is pretrained knowledge the model supplied, not anything this run retrieved.

The auditor's internal-consistency check also holds: the LPA x coronary heart disease card in the same bundle, same outcome_gwas_id "7", reads beta = +0.252303585657371 correctly as "elevated LPA levels strongly increase coronary heart disease risk ... supported by clinical-stage antisense and RNAi inhibitors". Same tool, same sign convention, opposite sign — and only here is the sign not honoured.

I looked for a defensible reading and could not find one. "Modulation" in the reasoning is vague enough to survive on its own, but the verdict commits to "inhibition", and nothing downgrades that to a hypothesis. The fact that the sentence happens to be true of the real biology (the Asp358Ala/rs4129267 story) is exactly why it is dangerous rather than why it is acceptable — it is the pretrained-knowledge-leakage failure mode expressed as a direction claim, and no reviewer reading the card would catch it.

Severity confirmed critical rather than major: this is the one quantitative causal number in the run, and the claim it is used to license — inhibit versus raise — is the agonist/antagonist decision itself. Corroborating (not part of the allegation, so not scored): the same MR record carries steiger_direction_ok "NA", steiger_p null, coloc_prob null, ld_check null, n_snp 1, and the tool note explicitly says to check those "before treating this as causal"; the card calls it causal anyway.

Suggested fix as filed is sound: state the retrieved direction literally, state that this run retrieved nothing linking that exposure direction to receptor blockade, and downgrade from a GO on inhibition to a directional-uncertainty flag.

### 4. SCN2A x epilepsy — `verdict-not-supported`

> the overwhelming genetic and clinical validation establishes it as a prime target for epilepsy

**Why it is wrong.** Everything retrieved (Open Targets 0.9057, UniProt disease entries, ClinVar counts) establishes that SCN2A variation is ASSOCIATED with epilepsy — it says nothing about which direction of modulation would help. Against that, get_gnomad_constraint returned pLI = 1, LOEUF = 0.154, lof_z = 10.4 with the declared caveat 'inhibiting it carries a safety warning', and get_chembl_modulators returned n_modulators = 0 for the resolved target SCN2A/SCN1B. The card acknowledges the constraint caveat in a subordinate clause and then overrides it with the association evidence, which is not evidence about the same question. A GO on a target with zero retrieved modulators and the strongest LoF-intolerance signal in the bundle does not follow from the assembled ledger.

**Why the validator missed it.** MR returned found=false and the card avoids the word 'causal', so the causal-language rule is not triggered; nothing numeric is misquoted. Whether a verdict follows from the evidence set is outside a regex checker's reach.

**Suggested fix.** Downgrade to REVIEW/CAUTION and state the conflict: strong disease association, but LoF-intolerance (pLI 1, LOEUF 0.15) argues against an inhibitory strategy and no ChEMBL modulator was retrieved; the retrieved evidence does not indicate which direction of modulation is therapeutic.

**Skeptic.** Quote verified verbatim as the card's `verdict` field (line 1136). A grep of the entire bundle for `amyloid|microgl|clearance|myeloid` returns only lines 1136-1137 (the model-written verdict and reasoning) plus three instances of the string "Triggering receptor expressed on myeloid cells 2" (UniProt `protein_name`, ChEMBL `target_name`, and the ChEMBL match note). The tokens "microglia", "amyloid", and "clearance" appear in NO tool return anywhere in the file. The specific field that makes the clause unearned: `get_uniprot_dossier` for TREM2 returns only `gene_symbol`, `accession`, `gene_names`, `protein_name`, `subcellular_location` (["Cell membrane","Secreted","Secreted"]) and `diseases` (["Polycystic lipomembranous osteodysplasia with sclerosing leukoencephalopathy 2","Alzheimer disease 17"]) — there is no function, ligand-binding, or pathway field, so no mechanism is derivable. The same card's reasoning compounds this with fabricated provenance: "explicitly annotated in UniProt as playing a causal role in Alzheimer disease 17 via amyloid-beta and lipoprotein binding" — UniProt returned the disease NAME only, with no causal-role and no binding annotation. Tractability claims check out: `get_chembl_modulators.n_modulators` = 0 with `modulators: []`, `get_pharmgkb_drug_gene.found` = false, and `get_mr_result` returns `found: false`, `matched_disease_estimates: []`, note "TREM2 HAS pQTL MR estimates in this resource, but NONE matched the requested disease 'Alzheimer disease'." Defenses considered and rejected: "myeloid cells 2" in `protein_name` is the only nearby thread, and myeloid→microglia requires external knowledge (nothing in the ledger mentions brain or CNS) while rescuing at most half the clause — "amyloid-beta clearance" has no anchor of any kind. This is not style: "activation" asserts a therapeutic DIRECTION (agonist programme) in the headline GO line, while the only directional signal actually in the ledger (the PLOSL2 entry in UniProt `diseases`) is never mentioned. Validator-blindness explanation confirmed: the validator block lists exactly one unsupported item, `qualitative-claim: small-molecule`. Critical severity holds — a mechanism the model simply knows, laundered into a GO verdict and additionally mis-sourced to a named tool, in a pipeline whose entire value proposition is ledger traceability.

### 5. PNPLA3 x MASLD — `truncated-lookup-as-negative`

> While ChEMBL currently lists no small-molecule modulators

**Why it is wrong.** get_chembl_modulators returned found=false with note 'No ChEMBL target for PNPLA3.' — the lookup did not resolve a target at all, so no modulator query was ever run. That is a failed resolution, not a curated finding of zero modulators. Compare OR51E2 and TREM2 in this bundle, where a target DID resolve and n_modulators = 0 was genuinely returned; the ledger distinguishes those two situations and the card collapses them. The card also asserts the absent modulators are specifically 'small-molecule', a modality the tool never had a chance to report.

**Why the validator missed it.** The validator caught the 'small-molecule' token but has no rule for found=false semantics — it cannot tell a resolution failure from a legitimate empty result set, so the negative framing passes.

**Suggested fix.** 'ChEMBL returned no target record for PNPLA3 in this run (lookup did not resolve), so the modulator landscape is unknown rather than empty.'

**Skeptic.** Quote verified verbatim in the model-written reasoning of the PCSK9 x high cholesterol card. The ledger contradicts it on two counts. (1) Modality: get_chembl_modulators returned n_modulators=2 with modulators[0].action='RNAI INHIBITOR' and modulators[1].action='RNAI INHIBITOR', against target_chembl_id CHEMBL4630662 whose target_name is 'PCSK9 mRNA'. No antibody, protein, or any second modality class appears in any field. The tool's own note — 'ChEMBL target matched by text search on PCSK9 and resolved to PCSK9 mRNA — confirm this is the intended target' — is a declared caveat the reasoning silently ignores; because the search landed on the mRNA entity, a biologic modulator could not have been retrieved even if one exists, so the card asserts a modality class this run provably could not have seen. The 'RNAi AND biologic' conjunction asserts an additional class, so it cannot be defended by calling siRNA a biologic (that reading makes the clause redundant and still has no supporting field). (2) Clinical status: no field in any of the 8 tool returns carries development phase, approval, or drug names — only bare ChEMBL IDs (CHEMBL3990033, CHEMBL5095052) — and get_pharmgkb_drug_gene returned found=false ('No PharmGKB/ClinPGx clinical annotations'), so the one tool that could have named a marketed drug returned nothing. 'in clinical practice' is pretrained-knowledge leakage (evolocumab/alirocumab/inclisiran). The number 246 is genuine (get_gwas_catalog.n_association_rows=246), which is precisely why the sentence reads as verified. Validator passed the card (ok=true, checked=1) because its fixed token list covers 'monoclonal antibody' and 'FDA-approved', not the paraphrases 'biologic modulators' and 'in clinical practice'. Severity kept at critical: the sentence is the modality line on a GO card, and for a siRNA-vs-mAb portfolio decision it asserts both a modality class and an existing approved competitor that the pipeline never established. Note the GO verdict itself is otherwise sound — MR beta=+0.277 (higher PCSK9 -> higher cholesterol) means inhibition lowers it, so the direction is correct; the defect is confined to the unearned modality/approval clause. The auditor's suggested_fix is accurate.

### 6. VKORC1 x venous thrombosis — `pretrained-leakage`

> whose inhibition by coumarin anticoagulants is directly used for the prevention and treatment of venous thrombosis

**Why it is wrong.** No tool in this card reports an indication, a treatment use or an outcome. ChEMBL returned 7 unnamed inhibitor entries by CHEMBL ID; PharmGKB returned drug names (warfarin, acenocoumarol, phenprocoumon, fluindione) attached to clinical annotations whose subject — as the card's own next sentence concedes — is 'dosing and toxicity outcomes'. The card converts a pharmacogenetic dose-response resource into proof of therapeutic indication for a specific disease. The therapeutic-use statement is the model's own knowledge, placed in the verdict line. The ledger even contains a counter-signal the card omits: UniProt lists VKORC1 disease entries 'Combined deficiency of vitamin K-dependent clotting factors 2' and 'Coumarin resistance', i.e. loss of function produces a bleeding phenotype.

**Why the validator missed it.** All four drug names are genuine ledger strings and the sentence avoids 'FDA-approved' and 'clinical trial'. The validator checks that named entities exist somewhere in the ledger, not that the RELATIONSHIP asserted between them (drug treats this disease via this target) was ever retrieved.

**Suggested fix.** 'PharmGKB holds 38 VKORC1 clinical annotations (3 at level 1A, 5 at 1B, 8 at 2A) linking VKORC1 variation to coumarin dosing and toxicity; ChEMBL lists 7 inhibitor records against VKORC1. These establish a pharmacogenetic dose-response relationship, not an indication for venous thrombosis.'

**Skeptic.** Verified and upheld. The quote appears verbatim in the model-written reasoning of the TREM2 x Alzheimer disease card (bundle line 1137), so it is correctly attributed to model text rather than a mechanically rendered block. The specific tool field that makes it unearned is get_uniprot_dossier (lines 1176-1195), which returns only found, gene_symbol, accession, gene_names, protein_name, subcellular_location (Cell membrane / Secreted / Secreted), diseases, url and source_release. The diseases value is a bare string array — ["Polycystic lipomembranous osteodysplasia with sclerosing leukoencephalopathy 2", "Alzheimer disease 17"] — with no causality qualifier, and the return has no function, interaction, ligand or binding field of any kind. This is not a trimming artifact: the identical nine-key schema holds for all ten cards' UniProt returns (P04035, P08887, P08519, Q9H255, Q8NBP7, Q9NST1, Q99250, Q9BQB6), and truncation elsewhere in this bundle is explicitly marked (PNPLA3 reasoning ends "...[truncated]") while tool_results blocks are intact. A case-insensitive grep confirms "amyloid", "lipoprotein" and "microglia" occur in this card ONLY on lines 1136-1137 (the model-written verdict and reasoning) and nowhere in the TREM2 tool_results block at 1148-1255; the sole other lipoprotein hit in the file is "Apolipoprotein(a)" at line 533, which belongs to the LPA card. So neither binding mechanism is derivable from any of the eight tools in this run, not just from UniProt. Two independent faults compound: a bare disease-association string is upgraded to "playing a causal role", and two molecular mechanisms are attributed to a tool that returned no mechanism data — with the word "explicitly" converting the whole clause into a falsifiable provenance claim aimed at a UniProt record the SOURCES block links (Q9NZC2). Severity is critical rather than major because this fabricated mechanism carries the entire load of a GO verdict whose other evidence cannot support the word "causal": get_mr_result returned found=false with matched_disease_estimates=[] and the explicit note "TREM2 HAS pQTL MR estimates in this resource, but NONE matched the requested disease 'Alzheimer disease'"; get_chembl_modulators returned n_modulators=0; gnomAD is LoF-tolerant (pLI 7.9e-07, LOEUF 1.268) with the tool's own caveat that this "is NOT positive evidence of efficacy"; and PharmGKB found=false. The same leakage also contaminates the verdict line ("through microglial activation and amyloid-beta clearance"), which strengthens the finding. The validator flagged only the "small-molecule" token here, consistent with the auditor's explanation: "Alzheimer disease 17" is a verbatim ledger string that passes any string-presence check, and "amyloid-beta"/"lipoprotein binding" carry no numbers or accessions and are not on the fixed qualitative-claim list. This is textbook pretrained-knowledge leakage — the sentence is true of TREM2 biology, so a scientist reads past it, while the pipeline never established it.

## What this changes in the manuscript

1. The funnel `86 → 36 adjudicated → 27 upheld → 18 distinct` now has a record behind it, and can replace the bare `18`.
2. **The 36-cap must be stated.** Reporting 86 → 27 → 18 without it overstates how much of the allegation set was actually tested.
3. The IL6R direction error the paper builds on is item 2 of the six critical defects, with the skeptic's independent confirmation.
4. `41 agents` is the total agent count — 4 attackers, 36 skeptics, 1 synthesiser — not 41 critics. The manuscript should say so.

Full machine-readable record, all 86 allegations and all 36 verdicts: `audit_2026-08-07.json`.
