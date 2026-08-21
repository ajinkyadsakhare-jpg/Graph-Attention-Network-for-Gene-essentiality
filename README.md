# Gene Essentiality with Graph Attention Networks

Gene Essentiality means how much a gene is necessary for the cell to survive in cancer cell lines.

There are a lot of prediction models that are able to predict the essential genes by integrating gene network, but the aim of this project is to think about the gene interactions the model is relying on to predict the essentiality.

These gene interactions vary across lineages so to limit that variation,the model is entirely made around one lineage. The goal is to have context specific interactions of genes.

Existing models that were mentioned above are (HELP, MAHI & GATDep, 2025),(GATDep specifically does much the same, and appeared while this was underway.)

## The runs

There were four runs each with their addition, An MLP with no graph reaches 0.39; a physical protein-protein interaction network from (IID) lifts it to 0.56; a functional (HumanBase) network holds it at 0.52.

Each were trained on kidney-specific labels and the final one where each gene's pan-cancer average subtracted which collapses to 0.035, There seems to be signal but the model isn't reaching it.

The weights say the same, per gene they're right (VHL returns its complex, MTOR returns mTORC1 and mTORC2 subunits), but ranked globally they just return whatever is abundant.(includes the housekeeping genes, which is why we took differentiated label).

## The finding

The model predicts essentiality, but the goal was to read the attention weights and see which interactions the model used to make those predictions.

Since the goal depends on which genes are essential and which are not, Pearson's correlation was used, the same metric used by HELP and GATDep.

The model with no graph network reached 0.39. Of the four runs, the physical protein-protein interaction network and the HumanBase functional network gave correlations of around 0.56 and 0.52 respectively, both using raw GeneEffect scores as the label.

GeneEffect also carries a mixed signal from housekeeping genes, so a further run used a differential label, each gene's average subtracted, to remove that common part, and this gave around 0.035. That points towards the model not being able to capture the cell-line-specific part.

Even a somewhat better architecture would not probably fix this, the direction now is how to get the signal to the model in the first place. But the size of the drop itself suggests a complex signal that may or may not be findable, since it could also just be noise.

## Where it goes

Adding a gene network raised the correlation on the raw label but not on the differential one, so the interactions the model used carry the common part of essentiality, not the cell-line-specific part.

Reading those interactions was the goal, and the correlation is too low to trust them for it.

That low correlation comes down to the architecture like the network's high node degree meaning GATConv averages each gene over too many neighbors, and that aggregation blends the cell-line-specific signal away. So this is a limit of the approach as built, not one to read too much into.
