Citations and acknowledgments

This project builds on data and methods from the following work.

Methods

Graph Attention Networks (GAT)
Veličković P, Cucurull G, Casanova A, Romero A, Liò P, Bengio Y. (2018).
Graph Attention Networks. International Conference on Learning Representations (ICLR).
arXiv:1710.10903

GATv2
Brody S, Alon U, Yahav E. (2022). How Attentive are Graph Attention Networks?
International Conference on Learning Representations (ICLR). arXiv:2105.14491

PyTorch Geometric
Fey M, Lenssen JE. (2019). Fast Graph Representation Learning with PyTorch Geometric.
ICLR Workshop on Representation Learning on Graphs and Manifolds. arXiv:1903.02428

Data

DepMap / Chronos (CRISPR gene effect scores)
Dempster JM, Boyle I, Vazquez F, Root DE, Boehm JS, Hahn WC, Tsherniak A, McFarland JM.
(2021). Chronos: a cell population dynamics model of CRISPR experiments that improves
inference of gene fitness effects. Genome Biology 22:343. doi:10.1186/s13059-021-02540-7
DepMap data: Broad Institute, https://depmap.org

HumanBase — tissue-specific functional networks
Greene CS, Krishnan A, Wong AK, Ricciotti E, Zelaya RA, Himmelstein DS, Zhang R,
Hartmann BM, Zaslavsky E, Sealfon SC, Chasman DI, FitzGerald GA, Dolinski K, Grosser T,
Troyanskaya OG. (2015). Understanding multicellular function and disease with human
tissue-specific networks. Nature Genetics 47(6):569-576. doi:10.1038/ng.3259

IID — Integrated Interactions Database (tissue-specific PPI)
Kotlyar M, Pastrello C, Sheahan N, Jurisica I. (2016). Integrated interactions database:
tissue-specific view of the human and model organism interactomes.
Nucleic Acids Research 44(D1):D536-D541. doi:10.1093/nar/gkv1115

Related frameworks

HELP (the framework this work started from and compares against)
Granata I, Maddalena L, Manzo M, Guarracino MR, Giordano M. (2024). HELP: A computational
framework for labelling and predicting human common and context-specific essential genes.
PLoS Computational Biology 20(9):e1012076. doi:10.1371/journal.pcbi.1012076

MAHI (tissue-aware GNN for gene essentiality, compared against)
Aggarwal A, Sokolova K, Troyanskaya OG. (2026). Multi-modal tissue-aware graph neural network
for in silico genetic discovery. bioRxiv (RECOMB 2026). doi:10.64898/2026.02.17.706433

Acknowledgments

This work uses data and tools from the Broad Institute (DepMap), the Flatiron Institute and
Princeton (HumanBase), and the University of Toronto (IID). The graph attention methods are
due to Veličković et al. and Brody et al., implemented with PyTorch Geometric.
