# Gene Essentiality with Graph Attention Networks

A graph attention network predicts gene essentiality (how much a cell needs each gene) in cancer cell lines, read afterwards for the gene relationships it used — the attention weights, not the prediction, are the point. Everything is in one lineage, kidney, which is what makes the context-specific question askable at all.

Existing models (HELP, MAHI) predict essentiality well but include a gene network that barely contributes. The question here is what happens when the graph has to carry the prediction alone. (GATDep, 2025, does much the same, and appeared while this was underway.)

## The runs

Four, each changing one thing. An MLP with no graph reaches 0.39; a physical interaction network (IID) lifts it to 0.56; a functional one (HumanBase) holds it at 0.52. Trained on kidney-specific labels — each gene's pan-cancer average subtracted — it collapses to 0.035, while a split-half check puts the kidney signal's reliability near 0.68. The signal is real; the model isn't reaching it. The weights say the same: per gene they're right (VHL returns its complex, MTOR returns mTORC1 and mTORC2 subunits), but ranked globally they just return whatever is abundant.

## The finding

The result isn't the 0.035, it's why. Context-specific essentiality is directed, conditional, and pathway-mediated — a gene becomes essential because something happened to another gene, only in the cell where it happened, through the pathway they share. A GAT on a fixed undirected network is the opposite at every point: it averages a gene's neighbours (symmetric), from the same graph in every line (static), assuming a gene resembles them (homophily). So it can only carry the gene-intrinsic baseline — exactly the part that vanishes when the average is subtracted — and is blind to what is specific to one line. Thin features compound this but don't cause it. The model was built from what the data allowed, not from how context-specific essentiality actually arises; that is the lesson.

## Where it goes

Everything the model sees should be a state — what's in the cell now — with essentiality as what follows removing a gene. That needs a graph that changes per line (conditional), driven by mutation as the event that reshapes it (directed), with the pathway as what the effect travels through. DependANT, which rewires networks per line but with hand-built rather than learned features, is the closest existing work.

## Repository

`notebooks/` (processing, the MLP baseline, the base GAT, one variation per experiment), `results/` (numbers and attention weights per run), `CITATIONS.md`. Runs in Colab; data isn't included — DepMap's terms, large files — but `data/data_README.md` links to the sources. This is diagnostic groundwork: the global attention ranking isn't usable, the partner check is a look not a test, and the two network runs differ by more than the network.
