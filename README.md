
[![Hugging Face](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-blue)](https://huggingface.co/hdnndh/NEMO-Underwater_Finetuned_DepthAnythingV2)
[![Downloads](https://img.shields.io/badge/dynamic/json?url=https://huggingface.co/api/models/hdnndh/NEMO-Underwater_Finetuned_DepthAnythingV2&query=downloads&label=Downloads)](https://huggingface.co/hdnndh/NEMO-Underwater_Finetuned_DepthAnythingV2)
[![Likes](https://img.shields.io/badge/dynamic/json?url=https://huggingface.co/api/models/hdnndh/NEMO-Underwater_Finetuned_DepthAnythingV2&query=likes&label=Likes)](https://huggingface.co/hdnndh/NEMO-Underwater_Finetuned_DepthAnythingV2)
[![Demo](https://img.shields.io/badge/%F0%9F%A4%97%20Demo-Space-brightgreen)](https://hdnndh-nemo-underwater-demo.hf.space/)

# Nemo-MDE

nemo.. bc.. underwater...

This repository presents a real-data underwater metric depth model using **Monocular Depth Estimation** on **Sea-thru** and **SQUID**.

## Demo

Demo can be found in the huggingface space
The model produces accurate **relative** depth out of the box, but absolute metric values may need correction for your specific underwater environment (different water type, camera, depth range).

The [demo Space](https://hdnndh-nemo-underwater-demo.hf.space/) includes a **built-in calibration tool**:

1. Upload **1–10 underwater images** where you have known ground-truth depth (from stereo, SfM, or a depth sensor).
2. Upload the **matching depth maps** (grayscale, pixel values = meters).
3. Click **Calibrate** — we solve for the optimal `scale` and `shift` via least squares so that `corrected = scale × predicted + shift`.
4. All subsequent predictions automatically apply the correction.

Even 1–3 pairs from your target environment can dramatically improve absolute accuracy. The tool reports R² so you can judge fit quality.

## Overview

Underwater metric depth estimation remains difficult because visibility degradation changes appearance in ways that are very different from standard in-air imagery. This work evaluates a metric-depth model in underwater scenes with real underwater RGB-depth supervision, comparing two pretraining initializations (Indoor vs. Outdoor) and measuring the impact of geometric augmentation on cross-dataset generalization.

## Dataset

Experiments use two publicly available underwater depth datasets:

- **Sea-thru** — 1,205 image–depth pairs from five Red Sea dive sites (D1–D5), with SfM depth maps in meters.
- **SQUID** — 57 stereo pairs from four sites spanning two bodies of water:
  - *Katzaa* (Red Sea coral reef, 10–15 m) and *Satil* (Red Sea shipwreck, 20–30 m)
  - *Nachsholim* (Mediterranean, 3–6 m) and *Mikhmoret* (Mediterranean, 10–12 m)

Split sizes (proportional within each source, ~80/10/10%):

- **Train:** 1,010
- **Validation:** 126
- **Test:** 126

## Training Details

Both Indoor and Outdoor variants use identical training: AdamW optimizer, cosine annealing, 20 epochs, effective batch size 64, mixed precision, and gradient checkpointing. Each sample is augmented with horizontal flip (50%), random rotation ±30° (50%), and random resized crop at 70–100%. The loss is scale-invariant logarithmic error (SILog). Best checkpoint is selected by validation loss.

## Results

### Zero-shot vs. Fine-tuned

| | AbsRel ↓ | RMSE ↓ | SILog ↓ | δ1 ↑ |
|---|---:|---:|---:|---:|
| Zero-shot (raw) | 0.611 | 1.798 | 15.21 | 0.197 |
| Zero-shot (median-scaled) | 0.222 | 0.487 | 15.21 | 0.870 |
| **Fine-tuned** | **0.187** | **0.243** | **8.35** | **0.979** |
| Fine-tuned (median-scaled) | 0.173 | 0.233 | 8.35 | 0.985 |

The pretrained model already captures relative depth structure (median-scaled δ1 = 0.870); fine-tuning fixes the absolute scale and sharpens ordering.

### Indoor vs. Outdoor Initialization

| Init | AbsRel ↓ | SqRel ↓ | RMSE ↓ | SILog ↓ | δ1 ↑ | δ2 ↑ |
|---|---:|---:|---:|---:|---:|---:|
| **Indoor** | **0.187** | **0.966** | **0.243** | **8.35** | **0.979** | **0.998** |
| Outdoor | 0.217 | 1.043 | 0.314 | 9.08 | 0.947 | 0.998 |

Indoor wins on every aggregate metric. Close-range, structured-surface priors transfer better to underwater scenes than road/driving priors, even though the Indoor depth ceiling (20 m) is lower than Outdoor (80 m).

### Per-scene Breakdown (Indoor vs. Outdoor)

| Scene | n | AbsRel (In) | AbsRel (Out) | δ1 (In) | δ1 (Out) |
|---|---:|---:|---:|---:|---:|
| **Sea-thru D1** | 62 | **0.088** | 0.126 | **0.972** | 0.915 |
| **Sea-thru D2** | 32 | **0.132** | 0.144 | **1.000** | 0.999 |
| **Sea-thru D3** | 7 | **0.254** | 0.277 | **0.964** | 0.955 |
| **Sea-thru D4** | 15 | **0.271** | 0.301 | **0.999** | 0.997 |
| **Sea-thru D5** | 4 | **1.831** | 1.911 | **0.961** | 0.951 |
| **SQUID Katzaa** | 2 | **0.140** | 0.197 | **0.903** | 0.787 |
| **SQUID Mikhmoret** | 2 | 0.121 | **0.116** | 0.931 | **0.944** |
| **SQUID Nachsholim** | 1 | **0.118** | 0.126 | **0.962** | 0.947 |
| **SQUID Satil** | 1 | **0.101** | 0.141 | **0.873** | 0.774 |

Indoor wins on 8 of 9 scenes. Even on Satil (20–30 m range, beyond Indoor's 20 m ceiling), Indoor outperforms Outdoor.

### Augmentation Impact

| Config | AbsRel (Overall) | δ1 (Overall) | AbsRel (Satil) | δ1 (Satil) |
|---|---:|---:|---:|---:|
| Flip only (6 ep) | 0.215 | 0.943 | 0.186 | 0.557 |
| **+Rot+Crop (20 ep)** | **0.187** | **0.979** | **0.101** | **0.873** |

Geometric augmentation (rotation ±30°, random crop 70–100%) substantially improves cross-dataset generalization, particularly on out-of-distribution scenes like Satil.

### Comparison with Prior Methods (Atlantis)

| Method | Training Data | SQUID AbsRel ↓ | SQUID RMSE ↓ | SQUID δ1 ↑ | Sea-thru AbsRel ↓ | Sea-thru RMSE ↓ | Sea-thru δ1 ↑ |
|---|---|---:|---:|---:|---:|---:|---:|
| iDisc + Atlantis | Atlantis (synthetic) | 0.249 | 2.663 | 0.637 | 1.630† | 1.371† | 0.553† |
| NeWCRFs + Atlantis | Atlantis (synthetic) | 0.229 | 2.563 | 0.675 | 1.683† | 1.435† | 0.476† |
| IEBins + Atlantis | Atlantis (synthetic) | 0.263 | 2.896 | 0.615 | 1.687† | 1.597† | 0.425† |
| VA-DepthNet + Atlantis | Atlantis (synthetic) | 0.204 | 2.666 | 0.705 | 1.781† | 1.204† | 0.648† |
| **DA-V2 Indoor-S (Ours)** | Sea-thru + SQUID | **0.120‡** | **—** | **0.917‡** | **0.187‡** | **0.243‡** | **0.979‡** |

†Evaluated on D3+D5 subsets only. ‡Evaluated on combined test split (n=126).

**Note:** This comparison is not strictly apples-to-apples. Atlantis evaluates under a harder zero-shot transfer protocol with no real underwater training data, whereas our within-source proportional split means Sea-thru test frames share dive sites (though not identical frames) with training data. Nevertheless, the gap (δ1 from 0.705 → 0.979) suggests that even modest amounts of real underwater supervision, combined with a strong pretrained encoder, can close most of the domain gap that purely synthetic pipelines leave open.

## Discussion

The model performs well on D1–D2, remains usable on D3–D4, and drops sharply on D5 (deep shipwreck with severe backscatter). SQUID cross-dataset results are encouraging but based on very small sample counts (1–2 per site).

**Key takeaways:**
- Pretraining domain matters more than nominal depth range — Indoor priors transfer better than Outdoor for underwater.
- Geometric augmentation meaningfully reduces appearance overfitting on small datasets.
- D5 remains a clear failure case, pointing to a distribution gap for deep, high-backscatter scenes.

**Where the model fails:** D5 remains a clear outlier across all configurations, with AbsRel above 1.8. This site is a deep Red Sea shipwreck (20–30 m) with severe backscatter and strong color attenuation — conditions underrepresented in training. The error magnitude points to a genuine distribution gap rather than a simple scale or augmentation issue.

**Limitations.** Sea-thru test frames share dive sites with training data (within-source split), likely inflating D1–D4 numbers. SQUID provides a more honest cross-dataset signal but with very few test samples. Scene-level holdout and larger cross-dataset test sets are needed.

## Future Work

- **ORB-SLAM integration** — combining monocular depth estimates with visual SLAM for full underwater 3D reconstruction from monocular video.
- **Newer backbones** — evaluating stronger foundation-depth models to test whether better pretraining further improves underwater transfer.
- **Model distillation** — compressing the model into a smaller student suitable for deployment on embedded robotic platforms with limited compute.

## Visualization

![Underwater depth demo GIF](https://github.com/user-attachments/assets/797fd016-4ba6-4f7b-b998-11e0739024db)

## References

- Sea-Thru: A Method for Removing Water From Underwater Images
- Underwater Single Image Color Restoration Using Haze-Lines and a New Quantitative Dataset (SQUID)
- Atlantis: Enabling Underwater Depth Estimation with Stable Diffusion
- Depth Anything V2 (Yang et al., NeurIPS 2024)
- A Physical Model-Guided Framework for Underwater Image Enhancement and Depth Estimation
