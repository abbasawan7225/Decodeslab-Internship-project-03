# Job Role → Skills Mapping

A small data-cleaning and analysis project that maps common tech job roles to
the skills typically required for them. Built as an internship project
deliverable.

## What it does

1. **`generate_data.py`** — creates a sample raw dataset (`data/raw_skills.csv`)
   with one row per job role and a free-text, comma-separated skills column.
2. **`clean_skills.py`** — cleans and reshapes the raw data:
   - strips stray whitespace from each skill
   - fixes inconsistent capitalization (e.g. `aws` → `AWS`, `tensorflow` →
     `TensorFlow`) so the same skill isn't counted twice under different
     spellings
   - removes duplicate skills within a role and empty entries from stray
     commas
   - outputs `data/cleaned_skills.csv` (long format: one row per
     `job_role, skill` pair) and `data/skills_matrix.csv` (a one-hot
     `job_role x skill` matrix, ready for ML / similarity use cases)
3. **`analyze_skills.py`** — basic exploratory analysis:
   - most in-demand skills across all roles
   - number of distinct skills required per role
   - saves a bar chart to `outputs/top_skills.png`
4. **`recommend.py`** — a content-based recommender: given a list of a
   user's skills, it TF-IDF vectorizes each role's skill set and ranks
   roles by cosine similarity to the user's input. Reads from the cleaned
   dataset by default (falling back to raw if cleaning hasn't been run
   yet), so recommendations aren't skewed by inconsistent casing/duplicate
   skills.

## Project structure

```
job-skills-project/
├── generate_data.py      # creates the raw sample dataset
├── clean_skills.py        # cleans + reshapes the data
├── analyze_skills.py      # exploratory analysis + chart
├── recommend.py            # TF-IDF + cosine similarity role recommender
├── requirements.txt
├── data/                   # generated CSVs (not committed if empty repo)
│   ├── raw_skills.csv
│   ├── cleaned_skills.csv
│   └── skills_matrix.csv
├── outputs/
│   └── top_skills.png
├── README.md
└── LICENSE
```

## Setup

```bash
pip install -r requirements.txt
```

## Usage

Run the three stages in order:

```bash
python generate_data.py
python clean_skills.py
python analyze_skills.py
```

Then get role recommendations for a set of skills:

```bash
python recommend.py Python "Cloud Computing" Automation
python recommend.py Python SQL "Machine Learning" Statistics --top-n 5
```

Skills are space-separated; quote any multi-word skill. At least 3 skills
are required for a reliable match. If you omit the argument entirely, it
runs with a small built-in sample skill set.

Each script can also be imported and reused, since the logic lives in
functions rather than top-level notebook code:

```python
from clean_skills import load_raw, to_long_format, to_skill_matrix

df_raw = load_raw("data/raw_skills.csv")
long_df = to_long_format(df_raw)
matrix_df = to_skill_matrix(long_df)
```

## Data quality notes

The original raw dataset stores skills as a single comma-separated string
per role. That's fine for display but awkward for analysis, so the cleaning
step explodes it into a long format and a one-hot matrix, and normalizes
casing/whitespace so that, for example, `AWS` and `aws` aren't treated as two
different skills.

## Possible extensions

- Replace the sample data with a real job-postings dataset (e.g. scraped or
  from a public dataset) by swapping out `generate_data.py`.
- Use `data/skills_matrix.csv` for a matrix-based similarity model as an
  alternative to the TF-IDF approach in `recommend.py`.
- Add unit tests for `normalize_skill()`, `to_long_format()`, and
  `TechStackRecommender.recommend()`.
- Wrap `recommend.py` in a small Flask/FastAPI endpoint or Streamlit app
  for an interactive demo.

## License

MIT — see [LICENSE](LICENSE).
