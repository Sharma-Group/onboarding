# Installing Orca with MPI Support

## What is ORCA?
A quantum chemistry packages with the methods from semi-imperical to single-reference to multireference PT. 

Their [official description](https://sites.google.com/site/orcainputlibrary/home): 
> ORCA is an ab initio, DFT, and semi-empirical SCF-MO package developed by Frank Neese et al. at the Max Planck Institut für Kohlenforschung.

You can find the complete manual [here](https://cec.mpg.de/fileadmin/media/Forschung/ORCA/orca_manual_4_0_1.pdf).

---
## Legal Info (__READ CAREFULLY__)

> Read this carefully if you plan to use ORCA in published work especially if the project is a collaboration. Make sure you know __where__ the funding in your project comes from so you do not violate the license agreement.

1. DEFINITIONS
   1. “ACADEMIA” refers to all schools, colleges, universities and the Institutes of the MPG. Expressly excluded from this term are institutions with a connection to the military.
   2. “ACADEMIC PURPOSE” refers to research and teaching in ACADEMIA. Expressly excluded from this term are research and development for commercial purposes, research and development in cooperation or other collaboration with or sponsored by a for-profit organization, and research and development for a for-profit organization.
   3. “SOFTWARE” refers to the Software ORCA, version 4.0 or later, including its documentation.
   4. “PRIVATE USE” refers to the usage of SOFTWARE by a single, private person for private, non-commercial, not work-related research only. Expressly excluded from PRIVATE USE is use for a for-profit, a non-profit, governmental or any other type of organization. Expressly excluded from PRIVATE USE is a use that results or contributes to any commercial activity. Expressly excluded from PRIVATE USE are persons whose work could potentially benefit from or could potentially be influenced in any way through the usage of SOFTWARE.
   
2. CONDITIONS OF USE

   1. SOFTWARE may be used exclusively in ACADEMIA for ACADEMIC PURPOSES and for PRIVATE USE.

   2. SOFTWARE is an academic program code in development. The MPI expressly points out that it is possible that SOFTWARE will not have standard properties, or may not be suitable for use or fit for a particular purpose. If SOFTWARE is being used by a research group, the research group leader guarantees compliance with the rights and obligations arising from this Agreement and shall be available to the MPI as a point of contact.

---
## Install

Make sure to get ORCA first because the version of ORCA you get will determine what version of OpenMPI you need.

The official setup docs from ORCA are [here](https://sites.google.com/site/orcainputlibrary/setting-up-orca) and from OpenMPI are [here](https://www.open-mpi.org/faq/?category=building).


1. Start by registering with [ORCA](https://cec.mpg.de/orcadownload/) and getting a copy of the appropriate binary.

1. Unpack the compressed file in the location of your choice

```bash
tar -I zstd -xvf orca_4_0_1_2_linux_x86-64_openmpi202.tar.zst
```

> Note: If we didn't want MPI we could add ORCA to our environment PATH and LD_LIBRARY_PATH variables and be done. 

3. Download the correct version of [OpenMPI](https://www.open-mpi.org/software/ompi/v4.0/) ([2.0.2](https://www.open-mpi.org/software/ompi/v2.0/) for the version of ORCA in this example)

4. Unpack OpenMPI (to keep everything together I've make a directory within the orcadirectory)
```bash
mkdir orca_4_0_1_2_linux_x86-64_openmpi202 && cd orca_4_0_1_2_linux_x86-64_openmpi202
mkdir openmpi && cd openmpi
wget https://download.open-mpi.org/release/open-mpi/v2.0/openmpi-2.0.2.tar.gz
tar -xzvf openmpi-2.0.2.tar.gz
cd openmpi-2.0.2
```
5.  Configure your OpenMPI build. Here I'm loading the gcc module to build OpenMPI and specifying where we want to install OpenMPI with the `prefix` flag.
```bash
ml gcc
./configure --prefix=/projects/jasm3285/apps/orca_4_0_1_2_linux_x86-64_openmpi202/openmpi/.
...
...
```
6. Make OpenMPI and install.
```bash
make all install
```

That's it for installation, but before we can run any calculations we have to set up our environment variables. 

## Setting up environment

I would strongly recommend ___not___ putting the following in `~/.bash_profile` or `~/.bashrc` and only put it in your bash submit script or any script you use that's specific to the job at hand.

```bash
ml gcc # Load GCC (or whatever you used to compile OpenMPI)

export ORCA_ROOT=/home/jasm3285/projects/apps/orca_4_0_1_2_linux_x86-64_openmpi202/
export OPENMPI_ROOT=$ORCA_ROOT/openmpi

# OpenMPI
export PATH=$OPENMPI_ROOT/bin:$PATH
export LD_LIBRARY_PATH=$OPENMPI_ROOT/lib:$LD_LIBRARY_PATH

# ORCA
export PATH=$ORCA_ROOT:$PATH
export LD_LIBRARY_PATH=$ORCA_ROOT:$LD_LIBRARY_PATH
export ORCA=$ORCA_ROOT/orca
```

Here is an example bash file:
```bash
#!/bin/bash

#
# Set up ORCA ENV variables
#
ml gcc # Load GCC (or whatever you used to compile OpenMPI)

ORCA_ROOT=/home/jasm3285/projects/apps/orca_4_0_1_2_linux_x86-64_openmpi202/
OPENMPI_ROOT=$ORCA_ROOT/openmpi

# OpenMPI
export PATH=$OPENMPI_ROOT/bin:$PATH
export LD_LIBRARY_PATH=$OPENMPI_ROOT/lib:$LD_LIBRARY_PATH

# ORCA
export PATH=$ORCA_ROOT:$PATH
export LD_LIBRARY_PATH=$ORCA_ROOT:$LD_LIBRARY_PATH
ORCA=$ORCA_ROOT/orca

#
# Run Calculation
#
JOB_NAME=test

$ORCA $JOB_NAME.inp > _orca.out


#
# Clean up
#


for D in _orca_files
do
if [ -d $D ] 
then
    echo "Directory $D exists." 
else
    echo "Directory $D does not exist, making directory..."
    mkdir $D
fi
done

mv $JOB_NAME.* _orca_files/.
mv ${JOB_NAME}_property.txt _orca_files/.
mv _orca_files/$JOB_NAME.inp .
```



Here's a sample calculation to make sure everything is working:

```
# test.inp
!def2-svp nofrozencore
%pal
nprocs 4
end
%casscf nel     8
	norb    8
	nroots  3
	mult    1
	bweight 1
	NEVPT2 SC # SC for the strongly contracted NEVTP2
end

* xyz 0 1
N 0.0 0.0 0.0
N 0.0 0.0 1.09768
*

```