# Install blasr on Mac OS X

## Prerequisites

These instructions assume you have [Homebrew](http://brew.sh) already installed.

## Installation instructions

    brew uninstall --force hdf5 gcc boost blasr
    brew install open-mpi
    brew install gcc --without multilib  ## v. 5.3.0, be patient as this takes a long time to compile.
    brew install boost
    brew install hdf5 --cc=gcc-5
    brew install blasr --cc=gcc-5
