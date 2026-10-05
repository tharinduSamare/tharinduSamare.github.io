---
title: Sparse Light Field Refocusing
publishDate: 2021-04-01 00:00:00
img: /assets/images/light_field_refocusing/light_field_hero_banner.png
img_alt: HeiChips26_SOC_architecture
description: |
  Arbitrary Volumetric Refocusing of Dense and Sparse Light Fields
start_date: "2021/04"
end_date: "2021/09"
tags:
  - PyTorch
  - U-Net
  - 4D Volumetric Light Fields
  - Sparse Light Fields
  - Image Refocusing
---

## Arbitrary Volumetric Refocusing of Dense and Sparse Light Fields

<!-- ![Arbitrary Volumetric Refocusing of Dense and Sparse Light Fields](/assets/images/light_field_refocusing/light_field_hero_banner.png) -->

<!-- > **Research paper** · B.Sc. research project · Tharindu Samarakoon, Kalana Abeywardena, Chamira U. S. Edussooriya · arXiv preprint (2025) -->

📄 **[Read the paper (arXiv)](https://arxiv.org/abs/2502.19238)**  ·  [PDF](https://arxiv.org/pdf/2502.19238)

**In one line:** a pipeline that lets you pick several regions of a photo *after* it has been captured and choose, for each one, whether it should be sharp or blurred. It works on both dense light fields and sparse light fields that need about 79% less data, and a small neural network cleans up the artifacts that sparse data introduces.

---

### At a glance

| | |
|---|---|
| **Topic** | Post-capture refocusing of 4-D light fields (computational photography) |
| **Problem** | Earlier methods refocus one planar or volumetric region at a time, and cannot keep a region at the same depth range blurred while another is sharp. They are also built for dense light fields |
| **Contribution** | An end-to-end pipeline that refocuses **multiple arbitrary planar or volumetric regions at once**, for **dense and sparse** light fields |
| **Key ideas** | Pixel-dependent shifts in shift-and-sum refocusing · interactive region selection · MS-SSIM depth search for sparse data · U-Net to remove ghosting artifacts |
| **Headline results** | **SSIM > 0.9** (Structural Similarity Index) on 3 of 4 test light fields from sparse views only · up to **79% less data** · **0.71 s** for a sparse light field vs 3.64 s for its dense version |
| **Tech** | PyTorch · U-Net · epipolar image analysis · MS-SSIM · BRISQUE · Google Colab (Tesla T4 GPU) |

---

### Background: what is a light field?

A normal photo records only *how bright* each pixel is. A **4-D light field (LF)** also records *where the light came from*, using many slightly shifted views of the same scene (called sub-aperture images, or SAIs). That extra information makes it possible to estimate depth, suppress occluding objects, and **refocus after capture**.

The classic refocusing method, **shift-and-sum**, shifts every view by an amount that depends on depth and then adds them up. Objects at that depth come out sharp, and everything else blurs.

#### The gap this work fills

| | Earlier methods | This work |
|---|---|---|
| Regions refocused at once | One planar region, one volumetric (wide-depth) region, or several wide-depth regions | **Any number of arbitrary planar or volumetric regions** |
| Same depth range, different focus | Not possible: everything in the chosen depth range is in focus | **Possible**: one region sharp, another at the same depth range blurred |
| Light field type | Mostly dense (camera arrays or lenslet cameras) | **Dense and sparse** |
| Sparse light fields | Planar refocusing of one narrow depth range | **Multiple regions**, without building a dense light field first |

![Refocusing comparison from the paper](/assets/images/light_field_refocusing/paper_fig1_refocusing_comparison.png)

*Figure 1 of the paper, "Bush" light field: (a) single planar refocus, (b) two volumetric regions, (c) proposed arbitrary volumetric refocusing with a dense light field, (d) the same with a sparse light field. Narrow-depth and wide-depth regions are outlined in green and red.*

Light fields come from different cameras. Camera arrays and lenslet cameras give **dense** light fields, while compact cameras such as the EPIModule give **sparse** ones.

![Types of light field cameras](/assets/images/light_field_refocusing/paper_fig2_lf_cameras.png)

*Figure 2 of the paper: (a) light field video camera array, (b) Lytro Illum dense light field camera, (c) EPIModule sparse light field camera.*

---

### Our approach

#### Pixel-dependent refocusing

Standard shift-and-sum uses one depth value for the whole image. We use a **different depth value for every pixel**, stored in an "α mask". This is what lets each region have its own focus. The user picks regions of interest (ROIs) on the middle view and says whether each has a narrow or wide depth range. Wide regions are split into small patches (20 × 20 pixels) so the focus can change smoothly across them. A Gaussian filter (15 × 15, σ = 5) smooths the α mask so that transitions between sharp and blurred regions look natural.

![Dense and sparse refocusing pipelines](/assets/images/light_field_refocusing/refocusing_pipelines.png)

The heat map below shows part of the α mask for the "Bush" example. Notice the smooth transitions between focused and blurred regions, and the α value changing continuously inside the wide-depth region (upper right).

![Sections of the α mask shown as a heat map](/assets/images/light_field_refocusing/paper_fig5_alpha_mask_heatmap.png)

*Figure 5 of the paper.*

#### Dense light fields

1. The user selects the ROIs on the middle view.
2. A **depth map** is estimated with epipolar image analysis (gradients of epipolar lines plus a confidence measure, then smoothing).
3. The α value for each ROI (or patch) is taken as the most common value inside it, and unselected areas get a smaller α so they stay out of focus.
4. Each region with a unique α is refocused with shift-and-sum, and the results are joined into one image.

#### Sparse light fields

A **sparse LF** keeps only some of the views. We use a **cross-shaped** layout, similar to the EPIModule sparse light field camera: 17 views instead of 81 for a 9 × 9 dense light field.

![Dense light field vs a cross-shaped sparse light field](/assets/images/light_field_refocusing/sparse_cross_lf.png)

1. **No depth map.** For each ROI we try different α values, refocus it, and compare it with the same area of the middle view using **MS-SSIM**. The similarity curve has a single peak, so we first find the range that contains the best α and then search that range in steps of 0.1.
2. Build the α mask and refocus each region, as for dense light fields.
3. With so few views, the out-of-focus areas show **ghosting artifacts**. A **U-Net image restoration network** removes them.

#### Training the ghosting-removal network

| | |
|---|---|
| **Input** | Sparse refocused images from our sparse pipeline |
| **Ground truth** | Matching images from our dense pipeline |
| **Data handling** | Patch-wise training on non-overlapping 100 × 100 patches (few light fields are available) |
| **Loss** | MSE + β·(1 − MS-SSIM) + (1 − β)·L1 + γ / PSNR, with β = 0.65 and γ = 500 |
| **Training** | 30 epochs, batch size 256, Xavier initialisation, RMSProp, learning rate 0.001 |
| **Why a U-Net** | A vision transformer would need far more data than the few available light fields |

---

### Results

We tested on light fields from three public datasets.

| Dataset | Views | Resolution |
|---|---|---|
| EPFL | 15 × 15 | 434 × 625 |
| HCI | 9 × 9 | 512 × 512 |
| Stanford | 17 × 17 | 1024 × 1024 |

#### Quality and speed of sparse refocusing, compared with dense

| Light field | SSIM | PSNR (dB) | Sparse time as % of dense time |
|---|---:|---:|---:|
| Lego Knights | 0.774 | 24.44 | 58.7 % |
| Mirabelle Prune Tree | 0.907 | 24.54 | 78.2 % |
| Books | 0.934 | 27.93 | 89.7 % |
| Sideboard | 0.923 | 27.81 | 19.5 % |

![Refocusing time and SSIM across the four test light fields](/assets/images/light_field_refocusing/results_time_ssim.png)

Three of the four light fields reach an SSIM above 0.9 with only the cross-shaped views. Lego Knights scores lower because it comes from a camera array, whose views are further apart than those of a lenslet camera.

#### Total processing time

| Light field | ROIs | Dense (s) | Sparse (s) | BRISQUE, dense | BRISQUE, sparse |
|---|---:|---:|---:|---:|---:|
| Lego Knights | 4 | 19.94 | 11.71 | 44.63 | 69.11 |
| Mirabelle Prune Tree | 4 | 6.15 | 4.81 | 47.58 | 46.59 |
| Books | 3 | 8.54 | 7.66 | 60.30 | 47.31 |
| Sideboard | 3 | 3.64 | **0.71** | 14.96 | 42.83 |

*BRISQUE is a no-reference image quality score, where lower is better. Times are on a Google Colab Tesla T4 GPU.*

- Removing the ghosting artifacts takes only 0.08 to 0.35 s of the total.
- A single refocus takes under 0.2 s on the GPU. For example, 81 refocuses on Lego Knights take about 11 s in total.
- For dense light fields, the depth map is a one-time cost that can be computed at setup time, which cuts the per-use time further.

![Dense, sparse without post-processing, and sparse with the U-Net](/assets/images/light_field_refocusing/paper_fig6_dense_vs_sparse_quality.png)

*Figure 6 of the paper: the U-Net removes most of the ghosting seen in the plain sparse result.*

![Qualitative results on four light fields](/assets/images/light_field_refocusing/paper_fig7_qualitative_results.png)

*Figure 7 of the paper: multi arbitrary-volume refocusing on Lego Knights, Mirabelle Prune Tree, Books and Sideboard.*

---

### Limitations and future work

- The sparse result still shows slight aliasing artifacts on some scenes (for example Mirabelle Prune Tree) and a small colour and brightness shift. More training data and better augmentation should reduce both.
- Lego Knights (wide-baseline camera array) gets a lower SSIM than the lenslet-camera light fields.
- **Future work:** hardware architectures for real-time interactive light field refocusing.

---


### Authors

- **[Tharindu Samarakoon](https://www.linkedin.com/in/tharindusamare/)**, [Kalana Abeywardena](https://www.linkedin.com/in/kalana-abeywardena/), [Chamira U. S. Edussooriya](https://www.linkedin.com/in/chamira-edussooriya-87766a24b/)
- Paper: *Arbitrary Volumetric Refocusing of Dense and Sparse Light Fields*, [arXiv:2502.19238](https://arxiv.org/abs/2502.19238)