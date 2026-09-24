---
layout: post
title:  "JackSolver: Hartree-Fock in Java"
date:   2012-12-10
excerpt: "A restricted Hartree-Fock solver written in Java, ported from the Fortran IV listing in the back of Szabo and Ostlund."
---

[JackSolver](https://github.com/AndrewKL/JackSolver) is a small restricted
Hartree-Fock solver. Give it the positions and atomic numbers of a set of
atoms and a total electron count, and it runs a self-consistent field
calculation and reports the electronic, nuclear repulsion and total energies
of the molecule.

It started as a port of the Fortran IV program in Appendix B of Szabo and
Ostlund's *Modern Quantum Chemistry*. The `step #N` comments in the solver
follow the SCF procedure on page 146 of that book, which is the honest way to
describe it: the physics is theirs, the code is mine. I was advised by Jason
A. C. Clyburne, with additional help from Cory Pye.

The shape of the calculation is the part worth knowing, because it is the
same shape in every serious quantum chemistry package. Put a contracted
Gaussian 1s orbital on each atom. Compute the one-electron integrals — the
overlap matrix, the kinetic energy, the nuclear attraction — and add the last
two to get the core Hamiltonian. Diagonalize the overlap matrix to build a
transformation that orthogonalizes the basis, because the atomic orbitals you
started with are not orthogonal and nothing downstream wants to deal with
that.

Then the expensive part. Every two-electron integral is a four-index object,
so the count grows as the fourth power of the basis size, and you need all of
them before the iteration can start. JackSolver precomputes them across a
thread pool sized to the number of cores, which is the one place where
writing this in Java instead of Fortran paid for itself.

The self-consistent loop is the idea the whole method rests on. The Fock
matrix depends on the electron density, and the density is what you are
trying to find, so you guess, solve, and use the answer to build a better
guess. JackSolver starts from a zero density matrix, then each pass builds
the Fock matrix, transforms and diagonalizes it, and forms a new density —
mixing 90% of the new one with 10% of the old, because taking the new one
outright makes it oscillate instead of settle. It stops when the RMS change
in the density falls below `1e-4`.

Everything is in atomic units: bohr for distance, hartree for energy.

The built-in example reproduces the Szabo and Ostlund HeH⁺ calculation at a
bond length of 1.4632 bohr, and should print a total energy of about −2.8607
hartree. That number is how you know the thing works — it is the same one in
the back of the book.

The code is on GitHub: [AndrewKL/JackSolver](https://github.com/AndrewKL/JackSolver).
