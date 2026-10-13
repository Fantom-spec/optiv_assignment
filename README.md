# Case Study 1: Vendor AI Usage Analysis (Optiv Consulting / Meridian Financial)

Public-sources-only method, working notebook, completed workbook and 14-slide deck. Nothing here contacted a vendor.

## What is in this folder

| Path | What it is |
|---|---|
| `Case_Study_1_Vendor_AI_Usage_Analysis.pptx` | 14-slide deck for a 10 to 15 minute walkthrough; speaker notes with timings are embedded |
| `Speaker_Notes.md` | The same notes as plain text |
| `Case_Study_1_Vendor_AI_Analysis.ipynb` | Working demonstration notebook (Colab or Jupyter); outputs from a real run are saved in it |
| `vendor_ai_pipeline.py` | The same code as a plain script |
| `Meridian_Vendor_Input_CS1_criticality.xlsx` | Meridian's workbook, completed (columns L to V for V-001 to V-006) plus the rule sheets and the Evidence Log |
| `Source_Verification.csv` | Per-source check of every Evidence Log row (see "Source verification") |
| `sample_run_output/` | Output of the REPLAY run saved in the notebook; safe to delete, re-running recreates it |
| `diagrams/` | Functional design (slide 3) and technical architecture (slide 4) as images |
| `tests/` | 20 automated tests plus a notebook-cell runner |
| `METHODOLOGY.md` | Scoring, evidence tags, coverage rules, what is fact vs inference vs assumption |
| `AUDIT_AND_CHANGES.md` | What was found wrong, what changed, remaining disagreements and manual steps |

Where each brief deliverable lives: functional design = slide 3; architecture = slide 4; source strategy and weighting = slides 7 to 9 and sheets *Source Strategy*, *Evidence Scoring*; criticality ratings = slides 5, 6 and sheets *Criticality Rubric*, *Criticality Scoring*; working demonstration = slide 12 and the notebook.

## Install and run

Python 3.10+.

```
pip install openpyxl pandas requests beautifulsoup4 python-pptx      # python-pptx only for the deck test
pip install ipywidgets jupyter                                       # only for the interactive upload widget
```

* **Notebook:** open `Case_Study_1_Vendor_AI_Analysis.ipynb` in Jupyter or Colab (upload the `.ipynb` and the `.xlsx` to the same folder; uncomment the `!pip` line in the first code cell on Colab), then Run all.
* **Script:** `python vendor_ai_pipeline.py` (runs REPLAY on the supplied workbook, prints the table and writes `outputs/`).
* **Programmatic:** `run_assessment("your_file.xlsx")`, or `analyze_upload(bytes, "name.xlsx", "REPLAY")` which is what the widget calls.
* **Team name:** set `TEAM_NAME` in `CONFIG` (first code cell). It is **not known to the tool, so the placeholder `[Team name]` remains**. Enter it in two places: `CONFIG["TEAM_NAME"]` (then re-run to regenerate outputs) and column V of `Vendor Inventory` in the supplied workbook (rows 6 to 11). Dates are DD-MM-YYYY, following the template.

### Modes
* `REPLAY` (default): evidence is read from the workbook's *Evidence Log* sheet; offline and deterministic. Use for the walkthrough.
* `LIVE`: crawls the vendor's own site (robots.txt respected, 1 s delay), optionally a search API (`TAVILY_API_KEY` or `BRAVE_API_KEY`) and SEC EDGAR; optional `ANTHROPIC_API_KEY` for LLM refinement. Set `CONFIG["USER_AGENT"]` to include a real contact email. It needs outbound HTTPS to the vendor sites. **It has not been run on the open internet** (see below).

### Input format
An `.xlsx` in the Meridian template layout: sheet `Vendor Inventory` with the header row containing Vendor ID, Vendor Name, Vendor Description, Service / Product Provided, Category, Public Website, Business Process Supported, Operational Dependency, Data Classification Accessed, Data Volume. In REPLAY the file must also have an `Evidence Log` sheet (otherwise the tool warns and every vendor shows NO-EVIDENCE with partial coverage). V-000 is held back as the benchmark.

## Tests (all were run; results in `AUDIT_AND_CHANGES.md`)
```
python -m unittest discover -s tests -v      # 20 tests, about 7 seconds, no internet needed
python tests/run_notebook_cells.py           # executes every notebook code cell and saves outputs into the .ipynb
```
The notebook also runs its own offline self-tests in its last cell.

## Results in one table (REPLAY, as of 11-10-2026)

| Vendor | Tier (score) | AI usage | Tag (E) | AI risk | Source classes required but not recorded |
|---|---|---|---|---|---|
| V-002 Fiserv | Critical (4.00) | Yes | CONFIRMED (0.99) | Critical | S2, S4, S5 |
| V-006 The Clearing House | Critical (3.50) | Inconclusive | NO-EVIDENCE (0) | Low | S1, S5 |
| V-005 BNY | High (3.00) | Yes | CONFIRMED (0.94) | High | S1, S2, S4, S6 |
| V-003 FSSI | High (2.75) | Inconclusive | NO-EVIDENCE (0) | None identified | S2, S3, S6 |
| V-001 AutomWorx | High (2.35, privileged-access floor) | Inconclusive | NO-EVIDENCE (0) | None identified | S1, S3, S6 |
| V-004 Terrapin | Medium (2.05) | Inconclusive | NO-EVIDENCE (0) | None identified | S2 |

Five different things are reported and must not be mixed up: **criticality** (how much Meridian depends on the vendor), **evidence confidence** (tag and E), **observed AI usage** (Yes / No / Inconclusive), **AI security risk** (tier x tag matrix), and **investigation status** (which source classes are still unrecorded). "None identified" risk with "Inconclusive" usage means: nothing found so far, and the review is not finished.

## Limits you should know before presenting
* **Coverage is partial for all six vendors.** The earlier claim of complete coverage for four vendors was not supported by the Evidence Log and was removed. Coverage is now computed from the log, never typed.
* **Source verification is limited.** Page fetching was not available, so no excerpt, date or tag was re-read against a live page. A search check confirmed that 18 of the 24 URL-bearing sources exist (title or URL returned); 6 could not be found; 6 further rows have no URL. See `Source_Verification.csv`.
* **The 6-of-6 comparison is a consistency check, not an accuracy test.** The criticality rules reproduce the analyst scores, and the analyst "AI usage" for V-001, V-003 and V-004 was changed from No to Inconclusive in this version because their coverage is incomplete (see `AUDIT_AND_CHANGES.md`).
* **LIVE was tested against a local mock web server only** (robots.txt Disallow respected, AI passages classified, unreachable site degrades cleanly). Real vendor sites were unreachable from the build environment. Run it once on your machine before relying on it.
* **The upload widget was tested through a stub widget library**, not real Jupyter; the underlying upload-and-classify function was tested directly.
* "No AI found" is never proof of absence; staff use of external AI assistants (for example at AutomWorx) cannot be seen from public sources.
* Weights and thresholds are proposals for Meridian's risk team to calibrate.
