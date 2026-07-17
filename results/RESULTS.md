# Results

Prediction accuracy and attention weights, as they came out. The attention weights are the point, so they are shown in full for the current model.

---

## Summary

| Experiment | Network | Pearson r | Outcome |
|---|---|---|---|
| MLP baseline | none | ~0.39 | Reference (no graph) |
| GATv1 base | IID (physical PPI) | 0.5108 ± 0.0602 | Graph beats baseline; attention mostly spurious partners |
| Filtered edges (coexpr ≥ 0.2) | IID, pruned | 0.3640 | Worse; pruning weak edges broke connectivity |
| Differential labeling | IID | — | Training collapsed; label was sound (PAX8) but signal too sparse |
| GATv1 HumanBase | HumanBase (functional) | 0.5253 ± 0.0208 | Matches IID; recovers real partners IID missed |

The move from IID to HumanBase is the main result: similar prediction accuracy, but the functional network surfaces real gene-gene partners that the physical network does not.

---

## GATv1 — IID Base (Physical PPI)

Node features: expression + somatic mutation. Edge feature: PPI confidence. Heads = 4. Unidirectional graph.

### Prediction

| Fold | Pearson r |
|---|---|
| 1 | 0.3972 |
| 2 | 0.5139 |
| 3 | 0.5491 |
| 4 | 0.5704 |
| 5 | 0.5237 |
| **Mean** | **0.5108 ± 0.0602** |

### Attention Weights — Selected Cancer Genes

Top partner per gene. Real partners largely absent.

| Gene | Edges | Top partner | Attn |
|---|---|---|---|
| VHL | 782 | ASTL | 0.2834 |
| TP53 | 3,317 | ZNF679 | 0.5676 |
| EGFR | 2,821 | GPR149 | 0.5763 |
| MTOR | 594 | ARSL | 0.0827 |
| BRCA1 | 1,610 | LMNTD1 | 0.3673 |
| KRAS | 1,611 | PRR15L | 0.6048 |

The top partners are mostly spurious. This is the physical PPI network: functional partners are not the strongest edges.

---

## Filtered Edges — Coexpression ≥ 0.2

Weak coexpression edges removed from the IID network.

| Model | MSE | Pearson r |
|---|---|---|
| GAT | 0.1915 | 0.3640 |
| MLP | 0.1916 | 0.0763 |

Both drop sharply. Removing weak edges broke graph connectivity. No attention extracted (results too poor).

---

## Differential Labeling

Label re-centered by subtracting each gene's mean essentiality across all cancer cell lines, to isolate kidney-specific dependency.

| Sanity check | Value | Reading |
|---|---|---|
| PAX8 global mean | -0.1766 | Kidney lineage confirmed (strong negative differential) |
| HIF1A global mean | 0.1288 | Near zero — poor test gene, no clear mutant vs wild-type separation |

Training collapsed and never converged, so no Pearson r or fold results. The label itself was sound (PAX8 behaved as expected), but the differential values clustered too close to zero — the cancer-specific signal is genuinely sparse.

---

## GATv1 — HumanBase Functional Network (Current Model)

| Setting | Value |
|---|---|
| Node features | Expression + somatic mutation |
| Edge feature | Functional-association posterior (HumanBase) |
| Genes | 10,716 |
| Cell lines | 32 (kidney) |
| Validation | 5-fold cross-validation |

### Prediction

| Fold | Pearson r |
|---|---|
| 1 | 0.5321 |
| 2 | 0.5430 |
| 3 | 0.4899 |
| 4 | 0.5151 |
| 5 | 0.5464 |
| **Mean** | **0.5253 ± 0.0208** |
| MLP baseline (no graph) | ~0.39 |

### Attention Weights — Known Cancer Genes

Top attention partners per gene. Actual output, nothing removed.

| Gene | Edges | Partner 1 | Partner 2 | Partner 3 | Partner 4 | Partner 5 |
|---|---|---|---|---|---|---|
| VHL | 12 | PKD1L3 (0.4580) | MYL12A (0.1745) | CCDC82 (0.1519) | ELOB (0.1477) | CUL2 (0.0969) |
| TP53 | 434 | CUL9 (0.3516) | CCL18 (0.1849) | THAP8 (0.1743) | XRCC1 (0.1670) | MAGEB18 (0.1469) |
| EGFR | 780 | GYPB (0.5253) | EGF (0.4896) | KRT14 (0.4827) | DOK6 (0.4760) | SHC3 (0.4642) |
| PTEN | 654 | MAGI3 (0.1956) | DLL1 (0.1429) | GPSM1 (0.0496) | IGBP1 (0.0337) | FBXO33 (0.0318) |
| MTOR | 48 | RICTOR (0.1331) | RPTOR (0.0962) | STAT1 (0.0831) | YWHAQ (0.0523) | YWHAZ (0.0439) |
| MYC | 690 | ZNF121 (0.3997) | RPL11 (0.1767) | ZBTB17 (0.0697) | MCM7 (0.0511) | RPSA (0.0501) |
| KRAS | 650 | CALM1 (0.0244) | HIRA (0.0215) | HIRA (0.0178) | ZCCHC2 (0.0173) | CCNH (0.0151) |
| BRCA1 | 1390 | ZNF350 (0.3602) | BRIP1 (0.2836) | BRAP (0.2254) | MSH5 (0.1926) | DCLRE1A (0.1674) |
| PIK3CA | 876 | CCDC126 (0.0352) | SQSTM1 (0.0348) | HCFC2 (0.0278) | EEF1D (0.0238) | RAB2B (0.0209) |
| BRAF | 14 | YWHAG (0.2100) | YWHAQ (0.1099) | YWHAZ (0.0912) | RAF1 (0.0772) | YWHAB (0.0707) |

Attention magnitude tracks node degree: low-degree genes concentrate attention (VHL, 12 edges, 0.46; BRAF, 14 edges, 0.21), high-degree hubs dilute toward noise (KRAS, 650 edges, 0.024; PIK3CA, 876 edges, 0.035).

### Real Partners Recovered

| Gene | Real partner(s) in top 5 | Rank |
|---|---|---|
| MTOR | RICTOR, RPTOR | 1, 2 |
| BRCA1 | BRIP1, MSH5 | 2, 4 |
| EGFR | EGF | 2 |
| BRAF | RAF1, YWHA proteins | 1, 4 |
| VHL | CUL2, ELOB | 4, 5 |

Compared with the IID network above, where none of these real partners appear, the functional network recovers them.

### Global Top Interactions (Averaged Over Folds)

| Rank | Interaction | Attention |
|---|---|---|
| 1 | EWSR1 ↔ SLC22A24 | 0.6013 |
| 2 | CDC20 ↔ KRTAP19-5 | 0.5953 |
| 3 | FLNA ↔ DRD3 | 0.5921 |
| 4 | RAB2A ↔ GARIN6 | 0.5858 |
| 5 | UBE2I ↔ IKZF3 | 0.5852 |

Hub genes recur across the top 100 (EWSR1, UBE2I, RBPMS several times each). Genuine pairs appear lower in the list: MAX–MYCN, MSH5–MSH4, FZD1–WNT3A, HSPB8–HSPB7.

---

## Notes

- Graph beats the MLP baseline (0.53 vs 0.39).
- Functional network (HumanBase) recovers real partners that physical PPI (IID) does not.
- Real partners are recovered but rarely top-ranked.
- Attention magnitude is confounded with node degree.
- Cancer-specific signal is real (PAX8) but sparse (differential collapse).
- Next: degree-normalize attention, then test high-attention edges against CORUM/KEGG with a degree-matched null.
