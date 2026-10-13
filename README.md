<p align="center">
  <img src="images/banner.svg" alt="CardioScope 3D" width="100%">
</p>

<p align="center">
  <a href="https://cardioscope-3d.netlify.app"><img alt="Live demo" src="https://img.shields.io/badge/Live%20demo-cardioscope--3d.netlify.app-e0364b?style=for-the-badge&logo=netlify&logoColor=white"></a>
</p>

<p align="center">
  <img alt="Hackathon" src="https://img.shields.io/badge/Multimodal%20AI%20Hackathon-2026-6f42c1">
  <img alt="Track" src="https://img.shields.io/badge/Track-A%20Cardiac%20Risk-e0364b">
  <img alt="Three.js" src="https://img.shields.io/badge/Three.js-r128-000000?logo=threedotjs&logoColor=white">
  <img alt="JavaScript" src="https://img.shields.io/badge/JavaScript-vanilla-f7df1e?logo=javascript&logoColor=black">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-2ea44f">
</p>

<p align="center">
  <b>Predict coronary artery disease risk from multimodal patient data and see it painted onto an interactive, beating 3D heart.</b><br>
  <a href="https://cardioscope-3d.netlify.app">Live demo</a> ·
  <a href="ARCHITECTURE.md">Architecture</a> ·
  <a href="docs/MODEL.md">Model card</a> ·
  <a href="#run-locally">Run locally</a>
</p>

---

## Screenshots

| Low-risk patient | High-risk patient | Risk summary and what-if |
|:---:|:---:|:---:|
| <img src="images/screenshot-low-risk.png" alt="Low-risk heart" width="300"> | <img src="images/screenshot-high-risk.png" alt="High-risk heart" width="300"> | <img src="images/screenshot-panel.png" alt="Risk panel" width="300"> |

## Why this exists

Most cardiac risk tools return a single percentage. That hides two things people want to know: **where** in the heart the risk sits, and **why** the number is what it is. CardioScope 3D puts both in one view. It combines clinical, lab, ECG and imaging inputs, predicts risk, and maps it onto the coronary arteries of a 3D heart you can rotate and inspect.

## Features

- 🫀 **Interactive 3D heart.** It beats at the patient's heart rate. The left main, LAD, left circumflex and right coronary arteries are split into 9 segments, each coloured green to red by risk. Vessels narrow and plaque appears as risk rises.
- 🧬 **Multimodal inputs.** Clinical data, labs (LDL, HDL, hs-CRP), exercise ECG findings, and a CT calcium score.
- 📊 **Predictions.** Obstructive CAD probability with a plausible range, 10-year cardiac event risk, and per-segment narrowing estimates.
- 🔍 **Explainable.** A driver chart shows which factors raise or lower risk, and which modifiable factor matters most.
- 🔁 **What-if simulator.** Quit smoking, lower LDL, blood pressure or BMI, and watch both the numbers and the 3D heart change.
- 📈 **Synthesised ECG** that reflects heart rate and ST depression.
- 🖱️ **Controls.** Drag to rotate, scroll to zoom, hover a segment for its details, preset views, opacity slider, three patient presets, light and dark themes.

## How it works

```
Patient inputs ─► risk model ─► per-artery and per-segment scores ─► 3D vessel colour, width and plaque
(clinical, labs,   (explainable    (LAD highest, left main lowest,
 ECG, CT calcium)   logistic)       proximal highest)
```

The full data flow, layers and design decisions are in [ARCHITECTURE.md](ARCHITECTURE.md). Every coefficient and the segment mapping are in [docs/MODEL.md](docs/MODEL.md).

## Example results

| Preset | CAD probability | 10-year risk | Worst segment |
|--------|:---:|:---:|---|
| Low risk | 0.3% | 0.1% | ~0% |
| Moderate | 26.6% | 4.5% | LAD proximal, ~30% |
| High risk | 81.9% | 29.1% | LAD proximal, ~86% |
| High risk after quitting smoking, LDL 70, BP 120, BMI 24 | 44.7% | 9.8% | LAD proximal, ~51% |

## Tech stack

| Area | Choice |
|------|--------|
| 3D rendering | Three.js r128 (procedural geometry, no model files) |
| App | Vanilla JavaScript, HTML, CSS in a single file |
| Fonts | Fraunces and Instrument Sans (Google Fonts, with fallbacks) |
| Hosting | Netlify (static) |

## Run locally

**Prerequisites:** a modern browser with WebGL, and an internet connection on first load (Three.js and fonts load from CDNs). Nothing to install and no build step.

1. Clone the repo: `git clone https://github.com/<your-username>/cardioscope-3d.git`
2. Open `index.html` in your browser, or serve it locally:
   ```
   python -m http.server 8000
   ```
   then visit http://localhost:8000.
3. Click a preset (low, moderate, high) or move the sliders on the left.

## Replacing the model

The prediction layer is two functions, `lg(v)` (CAD logit) and `r10(v)` (10-year risk). Swap them for a trained model, such as logistic regression or gradient boosting on the Cleveland or Framingham datasets, and the 3D layer and UI work unchanged.

## Limitations

> ⚠️ **Research and education prototype. Not a medical device. Do not use for diagnosis or treatment.**

- The coefficients are illustrative and were not trained or validated on a patient cohort, so absolute probabilities are not clinically calibrated.
- Per-segment narrowing is derived from overall risk, not measured from imaging.
- The 3D heart is stylised, not patient-specific.

## Roadmap

- [ ] Train and calibrate the model on public datasets
- [ ] Report discrimination, calibration and subgroup performance
- [ ] Import real CT-derived coronary geometry
- [ ] Uncertainty-aware outputs

## Repository layout

| Path | Purpose |
|------|---------|
| `index.html` | The whole app: UI, risk model, 3D scene |
| `ARCHITECTURE.md` | System architecture and data flow |
| `docs/MODEL.md` | Model card |
| `images/` | Banner and screenshots |
| `LICENSE` | MIT license |

## Author

**Manya Agrawal**, built solo for the Multimodal AI Hackathon 2026 (IIT Mandi, KamandPrompt).

## License

MIT, see [LICENSE](LICENSE).
