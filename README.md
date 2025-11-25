# Toric Nash Blowups

A SageMath implementation for computing Nash blowups and normalized Nash blowups of toric varieties. This code was used to discover counterexamples to the Nash blowup conjectures in dimensions 3 and higher.

## Overview

This repository contains the computational tools described in the paper:

**"On a Computational Approach to the Nash Blowup Problem"**  
by Federico Castillo, Daniel Duarte, Maximiliano Leyton-Alvarez, and Alvaro Liendo

The Nash blowup is a method proposed by John Nash in the 1960s for resolving singularities of algebraic varieties. Our implementation translates the Nash blowup process into algorithms on combinatorial data (cones and semigroups) that define toric varieties, enabling systematic computational exploration of the Nash conjectures.

### Key Results

Through this computational approach, we:
- Found the first counterexamples to the Nash blowup conjecture (dimension ≥3)
- Found the first counterexamples to the normalized Nash blowup conjecture (dimension ≥4)
- Provided extensive positive evidence: millions of tested cases where Nash blowups do resolve singularities

## Mathematical Background

### Toric Varieties and Cones

Toric varieties are algebraic varieties with a rich combinatorial structure. An affine toric variety can be described by:
- A pointed cone σ⊂ℤⁿ (on the N-side), or equivalently
- Its dual cone σ∨⊂ℤⁿ (on the M-side) with corresponding semigroup S = σ∨ ∩ ℤⁿ

This implementation works primarily on the M-side, which provides a unified framework for both normalized and non-normalized Nash blowups.

### Nash Blowup Conjectures

**Nash Blowup Conjecture:** Iterating the Nash blowup on a singular variety eventually produces a non-singular variety.

**Normalized Nash Blowup Conjecture:** Iterating the normalized Nash blowup on a singular variety eventually produces a non-singular variety.

Both conjectures are **false** in dimensions 3 and 4 respectively (over characteristic zero fields).

## Installation

### Requirements

- SageMath 10.4 or higher
- Python 3.12 or higher
- Normaliz (for computing Hilbert bases)

### Setup

1. Clone this repository:
```bash
git clone https://github.com/efecastillo/toric-nash-blowups.git
cd toric-nash-blowups
```

2. Ensure SageMath is installed. If not, install via:
```bash
# On Ubuntu/Debian
sudo apt-get install sagemath

# Or use conda
conda install -c conda-forge sage
```

3. The code automatically uses Normaliz through SageMath's interface for computing Hilbert bases.

## Usage

### Basic Example: Normalized Nash Blowup

```python
import nash
from sage.matrix.constructor import matrix

# Define a cone by its ray generators (columns)
cone = matrix(ZZ, [
    [1, 0, 0, 1],
    [0, 1, 0, 1],
    [0, 0, 1, 1],
    [0, 0, 0, 3]
])

# Compute the normalized Nash blowup (characteristic 0)
children = nash.cone_normalized_nash_blowup(cone, Char=0)

# Get children in PALP normal form
children_normal = nash.cone_chi(cone, Char=0)
```

### Computing Resolution Trees

The main computational tool is the resolution tree, which explores all descendants of a cone under iterated Nash blowups:

```python
# Compute the complete resolution tree
resolution_tree = nash.cone_normalized_nash_tree(cone, Char=0)

# Check if resolution is achieved (reaches unimodular cone)
I = matrix.identity(ZZ, 4)
I.set_immutable()

if I in resolution_tree.vertices():
    print("Cone is resolved!")
else:
    print("Cone is not resolved")
    
# Check for cycles (counterexamples)
resolution_tree.allow_loops(False)
cycle = resolution_tree.longest_cycle()
if len(cycle) > 0:
    print(f"Found cycle of length {len(cycle.vertices())}")
```

### Non-normalized Nash Blowup (Semigroups)

For non-normalized Nash blowups, work with semigroups:

```python
# Define a semigroup by its generators
semig = matrix(ZZ, [
    [1, 0, 0, -2, 1, 2],
    [0, 1, 0, -1, -1, -2],
    [0, 0, 1, 2, 1, 1]
])

# Compute Nash blowup
children = nash.semig_nash_blowup(semig)

# Compute resolution tree
resolution_tree = nash.semig_nash_tree(semig)
```

## Module Reference

### `nash.py`

Main module for Nash blowup computations.

#### Key Functions

**`cone_normalized_nash_blowup(cone, Char=0)`**
- Input: Matrix representing cone generators (columns), optional characteristic
- Output: List of child cones as matrices
- Computes a single normalized Nash blowup step

**`cone_chi(cone, Char=0)`**
- Input: Matrix representing cone generators, optional characteristic  
- Output: Set of child cones in PALP normal form
- This is the function χ described in the paper

**`cone_normalized_nash_tree(root, DB=None, Char=0)`**
- Input: Root cone, optional database dict, optional characteristic
- Output: DiGraph representing complete resolution tree
- Computes all descendants via iterated normalized Nash blowups
- Returns False if input is not strictly convex or full-dimensional

**`semig_nash_blowup(semig)`**
- Input: Matrix representing semigroup generators (columns)
- Output: List of child semigroups
- Computes non-normalized Nash blowup

**`semig_chi(semig)`**
- Input: Matrix representing semigroup generators
- Output: Set of child semigroups in normal form

**`semig_nash_tree(root, DB=None)`**
- Input: Root semigroup, optional database
- Output: DiGraph representing complete resolution tree
- For non-normalized Nash blowups

### `helpers.py`

Auxiliary functions for cone and semigroup computations.

#### Key Functions

**`primitive(v)`**
- Returns primitive vector (gcd = 1)

**`tangent_cones(Q)`**
- Input: Polyhedron
- Output: List of tangent cones at vertices
- Each cone represented as matrix with generators as columns

**`is_unimodular(cone)`**
- Checks if cone is unimodular (corresponds to smooth variety)

**`palp_cone(cone)`**
- Returns PALP normal form of cone
- Canonical form using Kreuzer-Skarke algorithm

**`palp_semig(semig)`**
- Returns PALP normal form of semigroup

**`reduction(semig, check_pointed=False, check_generate=False)`**
- Returns minimal generators of semigroup
- Optional checks for pointed and full-dimensional properties

## Examples

See `demo.ipynb` for detailed examples including:
- Computing Nash blowups of specific cones
- Checking resolution of Reeves cones
- Finding counterexamples (cycles in resolution graphs)
- Working with special classes of singularities

### Counterexample to Normalized Nash Blowup (Dimension 4)

The cone that provides a loop (length-1 cycle) in characteristic 0:

```python
B = matrix(ZZ, [
    [1, 0, 0, 0, 2, 1],
    [0, 1, 0, 0, 3, 3],
    [0, 0, 1, 0, -2, -1],
    [0, 0, 0, 1, -1, -1]
])

tree = nash.cone_normalized_nash_tree(B)
# This cone is a child of itself!
```

### Counterexample to Nash Blowup (Dimension 3)

A semigroup that participates in a length-2 cycle:

```python
S = matrix(ZZ, [
    [1, 0, 0, -2, 1, 2],
    [0, 1, 0, -1, -1, -2],
    [0, 0, 1, 2, 1, 1]
])

tree = nash.semig_nash_tree(S)
# This semigroup is a grandchild of itself!
```

## Computational Notes

### Performance Considerations

- **Bottleneck:** Computing Hilbert bases (done via Normaliz)
- **Memory:** Store computed portions of digraphs to avoid recomputation
- **Scalability:** Feasible up to dimension ~5 for systematic exploration

### PALP Normal Form

Cones and semigroups are represented in PALP normal form to:
- Handle unimodular equivalence efficiently
- Provide canonical representatives
- Use vertex-facet incidence graphs for ordering

Note: For semigroups, uniqueness may fail due to automorphisms, but this doesn't significantly affect computational complexity since full-dimensional cones rarely have automorphisms.

## Computational Results Summary

From the paper's experiments:

### Positive Evidence
- **2D non-normalized:** 50,000+ toric surfaces resolved
- **3D normalized:** 2.5 million toric varieties resolved
- **Reeves cones ρ₃(j):** All cases j ≤ 1000 are resolved

### Counterexamples Found
- **3D Nash blowup:** Cycles of length 2, 3, 4, 5, 6, 7
- **4D normalized Nash blowup:** One known loop (minimal counterexample)
- **5D normalized Nash blowup:** Cycles of length 1, 2, 4, 5, 8, 9

## Citation

If you use this code in your research, please cite:

```bibtex
@article{CDLAL2025,
  title={On a Computational Approach to the Nash Blowup Problem},
  author={Castillo, Federico and Duarte, Daniel and Leyton-Alvarez, Maximiliano and Liendo, Alvaro},
  journal={arXiv preprint arXiv:2511.17862},
  year={2025}
}
```

## References

- Castillo, Duarte, Leyton-Alvarez, Liendo. "Nash blowup fails to resolve singularities in dimensions four and higher." To appear in Annals of Mathematics.
- Atanasov et al. "Resolving toric varieties with Nash blowups." Experimental Mathematics, 2011.
- Kreuzer & Skarke. "PALP: a package for analysing lattice polytopes." Computer Physics Communications, 2004.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues for:
- Performance improvements
- Additional examples
- Documentation enhancements
- Bug reports

## License

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 2 of the License, or (at your option) any later version.

## Authors

- Federico Castillo (Pontificia Universidad Católica de Chile)
- Alvaro Liendo (Universidad de Talca)

## Acknowledgments

This work was initiated at the AGREGA workshop at Universidad de Talca in January 2024. We thank the institution for its support and hospitality.

## Contact

For questions or comments:
- Federico Castillo: efecastillo.math@gmail.com
- Repository: https://github.com/efecastillo/toric-nash-blowups
