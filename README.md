# Rational Mutagenesis of [FeFe]-Hydrogenase for O₂ Tolerance — CpI (1FEH)

> **Self-learning project** | Structural Bioinformatics | PyMOL · CAVER 3.0  
> First-year Biochemistry student, Universidad de Sevilla

---

## Overview

This project explores **computational rational mutagenesis** of the [FeFe]-hydrogenase CpI from *Clostridium pasteurianum* (PDB: [1FEH](https://www.rcsb.org/structure/1FEH)), with the goal of designing a mutant with improved oxygen tolerance while preserving H₂ production activity.

The core idea is simple: H₂ and O₂ share the same gas channel to reach the active site (the H-cluster), but they differ slightly in kinetic diameter — H₂ (2.89 Å) vs O₂ (3.46 Å). By introducing a bulky residue at the bottleneck of the main gas tunnel, we can theoretically restrict O₂ access without completely blocking H₂.

This is a **learning project**. The workflow was built step by step as an introduction to structural bioinformatics tools, and all decisions are documented and reasoned below.

---

## Why This Matters — Industrial Context

One of the main bottlenecks in biological hydrogen production is the 
oxygen sensitivity of [FeFe]-hydrogenases. These enzymes are among the 
most catalytically efficient H₂-producing systems known, but they are 
irreversibly inactivated by O₂ at trace concentrations — making them 
incompatible with real-world production conditions, where full anaerobicity 
is difficult and costly to maintain.

Engineering O₂-tolerant variants could unlock the use of these enzymes in:
- Photobiological H₂ production (coupled to photosynthesis)
- Dark fermentation systems using agricultural residues
- Whole-cell or cell-free bioreactors for green hydrogen

A hydrogenase that tolerates even modest O₂ exposure would dramatically 
reduce infrastructure costs and open the door to scalable biohydrogen 
production — a key piece in the transition to a sustainable energy economy.

This project is a first computational step in that direction.

---

## Biological Background

[FeFe]-hydrogenases are among the most efficient known catalysts for H₂ production, but their major limitation is extreme sensitivity to O₂, which irreversibly damages the H-cluster. Engineering O₂-tolerant variants is a key challenge in the field of biological hydrogen production.

The **H-cluster** consists of a [4Fe-4S] cubane bridged to a unique [2Fe] subcluster where catalysis occurs. Gas molecules (H₂ and O₂) reach it through hydrophobic tunnels that run through the protein interior — no membrane channel, just passive diffusion through cavities.

---

## Workflow

### 1. Structure Loading

The crystal structure of CpI was fetched directly from the RCSB Protein Data Bank:

```python
fetch 1feh
```

The H-cluster and surrounding residues were visualized using a molecular surface representation with the active site residues shown as sticks.

---

### 2. Gas Tunnel Identification with CAVER 3.0

To identify the gas channels, we used **CAVER 3.0** ([caver.cz](https://caver.cz)), a PyMOL plugin that computes tunnels and cavities in protein structures.

**Starting point:** The H-cluster was selected as the origin point for tunnel calculation, specifically the [2Fe] subcluster (the catalytic site), as all gas molecules must reach this point to interact with the enzyme.

**Parameters used:**
- Minimum probe radius: 0.9 Å
- Shell depth: 4
- Shell radius: 3
- Clustering threshold: 3.5

CAVER identified **multiple tunnels**, each rendered in a different color according to probability:

| Color | Tunnel | Interpretation |
|-------|--------|----------------|
| 🟢 Green | Cluster 1 & 2 | Most probable — main gas channel |
| 🔵 Blue | Cluster 3 | Secondary pathway |
| 🔴 Red | Cluster 4 | Minor pathway |
| 🟡 Yellow | Cluster 5 | Minor pathway |

**We focused on the green tunnel (cluster 1)** as it is the shortest, widest, and most direct route from the protein surface to the H-cluster — consistent with literature describing the main hydrophobic gas channel in CpI.

---

### 3. Bottleneck Identification

To find the narrowest point of the main tunnel (the bottleneck), we selected a CAVER sphere atom near the constriction and identified surrounding protein residues within 4 Å:

```python
select bottleneck, byres (polymer within 4 of sele)
label bottleneck and name CA, "%s%s" % (resn, resi)
show sticks, bottleneck
```

The residues lining the bottleneck of tunnel 1 were:

| Residue | Notes |
|---------|-------|
| PHE417 | Best candidate — aromatic, oriented into the tunnel |
| THR275 | Possible — small, space available |
| PRO324 | Avoided — backbone geometry |
| CYS299 | Avoided — likely cofactor-coordinating |

---

### 4. Mutation Design: PHE417 → TRP

**PHE417** was selected as the mutation target based on:

- Its aromatic ring is already **oriented toward the tunnel interior**
- Phe → Trp is an **aromatic-to-aromatic** substitution (benzyl → indol), minimally disruptive to local structure
- Trp adds ~28 Å³ of extra volume, which should reduce the tunnel radius from ~1.7 Å to ~1.3 Å — sufficient to block O₂ (3.46 Å) while potentially still allowing H₂ (2.89 Å)

The mutation was performed using the **PyMOL Mutagenesis Wizard** (Wizard → Mutagenesis → TRP), selecting the rotamer with minimal steric clashes among the 8 available conformations.

---

### 5. Output

The mutant structure was saved as:
```
structures/1FEH_2.0.pdb   →   F417W mutant
```

The original wildtype structure is:
```
structures/1FEH.pdb
```

---

## Repository Structure

```
Mutantes_Hidrogenasa_O2t/
│
├── README.md
├── structures/
│   ├── 1FEH.pdb             # Wildtype CpI
│   └── 1FEH_2.0.pdb         # F417W mutant
│
├── images/
│   ├── Cartoon_Hidrogenase.png
│   ├── H2_Channel.png
│   ├── H2_Channel_2.png
│   ├── H2_Possible_paths.png
│   ├── PHE417_premutagenesys.png
│   └── TRP417_postmutagenesys.png
│
└── sessions/
    └── 1FEH_2.0.pse         # PyMOL session file
```

---

## Tools Used

- [PyMOL](https://pymol.org) — molecular visualization and mutagenesis
- [CAVER 3.0](https://caver.cz) — tunnel/channel analysis
- [RCSB PDB](https://rcsb.org) — structure source

---

## Limitations & Next Steps

- PyMOL mutagenesis is **static modeling** — no energy minimization is performed. A real validation would require refinement with Rosetta or MODELLER.
- The selected rotamer shows minor steric clashes, suggesting this mutation may slightly reduce catalytic activity — a known trade-off in the literature for O₂-tolerant variants.
- Future steps could include:
  - Re-running CAVER on the F417W mutant to quantify bottleneck radius change
  - Gas docking with AutoDock Vina (H₂ vs O₂ binding energy comparison)
  - Molecular dynamics with GROMACS to assess tunnel dynamics

---

## Disclaimer

This is a self-learning project developed as part of my autodidactic training in structural bioinformatics and protein engineering. It is not peer-reviewed and should not be taken as a validated scientific result. All reasoning and decisions are documented for transparency.

---

*If you have suggestions or spot any errors, feel free to open an issue.*
