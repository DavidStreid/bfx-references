# Bioinformatics References

---

## Online Bioinformatics Tools

### Swiss-army-knife of transcripts, coordinates, liftover, normalization, etc.

[`mutalyzer`](https://mutalyzer.nl/)

### Retrieval of transcripts and coordinates 

Here's how to get transcript FASTA & coordinates given a gene, for instance, the MANE Select transcripts of DMD - `ENST00000357033.9` & `NM_004009.3`

#### ENSEMBL: [biomart](https://useast.ensembl.org/info/data/biomart/index.html?)

Go to biomart and enter transcript, e.g. `ENST00000357033.9`. Left-side navigates to [exons](https://useast.ensembl.org/Homo_sapiens/Transcript/Exons?db=core;g=ENSG00000198947;r=X:31119222-33211549;t=ENST00000357033)

#### NCBI: [Genome Data Viewer](https://www.ncbi.nlm.nih.gov/gdv/?org=homo-sapiens)

E.g. What are the coordinates of exon 1 on the NM_004009.3 transcript?

1. Go to the [DMD Gene](https://www.ncbi.nlm.nih.gov/gdv/browser/nucleotide/?id=NM_004009.3) and enter `NM_004009.3`. The track will show the exons. 
2. Select exon1, which has genomic coordinate positions of `chrX:33,128,147-33,128,426` (cds range: 33,128,147 - 33,128,165), see below. Note the specification of CDS, as Exon 1 on a transcript will almost always contain the 5’ UTR region.

```
[ 33,128,426 <------------------------- 33,128,166 ] [ 33,128,165 <--- 33,128,147 ]
|__________________________________________________| |____________________________|
                 5' UTR (261 bp)                        Coding / CDS (19 bp)
|_________________________________________________________________________________|
                             Full Exon 1 (280 bp)
```

#### Other Helpful Sites
* [GeneBe](https://genebe.net/gene/hg38/DMD)

## Gene IDs

|  **System**  |     **Example**    |                                           **Notes**                                           |
|:------------:|:------------------:|:---------------------------------------------------------------------------------------------:|
| Ensembl gene | ENSG00000012048.23 | IDEAL - Versioned and unique, but not discernible                                             |
| HGNC symbol  | BRCA1              | DISPLAY-ONLY - Issues: Excel (2020) & non-unique from aliases matching other genes (e.g. TAZ) |
| OMIM         | 113705             | Disease-oriented; Gene & phenotypes                                                           |
| HGNC ID      | HGNC:1100          | Ideal ID for HGNC, but not used often                                                         |

### Issue of HGNC symbol
* PAR regions (on both chrX & chrY) are often one-to-many HGNC-to-ENSG
* This one-to-many is found throughout the human genome, specifically 261 human ENSGs map to more than one Entrez GeneID as of Sept, 2026. Examples below w/ how to recreat e this -


| ENSG | Entrez GeneIDs → symbols |
|---|---|
| `ENSG00000115239` | 51130 → **ASB3** · 100302652 → **GPR75-ASB3** |
| `ENSG00000120341` | 89866 → **SEC16B** · 111240474 → **CRYZL2P-SEC16B** |
| `ENSG00000137843` | 56924 → **PAK6** · 106821730 → **BUB1B-PAK6** |
| `ENSG00000139323` | 282809 → **POC1B** · 133039968 → **POC1B-DUSP6** |
| `ENSG00000145979` | 51256 → **TBC1D7** · 107080638 → **TBC1D7-LOC100130357** |
| `ENSG00000152926` | 51351 → **ZNF117** · 109504726 → **ERV3-1-ZNF117** |
| `ENSG00000158747` | 4681 → **NBL1** · 100532736 → **MICOS10-NBL1** |
| `ENSG00000163156` | 79005 → **SCNM1** · 100534012 → **TNFAIP8L2-SCNM1** |
| `ENSG00000178882` | 144347 → **RFLNA** · 100533183 → **ZNF664-RFLNA** |
| `ENSG00000186184` | 51082 → **POLR1D** · 147380392 → **LOC147380392** |
| `ENSG00000186448` | 10168 → **ZNF197** · 110354863 → **ZNF660-ZNF197** |
| `ENSG00000212127` | 50840 → **TAS2R14** · 106707243 → **PRH1-TAS2R14** |
| `ENSG00000213160` | 151230 → **KLHL23** · 100526832 → **PHOSPHO2-KLHL23** |
| `ENSG00000213999` | 100271849 → **MEF2B** · 4207 → **BORCS8-MEF2B** |
| `ENSG00000215440` | 79716 → **NPEPL1** · 124904942 → **LOC124904942** |

```
curl -O https://ftp.ncbi.nlm.nih.gov/gene/DATA/gene2ensembl.gz
zcat gene2ensembl.gz | awk -F'\t' '$1==9606{print $3"\t"$2}' | sort -u \
  | awk -F'\t' '{c[$1]++} END{for(k in c) if(c[k]>1) print c[k]"\t"k}' | sort -rn
```

## Notes

[Variant Naming](https://genome.sph.umich.edu/wiki/Variant_Normalization) - left-aligned & parsimonious
