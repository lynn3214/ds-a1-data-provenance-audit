# AI Use and Independent Verification Log — DS-A1

This log discloses AI assistance per the course's common requirement #5
("an AI-use record covering tool, task, material advice, acceptance/rejection, and
independent verification").

## Tools

- **Claude (Anthropic)**, web chat interface with web-search and code-execution tools
  enabled. Used throughout planning, scaffolding, and iterative debugging of this
  repository. Claude's code-execution environment has no network access to
  huggingface.co, so Claude never ran this notebook against the real dataset — every
  number reported anywhere in this repository comes from my own local runs.
- **ChatGPT (GPT 5.6 Luna, free tier)**. Used once, after the notebook scaffold and
  first full run were complete, as a second independent reviewer of the whole
  repository (not just isolated cells).

## Tasks the AI was used for

1. Reading the course syllabus/assignments pages to confirm DS-A1's exact rubric and
   submission requirements, and looking up `lerobot/svla_so101_pickplace` on the
   Hugging Face Hub (dataset card, schema, episode/frame counts, license) to confirm
   the dataset was suitable before committing to it.
2. Proposing the testable question, population/sample/unit/target/estimand framing, and
   the leakage-demonstration design (k-NN, frame-level vs episode-level split).
3. Writing the initial notebook code scaffold (Sections 1–16), the Data Card, data
   dictionary, source/license/hash template, and this log's structure.
4. Iterative debugging as I ran the notebook against the real, pinned dataset snapshot
   (see "AI-authored code defects found and fixed" below).
5. Drafting the interpretive/results text in Sections 6–12 and 14, from the real cell
   outputs I pasted into the conversation after running them.

## AI-authored content that I kept largely as written

The interpretive prose in Sections 2, 3, 6–14 was drafted by Claude from my real
outputs, and I kept most of it close to as-written — English is not my first language,
and Claude's phrasing was generally clearer than what I would have written myself. What
I actually did, rather than write from scratch, was: (a) supply every number and figure
myself by running the code and pasting the output back, (b) check each drafted
conclusion against the number/plot it claims to describe before accepting it (this is
how the two content errors listed below were caught), and (c) make small edits to
formatting and cell placement, and discuss specific wording/code changes turn by turn
(e.g. asking Claude to adjust a comment, move a cell, or fix a stated range). I did not
independently re-derive the interpretive claims in a separate pass; my verification was
checking each claim against the real output it was based on, not re-analysis from
scratch.

## AI-authored code defects found while running this notebook, and how each was caught

| # | Defect | How I caught it | Fix |
|---|---|---|---|
| 1 | Section 4's hashing code assumed `meta/` contained only files; the pinned snapshot's `meta/episodes/` is a directory of parquet shards → `IsADirectoryError` | The notebook crashed on my first real run | Filtered to `is_file()`, recursively |
| 2 | Section 5 assumed a single `meta/episodes.jsonl` file with a `length` column | Same run revealed `meta/episodes/` is a directory (LeRobot v3.0 layout, not the v2.1 layout the rendered webpage suggested during planning) | Added a format-detection branch; derived length as `dataset_to_index - dataset_from_index` from the actual `episodes_meta` columns I printed |
| 3 | Section 5 assumed one parquet file per episode under `data/` | v3.0 format packs multiple episodes per file | Changed the sanity check to compare total frame count and distinct `episode_index` count instead of file count |
| 4 | Section 9's `leading_dup_run` used `cumprod()` on a boolean array whose first element is always `False` (no previous frame within an episode), so it silently returned 0 for every episode | I noticed the printed leading-run values (all 0) were inconsistent with the separately reported 14.13% overall duplicate rate, and flagged the contradiction | Rewrote as an explicit loop counting the true leading run of consecutive duplicates |
| 5 | Section 4's revision-pinning code compared `PINNED_REVISION` against the *current* HEAD SHA via `assert`, which would fail the notebook as soon as upstream pushed a new commit — contradicting reproducibility | Found by GPT's review of the full notebook, not by running it (this specific failure mode only manifests after upstream changes) | Rewrote to resolve and verify only the pinned revision itself, never compared to current HEAD |
| 6 | Section 3 stated `codebase_version = v2.1` (carried over from the webpage read during planning), while the pinned snapshot's own `meta/info.json` says `v3.0` | GPT flagged the inconsistency; I did not accept GPT's accompanying claim about which LeRobot version the specific commit corresponds to (unsourced), and instead ran `print(info_json.get("codebase_version"))` myself to confirm the real value | Rewrote Section 3 to treat the pinned `meta/info.json` as authoritative and record the webpage/snapshot discrepancy as a provenance finding |
| 7 | `data_card/DATA_DICTIONARY.csv`, as drafted by Claude, had unquoted shape values containing commas (e.g. `(1,)`), which broke CSV column alignment | I noticed the row structure looked wrong when I pasted the file content back for review | Claude regenerated the file programmatically with proper CSV quoting and I verified it parses to 19 rows × 7 columns with no nulls |
| 8 | Section 7's text asserted the joint values were in "degrees," and Section 12's text stated specific numeric ranges (e.g. "-70 to -22") for the coverage scatter plot | I checked: no unit field exists anywhere in `meta/info.json`, so "degrees" was unsupported; and I ran `start_arr.min/max(axis=0)` myself and found the real values (-4.03 to 15.05 and -63.27 to -42.08) did not match what had been drafted from a visual read of the plot | Rewrote both sections to state the unit is undocumented, and replaced the plot-derived numbers with the code-derived exact values |
| 9 | A verification cell I added to check `info_json` against expected values, and used `df`, was placed before the cell that defines `df` (Section 5's third cell) | Cell ran fine when executed alone against a kernel with leftover state from earlier runs, but `NameError: name 'df' is not defined` appeared on a clean `Restart & Run All` | Moved the cell to after `df` is defined; this is also why every reproducibility check below used a full clean restart, not cell-by-cell reruns |

## Independent verification performed

- **Two full clean runs** (`Kernel → Restart & Run All`) on the final version of the
  notebook produced identical `report/key_metrics.json` (matching to 6 decimal places
  on every reported metric: duplicate rates, leakage MSEs at k=1/3/5, naive baseline
  MSE, length/timestamp anomaly counts, coverage std values) and byte-identical
  `data_card/file_hashes.csv`. `git status` after the second run showed no diff for
  either file, an independent confirmation via a different mechanism than `diff`.
- **Command-line hash recomputation**: independently reran `shasum -a 256` on all five
  hashed files outside the notebook/Python entirely; all five values matched
  `file_hashes.csv` exactly.
- **`info_json` field check**: wrote and ran a cell comparing `robot_type`, `fps`,
  `total_episodes`, `total_frames`, `total_tasks`, and `codebase_version` against the
  values planning-stage research had assumed; all six matched, and I additionally
  confirmed `len(df) == total_frames`, `df["episode_index"].nunique() == total_episodes`,
  and the six `action` channel names.
- **Environment versions**: cross-checked `pip freeze` (`environment_freeze.txt`)
  against `conda list`, since some packages (e.g. pandas, installed via conda-forge)
  do not appear in `pip freeze`; recorded the confirmed versions of all key packages
  in `SOURCE_LICENSE_HASH.md`.
- Every numeric claim in Sections 6–14 of the notebook, the Data Card, and this log
  was checked against the actual cell output before being written down, rather than
  accepted from AI-drafted text as-is.

## What the AI did not do

Neither AI executed this notebook or downloaded the dataset. No number in this
repository was generated by an AI; every reported value was produced by my own runs and
pasted back for interpretation, discussion, or drafting help.

## Statement of authorship

I, Jin Xinhui, confirm that I executed every cell in
`notebook/DS_A1_provenance_audit.ipynb` myself on my own machine (macOS, Python
3.10.18, conda environment), that the reported hashes, counts, and metrics reflect real
output from those runs, that I found and diagnosed the nine defects listed above in
AI-authored code or text before they were fixed, and that this log accurately discloses
which parts of the write-up were AI-drafted versus independently checked or corrected
by me.