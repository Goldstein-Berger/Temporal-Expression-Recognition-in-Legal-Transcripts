# Temporal Expression Recognition in Legal Transcripts

**Goldstein, E. J. & Berger, M. (2026)**  
ORRO AI Genius / Ruhr University Bochum  
LREC 2026

This repository contains the datasets and annotations used in the paper *Temporal Expression Recognition in Legal Transcripts*. The data covers the 1995 criminal trial *People of the State of California v. Orenthal James Simpson* and includes train, development, and test splits annotated for Named Entity Recognition (NER) of date expressions and date normalization to ISO 8601 format.

---

## Citation

If you use this dataset, please cite:

```
Goldstein, E. J. & Berger, M. (2026). Temporal Expression Recognition in Legal Transcripts.
ORRO AI Genius / Ruhr University Bochum.
https://github.com/GoldsteinBerger/Temporal-Expression-Recognition-in-Legal-Transcripts
```

---

## Repository Contents

| File | Rows | Description |
|------|------|-------------|
| `train_simp011424_baseline_tag_trim.csv` | 23,978 | Training set — sentences containing date expressions, with silver NER tags |
| `dev_simp011424_baseline_tags_trim.csv` | 2,966 | Development set — sentences containing date expressions, with silver NER tags |
| `simpson_ner_span_identification.csv` | 2,930 | Test set — NER date span identification with gold labels |
| `simpson_date_normalization.csv` | ~291 | Test subset — gold date normalization with ISO values and anchor dates |
| `simpson_full_corpus_final_part1.csv` | 224,374 | Full 448,748-sentence corpus with gold labels merged in (part 1 of 3) |
| `simpson_full_corpus_final_part2.csv` | ~112,000 | Full corpus with gold labels (part 2 of 3) |
| `simpson_full_corpus_final_part3.csv` | ~112,000 | Full corpus with gold labels (part 3 of 3) |
| `simpson_FULL_creation_date_part1.csv` | 224,374 | Full corpus with session creation dates (part 1 of 2) |
| `simpson_FULL_creation_date_part2.csv` | 224,374 | Full corpus with session creation dates (part 2 of 2) |

> **Note:** The full corpus files are split for GitHub's file size limits. See [Reassembling Split Files](#reassembling-split-files) below.

---

## File Descriptions

### Train and Development Sets

**`train_simp011424_baseline_tag_trim.csv`** and **`dev_simp011424_baseline_tags_trim.csv`**

These files contain sentences from the OJ Simpson trial transcript that were identified as containing date expressions, annotated with silver NER tags produced by the FLAIR Ontonotes model.

| Column | Description |
|--------|-------------|
| `Sentences` | Raw tokenized sentence from the trial transcript |
| `INDX` | Original row index in the full 448,748-sentence corpus |
| `date_binary` | 1 = FLAIR predicted at least one date entity; 0 = no date predicted |
| `Tags_Clean` | Silver NER tags (BIOES format) — one tag per token |
| `Has_Date` | 1 = sentence contains at least one date entity |
| `Tags_Trim` | Silver NER tags paired with tokens (token + tag together) |

---

### Test Set — NER Span Identification

**`simpson_ner_span_identification.csv`** (2,930 rows)

The full NER test set. Contains silver tags from FLAIR and human gold-corrected tags. The first 1,600 rows are the official test set used in the paper's evaluation; the remaining 1,330 rows are annotated but were not included in the reported evaluation.

| Column | Description |
|--------|-------------|
| `Orig_Ind` | Original row index in the full corpus. Use to join with `simpson_date_normalization.csv` |
| `Sentences` | Raw tokenized sentence |
| `Doc_Creation_Date` | Date of the trial session (M/D/YYYY) |
| `date_binary` | 1 = FLAIR predicted a date; 0 = did not |
| `Tags_Clean` | Silver NER tags (BIOES format) |
| `GL_TAGS_DATES_CORRECTED` | Gold NER tags — human-corrected. Null where gold agreed with silver |
| `GL_NER` | **Primary gold label.** Uses `GL_TAGS_DATES_CORRECTED` where a correction exists; falls back to `Tags_Clean` where gold agreed with silver. Populated for official 1,600 test rows only |
| `Official_Test_NER` | 1 = row is in the official 1,600-sentence test set used in the paper |
| `Test_NER_Unused` | 1 = row was annotated but not included in the reported evaluation |
| `Has_Date` | 1 = sentence contains at least one date entity |
| `Has_DTE_GL` | Gold label confirms date present |
| `Has_DTE_CTN_GL` | Contains a definite Calendar date (DC), e.g., "June 12th", "1995" |
| `Has_DTE_REL_GL` | Contains a Relative date (REL), e.g., "last week" |
| `Has_DTE_DUR` | Contains a Duration (DUR), e.g., "three days", "24 hours" |
| `Has_DTE_UNCERTN` | Contains an Uncertain date (UNCRTN), e.g., "some time ago" |
| `Has_DTE_CREF` | Contains a Contextual cross-Reference (CREF), e.g., "that day" (referring to June 13, 1994) |
| `Has_DTE_SET` | Contains a recurring/Set date, e.g., "every Monday" |
| `Has_DTE_OTHER` | Contains a date not fitting other categories |
| `Has_Age` | Contains an age expression used as a date reference, e.g., "15-year-old" |

---

### Test Subset — Date Normalization

**`simpson_date_normalization.csv`** (~291 rows)

A subset of the test set additionally gold-labeled for date normalization. Each sentence has up to three date spans annotated with surface text, normalized ISO value, type code, and anchor date.

| Column | Description |
|--------|-------------|
| `Orig_Ind` | Links to `simpson_ner_span_identification.csv` and the full corpus |
| `Sentences` | Raw tokenized sentence |
| `Doc_Creation_Date` | Trial session date (M/D/YYYY) |
| `DATE1A_TXT` | Surface text of the first date span (e.g., "JUNE 12TH", "MONDAY") |
| `DATE1A` | Normalized ISO value for DATE1A (see ISO format section below) |
| `DATE1AT` | Date type code for DATE1A (see Date Type Codes below) |
| `LINK1A` | Linking preposition, e.g., "from", "between". Null if none |
| `DATE1B_TXT / DATE1B / DATE1BT` | Second endpoint of a range expression, if DATE1A begins a range |
| `DATE2_TXTA / DATE2A / DATE2AT` | Second independent date span in the sentence |
| `DATE3_TXT / DATE3 / Date3T` | Third independent date span in the sentence |
| `GL_ISO_DATES` | Resolved ISO date(s) for all spans, comma-separated |
| `GL_ISO_DATES_SENT_LEVEL` | Sentence-level ISO normalization using Doc_Creation_Date as anchor |
| `GL_TE` | Temporal expression subtype: Explicit / Relative / Vague |
| `GL_Types` | Category: Date / DUR / Set / Vague (comma-separated for multi-date sentences) |
| `GL_Prior_After` | Temporal direction preposition(s), e.g., "after", "since". "NONE" if absent |
| `Anchor_Date` | Resolved anchor date (YYYY-MM-DD) needed to interpret relative expressions |
| `Anchor_Type` | How the anchor was determined: `coref` (crime date, June 12–13 1994), `trans` (transcript session date), or `none` |

---

### Full Corpus Files

**`simpson_full_corpus_final_part1/2/3.csv`** (448,748 rows total, split across 3 files)

The complete 448,748-sentence trial transcript with gold NER labels and date normalization values merged in for the test set sentences. Non-test sentences have null values in gold label columns.

**`simpson_FULL_creation_date_part1/2.csv`** (448,748 rows total, split across 2 files)

The complete transcript with session creation dates. Use this file to look up the trial session date for any sentence in the corpus.

---

## Reassembling Split Files

```python
import pandas as pd

# Full corpus with gold labels
df = pd.concat([
    pd.read_csv("simpson_full_corpus_final_part1.csv", encoding="utf-8-sig"),
    pd.read_csv("simpson_full_corpus_final_part2.csv", encoding="utf-8-sig"),
    pd.read_csv("simpson_full_corpus_final_part3.csv", encoding="utf-8-sig"),
], ignore_index=True)

# Full corpus with creation dates only
df_dates = pd.concat([
    pd.read_csv("simpson_FULL_creation_date_part1.csv", encoding="latin1"),
    pd.read_csv("simpson_FULL_creation_date_part2.csv", encoding="latin1"),
], ignore_index=True)
```

---

## NER Tag Format (BIOES)

Each token is assigned one tag. The prefix indicates the token's position within an entity span:

| Tag | Meaning | Example |
|-----|---------|---------|
| `S-TAG` | Single-token entity | `MONDAY S-DATE` |
| `B-TAG` | Beginning of a multi-token entity | `24 B-DATE` |
| `I-TAG` | Inside (middle) of a multi-token entity | `hours I-DATE` |
| `E-TAG` | End of a multi-token entity | `day E-DATE` |
| `O` | Outside — not an entity | `THE O` |

Entity types used: `DATE`, `TIME`, `PERSON`, `ORG`, `GPE`, `CARDINAL`, `ORDINAL`

---

## Date Type Codes

| Code | Type | Example |
|------|------|---------|
| `DC` | Definite Calendar date | "June 12th", "1995", "Monday" |
| `REL` | Relative date | "last week", "the following day" |
| `DUR` | Duration | "three days", "24 hours" |
| `SET` | Recurring / set date | "every Monday", "24 hours a day" |
| `UNCRTN` | Uncertain date | "some time ago", "a while back" |
| `CREF` | Contextual cross-reference | "that day", "THE DAY" (referring to June 13, 1994) |
| `AMP` | Amplified / range-uncertain | — |
| `AGE` | Age expression used as date | "15-year-old" |
| `REL-DUR` | Relative duration | — |

---

## ISO Date Format (Modified ISO 8601)

This project uses ISO 8601 with `X` placeholders for unknown components:

| Format | Meaning |
|--------|---------|
| `1995-01-11` | Full known date |
| `XXXX-06-12` | Month and day known, year unknown |
| `1995-XX-XX` | Year known, month and day unknown |
| `XXXX-XX-XX-MON` | Day of week only (MON/TUE/WED/THU/FRI/SAT/SUN) |
| `P1D` | Duration: 1 day (ISO 8601 period notation) |
| `PT24H` | Duration: 24 hours |
| `PXW / PXM / PXY` | Duration: unknown count of weeks / months / years |
| `XXXX` | Date confirmed but year not fully resolvable |

---

## License

MIT License. See `LICENSE` for details.
