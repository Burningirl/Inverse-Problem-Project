# Practical Work: Modeling of Light Propagation and Tomographic Reconstruction

Practical work for the **MScT CSE** program (2025-2026), submitted to Martin Rodriguez-Vega.
Author: **Iris ZAME NGUINA**

This project builds, step by step, the numerical tools used in optical and X-ray tomography: simulating how light travels through tissue (forward problem), then recovering internal properties from boundary measurements (inverse problem).

## Overview

The work is split into three sections:

| Section | Topic | Main method |
|---|---|---|
| 1 | Forward problem modelling | Tissue absorption model, 1D and 2D Diffusion Approximation (DA) solved by Finite Differences |
| 2 | Measures and linear inverse problems | X-ray / Radon transform, sinograms, ART and Gradient Descent reconstruction |
| 3 | Non-linear inverse problems | Diffuse Optical Tomography (DOT), Jacobian by the adjoint method, Gauss-Newton reconstruction |

## Section 1: Forward Problem Modelling

- **Tissue absorption (E1.1):** optical absorption of water, fat and hemoglobin (Hb, HbO2) over 600-1100 nm, with spectra from the OMLC database. The total absorption is a linear mix weighted by tissue composition and blood oxygen saturation (StO2).
- **1D Diffusion Approximation (E1.2-E1.5):** finite-difference discretization with a central scheme, solved as a tridiagonal system (`solve_DA_1D`) with zero-fluence boundaries. Validated against the analytical Green's function with the method of images (**max relative error 2.08%**). Also tested with moving sources and an absorbing inclusion (shadowing effect).
- **2D Diffusion Approximation (E1.6-E1.7):** 5-point stencil on a rectangular grid, sparse matrix assembly and a point source (`solve_DA_2d`).

## Section 2: Linear Inverse Problems

- **Forward model (E2.1-E2.3):** derivation of the Beer-Lambert law from the radiative transfer equation without scattering, and the matrix form `p = W mu`.
- **Phantoms and sinograms (E2.4-E2.5):** disk, two-disk and block phantoms. The matrix model `W mu` matches the functional Radon transform with relative L2 errors of 3.77% (disk), 5.92% (two disks) and 3.25% (block).
- **Reconstruction (E2.6-E2.7):** two solvers were implemented and applied to the provided `sinogram1` and `sinogram2`:
  - **ART (Kaczmarz)** with row shuffling and relaxation (50 epochs, lambda = 0.8)
  - **Gradient Descent** with a step size from power iteration (1000 steps)

  ART converged faster and gave sharper edges. Gradient Descent gave smoother images with fewer streak artifacts but needed more iterations.

## Section 3: Non-Linear Inverse Problems (DOT)

- **Baseline and inclusions (E3.1-E3.3):** sparse FDM forward model with 10 sources (top) and 15 detectors (bottom), first in a homogeneous medium, then with a high-absorption and a low-absorption circular inclusion. The difference map shows which source-detector pairs are sensitive to the inclusions.
- **Jacobian (E3.4-E3.5):** Rytov Jacobian of size 150 x 3,876 computed with the adjoint state method. A single row shows the expected "banana-shaped" sensitivity path.
- **Gauss-Newton reconstruction (E3.6-E3.7):** iterative solver using ART for the linearized step, damping, a line search and bounds on k^2 (0.01 to 0.10 mm^-2). It was applied to a simulated phantom (E3.6) and to a provided dataset `dot_data.npz` (E3.7).

### Results and limitations

- The 1D and 2D forward solvers and the Radon-matrix model are validated against analytical or functional references.
- On the simulated DOT phantom (E3.6), the reconstruction **did not localize the two inclusions accurately**; the result is blurred and noisy near the boundaries, which reflects the ill-posed problem (3,876 unknowns from 150 measurements).
- On the provided dataset (E3.7), the residual norm fluctuated between about 240 and 310 rather than decreasing steadily, so the reconstruction should be seen as a work in progress rather than a converged result.

## Repository Structure

> Adjust this to match your actual files.

```
.
├── README.md
├── LICENSE
├── Practical_Work_MODELING_Iris_ZAME_NGUINA.pdf   # full report with figures
├── Project.ipynb                                  # code (notebook)
├── data/                                          # sinogram1, sinogram2, dot_data.npz
└── results/                                       # figures
```

## Requirements

- Python 3.9+
- numpy
- scipy
- matplotlib
- requests (used to download spectra from OMLC)
- `xray` module (`make_sinogram`, `radon_matrix`), provided with the course material

```bash
pip install numpy scipy matplotlib requests
```

## Usage

```bash
git clone https://github.com/YOUR_USERNAME/PORH-S8-Project.git
cd PORH-S8-Project
jupyter notebook Project.ipynb
```

Run the cells in order. Each exercise (E1.1 to E3.7) corresponds to a section of the report.

## Data

`sinogram1`, `sinogram2` and `dot_data.npz` were provided with the course. Check that you are allowed to redistribute them before making the repository public.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
