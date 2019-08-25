Inner working of the code
=========================

When running lammps-interface, these are essential steps:

* parsing of the CIF file
* recognition of the frameworks and molecules in the systems
* assignment of atom types and bond connectivity
* (eventually) re-sizing the unit cell and updating the atom types
* assignment of bonds/angles/dihedrals/impropers
* printing of the LAMMPS inputs in. and data. files

It is necessary to understand all the steps to verify that a proper molecular topology is produced by the program.

Printing a debug CIF/PDB output
-------------------------------

In order to better check the result of lammps_interface in re-sizing, assigning atom types and bond connectivity,
the option ``-o`` allows to print the results of the analysis in a CIF file instead of LAMMPS's in. and data. files.
The produced file .debug.cif contains:

* the re-sized system (i.e., with expanded unit cell and multiplied atoms)
* the atom types (as ``_atom_site_description``)
* the partial charge if assigned by the force field or read from the input CIF (as ``_atom_type_partial_charge``)
* the atomic bonds, labeled by "CCDC bond type"

The option ``-p``, similarly prints these information in a PDB file.
