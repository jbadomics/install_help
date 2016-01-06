The following steps were successful for installing [QIIME](http://qiime.org) on a clean installation of Mac OS X 10.11.2 "El Capitan." 
These instructions assume you have administrator (i.e. `sudo`) privileges.

For more information: [http://qiime.org/1.8.0/install/install.html](http://qiime.org/1.8.0/install/install.html)

## Install Homebrew

[Homebrew](http://brew.sh) is an incredibly helpful package manager for OS X and I highly recommend it. You will need to install Xcode (available in the Mac App Store; be patient as the installation takes several minutes) and XQuartz (https://dl.bintray.com/xquartz/legacy-downloads/SL/XQuartz-2.7.8.dmg) to use Homebrew. Open Xcode and agree to the terms and conditions, then reboot.

In a Terminal, type

    xcode-select --install

From Apple's [documentation](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/xcode-select.1.html), `xcode-select` controls the location of the developer directory used by xcrun(1), xcodebuild(1), cc(1), and other Xcode and BSD development tools.

To check Homebrew's status, type

    brew update
    brew doctor

Follow any instructions provided by `brew doctor` until its output reports `Your system is ready to brew.`

Install some dependencies:

    brew install wget open-mpi

## Optional Add-ons

### Install Java

Install JRE 8u66 http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html

Install JDK 8u66http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

### Install R and R packages

    brew install r
    
    r
    install.packages('randomForest')
    install.packages('ape')
    install.packages('vegan')
    install.packages('optparse')
    install.packages('gtools')
    install.packages('klaR')
    install.packages('RColorBrewer')
    install.packages("biom")
    source("http://bioconductor.org/biocLite.R")
    biocLite("metagenomeSeq")
    biocLite("DESeq2")

### Install legacy BLAST, USEARCH, and other tools

[MacQIIME](http://www.wernerlab.org/software/macqiime/macqiime-installation) has some helpful instructions for installing other optional add-ons.

## Install GCC

The [GNU Compiler Collection](https://gcc.gnu.org) is used on GNU-based *nix operating systems (e.g. Linux) for compiling C/C++ code. However, Apple's default compiler `clang` often fails to compile code designed for Linux. Fortunately, Homebrew provides the capability to install the same GNU Compiler Collection (GCC) used in Linux to compile code on a Mac. Run

    brew install gcc --without-multilib
    
**NOTE:** This process takes a considerable amount of time (>30 min), so be patient during the `make bootstrap` step.

This command installs the most recent GCC version 5.3.0.

## Install FastTree

QIIME requires version 2.1.3 of FastTree. Navigate to your standard software installation directory:

    cd /path/to/software/directory
    wget http://www.microbesonline.org/fasttree/FastTree-2.1.3.c
    
Compile FastTree:

    /usr/local/bin/gcc-5 -O3 -finline-functions -funroll-loops -Wall -o FastTree FastTree-2.1.3.c -lm

Optionally, also compile FastTreeMP:

    /usr/local/bin/gcc-5 -DOPENMP -fopenmp -O3 -finline-functions -funroll-loops -Wall -o FastTreeMP FastTree-2.1.3.c -lm

Put `FastTree` in your `PATH`:

    export PATH=/path/to/software/directory:$PATH
    
Or, add that path to your `.bash_profile`:

    echo 'export PATH=/path/to/software/directory:$PATH' >> ~/.bash_profile && source ~/.bash_profile

## Configure Python

Xcode provides Python v. 2.7.10, which will run QIIME no problem. If, however, you want a more recent version of Python:

    brew install python

To check which Python your system is running and its version, type

    which python
    python -V

### Install Python dependencies

The following commands will install some key dependencies, including NumPy and SciPy:


    sudo -H pip install requests[security]
    sudo -H pip install --upgrade pip setuptools
    sudo -H pip install matplotlib
    sudo -H pip install biopython cython mpi4py virtualenv h5py

## Install sumaclust

Navigate to a directory where you typically install packages and type

    wget https://git.metabarcoding.org/obitools/sumaclust/uploads/d71fb826e86872cc1d11f93e215806a0/sumaclust_v1.0.10.tar.gz
    tar -xvzf sumaclust_v1.0.10.tar.gz
    cd sumaclust_v1.0.10
    make CC=/usr/local/bin/gcc-5

Optionally, put `sumaclust` in your `PATH`:

    export PATH=/path/to/sumaclust_v1.0.10:$PATH
    
Or, add that path to your `.bash_profile`:

    echo 'export PATH=/path/to/sumaclust_v1.0.10:$PATH' >> ~/.bash_profile && source ~/.bash_profile

## Install QIIME

The biggest problem I encountered was installing QIIME's dependency PyCogent, which kept failing for two reasons:

1.  Header files were not found in the default `include` path.
2.  It appears that PyCogent cannot be compiled with `clang` but rather requires `gcc`.

To address the header problem, download and untar PyCogent from SourceForge:

    brew install wget
    cd ~/Downloads
    wget http://sourceforge.net/projects/pycogent/files/PyCogent/1.5.3/PyCogent-1.5.3.tgz
    tar -xvzf PyCogent-1.5.3.tgz

Then copy `include/array_interface.h` to a default search path for header files:

    cp ~/Downloads/PyCogent-1.5.3/include/array_interface.h /usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/include/python2.7/

Then create a symbolic link to NumPy header files in the same default search path

    cd /usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/include/python2.7/
    ln -s /usr/local/lib/python2.7/site-packages/numpy/core/include/numpy

To address the compiler issue, pass additional arguments to `pip` to specify `gcc` as the desired compiler and install QIIME:

    sudo -H env CC=/usr/local/bin/gcc-5 pip install --global-option build_ext qiime

This command will install QIIME and any unmet dependencies, including PyCogent.

## Check the installation

QIIME's [scripts](http://qiime.org/scripts/) should now be in your `$PATH`. In a Terminal, type

    prin

and then press `Tab` twice. If the installation was successful, you should see `principal_components.py`, `print_metadata_stats.py`, and `print_qiime_config.py` as some of the options. Then run

    print_qiime_config.py -tb

To test and verify your installation.

