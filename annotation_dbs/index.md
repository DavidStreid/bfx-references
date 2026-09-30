---
title: Gene Databases
slug: gene-databases
section: Databases
order: 10
summary: External gene-level annotation resources (GO, OMIM, ...) and how to query them.
tags: [go, omim, annotation]
updated: 2026-09-27
draft: false
---

# Gene Databases

## Gene Ontology (GO)
- **What:** Shared vocabulary to describe gene functions. It's organized into three aspects - Molecular Function (MF), Cellular Component (CC), and Biological Process (BP)
  - GO ontology is organized in DAG hierarchy like below, which shows a biological process for "hexose biosynthetic process"
  - Go Annotations provide relations between genes
- **Link:** https://geneontology.org
- **Access:** Free, CC BY 4.0
  - Ontology: `go-basic.obo` from https://current.geneontology.org/ontology/
  - Annotations: GAF files per species from https://current.geneontology.org/annotations/
- **Files:**
  - GAF (GO Annotation File), [download link](current.geneontology.org/annotations/goa_human.gaf.gz) - provides gene -> term and allows for co-annotations (e.g. these two genes share term 1, i.e. these two genes are related in the same process)
    ```
    $ grep "GO:0019319" goa_human.gaf | head -1
    UniProtKB	Q7LFX5	CHST15	involved_in	GO:0019319	PMID:11572857	IDA		P	Carbohydrate sulfotransferase 15	CHST15|BRAG|GALNAC4S6ST|KIAA0598	protein	taxon:9606	20061107	UniProt		UniProtKB:Q7LFX5
    ```
- **Example**: [hexose biosynthetic process, GO:0019319](https://www.ebi.ac.uk/QuickGO/term/GO:0019319)

![GO DAG for hexose biosynthetic process](https://geneontology.org/assets/hexose-biosynthetic-process.png)

*Source: [Gene Ontology documentation](https://geneontology.org/docs/ontology-documentation/)*

## OMIM
- **What:** Catalog of human genes and genetic phenotypes
- **Link:** https://omim.org
- **Access:** Registration required
  - `mim2gene.txt` is freely downloadable
  - `genemap2.txt` and the REST API need a free API key from https://omim.org/downloads

