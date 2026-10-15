<div align="center">

# Which Reliability Signal to Trust? Output vs. Representation Space Uncertainty Under Distribution Shift in Pathology Foundation Models

**Official implementation of the MI4MedFM @ MICCAI 2026 workshop paper**

**Md. Noman Biswas Sibly<sup>1</sup> · Shadril Hassan Shifat<sup>2</sup> · Md. Shakil Ahmed<sup>3</sup> · Md Kishor Morol<sup>2</sup>**

<sup>1</sup> Rajshahi University of Engineering & Technology<br>
<sup>2</sup> ELITE Research Lab, Queens, NY, United States<br>
<sup>3</sup> Green University of Bangladesh

<br>

![Venue](https://img.shields.io/badge/MI4MedFM-MICCAI%202026-6f42c1)
![Status](https://img.shields.io/badge/status-camera--ready-2da44e)
![Framework](https://img.shields.io/badge/framework-PyTorch-ee4c2c?logo=pytorch&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-0969da)

[**Official paper**](https://papers.miccai.org/miccai-2026-sat/MI4MedFM_010.html) · [**PDF**](https://papers.miccai.org/miccai-2026-sat/paper/MI4MedFM_010.pdf) · [**Method**](#method) · [**Results**](#results-from-the-paper) · [**Code**](#code-and-setup) · [**Reproduce**](#reproducing-the-paper) · [**BibTeX**](#citation)

</div>

<blockquote>
<p align="justify"><strong>TL;DR:</strong> Output-space and representation-space uncertainty capture different failure modes. Under stain shift, classifier reliability deteriorates while embedding geometry clearly moves; under organ shift, geometry changes little; and in open-set detection, the stronger signal depends on the unseen class.</p>
</blockquote>

## Paper

|  |  |
|---|---|
| **Venue** | Mechanistic Interpretability for Medical Foundation Models (MI4MedFM), MICCAI 2026 Workshops |
| **Official paper page** | [MICCAI 2026 Open Access](https://papers.miccai.org/miccai-2026-sat/MI4MedFM_010.html) |
| **Open-access PDF** | [Download paper](https://papers.miccai.org/miccai-2026-sat/paper/MI4MedFM_010.pdf) |
| **Official code** | [shadril238/trustshift-pfm](https://github.com/shadril238/trustshift-pfm) |
| **Foundation model** | [CONCH](https://github.com/mahmoodlab/CONCH), frozen throughout |
| **Tasks** | Nucleus classification, uncertainty estimation, distribution-shift analysis, open-set detection |
| **Benchmarks** | PanNuke and MoNuSAC |

## Abstract

<p align="justify">Reliable uncertainty estimation is essential for deploying pathology foundation models, yet the behaviour of different reliability measurements under distinct forms of distribution shift remains poorly understood. We investigate this problem using a frozen CONCH pathology foundation model with a lightweight linear probe on the PanNuke and MoNuSAC nucleus classification benchmarks. We compare output-space reliability measurements with representation-space measurements under controlled appearance, semantic, and open-set distribution shifts. Our experiments reveal a shift-dependent asymmetry between the two reliability families. Appearance shift severely degrades confidence and calibration while representation geometry remains informative, whereas semantic shift degrades confidence more mildly and leaves representation geometry largely unchanged. Open-set experiments further show that neither reliability family consistently dominates across unseen classes. These findings demonstrate that no single reliability measurement remains universally reliable and motivate jointly monitoring classifier outputs and representation geometry for more trustworthy deployment of pathology foundation models. To support reproducibility, we publicly release the experimental code in the GitHub repository.</p>

## At a glance

| Component | Configuration |
|---|---|
| **Encoder** | Frozen CONCH pathology foundation model |
| **Input** | Ground-truth-mask-centred `64 x 64` nucleus crops |
| **Representation** | Cached 512-dimensional embeddings |
| **Classifier** | Class-balanced multinomial logistic-regression probe |
| **Output-space signals** | MSP, energy, ECE, confidence-based selective prediction |
| **Representation-space signals** | Mahalanobis distance and k-NN label disagreement (`k=20`) |
| **Distribution shifts** | HED stain perturbation, held-out organs/tissues, leave-one-class-out novelty |

## Method

<p align="center">
  <img src="assets/pipeline.png" width="95%" alt="Experimental pipeline for output-space and representation-space reliability evaluation">
</p>

<p align="justify">Each annotated nucleus is converted into a mask-centred <code>64 x 64</code> crop and encoded by the frozen CONCH feature extractor. Baseline embeddings are cached; appearance-shift variants are re-embedded after perturbation. A multinomial logistic-regression probe is trained on standardized 512-dimensional embeddings with L2 regularization and balanced class weights. The CONCH backbone is never fine-tuned.</p>

<p align="justify">The study evaluates two complementary reliability families:</p>

- **Output space:** maximum softmax probability (MSP), energy score, expected calibration error (ECE), and confidence-based selective prediction.
- **Representation space:** shared-covariance Mahalanobis distance with Ledoit-Wolf shrinkage and k-NN label disagreement with `k=20`.

<p align="justify">Three shift settings are considered:</p>

- **Appearance shift:** HED stain perturbation at `sigma = {0.05, 0.10, 0.20, 0.35}`.
- **Semantic shift:** unseen organs or tissues are excluded from training and representation-statistics estimation, then used only for evaluation.
- **Open-set shift:** one cell class is excluded from training and treated as novel at test time.

## Results from the Paper

### Key findings

- **Appearance shift:** confidence and calibration can fail silently while Mahalanobis distance detects a strong population-level representation shift.
- **Semantic shift:** output reliability degrades more mildly, while the representation-space response remains comparatively small.
- **Open-set novelty:** neither reliability family dominates; the strongest signal reverses across held-out classes.
- **Deployment implication:** monitor output and representation signals jointly instead of treating either as universally reliable.

### In-distribution baseline

| Metric | PanNuke | MoNuSAC |
|---|---:|---:|
| Balanced accuracy (95% bootstrap CI) | 0.734 [0.720, 0.747] | 0.807 [0.787, 0.828] |
| Macro-F1 | 0.68 | 0.73 |
| ECE after temperature scaling | 0.017 | 0.026 |
| Accuracy at 70% coverage | 0.82 | 0.98 |

### Shift-dependent asymmetry

<p align="justify">PanNuke demonstrates the central result. Stain perturbation strongly displaces the embeddings while calibration and accuracy deteriorate. Under organ shift, balanced accuracy still falls substantially, but the change in Mahalanobis distance is comparatively small.</p>

| Condition | Balanced accuracy | ECE | Mean Mahalanobis distance |
|---|---:|---:|---:|
| In distribution | 0.734 | 0.017 | 308 |
| Stain, `sigma=0.05` | 0.533 (0.008) | 0.184 (0.005) | 803 (4) |
| Stain, `sigma=0.10` | 0.388 (0.005) | 0.330 (0.003) | 1,491 (6) |
| Stain, `sigma=0.20` | 0.268 (0.008) | 0.470 (0.006) | 2,687 (12) |
| Stain, `sigma=0.35` | 0.228 (0.004) | 0.503 (0.005) | 3,606 (11) |
| Held-out tissues | 0.449 (0.020) | 0.118 (0.039) | 390 (56) |

<p align="justify">Values in parentheses are standard deviations over five stain seeds or five held-out-tissue draws. The mildest stain condition retains higher balanced accuracy than the held-out-tissue condition, so the contrast is not explained only by the strongest corruption being more severe.</p>

### Open-set detection

<p align="justify">AUROC under leave-one-class-out evaluation confirms that output and representation signals are complementary. Bold values mark the strongest signal for each held-out class.</p>

| Dataset | Held-out class | MSP | Energy | Mahalanobis |
|---|---|---:|---:|---:|
| PanNuke | Neoplastic | **0.623** | 0.596 | 0.565 |
| PanNuke | Inflammatory | 0.572 | **0.692** | 0.574 |
| PanNuke | Connective | 0.639 | **0.676** | 0.633 |
| PanNuke | Dead | 0.602 | 0.371 | **0.933** |
| PanNuke | Epithelial | **0.674** | 0.533 | 0.499 |
| MoNuSAC | Epithelial | 0.803 | **0.845** | 0.755 |
| MoNuSAC | Lymphocyte | 0.765 | **0.809** | 0.741 |
| MoNuSAC | Macrophage | 0.574 | 0.602 | **0.891** |
| MoNuSAC | Neutrophil | 0.640 | 0.721 | **0.806** |

<p align="justify">The winner changes both across and within datasets. In particular, energy is below chance for the held-out PanNuke <code>Dead</code> class while Mahalanobis distance reaches 0.933 AUROC. Conversely, output-space scores lead for several abundant classes.</p>

## Datasets

| Dataset | Data used | Tissues/organs | Classes | Nuclei | Train / validation / test |
|---|---|---:|---:|---:|---:|
| [PanNuke](https://arxiv.org/abs/2003.10778) | Part 1, 2,656 images | 19 | 5 | 51,365 | 31,187 / 7,495 / 12,683 |
| [MoNuSAC](https://monusac-2020.grand-challenge.org/) | Train and test splits | 4 | 4 | 46,686 | 27,416 / 7,822 / 11,448 |

<p align="justify">All partitions are grouped by source image to prevent nucleus-level leakage. The datasets are strongly imbalanced: <code>Dead</code> contributes 790 PanNuke nuclei, while <code>Macrophage</code> and <code>Neutrophil</code> contribute 893 and 791 MoNuSAC nuclei, respectively.</p>

<p align="justify">For semantic shift, the protocols differ by dataset:</p>

- **PanNuke:** six of nineteen tissues are held out at random; results are averaged over five draws. This is a multi-tissue holdout, not leave-one-organ-out.
- **MoNuSAC:** strict leave-one-organ-out over Breast, Kidney, Lung, and Prostate. `Macrophage` and `Neutrophil` are merged into `Other`, producing a three-class problem whose values are not directly comparable with the four-class baseline.

## Code and setup

### Repository layout

```text
trustshift-pfm/
|-- assets/
|   `-- pipeline.png
|-- notebooks/
|   |-- 01_pannuke_conch_embeddings.ipynb
|   |-- 02_pannuke_reliability.ipynb
|   |-- 03_monusac_conch_embeddings.ipynb
|   `-- 04_monusac_reliability.ipynb
|-- LICENSE
`-- README.md
```

### Installation

<p align="justify">The notebooks were developed on Kaggle with an NVIDIA T4 GPU. A typical environment requires:</p>

```bash
pip install torch timm transformers==4.40.0 scikit-learn numpy Pillow scikit-image datasets matplotlib tqdm
pip install "git+https://github.com/mahmoodlab/CONCH.git"
```

<p align="justify">CONCH is gated. Request access through the <a href="https://huggingface.co/MahmoodLab/CONCH">CONCH model page</a>, then add the token to Kaggle Secrets as <code>HF_TOKEN</code>:</p>

```python
from kaggle_secrets import UserSecretsClient

token = UserSecretsClient().get_secret("HF_TOKEN")
```

<p align="justify">No Hugging Face token, model weights, source images, or cached embeddings are stored in this repository.</p>

## Reproducing the paper

<p align="justify">Run the notebooks in this order:</p>

| Step | Notebook | Purpose | Approximate T4 runtime |
|---:|---|---|---:|
| 1 | [`01_pannuke_conch_embeddings.ipynb`](notebooks/01_pannuke_conch_embeddings.ipynb) | Build PanNuke crops and cache frozen CONCH embeddings | 1 h 40 min |
| 2 | [`02_pannuke_reliability.ipynb`](notebooks/02_pannuke_reliability.ipynb) | PanNuke probe, calibration, geometry, open-set, stain, and tissue-shift experiments | Minutes from cache; longer when re-embedding perturbed crops |
| 3 | [`03_monusac_conch_embeddings.ipynb`](notebooks/03_monusac_conch_embeddings.ipynb) | Build MoNuSAC crops and cache frozen CONCH embeddings | 1 h 25 min |
| 4 | [`04_monusac_reliability.ipynb`](notebooks/04_monusac_reliability.ipynb) | MoNuSAC probe, calibration, geometry, open-set, stain, and organ-shift experiments | Minutes from cache; longer when re-embedding perturbed crops |

<p align="justify">Notebooks 2 and 4 consume the <code>.npz</code> caches produced by notebooks 1 and 3. Their paths follow Kaggle&#x27;s attached-notebook-output layout, so update the glob patterns when running elsewhere. The committed notebooks retain their outputs, allowing the reported values to be inspected without downloading CONCH or rerunning the experiments.</p>

### Paper-to-code map

<p align="justify">Cell indices below are zero-based and refer to the committed notebooks.</p>

| Paper item | Notebook | Cell |
|---|---|---:|
| Table 1: dataset counts and splits | `01_pannuke_conch_embeddings` / `03_monusac_conch_embeddings` | 4 / 5 |
| Table 2: PanNuke baseline and calibration | `02_pannuke_reliability` | 4, 6 |
| Table 2: MoNuSAC baseline and calibration | `04_monusac_reliability` | 4 |
| Table 3: per-class F1 | `02_pannuke_reliability` / `04_monusac_reliability` | 4 / 4 |
| Table 4: PanNuke / MoNuSAC open-set AUROC | `02_pannuke_reliability` / `04_monusac_reliability` | 10 / 7 |
| Table 5 and Figure 2: PanNuke stain and tissue shifts | `02_pannuke_reliability` | 13, 14 |
| Figure 3: selective classification | `02_pannuke_reliability` | 6, 13, 14 |
| Section 4.5: per-sample error detection | `02_pannuke_reliability` | 15 |
| Appendix Table 6: per-class geometry | `02_pannuke_reliability` / `04_monusac_reliability` | 9 / 6 |
| Appendix Table 7: MoNuSAC organ folds | `04_monusac_reliability` | 8 |
| MoNuSAC stain-severity analysis | `04_monusac_reliability` | 11 |

## Reproducibility notes and scope

- Main appearance-shift sweeps use five perturbation seeds on both datasets. The auxiliary per-sample gate analysis in PanNuke notebook cell 15 uses three stain seeds because each run re-embeds the full test set; it is not the source of the main severity table.
- Confidence intervals use 1,000 nucleus-level bootstrap resamples. Because multiple nuclei may originate from one image, these intervals should be interpreted as descriptive rather than cluster-robust.
- MoNuSAC stain-shift estimates vary considerably across perturbation seeds; the committed outputs in notebook 4, cell 11 report the standard deviations.
- Mahalanobis distance clearly detects appearance shift at the population level but is only a weak per-sample error detector: AUROC is 0.577 at `sigma=0.05`, 0.614 at `sigma=0.10`, and 0.531 under PanNuke tissue shift.
- The evidence is limited to a frozen CONCH encoder, a linear probe, two nucleus-classification datasets, and synthetic HED perturbations. It does not establish that the same asymmetry holds for other encoders, tasks, scanners, or institutions.
- Paper figures were assembled separately. The underlying values are preserved in the notebook outputs identified above.

## Citation

<p align="justify">If this work is useful in your research, please cite the <a href="https://papers.miccai.org/miccai-2026-sat/MI4MedFM_010.html">official MICCAI paper</a>:</p>

```bibtex
@InProceedings{SibMd._Which_MICCAISAT2026,
  author    = {Sibly, Md. Noman Biswas and Shifat, Shadril Hassan and Ahmed, Md. Shakil and Morol, Md Kishor},
  title     = {{Which Reliability Signal to Trust? Output vs. Representation Space Uncertainty Under Distribution Shift in Pathology Foundation Models}},
  booktitle = {Medical Image Computing and Computer Assisted Intervention -- MICCAI 2026 Workshops and Challenges},
  year      = {2026},
  publisher = {Springer Nature Switzerland},
  volume    = {LNCS 17261},
  url       = {https://papers.miccai.org/miccai-2026-sat/MI4MedFM_010.html}
}
```

<p align="justify">The MICCAI page has not yet assigned a DOI, month, or page range.</p>

## Contact

<p align="justify">For questions about the paper or code, contact:</p>

- **Shadril Hassan Shifat**, Project Lead - [shadril.hassan@elitelab.ai](mailto:shadril.hassan@elitelab.ai)
- **Md Kishor Morol**, Corresponding Author - [kishor@elitelab.ai](mailto:kishor@elitelab.ai)

## Acknowledgements

<p align="justify">This work uses the <a href="https://github.com/mahmoodlab/CONCH">CONCH</a> pathology foundation model from the Mahmood Lab. CONCH is gated and distributed under its own licence, independently of this repository.</p>

## License

<p align="justify">The code in this repository is released under the <a href="LICENSE">MIT License</a>.</p>
