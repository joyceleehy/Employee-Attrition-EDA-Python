# 📝 Project Notes — Python HR Attrition EDA

> Internal working notes documenting the full process of building this project, step by step. Useful for reviewing later, reproducing the workflow on future projects, or explaining the process in interviews.

---

## 1. Project Goal

Build a standalone Python Exploratory Data Analysis (EDA) project — separate from Power BI — to demonstrate the ability to clean, analyse, and visualise HR data using Python, and publish it as a portfolio piece on GitHub.

**Dataset:** IBM HR Analytics Attrition Dataset (Kaggle) — the same dataset previously used in the SQL project, allowing direct comparison of results across tools.

**Why this project:** To prove Python competency beyond Power BI's embedded Gemini AI integration, and to show recruiters a complete, independent technical workflow: setup → clean → analyse → visualise → document → publish.

---

## 2. Environment Setup

### 2.1 Project folder
Created in VS Code terminal:
```bash
cd %USERPROFILE%\Documents
mkdir python-hr-attrition-eda
cd python-hr-attrition-eda
code .
```

### 2.2 Virtual environment
Isolates project dependencies from the system Python install.
```bash
python -m venv .venv
.venv\Scripts\activate
```
Terminal prompt changes to `(.venv) PS ...` once active — confirms the environment is running.

### 2.3 Libraries installed
```bash
pip install pandas matplotlib seaborn jupyter
```

### 2.4 Folder structure
```
python-hr-attrition-eda/
├── charts/              # exported PNG chart files
├── data/                # raw CSV dataset
├── notebooks/
│   └── hr_attrition_eda.ipynb
├── .gitignore
├── README.md
└── requirements.txt
```

### 2.5 requirements.txt
Generated to let anyone reproduce the environment:
```bash
pip freeze > requirements.txt
```

### 2.6 .gitignore
Prevents the virtual environment and cache files from being pushed to GitHub:
```
.venv/
__pycache__/
.ipynb_checkpoints/
```

---

## 3. Loading the Dataset

The IBM HR Attrition CSV was downloaded from Kaggle and placed inside `data/`.

```python
import pandas as pd

df = pd.read_csv("../data/WA_Fn-UseC_-HR-Employee-Attrition.csv")
df.head()
```

Initial checks performed:
```python
df.shape            # rows and columns
df.info()           # data types and non-null counts
df.isnull().sum()   # missing values per column
df.duplicated().sum()  # duplicate rows
```

**Result:** Dataset was clean — no missing values or duplicates, consistent with it being a well-prepared sample dataset commonly used for learning.

---

## 4. Data Preparation

### 4.1 Creating income bands
Monthly income was grouped into bands to support the income-vs-attrition analysis:
```python
df['IncomeBand'] = pd.cut(
    df['MonthlyIncome'],
    bins=[0, 3000, 6000, 10000, 20000],
    labels=['Low', 'Medium', 'High', 'Very High']
)
```

### 4.2 Encoding Attrition as numeric (for calculations)
```python
df['AttritionFlag'] = df['Attrition'].map({'Yes': 1, 'No': 0})
```

---

## 5. Analysis — Business Question by Business Question

### 5.1 Overall attrition rate
```python
attrition_rate = df['AttritionFlag'].mean() * 100
print(f"Overall Attrition Rate: {attrition_rate:.2f}%")
```
**Result: 16.12%**

### 5.2 Attrition by department
```python
dept_attrition = df.groupby('Department')['AttritionFlag'].mean() * 100
dept_attrition.sort_values(ascending=False)
```

### 5.3 Overtime vs attrition
```python
overtime_attrition = df.groupby('OverTime')['AttritionFlag'].mean() * 100
print(overtime_attrition)
```
**Result: Overtime = Yes → 30.53% | Overtime = No → 10.44%**
→ Overtime employees leave at nearly **3× the rate**.

### 5.4 Income vs attrition
```python
income_attrition = df.groupby('IncomeBand')['AttritionFlag'].mean() * 100
income_attrition.sort_values(ascending=False)
```
**Result: Low income band → 28.61% attrition (highest)**

### 5.5 Job role attrition
```python
role_attrition = df.groupby('JobRole')['AttritionFlag'].mean() * 100
role_attrition.sort_values(ascending=False)
```
**Result: Sales Representative → 39.76% (highest of all roles)**

---

## 6. Visualizations

Each chart was built with seaborn/matplotlib and exported as a PNG into the `charts/` folder for use in the README and notebook.

Example — Overtime vs Attrition bar chart:
```python
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(6, 4))
sns.barplot(x='OverTime', y='AttritionFlag', data=df, estimator=lambda x: sum(x)/len(x)*100)
plt.title('Attrition Rate by Overtime Status')
plt.ylabel('Attrition Rate (%)')
plt.xlabel('Overtime')
plt.savefig('../charts/attrition_by_overtime.png', dpi=150, bbox_inches='tight')
plt.show()
```

The same pattern (groupby → plot → `plt.savefig()`) was repeated for:
- `overall_attrition_count.png`
- `attrition_by_department.png`
- `attrition_by_income_group.png`
- `income_vs_attrition_boxplot.png`
- `attrition_by_job_role.png`

**Note on chart sizing:** Charts initially rendered too large in the README. Standard markdown `![]()` syntax does not support resizing on GitHub — the fix is to control `figsize` and `dpi` at the point of saving in Python (e.g. `figsize=(6,4), dpi=150`), then re-export and re-push, rather than trying to resize after the fact in markdown.

---

## 7. Documentation — README.md

The README was iterated through several versions to make it more recruiter-friendly:

| Version | What Changed |
|---|---|
| v1 (GitHub auto-suggested) | Basic structure only |
| v2 | Added badges, key findings table, tools table |
| v3 (merged with ChatGPT draft) | Added Data Preparation, Recommendations, and Future Enhancements sections |
| v4 (final) | Displayed all 6 charts inline instead of just one |

**Key README sections included:**
- Badges (Python, Jupyter, pandas, seaborn, Status)
- Project Overview
- Notebook links (GitHub + NBViewer fallback, since GitHub's native `.ipynb` preview occasionally fails to render)
- Business Questions
- Data Preparation summary
- All 6 chart visualizations
- Key Findings table
- Recommendations (business-facing insights, not just stats)
- Tools & Libraries table
- Skills Demonstrated
- Project Structure
- Future Enhancements
- Dataset source credit + LinkedIn link

---

## 8. Git & GitHub Workflow

### 8.1 Standard update cycle used throughout the project
```bash
git add .
git commit -m "Descriptive message about the change"
git push
```

### 8.2 Checking sync status
```bash
git status
```
Useful for confirming whether local changes have actually been pushed before assuming there's a rendering bug on GitHub's side.

### 8.3 Lesson learned
When a GitHub README update didn't appear to take effect, the cause was browser caching rather than a git issue — confirmed via `git status` showing "working tree clean" and "up to date with origin/main". **Hard refresh (`Ctrl + Shift + R`) before assuming the push failed.**

### 8.4 Editing directly on GitHub
For minor README tweaks, editing directly in GitHub's web editor (pencil icon → edit → commit) is acceptable **as long as local and remote are already in sync** — followed by a `git pull` locally afterward to stay synced.

---

## 9. Publishing & Promotion

### 9.1 LinkedIn Projects section
Added as a standalone project entry (not tied to a specific employer) with:
- Title: *AI-Enhanced HR Attrition Analysis | Python EDA*
- Description including key findings, tools, and GitHub link
- Honest framing: described as a guided first Python EDA project rather than implying independent fluency — important for credibility during interviews

### 9.2 LinkedIn post
Drafted two versions (insight-led hook vs story-led hook) to announce the project to network and recruiters, tagged with relevant hashtags (#PeopleAnalytics #HRAnalytics #Python #DataAnalytics).

---

## 10. Key Findings Summary (for quick reference)

| Metric | Result |
|---|---|
| Overall attrition rate | 16.12% |
| Overtime vs non-overtime attrition | 30.53% vs 10.44% (≈3×) |
| Highest-risk job role | Sales Representative — 39.76% |
| Highest-risk income band | Low income — 28.61% |

---

## 11. Skills Practiced

- Python fundamentals in a standalone script/notebook context (vs. embedded Power BI usage)
- pandas: groupby, aggregation, binning (`pd.cut`), boolean mapping
- matplotlib/seaborn: bar charts, boxplots, figure sizing, exporting to PNG
- Git/GitHub: add–commit–push cycle, status checks, resolving sync confusion
- Markdown documentation for technical portfolios
- Translating statistical findings into business recommendations
- LinkedIn content writing for portfolio visibility

---

## 12. Possible Future Enhancements

- Build a simple logistic regression model to predict attrition risk
- Connect this analysis to the existing Power BI HR dashboard for a combined Python + BI case study
- Add employee tenure and demographic breakdowns
- Automate chart regeneration with a single script instead of manual cell-by-cell export

---

*Last updated: reflects project state as of the final README and chart-display version.*
