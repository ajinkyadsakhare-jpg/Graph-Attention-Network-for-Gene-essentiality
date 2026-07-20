# Gene Essentiality with Graph Attention Networks

This project predicts gene essentiality in cancer cell lines using a graph attention network,
and then looks at what the model pays attention to. It's modelled around kidney, which was a
random choice, not by design.

## The idea

Gene essentiality means how important a gene is for the survival of a cell. If a gene is
essential, removing it kills the cell. This gets measured by knocking out every gene one at a
time across hundreds of cancer cell lines, which is what the DepMap project does. The result is
a score for every gene in every cell line, and the more negative the score, the more the cell
depended on that gene.

Which genes are essential isn't fixed. It changes between tissues, and between one tumour and
another. That's why these screens are used to look for drug targets, since a gene a tumour needs
and normal tissue doesn't is worth targeting.

Existing models predict this reasonably well. HELP (Granata et al.) and MAHI, from the
Troyanskaya lab, both combine several kinds of information about each gene, including a gene
network.

A gene network here means a map of which genes are connected to which. Two kinds show up in this
project. A protein-protein interaction network, where an edge means two proteins physically
bind. And a functional network, where an edge means two genes are predicted to work together,
whether or not they touch. HumanBase publishes functional networks for each tissue separately.

But these models are precise. Are they rich? In HELP, most of the prediction comes from around
3,000 features describing which compartment of the cell each protein ends up in. The network
contributes very little. So the network is in the model, but it isn't doing the work.

That's the gap. What happens if the graph has to carry the prediction on its own?

A graph attention network is a model that works directly on a network. For each gene, it looks
at that gene's neighbours and decides how much to rely on each one, and it learns those
decisions from the essentiality data. Those decisions are stored as a weight on every edge, and
they can be read afterwards.

Which means the network going in is generic, the same edges for every cell line, and the weights
coming out are not. The prediction isn't the point. The point is seeing what the model thinks
matters for survival.

## How to read this repository

| File | What it covers |
|---|---|
| README.md | This file. The idea, what was run, what came out |
| METHOD.md | What the model computes, stated formally |
| RESULTS.md | Each run in full: what changed, the numbers, and the attention weights |
| CITATIONS.md | The data and methods this builds on |
| `notebooks/` | The code |

Start here. METHOD next if you want to know how the model works, RESULTS if you want the
numbers.

## What's inside

One base model, and each experiment is a variation on it. Easier to read as "here's what changed
and what happened" than as separate projects.

- `notebooks/processing/` — turning raw DepMap and IID data into graphs and features. The
  HumanBase network is prepared in `notebooks/experiments/HumanBase/`, next to the run that uses
  it
- `notebooks/training/` — the MLP baseline (no graph) and the base GAT everything else builds
  from
- `notebooks/experiments/` — variations, each changing one thing
- `results/` — the numbers and the attention findings

Run in Google Colab, so some cells handle Colab setup (mounting Drive, installing packages, file
paths).

## The runs

Four, each changing one thing from the one before.

| # | Setup | What changed | Correlation |
|---|---|---|---|
| 1 | MLP | no graph at all | 0.39 |
| 2 | GAT on IID | physical interaction network | 0.5321 |
| 3 | GAT on HumanBase | functional network instead | 0.5377 |
| 4 | GAT on HumanBase | kidney-specific labels | 0.0351 |

Plus one check with no model involved: split-half reliability of the kidney-specific labels,
0.5182.

Correlation here is between predicted and actual essentiality across all genes in a cell line
the model hadn't seen. Higher is better, 1.0 would be perfect. It's a check that the model
learned something worth reading the weights for, not the result itself.

## What came out

**The graph does work.** Without it, 0.39. With it, 0.54. So the network is carrying something
the two features per gene aren't.

**Changing the network barely changed the score, but changed the weights completely.** IID is
physical interaction, HumanBase is functional association. Almost the same correlation from both.
What differs is where the attention lands.

| Gene | Connections | Top weighted partners |
|---|---|---|
| VHL | 12 | PKD1L3, CUL2, ELOC, ELOB |
| BRAF | 14 | RAF1, YWHAB, YWHAG |
| MTOR | 48 | RPTOR, RICTOR |
| BRCA1 | 1390 | ZNF350, BRIP1, BRAP, MSH5 |

CUL2, ELOC and ELOB are members of the complex VHL belongs to. RPTOR and RICTOR are what define
mTORC1 and mTORC2. RAF1 pairs with BRAF, and YWHAB and YWHAG bind it. Partner identities were
looked up in CORUM and UniProt. On the physical interaction network, these genes didn't return
their known partners.

**Most of what the model predicts is generic.** Many genes are essential in every cancer, not in
kidney in particular. To remove those, each gene's average score across all cancer cell lines
was subtracted, so the label keeps only what's specific to kidney. A gene essential everywhere
flattens to zero, one essential specifically in kidney stands out. The correlation dropped from
0.54 to 0.035.

At first this looked like the kidney signal being too sparse to learn. The label was checked
before training and PAX8, a known kidney lineage dependency, sits clearly below zero, so
something was there. But nearly all the other values sat near zero, half of them within 0.1 of
it.

**Checking whether the signal exists at all.** The 32 kidney cell lines were split into two
halves. Each half was used separately to work out which genes look kidney-specific, and the two
answers were compared. If it were noise the halves would disagree. Over 20 random splits they
agree at 0.52, which works out to 0.68 for the full 32 lines.

So the signal is real and reproducible, and the model couldn't reach it. That's a different
problem from the signal being absent.

## Why it stopped there

The features are thin. Expression and damaging mutations, two numbers per gene per cell line.
Mutation data is sparse so expression dominates, and the network's edge score is really just the
backbone of the graph. The model leaning toward housekeeping genes is expected.

The graph is also the same for all 32 cell lines. Only those two numbers change between them.
So everything the graph contributes is the same in every cell line, and subtracting each gene's
average removes exactly the kind of thing that can explain. METHOD.md works this through
properly.

A few parameter changes didn't move it.

## Limits

The ranking of all connections in the graph isn't usable on its own. Genes with few connections
are favoured by construction, because the weights on a gene's connections have to add up to 1.
The list is also full of paralog pairs. Only the per-gene view above avoids this.

The partner check is a look, not a test. Ten genes were picked from the COSMIC Cancer Gene
Census and their partners looked up. What would settle it is an enrichment test against CORUM
using a comparison that matches on number of connections. That hasn't been run.

Each edge is stored in both directions and gets a weight in each, so the same partner can turn
up twice in the raw output with two different values. The tables above list each partner once.

The IID run carries two edge features to HumanBase's one. That's a difference between the two
runs beyond the network itself.

## What's here and what isn't

Code and results are here. Data isn't, DepMap has its own terms and the files are large, so
`data/data_README.md` has links to download everything from the original sources. There's a
`requirements.txt` for the environment.

This is a work in progress, not a finished pipeline, so some parts are rougher than others.
