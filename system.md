# SYSTEM PROMPT

# RuO₂ Biaxial Strain Sweep

Starting only from the provided RuO₂ `POSCAR`, set up a biaxial strain sweep with separate `relax/` and `static/` stages.

## Strain grid

Apply equal strain to the **a and b lattice vectors only**:

$$
\mathbf a'=(1+\epsilon)\mathbf a,\qquad
\mathbf b'=(1+\epsilon)\mathbf b,\qquad
\mathbf c'=\mathbf c
$$

Use:

```python
np.linspace(-0.02, 0.10, 100)
```

giving **100 evenly spaced strain points from −2% through +10%, inclusive**.

Preserve fractional atomic coordinates.

## Directory structure

```text
biaxial/
├── make_strain.py
├── collect_static.py
├── relax/
│   ├── strain_-0.020000/
│   │   ├── POSCAR
│   │   ├── INCAR
│   │   └── KPOINTS
│   └── ...
└── static/
    └── strain_*/
        ├── POSCAR
        ├── INCAR
        └── KPOINTS
```

## Relax setup

`make_strain.py` should generate all 100 `relax/strain_*` directories.

Each directory gets the strained `POSCAR`, the same gamma-centered `12×12×18` `KPOINTS`, and this relaxation INCAR:

```text
SYSTEM = RuO2 biaxial relax

GGA = PE
LASPH = .TRUE.
GGA_COMPAT = .FALSE.
PREC = Accurate
ENCUT = 400

ISPIN = 2
MAGMOM = 5 -5 0 0 0 0

EDIFF = 1E-7
NELM = 200
ALGO = Normal
LREAL = .FALSE.

ISMEAR = 1
SIGMA = 0.1

IBRION = 2
NSW = 160
EDIFFG = 6E-06

LORBIT = 11
LWAVE = .FALSE.
LCHARG = .TRUE.
```

Do **not** implement the z-relaxation constraint or job submission. I will handle the required z-relax behavior manually using a separate run script/custom VASP workflow.

## KPOINTS

Every relax and static directory should contain:

```text
Gamma-centered 12x12x18
0
Gamma
12 12 18
0 0 0
```

## Static setup

Create `collect_static.py`.

After the relaxations have run:

1. Iterate through every `relax/strain_*` directory.
2. Check that `CONTCAR` exists and is non-empty.
3. Create the corresponding `static/strain_*` directory.
4. Copy:

   ```text
   relax/strain_X/CONTCAR
   ```

   to:

   ```text
   static/strain_X/POSCAR
   ```
5. Generate the static `INCAR`.
6. Generate the same gamma-centered `12×12×18` `KPOINTS`.
7. Skip incomplete relaxations and print their names.

Use this static INCAR:

```text
SYSTEM = RuO2 biaxial static

GGA = PE
LASPH = .TRUE.
GGA_COMPAT = .FALSE.
PREC = Accurate
ENCUT = 400

ISPIN = 2
MAGMOM = 5 -5 0 0 0 0

EDIFF = 1E-7
NELM = 200
ALGO = Normal
LREAL = .FALSE.

ISMEAR = 1
SIGMA = 0.1

IBRION = -1
NSW = 0

LORBIT = 11
LWAVE = .FALSE.
LCHARG = .TRUE.
```

The static calculation must use the relaxed `CONTCAR` exactly as produced. Do not further modify the lattice or atomic coordinates.

Keep everything simple and dependency-light. Python + NumPy is sufficient. Do not handle `POTCAR`, submission scripts, or VASP execution.
