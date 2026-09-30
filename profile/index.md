# Profile (e.g. `./bash_profile`)

## TSV -> Parquet
```
pq_tsv() {
  if [ -z "$1" ]; then
    echo "Usage: tsv2parquet <input.tsv> [output.parquet]"
    return 1
  fi

  local input="$1"
  local output="${2:-${input%.*}.parquet}"

  if [ ! -f "$input" ]; then
    echo "Error: File '$input' not found."
    return 1
  fi

  duckdb -c "COPY (SELECT * FROM read_csv('$input', delim='\t')) TO '$output' (FORMAT PARQUET);"
  echo "Successfully converted '$input' -> '$output'"
}
```

## BAM -> CRAM
```
b2c() {
    if [ -z "$1" ]; then
        echo "Usage: b2c <filename.bam>" >&2
        return 1
    fi

    local bam="$1"
    # Strips '.bam' suffix if present and appends '.cram'
    local base="${bam%.bam}"
    local cram="$(basename ${base}.cram)"

    cat << EOF
samtools \\
  view -@ 50 \\
  -C \\
  -T GRCh38.fa \\
  -o ${cram} \\
  --write-index \\
  ${bam}
EOF
}
```

## CRAM -> BAM
```
c2b() {
    if [ -z "$1" ]; then
        echo "Usage: c2b <filename.cram>" >&2
        return 1
    fi

    local cram="$1"
    # Strips '.cram' suffix if present and appends '.bam'
    local base="${cram%.cram}"
    local bam="$(basename ${base}.bam)"
    local bai="${bam}.bai"

    cat << EOF
samtools \\
  view -@ 50 \\
  -T GRCh38.fa \\
  -b ${cram} \\
  | tee ${bam} \\
  | samtools index - ${bai}
EOF
}
```
