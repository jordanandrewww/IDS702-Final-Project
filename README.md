# Music Mental Health Analysis

This repository contains an R-based analysis of the **Music & Mental Health (MxMH) Survey** dataset from Kaggle. The project examines how university students’ music-listening habits relate to self-reported mental-health symptoms, with a focus on **depression, anxiety, and insomnia**.

## Research Questions

1. **RQ1 – Listening Time & Platforms**  
   *How does the number of hours spent listening to music per day vary by depression status and primary streaming service?*

2. **RQ2 – Predicting High Depression**  
   *Does the number of hours spent listening to music each day increase the likelihood of reporting high depression symptoms?*

## Stakeholder

The primary stakeholder is the **Director of University Counseling & Wellness Services**.  
They are interested in whether **music-listening behaviors** can provide **low-burden behavioral indicators** of mental-health risk that could complement existing screening and outreach tools.

## Data

- **Source:** Music and Mental Health (MxMH) Survey Results (Kaggle)  
- **Type:** Cross-sectional, self-reported survey  
- **Key variables:**
  - Demographics (e.g., `Age`)
  - Music behaviors:
    - `Hours.per.day`
    - `Primary.streaming.service`
    - Favorite genre (`Fav.genre`)
    - Genre listening frequencies (e.g., `Frequency..Classical.`, `Frequency..Hip.hop.`, etc.)
    - Context variables (e.g., `While.working`, `Instrumentalist`, `Composer`, `Exploratory`, `Foreign.languages`)
  - Mental-health scores (0–10):
    - `Anxiety`, `Depression`, `Insomnia`, `OCD`

### Data Cleaning (Summary)

Key cleaning steps implemented in the Quarto/R script:

- Dropped **`BPM`** due to:
  - 100+ missing values
  - Inconsistent interpretation (“beats per minute of favorite genre” answered unreliably)
- Removed:
  - 1 row with missing `Age`
  - 1 implausible case: `Age == 89` and `Hours.per.day == 24` while working
- Restricted Yes/No variables (`While.working`, `Instrumentalist`, `Composer`, `Exploratory`, `Foreign.languages`) to valid `"Yes"` / `"No"` responses only.
- Removed rows with blank `Music.effects`.
- For **RQ1**, excluded:
  - Rows with missing `Primary.streaming.service`
  - Rows where `Primary.streaming.service == "I do not use a streaming service."`
- Standardized frequency variable names to `freq_*` format (e.g. `freq_Classical`, `freq_Hiphop`, etc.).
- Created a binary outcome for RQ2:  
  `high_depression = 1` if `Depression >= 6`, else `0`.

Final sample sizes:

- **RQ1 (linear regression models):** 645 respondents (after filtering + outlier removal)
- **RQ2 (logistic regression):** 718 respondents

## Methods & Models

### RQ1 – Linear Regression on Listening Time

**Outcome:**

- Raw: `Hours.per.day`
- Final main model: `log_hours = log(Hours.per.day + 0.1)`

**Key predictors:**

- `Depression`
- `Primary.streaming.service`  
- Interaction: `Depression * Primary.streaming.service`
- Covariates:
  - `Age`
  - `While.working`
  - `Instrumentalist`
  - `Composer`
  - `Exploratory`
  - `Foreign.languages`
  - `Anxiety`, `Insomnia`, `OCD`
  - `Fav.genre`
  - `Music.effects`

**Model development:**

1. **Model A – Raw-scale model:**  
   Linear regression on `Hours.per.day` with all predictors.
2. **Model B – Reduced model:**  
   Removed weakly justified predictors (genre frequency variables) for parsimony.
3. **Model C – Log-transformed model:**  
   Uses `log(Hours.per.day + 0.1)` to reduce skew and improve residual behavior.
4. **Final model – Log + outlier removal:**  
   - Identified outliers using diagnostic plots and **Cook’s distance**.
   - Removed three influential observations (`rownames` 407, 345, 537).
   - Refit the log model (`model_rq1_log_final`).

**Diagnostics & Fit:**

- Checked:
  - Linearity
  - Homoscedasticity
  - Normality of residuals
  - Influence (Cook’s distance)
  - Multicollinearity (GVIF / VIF)
- Compared models via:
  - Adjusted \(R^2\)
  - AIC (see “Table 1” in the report)

**High-level findings:**

- Higher **depression scores** are associated with **more listening time**, but:
  - The effect varies by **streaming platform** (interaction terms).
- **Listening while working** shows the largest positive association with listening time.
- **Insomnia**, **composer status**, **instrumentalist status**, **age**, and some favorite genres (e.g. Jazz, Latin) also show significant effects.
- The final log-transformed model with outlier removal gives a more stable and interpretable fit.

### RQ2 – Logistic Regression on High Depression

**Outcome:**

- Binary: `high_depression` (1 if `Depression >= 6`, 0 otherwise)

**Main predictor:**

- `Hours.per.day` (daily listening time)

**Covariates:**

- `Age`
- `Instrumentalist`, `Composer`
- `Anxiety`, `Insomnia`, `OCD`
- `Fav.genre`
- `Music.effects`

**Model & Diagnostics:**

- Fit logistic regression using `glm(..., family = binomial)`.
- Checked:
  - Linearity of `Hours.per.day` on the log-odds scale (via `crPlot`)
  - Multicollinearity (VIF)
  - Overall model fit (Hosmer–Lemeshow test)
  - Leverage and residuals
- Evaluated predictive performance:
  - Confusion matrices at:
    - Default threshold 0.50
    - Optimized threshold ≈ 0.456 (from ROC curve)
  - Metrics: Accuracy, Sensitivity, Specificity, Precision, F1, Balanced Accuracy
  - **AUC ≈ 0.80**, indicating good discrimination.

**Key findings:**

- **Hours.per.day is *not* a significant predictor** of high depression:  
  - OR ≈ 1.05, 95% CI [0.99, 1.12], p ≈ 0.14
- **Anxiety** and **insomnia** are the strongest predictors:
  - Anxiety: OR ≈ 1.40 (each point increases odds of high depression by ≈ 40%)
  - Insomnia: OR ≈ 1.17
- Music-related variables (favorite genre, musician status, music effects) do **not** show robust, significant associations with high depression.
- Likelihood ratio test comparing:
  - Full model (with `Hours.per.day`)
  - Reduced model (without `Hours.per.day`)
  shows **no significant improvement** from including listening time.

## Main Conclusions

- Students with **higher depression scores** tend to listen to **more music per day**, but this pattern depends on **streaming platform** and other contextual factors.
- **Daily listening hours alone are not a reliable indicator** of high depression risk once co-occurring symptoms and basic covariates are included.
- **Anxiety and insomnia** provide far more informative predictive value for high depression than total listening time.
- For stakeholders (e.g., university counseling services), **brief questions about music use** may still be useful as supporting context, but **screening tools should prioritize validated mental-health indicators** rather than relying on listening time alone.

## Limitations

- **Self-reported and cross-sectional** data:
  - No causal inference
  - Susceptible to recall and reporting bias
- **Single-item measures** for key constructs (e.g., depression, anxiety, music effects).
- **Coarse music behavior measures**:
  - Total hours per day
  - Primary streaming platform
  - Do not capture motivation, emotional purpose, or context of listening.
- **Unmeasured confounders** such as:
  - Socioeconomic status
  - Academic stress
  - Physical health
  - Sleep quality (beyond insomnia score)
  may bias estimates.

## Future Work

Potential extensions include:

- Collecting **richer, campus-specific data** with:
  - Validated multi-item mental-health scales (PHQ-9, GAD-7, etc.)
  - More detailed music-use questions (motivation, time-of-day, emotion regulation).
- Using **longitudinal** or **experimental** designs to clarify causal relationships.
- Exploring **more flexible models** (GAMs, random forests, gradient boosting) to capture non-linearities and complex interactions.
- Incorporating **streaming log data** or track-level features (tempo, valence, energy) to better characterize music behavior.

## Reproducibility & Setup

### Requirements

- **R** (version ≥ 4.0 recommended)
- **Quarto** (if rendering `.qmd` to HTML)
- R packages (as loaded in the script):

```r
library(tidyverse)
library(dplyr)
library(ggplot2)
library(car)               # Diagnostics (VIF, CR plots)
library(broom)             # Tidy model output
library(kableExtra)        # Table formatting
library(gridExtra)         # Multi-plot grids
library(pROC)              # ROC curves
library(caret)             # Confusion matrix and metrics
library(ResourceSelection) # Hosmer–Lemeshow test
