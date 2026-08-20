可以。下面這份 README 是按照你目前這個 **RCWA + metallic nanorod + diffraction orders + optical phase + metalens application** 的方向寫的。你可以直接存成 `README.md`。

# RCWA Metallic Nanorod Diffraction & Phase Simulator

A Python-based RCWA (Rigorous Coupled-Wave Analysis) simulator for studying the optical response of periodic metallic nanorod arrays.

The code is designed to investigate how nanorod geometry and lattice structure affect:

* Reflection
* Transmission
* Absorption
* Individual diffraction orders
* Diffraction efficiencies
* Complex transmission/reflection amplitudes
* Optical phase
* Phase response versus wavelength
* Phase response versus nanorod diameter

The long-term goal is to use the simulated unit-cell responses as a foundation for **metalens design**.

---

## 1. What does this code do?

The simulator takes a periodic nanorod unit cell and solves its electromagnetic response using RCWA.

The basic workflow is:

```text
Nanorod geometry
       ↓
      RCWA
       ↓
┌──────────────────────────────┐
│ Reflection                   │
│ Transmission                 │
│ Absorption                   │
│ Diffraction orders           │
│ Complex optical amplitudes   │
│ Optical phase                │
└──────────────────────────────┘
```

The program can handle both:

* Square lattices
* Hexagonal / triangular lattices

The nanorods are modeled as cylindrical metallic rods, with Au (gold) as the default material.

---

## 2. Physical problem

The structure consists of a periodic array of metallic nanorods on a substrate.

For example:

```text
        Air
────────────────────────

       ││      ││
       ││      ││
       ││      ││
       ││      ││
────────────────────────
        Glass substrate
```

Each nanorod is characterized primarily by:

* Diameter
* Height
* Position inside the unit cell

The periodicity is defined by the lattice period.

The simulator then determines how an incident plane wave interacts with the structure.

---

## 3. Required inputs

The main simulation requires only three parameters:

```python
lattice
rod_height
rod_diameter
```

For example:

```python
result = simulate(
    lattice="square",
    rod_height=150e-9,
    rod_diameter=100e-9
)
```

The remaining parameters have default values.

Optional parameters can be specified when more control is required:

```python
result = simulate(
    lattice="hexagonal",
    rod_height=150e-9,
    rod_diameter=100e-9,
    period=500e-9,
    wavelength_min=450e-9,
    wavelength_max=850e-9,
    wavelength_step=5e-9
)
```

---

## 4. Default simulation setup

Typical defaults include:

| Parameter        | Default          |
| ---------------- | ---------------- |
| Metal            | Au               |
| Superstrate      | Air              |
| Substrate        | Glass            |
| Substrate index  | 1.5              |
| Incidence        | Normal incidence |
| Polarization     | x-polarized      |
| Nanorod shape    | Cylinder         |
| Wavelength range | 400–1000 nm      |
| Lattice          | Square           |
| Period           | 500 nm           |

The exact defaults are defined in the Python source code and can be modified there.

---

## 5. Lattice structures

### Square lattice

The square lattice uses primitive vectors:

```text
a1 = (P, 0)
a2 = (0, P)
```

where `P` is the lattice period.

### Hexagonal / triangular lattice

The hexagonal lattice is represented using triangular primitive vectors:

```text
a1 = (P, 0)

a2 = (P/2, sqrt(3)P/2)
```

The corresponding reciprocal lattice is therefore different from the square lattice.

This distinction is important because the diffraction orders and their propagation conditions depend on the reciprocal lattice.

---

## 6. RCWA diffraction orders

For a periodic structure, the outgoing field can be separated into diffraction orders:

```text
(m,n) = (0,0)
(m,n) = (1,0)
(m,n) = (-1,0)
(m,n) = (0,1)
(m,n) = (0,-1)
(m,n) = (1,1)
...
```

Each order corresponds to a different outgoing wavevector.

The in-plane wavevector of an order is:

```text
k_parallel(m,n)
=
k_parallel,incident
+
m*b1
+
n*b2
```

where `b1` and `b2` are reciprocal-lattice vectors.

The simulator identifies which diffraction orders are physically propagating and calculates their optical response.

Evanescent orders are not treated as propagating output beams.

---

## 7. Diffraction efficiency

For each propagating order, the simulator calculates quantities such as:

```text
R(m,n)
T(m,n)
```

where:

* `R(m,n)` = reflected diffraction efficiency
* `T(m,n)` = transmitted diffraction efficiency

The total reflected and transmitted power are obtained by summing over propagating orders:

```text
R_total = Σ R(m,n)

T_total = Σ T(m,n)
```

For a passive structure:

```text
R_total + T_total + A ≈ 1
```

where `A` is the absorbed power.

---

## 8. Optical phase

The simulator also extracts the complex diffraction amplitudes.

For transmission:

```text
t(m,n) = |t(m,n)| exp(i φ(m,n))
```

The optical phase is:

```text
φ(m,n) = arg(t(m,n))
```

This is fundamentally different from diffraction efficiency.

Diffraction efficiency tells us **how much optical power** is present in a particular order.

The complex amplitude additionally tells us the **phase** of that optical field.

Therefore, the program should not calculate phase from:

```python
np.angle(T_mn)
```

if `T_mn` is already a power quantity.

Instead, phase must be extracted from the underlying complex RCWA amplitude.

---

## 9. All diffraction-order phases

The simulator is designed to retain the phase information for all propagating diffraction orders.

For example:

```text
Order       Transmission       Phase
----------------------------------------
(0,0)          82%              137°
(1,0)           3%               82°
(-1,0)          3%              -82°
(0,1)            3%               91°
(0,-1)           3%              -91°
...
```

This allows the full optical transformation of the periodic structure to be investigated.

The zeroth order is particularly important for many transmissive metalens designs, but higher diffraction orders should not be discarded because they may carry significant optical power.

---

## 10. Why phase matters for a metalens

The purpose of a metalens is generally not to maximize absorption.

Instead, a metalens uses nanostructures to control the optical wavefront.

A desired focusing phase profile can be written as:

```text
φ(x,y)
```

Each nanostructure provides a local phase shift.

Conceptually:

```text
Nanorod geometry
       ↓
RCWA
       ↓
Transmission amplitude + phase
       ↓
Choose geometry for desired phase
       ↓
Spatial phase distribution
       ↓
Metalens
       ↓
Focused optical wave
```

Therefore, a useful unit cell should ideally provide:

* High transmission
* Low absorption
* Low unwanted reflection
* A broad range of controllable optical phase

---

## 11. Metalens unit-cell library

One of the main purposes of this simulator is to generate a unit-cell optical-response library.

For a fixed wavelength, for example:

```text
λ = 633 nm
```

the rod diameter can be swept:

```text
50 nm
60 nm
70 nm
80 nm
...
200 nm
```

For every geometry, the simulator can calculate:

```text
Diameter
T(m,n)
Phase(m,n)
```

This produces a mapping such as:

```text
rod diameter
      ↓
RCWA
      ↓
┌────────────────────────┐
│ transmission amplitude │
│ optical phase          │
│ diffraction efficiency │
└────────────────────────┘
```

The resulting data can later be used as a lookup table for metalens design.

For example:

```text
Desired phase → Nanorod diameter
```

---

## 12. Phase versus wavelength

The simulator can also calculate:

```text
phase (degrees) vs wavelength
```

This shows how the optical phase response of a particular nanorod changes with wavelength.

This is useful for studying:

* Optical dispersion
* Resonances
* Wavelength-dependent phase response
* Chromatic effects
* Plasmonic behavior

A phase feature around a particular wavelength should be interpreted together with transmission, reflection, absorption, and diffraction-order data.

---

## 13. Phase versus nanorod diameter

For metalens design, a particularly useful simulation is:

```text
Fixed wavelength
        ↓
Sweep rod diameter
        ↓
Calculate phase for every geometry
```

For example:

```text
phase
  ↑
360°|                  ●
    |              ●
270°|          ●
    |       ●
180°|    ●
    | ●
 90°|
    |
  0°+--------------------------→
       rod diameter
```

The goal is to determine whether the available nanorod geometries provide a sufficiently large phase range while maintaining high transmission.

A useful design region may look like:

```text
Phase coverage:       ~0–360°
Transmission:         high
Absorption:            low
Unwanted diffraction: low
```

---

## 14. FAST and ACCURATE presets

The code provides presets for balancing computational speed and numerical accuracy.

### FAST

Example:

```python
FAST_PRESET = dict(
    wavelength_min=550e-9,
    wavelength_max=700e-9,
    wavelength_step=25e-9,
    harmonics_xy=(5, 5),
    grid_nx=61,
    grid_ny=61,
    out_prefix="rcwa_fast_test",
)
```

This is useful for:

* Testing
* Debugging
* Quickly checking trends
* Exploring geometry

### ACCURATE

Example:

```python
ACCURATE_PRESET = dict(
    wavelength_min=450e-9,
    wavelength_max=850e-9,
    wavelength_step=5e-9,
    harmonics_xy=(11, 11),
    grid_nx=121,
    grid_ny=121,
    out_prefix="rcwa_accurate",
)
```

This is intended for more reliable final calculations.

Higher Fourier harmonics and finer geometry discretization generally increase computational cost but can improve numerical convergence.

---

## 15. Example simulation

A simple square-lattice simulation:

```python
res = simulate(
    lattice="square",
    rod_height=150e-9,
    rod_diameter=100e-9,
    period=500e-9,
    **FAST_PRESET
)
```

A hexagonal-lattice simulation:

```python
res = simulate(
    lattice="hexagonal",
    rod_height=150e-9,
    rod_diameter=100e-9,
    period=500e-9,
    **FAST_PRESET
)
```

---

## 16. Example output

The simulation returns a result dictionary containing numerical data and generated files.

For example:

```python
print(res["files"])
```

The output may include:

```text
diffraction response data
total response plot
individual diffraction-order data
phase data
zeroth-order phase data
```

The exact filenames depend on the selected `out_prefix`.

---

## 17. Energy conservation

The simulator checks:

```text
R_total + T_total + A ≈ 1
```

This is an important numerical sanity check.

A decrease in:

```text
R_total + T_total
```

can indicate increased absorption if all propagating diffraction orders are included.

However, a decrease in:

```text
R(0,0) + T(0,0)
```

does not necessarily mean increased absorption.

Power may instead have been redistributed into higher diffraction orders.

Therefore, total response and individual diffraction-order responses should be examined together.

---

## 18. Recommended workflow

For exploratory simulations:

```text
1. Choose lattice
       ↓
2. Choose rod height
       ↓
3. Choose rod diameter
       ↓
4. Run FAST simulation
       ↓
5. Inspect diffraction response
       ↓
6. Inspect absorption
       ↓
7. Inspect individual diffraction orders
       ↓
8. Inspect optical phase
```

For metalens development:

```text
1. Choose target wavelength
       ↓
2. Sweep rod diameter
       ↓
3. Calculate transmission + phase
       ↓
4. Build unit-cell library
       ↓
5. Identify high-transmission geometries
       ↓
6. Map desired phase → rod geometry
       ↓
7. Construct metalens phase profile
       ↓
8. Perform full-device simulation
```

---

## 19. Important interpretation

The simulator is not simply looking for the nanorod that absorbs the most light.

For a transmissive metalens, the desired behavior is generally closer to:

```text
High useful transmission
        +
Controlled optical phase
        +
Low absorption
        +
Low unwanted diffraction
```

The key design relationship is:

```text
nanorod geometry
        ↓
complex optical response
        ↓
amplitude + phase
        ↓
wavefront control
```

This is the connection between the RCWA unit-cell simulation and a practical metalens.

---

## 20. Limitations

RCWA is particularly useful for periodic structures and unit-cell analysis.

However, a real metalens is generally not an infinite periodic structure. Its nanostructure geometry changes across the lens.

Therefore, the unit-cell RCWA results should be understood as a **local periodic approximation** or unit-cell library.

The final metalens should ideally be validated using a full-wave method capable of treating the complete finite device, especially when:

* The lens is small
* The phase changes rapidly between neighboring cells
* Neighboring nanostructures strongly interact
* The local periodic approximation breaks down
* Edge effects become important

---

## 21. Project goal

The ultimate goal of this project is to create a practical workflow:

```text
                    RCWA
                      │
                      ▼
             Unit-cell simulation
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
   Diffraction response      Optical phase
          │                       │
          └───────────┬───────────┘
                      ▼
             Unit-cell library
                      │
                      ▼
             Metalens design
                      │
                      ▼
             Wavefront control
                      │
                      ▼
                   Focusing
```

The RCWA simulator therefore serves as the electromagnetic engine for understanding and designing the nanostructured optical elements that make up a metalens.
