# Homework 1 — Data Mining for Business Analytics

**Course:** MBA/GBUS 738 — Data Mining for Business Analytics  
**Assignment:** Homework 1  
**Tool:** RapidMiner / Altair AI Studio  
**Dataset:** `stroke_data.csv`  
**Required process file:** `HW1.rmp`  
**Required written submission:** `HW1.docx` or `HW1.pdf`  

---

## 1. Assignment Objective

This homework uses a stroke-prediction dataset to practice core data-preparation and descriptive-analysis operations in RapidMiner / Altair AI Studio.

The original dataset contains **5,110 observations and 12 attributes**. Each row represents one patient, and the attributes include demographic, health, lifestyle, and stroke-status information.

The assignment requires you to:

1. clean the data;
2. calculate descriptive statistics;
3. filter observations;
4. aggregate grouped data;
5. normalize numerical attributes;
6. dummy-code categorical attributes;
7. keep all analyses in **one RapidMiner process** using the `Multiply` operator.

---

# 2. Important RapidMiner Concepts

| RapidMiner term | Meaning |
|---|---|
| **Example** | One row / one patient |
| **Attribute** | One column / variable |
| **ExampleSet** | A complete data table |
| **Retrieve** | Loads a dataset from the repository |
| **Select Attributes** | Keeps/removes columns |
| **Filter Examples** | Keeps/removes rows |
| **Aggregate** | Calculates statistics by group or for the whole dataset |
| **Multiply** | Creates several copies of the same ExampleSet |
| **Normalize** | Rescales numerical attributes |
| **Nominal to Numerical** | Converts categorical attributes into numeric variables |
| **res** | Result port |

A useful mental model is:

```text
Retrieve data
   ↓
Clean columns
   ↓
Clean rows
   ↓
Multiply clean dataset
   ↓
Run separate analyses for Q1–Q5
```

---

# 3. Required Initial Data Preparation

The assignment requires the same cleaned sample to be used for all later questions.

## 3.1 Load the dataset

Import `stroke_data.csv` into RapidMiner / Altair AI Studio and save it in the repository, for example as:

```text
stroke_data
```

Drag it onto the Design canvas.

RapidMiner creates:

```text
Retrieve stroke_data
```

This operator only loads the dataset.

---

## 3.2 Remove `id` and `gender`

Add:

```text
Select Attributes
```

Connect:

```text
Retrieve stroke_data
        ↓
Select Attributes
```

### Recommended configuration

```text
attribute filter type = subset
attributes = id, gender
invert selection = true
```

The assignment explicitly requires `id` and `gender` to be excluded from subsequent analysis.

`Select Attributes` works on **columns**.

After this step, the remaining analysis variables should include:

```text
age
hypertension
heart_disease
ever_married
work_type
Residence_type
avg_glucose_level
bmi
smoking_status
stroke
```

---

## 3.3 Remove invalid observations

Add:

```text
Filter Examples
```

Connect:

```text
Retrieve stroke_data
        ↓
Select Attributes
        ↓
Filter Examples
```

The assignment requires elimination of records where:

```text
bmi is missing
```

or:

```text
smoking_status = Unknown
```

The easiest way to implement this is to **keep only valid records**:

```text
bmi is not missing
AND
smoking_status != Unknown
```

Use an AND / `match all` condition.

### Logic check

| BMI | smoking_status | Keep? |
|---|---|---|
| Valid | Valid | Yes |
| Missing | Valid | No |
| Valid | Unknown | No |
| Missing | Unknown | No |

The output of this operator is the **master filtered sample**.

---

# 4. Add the `Multiply` Operator

The assignment requires all analyses to remain in one RapidMiner process.

Add:

```text
Multiply
```

after the cleaning steps:

```text
Retrieve stroke_data
        ↓
Select Attributes
        ↓
Filter Examples
        ↓
Multiply
```

Think of `Multiply` as a photocopier.

One clean dataset goes in. Several identical copies come out.

Recommended branches:

```text
Output 1 → Q1(1)
Output 2 → Q1(2)
Output 3 → Q2(1)
Output 4 → Q2(2)
Output 5 → Q3(1)
Output 6 → Q3(2)
Output 7 → Q4
Output 8 → Q5
```

---

# 5. Question 1 — Average and Standard Deviation

The assignment asks for the average and standard deviation of:

```text
avg_glucose_level
age
bmi
```

for:

1. the entire filtered sample;
2. only patients with `stroke = 1`.

## 5.1 Q1(1) — Entire Filtered Sample

Use the first `Multiply` output and add:

```text
Aggregate
```

Rename it if desired:

```text
Aggregate_Q1_1
```

Do **not** use any group-by attribute.

Add these aggregation calculations:

```text
avg_glucose_level → average
avg_glucose_level → standard_deviation

age → average
age → standard_deviation

bmi → average
bmi → standard_deviation
```

Connect the output to a `res` port.

### Reporting template

| Attribute | Mean | Standard Deviation |
|---|---:|---:|
| avg_glucose_level | [RapidMiner result] | [RapidMiner result] |
| age | [RapidMiner result] | [RapidMiner result] |
| bmi | [RapidMiner result] | [RapidMiner result] |

The mean is:

\[
\bar{x}=\frac{\sum x_i}{n}
\]

The standard deviation measures how dispersed observations are around the mean.

## 5.2 Q1(2) — Only Patients Who Had a Stroke

Use the second `Multiply` output.

Add:

```text
Filter Examples
```

Configure:

```text
stroke = 1
```

Then add:

```text
Aggregate
```

Connect:

```text
Multiply
   ↓
Filter Examples
[stroke = 1]
   ↓
Aggregate_Q1_2
```

Configure the same six statistics:

```text
avg_glucose_level → average
avg_glucose_level → standard_deviation
age → average
age → standard_deviation
bmi → average
bmi → standard_deviation
```

No group-by is required.

### Q1 final reporting table

| Attribute | Entire Sample Mean | Entire Sample SD | Stroke=1 Mean | Stroke=1 SD |
|---|---:|---:|---:|---:|
| avg_glucose_level | [ ] | [ ] | [ ] | [ ] |
| age | [ ] | [ ] | [ ] | [ ] |
| bmi | [ ] | [ ] | [ ] | [ ] |

---

# 6. Question 2 — Patients With `stroke = 0`

Both Q2 branches begin with:

```text
Filter Examples
stroke = 0
```

## 6.1 Q2(1) — Urban vs Rural

Use the third `Multiply` output.

Add:

```text
Filter Examples
```

Set:

```text
stroke = 0
```

Then add:

```text
Aggregate
```

Configure:

```text
group by = Residence_type
```

Count a reliably populated attribute, for example:

```text
age → count
```

### Expected output

| Residence_type | Count |
|---|---:|
| Rural | [ ] |
| Urban | [ ] |

### Answer template

> Among patients who did not have a stroke, there were more patients living in **[Urban/Rural]** areas.

## 6.2 Q2(2) — Married vs Not Married

Use the fourth `Multiply` output.

Again filter:

```text
stroke = 0
```

Then aggregate:

```text
group by = ever_married
```

and:

```text
age → count
```

### Expected output

| ever_married | Count |
|---|---:|
| No | [ ] |
| Yes | [ ] |

### Answer template

> **[Yes/No]**, there were more **[married/non-married]** patients among patients who did not have a stroke.

---

# 7. Question 3 — Analysis by `work_type`

## 7.1 Q3(1) — Highest Average Glucose Level

Use the fifth `Multiply` output.

Add:

```text
Aggregate
```

Configure:

```text
group by = work_type
avg_glucose_level → average
```

### Expected output

| work_type | Average avg_glucose_level |
|---|---:|
| Private | [ ] |
| Self-employed | [ ] |
| Govt_job | [ ] |
| children | [ ] |
| Never_worked | [ ] |

### Answer template

> The work type with the highest average `avg_glucose_level` is **[work type]**, with an average of **[value]**.

## 7.2 Q3(2) — Highest Stroke Proportion

Use the sixth `Multiply` output.

Add:

```text
Aggregate
```

Configure:

```text
group by = work_type
stroke → average
```

Because `stroke` is binary:

```text
0 = no stroke
1 = stroke
```

the average of `stroke` equals the proportion of stroke cases.

Example:

```text
0, 0, 1, 0, 1
```

gives:

\[
\frac{0+0+1+0+1}{5}=0.40=40\%
\]

### Answer template

> The work type with the highest stroke proportion is **[work type]**, with a proportion of **[value]**, equivalent to **[percentage]%**.

---

# 8. Question 4 — Z-Score Normalization

Use the seventh `Multiply` output.

Add:

```text
Normalize
```

Select only:

```text
bmi
avg_glucose_level
```

Choose:

```text
Z-transformation
```

or the equivalent Z-score option.

The formula is:

\[
z=\frac{x-\mu}{\sigma}
\]

After normalization:

\[
Mean \approx 0
\]

and:

\[
SD \approx 1
\]

To make reporting easy, add an `Aggregate` after Normalize:

```text
bmi → average
bmi → standard_deviation
avg_glucose_level → average
avg_glucose_level → standard_deviation
```

### Reporting template

| Normalized Attribute | Mean | Standard Deviation |
|---|---:|---:|
| bmi | [ ] | [ ] |
| avg_glucose_level | [ ] | [ ] |

A very small floating-point value such as `2.1E-16` should be interpreted as approximately zero.

---

# 9. Question 5 — Dummy Coding

Use the eighth `Multiply` output.

Add:

```text
Nominal to Numerical
```

Select:

```text
ever_married
Residence_type
smoking_status
```

Choose:

```text
coding type = dummy coding
```

No written numerical result is required, but this design must be present in `HW1.rmp`.

Dummy coding converts categorical values into 0/1 indicators so they can be used by numerical algorithms without creating a false ranking between categories.

---

# 10. Recommended Final Process Architecture

```text
Retrieve stroke_data
        ↓
Select Attributes
[remove id and gender]
        ↓
Filter Examples
[bmi not missing AND smoking_status != Unknown]
        ↓
Multiply
   │
   ├── Aggregate Q1(1)
   │
   ├── Filter stroke=1 → Aggregate Q1(2)
   │
   ├── Filter stroke=0 → Aggregate by Residence_type
   │
   ├── Filter stroke=0 → Aggregate by ever_married
   │
   ├── Aggregate by work_type → average(avg_glucose_level)
   │
   ├── Aggregate by work_type → average(stroke)
   │
   ├── Normalize → Aggregate mean + SD
   │
   └── Nominal to Numerical → dummy coding
```

---

# 11. Suggested Operator Naming

```text
Retrieve_stroke_data
Select_Attributes
Filter_Clean_Data
Multiply

Aggregate_Q1_1
Filter_Q1_Stroke1
Aggregate_Q1_2

Filter_Q2_Residence
Aggregate_Q2_1

Filter_Q2_Married
Aggregate_Q2_2

Aggregate_Q3_1
Aggregate_Q3_2

Normalize_Q4
Aggregate_Q4

DummyCoding_Q5
```

---

# 12. Final Written Answer Template

## Question 1

### Q1(1) Entire Filtered Sample

| Attribute | Mean | Standard Deviation |
|---|---:|---:|
| avg_glucose_level | [ ] | [ ] |
| age | [ ] | [ ] |
| bmi | [ ] | [ ] |

### Q1(2) Patients With `stroke = 1`

| Attribute | Mean | Standard Deviation |
|---|---:|---:|
| avg_glucose_level | [ ] | [ ] |
| age | [ ] | [ ] |
| bmi | [ ] | [ ] |

## Question 2

### Q2(1)

```text
Urban = [ ]
Rural = [ ]
```

> Among patients who did not have a stroke, there were more patients living in **[Urban/Rural]** areas.

### Q2(2)

```text
Married = Yes: [ ]
Married = No: [ ]
```

> **[Yes/No]**, there were more **[married/non-married]** patients among patients who did not have a stroke.

## Question 3

### Q3(1)

> The work type with the highest average `avg_glucose_level` is **[work type]**, with an average of **[value]**.

### Q3(2)

> The work type with the highest stroke proportion is **[work type]**, with a proportion of **[value]**, equivalent to **[percentage]%**.

## Question 4

| Attribute | Mean After Z-Score | SD After Z-Score |
|---|---:|---:|
| bmi | [ ] | [ ] |
| avg_glucose_level | [ ] | [ ] |

## Question 5

No numerical reporting is required. The RapidMiner process must include dummy coding for:

```text
ever_married
Residence_type
smoking_status
```

---

# 13. Common Mistakes to Avoid

- Using `Select Attributes` to remove rows. It works on columns.
- Using `Filter Examples` to remove columns. It works on rows.
- Forgetting to remove `Unknown` from `smoking_status`.
- Using OR instead of AND when keeping valid records.
- Using raw stroke counts instead of `average(stroke)` for Q3(2).
- Normalizing attributes other than `bmi` and `avg_glucose_level` for Q4.
- Forgetting Q5 because no written result is required.
- Creating several process files instead of the one required Homework 1 process.
- Forgetting to save the process as `HW1.rmp`.

---

# 14. Final QA Checklist

- [ ] `stroke_data.csv` imported correctly.
- [ ] `id` excluded.
- [ ] `gender` excluded.
- [ ] Missing `bmi` rows removed.
- [ ] `smoking_status = Unknown` rows removed.
- [ ] `Multiply` placed after data cleaning.
- [ ] Q1(1) mean and SD completed.
- [ ] Q1(2) filters `stroke = 1`.
- [ ] Q2 uses `stroke = 0`.
- [ ] Q2(1) groups by `Residence_type`.
- [ ] Q2(2) groups by `ever_married`.
- [ ] Q3(1) groups by `work_type` and averages glucose.
- [ ] Q3(2) groups by `work_type` and averages stroke.
- [ ] Q4 uses Z-score normalization.
- [ ] Q4 mean and SD reported.
- [ ] Q5 dummy coding included.
- [ ] Final process saved as `HW1.rmp`.
- [ ] Written answer saved as `HW1.docx` or `HW1.pdf`.
- [ ] Both files open successfully before submission.

---

# 15. Core Learning Summary

```text
Data ingestion
    ↓
Data cleaning
    ↓
Attribute selection
    ↓
Row filtering
    ↓
Descriptive statistics
    ↓
Grouped aggregation
    ↓
Proportion analysis
    ↓
Feature normalization
    ↓
Categorical encoding
```

The central lesson is:

> Build one reliable cleaned dataset first, then use that same data consistently across every analysis branch.
