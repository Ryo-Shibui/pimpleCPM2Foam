# pimpleCPM2Foam

`pimpleCPM2Foam` is a compact custom OpenFOAM solver for transient incompressible PIMPLE flow with electrostatic potential, bipolar ion drift-diffusion, ion recombination, and electric body-force coupling. Compared with `pimpleCPMFoam`, it is a stricter CPM-focused variant: it requires a patch named `CPM` and updates the `phiE` value on that patch from the net deposited current.

This solver has been checked only with OpenFOAM-v2406 through OpenFOAM-v2506.

## Features

- Based on `pimpleFoam` for incompressible transient flow.
- Solves `phiE`, reconstructs `E`, and advances positive/negative ion densities `nP` and `nN`.
- Computes drift and diffusion current at the `CPM` patch and updates the fixed-value electric potential using `Ccap`.

## Compilation

Load an OpenFOAM-v2406, v2412, or v2506 environment first:

```bash
source /path/to/OpenFOAM-v2506/etc/bashrc
```

Build from this directory:

```bash
wmake
```

The executable is written to:

```text
$FOAM_USER_APPBIN/pimpleCPM2Foam
```

To clean:

```bash
wclean
```

## Required Case Setup

Prepare a normal incompressible PIMPLE case and add:

- `0/U`
- `0/p`
- `0/phiE`
- `0/nP`
- `0/nN`

Optional read/write fields:

- `E`
- `rho`
- `phiEF`

Add the following physical properties in `constant/physicalProperties`:

```text
T       T       [0 0 0 1 0 0 0] 300;
rho0    rho0    [1 -3 0 0 0 0 0] 1.2;
nu      nu      [0 2 -1 0 0 0 0] 1.5e-05;
Ccap    Ccap    [-1 -2 4 0 0 2 0] 1e-12;
muP     muP     [-1 0 2 0 0 1 0] 1.5e-04;
muN     muN     [-1 0 2 0 0 1 0] 1.5e-04;
DP      DP      [0 2 -1 0 0 0 0] 1e-05;
DN      DN      [0 2 -1 0 0 0 0] 1e-05;
beta    beta    [0 3 -1 0 0 0 0] 1e-12;
```

The mesh must contain a boundary patch named `CPM`. This solver stops with a fatal error if that patch is absent. The `phiE` boundary condition on `CPM` should normally be `fixedValue`.

## Usage

Run it like a standard OpenFOAM solver:

```bash
pimpleCPM2Foam
```

For parallel cases:

```bash
decomposePar
mpirun -np <N> pimpleCPM2Foam -parallel
reconstructPar
```

At each time step the solver:

1. Solves the flow equations with the electric body force.
2. Solves the electrostatic potential and ion transport equations.
3. Calculates net drift/diffusion current on `CPM`.
4. Updates the `CPM` `phiE` value by `-deltaT*Inet/Ccap`.

## Notes

- Use `pimpleCPMFoam` if you want graceful handling of missing `CPM` or `walls` patches and ion-based time-step control.
- Use this solver when the case is centered on a single CPM/electrode patch and you want the stricter setup check.

