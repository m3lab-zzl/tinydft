# Tiny DFT

在学习完最简单的实空间网格基底后，该熟悉一下原子轨道相关的东西了，而且还有一堆作业可以强化代码能力

这是一个基于高斯轨道基的代码，公式部分可参考
<https://zh.wikipedia.org/wiki/%E9%AB%98%E6%96%AF%E8%BD%A8%E9%81%93>

Tiny DFT is a minimalistic atomic Density Functional Theory (DFT) code,
mainly for educational purposes. It only supports spherically symmetric
atoms and local exchange-correlation functionals (at the moment only
Dirac exchange).

The code is designed with the following criteria in mind:

-   It depends only on established scientific Python libraries:
    [numpy](https://www.numpy.org/), [scipy](https://www.scipy.org/),
    [matplotlib](https://matplotlib.org/) and (the lesser known)
    [autograd](https://github.com/HIPS/autograd/). The latter is a
    library for algorithmic differentiation, used to computed the
    analytic exchange(-correlation) potential and the grid
    transformation.
-   The numerical integration and differentiation algorithms should be
    precise enough to at least 6 significant digits for the total
    energy, but in many cases the numerical precision is better. (Some
    integrals over Gaussian basis functions are computed analytically.
    The pseudo-spectral method with Legendre polynomials is used for the
    Poisson solver.)
-   The total number of lines should be minimal and the source-code
    should be easy to understand, provided some background in DFT and
    spectral methods. As in most atomic DFT codes, the occupation
    numbers of the orbitals are all given the same value within one pair
    of angular and principal quantum number, to obtain a spherically
    symmetric density. The code only keeps track of the total number of
    electrons for each pair of quantum numbers.

## \"Installation\"

1)  Make sure you have the dependencies installed: Python 3 and fairly
    recent versions of [numpy](https://www.numpy.org/) (\>= 1.4.0),
    [scipy](https://www.scipy.org/) (\>=1.0.0),
    [matplotlib](https://matplotlib.org/) (\>= 2.2.4) and
    [autograd](https://github.com/HIPS/autograd/) (\>=1.2). In case of
    doubt, ask some help from your local Python guru. If you have Python
    3, you can always install or upgrade the other dependencies in your
    user account with pip:

    ``` bash
    python3 -m pip install numpy scipy autograd --upgrade
    ```

    Packages from your Linux distribution or the Conda package manager
    should also work.

2)  Download Tiny DFT. This can be done with your browser, after which
    you unpack the archive:
    <https://github.com/theochem/tinydft/archive/master.zip>. Or you can
    use git:

    ``` bash
    git clone https://github.com/theochem/tinydft.git
    cd tinydft
    ```

## Usage

To run a series of atomic DFT calculation, up to argon, just execute

``` 
python3 program_mendelejev.py
```

This generates some screen output with SCF convergence, energy
contributions and the figures `rho_z0*.png` with the radial electron
densities on a semi-log plot. To modify the settings for these
calculation, you can directly edit the source code.

When you make serious modifications to Tiny DFT, you can run the unit
tests to make sure the original features still work. For this, you first
need to install [pytest](https://pytest.org/).

``` bash
# Install pytest in case you don't have it yet.
python3 -m pip install pytest pytest-regressions pandas --upgrade
pytest
```

## Programming assignments

In order of increasing difficulty:

1)  Change `tinydft.py` to also make plots of the radial probability
    density, the occupied orbitals, the potentials (external, Kohn-Sham)
    with horizontal lines for the (higher but still negative) energy
    levels. Does this code reproduce the Rydberg spectral series well?
    (See
    <https://en.wikipedia.org/wiki/Hydrogen_spectral_series#Rydberg_formula>)

2)  Add a second unit test for the Poisson solver in `test_tinydft.py`.
    The current unit test checks if the Poisson solver can correctly
    compute the electrostatic potential of a spherical exponential
    density distribution (Slater-type). Add a similar test for a
    Gaussian density distribution. Use the `erf` function from
    `scipy.special` to compute the analytic result. See
    <https://docs.scipy.org/doc/scipy/reference/generated/scipy.special.erf.html#scipy.special.erf>

3)  At the moment, the DFT calculations assume spin-unpolarized
    densities, which mainly affects the exchange-correlation energy.
    Change the code to work with spin-polarized densities. For this,
    also the occupation numbers for each angular momentum and principal
    quantum number should be split into a spin-up and spin-down
    occupation number.

4)  Change the external potential to take into account the finite extent
    of the nucleus. You can use a Gaussian density distribution. Write a
    python script that combines isotope abundancies from NUBASE2016
    (<http://amdc.in2p3.fr/nubase/nubase2016.txt>) and nucleotide radii
    from ADNDT (<https://www-nds.iaea.org/radii/>) to build a table of
    abundance-averaged nuclear radii.

5)  Add a correlation energy density to the function `excfunction` and
    check if it improve the results in assignment (2). The following
    correlation functional has a good compromise between simplicity and
    accuracy: <https://aip.scitation.org/doi/10.1063/1.4958669> and
    <https://aip.scitation.org/doi/full/10.1063/1.4964758>

6)  The provided implementation has a rigid algorithm to assign
    occupation numbers using the Klechkowski rule. Replace this by an
    algorithm that just looks for all the relevant lowest-energy
    orbitals at every SCF iteration.

7)  Implement an SCF convergence test, which checks if the new Fock
    operator, in the basis of occupied orbitals from a previous
    iteration, is diagonal with orbital energies from the previous
    iteration on the diagonal

8)  Implement the zeroth-order regular approximation to the Dirac
    equation (ZORA). ZORA needs a pro-atomic Kohn-Sham potential as
    input, which remains fixed during the SCF cycle. Add an outer loop
    where the first iteration is without ZORA and subsequent iterations
    use the Kohn-Sham potential from the previous SCF loop as
    pro-density for ZORA. (To avoid that the density diverges at the
    nucleus, assignment 4 should be implemented first.)

    In ZORA, the following operator should be added to the Hamiltonian:

    ![t\_{ab} = \\int (\\nabla \\chi_a) (\\nabla \\chi_b) \\frac{v\_{KS}(\\mathbf{r})}{4/\\alpha\^2 - 2v\_{KS}(\\mathbf{r})} \\mathrm{d}\\mathbf{r}](zora.png){.align-center}

    where the first factors are the gradients of the basis functions
    (similar to the kinetic energy operator). The Kohn-Sham potential
    from the previous outer iteration can be used. The parameter alpha
    is the dimensionless inverse fine-structure constant, see
    <https://physics.nist.gov/cgi-bin/cuu/Value?alphinv> and
    <https://docs.scipy.org/doc/scipy/reference/constants.html>
    (`inverse fine-structure constant`). Before ZORA can be implemented,
    the formula needs to be worked out in spherical coordinates,
    separating it in a radial and an angular contribution.

9)  Extend the program to perform unrestricted Spin-polarized KS-DFT
    calculations. (Assignment 6 should done prior to this one.) In
    addition to the Aufbau rule, you now also have to implement the Hund
    rule. You also need to keep track of spin-up and spin-down orbitals.
    The original code uses the angular momentum quantum number, `angqn`
    as keys in the `eps_orbs_u` dictionary. Instead, you can now use
    `(angqn, spinqn)` keys.

10) Extend the program to support (fractional) Hartree-Fock exchange.

11) Extend the program to support (meta) generalized gradient
    functionals.

## Dictionary of variable names

The variable names are not always the shortest possible, e.g. `atnum`
instead of `z`, to make them more self-explaining and to comply with
good practices.

-   `alphas`: Gaussian exponents in basis functions
-   `atcharge`: Atomic charge
-   `atnum`: Atomic number
-   `angqn`: Angular momentum (or azimuthal) quantum number
-   `coeffs`: Expansion coefficients of a function in Gaussian
    primitives or Legendre polynomials.
-   `econf`: Electronic configuration
-   `energy_hartree`: Hartree energy, i.e. classical electron-electron
    repulsion.
-   `eps`: Orbital energies
-   `eps_orbs_u`: A list of tuples of (orbital energy, orbital
    coefficients). One tuple for each angular momentum quantum number.
    The orbital coefficients represent the radial solutions U = R/r.
-   `energy_xc`: Exchange-correlation energy
-   `exc`: Exchange-correlation energy density
-   `evals`: Eigenvalues
-   `evecs`: Eigenvectors
-   `ext`: Integrals for interaction with the external field (proton)
-   `fnvals`: Function values on grid points
-   `fock`: Fock operator
-   `iscf`: Current SCF iteration
-   `jxc`: Hartree-Exchange-Correlation operator
-   `kin_rad`: Radial kinetic energy integrals
-   `kin_ang`: Angular kinetic energy integrals
-   `maxangqn`: Maximum angular quantum number of the occupied orbitals
-   `nbasis`: Number of basis functions
-   `nelec`: Number of electrons
-   `nscf`: Number of SCF iterations
-   `occups`: Occupation numbers
-   `olp`: Overlap integrals
-   `orb_u`: Orbital divided by r: U = R/r
-   `orb_r`: Orbital: R = U\*r
-   `priqn`: Primary quantum numbers
-   `rho`: Electron density on grid points
-   `vhartree`: Hartree potential, i.e. minus classical electrostatic
    potential due to the electrons.
-   `vol`: Volume element in spherical coordinates
-   `vxc`: Exchange-correlation potential
