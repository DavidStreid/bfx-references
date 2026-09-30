# Profile (e.g. `./bash_profile`)

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
