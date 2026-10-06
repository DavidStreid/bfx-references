

## Parquet

**Features**

* Binary, columnar, and index file
* Supports data streaming, as opposed to in-memory (like `pandas`), which loads the entire file into memory before processing
* Most metadata is maintained in a footer
  * Allows for one-time pass through b/c extra data can be appended
  * Allows constant time for reading metadata by reading the end of the file
* Well supported by [DuckDB](https://duckdb.org/install/), which is industry standard for processing tabular genomic data

**Advantages**

- Massively reduced storage compared to simple columnar file formats (e.g. TSV, CSV)
- Excels in high-read use cases

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

R/W with parquet
```
def my_func(chrom, pos, ref):
    return f"{chrom}:{pos}:{ref}:{'common' if pos < 11000 else 'other'}"
 
src = pq.ParquetFile("variants.parquet")
tmp = "variants.annotated.parquet.tmp"
out_path = "variants.annotated.parquet"
writer = pq.ParquetWriter(tmp, out.schema, compression="zstd")
for batch in src.iter_batches(batch_size=10_000):
    t = pa.Table.from_batches([batch])
    vals = [my_func(c, p, r)
            for c, p, r in zip(t["chrom"].to_pylist(), t["Pos"].to_pylist(), t["Ref"].to_pylist())]
    out = t.append_column("annotation", pa.array(vals))
    writer.write_table(out)
writer.close()
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

## GWAS

Most importantly, genes and their p-values. Where the p-value is the likelihood of observing the given signal linking the gene to the condition given the null hypothesis that none of the gene's variants have any relationship to the condition. Note: confounders like gene size
* Formats:
  * MAGMA
  * VEGAS, e.g. [BMI condition](https://s3-us-west-2.amazonaws.com/humanbase/netwas/examples/bmi-vegas.txt)

Example VEGAS
```
Chr	Gene	nSNPs	nSims	Start	Stop	TestStat	Pvalue
10	A1CF	90	1000	52236330	52315441	145.339170015442	0.155
10	ABCC2	99	1000	101532452	101601652	39.7805508425323	0.886
...
```

## DockerFile

### Deployment

**Build with docker, load with podman** - podman has rootless mode, which allows running containers w/o root privileges

1. Build
 * e.g. building a linux image on macOS
```
docker build --platform=linux/amd64 -t "my_image:latest" .  2>&1 | tee log.my_image.latest.out
```

2. Save Image Build Artefact
```
$ docker image ls
REPOSITORY      TAG       IMAGE ID       CREATED        SIZE
my_image   latest    ef54992g4h38   16 hours ago   4.4GB
$ docker save -o my_image.latest.tar my_image:latest
$ rsync -azvP my_image.latest.tar <TARGET_HOST>:<PATH>
```

3. Load Image on target host
```
podman load -i <PATH>/my_image.latest.tar
```

### Docker cleanup

Delete stopped containers (leaves tagged images and build caches)
```
docker container prune
```

Delete cached layers (leaves containers and images)
```
# Removes: Cached BuildKit build stages
# Leaves: Containers, images, volumes, networks
# next `docker build` will need to rebuild the cached layers
docker builder prune -a
```

Nuclear Option - delete everything not attached to currently running container
```
docker system prune -a --volumes
```
