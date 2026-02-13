# Single Sample Gene Set Enrichment Analysis OmniBenchmark framework

This repo contains the main specification yaml to run the benchmarking framework used in the "Comparison of single sample Gene Set Enrichment Analysis methods for functional analysis in research and diagnostics" study by Robinson et al.

## Overview
This benchmarking study compares the performance of 17 single-sample GSEA methods on a diverse set of datasets with known biology and test them on a matched collection of gene signatures, with relevance to both experimental research and clinical diagnostics. All methods were run with a universal reference from normal tissue to adapt the methods to solve the N-of-1 problem.

The implementation in Omnibenchmark invites and facilitates continuous additions of relevant tools and datasets.

The methods can be grouped in three classes based on input type provided:
- RankExpr: This input type is simply the rank of gene expression values, normally used to run truly single sample (class III) tools;
- RankReference: This input type consists of both the RankExpr input and the fully ranked reference matrix, used to validate the quasi-single sample (class II) tools;
- DeltaCentroid: This input type consists of the rank differences between the RankExpr input type and the rank centroid of the reference, analogous to the classical GSEA approach (class I).

## How to run
1. Install [Omnibenchmark](https://docs.omnibenchmark.org/latest/howto/#install-omnibenchmark)
2. Install [Apptainer](https://github.com/apptainer/apptainer/blob/main/INSTALL.md) (currently only supports running with singularity images)
3. Clone this repository `git clone git@github.com:cphgeno/OB_GSEA-main.git`
4. Run locally `ob run benchmark -b OB_GSEA-specification.yaml --local-storage`

The pipeline is set up to run with all publicly available datasets used in the study, but can be modified to include new datasets and gene signatures by providing the `input_dir` argument in the specification yaml for the corresponding data entry.

## Outputs
The pipeline outputs for each method the score obtained for that analysis, a heatmap of said scores, as well as fully computed metrics. It also produces a sina plot of MCC scores distribution (`out/Metricsplotting/mcc_sina_plot.png`) across all analyses for all methods, divided by method class, with additional metrics plotting.

---

## Summary
- Data collection
    - https://github.com/cphgeno/OB_GSEA-data_collection
- Preprocessing of input data
    - https://github.com/cphgeno/OB_GSEA-preprocessing
- Methods
    - fGSEA - https://github.com/cphgeno/OB_GSEA-fGSEA_scoring
        - args: ["--input_type", "DeltaCentroid"]
        - args: ["--input_type", "RankExpr"]
    - GSVA - https://github.com/cphgeno/OB_GSEA-GSVA_scoring
        - ["--algorithm", "gsva", "--input_type", "RankExpr"]
        - ["--algorithm", "gsva", "--input_type", "DeltaCentroid"]
        - ["--algorithm", "gsva", "--input_type", "RankReference"]
        - ["--algorithm", "plage", "--input_type", "RankExpr"]
        - ["--algorithm", "plage", "--input_type", "DeltaCentroid"]
        - ["--algorithm", "plage", "--input_type", "RankReference"]
        - ["--algorithm", "zscore", "--input_type", "RankExpr"]
        - ["--algorithm", "zscore", "--input_type", "DeltaCentroid"]
        - ["--algorithm", "zscore", "--input_type", "RankReference"]
    - ssGSEA - https://github.com/cphgeno/OB_GSEA-ssGSEA_scoring
        - args: ["--input_type", "RankExpr"]
        - args: ["--input_type", "DeltaCentroid"]
    - singscore - https://github.com/cphgeno/OB_GSEA-singscore_scoring
        - args: ["--input_type", "RankExpr"]
        - args: ["--input_type", "DeltaCentroid"]
    - UCell - https://github.com/cphgeno/OB_GSEA-UCell_scoring
        - args: ["--input_type", "RankExpr"]
        - args: ["--input_type", "DeltaCentroid"]
- Metrics
    - top1 validation - https://github.com/cphgeno/OB_GSEA-top1validation
    - AUC - https://github.com/cphgeno/OB_GSEA-AUCvalidation
- Metrics collector
    - https://github.com/cphgeno/OB_GSEA-MetricsCollector

---

## Data and Tool integration
### Adding new dataset/gene signatures
1. In the "data" stage, create a new module, named after your dataset, pointing to the same data collection repository. In the parameters, include the gene set collection against which to run, plus the "input_dir" parameter specifying where the files are located. Ensure to have three files for each new input combination:
    - counts .tsv file, containing a Geneid column and a column for each sample, e.g. <dataset1-counts.tsv>
    - gene set collection .gmt file <gene_signature1.gmt>
    - metadata .tsv file, with the following naming structure <dataset1-gene_signature1.tsv>, with a filename column (sample identifier in counts file), an annotation column (feature of interest) and a groundtruth column (gene set mapped to the annotation).

2. (Optional) Specify the reference data to be used if not the universal reference, i.e. same cohort reference, with the ["--wreference", "VSdf"] arguments.

3. Add the respective data to the specification yaml file, e.g.
```
    id: dataset1
        name: "dataset1 description"
        software_environment: "R_base"
        repository:
            url: https://github.com/cphgeno/OB_GSEA-data_collection.git
            commit: main
        parameters:
            - values: ["--genesets", "gene_signature1", "--input_dir", "path/to/input_dir"] # against universal reference
            - values: ["--genesets", "gene_signature1", "--input_dir", "path/to/input_dir", "--wreference", "VSdf"] # same cohort reference
```

### Adding new tool
1. Create a repository for the tool, structured as one of the Methods listed in the [Summary](#summary) section.
    - It must take as input the same input types and output the scores with the same format and naming.
    - If the new tool is a wrapper for multiple scrogin approaches, see the [gsva module repository](https://github.com/cphgeno/OB_GSEA-GSVA_scoring), allowing to specify the algorithm/tool to be used
2. Add the method in the "ScoringTools" stage in the specification yaml
