# Nemo-MDE
nemo.. bc.. underwater...

This repository presents a real-data underwater metric depth experiment using **Monocular Depth Estimation** on **SeaThru**.

## Overview

Underwater metric depth estimation remains difficult because visibility degradation changes appearance in ways that are very different from standard in-air imagery. This work evaluates metric-depth model in underwater scenes with real underwater RGB-depth supervision.

## Dataset

Experiments reported here use **SeaThru D1–D5**.

## Training

Split sizes:

- **Train:** 965
- **Validation:** 120
- **Test:** 120

## Results

### Overall test metrics

- **AbsRel:** `0.182179`
- **SqRel:** `0.850616`
- **RMSE:** `0.215251`
- **RMSE(log):** `0.102055`
- **SILog:** `8.412296`
- **δ1:** `0.965947`
- **δ2:** `0.997745`
- **δ3:** `0.999514`

### Per-subset breakdown

| Subset | n | AbsRel | SqRel | RMSE | RMSE(log) | SILog | δ1 | δ2 | δ3 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| SeaThru D1 | 62 | 0.100177 | 0.041793 | 0.118524 | 0.109714 | 8.590293 | 0.953812 | 0.997320 | 0.999614 |
| SeaThru D2 | 32 | 0.112838 | 0.570298 | 0.241324 | 0.061383 | 5.118208 | 0.999848 | 0.999879 | 0.999893 |
| SeaThru D3 | 7 | 0.222019 | 1.340428 | 0.565186 | 0.132210 | 12.175379 | 0.943415 | 0.993116 | 0.997592 |
| SeaThru D4 | 15 | 0.201480 | 0.461528 | 0.188116 | 0.108031 | 9.110483 | 0.957734 | 0.998459 | 0.999608 |
| SeaThru D5 | 4 | 1.865819 | 16.231833 | 0.995298 | 0.233543 | 22.802455 | 0.953058 | 0.992672 | 0.997930 |

## Discussion

The model performs well on **D1** and **D2**, remains usable on **D3** and **D4**, and drops sharply on **D5**, which is clearly the hardest subset in this setting.

Overall, these results show that a pretrained metric-depth model can transfer effectively to underwater imagery with real underwater supervision, while also showing that robustness across different underwater conditions is still the main challenge.

## Comparison with Recent SeaThru-Based Work

A recent SeaThru-based effort, **A Physical Model-Guided Framework for Underwater Image Enhancement and Depth Estimation (2024)**, reports the following SeaThru depth results:

- **RMSE 1.5969 / AbsRel 0.4039** improved to **RMSE 1.3811 / AbsRel 0.3969**
- **RMSE 4.3064 / AbsRel 0.5757** improved to **RMSE 3.8157 / AbsRel 0.5361**

That work combines physically guided underwater enhancement with depth estimation. In contrast, this repository focuses on direct adaptation of MDE for underwater metric depth prediction.

The current result suggests that a strong pretrained monocular metric-depth model is a competitive baseline on real underwater RGB-D data, especially on the easier SeaThru subsets.

## Visualization

![Underwater depth demo GIF](https://github.com/user-attachments/assets/a2fdddbe-37a2-4fce-bb08-132df5d33088)

## References

- Sea-Thru: A Method for Removing Water From Underwater Images
- A Physical Model-Guided Framework for Underwater Image Enhancement and Depth Estimation
