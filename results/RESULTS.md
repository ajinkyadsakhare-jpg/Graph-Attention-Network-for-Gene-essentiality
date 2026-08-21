# Results

Three runs, each changing one thing from the one before, and one check with no model in it.

Correlation is between predicted and actual essentiality across all genes in a cell line the
model hadn't seen. It's a check that the model learned something worth reading the weights for,
not the result itself.

## What was run

All three runs use the same 32 kidney cell lines from DepMap and the same two features per gene,
expression and damaging mutations as nodes , 4 attention heads and 200 epochs. The cell lines are
split five ways using (k-fold), so the model is always tested on ones it hasn't seen. The ± is the 
spread across those five splits.

0  MLP baseline | raw | 0.39 |
1  IID physical PPI, kidney-filtered | raw | 0.5551 ± 0.0500 |
2  HumanBase kidney functional | raw | 0.5191 ± 0.0449 |
3  HumanBase kidney functional | differential | 0.0351 ± 0.0156 |

A gene's weights are spread across its own connections and add up to 1. VHL has 7 connections,
BRCA1 has 696. So a raw weight means something within a gene and nothing between genes. The
tables below give a normalised weight alongside it, the raw weight multiplied by the number of
connections. That's 1.0 if a gene spread its attention evenly, higher if it concentrated on one
partner, and it can be compared between genes.

Connection counts are lower here than in earlier versions of this file. The readout now counts
incoming edges only, plus the self-connection the model adds to every gene. Before, each edge
was counted twice, once in each direction. The network is unchanged.

---

## Run 1: physical interaction network

IID, where an edge means two proteins physically bind. Two edge features, the physical
interaction score and coexpression.

Folds: 0.5845, 0.4618, 0.5626, 0.5583, 0.6084 → **0.5551 ± 0.0500**

This run stores each edge one way only, in whichever direction the IID file lists it. Both
directions wouldn't fit in memory at this edge count.

Ten genes from the COSMIC Cancer Gene Census, and the connection each one weighted highest:

| Gene | Connections | Top partner | Weight | Normalised |

| VHL | 260 | ATE1 | 0.0324 | 8.44 |
| TP53 | 474 | HRK | 0.0155 | 7.35 |
| EGFR | 367 | CD300LB | 0.0245 | 8.99 |
| PTEN | 533 | PTPRT | 0.0297 | 15.82 |
| MTOR | 220 | FBP2 | 0.0618 | 13.60 |
| MYC | 437 | GALR3 | 0.0424 | 18.51 |
| KRAS | 224 | PHACTR2 | 0.0316 | 7.08 |
| BRCA1 | 512 | KIF2A | 0.0405 | 20.72 |
| PIK3CA | 230 | FBP2 | 0.0382 | 8.78 |
| BRAF | 220 | ODAM | 0.0313 | 6.88 |

None of them is a known partner. Widening to the top five doesn't change it. None of the eight
partners run 2 recovers, CUL2, ELOC, ELOB, RPTOR, RICTOR, RAF1, YWHAB and YWHAG, appears
anywhere in these ten lists. FBP2 is top partner for both MTOR and PIK3CA, and appears again
under BRAF, which suggests it's generic rather than specific to any of them.

Ranking every connection at once is no better. MAP1LC3B2–MAP1LC3B leads, and the rest is the
same genes repeating, KIF2A in ten of the top 100, TADA2A in seven, RRP9 in six.

So the accuracy is there and the weights aren't readable.

---

## Run 2: functional network

HumanBase kidney, where an edge means two genes are predicted to work together whether or not
they touch. One edge feature instead of two. Nothing else changed.

Folds: 0.5502, 0.5565, 0.4370, 0.5054, 0.5466 → **0.5191 ± 0.0449**

Almost the same average as run 1, and a similar spread across the splits. Unlike run 1, this run
stores each edge in both directions.

**Weights.** Same ten genes, top five connections each. Raw weight first, normalised in
brackets:

| Gene | Connections | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| VHL | 7 | CUL2 (0.4287, 3.00) | ELOB (0.1799, 1.26) | ELOC (0.1632, 1.14) | CCDC82 (0.0792, 0.55) | MYL12A (0.0461, 0.32) |
| TP53 | 218 | YBX1 (0.0692, 15.09) | HDAC1 (0.0427, 9.31) | PML (0.0394, 8.60) | PKM (0.0393, 8.57) | STX5 (0.0345, 7.53) |
| EGFR | 391 | ERBB2 (0.1490, 58.27) | CDCP1 (0.0891, 34.83) | CD82 (0.0400, 15.62) | AREG (0.0180, 7.06) | ATP5F1C (0.0167, 6.51) |
| PTEN | 328 | IGBP1 (0.0814, 26.71) | NEDD4 (0.0795, 26.06) | EIF3E (0.0524, 17.18) | CDC27 (0.0467, 15.31) | NSA2 (0.0329, 10.80) |
| MTOR | 25 | RICTOR (0.2761, 6.90) | PPP2R2A (0.0813, 2.03) | MAPKAP1 (0.0614, 1.53) | SDHB (0.0510, 1.27) | RPTOR (0.0508, 1.27) |
| MYC | 346 | RPL11 (0.1974, 68.31) | GEMIN4 (0.0848, 29.35) | MCM7 (0.0754, 26.10) | MAX (0.0511, 17.67) | NMI (0.0465, 16.08) |
| KRAS | 326 | CCNH (0.0653, 21.30) | HNRNPK (0.0431, 14.04) | PRKACB (0.0374, 12.20) | HIRA (0.0253, 8.25) | CLTA (0.0225, 7.34) |
| BRCA1 | 696 | NONO (0.1052, 73.24) | BARD1 (0.0727, 50.62) | FANCA (0.0494, 34.41) | MSH2 (0.0455, 31.70) | SNRPA (0.0341, 23.74) |
| PIK3CA | 439 | CNBP (0.1219, 53.50) | CREB1 (0.0984, 43.20) | PIK3R1 (0.0538, 23.60) | HNRNPK (0.0339, 14.90) | SELENOT (0.0220, 9.65) |
| BRAF | 8 | YWHAG (0.1867, 1.49) | RAF1 (0.1621, 1.30) | CCDC88A (0.1164, 0.93) | YWHAB (0.1162, 0.93) | PAK2 (0.1009, 0.81) |

Partners were looked up in CORUM and UniProt. Six of the ten came back with partners they're
known to work with:

| Gene | Partners in top 5 | Rank |
|---|---|---|
| VHL | CUL2, ELOB, ELOC | 1, 2, 3 |
| MTOR | RICTOR, MAPKAP1, RPTOR | 1, 3, 5 |
| BRAF | YWHAG, RAF1, YWHAB | 1, 2, 4 |
| BRCA1 | BARD1 | 2 |
| EGFR | ERBB2 | 1 |
| PIK3CA | PIK3R1 | 3 |

VHL returns the Elongin–Cullin complex it belongs to, in order. RICTOR and MAPKAP1 are both
mTORC2, with RPTOR from mTORC1 behind them. BARD1 is BRCA1's binding partner, ERBB2 is what EGFR
dimerises with, and PIK3R1 is PIK3CA's regulatory subunit.

The raw and normalised columns disagree about which genes look strong. VHL puts 0.4287 on CUL2,
the largest raw weight in the table, but it only has 7 connections so an even spread would give
0.14 and that's a normalised 3.00. EGFR puts 0.1490 on ERBB2 across 391 connections, a normalised
58.27. On the raw number VHL looks decisive and EGFR doesn't. On the normalised one it's the
other way round.

The full ranking is not the same picture. Ranked by normalised weight, the top 100 is dominated
by a handful of abundant genes as the sending side, PPIA in about thirty of them, then RPL17,
RPS24 and NACA, paired with partners they have no particular relationship to. Real complexes do
appear, SMAD4–SMAD2, SMC3–RAD21, SMARCA4–SMARCC1, NCBP1–NCBP2, AIMP2–KARS1, but they sit among
the rest.

So the per-gene view and the global ranking disagree. Asking what a named gene attends to
returns known partners. Asking which connections in the whole graph got the most weight returns
abundance.

---

## Run 3: kidney-specific labels

Same network and settings as run 2. The label changed. Each gene's average score across all
cancer cell lines is subtracted, so a gene essential everywhere flattens to zero and what's left
is specific to kidney.

The numbers and tables below are from the earlier version of the pipeline, before the readout
was changed to count incoming edges only. Weights here are raw, not normalised, and a partner
can appear twice because each edge was counted in both directions.

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

## Is the kidney signal there at all?


