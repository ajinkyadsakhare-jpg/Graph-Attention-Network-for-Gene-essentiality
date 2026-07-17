# Data

The data isn't stored in this repo (the files are large and DepMap has its own terms of
use). Everything can be downloaded from the original sources below. Once downloaded, place
the files under `data/raw/` and run the processing notebooks in `notebooks/processing/` to
generate the per-tissue graphs and features.

## Sources

| Source | What it provides | Files used | Version |
|---|---|---|---|
| DepMap | CRISPR gene effect scores (labels), expression, somatic mutations | `CRISPRGeneEffect.csv`, `OmicsExpressionTPMLogp1HumanProteinCodingGenes.csv`, `OmicsSomaticMutationsMatrixDamaging.csv`, `Model.csv` | 25Q3 |
| HumanBase | Kidney tissue-specific functional network (edge posteriors) | kidney top-edges file | — |
| IID | Kidney tissue-specific PPI network (used in the earlier IID experiments) | `human_annotated_PPIs.txt` | — |

Download links:
- DepMap: https://depmap.org/portal/data_page/
- HumanBase (tissue networks): https://humanbase.flatironinstitute.org/
- IID: https://iid.ophid.utoronto.ca/

## Processing notes

- The HumanBase kidney network is thresholded at posterior ≥ 0.2 before use.
- Edges are made bidirectional (each undirected edge appears in both directions) so message
  passing flows both ways.
- DepMap, the network, and the features are all intersected to a common gene set before the
  graph is built.

See the notebooks in `notebooks/processing/` for the exact steps.
