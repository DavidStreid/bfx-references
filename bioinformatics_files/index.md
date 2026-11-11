## Reference Genome Assemblies

| Build | Institution | Date | Basis | Naming |
|---|---|---|---|---|
| **GRCh37** | Genome Reference Consortium | Feb 2009 (final patch, p13, in 2013) | Primary reference that other builds derive from | No `chr`; mitochondria `MT` (rCRS). Accession style `NC_000001.10`. File name: `GRCh37.p13.genome.fasta` |
| **hg19** | UCSC | Feb 2009 | Derived from GRCh37, but not identical: `chrM` is the older Cambridge sequence (`NC_001807`), not rCRS. Only the nuclear chromosomes are equivalent | `chr` prefix; mitochondria `chrM`. File name: `ucsc.hg19.fasta` |
| **b37** | Broad Institute | Feb 2009 (assembly) | Repackaged GRCh37. Nuclear chromosomes match hg19's sequence; the difference is contig naming | No `chr`; mitochondria `MT`. File name: `Homo_sapiens_assembly19.fasta` |
| **hs37-1kg** (`human_g1k_v37`) | 1000 Genomes Project | Assembly: Feb 2009 | b37 | Chromosomes `1`–`22`, `X`, `Y` from Ensembl; `MT` is rCRS. Unlocalized and unplaced contigs named by accession (e.g. `GL000191.1`, `GL000211.1`). No alternate loci |
| **hs37d5** | 1000 Genomes Project (Phase II) | Assembly: Feb 2009; README dated 2011-07-07 (from file name, unverified) | hs37-1kg plus decoy sequence (basis listed as GRCh37.p4 in your notes, unverified) | Same naming as hs37-1kg. Decoy sequence (BAC/fosmid clones, HuRef contigs, Epstein-Barr virus genome) reduces false-positive mappings |
| **GRCh38 (hg38)** | Genome Reference Consortium | Dec 2013 | Updated assembly replacing GRCh37 | Depends on distributor: GRC/EBI `1`; UCSC `chr1`; NCBI `NC_000001.11`. `hg38` is UCSC's ID for this build. ALT configs added in patches |
| **T2T-CHM13** | Telomere-to-Telomere (T2T) Consortium | 2022 | First gapless human assembly. Built from a single haploid cell line (CHM13); chrY is added from another individual (unverified). A single linear reference, not a pangenome | UCSC calls it `hs1` and uses `chr1`, `chrM`. NCBI accession `GCA_009914755.4` |
| **HPRC pangenome** | Human Pangenome Reference Consortium | 2023-current| Many haplotype-resolved assemblies from diverse individuals, linked as a graph. Intended as a replacement for the single linear reference. GRCh38 and T2T-CHM13 serve as linear backbones (unverified) | No single coordinate system or naming scheme. Each haplotype contig is named per sample and haplotype (e.g. the PanSN convention `sample#haplotype#contig`, unverified). No UCSC `hg` ID |

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

## Containerization

### Development

#### **[Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)**

**Definition**: Mutliple `FROM` **and** `COPY` instructions to take only what is needed. This powerfully provides selected functionality without the additional steps

Key Optimizations
* Order is important - subsequent layers depend on previous layers
* As much as possible, put frequently changed layers in the final layers
* Optimize for deployment by putting the large-file base installations early on in the build. This allows the lighter layers to quickly be deployed


**Example:** Multi-Stage Build creating a container of bioinformatics tools and the app that uses them
* Pruned to only keep what is needed via `FROM...COPY` syntax
* Large base installations provide the foundation

**DOCKER FILES**

`Dockerfile.base` - installs the shared runtime library (`libhts3`) required for `samtools`

```
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y libhts3
```

`Dockerfile.tools` - installs `samtools` AND prunes image with `FROM... runner; COPY ...` syntax, i.e. `libhts3` will NOT be in the final image

```
# Install `samtools`
FROM my-base:v1.0 AS builder
RUN apt-get update && apt-get install -y gcc make libhts-dev curl
RUN curl -O https://.../samtools-1.20.tar.bz2 && tar -xjf ...
WORKDIR /samtools-1.20
RUN ./configure --prefix=/opt/samtools && make && make install

# Clean slate w/ `FROM` and then copy just `samtools`
FROM my-base:v1.0 AS runner
COPY --from=builder /opt/samtools /opt/samtools
ENV PATH="/opt/samtools/bin:${PATH}"
```

`Dockerfile.app`
```
FROM my-tools:v1.0

WORKDIR /app
COPY run_pipeline.sh .

CMD ["/bin/bash", "run_pipeline.sh"]
```

**BUILD STEPS**

```
podman build -f Dockerfile.base  -t my-base:v1.0 .
podman build -f Dockerfile.tools -t my-tools:v1.0 .
podman build -f Dockerfile.app   -t my-app:v1.0 .
```

#### Pruning

To reduce size of the final image, it may help to cleanup any unused C source, headers, objects, and static archive.

Consider the example below where the final image size is cut in half by removing no-longer required libraries, code, and other files.

`Dockerfile` - would call this script to build bwamem-2
```
RUN install_bwamem2.sh
```

`install_bwamem2.sh` - Final executable size is reduced by >50% by removing static libraries, raw code, and intermediate object files
```
git clone --recurse-submodules --branch "v2.2.1" --depth 1 \
  https://github.com/bwa-mem2/bwa-mem2.git "bwa-mem2-2.2.1"
cd bwa-mem2-2.2.1
# 3.2M

make -j"$(nproc)" # `-j`, processes w/ number of processing units available, e.g. `-j20`
# 22M

# RETAIN ONLY REQUIRED - all sibling dispatcher binaries (bwa-mem2, bwa-mem2.avx2, etc.)
# Below are removed files that are no longer needed
#  - Static Library Archive     - compiled object files that are deadweight one the standalone bwa executable is created e.g. `libbwa.a`
#  - C Source Code/Header Files - raw code files
#  - Object Files               - intermediate machine-code files
find . -maxdepth 1 ! -name 'bwa-mem2*' ! -name '.' -exec rm -rf {} +
# 10M

# SANITY CHECK - cleanup didn't affect anything
./bwa-mem2 version 2>&1 | grep "${VERSION#v}"
```

**Other Container Build Size Optimizations**

* `pip install --no-cache-dir` - instructs pip not to save copies of the downloaded `.whl` or `.tar.gz` files into pip's cache directory

### DockerFile

#### Deployment

Can build with either docker or podman (rootless mode)

**COMMANDS** - `docker` is the same, just substitute it for `podman` below

1. Build
 * e.g. building a linux image on macOS
```
podman build --platform=linux/amd64 -t "my_image:latest" .  2>&1 | tee log.my_image.latest.out
# podman run --rm -it my_image:latest # Optional: Manual Verification
```

2. Save
```
podman save my_image:latest | gzip > my_image.latest.tar.gz # or, pigz
```

3. Transfer to other host

4. Load
```
podman load -i /path/to/destination/my_image.latest.tar.gz 
```

#### Podman Notes

* podman provisions a lightweight, headless Linux VM - podman commands run in host terminal are forwarded by the CLIover an SSH Unix socket to the Podman daemon inside that VM
* CLI-first approach, but does provide [podman desktop](https://podman-desktop.io/downloads), which is similar to Docker Desktop


##### Sibling Containers

Sibling containers can be run by creating a socket that allows processes running inside container to make API calls to host container engine. This is done via [domain sockets](https://www.lenovo.com/us/en/glossary/unix-domain-socket/?orgRef=https%253A%252F%252Fwww.google.com%252F&srsltid=AYCX9vLwcOKruDRNKP5dlkL0EvrgmXdKpP5hUtLkTpQlbb7Q8BQ4X6hk), "an interprocess communication (IPC) mechanism"

**1. Create Socket**
```
systemctl --user enable --now podman.socket
ls -l "/run/user/$(id -u)/podman/podman.sock"
```

**2. Integrate Socket**

* Set the `CONTAINER_HOST`, a built-in environment variable for podman, to the socket created
* This will delegate all container operations inside pv3 directly to the host's daemon

When `podman` runs INSIDE the container, it will inspect the environment for `CONTAINER_HOST` - 

* If `CONTAINER_HOST` is defined, podman will run as `podman --remote --url unix:///run/podman/podman.sock run...`
* This is basically an API request to run the `podman` command on the `url` provided, i.e. the host machine

```
podman run --rm \
  -v /run/user/$(id -u)/podman/podman.sock:/run/podman/podman.sock:Z \ 
  -e CONTAINER_HOST=unix:///run/podman/podman.sock \
  my_image:latest echo "it works!"
```

##### General TroubleShooting

_User Namespaces_

More than once, I've run into an issue with User Namespaces, which is where podman maps system UIDs and GIDs into a container. 

* Podman avoids root privilege issues by running the unprivileged host user, i.e. whoever is running the container (e.g. `my_user` with UID `1001`), and mapping that to UID `0`, or root, inside the container.
* Packages expecting to be installed as root inside the container think they are and can.
* Outside on the host however, the kernel still treats the user as unprivileged, i.e. the `my_user` with UID `1001`, and never grants it root privileges.

For instance, see the translation table for the provided `/etc/subuid`

```
$ cat /etc/subuid
my_user:100000:65536
```

| Container     | Host                                 | Where It Comes From                         |
|:--------------|:-------------------------------------|:--------------------------------------------|
| UID 0 (root)  | Host UID of my_user (e.g., 1001)     | Your real login UID                         |
| UID 1         | UID 100000                           | Subordinate range start                     |
| UID 2         | UID 100001                           | Subordinate range offset (100000 + 2 - 1)   |
| ...           | ...                                  | ...                                         |
| UID 1000      | UID 100999                           | Subordinate range offset (100000 + 1000 - 1)|
| ...           | ...                                  | ...                                         |
| UID 65536     | UID 165535                           | Subordinate range end (100000 + 65536 - 1)  |

In this example `my_user` will only have a limited set of unused host IDs - 

* Container UID 0 maps to host UID testuser (e.g., 1001).
* Container UIDs 1 through 65,536 map to host subordinate UIDs 100000 through 165535.

When pulling the `clair3` docker image [See instructions](https://github.com/HKU-BAL/clair3#option-1-docker), 

there is an error because the container files inside `/opt/models/hifi` have an internal user account with UID 1000088, which is outside the range of host subordinate UIDs available to the user
```
podman run --rm docker.io/hkubal/clair3:v1.0.10 /bin/true
# "Error: writing blob" - processing tar file(potentially insufficient UIDs or GIDs available in user namespace (requested 1000088:100 for /opt/models/hifi): Check /etc/subuid and /etc/subgid if configured locally and run "podman system migrate": lchown /opt/models/hifi: invalid argument): exit status 1
```

To address this, 

either **Update** - `/etc/subuid` to include more (below, 2_000_000 are provided)

```
# Update /etc/subuid
sudo sed -i 's/^my_user:.*/my_user:100000:2000000/' /etc/subuid
podman system migrate
```

or **Squash** ownership by pulling with `--storage-opt ignore_chown_errors=true` 
* istructs Podman's storage driver to catch that failure, ignore it, and squash the ownership of those files down to your user

```
podman --storage-opt ignore_chown_errors=true pull docker.io/hkubal/clair3:v1.0.10
```

#### Podman Cleanup

**Dangling Images** - images without a tag pointing to them from either updating or pulling an image. For instance, re-running `podman build ... -t "tools:v1" .` and then updating the tag to `v2`.

```
podman image prune
```

**Targeted repo removal**

```
REPO="tools"
podman rmi $(podman images -q ${REPO}) # -q will get the image IDs for all REPO images
```

#### Docker cleanup

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
