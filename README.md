# Automotive Test Case Generator

An AI-powered tool that reads an automotive feature specification PDF and automatically generates structured test cases in CSV format — ready to import into test management tools like JIRA, qTest, or any HIL/SIL test framework.

---

## What It Does

- Reads a feature specification PDF (functional requirements, operational scenarios, interface & diagnostics)
- Learns your test case format from a sample Excel file
- Uses OpenAI GPT-4o to generate test cases written like an experienced automotive test engineer — not like a template
- Outputs a structured CSV with hierarchical rows (header → precondition → steps → postcondition) matching your exact format

---

## Output Format

Each test case is broken into multiple rows in the CSV:

| XP-nnn Source id | Test Case Detail Type | Step Description | Input conditions | Expected Output | Input signals | Output Signals |
|---|---|---|---|---|---|---|
| XP-100_001 | XP-100_001 | High-Speed Cut-In | Validates ACC deceleration when vehicle cuts in | | | |
| XP-100_001_01 | XP-100_001_01 | Precondition | Set ego speed to 100 km/h, ACC engaged, lane clear | System armed and tracking | | |
| XP-100_001_02 | XP-100_001_02 | Inject cut-in vehicle | Inject target at 80 km/h, 20m gap into ego lane | TTC drops below 2.0s | ego_speed, lane_id | TTC_value |
| XP-100_001_03 | XP-100_001_03 | Monitor BCM request | Check BCM deceleration request output | -3.0 m/s² request sent within 200ms | | BCM_decel_req |
| XP-100_001_last | XP-100_001_last | Postcondition | Ego vehicle decelerating, safe gap maintained | No DTC set, speed stabilized | | |

---

## Project Structure

```
automotive-test-case-generator/
│
├── generate_test_cases.py
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Setup

**1. Clone the repo**
```bash
https://github.com/Anushaakki/Test_case_generation_using_AIAgent
cd automotive-test-case-generator
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

**3. Create your `config.py`**

Create a file called `config.py` in the same folder and add your OpenAI API key:
```python
api_key = "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
```

**4. Add your files**

Place these in the same folder as the script:
- Your feature specification PDF
- Your sample test case Excel file

**5. Update the config variables in the script**

Open `generate_test_cases.py` and update these lines at the top:
```python
SPEC_PDF_PATH    = "your_feature_spec.pdf"
SAMPLE_XLSX_PATH = "your_sample.xlsx"
OUTPUT_CSV_PATH  = "test_cases_generated.csv"
MODEL            = "gpt-4o"   # or gpt-4o-mini to save cost
```

---

## Running

**In Jupyter Notebook:**
- Open `generate_test_cases.py` or paste the code into a notebook cell
- Run the cell

**From terminal:**
```bash
python generate_test_cases.py
```

**Expected output:**
```
[STEP 1] Reading feature spec PDF ...
         Extracted 4321 characters.

[STEP 2] Reading sample columns from xlsx ...
         Columns: ['XP-nnn Source id', 'Test Case Detail Type', ...]

[STEP 3] Generating test cases via OpenAI ...
[INFO] Calling OpenAI gpt-4o ...
[INFO] Generated 87 rows.

[STEP 4] Preview — first 10 rows:
...

[STEP 5] Saving to CSV ...
[INFO] CSV saved → test_cases_generated.csv

[DONE] All steps completed successfully.
```

---

## How It Works

```
Feature Spec PDF
      │
      ▼
Extract text (pdfplumber)
      │
      ▼
Read column schema from sample xlsx (pandas)
      │
      ▼
Build prompt with spec text + column schema
      │
      ▼
Call OpenAI GPT-4o → returns JSON rows
      │
      ▼
Parse JSON → insert empty separator rows between test cases
      │
      ▼
Write to CSV (preserving exact column order from sample)
```



## Dependencies

```
openai
pandas
openpyxl
pdfplumber
```

Install all at once:
```bash
pip install -r requirements.txt
```

---

## .gitignore

Make sure your `.gitignore` includes:
```
config.py
test_cases_generated.csv
__pycache__/
*.pyc
.env
```

---

## Notes

- Tested with OpenAI `gpt-4o`. You can switch to `gpt-4o-mini` in the config for lower cost.
- The tool uses `pdfplumber` for PDF text extraction — works best with text-based PDFs (not scanned images).
- Column names in the output CSV are driven entirely by your sample Excel file, so the format adapts automatically to different projects.
