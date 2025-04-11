
# 🧬 Mask R-CNN for Cell Segmentation  
**Compression Microscope, Hanyang University**

This repository contains a full Mask R-CNN pipeline for segmenting DIC cell images acquired with a Nikon Eclipse Ti2 microscope.

- **Objective lens**: CFI Plan Apochromat VC 60X WI  
- **Cell line**: Raji  
- **Training GPU**: one A4000
- **Also works for**: 40X images or other cell types

---

## 🔬 Example Input & Ground Truth

| DIC Image (MIP) | Segmentation Mask (Labkit, ImageJ) |
|------------------|-----------------------------------|
| ![DIC](https://github.com/user-attachments/assets/4a7b175b-bf39-4435-a85e-fcdd379374aa) | ![Mask](https://github.com/user-attachments/assets/d6ebb989-343d-4fb7-ab8d-012d2cfd152e) |

---

## 🧩 Code 1: 32-bit TIFF → 8-bit PNG Conversion

Converts raw `.tif` images (DIC and masks) from 32-bit float to normalized 8-bit `.png`.

### Why?  
Most microscopy images are in 32-bit or 16-bit format. PyTorch models expect 8-bit inputs.

### 🔧 Operations:
- Reads `.tif` using `PIL`
- Applies min-max normalization → scales to 0–255 → converts to `uint8`
- Saves to:
  ```
  png_convert/
  ├── DIC/
  └── mask/
  ```
- Skips non-32-bit images and logs conversion info

---

## 🧱 Code 2: Image Splitting & Augmentation

Splits full-size images into **512×512 patches** with overlap and performs augmentation.

### 🧪 Features:
- **Overlap**: 256 pixels (stride = 256)
- **Mask filtering**: skips masks with area < 200
- **Augmentation**:
  - Horizontal flip
  - Vertical flip
  - 90°, 180°, 270° rotation

### 📁 Output Structure:
```
Training_data_512/
├── train/
│   ├── DIC/
│   └── mask/
└── val/
    ├── DIC/
    └── mask/
```

| Augmented Example |
|--------------------|
| ![rot180](https://github.com/user-attachments/assets/bbc6b1a7-965f-4a6e-92c2-a884bcaae211) ![rot270](https://github.com/user-attachments/assets/a9648504-7baa-4956-88b9-a0cb9f44e7f6) |

---

## 🧠 Code 3: Model Training

Trains `maskrcnn_resnet50_fpn` for binary cell segmentation.

### 🔍 Key Elements:
- **Custom Dataset**: 512×512 patch loader with bbox + mask extraction
- **Modified heads**: for binary classification
- **Loss logging**: every 300 steps
- **Visual validation**: every 500 steps
- **Checkpoint saving**: every 5 epochs

### 📁 Logs and Outputs:
```
training_logs_512/
├── mask_rcnn_final.pth
├── epoch_10_step_500.png
└── ...
```

| Training Visualization |
|------------------------|
| ![epoch18](https://github.com/user-attachments/assets/4ce46c31-cd20-492b-9412-57b12d57830e) ![epoch30](https://github.com/user-attachments/assets/e86a2ee4-ce8f-4710-bb6f-4315cbf8bcd2) |

---

## 🎯 Code 4: Inference on Full MIP Image

Applies trained model to full-size (2048×2048) DIC MIP images.

### 🛠️ Procedure:
- Splits image into overlapping 512×512 patches (384px overlap)
- Runs each patch through model on CUDA
- Merges masks using **Hanning weight map**
- Applies:
  - **Opening** to remove noise
  - **Dilation** to refine edges
- Saves final binary masks

| Prediction Example |
|--------------------|
| ![MIP](https://github.com/user-attachments/assets/d343d08b-95da-4bc2-bbb3-151f099effd9) ![Mask](https://github.com/user-attachments/assets/7a7eb87b-0575-4c1a-a82e-dc1ed9a43b0b) |

---

## 📌 Citation
If you use this code for research or publications, please cite:

>
