# Bioinformatics References

A running list of references I keep coming back to — file formats, tools, data
resources, and reading. This page is rendered directly from the `README.md` of
[DavidStreid/bfx-references](https://github.com/DavidStreid/bfx-references), so
it can be edited from anywhere through GitHub.

---

## File Formats

| Format | Holds | Spec |
|---|---|---|
| FASTA | Reference sequence | [samtools/hts-specs](https://samtools.github.io/hts-specs/) |
| FASTQ | Unaligned reads + base qualities | [NAR 2010](https://doi.org/10.1093/nar/gkp1137) |
| SAM / BAM / CRAM | Aligned reads | [SAMv1](https://samtools.github.io/hts-specs/SAMv1.pdf), [CRAMv3](https://samtools.github.io/hts-specs/CRAMv3.pdf) |
| VCF / BCF | Variant calls | [VCFv4.3](https://samtools.github.io/hts-specs/VCFv4.3.pdf) |
| BED | Genomic intervals (0-based, half-open) | [UCSC FAQ](https://genome.ucsc.edu/FAQ/FAQformat.html#format1) |
| GFF3 / GTF | Feature annotations | [GFF3](https://github.com/The-Sequence-Ontology/Specifications/blob/master/gff3.md) |
| bigWig / bigBed | Indexed genome-wide signal | [Bioinformatics 2010](https://doi.org/10.1093/bioinformatics/btq351) |
| MAF | Mutation annotations (TCGA-style) | [GDC MAF](https://docs.gdc.cancer.gov/Data/File_Formats/MAF_Format/) |

### Coordinate systems

The single most common source of off-by-one bugs:

- **0-based, half-open** — BED, BAM (internally), UCSC. `[start, end)`
- **1-based, closed** — VCF, GFF/GTF, SAM (text), Ensembl. `[start, end]`

---

## Core Tools

### Read handling
- [samtools](https://www.htslib.org/doc/samtools.html) — SAM/BAM/CRAM manipulation
- [htslib](https://github.com/samtools/htslib) — the C library underneath most of the above
- [pysam](https://pysam.readthedocs.io/) — Python bindings for htslib
- [sambamba](https://lomereiter.github.io/sambamba/) — parallel BAM sort/markdup
- [Picard](https://broadinstitute.github.io/picard/) — metrics, duplicate marking, format conversion

### Alignment
- [BWA-MEM](https://github.com/lh3/bwa) / [BWA-MEM2](https://github.com/bwa-mem2/bwa-mem2) — short-read DNA
- [minimap2](https://github.com/lh3/minimap2) — long-read and spliced alignment
- [STAR](https://github.com/alexdobin/STAR) — spliced RNA-seq alignment
- [bowtie2](https://bowtie-bio.sourceforge.net/bowtie2/) — short-read, sensitive local mode

### Variant calling
- [GATK](https://gatk.broadinstitute.org/) — HaplotypeCaller, Mutect2, best-practice workflows
- [bcftools](https://samtools.github.io/bcftools/bcftools.html) — calling, filtering, VCF munging
- [DeepVariant](https://github.com/google/deepvariant) — CNN-based germline calling
- [Manta](https://github.com/Illumina/manta) / [DELLY](https://github.com/dellytools/delly) — structural variants
- [Strelka2](https://github.com/Illumina/strelka) — germline and somatic small variants

### Annotation
- [VEP](https://www.ensembl.org/info/docs/tools/vep/index.html) — Ensembl Variant Effect Predictor
- [SnpEff / SnpSift](https://pcingola.github.io/SnpEff/)
- [ANNOVAR](https://annovar.openbioinformatics.org/)
- [vcf2maf](https://github.com/mskcc/vcf2maf) — VEP-annotated VCF to MAF

### Intervals and utilities
- [bedtools](https://bedtools.readthedocs.io/) — interval arithmetic
- [seqkit](https://bioinf.shenwei.me/seqkit/) — FASTA/FASTQ swiss army knife
- [MultiQC](https://multiqc.info/) — aggregate QC across a run
- [IGV](https://igv.org/) — genome browser, desktop and [igv.js](https://github.com/igvteam/igv.js)

---

## Reference Data

- [GRCh38 / T2T-CHM13](https://www.ncbi.nlm.nih.gov/datasets/genome/) — human reference assemblies
- [gnomAD](https://gnomad.broadinstitute.org/) — population allele frequencies
- [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/) — clinical variant interpretations
- [COSMIC](https://cancer.sanger.ac.uk/cosmic) — somatic mutations in cancer
- [dbSNP](https://www.ncbi.nlm.nih.gov/snp/) — short variant identifiers (rsIDs)
- [OncoKB](https://www.oncokb.org/) — precision oncology knowledge base
- [GENCODE](https://www.gencodegenes.org/) — gene annotation
- [GIAB](https://www.nist.gov/programs-projects/genome-bottle) — truth sets for benchmarking

---

## Workflows and Infrastructure

- [Nextflow](https://www.nextflow.io/) + [nf-core](https://nf-co.re/) — pipelines with curated, reusable modules
- [WDL](https://openwdl.org/) + [Cromwell](https://cromwell.readthedocs.io/) — Broad's workflow stack
- [Snakemake](https://snakemake.readthedocs.io/) — Python-native workflow DSL
- [Bioconda](https://bioconda.github.io/) / [BioContainers](https://biocontainers.pro/) — packaged tools
- [GA4GH](https://www.ga4gh.org/) — interoperability standards (htsget, refget, DRS)

---

## Reading

- [Bioinformatics Data Skills](https://www.oreilly.com/library/view/bioinformatics-data-skills/9781449367480/) — Vince Buffalo
- [Biostar Handbook](https://www.biostarhandbook.com/)
- [Heng Li's blog](https://lh3.github.io/) — format and algorithm internals
- [Biostars](https://www.biostars.org/) — the Q&A archive worth searching first
- [Rosalind](https://rosalind.info/) — algorithm problems

---

## Notes to Self

- Always confirm the reference build before comparing coordinates across sources.
- `bcftools norm -m -any -f ref.fa` before any variant comparison — representation
  differences account for most false discordance.
- CRAM is reference-dependent; archive the exact reference alongside the data.
