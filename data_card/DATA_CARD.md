# Data Card — `lerobot/svla_so101_pickplace`

> This Data Card follows the spirit of the Hugging Face / Google "Data Cards" template,
> scoped to what DS-A1 requires. The audit numbers in Section 6 were produced by running
> `notebook/DS_A1_provenance_audit.ipynb` top-to-bottom on the pinned revision; none are
> estimated or assumed. Descriptive facts in Sections 1–5 come from the pinned snapshot's own
> metadata or, where stated, from the SmolVLA paper (arXiv:2506.01844v1, checked on 2026-09-30).

## 1. Dataset in one paragraph

`lerobot/svla_so101_pickplace` is a real-world robot manipulation dataset for a single task,
"pink lego brick into the transparent box" (from `meta/tasks.parquet`), recorded with the
LeRobot pipeline. The pinned `meta/info.json` declares `robot_type = so100_follower`, while
the repository name and the SmolVLA paper refer to an SO-101 arm. It contains 50 demonstration
episodes (11,939 frames at 30 fps; teleoperation is inferred, not stated in the metadata), each with
a 6-dimensional commanded `action` and measured `observation.state` (joint positions), plus two
camera streams (`up`, `side`) declared in `info.json`. It is distributed under the Apache-2.0
license.

The SmolVLA paper (arXiv:2506.01844, Section 4.1, footnote 8) lists this dataset as its SO-101
"Pick-Place-Lego" real-world dataset and reports policy success rates on that task (Table 4).
Those success rates come from physical robot trials described in the paper; the dataset itself
contains no success label (see Section 5). The paper also states that SmolVLA was not pretrained
on any SO-101 data.

## 2. Source and access

| Field | Value |
|---|---|
| Hub URL | https://huggingface.co/datasets/lerobot/svla_so101_pickplace |
| Repo type | `dataset` |
| Pinned revision (commit SHA) | `f641879e22172be7e8161d5e6c1503c2d2feb657` |
| Upstream last-modified time of this commit | 2025-09-27 11:25:41 UTC (Hub API) |
| Snapshot access date | 2026-09-27 |
| Files in the pinned snapshot | 9: 1 data parquet, 4 metadata files, 2 video files (one per camera, each packing multiple episodes), `.gitattributes`, `README.md` |
| Format | Parquet (tabular/timeseries) + MP4 (video) |
| License | Apache License 2.0 (declared `apache-2.0` at the pinned revision; see `SOURCE_LICENSE_HASH.md`) |
| Total downloaded size (parquet + meta + video, at pinned revision) | 82 MB (measured via `du -sh data_cache/svla_so101_pickplace` after `snapshot_download`; `du` also counts the small `.cache/` bookkeeping folder that `huggingface_hub` creates there). Sizes displayed on the Hub web page were not used. |
| Codebase/format version at pinned revision | `v3.0` (from the pinned `meta/info.json`; see Section 5 — the dataset card's own embedded copy of `info.json` says `v2.1`) |
| Hashes | 5 of the 9 files are SHA-256 hashed (`file_hashes.csv`); the README hash is in `ai_use_log/verification_commands.txt`; the 2 video files are not hashed (out of audit scope) |
| Citation | No formal BibTeX published by the maintainers at time of access (the dataset card lists Homepage and Paper as "More Information Needed"); cite by Hub URL + pinned revision. The SmolVLA paper describes the dataset (Section 1) but is not given as its citation in the card. |

## 3. Data-generating process summary

Human teleoperator (inferred; count undisclosed) drives a leader arm → follower arm executes
and logs `state`/`action` → two cameras record video (declared in `info.json`; not decoded or
checked for synchronization in this audit) → LeRobot v3.0 packages per-frame parquet plus
multi-episode mp4 files (2 video files in total) → pushed to the Hub. Full diagram: notebook
Section 3.

## 4. Population, sample, unit (restated from the notebook)

- **Population of interest**: SO-100/SO-101 pick-and-place behavior in general.
- **Sample actually collected**: 50 episodes of one specific task, one declared robot type, and
  a near-identical arm start pose across episodes (notebook Section 12). The start pose being
  similar does not by itself show that objects, box position, lighting or operators were fixed
  (see Section 5).
- **Unit of analysis**: one frame (30 fps); secondary unit: one episode.

## 5. Known provenance gaps and findings

**Not disclosed by the dataset card or metadata (treated as open questions, not assumed away):**
- Number of distinct human teleoperators (also not stated in the SmolVLA paper's experimental
  setup, Sections 4.1–4.2).
- Lighting and camera pose variation across episodes.
- No success/reward/outcome label of any kind is included in the schema.

**Object placement: disclosed only second-hand, by the SmolVLA paper.** Section 4.1 of the
paper says each of its real-world datasets holds 50 demonstrations: 10 trajectories for each of 5
distinct starting positions. This is the paper authors' description, not a field in the data, and
it was not checked against the videos. It is qualitatively consistent with the several separated
`shoulder_pan` peaks seen in the notebook's Section 7 histogram, but that link was not tested.
Consequence for evaluation: if the description holds, roughly ten episodes share each starting
position, so even an episode-level split places episodes from the same position on both sides.
The episode-level MSE in this audit therefore does not measure generalization to a new object
position.

**Version drift finding (discovered during this audit, not assumed):** the dataset card
(`README.md`) at the pinned commit embeds a copy of `meta/info.json` that states
`codebase_version = v2.1` (line 27 of the card). The snapshot's actual `meta/info.json` at the
same commit declares `v3.0`, and the on-disk layout matches v3.0 (`meta/episodes/` as sharded
parquet, `meta/tasks.parquet`, multi-episode parquet chunks under `data/`, multi-episode video
files). At freeze time the Hub HEAD equaled the pinned commit, so the inconsistency lies inside a
single commit: the card's embedded snippet disagrees with the repository's own metadata files.
My planning-stage assumption of a v2.1-style layout (one parquet file per episode, a single
`episodes.jsonl`) matched the card, not the data. The pinned `meta/info.json` and the files
actually on disk are treated as ground truth. This is concrete evidence for why this audit
freezes and hashes the actual snapshot rather than describing the dataset from its documentation.

## 6. Audit summary (all values from actual notebook runs)

| Check | Result | Notebook section |
|---|---|---|
| Schema (dtype/shape vs `info.json`) | All 7 tabular columns matched their declared shape (first row of each column only was inspected); the 2 video columns are absent from parquet as expected (video not decoded, out of scope) | §6 |
| Range / plausibility | 0 non-finite values across 11,939 frames × 12 channels. `action` − `state` mean difference: under 0.1 for shoulder_pan (0.034), wrist_roll (0.019), wrist_flex (0.071); `action` lower than `state` on average by 1.52 (elbow_flex), 1.26 (gripper), 0.78 (shoulder_lift), where lag and constant offset cannot be told apart. Four `action` channels reach an exact round-number boundary that `observation.state` never reaches: shoulder_lift min −100, elbow_flex max 100, wrist_roll max −20, gripper min 0. This is consistent with clipped commands; the cause was not verified. Units of all channels are undocumented | §7 |
| Missingness (tabular NaNs) | 0 NaN cells across all columns | §8 |
| Structural missingness (no outcome label) | Confirmed: no success/reward/done field anywhere in the schema | §8 |
| Repeated-action frames | 14.13% of all frames have an `action` identical to the previous frame's within the same episode (per-episode range 5.9%–32.3%, every episode ≥5.9%). `observation.state` and video of these frames were not compared. Leading static runs: median 3, mean 7.18 frames (37/50 episodes have ≥1), about 3% of frames; the other ~11 percentage points lie outside the leading run (mid-episode vs. trailing was not separated) | §9 |
| Anomaly: parquet vs episode-metadata length | 50/50 episodes matched exactly (0 discrepancies) | §10 |
| Anomaly: timestamp regularity | 0/50 episodes exceeded the 5% (1.7 ms) tolerance on the declared 1/30 s step. If `timestamp` is nominal (frame_index / fps) this checks internal consistency only; not verified | §10 |
| Leakage gap (episode-level MSE − frame-level MSE), k-NN, k=1 | 18.53 − 5.35 = **13.18** (episode-level is 3.46× higher); same direction at k=3 (3.39×) and k=5 (2.59×). One k-NN model, 5 random splits, mechanism not tested directly. A naive carry-forward baseline (MSE 1.88, not like-for-like) beats the k-NN model under every split and k (disclosed; does not affect the leakage conclusion) | §11 |
| Coverage of episode-start states | 4/6 joints essentially constant at start (std 0.14–0.98, vs dataset-wide `observation.state` std 8.4–36.8 for those joints); shoulder_pan (std 4.03) and wrist_roll (std 4.41) vary over about 19 and 21 units. Similar start pose does not show fixed objects or environment | §12 |

## 7. Intended and out-of-scope uses

**Reasonably supported by this sample:** diagnosing evaluation-protocol pitfalls (e.g. the
leakage demonstrated for naive frame-level splits with a k-NN baseline) for imitation-learning
research on this exact task/robot; illustrating single-task behavior-cloning baselines with an
episode-aware evaluation protocol.

**Not supported (see notebook Section 13 for the full counterexample):** claims about
generalization to new objects, containers, lighting, operators, or robots; claims about
generalization to new object positions (an episode-level split does not test this, see
Section 5); claims about task *success* rate (no outcome label exists); reporting a benchmark
number from this dataset's default single `train` split without disclosing an episode-aware
split protocol. For the k-NN baseline audited here, the frame-level split gave error estimates
2.6×–3.5× lower than the episode-level split; this ratio should not be assumed to carry over to
other models.

## 8. Ethical / legal notes

- No personal data of identifiable people is intentionally present in the tabular
  channels (joint angles, timestamps). The video streams may incidentally show a human
  hand/operator; this was not reviewed frame-by-frame as part of this audit — flagged
  as an explicit limitation, not confirmed absent.
- Apache-2.0 permits reuse, modification and redistribution with attribution and
  inclusion of the license notice; see `SOURCE_LICENSE_HASH.md`.