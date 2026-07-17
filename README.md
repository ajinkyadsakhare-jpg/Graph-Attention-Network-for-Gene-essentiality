# Gene Essentiality with Graph Attention Networks

This project predicts gene essentiality in cancer cell lines using a graph attention network, and then looks at what the model pays attention to. It's modelled around kidney, which was a random choice, not by design.

## The idea

Gene essentiality means how important a gene is for the survival of a cell. If a gene is essential, removing it results in cell death. Existing models like HELP (Granata et al) and MAHI (Troyanskaya lab) can predict gene essentiality reasonably well, HELP specifically for use as gene targets for gene therapy. These models use graph embeddings stacked with other features to get a precise result.

But the models are precise, are they rich? For HELP, around 80% of the prediction power comes from CCcfs (which organelle the protein ends up in), around 3k features, while the network embedding contributes only about 5%. So what if the model is given the freedom to work out essentiality on its own, and learn which genes are essential in an organ? The prediction itself isn't the point here. The point is seeing what the model thinks matters for survival.

## What's inside

The repo is built around one base model, with each experiment being a variation on it, so it's easier to read them as "here's what changed and what happened" rather than as separate projects.

- `notebooks/processing/` — turning the raw DepMap, IID, and HumanBase data into the graphs and features.
- `notebooks/training/` — the MLP baseline (no graph) and the base GAT that everything else is built from.
- `notebooks/experiments/` — variations on the base GAT, each changing one thing (the graph, the label, the tissue, the edges).
- `results/` — the numbers and the attention findings.

The notebooks were run in Google Colab, so some cells handle Colab-specific setup (mounting Drive, installing packages, and file paths).

## What I found

The short version is that the graph does help, but the modelling can be made more specific to find the cancer-specific signal. The signal is there, it's just small. Most of what the model picked up was generic housekeeping genes, along with some genuine signal and some noise. The correlation came out around 0.53, and I think I understand why.

Two reasons. First, the features. The model uses expression, somatic mutations, and HumanBase's posterior score (basically a functional-relatedness score between genes). Expression ends up being a very strong signal because the mutation data is sparse, and the posterior score is really just the backbone of the graph and a direction. So the model leaning toward housekeeping is expected.

Second, I ran a few experiments to test the other factors. One was differential labeling, where I tried to focus on genuine cancer-specific signal by removing the generic essentiality. Instead of using each gene's raw essentiality score, I subtracted its average score across all cancer cell lines, so the label kept only what's specific to kidney. A gene that's essential everywhere flattens out, and a gene that's essential specifically in kidney stands out. The signal was there (PAX8, a known kidney lineage dependency, showed up as expected), but all the values ended up near zero, because the specific signal is genuinely sparse and the range shrank so much there was little left to learn from.

I also tried increasing the dataset by using a different organ, lung, which had a larger and richer dataset, but the results were about the same. A few other parameter changes didn't move it much either.

So what I end up with is: the signal is there, but sparse. A better model design might do a better job of picking it out, and that's what I'm working on now.

## What's here and what isn't

The code and results are here. The data isn't, DepMap has its own terms and the files are large, so `data/README.md` has links to download everything from the original sources. There's a `requirements.txt` for the environment.

This is a work in progress, not a finished pipeline, so some parts are rougher than others.
