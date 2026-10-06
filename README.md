# Delinquency Trend Analysis by Credit Grade Using SQL

## 📊 Project Overview
This project analyses over 2 million peer-to-peer loans from the Lending Club dataset (2007–2018) to examine delinquency, the stages a loan passes through on its way from performing to default. It measures how much of the live book is in arrears by grade, estimates the provision the arrears pipeline would require under IFRS 9, tests whether borrowers' past delinquency is already priced into the grade, builds a credit-quality transition matrix, and checks whether forbearance hides true stress. The analysis draws on rating-migration and Markov-chain concepts from the CQF credit risk module, implemented entirely in SQL.

## 🎯 Objective
To replicate the arrears monitoring a retail credit risk team performs: tracking delinquency buckets as early-warning indicators, linking them to IFRS 9 staging and expected loss, assessing whether grading and pricing capture borrower history, and identifying deterioration among loans that still appear healthy, presented as a portfolio-ready case study.

## 🧰 Tools Used
- SQL (SQLite via DB Browser for SQLite): CTEs, window functions, `VALUES` assumption tables, conditional aggregation
- GitHub (project documentation)
- Dataset: [Lending Club Loan Data 2007–2018](https://www.kaggle.com/datasets/wordsforthewise/lending-club) via Kaggle

```

## 🔄 Why Delinquency Matters
Default is the end of the road; delinquency is the road itself. A borrower typically moves through these stages before a loan is written off:

```
Current → Grace (1–15 DPD) → Late 16–30 DPD → Late 31–120 DPD → Default (121+ DPD) → Charged Off
```

Credit risk teams monitor these stages because they provide early warning, determine IFRS 9 staging and provisions, and show collections teams where to focus. This analysis looks at both sides of delinquency: the **current arrears status of each loan** and the **borrower's credit-bureau delinquency history** at application.

## 📌 Key Business Questions
1. How much of the live book is behind on payments, by grade?
2. What provision would the arrears pipeline require under IFRS 9?
3. After a borrower falls behind and catches up, how often do they default anyway?
4. Is a borrower's past delinquency already priced into the grade?
5. Does a recent or severe delinquency matter more than an old, minor one?
6. How does borrower credit quality migrate after origination?
7. Is a falling FICO score an early-warning signal?
8. How many loans that still appear Current show a significant increase in credit risk?
9. Is forbearance (hardship plans) hiding arrears?

## 🔍 Analysis Structure
**File:** `queries/05_delinquency_by_grade.sql`

| Query | Business Question | Technique |
|-------|-------------------|-----------|
| 0 | Setup: clean helper table | Text-to-number casting with blanks kept as NULL, ordered DPD buckets |
| 1 | Arrears pipeline by grade | 30+ DPD rate by count and by balance, multiple of grade A (window function) |
| 2 | Provision for the arrears pipeline | IFRS 9 stage mapping, roll-to-loss × LGD, coverage ratio |
| 3 | Re-default after cure | Late-fee flag as a cured-delinquency marker |
| 4 | Is past delinquency priced into grade? | Extra default and extra rate versus the Clean group within each grade |
| 5 | Recency and severity of delinquency | Months since last delinquency, 90+ DPD in the last 24 months |
| 6 | Credit quality migration | FICO transition matrix with an absorbing Default state |
| 7 | FICO drop as an early-warning signal | Arrears rate by FICO change band (live loans only) |
| 8 | Hidden deterioration among Current loans | SICR watch-list using 40- and 80-point FICO drop triggers |
| 9 | Forbearance and hidden arrears | Hardship plan uptake, broken plans and outcomes |

## 🧠 Methodology Notes
- **Single snapshot:** Lending Club provides loan status at one point in time (end of 2018), not monthly history. True roll rates (e.g. the share of 30 DPD loans that become 60 DPD next month) need two snapshots, so Query 2 uses **clearly labelled, illustrative roll-to-loss assumptions** that can be replaced with the 12-month PD and LGD estimated in the vintage analysis
- **IFRS 9 mapping:** Current = Stage 1; 1–30 DPD = Stage 1 watch-list; 30+ DPD = Stage 2 (rebuttable presumption); Default = Stage 3. Late 31–120 also contains some 90+ DPD loans that would be Stage 3, which a single snapshot cannot separate
- **Avoiding data leakage:** the latest FICO score is pulled after the loan outcome, so for defaulted loans a low score is a result of default rather than a predictor. Queries 7 and 8 therefore use live loans only
- **Cured delinquency:** a late fee is collected only when a borrower missed a payment and later paid, so Query 3 measures re-default after a cure rather than the full roll rate from first delinquency
- **Missing values:** blank CSV cells are kept as NULL rather than zero, so "never delinquent" is not confused with "delinquent 0 months ago"
- **Absorbing state:** in the transition matrix, a defaulted loan cannot migrate back, matching the "D" column of an agency rating migration matrix

## 💡 Concepts Applied
- **Markov chain and transition matrix:** the two-state default model and its extension to rating states; delinquency buckets act as intermediate credit states
- **Credit ratings migration:** rating transition matrix with an absorbing default state ( Hu et al., 2001)
- **Transitions between credit states:** PD extended from solvent-to-default to general credit-state transitions (Coleman, *A Practical Guide to Risk Management*)
- **Expected Loss (PD × LGD × EAD):** applied to the arrears pipeline through roll-to-loss rates
- **IFRS 9 staging and SICR:** 30 DPD backstop, credit-impaired Stage 3 and quantitative SICR triggers
- **Credit scoring:** testing which borrower-history variables add information beyond the grade ( Altman-style scoring)

## ✅ How to Reproduce
1. Download the Lending Club dataset from [Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club)
2. Open **DB Browser for SQLite** → New Database → name it `credit_portfolio.db`
3. Import the CSV: **File → Import → Table from CSV file** (tick "Column names in first line"). The table is named `accepted_2007_to_2018Q4` automatically
4. In the **Execute SQL** tab, run **Query 0** once to build the `delinquency_base` helper table, then click **Write Changes**
5. Highlight one query at a time and press **F5**
6. Optional: in Query 2, replace the illustrative roll-to-loss rates and LGD with your own estimates to test sensitivity

## 💼 Author
**Hannah (Huong) Tran** |
Financial Analysis |
MSc Banking & Finance with Distinction |
CFA Levels I & II cleared

[LinkedIn](https://www.linkedin.com/in/huong-hannah/) | [GitHub](https://github.com/Hannah-Tran)
