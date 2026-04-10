# 3D Segmentation of Craniofacial Structures

3D segmentation of craniofacial bone structures from CT data using 3D Slicer, with manual, semi-automatic, and AI-based pipelines. Includes cephalometric analysis supporting a Class III malocclusion diagnosis.

**Course:** Visualization of 2D and 3D Medical Images — Universitat Politècnica de Catalunya (UPC)  
**Authors:** Carrieri Simone, Seguel Rocio  
**Academic year:** 2025–2026

---

## Overview

The dataset consists of a contrast-enhanced CT volume of the craniofacial region (Option 2). Three anatomical structures were segmented: mandible, maxilla, and teeth. Cephalometric measurements were then performed to assess a suspected Class III skeletal malocclusion. AI-based segmentation was tested and compared against manual results.

---

## Task 1 — Manual Segmentation (3D Slicer Segment Editor)

All structures were segmented using the same core pipeline:

**Thresholding → Manual Refinement → Island Effect**

| Structure | HU Threshold | Notes |
|-----------|-------------|-------|
| Mandible | 306.31 | Eraser + Scissors to remove skull/vertebrae; Island effect (≥1000 voxels) |
| Maxilla | — | Duplicated from superior skull via Logical Operators; iterative Scissors + Island effect |
| Teeth | 1866.51 | Higher threshold due to enamel/dentin radiodensity; same refinement pipeline |

Teeth were further subdivided into functional groups using Logical Operators and Scissors: incisors, canines, premolars, molars. Teeth were then subtracted from bone segments to expose internal alveolar spaces, followed by Smoothing (Closing method).

---

## Task 2 — Visualization and Cephalometric Measurements

Enhanced visualization in 3D Slicer using:
- **Colorized Volume** module — color mapping for anatomical differentiation
- **Lights** module — depth and contrast optimization

### Cephalometric measurements (sagittal plane, Markups module)

| Measurement | Value | Threshold (Class III) | Unit | Classification |
|-------------|-------|-----------------------|------|----------------|
| ANB angle | −7.1 | ≤ 0 | ° | Class III |
| Wits appraisal | −12.04 | ≤ −3 | mm | Class III |
| Beta angle | 54.4 | ≥ 32–37 | ° | Class III |

All three metrics confirm a pronounced Class III skeletal pattern. Segmentation facilitated the analysis — in particular, the occlusal plane fit for the Wits appraisal was significantly easier with teeth segmented.

---

## Task 3 — AI Segmentation

### MONAI Auto3DSeg — Whole Head Segmentation
- Segments upper skull and facial muscles; does not include maxilla or teeth
- Best approximation: superior skull + mandible retained after hiding irrelevant structures
- Residual spurious voxels require manual cleanup (Erase tool in 2D + Islands tool)

### Limitations of automatic approaches
- **Maxilla**: no available plugin produces accurate automatic segmentation due to continuity with the cranial base
- **Teeth**: AI output as Segment nodes lacks internal tooth structure; workaround is to use the Volume output from MONAI and apply the manual workflow (Section 1.4)

---

## Task 4 — Plugin Usage

### DentalSegmentator (nnU-Net, trained on 470 CT/CBCT scans)
Provides: upper skull, mandible, upper teeth, lower teeth, mandibular canal.  
Input: cropped ROI (Crop Volume module) to reduce computational load.  
Result: significantly more accurate than MONAI for dental structures; mandibular canal clearly identified — clinically relevant for surgical planning.

### Virtual Reality Plugin (Oculus Quest 2)
Experimental VR visualization of the 3D anatomical model directly within 3D Slicer. Promising for immersive surgical planning; FPS limitations noted at current stage.

---

## Repository Structure

```
.
├── Manual.mrb               # Manual segmentation scene (segments + markups + colorized volume)
├── MonAIScene.mrb           # MONAI Auto3DSeg scene
├── DentalSegmentator.mrb    # DentalSegmentator scene (cropped ROI input)
└── README.md
```

All `.mrb` files are 3D Slicer Medical Reality Bundle format, based on the same base CT volume (`cap`).

---

## Requirements

- 3D Slicer 5.6.2 or later
- Extensions: MONAI Auto3DSeg, DentalSegmentator, SlicerVirtualReality (optional)
