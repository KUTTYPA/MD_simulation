# 🧬 GROMACS Full Workflow Tutorial (Markdown Guide)

This tutorial outlines the steps for running Molecular Dynamics (MD) simulations in **GROMACS** using a protein structure. It includes structure preparation, neutralization, energy minimization, equilibration, and final MD production with **analysis**.

---

## 🧾 0. Protein Preparation (Manual via PyMOL/SPDBV)

> Before using GROMACS, manually check:

* ✅ Protein has a **single chain**.
* ✅ **Missing residues or atoms** are modeled (use **SPDBV** or **Modeller**).
* ✅ Remove ligands/water unless required.
* ✅ Rename file as `protein_clean.pdb`.

---

## 🧮 1. PDB to GROMACS Conversion

### X-ray vs NMR:

* For **X-ray** structures: choose the single chain with best resolution.
* For **NMR** structures: select **first model only**.

```bash
gmx pdb2gmx -f protein_clean.pdb -o processed.gro -water tip3p
```

* Choose forcefield (e.g., 15 → CHARMM36).
* Output: `processed.gro`, `topol.top`, `posre.itp`.

---

## 📦 2. Define Simulation Box

```bash
gmx editconf -f processed.gro -o boxed.gro -c -d 1.0 -bt cubic
```

* `-c` centers protein
* `-d` distance to box wall (1.0 nm)
* `-bt` box type: cubic

---

## 💧 3. Solvate the Box

```bash
gmx solvate -cp boxed.gro -cs spc216.gro -o solvated.gro -p topol.top
```

* Adds water molecules.
* `topol.top` will be updated with solvent count.

---

## ⚡ 4. Preprocess for Ion Addition

```bash
gmx grompp -f ions.mdp -c solvated.gro -p topol.top -o ions.tpr
```

* `ions.mdp`: simple mdp file with `emtol=1000`

---

## 🧪 5. Neutralize System (Add Ions)

```bash
gmx genion -s ions.tpr -o neutralized.gro -p topol.top -pname NA -nname CL -neutral
```

* Choose group “SOL” (typically group 13).
* Adds Na⁺/Cl⁻ to neutralize.

---

## 🧘 6. Energy Minimization Input File

Create `em.mdp` with:

```mdp
integrator = steep
emtol = 1000.0
emstep = 0.01
nsteps = 50000
```

---

## ⚙️ 7. Generate EM Binary

```bash
gmx grompp -f em.mdp -c neutralized.gro -p topol.top -o em.tpr
```

---

## 🏃 8. Run Energy Minimization

```bash
gmx mdrun -v -deffnm em
```

---

## 📉 9. Plot Potential Energy

```bash
gmx energy -f em.edr -o potential.xvg
# Select "Potential"
```

---

## 🔬 10. Equilibration: Two Types

### 🔹 NVT (constant volume, temp)

* Stabilizes **temperature**.
* Use `nvt.mdp`:

```mdp
define = -DPOSRES
integrator = md
nsteps = 50000
tc-grps = Protein Water_and_ions
```

### 🔸 NPT (constant pressure)

* Stabilizes **pressure & density**.
* Use `npt.mdp` with additional pressure coupling:

```mdp
pcoupl = Parrinello-Rahman
ref-p = 1.0
```

---

## 🧪 11. Run NVT Equilibration

```bash
gmx grompp -f nvt.mdp -c em.gro -r em.gro -p topol.top -o nvt.tpr
gmx mdrun -deffnm nvt
```

---

## 🧪 12. Run NPT Equilibration

```bash
gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -p topol.top -o npt.tpr
gmx mdrun -deffnm npt
```

---

## ✅ 13. Check Equilibration

### 13a. Analyze:

```bash
gmx energy -f nvt.edr -o temperature.xvg
gmx energy -f npt.edr -o pressure.xvg
gmx energy -f npt.edr -o density.xvg
```

### 13b. Visualize structure:

```bash
vmd npt.gro
```

---

## 🎯 14. Stability Analysis

### 14a. Run Analysis Tools

* **RMSD**:

```bash
gmx rms -s npt.tpr -f md.xtc -o rmsd.xvg
```

* **RMSF**:

```bash
gmx rmsf -s md.tpr -f md.xtc -o rmsf.xvg
```

* **Radius of Gyration**:

```bash
gmx gyrate -s md.tpr -f md.xtc -o gyrate.xvg
```

---

## 🧪 15. Final MD Run

### 15a. Create `md.mdp`:

```mdp
integrator = md
nsteps = 5000000 ; 10 ns
dt = 0.002
tcoupl = V-rescale
pcoupl = Parrinello-Rahman
constraints = all-bonds
```

### 15b. Run Simulation:

```bash
gmx grompp -f md.mdp -c npt.gro -p topol.top -o md.tpr
gmx mdrun -deffnm md
```

---

## 📊 Post-Simulation Analysis

Repeat Step **14a** with final `.xtc` and `.tpr` to assess:

* System stability
* Conformational behavior
* Folding/unfolding

---

## 🔗 Resources

* [GROMACS Docs](http://www.gromacs.org/Documentation)
* [SPDBV](https://spdbv.vital-it.ch/)
* [PyMOL](https://pymol.org/)

---
