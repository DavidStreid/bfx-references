

## Parquet

**Features**

* Binary, columnar, and index file
* Most metadata is maintained in a footer
  * Allows for one-time pass through b/c extra data can be appended
  * Allows constant time for reading metadata by reading the end of the file
* Well supported by [DuckDB](https://duckdb.org/install/), which is industry standard for processing tabular genomic data

**Advantages**

- Massively reduced storage compared to simple columnar file formats (e.g. TSV, CSV)


**Example**

```
$ cat variants.tsv
chrom	Pos	Ref	Alt	AF
chr1	10146	A	T	0.051
chr1	10352	T	TA	0.120
chr2	11024	C	T	0.210
$ duckdb -c "COPY (SELECT * FROM read_csv('variants.tsv', delim='\t')) TO 'variants.parquet' (FORMAT PARQUET);"
$ duckdb -c "SELECT * FROM 'variants.parquet' WHERE AF > 0.10;"
┌─────────┬───────┬─────────┬─────────┬────────┐
│  Chrom  │  Pos  │   Ref   │   Alt   │   AF   │
│ varchar │ int64 │ varchar │ varchar │ double │
├─────────┼───────┼─────────┼─────────┼────────┤
│ chr1    │ 10352 │ T       │ TA      │   0.12 │
│ chr2    │ 11024 │ C       │ T       │   0.21 │
└─────────┴───────┴─────────┴─────────┴────────┘
```

## Genome Ordered Relations (GOR)

* GOR is a genomic-ordered relational database architecture including GORpipe and query language, but can also refer to,
  * `.gord` - Genomic-Ordered Relational Dictionary - metadata mapping file in the GORpipe ecosystem that allows grouping multiple individual genomic data files (.gor, .gorz, or .vcf) as a single, virtual table
  * [`GORpipe`](https://gorpipe.org/) which is the executable, often aliased to `gor`
  * `.gor` – simple tabular format sorted strictly by chromosome and genomic position (chr, pos), `.gorz` (compressed), `.gori` (index file)

**Features**

- Can use VCFs out-of-the-box, will strip header columns and parse with the standard VCF columns

```
gor sample.vcf | where CHROM = 'chr1' | top 10
```

**Example**

GOR
```
#Chr	Pos	Reference	Allele	AF
chr1	10146	A	T	0.051
chr1	10352	T	TA	0.120
chr2	11024	C	T	0.210
```

(Equivalent VCF)
```
##fileformat=VCFv4.2
##FILTER=<ID=PASS,Description="All filters passed">
##INFO=<ID=AF,Number=A,Type=Float,Description="Allele Frequency">
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO
chr1	10146	.	A	T	.	PASS	AF=0.051
chr1	10352	.	T	TA	.	PASS	AF=0.120
chr2	11024	.	C	T	.	PASS	AF=0.210
```
