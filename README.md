# Shor's Algorithm for the Discrete Logarithm Problem

A Q# (Microsoft Quantum) implementation of Shor's algorithm applied to the
discrete logarithm problem: given a prime `p`, a generator `g` and a target
`h`, find `x` such that `g^x ≡ h (mod p)`.

Developed by Florian Fogliani, Robin Paulik and Alexis Le Gall as part of the
Quantum major at EPITA.

## References

The theory behind the implementation is not repeated here. See the documents
included in the repository:

- `Paper.pdf` — the reference paper the implementation follows.
- `Slides.pdf` — the accompanying presentation.

## Project structure

All the source lives in `src/`. The pipeline goes from quantum modular
arithmetic, up to phase estimation, down to the classical post-processing that
recovers `x`.

### `src/UGate.qs` — quantum modular arithmetic

The lowest layer. Builds modular arithmetic operations on qubit registers:

- `AddConstantModulo` — adds a classical constant to a register modulo `N`.
- `MultiplyConstantModulo` — multiplies a register by a classical constant
  modulo `N`, using an auxiliary register and un-computing it through the
  modular inverse of the constant.

The file also contains standalone sanity-check operations
(`TestBasicMultiplication`, `TestMulFermat`, `TestAddCycle`, ...) that exercise
the arithmetic directly.

### `src/PhaseEstimation.qs` — iterative phase estimation

- `ApplyUGate` — connects the modular exponentiation to the estimation by
  applying `MultiplyConstantModulo` with `g^power mod N`.
- `EstimateFrequency` — an iterative (semiclassical) phase estimation that
  reuses a single control qubit with `R1Frac` feedback. It is run twice, once
  driven by `g` and once by `h`, and returns the two measured frequencies.

### `src/ClassicalPart.qs` — classical post-processing and solver

- `LogFromFrequencies` — turns the two frequencies into a candidate `x` using
  continued-fraction convergents and a modular inverse (Section II.B of the
  paper).
- `VerifyDiscreteLog` — checks a candidate by recomputing `g^x mod p`.
- `EstimateDiscreteLog` — glue between the quantum estimation and the classical
  recovery: it collects the frequencies and passes them on.
- `DiscreteLog` — the main solver. It runs a repeat-until-success loop (up to
  100 attempts), keeping the first candidate that verifies.

### `src/ClassicalPartTests.qs` — end-to-end tests

A test suite running the full algorithm on many `(p, g, x)` cases, from `p = 7`
up to `p = 59`. `RunAllTests` executes every case in sequence.

## Running it

The project is run from Visual Studio Code.

1. Install the [Azure Quantum Development Kit (Q#) extension](https://marketplace.visualstudio.com/items?itemName=quantum.qsharp-lang-vscode).
2. Open the repository folder in VS Code.
3. Open the file containing the operation you want to run.
4. Use the `Run` code lens shown above an operation to execute it on the
   built-in simulator.

To run the full test suite, run `RunAllTests` in `src/ClassicalPartTests.qs`.
To run a single case, run any of the individual `TestDLog_*` operations.
