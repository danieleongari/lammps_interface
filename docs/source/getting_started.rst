Getting Started
===============

Create the molecular topology for an empty framework
----------------------------------------------------

The most basic example is to create the molecular topology for IRMOF-1, form its CIF file contained in the
test_structure directory. The CIF file contains the information about the unit cell dimensions and the atomic position.
::

  _cell_length_a          25.8320
  _cell_length_b          25.8320
  _cell_length_c          25.8320
  loop_
  _atom_site_label
  _atom_site_type_symbol
  _atom_site_fract_x
  _atom_site_fract_y
  _atom_site_fract_z
  O30 O 0.13400 0.28190 0.21810
  C31 C 0.11130 0.25000 0.25000
  C32 C 0.05380 0.25000 0.25000
  ...

Note that lammps-interface is not able to read the symmetry information. Therefore, only CIF files
with no implicit symmetries (i.e., P1) can be used. If the CIF is not P1, it need to be converted before running
lammps-interface, for example using ASE or pymatgen.

The basic command:
::

  lammps-interface IRMOF-1.cif

creates two inputs files for LAMMPS, in.IRMOF-1 and data.IRMOF-1. These can be submitted for a LAMMPS calculation::

  lmp -i in.IRMOF-1

With this command LAMMPS will parse the molecular topology information and stop, since the in.IRMOF-1 file does not
contains any instruction for simulations (i.e, "run" command). However, if this command terminate with an error like:
::

  ERROR: Unknown angle style fourier (../force.cpp:471)

it means that that specific bond/angle/dihedral/improper/... style is not included in the version of LAMMPS. To solve
the problem one has to build LAMMPS again including the requested user packages (https://lammps.sandia.gov/doc/Packages_user.html).
For example the fourier angle style needed for UFF is contained in the package USER-MISC.

Select specific Force Field and settings
----------------------------------------

By default, lammps-inteface uses the Universal Force Field (UFF) to generate the molecular topology.
Other force fields such as 'Dreiding', 'UFF4MOFs', 'BTW_FF' and 'Dubbeldam' can be selected:
::

  lammps-interface IRMOF-1.cif -ff Dreiding

Moreover, considering the periodic boundary conditions, the cutoff threshold for non-covalent interactions needs to be
less than half the cell dimension (more technically, less than half the minimal perpendicular width of the unit cell).
If this condition is not respected, lammps-interface will automatically re-size the unit cell to meet this condition.

In this case, the default value of 12.5 Angsroms, is appropriate for IRMOF-1. However, increasing the cutoff value
beyond 12.916 Angstoms (half the lenght of this cubic unit cell), the structure will be re-sized.
::

  $ lammps-interface IRMOF-1.cif --cutoff 13

  WARNING: unit cell is not large enough to support a non-bonded cutoff of 13.00 Angstroms.
  Re-sizing to a 2 x 2 x 2 supercell.

Finally, considering that lammps-interface was primary designed to be used for Metal Organic Frameworks (MOFs),
the ``--fix-metal`` functionality need to be introduced, as it is essential for these materials. MOFs building units contains
coordination chemistry geometry for which traditional Force Fields such as UFF and Dreiding were not parametrized.
The option ``--fix-metal`` forces these metal complexes to stay in their initial configuration, e.g, tetrahedral for IRMOF-1's Zn or square
piramidal for the common Cu-paddlewheel.
When this option is activated, lammps-inteface search for the metals in the structure and overwrites the bonds and angles
equilibrium distances with their neighbor. In this case, when using
::

  lammps-interface IRMOF-1.cif --fix-metal

the following parameters in data.IRMOF-1
::

  Bond Coeffs

  2   160.291409     1.840266  # O_R Zn3+2
  6   172.603617     1.795426  # Zn3+2 O_2

  Angle Coeffs

  7   fourier   179.021796     0.343737     0.374972     0.281246 # O_R Zn3+2 O_2
  8   fourier   172.607259     0.343737     0.374972     0.281246 # O_R Zn3+2 O_R

are corrected in
::

  Bond Coeffs

  2   160.291409     1.921883  # O_R Zn3+2
  6   172.603617     1.941817  # Zn3+2 O_2

  Angle Coeffs

  7   fourier   138.211958     0.380438     0.451845     0.293479 # O_R Zn3+2 O_2
  8   fourier   164.888123     0.312707     0.300998     0.270902 # O_R Zn3+2 O_R

Note that lammps-interface append as a comment at the end of the line a list of the atom types involved in the bond/angle/dihedral/improper/pair,
for an easier understanding of the generated molecular topology. More details in the "Inner working" section.

Extra options ``--h-bonding`` and ``--dreid-bond-type`` allow to select different flavors of the Dreiding force field.

Run standard simulations on a bare framework
--------------------------------------------

Lammps-interface allows to automatically generate the inputs to run basic calculations in LAMMPS.

* ``--nvt``, NVT canonical simulation. The equilibration uses Langevin thermostat, while the production uses the Nosé-Hoover thermostat. Options:

    * ``--temperature`` (default: 298K)
    * ``--equilibration-steps`` (default: 200,000)
    * ``--production-steps`` (default: 200,000)

* ``--npt``, NPT isothermal-isobaric simulation. Options:

    * ``--temperature`` (default: 298K)
    * ``--pressure`` (default: 298K)
    * ``--equilibration-steps`` (default: 200,000)
    * ``--production-steps`` (default: 200,000)

* ``--minimize``, cell and geometry optimization.

* ``--bulk-moduli``, energy vs volume calculation. Options:

    * ``--iter-count`` (default: 10)
    * ``--max-deviation`` (default: 0.01)

* ``--thermal-scaling``, thermal ramp from 0 K using the Langevin thermostat. Options:

    * ``--iter-count`` (default: 10)
    * ``--temperature`` (default: 298K)

Run simulation for a loaded Frameworks
--------------------------------------

In order to run simulations with framework loaded with adsorbed molecules, two ways are possible: from a loaded CIF or
from an empty framework where molecules are later deposited in LAMMPS.

The first alternative is to use a CIF that already contains both the framework and the molecules.
This can be obtained, for example, from a GCMC
calculation done with another coed (e.g., Raspa) to obtain the number of adsorbates at a certain temperature/pressure
conditions and with an equilibrated geometry. Lammps-interface will recognize that the CIF contains molecules that are
not connected to the framework and treat them separately. Indeed, a different force field can be use for these molecule,
using the ``--molecule-ff`` option. As an example we compute the molecular topology of IRMOF-1_load, which is loaded with
ten CH4, two N2 and one CO2 molecules. We specify Dreiding force field for the framework and UFF for the molecules.::

   $ lammps-interface IRMOF-1_loaded.cif -ff Dreiding --molecule-ff UFF

   No bonds reported in cif file - computing bonding..
   Molecules found in the framework, separating.
   Files created!

Note that lammps-interface automatically recognize similar molecules and assign them to different groups. In this
example, group 1 is CH4, 2 is N2 and 3 is CO2. As in the bare framework example, the framework is labeled as "fram".
::

  #### Atom Groupings ####
  group           1        id   425:474
  group           1-1      id   425:429
  group           1-2      id   430:434
  group           1-3      id   435:439
  group           1-4      id   440:444
  group           1-5      id   445:449
  group           1-6      id   450:454
  group           1-7      id   455:459
  group           1-8      id   460:464
  group           1-9      id   465:469
  group           1-10     id   470:474
  group           2        id   475:478
  group           2-1      id   475:476
  group           2-2      id   477:478
  group           3        id   479:481
  group           fram     id   1:424
  #### END Atom Groupings ####

At the moment, no molecules-specific force field (such as TraPPE) are available, and the only two meaningful
force field available for the parametrization of gas/liquid molecules are UFF and Dreiding.

Note: if lammps-interface needs to re-size the unit cell because of a too large cutoff, it will ask the user if
he wants to replicate also the loaded molecules in the extended cell.
::

  $ lammps-interface IRMOF-1_loaded.cif --molecule-ff UFF --cutoff 13

  No bonds reported in cif file - computing bonding..
  Molecules found in the framework, separating.
  WARNING: unit cell is not large enough to support a non-bonded cutoff of 13.00 Angstroms.
  Re-sizing to a 2 x 2 x 2 supercell.
  Would you like to replicate molceule 1 with atoms (C, H, H, H, H) in the supercell? [y/n]:
  Would you like to replicate molceule 2 with atoms (N, N) in the supercell? [y/n]:
  Would you like to replicate molceule 3 with atoms (C, O, O) in the supercell? [y/n]:


::

  !!!!!!!!!! HERE THERE IS A BUG TO SOLVE: the grouping gets printed wrong
  #### Atom Groupings ####
  group           1        id   425:474 906:955 1387:1436 1868:1917 2349:2398 2830:2879 3311:3360 3792:3841
  group           1-1      id   425:429 906:910 1387:1391 1868:1872 2349:2353 2830:2834 3311:3315 3792:3796
  group           1-2      id   906:910
  group           1-3      id   1387:1391
  group           1-4      id   1868:1872
  group           1-5      id   2349:2353
  group           1-6      id   2830:2834
  group           1-7      id   3311:3315
  group           1-8      id   3792:3796
  group           1-9      id   430:434 911:915 1392:1396 1873:1877 2354:2358 2835:2839 3316:3320 3797:3801
  group           1-10     id   911:915
  group           1-11     id   1392:1396
  group           1-12     id   1873:1877
  group           1-13     id   2354:2358

The second alternative is to use a CIF file that only contains the framework, and the option --insert-molecule to
insert specific molecules, whose force field has been included in the program. The limit of this procedure is that
it is only possible to specify the number of molecules. (Future development will include compatibility with the GCMC
protocol implemented in LAMMPS, to allow loading the system at a specific pressure/temperature. However, consider that
for larger molecules, the Configurational Biased Monte Carlo (CBMC) protocol as not been implemented in LAMMPS yet.)


Other periodic systems
----------------------

Lammps-interface was originally designed to model empty and loaded frameworks (primarily MOFs).
However its use can be extended to other periodic systems.

* homogeneous gas/liquid mixtures
* molecular crystals
* layered materials (graphite?)
