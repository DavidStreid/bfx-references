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

## Notes

[Variant Naming](https://genome.sph.umich.edu/wiki/Variant_Normalization) - left-aligned & parsimonious
