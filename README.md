# Colab Notebook — Accuracy Is Not Robustness

Companion notebook for the paper *Accuracy Is Not Robustness: Measuring and Defending Against
Realistic Evasion Attacks on Gradient-Boosted Phishing Website Detectors*
(Jatin Kumar and Nisarg Chasmawala, Birmingham City University).

**File:** `phishing_adversarial_robustness.ipynb` — 65 cells (31 markdown, 34 code), 337 KB.

## How to run it

1. Go to https://colab.research.google.com
2. **File → Upload notebook** and pick the `.ipynb`
3. **Runtime → Run all**

There is nothing to upload and nothing to configure. Standard CPU runtime; no GPU needed.

## Datasets

| | Dataset A | Dataset B |
|---|---|---|
| Source | Tan (2018), Mendeley Data | Vrbančič et al. (2020), *Data in Brief* 33:106438 |
| Download page | https://data.mendeley.com/datasets/h3cgnj8hft/1 | https://github.com/GregaVrbancic/Phishing-Dataset |
| Direct file | *(embedded in the notebook)* | https://raw.githubusercontent.com/GregaVrbancic/Phishing-Dataset/master/dataset_small.csv |
| Rows / features | 10,000 / 48 | 58,645 → 57,405 / 111 |
| Mutable / immutable | 19 / 29 | 76 / 35 |

**Why you will not hit an import error.** Dataset A sits behind a Mendeley download page, which is
awkward to automate, so the CSV is embedded in the notebook as a gzip + base64 blob and decoded
in memory. The notebook prints its MD5 (`971f9ccd17276801538a7573d4caae66`) so you can confirm it is
byte-identical to the original. Dataset B downloads from GitHub through a retry loop.

## Cell map

| Cells | What happens |
|---|---|
| Header | Title, objective, dataset table, runtime note |
| 1 | Dependency check and install (CatBoost, XGBoost, LightGBM) |
| 2 | Imports, seed, experiment settings |
| 3 | Load Dataset A from the embedded blob + checksum |
| 4 | Download Dataset B, remove duplicates |
| 5 | Mutable / immutable feature partition |
| 6 | Validity projection Π |
| 7 | The four detectors and the metrics helper |
| 8 | NumPy MLP surrogate |
| 9 | Constrained Boundary attack |
| 10 | Full pipeline function |
| 11–12 | Run both datasets |
| Results 1–7 | Clean performance, transfer attack, Boundary attack, feature reliance, problem-space evasion, defence, five-seed reliability |
| Figures 1–5 | Clean bars, transfer curves, Boundary curves + evasion cost, reliance + accuracy-vs-robustness, evasion trajectory + defence |
| Final | Save all results to JSON, conclusion, references |

## Runtime

About **8–15 minutes** on Colab, dominated by the Boundary attack on Dataset B and the five-seed
reliability check. If you only want a quick pass, set `N_ATTACK = 150` in the settings cell near the
top: the story is unchanged and it finishes roughly three times faster. Leave it at `400` to
reproduce the published tables.

## Verification record

Every code cell was executed in a clean directory before release.

- 34 / 34 code cells parse without syntax errors.
- Dependency cell tested in a container missing **all three** of CatBoost, XGBoost and LightGBM. It
  falls back to `--break-system-packages` when a plain `pip install` is refused, so it does not
  crash on managed runtimes.
- Dataset A decode and Dataset B download both verified from a directory containing nothing but the
  script.

Numbers reproduced against the paper:

| Result | Notebook | Paper |
|---|---|---|
| Dataset A clean accuracy | .9850 / .9875 / .9855 / .9865 | same |
| Dataset B clean accuracy | .9571 / .9498 / .9580 / .9455 | same |
| Transfer detection, Dataset A @ ε=0.05 | 68.4 / 69.3 / 69.4 / 66.6 | same |
| Transfer detection, Dataset B @ ε=1.0 | 88.1 / 73.4 / 76.1 / 77.8 | same |
| Boundary median L2, Dataset A | 2.64 / 1.85 / 2.03 / 1.58 | same |
| Mutable-importance share | A .584 / .760 — B .630 / .269 | same |
| Defence, Dataset A | 2.03 → 3.00 | same |
| Defence, Dataset B (backfire) | 5.73 → 3.67 | same |

## One honest caveat

The problem-space evasion cell searches the test set for an illustrative page and applies edit groups
until the verdict flips. It uses the same procedure as the paper, but because it runs against a
freshly trained model rather than a saved artifact, it usually settles on a different page. In a
verification run it picked page #7 and drove it from P = 0.987 to P = 0.005 after two edit groups,
whereas the paper reports a page going from P = 1.000 to P = 0.445 after three. The mechanism, the
edit groups and the conclusion are identical; only the specific page differs. If you need the paper's
exact page for a viva, note the page index the notebook prints and mention that it varies with the
training split.

## Files

```
phishing_adversarial_robustness.ipynb   the notebook (upload this to Colab)
build_notebook.py                       the script that generated it, if you want to edit and rebuild
```
