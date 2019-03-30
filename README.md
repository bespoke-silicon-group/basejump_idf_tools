# BSG IDF Tools

This is a place where utility scripts and other tools will live for using the
[IDEA Dimensionless Floorplan (IDF)](https://docs.google.com/document/d/1rIB81hEOSNs2F1pIF6WbIwSNe-8HbfEtZRlaMFKdhKk/edit)
format. Below are the currently available tools.

## General Setup and Makefile.include

In the root of the repository is a `Makefile.include` file. This file can be included in other Makefile flows
that need to use the IDF tools. Including `Makefile.include` will give you access to the `$ make tools` target. This
target will download and build any dependencies required to run the IDF tools. Right now, IDF tools depend on
[pypy](https://pypy.org/) and [pyparsing](https://pypi.org/project/pyparsing/). If you would like to simply
build the tools without using another makefile, you can run:

```
$ make -f Makefile.include tools
```

The tools will be placed in the `tools` directory in the root of the repository.

## IDF to DEF Converter

IDF to DEF converter is a python script that will take an [IDEA Dimensionless Floorplan (IDF)](https://docs.google.com/document/d/1rIB81hEOSNs2F1pIF6WbIwSNe-8HbfEtZRlaMFKdhKk/edit)
file and convert it to a Design Exchange Format (DEF) file. In doing so, the script will also
re-dimensionalize the floorplan with process specific units. The IDF to DEF converter can be
found in the `idf_to_def/` directory with the main script being `idf_to_def.py`. The python
interpreter needs to have [pyparsing](https://pypi.org/project/pyparsing/) installed and we
also suggest using the [pypy](https://pypy.org/) python interpreter for speed (particularly
with file IO). You can use the `$ make tools` target which is part of the `Makefile.include`
file in the root of this repository. For more information, please read the
[General Setup and Makefile.include section](#general-setup-and-makefileinclude).

As part of the re-dimensionalizing process, the IDF to DEF converter requires a technology
file (.tf) from the PDK. This will give the script information about the metal stack and
the unit tile which it will use to add dimension to the values before creating the DEF file.

By default, the lowest metal layer is assumed to have a preferred routing direction that is
vertical. If this is not true, you can use the `-swap_metal` flag to change the preferred routing
direction of the first metal layer to be horizontal. The preferred routing direction for each
subsequent layer is assumed to flip.

The metal layer stack is determined by parsing the technology file (.tf), however it may be
that there are top-metal layers that you don't really want to consider as a normal routing
metal layer (e.g. RDL layers). If you would like to limit the top metal layer, use the
`-max_metal_layer_num` flag and specify the number of the max metal layer (Note: the
bottom metal layer is index 1). 

Once complete, the DEF files will be created in the current working directory the script
was launched from. There will be 1 DEF file for each design in the _design_ section of the
IDF file, and the name of DEF file will be _<design_name>.def_.

### Usage

```man
usage: idf_to_def.py [-h] -idf file -tf file [-swap_metal] [-max_metal_layer num]

This script takes an IDF json floorplanning file and converts it to a DEF
file. IDF files are "dimensionless" therefore the user must provide the
technology file (.tf) for the target process.

optional arguments:
  -h, --help            show this help message and exit
  -idf file             Input IDF .json file
  -tf file              Input technology file (.tf)
  -swap_metal           Changes the starting orientation of the metal stack.
                        By default, the first metal layer is assumed to be
                        horizontal. Use this flag to set the first metal layer
                        to be vertical.
  -max_metal_layer num  Used to specify the max metal layer number
```
