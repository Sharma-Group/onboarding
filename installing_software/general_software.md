# Installing Software
## Instructions
* The titles of all the software listed below are all hyperlinked to the installation pages. If the links don't seem to work a simple google search will get you to the same install page.
* If you have any questions during the installation process try Googling your problem. But if this fails feel free to email [Sandeep Sharma](sandeep.sharma@colorado.edu) or [James Smith](james.e.smith@colorado.edu).

_Table of Contents:_
- [Installing Software](#installing-software)
  - [Instructions](#instructions)
- [General Software](#general-software)
  - [Git](#git)
  - [Cmake](#cmake)
  - [Python 3.X](#python-3x)
  - [Eigen](#eigen)
  - [Boost](#boost)
- [Quantum Chemistry Software](#quantum-chemistry-software)
  - [PySCF](#pyscf)
  - [Dice](#dice)
  - [ORCA](#orca)
  - [Bagel](#bagel)
- [Visualizing Molecules](#visualizing-molecules)
  - [Jmol](#jmol)
  - [IQmol](#iqmol)
  - [NGLViewer](#nglviewer)
  - [Molview](#molview)
- [Misc. Software](#misc-software)
  - [sqa](#sqa)

---

# General Software

<!-- Fundamental Packages -->
## [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* You'll need `git` to download this repository and you may choose to use it to install some of the packages described below!



## [Cmake](https://cmake.org/download/)
* We'll use `cmake` to build `PySCF`, `Dice`, `Boost`, and some of the other packages we'll be using.



## [Python 3.X](https://docs.conda.io/en/latest/miniconda.html)

* We recommend using a package manager like [Anaconda](https://docs.conda.io/en/latest/miniconda.html) to download Python and its related packages.
* Regardless of what Python package manager you use, we suggest you work in virtual environments to keep your Python packages from getting too cluttered. If you're working with Anaconda, see their excellent documentation on environments [here](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html?highlight=environment#creating-an-environment-with-commands).



## [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)
* For our purposes we do not need to compile [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page), so installation only requires that you download a tar file from the link provided and unpack it.



## [Boost](http://www.boost.org/)
* Download and unpack the desired distribution of [Boost](http://www.boost.org/).
* Move into the boost directory and run `./bootstrap.sh` with the options specified below. If you are using intel use `--with-toolset=intel-linux` and make sure that you're mpi compiler is installed (and loaded).
    * `cd boost_1_57_0`
    * `./bootstrap.sh --with-toolset=intel-linux --prefix=/path_to/boost_1_57_0/stage`
    * Add `using mpi : mpiicpc ;` to the file "project-config.jam" (replace `mpiicpc` with your desire mpi compiler)
    * `./b2 install -j28` (Replace "-j28" with "-j<max_num_processors>" to reduce compile time)
    * `cd stage/lib` check that the files "libboost_mpi.so" and "libboost_serialization.so" have been created, if they have you're done! If not check for errors in the compilation of boost.


# Quantum Chemistry Software
<!-- Specific Quantum Chemistry Software -->
## [PySCF](https://github.com/sunqm/pyscf)
* The instructions on the [PySCF](https://github.com/sunqm/pyscf) website are great. We've given a summary below:

    ```bash
    cd pyscf/lib
    mkdir build; cd build
    cmake ..
    make
    export PYTHONPATH=/path_to/pyscf:$PYTHONPATH
    ```

* If you run into any issues, we suggest you visit the PySCF github page, which has a very helpful list of [common problems encountered during installation.](https://github.com/sunqm/pyscf#known-problems)


## [Dice](https://sanshar.github.io/Dice/installation.html)
* A summary of the installation instructions are given below, if you encounter any issues please see the installation section on the [Dice website](https://sanshar.github.io/Dice/installation.html).
* As an added precaution make sure that the compilers are the same used for your boost library.

* First download and unpack the latest version of Dice.
* Edit the following lines in the "Makefile":
    ```make
    USE_MPI =   yes
    USE_INTEL = yes

    FLAGS =  -std=c++11 -g -O3 -I/path_to/eigen -I/path_to/boost_1_57_0/
    DFLAGS = -std=c++11 -g -O3 -I/path_to/eigen -I/path_to/boost_1_57_0/ -DComplex
    ```
    * Choose whether or not you want to install with MPI or not and whether or not you would prefer to use and Intel compiler. (If you set `USE_INTEL = no`, Dice will only look for GCC compilers as alternatives.)

* In a terminal (if you haven't already) and in the Dice directory, type:

  ```bash
  make -j28 Dice
  cd tests
  ./runAllTest.sh
  ```
  * As with Boost the -j28 flag indicates that we want to compile using 28 processors. You should adjust this to the maximum number of processors available to expedite the compilation.
  * You may also need to edit the variable `MPICOMMAND` in the "runAllTests.sh" script to reflect the number of processors you wish to devote to the tests.


* To use Dice with PySCF you will need to edit `/path_to/pyscf/pyscf/shciscf/settings.py.example` and change the SHCIEXE and SHCILIB variables to their appropriate paths on your machine. Then rename the file to `/path_to/pyscf/pyscf/shciscf/settings.py`.

* Dice relies on the DMRGSCF module in PySCF and in order for this module to run properly the file `/path_to/pyscf/pyscf/dmrgscf/settings.py` must exist. *You do not need to install DMRG software,* you only need to rename the settings.py.example as settings.py.

## ORCA
- For a detailed tutorial on installing and setting up ORCA, see the `./orca.md` in this directory.

## Bagel
- For a detailed tutorial on installing and setting up Bagel, see the `./bagel.md` in this directory.

# Visualizing Molecules
## [Jmol](http://jmol.sourceforge.net/download/)
* Follow the direct link to the zip file and download it. Once you unpack the software you can run it by executing the jmol.jar file.

## [IQmol](http://iqmol.org/downloads.html)
- A free package that you can use to build molecules.

## [NGLViewer](http://nglviewer.org/ngl/api/)
- An open source package for moleculear visualization. You can use the web app to view molecules or orbitals. You can download the Python/Jupyter plugin to work with moleceules inside Jupyter Notebooks.

## [Molview](https://molview.org/)
- A browser based applications to create/visualize molecules.
- You can create molecules by hand or use SMILES strings.


# Misc. Software
## [sqa](https://github.com/mussard/SecondQuantizationAlgebra)
* __IT'S REALLY IMPORTANT THAT YOU DOWNLOAD/USE PYTHON 2.X BECAUSE THE SQA MODULE IS NOT COMPATIBLE WITH PYTHON 3.X__
* Make a clone of the git repository and make python aware of this new library by updating the `PYTHONPATH` variable (you need to do this in every bash environment you open, or add it to your `bashrc`):

```
git clone https://github.com/mussard/SecondQuantizationAlgebra your_folder
export PYTHONPATH=/path_to/your_folder:$PYTHONPATH
```

