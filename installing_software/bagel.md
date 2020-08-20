# Installing bagel
Bagel is quantum chemistry package that is solely based on the powerful technique called density-fitting. 
It has efficient implementations on lots of wavefunction methods with the exception of the entire class of coupled-cluster method.
The relativistic analogues of every method has also been implemented. 
So it can be considered a powerful tool to do relativistic electronic structure.
But building bagel has never been a pleasant experience.

## Prerequisites
    MPI C++ compiler
    GNU libtool
    GNU autoconf
    GNU automake
    Boost C++ libraries
    BLAS, LAPACK and ScaLAPACK
For the MPI C++ compiler, if you don't have it, you will not be able to compile smith, which is the code generator for bagel's mrpt and mrci code, so if you are pretty sure you are not going to use them, you can do it with a normal compiler. Based on my experience, intel C++ compiler will cause numerical error while doing some relativistic calculation, and it will also give error when compiling the Toru's private branch with DFT implementations, I am assuming he is not testing the utilities with intel compiler.

For GNU libtool, autoconf and automake, you can simply ``` module load autotools ``` on the cluster or ``` sudo apt-get install autotools``` on your local machine.

You must have boost compiled if you have experience in the group.

BLAS, LAPACK and ScaLAPACK I would just recommend using intel math kernel library (mkl), and I don't have experience with other kind of BLAS libraries.

In a word, if you are on the cluster, you just need to do the following thing to get yourself prepared:
``` bash
ml gcc, impi, mkl, boost, autotools
```
If you want to use your own boost,

## Build BAGEL
### Automake
``` bash
libtoolize
aclocal
autoconf
autoheader
automake -a
```
Then creat a build dir:
``` bash
mkdir obj
cd obj
```
### Configure
A minimal configure set looks like
``` bash
export BOOST_ROOT=/path/to/boost
export LD_LIBRARY_PATH=$BOOST_ROOT/lib
../configure --enable-mkl --with-boost=$BOOST_ROOT
```
If you are using a staged boost library instead of installed boost library, you should export ```$BOOST_ROOT``` then only use ```--with-boost``` instead of ```--with-boost=/path/to/boost``` like this
``` bash
../configure --enable-mkl --with-boost
```
A recommend configure for release build which I used is
``` bash
../configure --enable-mkl --with-boost=$BOOST_ROOT --with-mpi=intel CXXFLAGS="-DNDEBUG -O3 -mavx"
```
If you don't want to use mpi, you can configure with
``` bash
../configure --without-mpi --disable-smith
```
along with all other normal options.
Toru's private repository for BAGEL which have DFT implemented is sort of available within the group, it requires additional dependency on libxc, the most up-to-date git version is required to build up-to-date bagel_qsimulate. 
Since libxc by default does not enable the shared library option, so we need to turn it on mannually. 
``` bash
git clone https://gitlab.com/libxc/libxc.git
cd libxc
autoreconf -i
cd build
../configure --enable-shared --prefix=/path/to/libxc
```
Configure the bagel_qsimulate can also be troublesome, due to it is not yet a code ready to release, you cannot do  ```--with-libxc=/path/to/libxc```, so my solution is to mannual assign the ```--with-include``` tag and the ```LDFLAGS``` tag
``` bash
...configure --enable-mkl --with-boost=$BOOST_ROOT --with-mpi=intel --with-include=-I/path/to/libxc/include LDFLAGS="-L/path/to/libxc/lib" CXXFLAGS="-DNDEBUG -O3 -mavx"
```
I am hoping that Toru will fix it someday so that I won't need to do such kind of thing to work around it in such a ugly way.

### Compile
``` bash
make -j28 (assuming you are currently at build directory)
make check (optional)
cd src (optional)
./TestSuite --log_level=all (optional)
make install
```
## Remarks
This note is based on [BAGEL](https://nubakery.org)'s official website, along with some of my personal experience, hope to be useful, but the bagel_qsimulate part, since it is changing everyday, I would not gaurantee anything on that.