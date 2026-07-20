# Results

Three runs, each changing one thing from the one before, and one check with no model in it.

Correlation is between predicted and actual essentiality across all genes in a cell line the
model hadn't seen. It's a check that the model learned something worth reading the weights for,
not the result itself. The weights are given in full.

## What was run

All three runs use the same 32 kidney cell lines from DepMap, the same two features per gene,
expression and damaging mutations, and 200 epochs. The cell lines are split five ways, so the
model is always tested on ones it hasn't seen. The ± is the spread across those five splits.

| Run | Network | Label | Heads | Correlation |
|---|---|---|---|---|
| — | none (MLP baseline) | raw | — | 0.39 |
| 1 | IID physical PPI, kidney-filtered | raw | 2 | 0.5321 ± 0.0624 |
| 2 | HumanBase kidney functional | raw | 4 | 0.5377 ± 0.0137 |
| 3 | HumanBase kidney functional | differential | 4 | 0.0351 ± 0.0156 |

A gene's weights are spread across its own connections and add up to 1. VHL has 12 connections,
BRCA1 has 1,390. So a weight means something within a gene and nothing between genes.

---

## Run 1 — physical interaction network

IID, where an edge means two proteins physically bind. Two edge features, and 2 attention heads
against 4, because the larger graph didn't fit in memory otherwise.

Folds: 0.4374, 0.4809, 0.5599, 0.5824, 0.5997 → **0.5321 ± 0.0624**

Ten genes from the COSMIC Cancer Gene Census, and the connection each one weighted highest:

| Gene | Connections | Top partner | Weight |
|---|---|---|---|
| VHL | 782 | ASTL | 0.3216 |
| TP53 | 3,317 | ZNF679 | 0.5019 |
| EGFR | 2,821 | GPR149 | 0.5132 |
| PTEN | 1,188 | QRFPR | 0.3932 |
| MTOR | 594 | ARSL | 0.0533 |
| MYC | 3,545 | MUC3A | 0.3559 |
| KRAS | 1,611 | PRR15L | 0.5261 |
| BRCA1 | 1,610 | LMNTD1 | 0.3851 |
| PIK3CA | 622 | DAP | 0.0854 |
| BRAF | 977 | CNBD1 | 0.2100 |

None of them is a known partner. Widening to the top five doesn't change it. None of the eight
partners run 2 recovers, CUL2, ELOC, ELOB, RPTOR, RICTOR, RAF1, YWHAB and YWHAG, appears
anywhere in these ten lists. PKD1L3 is the only gene in both, at rank 2 for VHL here.

Ranking every connection at once is no better. AZIN1–OAZ2 leads at 0.6636, and the rest is
paralog pairs, SYT6–SYT10 at 11 and TGM5–TGM6 at 38, and the same genes repeating, PRKN in eight
of the top 50 and MEX3A in seven of the top 100.

So the accuracy is there and the weights aren't readable.

---

## Run 2 — functional network

HumanBase kidney, where an edge means two genes are predicted to work together whether or not
they touch. One edge feature, 4 heads. Nothing else changed.

Folds: 0.5263, 0.5378, 0.5196, 0.5474, 0.5574 → **0.5377 ± 0.0137**

The same average as run 1, and four times tighter across the splits.

**Training.** Loss falls from about 0.27 to 0.21 over the first 40 epochs, then flattens, ending
near 0.198. After epoch 40 the test loss stays close to it: the gap averages 0.015, never passes
0.043, and at 11 of the 40 logged points the test loss is the lower of the two. Lowest test loss
lands at epoch 190, 189 and 194 in the three folds where it was recorded, right at the end
rather than early.

**Weights.** Same ten genes, top five connections each:

| Gene | Connections | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| VHL | 12 | PKD1L3 (0.7911) | CUL2 (0.4083) | ELOC (0.2117) | ELOB (0.1689) | MYL12A (0.0710) |
| TP53 | 434 | CUL9 (0.4927) | CCL18 (0.4042) | MAGEB18 (0.2949) | MDM4 (0.2930) | SETD7 (0.1561) |
| EGFR | 780 | GYPB (0.7895) | KRT14 (0.7438) | DOK6 (0.7290) | SHC3 (0.7256) | SLC7A2 (0.6219) |
| PTEN | 654 | MAGI3 (0.4195) | DLL1 (0.3075) | NEDD4 (0.1781) | IGBP1 (0.1445) | NEDD4 (0.0822) |
| MTOR | 48 | RPTOR (0.2877) | RICTOR (0.2596) | RICTOR (0.0909) | PPP2R2A (0.0795) | FKBP1A (0.0496) |
| MYC | 690 | ZNF121 (0.4980) | RPL11 (0.2283) | MAX (0.1090) | ZBTB17 (0.0991) | PARP10 (0.0900) |
| KRAS | 650 | CCNH (0.0844) | ZCCHC2 (0.0671) | ARID4B (0.0360) | HIRA (0.0344) | ARHGEF3 (0.0288) |
| BRCA1 | 1,390 | ZNF350 (0.5181) | BRIP1 (0.3205) | BRAP (0.3138) | MSH5 (0.2562) | DCLRE1A (0.2075) |
| PIK3CA | 876 | CNBP (0.1347) | CREB1 (0.1034) | CCDC126 (0.0757) | IRS1 (0.0501) | HCFC2 (0.0492) |
| BRAF | 14 | RAF1 (0.2329) | YWHAB (0.1554) | YWHAG (0.1530) | CCDC88A (0.0856) | PAK2 (0.0805) |

An edge is stored in both directions and gets a weight each way, so a partner can show up twice.
NEDD4 under PTEN and RICTOR under MTOR are one edge seen from both sides.

Partners were looked up in CORUM and UniProt. Three of the ten came back with partners they're
known to work with:

| Gene | Partners in top 5 | Rank |
|---|---|---|
| VHL | CUL2, ELOC, ELOB | 2, 3, 4 |
| MTOR | RPTOR, RICTOR | 1, 2 |
| BRAF | RAF1, YWHAB, YWHAG | 1, 2, 3 |

The other seven haven't been checked.

Weight doesn't follow connection count. VHL has 12 and puts 0.7911 on one. BRAF has 14 and
reaches 0.2329. EGFR has 780 and reaches 0.7895.

The full ranking is the same mix as run 1, though real pairs sit in it: MAP2K1–KSR2 at 4,
HSPB8–HSPB7 at 6, FZD1–WNT3A at 16, MAX–MYCN at 78, RICTOR–DEPTOR at 91.

---

## Run 3 — kidney-specific labels

Same network and settings as run 2. The label changed. Each gene's average score across all
cancer cell lines is subtracted, so a gene essential everywhere flattens to zero and what's left
is specific to kidney.

**Before training.** PAX8, a kidney lineage dependency, comes out at −0.1766, HIF1A at +0.1288.
Across all 336,588 values the standard deviation is 0.208, the range is −2.65 to 3.33, and 51%
sit within ±0.1 of zero.

Each gene's label against its number of connections, over 10,592 genes: the raw label at
ρ = −0.291 (p = 4.5 × 10⁻²⁰⁶), the kidney-specific one at ρ = +0.071 (p = 3.5 × 10⁻¹³).

**Prediction.** Folds: 0.0205, 0.0491, 0.0502, 0.0435, 0.0124 → **0.0351 ± 0.0156**

**Weights.**

| Gene | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| VHL | CUL2 (0.4311) | PKD1L3 (0.3981) | CCDC82 (0.3559) | PKD1L3 (0.3533) | MYL12A (0.2430) |
| TP53 | XRCC1 (0.4515) | CUL9 (0.3596) | DPH1 (0.3079) | STX5 (0.2790) | KAT8 (0.2748) |
| EGFR | CD82 (0.7558) | HOXC10 (0.7400) | CLTCL1 (0.6990) | AREG (0.6817) | AASS (0.5547) |
| PTEN | NEDD4 (0.2034) | NEDD4 (0.1396) | MAGI3 (0.0993) | DLL1 (0.0691) | FBXO33 (0.0654) |
| MTOR | RICTOR (0.3472) | RPTOR (0.2757) | STAT1 (0.1139) | RICTOR (0.0863) | PKP4 (0.0464) |
| MYC | ZNF121 (0.4608) | ZBTB17 (0.1145) | MAX (0.0924) | MCM7 (0.0858) | TRRAP (0.0792) |
| KRAS | HIRA (0.0390) | PRKACB (0.0342) | IL6ST (0.0275) | HNRNPC (0.0194) | GALNT1 (0.0162) |
| BRCA1 | BRAP (0.4191) | ZNF350 (0.3948) | BRIP1 (0.3904) | DCLRE1A (0.3726) | BARD1 (0.2665) |
| PIK3CA | CREB1 (0.1362) | CCDC126 (0.0758) | PIK3R1 (0.0650) | HCFC2 (0.0432) | RAB2B (0.0420) |
| BRAF | YWHAG (0.3355) | YWHAB (0.1432) | RAF1 (0.1093) | YWHAQ (0.1029) | CCDC88A (0.0914) |

All three genes checked in run 2 keep their partners here. VHL has CUL2 at 1, MTOR has RICTOR
and RPTOR at 1 and 2, BRAF has YWHAG, YWHAB and RAF1 at 1, 2 and 3. The correlation is 0.035.

The full ranking leads with HSP90AB1–TPM3 at 0.9164, then COPS6–COPS9 at 10, C3–CFB at 50 and
MDM2–MTBP at 52.

---

## Is the kidney signal there at all

No model in this one. The 32 cell lines were split into two halves of 16. Each half was used
separately to work out which genes look kidney-specific, and the two answers were compared. If
it were noise the halves would disagree.

Over 20 random splits, across the 10,518 genes with complete labels, they agree at
**0.5182 ± 0.0396**, which works out to **0.683** for the full 32 lines.

So the signal is in the label, and run 3 gets 0.035 on it.
