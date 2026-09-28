# CLaRR: Cognitive-Load-Aligned Rewriting and Repair

CLaRR is a reader-conditioned text adaptation framework. Given a source text
and a reader profile, it plans and verifies changes along three dimensions:

- **prior knowledge**, which controls terminology retention and explanation;
- **presentation preference**, which controls textual organization and style;
- **reading goal**, which controls schema support and cognitive guidance.

The repository releases the CLaRR V4.2.1 reference implementation, the public
source-text corpus, the 18 controlled reader profiles, and a qualitative case
study containing outputs from seven rewriting methods.

## Repository contents

```text
.
├── CLaRR_v4.py                         # CLaRR V4 main entry point
├── CLaRR_ablation_v4.py                # unified V4 ablation entry point
├── internal/                            # runtime modules and ECL rubrics
├── scripts/                             # main, ablation, and experiment runners
├── examples/
│   ├── articles.json                    # small synthetic input example
│   ├── claims.json                      # example source-claim contract
│   ├── readers.json                     # 18 controlled reader profiles
│   └── simplifications.json             # approved simplification map
├── data/source_corpus/
│   ├── corpus.jsonl                     # 1,000 public source texts
│   ├── metadata.csv                     # provenance and license index
│   ├── summary.json                     # corpus statistics
│   ├── DATA_LICENSES.md                 # item-level reuse requirements
│   ├── CHECKSUMS.sha256                 # release checksums
│   └── verify_corpus.rb                 # offline corpus verifier
└── case_study/
    └── case_study_18_readers_7_methods.xlsx
```

## Code

The public code exposes one implementation version:

- `CLaRR_v4.py` runs the complete V4 pipeline.
- `CLaRR_ablation_v4.py` runs the `no_icl`, `no_ecl`, and
  `no_schema_support` conditions through one shared V4 interface.

The `internal/` directory contains required implementation components. These
files form one V4 runtime rather than separate public releases.

### Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

export LLM_API_KEY="your-key"
export LLM_BASE_URL="https://api.deepseek.com"
export LLM_MODEL="deepseek-v4-flash"
```

The client uses an OpenAI-compatible chat-completions API. Provider and model
settings can be changed through the environment variables above.

### Run the main method

```bash
python3 CLaRR_v4.py --article A01 --readers U-L-E
```

To process several profiles concurrently:

```bash
python3 CLaRR_v4.py \
  --article A01 \
  --readers U-L-E L-L-E P-L-E U-E-E L-E-E P-E-E \
  --workers 4
```

### Run an ablation

```bash
python3 CLaRR_ablation_v4.py \
  --variant no_ecl \
  --article A01 \
  --readers U-L-E \
  --workers 1
```

Valid variants are:

| Variant | Removed adaptation component |
|---|---|
| `no_icl` | terminology analysis, ICL strategy, and IL repair |
| `no_ecl` | presentation analysis, ECL strategy, and ECL repair |
| `no_schema_support` | schema-support contract, GCL strategy, and support repair |

Custom datasets can be supplied with `CLARR_ARTICLES_FILE`,
`CLARR_READERS_FILE`, `CLARR_CLAIMS_FILE`, and
`CLARR_SIMPLIFICATIONS_FILE`.

## Public source-text corpus

`data/source_corpus/corpus.jsonl` contains 1,000 English source texts used in
the experiments. The release contains 221,381 words in total, with 150--509
words per record.

| Property | Composition |
|---|---|
| Experimental splits | 200 development, 200 pilot, 600 confirmatory |
| Domains | 200 records each from computing/AI/engineering, economics/finance/social science, law/public policy, medicine/health, and natural science/environment |
| Genres | 250 records each from encyclopedia, institutional guidance, public explainer/news, and scholarly abstract sources |
| Providers | Wikipedia, government guidance, PMC, institutional news, and Wikinews |

Each JSONL record includes its source text, provenance, attribution, revision,
license, experimental split, word count, and SHA-256 digest. Verify the release
from `data/source_corpus/` with:

```bash
ruby verify_corpus.rb
shasum -a 256 -c CHECKSUMS.sha256
```

The corpus is a collection of third-party texts under multiple licenses. The
software license does not replace the item-level source terms. Consult
[`DATA_LICENSES.md`](data/source_corpus/DATA_LICENSES.md) and the license fields
in each record before reuse.

## Reader profiles

The evaluation crosses three knowledge levels, two presentation preferences,
and three reading goals, producing 18 controlled profiles:

| Case-study ID | Code ID | Knowledge | Presentation | Reading goal |
|---|---|---|---|---|
| R01 | `U-L-E` | Novice | Logical | Analytic |
| R02 | `L-L-E` | Novice | Logical | Learning |
| R03 | `P-L-E` | Novice | Logical | Narrative |
| R04 | `U-E-E` | Novice | Emotional | Analytic |
| R05 | `L-E-E` | Novice | Emotional | Learning |
| R06 | `P-E-E` | Novice | Emotional | Narrative |
| R07 | `U-L-M` | Intermediate | Logical | Analytic |
| R08 | `L-L-M` | Intermediate | Logical | Learning |
| R09 | `P-L-M` | Intermediate | Logical | Narrative |
| R10 | `U-E-M` | Intermediate | Emotional | Analytic |
| R11 | `L-E-M` | Intermediate | Emotional | Learning |
| R12 | `P-E-M` | Intermediate | Emotional | Narrative |
| R13 | `U-L-X` | Expert | Logical | Analytic |
| R14 | `L-L-X` | Expert | Logical | Learning |
| R15 | `P-L-X` | Expert | Logical | Narrative |
| R16 | `U-E-X` | Expert | Emotional | Analytic |
| R17 | `L-E-X` | Expert | Emotional | Learning |
| R18 | `P-E-X` | Expert | Emotional | Narrative |

The machine-readable definitions are available in
[`examples/readers.json`](examples/readers.json). These profiles are controlled
experimental conditions and do not represent real individuals.

The case-study workbook uses the historical labels `Analytic`, `Learning`, and
`Narrative`. In the runtime JSON, the corresponding `reading_motivation` values
are `utility`, `learning`, and `entertainment`, respectively.

## Case study

[`case_study_18_readers_7_methods.xlsx`](case_study/case_study_18_readers_7_methods.xlsx)
supports direct qualitative comparison across the 18 reader profiles. It uses
two fixed source texts from different domains and genres:

- **A081 — Cancer**, a medical encyclopedia passage;
- **A398 — Laboratory earthquake forecasting: A machine learning
  competition**, a scholarly abstract.

For every reader profile and source, the workbook presents outputs from seven
archived conditions:

1. GenDem
2. GenRead
3. PERCS-A1
4. PERCS-A2
5. PERCS-A5
6. B-RS
7. CLaRR

The workbook contains an `Index` sheet followed by one sheet for each profile
from `R01` to `R18`. Each profile sheet records the controlled condition, the
shared source text, the input anchor used by each method, and the seven generated
texts. This gives 252 generated examples in total: 18 profiles × 2 sources × 7
methods.

The case study is intended for inspecting how methods respond to the same
source and reader condition. It is not, by itself, an independent evaluation or
evidence that one method is uniformly better on every objective.

## Reproducibility notes

- Model outputs can vary with provider revisions, model availability, and API
  behavior even when prompts and decoding settings are fixed.
- The released code is the CLaRR V4.2.1 reference implementation. Historical
  batch outputs did not record a source commit hash, so the repository does not
  claim byte-identical regeneration of every archived output.
- Source-content fidelity, terminology adaptation, presentation alignment, and
  schema support should be evaluated separately. Improvement in one dimension
  should not be described as uniform superiority across all dimensions.

## Citation

Please cite the accompanying CLaRR paper and this repository release. Citation
metadata will be added after publication.
