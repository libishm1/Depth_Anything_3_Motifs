

# Depth_Anything_3_Motifs

Reconstruction of architectural bas-relief motifs from single monocular photographs using Depth Anything 3.

**Source:** Thanjavur (Tanjore) temple complex, Tamil Nadu, India.  
**Output:** Depth map + colored point cloud + fabrication-ready STL mesh — from one image, no stereo rig, no photogrammetry setup.
Mesh to download - https://drive.google.com/file/d/1fZr83p7B8rgEfQRhdh1Uou-JDxpl2Jww/view?usp=sharing
---

<img width="987" height="898" alt="image" src="https://github.com/user-attachments/assets/6d24ce9c-a356-484c-be08-b1f3efb9b4ce" />

*Reconstructed relief mesh. 2,217,623 vertices · 4,428,998 faces. Source: single photograph, Thanjavur, 2026.*

---

## Research Context

Thanjavur temple architecture follows canonical Agamic rules — specifically the *Manasara* and *Mayamata* treatises — that govern the geometry of every carved motif. The resulting forms appear fluid and organic but are structurally rule-governed. This tension makes them an ideal test case for monocular depth reconstruction: can a learned depth model recover canonical geometry from a single view?

This pipeline tests that question across a dataset of 20+ motif photographs, covering typologies including Padma (lotus), Vyala (leogryph), Kirtimukha (face), and narrative frieze panels.

The downstream application is digital fabrication — specifically robotic milling and relief reproduction — where a single site photograph becomes a fabrication-ready mesh without photogrammetric multi-view capture.

---

## Pipeline

```
Input photograph (single view)
    ↓
Depth Anything 3 inference  [DA3NESTED-GIANT-LARGE or DA3BASE]
    ↓
Depth normalization + mask generation
    ↓
Bilateral smoothing (edge-preserving)
    ↓
RGBD → Point cloud (Open3D)
    ↓
Poisson reconstruction  →  mesh_output.stl
Ball Pivoting reconstruction  →  mesh_bpa.stl
Heightfield export (z-exaggerated)  →  relief_exaggerated.stl
    ↓
Fabrication-ready STL
```

---

## Key Parameters

These are the critical values that make the pipeline work on stone bas-relief surfaces specifically. Generic defaults produce flat or noisy results.

| Parameter | Value | Why |
|---|---|---|
| `z_exaggeration_export` | 24.0 | Temple relief is shallow (~5–15mm). Monocular depth compresses it further. Exaggeration recovers perceptible geometry. |
| `invert` | True | DA3 returns near=low values. Relief surfaces read correctly only when inverted. |
| `use_center_roi` | True | Background stone and frame skew normalization statistics. Central ROI isolates the motif zone. |
| `extra_erode_px` | 6 | Halo artifacts appear at mask boundaries. Additional erosion removes them before mesh generation. |
| `detail_boost` | 0.25 | High-pass filter added back to normalized depth. Recovers fine surface texture without amplifying noise. |
| `far_pct` | 85 | Depth percentile cutoff for background masking. Retains foreground motif, discards wall context. |

---

## Outputs Per Motif

Each processed image produces:

- `depth_map.png` — normalized depth preview (8-bit)
- `mesh_output.ply / .stl` — Poisson reconstruction
- `mesh_bpa.ply / .stl` — Ball Pivoting reconstruction  
- `relief_exaggerated.stl` — heightfield mesh, z-exaggerated, fabrication-ready

---

## Environment

```bash
pip install torch torchvision
pip install numpy==2.0.2
pip install pillow<12
pip install opencv-python-headless
pip install open3d
pip install git+https://github.com/ByteDance-Seed/Depth-Anything-3.git --no-deps
```

NumPy version pinning is required. DA3 pulls a conflicting version without `--no-deps`.  
Tested on Google Colab (T4 GPU). Works on local CUDA GPU with no changes.

---

## Colab Notebook

[Open in Google Colab](https://colab.research.google.com/drive/1I8iBNt6_6c2zGXEJoRk3srMS6JCt8uWp)

---

## Dataset

**Location:** Thanjavur Brihadeeswarar temple complex and surrounding gopurams, Tamil Nadu.  
**Capture protocol:**
- Camera parallel to relief surface
- Diffuse light preferred (overcast / shade)
- Scale reference in frame where possible
- No HDR processing

**Motif typologies in dataset:**
- Padma (lotus) — floral, radial symmetry
- Vyala (leogryph) — composite animal form
- Kirtimukha (face/mask) — frontal, high relief
- Narrative frieze panels — low relief, horizontal composition

Dataset expansion ongoing. Target: 5 typologies × minimum 4 specimens each.

---

## Citation / Reference

If you use this pipeline or dataset, please cite:

```
Murugesan, L. (2026). Depth_Anything_3_Motifs: Monocular depth reconstruction 
of Thanjavur temple bas-relief motifs. GitHub. 
https://github.com/libishm1/Depth_Anything_3_Motifs
```

---

## Status

- [x] Pipeline complete and tested (single motif)
- [x] Parameter tuning for stone bas-relief surfaces
- [ ] Batch processing script for full dataset
- [ ] Cross-motif geometry comparison
- [ ] Fabrication output validation (CNC/robotic milling)

---

## Author

Libish Murugesan — researcher and lecturer in Robotics for Architecture, Riyadh.  
[Portfolio](https://libishm1.github.io/portfolio/) · [LinkedIn](https://www.linkedin.com/in/libish-murugesan/) · [GitHub](https://github.com/libishm1)
