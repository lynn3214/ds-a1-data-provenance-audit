# AI Use and Independent Verification Log — DS-A1

This log discloses AI assistance per the course's common requirement #5
("an AI-use record covering tool, task, material advice, acceptance/rejection, and
independent verification"). Entries below marked **[pre-filled]** reflect the actual
planning conversation; entries marked **[TODO — author]** must be completed by the
student after running the notebook themselves.

## Tool

- **Tool**: Claude (Anthropic), web/desktop chat interface, with web-search and code-
  execution tools enabled.
- **Session scope**: single planning + scaffolding conversation, dated 2026-09-27.

## Tasks the AI was used for **[pre-filled]**

1. Reading the course syllabus/assignments pages to confirm DS-A1's exact rubric and
   submission requirements.
2. Looking up `lerobot/svla_so101_pickplace` on the Hugging Face Hub (dataset card,
   `meta/info.json` schema, episode/frame counts, license, usage in the SmolVLA paper)
   to confirm the dataset was suitable (small, well-documented, clearly licensed)
   before committing to it.
3. Proposing the testable question, population/sample/unit/target/estimand framing,
   and the specific leakage-demonstration design (k-NN, frame-level vs episode-level
   split) as one concrete way to satisfy the rubric's "counterexample" and "leakage
   check" requirements.
4. Writing the notebook code scaffold (Sections 1–14 of
   `notebook/DS_A1_provenance_audit.ipynb`), the Data Card / data dictionary /
   source-license-hash templates, and this log's structure.

## Material advice given, and what was accepted vs. rejected

| AI suggestion | Accepted / Rejected | Why |
|---|---|---|
| Use `lerobot/svla_so101_pickplace` rather than a larger/multi-task LeRobot dataset | Accepted | Small size keeps local runs fast; single-task design makes the bias counterexample cleaner to argue. |
| Frame the leakage check as a live k-NN split-comparison rather than only a written description | Accepted | Produces a real, falsifiable number instead of an asserted risk. |
| `<TODO — author: e.g. "AI suggested hashing all 50 parquet files; I judged 2 files + all meta files sufficient for time budget and said so explicitly">` | `<TODO>` | `<TODO>` |
| `<TODO — author: record any other suggestion you changed, dropped, or overrode>` | `<TODO>` | `<TODO>` |

## What the AI did **not** do (explicitly, to avoid any ambiguity)

- The AI did not download the dataset, execute the notebook, or generate any of the
  numeric results reported in Sections 6–14. The code-execution environment used while
  drafting this repository has no network access to huggingface.co, so **every number
  in this submission was produced by the student's own local run**, not fabricated or
  pre-computed by the AI.
- The AI did not choose the final wording of Sections 2, 13, 14, and 15 in the
  submitted notebook — those must be written by the student after reading real output
  (see `<FILL IN>` markers left throughout the notebook).

## Independent verification performed by the author **[TODO — author, required]**

Describe concretely what you did to check the AI-authored code was correct rather than
just trusting it, e.g.:
- Re-ran the notebook top-to-bottom **twice** on a clean kernel and confirmed the
  printed numbers (frame counts, hash values, leakage gap) were identical both times.
- Manually spot-checked one episode's parquet file with `pandas.read_parquet` outside
  the notebook and confirmed its row count matched `meta/episodes.jsonl`.
- Manually inspected 2–3 rows of `action` vs `observation.state` to sanity-check the
  "near-identical" claim in Section 7 before writing it up.
- `<add your own concrete checks here>`

## Statement of authorship

I, `<FILL IN NAME / STUDENT ID>`, confirm that I executed every cell in
`notebook/DS_A1_provenance_audit.ipynb` myself on `<FILL IN date/machine>`, that the
reported hashes, counts, and metrics reflect real output from that run, and that
Sections 2, 13, 14, 15, and this log's TODO items are written in my own words based on
that output.
