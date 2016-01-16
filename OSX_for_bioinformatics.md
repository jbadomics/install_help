# Turn your Mac into a bioinformatics workhorse

This page provides step-by-step instructions for setting up and optimizing Mac OS X to run several bioinformatics tools. I recently used this procedure to upgrade both my 2011 MacBook Pro (retrofitted with a 480 GB SSD) and the [Bond Lab's](http://thebondlab.org) 2009 Mac Pro (2 quad-core Intel Xeon processors, retrofitted with 32 GB total RAM and 2 SSDs) to run OS X 10.11 (El Capitan).

## Why not just use Linux?

Good question. I use Linux a lot too: I have my laptop set up as a dual-boot with Ubuntu, and the [Minnesota Supercomputing Intitute](http://www.msi.umn.edu) is obviously Linux-based. In certain situations Linux is unavoidable and that is not a bad thing as much of the popular Linux distributions are open source and most bioinformatics tools are designed for a Linux environment. But OS X is more familiar to many scientists just getting into bioinformatics/genomics, and since, like Linux, OS X is Unix-based, working in OS X can be a good stepping stone towards gaining experience and practice at the command line.

This procedure is designed to make OS X behave as closely as possible to Linux so that skills, tools, and pipelines can be more readily transferred to a Linux environment down the road.

## Important disclaimers

1.  I am a microbiologist, not a computer scientist or software engineer. It is possible (likely) that some of the steps I describe may be overkill, redundant, or sub-optimal. If that's the case, please let me know by submitting a pull request.

2.  The tools I recommend installing are, therefore, biased towards those used for working with microbial genomes. However, you can (should!) follow theses instructions if you work in another area of genomics/bioinformatics since many popular tools can be installed safely with [Homebrew](http://brew.sh).

3.  The procedure relies on making a clean installation (NOT an upgrade from an existing OS X) on an empty hard drive. This guarantees that bits of detritus from previous versions don't get carried along and gum up the works. [Daniel](http://thebondlab.org/people) discovered after several upgrades that OS X had somehow maintained drivers for his early-2000s Palm Pilot. A clean installation guarantees that no programs get installed and no files get transferred without your explicit permission.

## Before you begin

**Back up all data!** This procedure will erase your hard drive so make sure you have a readable copy (ideally, copies) of your important files on a separate disk.

## Create bootable OS X installer

Find a USB stick or external hard drive with at least 8 GB capacity, and transfer any important data off of it as this procedure will erase the disk.

From the App Store, download OS X El Capitain *but do not begin the installation process*. In Finder, open Applications. You should see an app called "Install OS X El Capitan." Insert your (bootable) USB stick or external hard drive and type

    diskutil list

Examine the output to determine the device number for your external drive. It will have the form `diskN` or, alternatively, if you want to put the installer on a specific partition, `diskNsX`. Create the bootable installer (**NOTE:** this will erase the disk!):

    sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app --nointeraction

and replace `/Volumes/Untitled` with the name of your external disk.

## Install OS X

Reboot the computer and, with your installer disk connected, hold down the Option key. You will see a menu with different boot options. Select the yellow icon labeled "Install OS X El Capitan."

When the installer loads, run disk utility. Select your internal hard drive and click "erase." Give the disk a name, and select GUID partition table and OS X Extended (Journaled). When the command completes, quit disk utility.

Select "Install OS X" from the menu and select your hard disk (i.e. the one you just erased and reformatted) as the installation target. The system will reboot and eventually ask you for configuration/preferences. **DO NOT transfer any files at this time.**

## Disable System Integrity Protection

El Capitan comes with a feature called System Integrity Protection, which locks down several low-level operating system directories (such as `/usr`) from being written to, even with root access. This becomes important later when installing Perl modules as root, for example. You can disable SIP at any time, however. To disable SIP:

1.  Reboot the computer and hold down Option + R. This will boot into Recovery Mode.

2.  From the Utilties menu, open Terminal and type

```
csrutil disable
```
    
3.  Reboot the computer.

## Prepare the OS for Homebrew

1.  In System Preferences --> Energy Saver, uncheck "turn off hard disks whenever possible" and slide the "put computer to sleep" bar to "never." (Some software installations take > 10 minutes and this guarantees that they will complete.)

2.  Install OS updates from App store.

3.  Install Xcode from the App store (be patient as it can take several minutes).

4.  Launch Xcode and agree to the terms and conditions.

5.  Install the command line developer tools. Open Terminal and type

```
xcode-select --install
```

6.  Install XQuartz. Go to [http://www.xquartz.org](http://www.xquartz.org) and download the latest version. The installer will guide you through the setup process (be patient).

7.  Install the latest Java SE Runtime Environment (JRE). Go [here](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) and select `jre-8u66-macosx-x64.dmg`. Download and run the installer.

8.  Install the latest Java Development Kit (JDK). Go [here](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) and select `jdk-8u66-macosx-x64.dmg`. Download and run the installer.

9.  Reboot.

## Install Homebrew

Open a Terminal and type

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

Enter your administrator password when prompted.

### Verify Homebrew

    brew update
    brew doctor
    
## Optional: Upgrade bash

Upgrading bash will get you on par with the versions shipped with the latest Linux distributions, and should be at least more on par with versions you might encounter on compute clusters.

    brew install bash
    sudo chsh -s /usr/local/bin/bash $(whoami)
    
    open -a /Applications/TextEdit.app/ /etc/shells
    
Add the following line to the end of the file:

    /usr/local/bin/bash

Save the file and close TextEdit.

## Create a bash profile

Now that Homebrew is installed, we want to use Homebrew to install other software such as the GNU flavors of certain shell commands. To do this, we need to tell `bash` where to look for these executables by modifying our `$PATH` variable. On OS X, we need to create a file called `.bash_profile` in our home directory (`~`):

    touch ~/.bash_profile
    open -a /Applications/TextEdit.app/ ~/.bash_profile

Follow this [reference](https://www.topbug.net/blog/2013/04/14/install-and-use-gnu-command-line-tools-in-mac-os-x/). In TextEdit, add these lines to your `.bash_profile`:


    export PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
    alias edit='open -a /Applications/TextEdit.app/'
    alias rc='source ~/.bash_profile'
    export PATH="/usr/local/sbin:$PATH"

Make sure the file ends with a blank line and save the file. In terminal, type

    source ~/.bash_profile

From this point forward, you can accomplish the same command (because we have it aliased) by typing

    rc
    
To tell `bash` to refresh itself with the settings you just added to your `.bash_profile`.

## Install GCC

Apple's C/C++ compiler is called `clang` and is aliased to `gcc`, which means that any time you need to compile source C/C++, `clang` will run by default. Unfortunately, many programs will fail to build with `clang` and instead need the GNU Compiler Collection (`gcc`). We can install GNU gcc with Homebrew:

    brew install gcc --without-multilib

Be patient as this can take 30-60 minutes to complete the `make bootstrap` step. This installs the latest GNU `gcc` with [OpenMP](http://openmp.org/wp/) support. **It does not** erase or replace Apple's default `clang` installation; instead, GNU `gcc` can be invoked as necessary to compile certain programs.

## Install GNU core utilities

```
brew tap homebrew/science
brew tap homebrew/dupes
brew tap homebrew/versions

brew install coreutils

brew install binutils
brew install diffutils
brew install ed --with-default-names
brew install findutils
brew install gawk
brew install gettext
brew install gnu-getopt
brew install gnupg
brew install gnu-indent --with-default-names
brew install gnu-sed --with-default-names
brew install gnu-tar --with-default-names
brew install gnu-which --with-default-names
brew install gnutls
brew install grep --with-default-names
brew install gzip
brew install screen
brew install watch
brew install wdiff --with-gettext
brew install wget
```

## Install/upgrade other utilities

````
brew install emacs
brew install gpatch
brew install m4
brew install make --with-default-names
brew install nano

brew install file-formula
brew install git
brew install less
brew install openssh
brew install perl518
brew install python
brew install python3
brew install rsync
brew install svn
brew install unzip
brew install zsh
```

## Install libraries and other key dependencies

```
brew install cmake automake fontconfig jpeg libtiff libpng libffi doxygen
brew install libevent --with-doxygen
brew install freetype
brew install glib
brew install gd --with-freetype
brew install libcaca libconfig libtasn1 libtool libunistring zlib icu4c
brew install expat tcl-tk open-mpi parallel openssl ncurses
```

## Configure Perl

    sudo cpan

In CPAN shell, **configure manually!** Use sudo. All other settings can be left as defaults.

When the cpan prompt returns, type

    install CPAN
    reload cpan
    install App::cpanminus
    exit

This will return you to a normal command prompt.

### Install Perl Modules

The following is a list of Perl modules I've collected as dependencies for several useful and/or popular bioinformatics programs, so it's best to install them upfront and not to have to worry about them later:

    sudo cpanm GD
    sudo cpanm GD::Polyline

    sudo cpanm Bio::Perl Carp Clone CPAN::Meta CPAN::Meta::Check CPAN::Meta::YAML Config::General Cwd Data::Dumper Devel::OverloadInfo Devel::StackTrace Digest::MD5 Exception::Class Exporter::Tiny ExtUtils::Install File::Basename File::Copy File::Spec::Functions File::Spec::Link File::Temp File::Find::Rule::Perl File::HomeDir FindBin Font::TTF::Font Getopt::Long IO::File List::MoreUtils List::Util Mac::SystemDirectory Math::Bezier Math::BigFloat Math::Round Math::VecStat Memoize Module::Build Module::Build::Tiny Module::Metadata Module::Runtime Module::Runtime::Conflicts Moose Number::Format Parse::CPAN::Meta POSIX Readonly Regexp::Common SVG Set::IntSpan Statistics::Basic Statistics::Descriptive Storable Sub::Identify Sys::Hostname Test::CleanNamespaces Test::Harness Test::Most Test::Warnings Text::Balanced Text::Format Time::HiRes XML::Simple XML::Twig

## Configure Python

`pip` is a handy module installer for Python. Since we have both Python 2 and Python 3 installed (note that Python 3 is a *fork* of Python, not simply a newer version of Python 2), we will want to make sure that our Python modules are installed to support both versions. `pip` will install Python 2 modules; `pip3` will install Python 3 modules. Note that not all modules are compatible with both Python versions.

    sudo -H pip install requests[security]
    sudo -H pip install --upgrade pip setuptools
    sudo -H pip install matplotlib
    sudo -H pip install biopython cutadapt cython mpi4py rpython pysam checkm-genome jcvi pandas mkdocs sphinx bx-python jupyter virtualenv ngslib

### Optional: Install QIIME

    cd ~/Downloads
    wget http://sourceforge.net/projects/pycogent/files/PyCogent/1.5.3/PyCogent-1.5.3.tgz
    cp ~/Downloads/PyCogent-1.5.3/include/array_interface.h /usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/include/python2.7/
    cd /usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/include/python2.7/
    ln -s /usr/local/lib/python2.7/site-packages/numpy/core/include/numpy
    sudo -H env CC=/usr/local/bin/gcc-5 pip install --global-option build_ext qiime

### Install Python3 modules

    sudo -H pip3 install requests[security]
    sudo -H pip3 install --upgrade pip setuptools
    sudo -H pip3 install matplotlib
    sudo -H pip3 install biopython cutadapt cython mpi4py rpython pysam checkm-genome jcvi pandas mkdocs sphinx bx-python jupyter virtualenv

## Install other Homebrew packages

    brew install boost --cc=gcc-5
    brew install source-highlight --cc=gcc-5
    brew install hdf5 --cc=gcc-5
    brew install blasr --cc=gcc-5
    brew install clustal-w --cc=gcc-5
    brew install glimmer3 --cc=gcc-5
    brew install amos --cc=gcc-5
    brew install diamond --cc=gcc-5
    brew install mysql # follow post-installation instructions

    brew install a5 apple-gcc42 aragorn arb barrnap bdw-gc bedtools bioawk bison blast bowtie bowtie2 cabal-install cairo cd-hit cloog cowsay ddate docbook docbook-xsl docker doxygen emboss entr exonerate fasta fastqc fastx_toolkit figlet flex flint fortune gdbm ghc glew gperftools gsl guile highlight hmmer htslib igv igvtools imagemagick imlib2 infernal kmergenie less libvpx lua lzip megahit mercurial minced most mummer nettle pandoc picard-tools pixman prodigal prokka quast r raxml readline repeatmasker rmblast rsync s-lang s3cmd samtools screen seqtk sqlite subversion szip tbb tbl2asn tinyxml2 tmux toilet trf trimmomatic trnascan wxmac xmlstarlet xmlto xmltoman
