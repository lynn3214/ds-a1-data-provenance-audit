# Source, License, and Hash Record

## Source

- **Hub URL**: https://huggingface.co/datasets/lerobot/svla_so101_pickplace
- **Repo type**: dataset
- **Maintainer**: `lerobot` organization on Hugging Face
- **Access method**: `huggingface_hub.snapshot_download(repo_id="lerobot/svla_so101_pickplace", repo_type="dataset", revision=PINNED_REVISION)`
- **Pinned commit SHA**: `f641879e22172be7e8161d5e6c1503c2d2feb657`
- **Snapshot access date**: 2026-09-27
- **Why pinned**: freezing the revision means this audit's numbers cannot be silently
  invalidated by a future update to the upstream repo.

## License

- **Declared license**: Apache License 2.0.
- **Where declared**: Hugging Face dataset card metadata (`license: apache-2.0`) and the
  license badge on the dataset page.
- **Practical implication for this coursework**: Apache-2.0 permits copying, modifying,
  and redistributing the data (including in this repository) provided the license
  notice is retained; no additional restriction was found on the dataset page at the
  time of access.
- **Full license text**: https://www.apache.org/licenses/LICENSE-2.0

## File hashes

The notebook (Section 4) computes SHA-256 hashes for every file under `meta/` plus a
spot-check of the first two episode parquet shards, and writes them to
`data_card/file_hashes.csv`. That CSV is the authoritative, machine-generated hash
record — this file only summarizes it.

| relative_path | size_bytes | sha256 |
|---|---|---|
| meta/info.json | 3401 | `254909942a6cbfb4692a239e4d0aa5c68ec8eb16f81b6593985ea7f2bb2823e3` |
| meta/tasks.parquet | 2246 | `1040cdef3328ec4376587152647df8e725e80a04210c426f891719e691c88533` |
| meta/stats.json | 5914 | `4ae86bed785e0f98914812e87736e216139a22b43cf2b990e68384d85168c3c8` |
| meta/episodes/chunk-000/file-000.parquet | 72560 | `191998bffd2680c477a4000270cf943bc423ba5fb54ee8b8244b57db062c5209` |
| data/chunk-000/file-000.parquet | 369943 | `579ad57e2454359fa9f2c0e83525991bd2e0305b7b2914d0b1181da3c6ad9949` |

> **Do not hand-type hash values.** Open `data_card/file_hashes.csv` after running the
> notebook and paste the real rows here (or simply reference the CSV directly in your
> submission and note that this table mirrors it).

## Execution environment (paste from notebook Section 1 output)

```
Python       : <FILL IN>
Platform     : <FILL IN>
pandas       : <FILL IN>
numpy        : <FILL IN>
pyarrow      : <FILL IN>
scikit-learn : <FILL IN>
matplotlib   : <FILL IN>
huggingface_hub: <FILL IN>
SEED         : 42
```

Full dependency freeze: see `environment_freeze.txt` (written automatically by the
notebook's first code cell) and `requirements.txt`.
