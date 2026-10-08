# The Archive

**Pair:** Herve and Ovie  
**Repository:** https://github.com/ovieo-aoa/archive

---

## 1. The record

| Field | Type | Example | If it is unknown, we… |
| --- | --- | --- | --- |
| id | `str` | `MS001` | Reject the record immediately; `id` is a primary key and cannot be missing. |
| title | `str` | `"Codex Sinaiticus"` | Set to `None` to allow indexing and referencing of the manuscript by its `id`. |
| city | `str` | `"Florence"` | Set to `None` to preserve historical records with unknown provenance. |
| year | `int` | `1655` | Set to `None` so numeric filter queries omit it without throwing an error. |
| condition | `str` | `"good"` | Set to `"unexamined"` to distinguish missing data from physical damage checks. |

---

## 2. Our validation rules

| Field | Rule(s) | Rejects (example) |
| --- | --- | --- |
| id | Must match regex `^MS\d{3}$` (starts with 'MS' followed by 3 digits) | `"123"`, `"MS01"`, `""` |
| title | Non-empty string (`len > 0`) | `""`, `12345` |
| city | Non-empty string containing alphabetic characters | `""`, `999` |
| year | Integer bounded between the range $[0, 2026]$ | `-200`, `150000`, `"45 BC"` |
| condition | String belonging to `{'poor', 'fair', 'good', 'excellent'}` | `"broken"`, `""`, `"great"` |

### Who decided the year range?

We reject the year range of 1100–1900 for the core catalog dataset. We believe that an archive’s metadata architecture must serve history, not the limitations of common software defaults. Restricting allowable date ranges to a narrow historical window like 1100–1900 undermines the core mission of our archival system: to preserve, organize, and provide access to the complete record of human thought across all eras—from ancient antiquities and early medieval texts to 20th-century movements and modern contemporary records. The project only restricts records before 0, which is what we implemented.

---

## 3. The `c.1590` decision

**Our choice:** (c) Store `1590` plus a separate `approximate` flag.

**Why:** Storing `1590` as an integer preserves our core database functionality. It allows numerical operations such as chronological sorting and range queries (e.g., `WHERE year BETWEEN 1500 AND 1600`) without breaking type safety. Storing a separate boolean `approximate=True` metadata attribute also preserves the qualitative historical context without forcing the column into an unstructured string.

**What it costs us:** It increases schema complexity by introducing an extra field (`approximate: bool`). It also requires additional parsing logic during CSV ingestion to strip non-numeric prefixes (`"c."`, `"circa"`, `"~"`) and assign the boolean flag.

---

## 4. Our test table

### `validate_year`

| Test data | Value | Expected | Actual | Pass? |
| --- | --- | --- | --- | --- |
| Normal | `1655` | valid | valid | Yes |
| Abnormal | `"1655"` (string type) | invalid | invalid | Yes |
| Extreme (low) | `1100` | valid | valid | Yes |
| Extreme (high) | `1900` | valid | valid | Yes |
| Boundary (below) | `-25` | invalid | invalid | Yes |
| Boundary (above) | `2035` | invalid | invalid | Yes |

### `validate_condition`

| Test data | Value | Expected | Actual | Pass? |
| --- | --- | --- | --- | --- |
| Normal | `"good"` | valid | valid | Yes |
| Abnormal | `123` (non-string type) | invalid | invalid | Yes |
| Extreme (low) | `"poor"` | valid | valid | Yes |
| Extreme (high) | `"excellent"` | valid | valid | Yes |
| Boundary (below) | `"fairly good"` | invalid | invalid | Yes |
| Boundary (above) | `"mint"` | invalid | invalid | Yes |

---

## 5. Collaboration reflection

**Herve:** One thing my partner did that I will steal: Ovie's approach to data parsing in the storage logic was wonderful, especially how he cleanly stripped string prefixes like "c." and "circa" during ingestion while simultaneously mapping the estimated boolean flag. I’ll definitely steal that technique for handling messy input data gracefully without corrupting the core data types.
**Ovie:** One thing my partner did that I will steal: Herve's strict and defensive approach to validation is something I want to take note of especially how he used exact patterns and explicit boundary checks to catch invalid types and out-of-range values before they ever reach the storage layer. I’ll steal that habit of writing tight, self-contained validation logic for future works.
---

## 6. Declaration

- [x] Both of us can explain every line in this repository.
- [x] AI assistants used for explanation only, not to generate our implementation or our tests.

**If you used an AI assistant, say what you asked and what you did with the answer:**
We used an AI assistant for general assistance with the syntax of github code, which helped us to carry out basic github functions.

---

## Running this project

```bash
pytest -v                              # all tests
pytest tests/test_provided.py -v       # the given suite (if this does not work, try below command)
python -m pytest tests/test_provided.py -v # the given suite
pytest tests/test_yours.py -v          # your suite
python tools/check_collaboration.py    # your Part C report
