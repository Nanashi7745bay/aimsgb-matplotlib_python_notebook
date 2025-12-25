# aimsgb-matplotlib_python_notebook

Language: [Japanese (日本語)](README_JP.md)

A guide for building and analyzing grain boundary (GB) models using the `aimsgb` library.

## Preparation

First, install the `aimsgb` library.

```bash
pip install aimsgb
```

## Basic Usage

### GrainBoundary Class

This class constructs a grain boundary model based on the input structure, specified rotation axis, sigma value, and grain boundary plane.

#### Argument List
- **`axis`**: **Rotation axis** (e.g., `[1, 1, 0]`)
- **`sigma`**: **Sigma value ($\Sigma$)** (e.g., `3`)
- **`face`**: **Grain boundary plane** (e.g., `[1, 1, 0]`)
- **`s_input`**: **Crystal structure** (`Grain` object)
- **`uc_a`**: **Left bulk expansion factor** (Default: `1`)
- **`uc_b`**: **Right bulk expansion factor** (Default: `1`)

> [!TIP]
> For better stability of grain boundary energy, it is recommended to set `uc_a` and `uc_b` to the same value.

### Basic Example

```python
from aimsgb import GrainBoundary, Grain
import os

# Load crystal structure
s_input = Grain.from_file("POSCAR_MASnI3")

axis = [1, 1, 0]
sigma = 3
face = [1, 1, 0]

# Create grain boundary model
gb = GrainBoundary(axis, sigma, face, s_input, uc_a=2, uc_b=1)

# Stack two grains
structure = Grain.stack_grains(gb.grain_a, gb.grain_b, direction=gb.direction)

# Save the structure
folder_path = f"masn001/sigma{sigma}_{face}"
if not os.path.exists(folder_path):
    os.makedirs(folder_path)
structure.to(filename=os.path.join(folder_path, "POSCAR"))
```

## Useful Utilities

### GBInformation
Displays available CSL information for a given rotation axis and maximum sigma value.

```python
from aimsgb import GBInformation

print(GBInformation([0, 0, 1], 20).__str__())
```

### Batch Generation of GB Models (`get_allgb`)

This is an example function that automatically generates grain boundary models up to $\Sigma \le 13$ for a specific rotation axis, saving structure files (POSCAR) and configuration details (Condition.txt) into separate directories.

```python
def get_allgb(axis, s_input):
    from aimsgb import GrainBoundary, Grain, GBInformation
    import os

    strings = GBInformation(axis, 13).__str__()
    # ... (Refer to the notebook for parsing logic details)
    # Creates folders based on each Sigma value and plane index, then saves POSCAR and Condition.txt.
```

## Other Tools and Examples

- **Retrieve structure from Materials Project**: You can fetch structures directly using an ID: `Grain.from_mp_id("mp-2815")`.
- **CSL Generator**: Use `gb_code.csl_generator` to calculate rotation angles and minimal cells corresponding to specific sigma values.

## Notes

> [!WARNING]
> **Directory Separators**:
> Some code snippets in the notebook may contain backslashes (`\`). On Linux or macOS, please change them to forward slashes (`/`).
> Example: `folder_path = "masn001/sigma" + str(sigma) + "_" + str(face)`

- **Dependencies**: `aimsgb` and `pymatgen` are required.
- **Non-Orthogonal Lattices**: When using non-orthogonal grains, you might see a warning that the lattice will be forced to be orthogonal, which could potentially break inherent symmetries.
