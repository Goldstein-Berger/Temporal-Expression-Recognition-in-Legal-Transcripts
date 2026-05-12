# Other Transcripts Dataset — Full Annotated File

## Overview

This dataset consists of sentence-level annotations derived from seven legal and scientific
transcripts. It is distributed as two types of files:

**Per-transcript files** (`other_transcripts_<case>_all_rows.csv`) — one file per transcript,
containing all sentences from the full pre-culled corpus for that case, with annotations merged
in where available. Seven files total.

**Annotated-only file** (`other_transcripts_annotated_only.csv`) — a single file containing
only the 4,892 sentences that were selected for annotation (TR + TST + UNLABELED rows), across
all seven transcripts.

Together the per-transcript files span 67,577 rows. All files share the same column structure
described below.

The dataset was constructed by:
1. Retaining the first 700 sentences of each transcript
2. Filtering for sentences containing temporal keywords to produce a concentrated subset of 677 sentences
3. Randomly splitting 80/20 into train (536 sentences) and test (136 sentences) sets
4. Merging gold labels back into the full pre-culled corpus, flagging unannotated sentences

### Cases

| Case | Type | Transcript Type |
|------|------|----------------|
| Gruber | Civil | Trial |
| Fauci | Civil | Deposition |
| Holmes | Criminal | Trial |
| Hermes | Civil | Trial |
| Gonzalez | Civil | Trial |
| Hill | Civil | Trial |
| Depp | Civil | Trial |

---

## Column Descriptions

### Provenance

| Column | Description |
|--------|-------------|
| `Full_Dataset_Row_Index` | Row number of this sentence in the full pre-culled source file (`other_transcripts_keywords.csv`). Can be used to trace any sentence back to its original position in the source corpus. |

### Sentence Text

| Column | Description |
|--------|-------------|
| `Sentence` | Raw sentence text as extracted from the transcript, including original whitespace and formatting artifacts (e.g., double spaces from PDF conversion, page-break markers). |
| `clean_text` | Cleaned version of the sentence with normalized whitespace and unicode. This is the version used for tokenization and model input. |
| `tokenized_text` | Whitespace-tokenized form of `clean_text` as a Python list. Token positions align directly with the BIOES tag sequences in `Silver_BIOES_NER_Tags` and `Gold_BIOES_NER_Tags`. |
| `Speaker` | Speaker label as it appeared in the transcript (e.g., `Q:`, `A:`, `THE COURT:`, `MR. KELLEY:`). Not available for all rows. |

### Case Metadata

| Column | Description |
|--------|-------------|
| `Case_Name` | Short name identifying the case (Gruber, Fauci, Holmes, Hermes, Gonzalez, Hill, Depp). |
| `Case_Type` | Legal classification: `Civil` or `Criminal`. |
| `Transcript_Type` | Type of proceeding: `Trial` or `Deposition`. |
| `Transcript_Date` | Date of the transcript session (ISO format). |

### Data Split

| Column | Description |
|--------|-------------|
| `Data_Split` | Partition assignment for each sentence. Values: `TR` (training set, 536 sentences), `TST` (test set, 136 sentences), `UNLABELED` (in the keyword-filtered subset but not annotated, 4,220 sentences), `UNUSED` (in the full pre-culled corpus but not selected into any split). |

### Silver NER Annotations (all TR, TST, and UNLABELED rows)

Silver labels were generated automatically using the [Flair NLP](https://github.com/flairNLP/flair)
named entity recognition model. They are available for all TR, TST, and UNLABELED rows.

| Column | Description |
|--------|-------------|
| `Silver_Flair_Date_Detected` | Binary flag: `1` if Flair detected at least one date/time entity in the sentence, `0` otherwise. |
| `Silver_Flair_Entities` | Full Flair entity output as a list of dicts, each with `text`, `score`, `start_pos`, and `end_pos`. |
| `Silver_BIOES_NER_Tags` | Silver BIOES tag sequence as a Python list, one tag per token. Tags follow the BIOES scheme: `B-DATE` (Begin), `I-DATE` (Inside), `O` (Outside), `E-DATE` (End), `S-DATE` (Single-token span). Aligns with `tokenized_text`. |

### Gold NER Annotations (TST rows only)

Gold labels were produced by two human annotators (EJG and Aleksandra) on the 136-sentence test
set. Where annotators disagreed (14 sentences), a consensus label was used.

| Column | Description |
|--------|-------------|
| `Gold_BIOES_NER_Tags` | Gold BIOES tag sequence as a Python list. Follows the same BIOES scheme as `Silver_BIOES_NER_Tags` and aligns with `tokenized_text`. Populated for TST rows only. |
| `Sentence_NER_Annotated` | Human-readable annotated sentence with date spans marked using `xxx...xxx` delimiters (e.g., `xxxthe prior fiscal yearxxx`). TST rows only. |

### Date Normalization Annotations (TST rows with date expressions)

Date normalization gold labels follow ISO 8601 extended format and were annotated on the 136-sentence
test set. Sentences without date expressions have `NaN` in these columns.

| Column | Description |
|--------|-------------|
| `Sentence_Date_Normalized` | ISO 8601 normalized form of each date expression in the sentence, separated by `,,,`. Ranges use `/` (e.g., `2011-XX-XX/2014-XX-XX`). Approximate or underspecified dates use `XX` for unknown components. |
| `Document_Date` | Document-level date used as context for normalizing relative or underspecified expressions (e.g., fiscal year references). ISO 8601 format. |
| `Anchor_Date` | Explicit anchor date(s) used to resolve relative expressions (e.g., `today`, `last year`). Multiple anchors separated by `;; `. `NA` where no external anchor was needed. |

---

## BIOES Tagging Scheme

| Tag | Meaning |
|-----|---------|
| `O` | Outside — token is not part of a date/time entity |
| `B-DATE` | Begin — first token of a multi-token date span |
| `I-DATE` | Inside — middle token of a multi-token date span |
| `E-DATE` | End — last token of a multi-token date span |
| `S-DATE` | Single — a date span consisting of exactly one token |

---

## Notes

- Token alignment: `tokenized_text` uses simple whitespace tokenization. All BIOES tag lists are
  aligned to this tokenization — the nth tag corresponds to the nth token.
- 8 TST sentences were not found in the full pre-culled corpus (due to sentence segmentation
  differences between the annotated and source files). In the per-transcript files these rows
  are appended at the end of the relevant case file. They have `NaN` for all full-dataset
  columns except `Sentence`, `Case_Name`, `Transcript_Date`, and annotation columns.
- `UNUSED` rows (sentences in the full pre-culled corpus not selected into any split) have
  `NaN` for all annotation columns. They appear in the per-transcript files but not in
  `other_transcripts_annotated_only.csv`.
- The merge script used to produce these files is included in this repository as
  `merge_annotations_into_full_dataset_colab.py`.
