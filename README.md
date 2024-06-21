# Experiments with Predicting Landmarks in 3D MRI

As a little side project I wanted to predict four landmarks used to identify the EEG 10-10 system points.
These four points include

- Nasion (Nz)
- Inion (Iz)
- Left Pre-Auricular point (LPA)
- Right Pre-Auricular point (RPA)

These points are visualized below: Nz (red), Iz (blue), LPA (green), and RPA (yellow)

|                   front                   |                  left                   |                  back                   |
| :---------------------------------------: | :-------------------------------------: | :-------------------------------------: |
| ![front](visualization/figures/front.png) | ![left](visualization/figures/left.png) | ![back](visualization/figures/back.png) |

## Create Training Data

To create training data for this experiment I annotated 3 _template_ datasets. Using our automatic head segmentation tool (head40), I segmented the _template heads_ and 20 subjects from the [Calgary-Campinas](https://sites.google.com/view/calgary-campinas-dataset/home) (CC) dataset. Grouping tissues I obtain simplified head labels:

- brain (including CSF)
- bone
- air
- spinal cord (including CSF)
- other

This looks something like this
![base mask](visualization/figures/base_mask.png)

Using [`SimpleITK`](https://pypi.org/project/SimpleITK/) I registered the _template_ head labels to the CC dataset (affine and B-spline). By design the registration finds transforms that map from the moving (CC) to fixed (_template_) space. To transform the _template_ landmarks to the CC space, we need to use the inverse transform.

Since I am using a box detector to locate 3D points, I simply create cubic boxes with fixed width (5mm) in `cccwhd` (X,Y,Z centers and width, height and depth).

## Adapt MONAI Tutorial

For this purpose I adapted the [detection tutorial](https://github.com/Project-MONAI/tutorials/tree/main/detection) from [MONAI](https://github.com/Project-MONAI). That tutorial predicts lung nodules from CT images. The things I needed to modify included

- image normalization: instead of scaling from a fixed range to \[0, 1\] I use a [`NormalizeIntensityd`](https://docs.monai.io/en/stable/transforms.html#normalizeintensityd) transform.
- since I want to learn left-right landmarks, I removed the random flip transforms
- since the input data is MRI, I use different intensity augmentation, e.g.,
  - RandAdjustContrastd
  - RandHistogramShiftd
  - RandBiasFieldd
  - RandGibbsNoised
  - RandKSpaceSpikeNoised

Running the training for 300 steps takes 1-2 hours on my GPU (NVIDIA GeForce RTX 3090). I am not yet 100 percent familiar with the COCO metrics, but Recall IoU on the test data seems pretty good.

| Nz AR IoU | Iz AR IoU | LPA AR IoU | RPA AR IoU |
| :-------: | :-------: | :--------: | :--------: |
|   0.85    |   0.87    |    0.89    |    0.91    |
