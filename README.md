
# Depth_Anything_3_Motifs

Reconstruction of architectural bas-relief motifs from single monocular photographs using Depth Anything 3.

**Source:** Thanjavur (Tanjore) temple complex, Tamil Nadu, India.  
**Output:** Depth map + colored point cloud + fabrication-ready STL mesh — from one image, no stereo rig, no photogrammetry setup.

---
 Mesh to download - https://drive.google.com/file/d/1fZr83p7B8rgEfQRhdh1Uou-JDxpl2Jww/view?usp=sharing
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

![pipeline_diagram](https://github.com/user-attachments/assets/472747b3-fa4d-4b37-bb6a-591fb8756051)<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 900 520" width="900" height="520">
  <defs>
    <style>
      @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;600&amp;family=IBM+Plex+Sans:wght@300;400;600&amp;display=swap');
    </style>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#0e0e0e"/>
      <stop offset="100%" style="stop-color:#1a1410"/>
    </linearGradient>
    <linearGradient id="nodeGrad" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%" style="stop-color:#2a1f14"/>
      <stop offset="100%" style="stop-color:#1c1510"/>
    </linearGradient>
    <linearGradient id="accentGrad" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%" style="stop-color:#c8692a"/>
      <stop offset="100%" style="stop-color:#e8954a"/>
    </linearGradient>
    <linearGradient id="outputGrad" x1="0%" y1="0%" x2="0%" y2="100%">
      <stop offset="0%" style="stop-color:#1f2a1a"/>
      <stop offset="100%" style="stop-color:#151c12"/>
    </linearGradient>
    <filter id="glow">
      <feGaussianBlur stdDeviation="2" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
    <filter id="shadow">
      <feDropShadow dx="0" dy="2" stdDeviation="4" flood-color="#c8692a" flood-opacity="0.2"/>
    </filter>
    <marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#c8692a" opacity="0.8"/>
    </marker>
    <pattern id="grid" width="30" height="30" patternUnits="userSpaceOnUse">
      <path d="M 30 0 L 0 0 0 30" fill="none" stroke="#2a2018" stroke-width="0.5"/>
    </pattern>
  </defs>

  <!-- Background -->
  <rect width="900" height="520" fill="url(#bg)"/>
  <rect width="900" height="520" fill="url(#grid)" opacity="0.4"/>

  <!-- Top accent line -->
  <rect x="0" y="0" width="900" height="2" fill="url(#accentGrad)"/>

  <!-- Title block -->
  <text x="40" y="42" font-family="'IBM Plex Mono', monospace" font-size="11" font-weight="600" fill="#c8692a" letter-spacing="3">DEPTH ANYTHING 3 — MOTIF RECONSTRUCTION PIPELINE</text>
  <text x="40" y="60" font-family="'IBM Plex Sans', sans-serif" font-size="11" font-weight="300" fill="#7a6a5a" letter-spacing="1">Thanjavur Temple Bas-Relief · Single Monocular View → Fabrication-Ready Mesh</text>
  <line x1="40" y1="70" x2="860" y2="70" stroke="#2a2018" stroke-width="1"/>

  <!-- STAGE NODES — horizontal pipeline -->
  <!-- Node dimensions: 120w x 80h, y=100 center -->

  <!-- Stage 1: Input Photo -->
  <g filter="url(#shadow)">
    <rect x="40" y="95" width="120" height="85" rx="4" fill="url(#nodeGrad)" stroke="#c8692a" stroke-width="1.5"/>
    <rect x="40" y="95" width="120" height="4" rx="4" fill="#c8692a" opacity="0.9"/>
  </g>
  <text x="100" y="120" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#c8692a" text-anchor="middle" letter-spacing="1">INPUT</text>
  <text x="100" y="136" font-family="'IBM Plex Sans', sans-serif" font-size="13" font-weight="600" fill="#f0e8d8" text-anchor="middle">Photograph</text>
  <text x="100" y="152" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">single view</text>
  <text x="100" y="167" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">8K → 1400px</text>

  <!-- Arrow 1 -->
  <line x1="163" y1="137" x2="193" y2="137" stroke="#c8692a" stroke-width="1.5" marker-end="url(#arrow)" opacity="0.8"/>

  <!-- Stage 2: DA3 Inference -->
  <g filter="url(#shadow)">
    <rect x="196" y="95" width="120" height="85" rx="4" fill="url(#nodeGrad)" stroke="#c8692a" stroke-width="1.5"/>
    <rect x="196" y="95" width="120" height="4" rx="4" fill="#c8692a" opacity="0.9"/>
  </g>
  <text x="256" y="120" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#c8692a" text-anchor="middle" letter-spacing="1">INFERENCE</text>
  <text x="256" y="136" font-family="'IBM Plex Sans', sans-serif" font-size="13" font-weight="600" fill="#f0e8d8" text-anchor="middle">DA3 Model</text>
  <text x="256" y="152" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">GIANT-LARGE</text>
  <text x="256" y="167" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">or DA3BASE</text>

  <!-- Arrow 2 -->
  <line x1="319" y1="137" x2="349" y2="137" stroke="#c8692a" stroke-width="1.5" marker-end="url(#arrow)" opacity="0.8"/>

  <!-- Stage 3: Depth Normalize -->
  <g filter="url(#shadow)">
    <rect x="352" y="95" width="120" height="85" rx="4" fill="url(#nodeGrad)" stroke="#c8692a" stroke-width="1.5"/>
    <rect x="352" y="95" width="120" height="4" rx="4" fill="#c8692a" opacity="0.9"/>
  </g>
  <text x="412" y="120" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#c8692a" text-anchor="middle" letter-spacing="1">NORMALIZE</text>
  <text x="412" y="136" font-family="'IBM Plex Sans', sans-serif" font-size="13" font-weight="600" fill="#f0e8d8" text-anchor="middle">Depth Map</text>
  <text x="412" y="152" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">mask + bilateral</text>
  <text x="412" y="167" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">center ROI stats</text>

  <!-- Arrow 3 -->
  <line x1="475" y1="137" x2="505" y2="137" stroke="#c8692a" stroke-width="1.5" marker-end="url(#arrow)" opacity="0.8"/>

  <!-- Stage 4: Point Cloud -->
  <g filter="url(#shadow)">
    <rect x="508" y="95" width="120" height="85" rx="4" fill="url(#nodeGrad)" stroke="#c8692a" stroke-width="1.5"/>
    <rect x="508" y="95" width="120" height="4" rx="4" fill="#c8692a" opacity="0.9"/>
  </g>
  <text x="568" y="120" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#c8692a" text-anchor="middle" letter-spacing="1">RECONSTRUCT</text>
  <text x="568" y="136" font-family="'IBM Plex Sans', sans-serif" font-size="13" font-weight="600" fill="#f0e8d8" text-anchor="middle">Point Cloud</text>
  <text x="568" y="152" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">RGBD → Open3D</text>
  <text x="568" y="167" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">voxel 5–10mm</text>

  <!-- Arrow 4 -->
  <line x1="631" y1="137" x2="661" y2="137" stroke="#c8692a" stroke-width="1.5" marker-end="url(#arrow)" opacity="0.8"/>

  <!-- Stage 5: Mesh -->
  <g filter="url(#shadow)">
    <rect x="664" y="95" width="120" height="85" rx="4" fill="url(#nodeGrad)" stroke="#c8692a" stroke-width="1.5"/>
    <rect x="664" y="95" width="120" height="4" rx="4" fill="#c8692a" opacity="0.9"/>
  </g>
  <text x="724" y="120" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#c8692a" text-anchor="middle" letter-spacing="1">MESH</text>
  <text x="724" y="136" font-family="'IBM Plex Sans', sans-serif" font-size="13" font-weight="600" fill="#f0e8d8" text-anchor="middle">Surface Mesh</text>
  <text x="724" y="152" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">Poisson depth=9</text>
  <text x="724" y="167" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" text-anchor="middle">or Ball Pivoting</text>

  <!-- Arrow 5 down to outputs -->
  <line x1="724" y1="183" x2="724" y2="213" stroke="#c8692a" stroke-width="1.5" marker-end="url(#arrow)" opacity="0.8"/>

  <!-- OUTPUT BLOCK -->
  <g filter="url(#shadow)">
    <rect x="40" y="216" width="840" height="70" rx="4" fill="url(#outputGrad)" stroke="#4a7a3a" stroke-width="1"/>
    <rect x="40" y="216" width="840" height="3" rx="4" fill="#6ab04a" opacity="0.7"/>
  </g>
  <text x="60" y="238" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#6ab04a" letter-spacing="2">OUTPUTS PER MOTIF</text>

  <!-- Output items -->
  <text x="100" y="260" font-family="'IBM Plex Mono', monospace" font-size="10" fill="#a0b890" text-anchor="middle">depth_map.png</text>
  <text x="100" y="274" font-family="'IBM Plex Sans', sans-serif" font-size="9" fill="#5a7050" text-anchor="middle">8-bit preview</text>

  <line x1="180" y1="240" x2="180" y2="278" stroke="#2a3a22" stroke-width="1"/>

  <text x="280" y="260" font-family="'IBM Plex Mono', monospace" font-size="10" fill="#a0b890" text-anchor="middle">mesh_output.ply / .stl</text>
  <text x="280" y="274" font-family="'IBM Plex Sans', sans-serif" font-size="9" fill="#5a7050" text-anchor="middle">Poisson reconstruction</text>

  <line x1="420" y1="240" x2="420" y2="278" stroke="#2a3a22" stroke-width="1"/>

  <text x="530" y="260" font-family="'IBM Plex Mono', monospace" font-size="10" fill="#a0b890" text-anchor="middle">mesh_bpa.ply / .stl</text>
  <text x="530" y="274" font-family="'IBM Plex Sans', sans-serif" font-size="9" fill="#5a7050" text-anchor="middle">Ball Pivoting reconstruction</text>

  <line x1="680" y1="240" x2="680" y2="278" stroke="#2a3a22" stroke-width="1"/>

  <text x="770" y="260" font-family="'IBM Plex Mono', monospace" font-size="10" fill="#a0b890" text-anchor="middle">relief_exaggerated.stl</text>
  <text x="770" y="274" font-family="'IBM Plex Sans', sans-serif" font-size="9" fill="#5a7050" text-anchor="middle">heightfield · fabrication-ready</text>

  <!-- PROBLEM ANNOTATIONS SECTION -->
  <line x1="40" y1="305" x2="860" y2="305" stroke="#2a2018" stroke-width="1"/>
  <text x="40" y="325" font-family="'IBM Plex Mono', monospace" font-size="9" font-weight="600" fill="#7a6a5a" letter-spacing="2">DOMAIN-SPECIFIC CHALLENGES — STONE BAS-RELIEF</text>

  <!-- Challenge boxes -->
  <!-- C1 -->
  <rect x="40" y="338" width="190" height="60" rx="3" fill="#1a1410" stroke="#3a2a1a" stroke-width="1"/>
  <text x="50" y="356" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a">SHALLOW RELIEF</text>
  <text x="50" y="372" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">5–15mm actual depth</text>
  <text x="50" y="386" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">DA3 compresses further</text>

  <!-- C2 -->
  <rect x="245" y="338" width="190" height="60" rx="3" fill="#1a1410" stroke="#3a2a1a" stroke-width="1"/>
  <text x="255" y="356" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a">NEAR=LOW INVERSION</text>
  <text x="255" y="372" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">DA3 convention: near surfaces</text>
  <text x="255" y="386" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">return low values → invert</text>

  <!-- C3 -->
  <rect x="450" y="338" width="190" height="60" rx="3" fill="#1a1410" stroke="#3a2a1a" stroke-width="1"/>
  <text x="460" y="356" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a">BACKGROUND SKEW</text>
  <text x="460" y="372" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">Stone wall dominates depth</text>
  <text x="460" y="386" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">stats → center ROI mask</text>

  <!-- C4 -->
  <rect x="655" y="338" width="205" height="60" rx="3" fill="#1a1410" stroke="#3a2a1a" stroke-width="1"/>
  <text x="665" y="356" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a">BOUNDARY HALOS</text>
  <text x="665" y="372" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">Mask edges produce noise</text>
  <text x="665" y="386" font-family="'IBM Plex Sans', sans-serif" font-size="10" fill="#9a8878">rings → extra erode 6px</text>

  <!-- Dataset note -->
  <line x1="40" y1="415" x2="860" y2="415" stroke="#2a2018" stroke-width="1"/>
  <text x="40" y="435" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#7a6a5a" letter-spacing="2">DATASET — THANJAVUR TEMPLE COMPLEX, TAMIL NADU · 20+ IMAGES · 5 TYPOLOGIES</text>

  <!-- Typology chips -->
  <rect x="40" y="448" width="90" height="24" rx="12" fill="#1f1510" stroke="#c8692a" stroke-width="1" opacity="0.7"/>
  <text x="85" y="464" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a" text-anchor="middle">Padma</text>

  <rect x="144" y="448" width="90" height="24" rx="12" fill="#1f1510" stroke="#c8692a" stroke-width="1" opacity="0.7"/>
  <text x="189" y="464" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a" text-anchor="middle">Vyala</text>

  <rect x="248" y="448" width="106" height="24" rx="12" fill="#1f1510" stroke="#c8692a" stroke-width="1" opacity="0.7"/>
  <text x="301" y="464" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a" text-anchor="middle">Kirtimukha</text>

  <rect x="368" y="448" width="130" height="24" rx="12" fill="#1f1510" stroke="#c8692a" stroke-width="1" opacity="0.7"/>
  <text x="433" y="464" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a" text-anchor="middle">Narrative Frieze</text>

  <rect x="512" y="448" width="120" height="24" rx="12" fill="#1f1510" stroke="#c8692a" stroke-width="1" opacity="0.7"/>
  <text x="572" y="464" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#c8692a" text-anchor="middle">Purna-ghata</text>

  <!-- Result stat -->
  <rect x="660" y="448" width="240" height="24" rx="4" fill="#1a2014" stroke="#6ab04a" stroke-width="1" opacity="0.8"/>
  <text x="780" y="464" font-family="'IBM Plex Mono', monospace" font-size="9" fill="#6ab04a" text-anchor="middle">2.2M verts · 4.4M faces · 1 photo</text>

  <!-- Bottom line -->
  <rect x="0" y="516" width="900" height="2" fill="url(#accentGrad)" opacity="0.4"/>
  <text x="860" y="512" font-family="'IBM Plex Mono', monospace" font-size="8" fill="#3a2a1a" text-anchor="end">github.com/libishm1/Depth_Anything_3_Motifs</text>
</svg>


![params_diagram](https://github.com/user-attachments/assets/05a27b6f-06a1-4b37-9a99-9d9b65bae6a0)

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
