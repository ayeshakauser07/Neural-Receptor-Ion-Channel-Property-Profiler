Neural Receptor & Ion Channel Property Profiler
A computational tool that predicts transmembrane (membrane-spanning) regions in neural
receptor and ion channel proteins using hydrophobicity and charge analysis, validated
against UniProt's curated ground-truth annotations.

**What this does**

Neurons communicate through receptors and ion channels — proteins embedded in the cell
membrane that let electrical and chemical signals cross the neuron's wall. This project
takes a protein's amino acid sequence and predicts **which parts of that sequence
physically cross the membrane**, using the classic Kyte-Doolittle hydrophobicity scale.

**Proteins studied:**
| Protein | Gene | Role |
| NMDA receptor subunit | GRIN1 | Excitatory neurotransmitter receptor |
| GABA-A receptor subunit | GABRA1 | Inhibitory neurotransmitter receptor |
| Voltage-gated sodium channel | SCN1A | Generates the action potential |
| AMPA receptor subunit | GRIA1 | Fast excitatory neurotransmitter receptor |

**Method**

1. **Fetch** the canonical protein sequence from the UniProt REST API.
2. **Score hydrophobicity** using a sliding 19-residue window over the Kyte-Doolittle
   scale — membrane-embedded regions must be hydrophobic to sit comfortably in the
   lipid bilayer.
3. **Score net charge** with a smaller sliding window, as a secondary signal.
4. **Flag transmembrane regions** using **hysteresis thresholding**: find confident
   high-hydrophobicity "seed" positions, then expand outward at a lower threshold to
   recover the full width of each helix (a single fixed cutoff only catches the narrow
   hydrophobic *core* of a real helix, since many ion-channel TM helices contain polar,
   pore-facing residues).
5. **Validate** predictions against UniProt's own manually curated "Transmembrane"
   feature annotations, reporting residue-level overlap.

 **Results**

| Protein | True TM domains | Predicted | Overlap |
| GRIN1 (NMDA) | 3 | 4 | 93.9% |
| GABRA1 (GABA-A) | 4 | 3 | 63.5% |
| SCN1A (Na⁺ channel) | 24 | 16 | 65.4% |
| GRIA1 (AMPA) | 3 | 4 | 90.5% |

**Why the results vary — an honest breakdown**

- **GRIN1 / GRIA1 score highest** because their TM helices are spaced far apart with no
  interference between neighbors. The one recurring false positive in both is a
  hydrophobic region at the very start of the sequence — this is almost certainly the
  **signal peptide** (a short hydrophobic tag that directs the protein to the membrane
  during synthesis, then gets cleaved off before the mature protein exists). It's real
  hydrophobic sequence, correctly detected, but not a true transmembrane domain.
- **GABRA1 / SCN1A score lower** because their TM helices are **densely packed**, sitting
  very close together with almost no gap between them. A pure hydrophobicity-threshold
  method structurally cannot always resolve two helices sitting back-to-back.
- **SCN1A specifically under-detects a known feature**: voltage-gated sodium channels
  contain **S4 voltage-sensor helices**, which are genuinely transmembrane but rely on a
  repeating pattern of positively charged residues rather than pure hydrophobicity —
  a documented, well-known blind spot for hydrophobicity-only methods.

This is treated as a genuine finding, not a flaw to hide: a simple, 40-year-old
hydrophobicity heuristic has real, explainable limits compared to modern ML-based
predictors (e.g. TMHMM, DeepTMHMM), and identifying *why* it succeeds or fails on
specific proteins is the actual point of the validation step.

**How to run**
Open `Neural Receptor & Ion Channel Property Profiler.ipynb` in Jupyter or Google Colab and run all cells top to
bottom. 
It will:
1. Download sequences for all 4 target proteins from UniProt
2. Predict transmembrane regions
3. Save a hydrophobicity/charge profile plot per protein
4. Run validation against UniProt's real annotations and print an overlap report

**Known limitations**

- Kyte-Doolittle hydrophobicity is a simple, fixed-table heuristic from 1982 — it has
  no knowledge of 3D structure and cannot distinguish a signal peptide from a true TM
  helix based on sequence alone.
- Charged/amphipathic transmembrane helices (e.g. S4 voltage-sensor segments) are
  systematically under-detected, since the method relies primarily on hydrophobicity.
- Densely packed polytopic channels (many TM helices close together) are harder to
  resolve than sparse single-pass receptors.
- This is not a substitute for structure-based or ML-based TM predictors — it's a
  transparent, interpretable baseline method with clearly understood failure modes.

