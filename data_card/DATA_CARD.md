# Data Card — `lerobot/svla_so101_pickplace`

> This Data Card follows the spirit of the Hugging Face / Google "Data Cards" template,
> scoped to what DS-A1 requires. All numbers below were produced by running
> `notebook/DS_A1_provenance_audit.ipynb` top-to-bottom on the pinned revision; none are
> estimated or assumed.

## 1. Dataset in one paragraph

`lerobot/svla_so101_pickplace` is a real-world robot manipulation dataset collected with
an SO-100/SO-101 "follower" robot arm performing a single task — "pink lego brick into the transparent box" — using the LeRobot recording pipeline. It contains 50 teleoperated
episodes (11,939 frames at 30 fps), each with a 6-dimensional commanded `action` and
measured `observation.state` (joint positions), plus two synchronized camera streams
(`up`, `side`). It has been used as a real-world evaluation benchmark in the SmolVLA
paper (arXiv:2506.01844). It is distributed under the Apache-2.0 license.

## 2. Source and access

| Field | Value |
|---|---|
| Hub URL | https://huggingface.co/datasets/lerobot/svla_so101_pickplace |
| Repo type | `dataset` |
| Pinned revision (commit SHA) | `f641879e22172be7e8161d5e6c1503c2d2feb657` |
| Snapshot access date | 2026-09-27 |
| Format | Parquet (tabular/timeseries) + MP4 (video) |
| License | Apache License 2.0 (declared `apache-2.0` at the pinned revision; see `SOURCE_LICENSE_HASH.md`) |
| Total downloaded size (parquet + meta + video, at pinned revision) | 82 MB* (measured via `du -sh data_cache/svla_so101_pickplace` after `snapshot_download`) |
| Codebase/format version at pinned revision | `v3.0` (see Section 5 — this differs from what the rendered dataset-card webpage appeared to describe during initial planning) |
| Citation | No formal BibTeX published by the maintainers at time of access (dataset card states "More Information Needed"); cite by Hub URL + pinned revision. |

*(\*Note: this differs in scope from the ~86.1 MB figure sometimes shown on the Hub's
dataset viewer, which typically reflects only the parquet-visible portion; 82 MB here
is the full local snapshot including video files, measured directly rather than read
off the webpage.)*

## 3. Data-generating process summary

Human teleoperator drives a leader arm → SO-100/SO-101 follower arm executes and logs
`state`/`action` → two cameras record synchronized video → LeRobot packages per-frame
parquet + per-episode mp4 → pushed to the Hub. Full diagram: notebook Section 3.

## 4. Population, sample, unit (restated from the notebook)

- **Population of interest**: SO-100/SO-101 pick-and-place behavior in general.
- **Sample actually collected**: 50 episodes of one specific task, one robot, one
  near-fixed physical starting configuration (Section 6 below).
- **Unit of analysis**: one frame (30 fps); secondary unit: one episode.

## 5. Known provenance gaps and findings

**Undisclosed by the dataset card (treated as open questions, not assumed away):**
- Number of distinct human teleoperators.
- Whether object placement / lighting / camera pose varied across episodes.
- No success/reward/outcome label of any kind is included in the schema.

**Version drift finding (discovered during this audit, not assumed):** during initial
planning, the rendered dataset-card webpage appeared to describe a v2.1-style layout (a
single `meta/episodes.jsonl` file, one parquet file per episode). The pinned snapshot
actually used in this audit (`f641879e22172be7e8161d5e6c1503c2d2feb657`) declares
`codebase_version = v3.0` in its own `meta/info.json`, with a correspondingly different
on-disk layout (`meta/episodes/` as sharded parquet, `meta/tasks.parquet`, multi-episode
parquet chunks under `data/`). This was confirmed by directly inspecting the downloaded
files, not by trusting the webpage — the pinned revision's own `meta/info.json` is
treated as ground truth wherever it disagrees with the rendered dataset card. This is
concrete evidence for why this audit freezes and hashes the actual snapshot rather than
describing the dataset from its webpage alone.

## 6. Audit summary (all values from actual notebook runs)

| Check | Result | Notebook section |
|---|---|---|
| Schema (dtype/shape vs `info.json`) | All 7 tabular columns matched their declared shape exactly; the 2 video columns are absent from parquet as expected (out of scope) | §6 |
| Range / plausibility | 0/11,939×12 non-finite values. `action` vs `state` mean diff: <0.1 for shoulder_pan/wrist_roll/wrist_flex, up to 1.52 (elbow_flex), 1.26 (gripper), 0.78 (shoulder_lift). `action.shoulder_lift.pos` hits an exact −100.0 min and `action.elbow_flex.pos` an exact 100.0 max — likely a joint-limit clamp, not free commanding | §7 |
| Missingness (tabular NaNs) | 0 NaN cells across all columns | §8 |
| Structural missingness (no outcome label) | Confirmed: no success/reward/done field anywhere in the schema | §8 |
| Duplicate frames | 14.13% of all frames are exact duplicates of the previous frame overall (per-episode range 5.9%–32.3%, every episode ≥5.9%). Episode-start static runs: median 3, mean 7.18 frames (37/50 episodes have ≥1); these account for only ~3% of frames, so most duplication occurs mid-episode, not just at the start | §9 |
| Anomaly: parquet vs episode-metadata length | 50/50 episodes matched exactly (0 discrepancies) | §10 |
| Anomaly: timestamp regularity | 0/50 episodes exceeded the 5% (1.7 ms) tolerance on the declared 1/30 s step | §10 |
| Leakage gap (episode-level MSE − frame-level MSE), k=1 | 18.53 − 5.35 = **13.18** (episode-level is 3.46× higher); consistent direction at k=3 (3.39×) and k=5 (2.59×). A naive carry-forward baseline (MSE=1.88) beats the k-NN model under every split/k tested (disclosed for honesty; does not affect the leakage conclusion) | §11 |
| Coverage of episode-start states | 4/6 joints essentially constant at start (std 0.14–0.98 vs ~90°+ ranges); the other 2 vary but only across a narrow band | §12 |

## 7. Intended and out-of-scope uses

**Reasonably supported by this sample:** diagnosing evaluation-protocol pitfalls (e.g.
the demonstrated leakage from naive frame-level splits) for imitation-learning research
on this exact task/robot; illustrating single-task behavior-cloning baselines with an
episode-aware evaluation protocol.

**Not supported (see notebook Section 13 for the full counterexample):** claims about
generalization to new objects, containers, lighting, operators, or robots; claims about
task *success* rate (no outcome label exists); reporting a benchmark number from this
dataset's default single `train` split without disclosing an episode-aware split
protocol — the audit shows this would misstate held-out error by roughly 2.6×–3.5×.

## 8. Ethical / legal notes

- No personal data of identifiable people is intentionally present in the tabular
  channels (joint angles, timestamps). The video streams may incidentally show a human
  hand/operator; this was not reviewed frame-by-frame as part of this audit — flagged
  as an explicit limitation, not confirmed absent.
- Apache-2.0 permits reuse, modification and redistribution with attribution and
  inclusion of the license notice; see `SOURCE_LICENSE_HASH.md`.