# AAE5303 OpenSplat UAV Scene Reconstruction Demo

## Project Overview

This project focuses on 3D scene reconstruction from UAV image data using **OpenSplat**, a 3D Gaussian Splatting framework. The project uses UAV scene data and camera pose information to reconstruct a 3D representation of an outdoor scene.

The overall workflow includes image data preparation, camera pose / sparse reconstruction generation, OpenSplat training, PLY model export, and reconstruction quality analysis.

In this project, my main responsibility is **Part 2: OpenSplat-based 3D Gaussian Splatting Reconstruction**.

---

## Part 2: OpenSplat 3D Gaussian Splatting Reconstruction

### Objective

The objective of this part is to use OpenSplat to generate a 3D Gaussian Splatting model from UAV image data. The input data should include RGB images and COLMAP-style sparse reconstruction files. The final output is a set of `.ply` files representing the reconstructed 3D scene at different training iterations.

The reconstruction results are compared at four training stages:

- 200 steps
- 2000 steps
- 6000 steps
- 10000 steps

The goal is to observe how the reconstruction quality improves as the number of training iterations increases.

---

## Repository Structure

```text
AAE5303_opensplat_demo-/
├── data/
│   └── HKisland_colmap/
│       ├── images/
│       └── sparse/
│           └── 0/
│               ├── cameras.bin
│               ├── images.bin
│               └── points3D.bin
├── output/
│   ├── hkisland_200.ply
│   ├── hkisland_2000.ply
│   ├── hkisland_6000.ply
│   └── hkisland_10000.ply
├── scripts/
│   ├── run_opensplat_200.sh
│   ├── run_opensplat_2000.sh
│   ├── run_opensplat_6000.sh
│   ├── run_opensplat_10000.sh
│   ├── inspect_ply.py
│   └── generate_result_summary.py
├── docs/
│   ├── opensplat_workflow.md
│   ├── reconstruction_analysis.md
│   └── personal_reflection.md
└── README.md
```

---

## Input Data Format

OpenSplat requires a COLMAP-style input structure. The dataset should contain an image folder and sparse reconstruction files.

Expected input format:

```text
data/HKisland_colmap/
├── images/
│   ├── frame_000001.jpg
│   ├── frame_000002.jpg
│   └── ...
└── sparse/
    └── 0/
        ├── cameras.bin
        ├── images.bin
        └── points3D.bin
```

The key input components are:

| Component | Description |
|---|---|
| `images/` | RGB images captured from the UAV scene |
| `cameras.bin` | Camera intrinsic parameters |
| `images.bin` | Camera poses and image information |
| `points3D.bin` | Sparse 3D points used for initialization |

---

## Environment Setup

### 1. Install Basic Dependencies

```bash
sudo apt update
sudo apt install -y git cmake build-essential libopencv-dev wget unzip
```

### 2. Clone OpenSplat

```bash
git clone https://github.com/pierotofy/OpenSplat OpenSplat
cd OpenSplat
```

### 3. Prepare LibTorch

Download the correct LibTorch version according to your environment.

For CPU-only environments, use the CPU version of LibTorch.

For GPU environments, use the CUDA-compatible version of LibTorch.

### 4. Build OpenSplat

```bash
mkdir build
cd build
cmake -DCMAKE_PREFIX_PATH=/path/to/libtorch ..
make -j$(nproc)
```

After compilation, the executable should be available as:

```bash
./opensplat
```

---

## Training Commands

The following commands were used to train the scene at different iteration settings.

### 200 Steps

```bash
./opensplat /root/OpenSplat/data/HKisland_colmap \
  -n 200 \
  -o /root/OpenSplat/output/hkisland_200.ply \
  --sh-degree 3 \
  --ssim-weight 0.2 \
  --num-downscales 2
```

### 2000 Steps

```bash
./opensplat /root/OpenSplat/data/HKisland_colmap \
  -n 2000 \
  -o /root/OpenSplat/output/hkisland_2000.ply \
  --sh-degree 3 \
  --ssim-weight 0.2 \
  --num-downscales 2
```

### 6000 Steps

```bash
./opensplat /root/OpenSplat/data/HKisland_colmap \
  -n 6000 \
  -o /root/OpenSplat/output/hkisland_6000.ply \
  --sh-degree 3 \
  --ssim-weight 0.2 \
  --num-downscales 2
```

### 10000 Steps

```bash
./opensplat /root/OpenSplat/data/HKisland_colmap \
  -n 10000 \
  -o /root/OpenSplat/output/hkisland_10000.ply \
  --sh-degree 3 \
  --ssim-weight 0.2 \
  --num-downscales 2
```

---

## Output Files

The OpenSplat training process generated four PLY files:

```text
output/
├── hkisland_200.ply
├── hkisland_2000.ply
├── hkisland_6000.ply
└── hkisland_10000.ply
```

Each `.ply` file stores the optimized 3D Gaussian representation of the scene at a specific training iteration.

---

## Result Comparison

| Training Iterations | Output File | Purpose | Reconstruction Quality | Main Observations |
|---|---|---|---|---|
| 200 | `hkisland_200.ply` | Fast baseline | Low | Rough scene structure, blurry regions, incomplete geometry |
| 2000 | `hkisland_2000.ply` | Early reconstruction | Medium | More stable structure, better color consistency, but floating artifacts remain |
| 6000 | `hkisland_6000.ply` | Intermediate result | Good | Clearer major structures, fewer extreme artifacts, but holes and boundary noise still exist |
| 10000 | `hkisland_10000.ply` | Final reconstruction | Best among tested results | Most complete and visually stable result, selected for final demonstration |

---

## Reconstruction Analysis

The reconstruction quality improved as the number of training iterations increased.

At **200 steps**, the model only produced a rough reconstruction. This stage was mainly used to verify that the OpenSplat pipeline could successfully load the input data and export a valid PLY file. The visual quality was limited, with obvious noise, blurry regions, and incomplete geometry.

At **2000 steps**, the scene structure became more stable. The color distribution was more consistent, and the reconstructed scene became easier to recognize. However, floating artifacts and missing surfaces were still visible.

At **6000 steps**, the reconstruction quality improved further. Major scene structures became clearer, and the overall appearance became more coherent. Some noisy regions were reduced, but holes, blurred textures, and boundary noise still existed in complex or weakly observed areas.

At **10000 steps**, the model achieved the best visual quality among the four tested settings. The geometry was more complete, and the scene appeared more stable. Therefore, the 10000-step model was selected as the final reconstruction result for presentation.

---

## Observed Artifacts

Several types of artifacts were observed during reconstruction.

### 1. Floating Artifacts

Some isolated Gaussian points appeared in empty space. These floating artifacts may be caused by inaccurate camera poses, feature mismatches, or insufficient multi-view coverage.

### 2. Blurred Textures

Some regions appeared blurry, especially in areas affected by UAV motion, lighting changes, or limited image overlap. The model may not have enough consistent visual information to reconstruct sharp details.

### 3. Holes and Missing Surfaces

Certain surfaces were incomplete or missing. This usually happened in areas that were not captured from enough viewpoints or were affected by occlusion.

### 4. Boundary Noise

Noise appeared around object boundaries, building edges, vegetation, and terrain transitions. These regions are geometrically complex and sensitive to pose errors.

---

## Limitations

Although the 10000-step model produced the best result, the reconstruction still had some limitations:

- The reconstruction quality depends heavily on the accuracy of camera poses.
- Sparse or uneven image coverage can cause missing surfaces.
- UAV motion may introduce blur and pose uncertainty.
- Outdoor lighting changes can affect color consistency.
- CPU-based training is slower than GPU-based training.
- More iterations do not completely remove all artifacts.

---

## Future Improvements

Future work could improve the reconstruction quality in the following ways:

1. Use a CUDA-enabled GPU for faster and longer training.
2. Improve camera pose estimation before OpenSplat training.
3. Use more images with better overlap.
4. Remove blurry or low-quality frames before reconstruction.
5. Test higher training iterations beyond 10000 steps.
6. Evaluate reconstruction quality with quantitative metrics such as PSNR, SSIM, and LPIPS.
7. Compare OpenSplat results with other 3D reconstruction or Gaussian Splatting methods.

---

## My Contribution

My contribution in this project focused on the OpenSplat reconstruction stage. Specifically, I completed the following tasks:

- Prepared and checked the COLMAP-style input structure.
- Set up the OpenSplat reconstruction workflow.
- Ran OpenSplat training at 200, 2000, 6000, and 10000 steps.
- Exported reconstructed `.ply` models.
- Compared reconstruction quality across different training stages.
- Identified visual artifacts such as floating points, holes, blur, and boundary noise.
- Documented the reconstruction process, result analysis, limitations, and reflection.

---

## Conclusion

This project demonstrates a complete OpenSplat-based 3D Gaussian Splatting reconstruction workflow for UAV scene data. By comparing the results at 200, 2000, 6000, and 10000 training steps, the experiment shows that increasing the number of iterations improves reconstruction completeness and visual stability.

The 10000-step model produced the best result among the tested settings and was selected as the final reconstruction output. However, the remaining artifacts show that the final quality is still strongly affected by input image quality, camera pose accuracy, scene coverage, and computational resources.
