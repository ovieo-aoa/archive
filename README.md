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
| year | Integer bounded in the inclusive range $[1100, 1900]$ | `1099`, `1901`, `"1655"` |
| condition | String belonging to `{'poor', 'fair', 'good', 'excellent'}` | `"broken"`, `""`, `"great"` |

### Who decided the year range?

We accept the year range of 1100–1900 for the core catalog dataset. Restricting records to this window protects data integrity by eliminating common data-entry typos (such as `19000` or `150`).

However, enforcing these boundaries incurs trade-off costs:
1. **1100 lower bound:** Throws away ancient, classical, and early-medieval manuscripts created before the 12th century (e.g., Carolingian texts or classical papyri).
2. **1900 upper bound:** Throws away modern 20th-century scholarly transcriptions, critical editions, and modern facsimiles of ancient texts.

We defend accepting these bounds because our dataset targets late-medieval to early-modern archival collections. Opening the range to arbitrary numbers would weaken automated error detection without adding value to our primary historical scope.

---

## 3. The `c.1590` decision

**Our choice:** (c) Store `1590` plus a separate `approximate` flag.

**Why:** Storing `1590` as an integer preserves core database functionality, allowing numerical operations such as chronological sorting and range queries (e.g., `WHERE year BETWEEN 1500 AND 1600`) without breaking type safety. Storing a separate boolean `approximate=True` metadata attribute preserves the qualitative historical context without forcing the column into an unstructured string.

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
| Boundary (below) | `1099` | invalid | invalid | Yes |
| Boundary (above) | `1901` | invalid | invalid | Yes |

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

**Herve:** One thing my partner did that I will steal: 
**Ovie:** One thing my partner did that I will steal: 
---

## 6. Declaration

- [x] Both of us can explain every line in this repository.
- [x] AI assistants used for explanation only, not to generate our implementation or our tests.

**If you used an AI assistant, say what you asked and what you did with the answer:**
We asked an AI assistant to explain the architectural trade-offs between storing dates as unstructured strings versus splitting them into integer years with a boolean flag. We used the explanation to evaluate query performance and wrote our own implementation, validation rules, and unit tests.

---

## Running this project

```bash
pytest -v                              # all tests
pytest tests/test_provided.py -v       # the given suite (if this does not work, try below command)
python -m pytest tests/test_provided.py -v # the given suite
pytest tests/test_yours.py -v          # your suite
python tools/check_collaboration.py    # your Part C report