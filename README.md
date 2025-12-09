# Music Mental Health Analysis

This repository contains an R-based analysis of the **Music & Mental Health (MxMH) Survey** dataset from Kaggle. The project investigates how college students’ music-listening habits relate to their self-reported mental-health symptoms, with a particular focus on **depression**, **anxiety**, and **insomnia**.

## Research Questions

1. **RQ1 – Listening Time, Platforms, and Depression**  
   *How does depression score vary by hours spent listening to music per day and primary streaming service?*

2. **RQ2 – Predicting High Depression**  
   *Does listening more hours per day increase the likelihood of reporting high depression symptoms?*  
   (High depression defined as **score ≥ 7**.)

## Stakeholder

The primary stakeholder is the **Director of University Counseling and Wellness Services**.  
They want to know whether **music-listening behaviors** could serve as **low-burden indicators** of mental-health risk that might complement campus screening practices.

## Data

- **Source:** Music & Mental Health (MxMH) Survey (Kaggle)  
- **Type:** Cross-sectional self-reported survey  
- **Key variables:**
  - **Demographics:** `Age`
  - **Music behaviors:**
    - `Hours.per.day`
    - `Primary.streaming.service`
    - `Fav.genre`
    - Listening context: `While.working`, `Instrumentalist`, `Composer`, `Exploratory`, `Foreign.languages`
  - **Mental-health scores (0–10):**
    - `Anxiety`, `Depression`, `Insomnia`, `OCD`
  - **Music effects:** self-reported impact on mood (`Music.effects`)

### Data Cleaning (Summary)

Major data-cleaning steps implemented in the Quarto/R workflow:

- Dropped the **`BPM`** column due to:
  - Over 100 missing or invalid entries  
  - Unreliable interpretation  
- Removed:
  - 1 case with missing `Age`
  - 1 implausible outlier (`Age == 89`, `Hours.per.day == 24`)
- Standardized all **Yes/No** variables and removed invalid responses.
- Removed rows where `Music.effects` was blank.
- For **RQ1**, excluded:
  - Rows missing `Primary.streaming.service`
  - Respondents who selected *“I do not use a streaming service.”*
- For **Poisson modeling**, removed 2 rows with **non-integer depression scores**.
- Created a binary RQ2 outcome:  
  `high_depression = 1` if `Depression >= 7`, else `0`.

Final sample sizes:

- **RQ1 (Poisson model):** 646 respondents  
- **RQ2 (Logistic model):** 718 respondents

## Methods & Models

### RQ1 – Poisson Regression on Depression Score

**Outcome:**  
- `Depression` (0–10 count scale)

**Main predictors:**  
- `Hours.per.day`  
- `Primary.streaming.service`  
- Interaction: `Hours.per.day * Primary.streaming.service`

**Covariates:**  
`Age`, `While.working`, `Instrumentalist`, `Composer`, `Exploratory`,  
`Foreign.languages`, `Anxiety`, `Insomnia`, `OCD`, `Fav.genre`, `Music.effects`

**Model development:**

1. Fit linear regression → diagnostics showed **heteroscedasticity** and **nonlinearity**.
2. Switched to **Poisson regression** (appropriate for count outcomes).
3. Compared Poisson vs **Negative Binomial** to assess overdispersion (AIC difference small).
4. Checked:
   - GVIF for multicollinearity  
   - Partial residual plot for linearity on log scale  
   - Pearson residuals for model fit  

**High-level findings:**

- Depression varies **significantly by streaming platform**, not uniformly.
- Hours of listening **increase** expected depression score *only for some platforms* (e.g., Apple Music, Spotify).  
- For others (Pandora, YouTube Music, Other), the relationship is **flat or negative**.  
- **Anxiety** and **Insomnia** remain dominant predictors of depression level.

---

### RQ2 – Logistic Regression Predicting High Depression

**Outcome:**  
- `high_depression` (1 if depression ≥ 7)

**Main predictor:**  
- `Hours.per.day`

**Covariates:**  
`Age`, `Instrumentalist`, `Composer`, `Anxiety`, `Insomnia`, `OCD`, `Fav.genre`, `Music.effects`

**Model & Diagnostics:**

- Fit logistic regression with `glm(..., family = binomial)`.
- Checked:
  - Linearity of Hours.per.day on the log-odds scale (`crPlot`)
  - VIF for multicollinearity  
  - Hosmer–Lemeshow test (model fit acceptable)
  - Leverage and Pearson residuals  
- Evaluated predictive performance using:
  - Confusion matrix (threshold 0.50)
  - ROC curve & AUC (AUC ≈ **0.78**)
  - Optimal threshold (≈ 0.456) for better sensitivity

**Key findings:**

- **Listening hours are *not* a significant predictor** of high depression.  
- Strongest predictors:
  - **Anxiety** (OR ≈ 1.39)
  - **Insomnia** (OR ≈ 1.15)
- Students who say music **worsens their mood** have **4–5×** the odds of high depression.
- Favorite genres and musician status show weak or inconsistent associations.

---

## Main Conclusions

- **Listening time alone is not a reliable depression indicator.**
- **Platform differences matter**—some associations are positive, others negative.
- **Anxiety and insomnia** are far more informative than music behaviors.
- For counseling stakeholders:  
  ➜ Music data can provide *context*, but **validated mental-health measures** should remain central in screenings.

## Limitations

- Cross-sectional, self-reported data → no causal inference  
- Single-item psychological measures  
- Music behavior measured coarsely (hours, platform, favorite genre)  
- Potential unmeasured confounders (stress, sleep routines, academic pressure)

## Future Work

- Collect richer campus-specific data with PHQ-9, GAD-7  
- Add questions on **emotional motivation** for listening  
- Use longitudinal or mixed-methods approaches  
- Explore non-linear or machine-learning models  
- Integrate track-level audio features (valence, tempo, energy)

## Reproducibility & Setup

### Requirements

- **R** (>= 4.0)
- **Quarto** for rendering `.qmd`
- Required libraries:
```r
library(tidyverse)
library(dplyr)
library(ggplot2)
library(car)
library(broom)
library(kableExtra)
library(gridExtra)
library(pROC)
library(caret)
library(ResourceSelection)
