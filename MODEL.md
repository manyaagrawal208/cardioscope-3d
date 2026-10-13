# Model Card: CardioScope risk model

## Intended use
Education and visualisation of how common cardiac risk factors combine. **Not** for diagnosis, treatment or screening decisions.

## Inputs
| Input | Modality |
|-------|----------|
| Age, sex, systolic BP, BMI, resting heart rate | Clinical |
| LDL, HDL, hs-CRP | Laboratory |
| Peak exercise heart rate, ST depression, chest pain type | Exercise ECG and symptoms |
| Coronary artery calcium score | CT imaging |
| Smoking, diabetes, family history | History |

## CAD logit (`lg()`)
`logit = -4.6 + sum of terms`, and `P(obstructive CAD) = sigmoid(logit)`.

| Term | Contribution |
|------|--------------|
| Age | 0.055 per year above 50 |
| Male sex | +0.55 |
| Systolic BP | 0.012 per mmHg above 120 |
| LDL | 0.0045 per mg/dL above 100 |
| HDL | -0.015 per mg/dL above 50 |
| Smoking | +0.85 |
| Diabetes | +0.75 |
| BMI | 0.035 per unit above 25 |
| Family history | +0.5 |
| Chest pain | none 0, atypical 0.5, typical 1.0 |
| ST depression | 0.5 per mm |
| Exercise capacity | 0.02 per bpm that peak HR falls below 85% of (220 - age) |
| Calcium score | 0.5 x log10(1 + score) |
| hs-CRP | 0.12 per mg/L (capped at 10) |

The plausible range shown in the UI is `sigmoid(logit +/- 0.6)`.

## 10-year event risk (`r10()`)
`sigmoid(-9.3 + 0.07 age + 0.5 male + 0.011 (SBP-120) + 0.004 (LDL-100) - 0.02 (HDL-50) + 0.7 smoker + 0.7 diabetes + 0.15 min(CRP,10) + 0.45 log10(1+CAC) + 0.25 ST + 0.35 family history)`

## Mapping to coronary segments
`segment score = sigmoid(logit + artery offset + segment offset)`

| Artery | Offset | | Segment | Offset |
|--------|--------|-|---------|--------|
| Left main | -1.2 | | Proximal | +0.35 |
| LAD | +0.35 | | Mid | 0 |
| Left circumflex | 0 | | Distal | -0.45 |
| Right coronary | +0.10 | | (left main is one segment) | none |

Estimated narrowing = `min(95, round(100 x score^1.4))` percent. Risk categories: below 15% low, 15 to 50% intermediate, above 50% high.

## Example outputs (built-in presets)
Computed from the equations above.

| Preset | CAD probability | 10-year risk | Worst segment |
|--------|-----------------|--------------|---------------|
| Low | 0.3% | 0.1% | ~0% |
| Moderate | 26.6% | 4.5% | LAD proximal, ~30% |
| High | 81.9% | 29.1% | LAD proximal, ~86% |
| High, after quitting smoking, LDL 70, BP 120, BMI 24 | 44.7% | 9.8% | LAD proximal, ~51% |

## Limitations
- Coefficients are illustrative. They reflect the direction and rough size of well-known risk-factor effects, but they were **not** fitted to or validated on a patient dataset.
- Probabilities are not calibrated, so absolute values should not be read as clinical risk.
- Per-segment narrowing is derived from overall risk with fixed offsets, not measured from imaging.
- No fairness or subgroup evaluation has been done.
- The calcium score is treated as a continuous log term, with no age- or sex-specific percentiles.

## Path to a validated model
1. Train a logistic regression or gradient-boosted model on a public dataset such as Cleveland heart disease or Framingham.
2. Check discrimination (AUC) and calibration on a held-out set.
3. Export the model and replace `lg()` and `r10()`.
4. Report subgroup performance before any wider use.
