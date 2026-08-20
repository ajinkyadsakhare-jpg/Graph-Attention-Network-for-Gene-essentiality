# Gene Essentiality with Graph Attention Networks

This project builds a graph attention network to predict gene essentiality in cancer cell lines,
and reads the attention weights afterwards to see which gene relationships the model used.

A graph attention network predicts each gene from its neighbours. But whether a gene is essential
in a particular tumour isn't a property of its neighbours. It is caused by one gene's state
changing what another gene needs to survive, and a single network, the same in every cell line,
cannot hold a relationship like that. So the model reaches the essentiality that is common across
cancers and misses the part specific to one. The runs below show where and why, and that is what
the next version is built from.

The work is in a single lineage, kidney. The lineage was an arbitrary starting point, but
working within one lineage is what makes the context-specific question askable at all.

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
coming out are not. The prediction is a check, not the point. The point is the weights, the
gene relationships the model relied on. So the real question this repository answers is narrower
than "can it predict": it is *can a graph attention network produce readable, context-specific
gene relationships from essentiality*, and what trying reveals turns out to be a structural fact
about the problem itself.

GATDep (Fan et al., 2025) does much the same thing, a graph attention network on DepMap scores,
published while this was underway. It runs on STRING rather than a tissue network, and describes
each gene by which pathways it belongs to rather than by its own expression. It also reads the
attention weights afterwards and picks out partners for a few known cancer genes.

## How to read this repository

| File | What it covers |
|---|---|
| README.md | This file. The idea, what was run, what came out |
| CITATIONS.md | The data and methods this builds on |
| `notebooks/` | The code |
| `results/` | Each run in full: what changed, the numbers, and the attention weights |

Start here, then `results/` if you want the numbers.

## What's inside

One base model, and each experiment is a variation on it. Easier to read as "here's what changed
and what happened" than as separate projects.

- `notebooks/processing/`: turning raw DepMap and IID data into graphs and features. The
  HumanBase network is prepared in `notebooks/experiments/HumanBase/`, next to the run that uses
  it
- `notebooks/training/`: the MLP baseline (no graph) and the base GAT everything else builds
  from
- `notebooks/experiments/`: variations, each changing one thing
- `results/`: the numbers and the attention findings

Run in Google Colab, so some cells handle Colab setup (mounting Drive, installing packages, file
paths).

## The runs

Four, each changing one thing from the one before.

| # | Setup | What changed | Correlation |
|---|---|---|---|
| 1 | MLP | no graph at all | 0.39 |
| 2 | GAT on IID | physical interaction network | 0.5551 |
| 3 | GAT on HumanBase | functional network instead | 0.5191 |
| 4 | GAT on HumanBase | kidney-specific labels | 0.0351 |

Plus one check with no model involved: split-half reliability of the kidney-specific labels,
0.5182.

Correlation here is between predicted and actual essentiality across all genes in a cell line
the model hadn't seen. Higher is better, 1.0 would be perfect. It's a check that the model
learned something worth reading the weights for, not the result itself.

## What came out

**The numbers.** MLP without a graph, 0.39. GAT on IID, 0.5551. GAT on HumanBase, 0.5191. On
kidney-specific labels, 0.0351.

**Changing the network barely changed the score, but changed the weights completely.** IID is
physical interaction, HumanBase is functional association. Almost the same correlation from both.
What differs is where the attention lands.

| Gene | Connections | Top weighted partners |
|---|---|---|
| VHL | 7 | CUL2, ELOB, ELOC |
| BRAF | 8 | YWHAG, RAF1, YWHAB |
| MTOR | 25 | RICTOR, MAPKAP1, RPTOR |
| BRCA1 | 696 | NONO, BARD1, FANCA |

CUL2, ELOB and ELOC are members of the complex VHL belongs to. RPTOR and RICTOR are what define
mTORC1 and mTORC2, and MAPKAP1 is also mTORC2. RAF1 pairs with BRAF, and YWHAB and YWHAG bind
it. BARD1 is BRCA1's binding partner. Partner identities were looked up in CORUM and UniProt.
On the physical interaction network the same genes returned partners with no known relationship
to them.

**Most of what the model predicts is generic.** Many genes are essential in every cancer, not in
kidney in particular. To remove those, each gene's average score across all cancer cell lines
was subtracted, so the label keeps only what's specific to kidney. A gene essential everywhere
flattens to zero, one essential specifically in kidney stands out. The correlation dropped from
0.52 to 0.035.

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

## What the field shows

The choices here, and the finding, sit against a few results from other work.

**The signal is real but hard to reach.** shinyDepMap (Shimada & Mitchison) finds context-specific
essentiality genuine and reproducible, and finds the selective genes are only *moderately*
essential, not the deep scores of common-essential genes. MAHI is weakest exactly on the
context-specific genes and strongest on the common ones. A model reaching the common signal and
missing the specific one is the known hard edge of the problem, not a one-off.

**Models across all lineages score higher because they ride lineage.** The largest, easiest signal
across a pan-cancer panel is which tissue a line came from. Single-lineage work removes that and
is harder, which is why a within-kidney number looks lower than pan-cancer papers, and why a
single lineage is the honest place the reorganization signal actually lives.

**Biology enters a model in one of two places, and it is chosen to match what the question is
asking.** In HELP and GATDep it enters through the *features*, which pathways a gene is in and
where it localises. In KG-SLomics (Lee & Nam) it enters through the *structure*, a heterogeneous
graph built to mirror the biology: its target is a synthetic-lethal *pair*, a mediated
relationship, so it uses a pair readout and intermediate nodes to route between the two genes.
The architecture is built to match the relationship it is predicting. The finding here is the
same idea from the failing side: a node-level, symmetric model has no way to express a directed,
conditional relationship.

**The attention is often not what carries the prediction.** GATDep's plain graph-convolution
matches its attention; KG-SLomics without attention nearly matches its full model. So reading
meaning off attention weights is fraught, because the weights may not be load-bearing. This
repository sees the same split, readable per-gene and abundance globally.

**The closest prior work rewires the network per line, but doesn't learn the rewiring.** DependANT
personalises interaction networks per cell line using mutation and expression, predicts
cell-line-specific dependencies, and helps most on selective genes, but as a random forest on
hand-computed network features, not a model that learns the rewiring, because the sample size
won't support learning it. That is the line the next version works along.

## The finding

The result isn't the 0.035. It is *why* the 0.035 happens, and it is structural. The model
predicts a gene from its neighbours, and the thing it is trying to predict does not work that
way.

Context-specific essentiality is a **directed, conditional, pathway-mediated** relationship. A
gene becomes essential *because* something happened to another gene (directed), *only in a cell
where it happened* (conditional), and the effect travels through the pathway the two share
(pathway-mediated). A graph attention network on a fixed undirected network is none of those. It
computes, for each gene, an average over its neighbours: **symmetric** (no direction), **static**
(the same graph in every cell line, so nothing conditional on state), and **homophily-assuming**
(a gene is predicted to resemble its neighbours). Three properties of the calculation, each the
opposite of a property of the biology. The model was never going to express the relationship,
because what it computes is the reverse of the biology at every one of these points.

The runs pin this down at three levels, from surface to structural:

- **Homophily is violated.** A graph attention network works when a gene is like the genes around
  it. Essentiality isn't always like that. Two genes in the same complex can have opposite
  scores, one carries the function and the other can be lost with little effect. It fits only
  where a whole group is essential together (ribosome, proteasome, spliceosome). This is why the
  per-gene attention recovers real partners for genes like VHL and MTOR but the *global* ranking
  returns abundance: the model latches onto the homophilous parts and has nothing to say about
  the rest.

- **The graph is line-invariant, and the baseline is exactly what it can carry.** The same network
  is used for all 32 cell lines; only two numbers per gene change between them. So everything the
  graph contributes is identical in every line, which is the gene-intrinsic *baseline*, not
  anything kidney-specific. Subtracting each gene's average (run 3) removes precisely the part a
  shared graph can explain, and the score collapses from 0.52 to 0.035. The collapse isn't the
  signal being absent. The split-half check shows the kidney signal is real (0.68). It is the
  model being structurally blind to it, because a line-invariant graph cannot produce a
  line-specific answer.

- **The direction and the condition are unrepresentable.** Even with a per-line graph, a
  *symmetric undirected* edge cannot hold "A's loss makes B essential" (which is directed), and a
  *static* edge cannot hold "only in the state where A is lost" (which is conditional). These
  aren't tuning problems; they are things the object cannot express.

The features are thin too, and this compounds it rather than causing it. Two numbers per gene,
and mutation is sparse enough that expression carries most of it, and other groups report the
same. GATDep (Fan et al., 2025), a GAT on the same DepMap labels, added mutation and copy number
to a much larger feature set, found no meaningful gain, and settled on the expression-only
version. DeepDEP's authors found their expression-only variant matched the full multi-omic model.
So the feature ceiling is real, but it is not why the kidney-specific signal is unreachable. The
mismatch above is.

The choices here, one fixed network, two numbers per gene, a symmetric aggregation, came from
what the data allowed, not from working out first how context-specific essentiality actually
comes about. That is the lesson the project produces, and it is a precise one.

A few parameter changes didn't move it.

## Where this goes

The next version is built around the biology, not patched with more features. Everything the
model gets is a *state*, what's in the cell right now. Essentiality is what happens *after* you
take a gene out. Holding that requires what this model doesn't have: a graph that changes per
cell line rather than one fixed for all of them (so it can be *conditional* on state), driven by
the mutation as the *event* that reshapes it (so it can be *directed*, cause to consequence),
with the pathway as the structure the effect travels through (so it can be *pathway-mediated*).
That is what the diagnosis here points to, and what makes it groundwork rather than a dead end.

## Limits

The ranking of all connections in the graph isn't usable on its own. Genes with few connections
are favoured by construction, because the weights on a gene's connections have to add up to 1.
The list is also full of paralog pairs. Only the per-gene view above avoids this.

The partner check is a look, not a test. Ten genes were picked from the COSMIC Cancer Gene
Census and their partners looked up. What would settle it is an enrichment test against CORUM
using a comparison that matches on number of connections. That hasn't been run.

Each edge is stored in both directions and gets a weight in each, so the same partner can turn
up twice in the raw output with two different values. The tables above list each partner once.

The IID run stores each edge one way only. Both directions wouldn't fit in memory at that edge
count. So each pair passes information in whichever direction the file happened to list it,
which is arbitrary. The HumanBase run stores both.

One of the IID edge features is coexpression, worked out across all 32 cell lines before the
data was split for cross-validation. So the edges carry a little information from the held-out
lines.

The IID run carries two edge features to HumanBase's one. That's a difference between the two
runs beyond the network itself.

## What's here and what isn't

Code and results are here. Data isn't, DepMap has its own terms and the files are large, so
`data/data_README.md` has links to download everything from the original sources. There's a
`requirements.txt` for the environment.

This is diagnostic groundwork, not a finished pipeline. It was run to find out what the problem
actually is, and it did, so some parts are deliberately left where they landed once they'd made
their point.
