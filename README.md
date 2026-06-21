# CC-ICP: Cylinder-Constrained ICP for Forest Point Cloud Fine Registration

This repository contains the official C++ implementation of **CC-ICP (Cylinder-Constrained Iterative Closest Point)**, a dedicated fine registration algorithm for forest laser point clouds, as described in the paper. The proposed method is not limited to specific scanning equipment and is universally applicable to various types of forest scene point cloud data.

## Overview

### Background

Laser scanning technology enables accurate, non-destructive 3D forest structure measurement for forest resource inventory and monitoring. Precise multi-view and multi-scan point cloud registration is a fundamental prerequisite for unified forest scene reconstruction and reliable forest parameter interpretation. The complete point cloud registration pipeline consists of two core stages:

1. **Coarse registration**: Roughly calibrates the relative rotation and translation offsets between different point cloud scans to eliminate large pose deviations;

2. **Fine registration**: Further optimizes and refines alignment accuracy to achieve high-precision fusion of overlapping point cloud regions.

Traditional fine registration methods (ICP, GICP, NDT, etc.) are originally designed for structured artificial scenes. When applied to complex forest environments, they suffer from poor robustness and low precision due to repetitive tree trunk geometric features, severe occlusion, sensor noise, and limited overlapping areas between point clouds.

### Core Idea of CC-ICP

Targeting the inherent geometric characteristics of forest scenes, CC-ICP constructs dual constraint mechanisms to optimize the traditional ICP objective function, effectively solving the poor adaptation problem of classic algorithms in forest point cloud registration:

1. **Ground plane constraint (vertical stabilization)**: Extract high-precision ground points via grid minimum filtering, construct point-to-plane residual terms, and provide stable vertical direction constraint for registration;

2. **Tree cylinder constraint (horizontal stabilization)**: Perform adaptive height slicing on point clouds based on ground interpolation, extract tree trunk primitives through DBSCAN clustering combined with normal-assisted RANSAC cylinder fitting, and build point-to-point residual terms relying on trunk cylinder center features to constrain horizontal offset errors.

The algorithm adaptively fuses two types of residual information through a weighted least squares framework, iteratively solves the rigid transformation matrix via the Gauss-Newton method, and dynamically updates ground features and trunk cylinder primitives in each iteration to realize robust and high-precision fine alignment of forest point clouds.

### Key Pipeline

1. Ground point extraction from raw forest point clouds using grid minimum filter;

2. Adaptive point cloud slicing based on IDW ground height interpolation;

3. Tree trunk detection: DBSCAN spatial clustering + RANSAC cylinder fitting;

4. Multi-feature matching: fusion of ground plane correspondences and trunk cylinder correspondences;

5. Weighted joint least-squares optimization to solve rigid transformation parameters;

6. Iterative transformation update until convergence to obtain the optimal registration result.

### Experiment Performance

Extensive experiments on 24 plots of mountainous forest scenes verify that the proposed CC-ICP method outperforms five mainstream fine registration algorithms in terms of registration robustness, alignment accuracy and computational efficiency:

- Standard ICP

- Point-to-Plane ICP

- Generalized ICP (GICP)

- Symmetric ICP

- Normal Distributions Transform (NDT)

## Dependencies

This implementation depends on classic point cloud processing and linear algebra libraries, with good compatibility and portability:

- PCL >= 1.8 (Tested on PCL 1.8 / 1.12)

- Eigen >= 3.3

- CMake >= 3.12

- C++14 and above compiler (GCC / Clang / MSVC)

Optional dependencies (for visualization and acceleration):

- VTK (bundled with PCL, for point cloud visualization)

- OpenMP (for point cloud traversal acceleration)

## Build Guide

### 1. Clone Repository

```Plain Text
git clone https://github.com/your-name/CC-ICP-Forest-Reg.git
cd CC-ICP-Forest-Reg
```

### 2. Compile with CMake

#### Linux / macOS

```Plain Text
mkdir Release && cd Release
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j$(nproc)
```

#### Windows (Visual Studio x64 Native Command Prompt)

```Plain Text
mkdir Release && cd Release
cmake -G "NMake Makefiles" -DCMAKE_BUILD_TYPE=Release ..
nmake
```

### 3. IDE Build Option

Open the root `CMakeLists.txt` with CLion, VS Code + CMake extension or Visual Studio, configure x64 Release mode and build the project directly.

## Usage

### Input Requirement

- Input: Two PCD format forest point cloud files

- `source.pcd`: Source point cloud to be transformed and aligned

- `target.pcd`: Reference target point cloud

- Output: Standard 4×4 rigid transformation matrix (printed to the console, supports custom file export)

### Quick Run

The default demo program adopts hard-coded file paths. Please modify the point cloud file paths in the `main()` function according to your data before compilation:

```Plain Text
readPointCloud("17-2.pcd", targetcloud);
readPointCloud("17-1.pcd", sourcecloud);
```

After compilation, execute the executable file directly:

```Plain Text
# Linux/macOS
./CCICP_Registration

# Windows
CCICP_Registration.exe
```

### Program Output

The program will automatically output detailed runtime logs and final results, including:

1. Time consumption of ground point extraction and trunk cylinder detection;

2. Statistical number of valid trunk cylinder features extracted from source and target point clouds;

3. Number of successful trunk feature matches in each iteration;

4. Real-time updated rotation matrix, translation vector and iteration convergence state;

5. Final converged 3×3 rotation matrix, 3D translation vector and complete 4×4 transformation matrix.

### Adjustable Core Parameters

All hyperparameters are centrally defined in the `main()` function, which can be flexibly tuned according to different forest density, terrain and point cloud quality:

|Parameter|Description|
|---|---|
|slice_height|Vertical slice thickness for trunk segmentation (m)|
|max_iterations|Maximum iterations of ICP optimization|
|tolerance|Convergence threshold for transformation update|
|corr_dist|Maximum spatial distance threshold for trunk feature matching|
|threshold_radius|Allowable radius difference for matched trunk cylinders|
|threshold_axisdirection|Maximum axis angle difference (°) for trunk matching|
|num_cylinder|Minimum valid point count for a single trunk cylinder|
|resolution|Grid size for ground point extraction|
|num_groundPnt|Number of nearest ground points for IDW height interpolation|
## File Structure

```Plain Text
CC-ICP-Forest-Reg/
├── CMakeLists.txt        # Project build configuration file
├── CCICP0621.cpp         # Complete CC-ICP algorithm source code
├── README.md             # Project usage documentation
└── test_data/            # Default storage directory for PCD test point clouds
```

## Citation

If you use this code, algorithm ideas or experimental results in your academic research, please cite our work:

```Plain Text
@article{ccicp2026forest,
  title={Cylinder-Constrained ICP for Fine Registration of Forest Laser Point Clouds},
  author={XXX, XXX, XXX},
  journal={XXX},
  year={2026}
}
```

## Contact

For code bugs, parameter tuning guidance, academic communication and cooperation, please contact:

Your Name <your-email@example.com>

## Disclaimer

1. The demo version adopts hard-coded file paths. For batch processing and engineering deployment, it is recommended to add command-line parameter parsing functions and optimize file input/output logic;

2. The trunk detection module based on DBSCAN + RANSAC has slight randomness, leading to tiny deviations in repeated experimental results, which do not affect the overall registration accuracy;

3. This algorithm is a high-precision fine registration method. For large-scale forest multi-point-cloud registration tasks, it is recommended to combine it with coarse registration algorithms to achieve end-to-end efficient and accurate alignment.

4. This method is applicable to all types of forest laser point cloud data, without being limited to specific scanning devices or data acquisition modes.
> （注：文档部分内容可能由 AI 生成）