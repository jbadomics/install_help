# Turn your Mac into a bioinformatics workhorse

This page provides step-by-step instructions for setting up and optimizing Mac OS X to run several bioinformatics tools. I recently used this procedure to upgrade both my 2011 MacBook Pro (retrofitted with a 480 GB SSD) and the [Bond Lab's](http://thebondlab.org) 2009 Mac Pro (2 quad-core Intel Xeon processors, retrofitted with 32 GB total RAM and 2 SSDs) to run OS X 10.11 (El Capitan).

The procedures I describe are geared towards novice and intermediate users, and if you are not super comfortable at the command line yet you can copy-paste.

## Why not just use Linux?

Good question. I use Linux a lot too: I have my laptop set up as a dual-boot with Ubuntu, and the [Minnesota Supercomputing Intitute](http://www.msi.umn.edu) which I often use also runs Linux. In certain situations Linux is unavoidable--that is not a bad thing as much of the popular Linux distributions are open source and most bioinformatics tools are designed for a Linux environment. But OS X is more familiar to many scientists just getting into bioinformatics/genomics, and since, like Linux, OS X is Unix-based, working in OS X can be a good stepping stone towards gaining experience and practice in a Unix-based environment.

This procedure is designed to make OS X behave as closely as possible to Linux so that skills, tools, and pipelines can be more readily transferred to a Linux environment down the road.

## Important disclaimers

1.  This page assumes you have administrator (i.e. `sudo`) privileges.

2.  I am a microbiologist, not a computer scientist or software engineer. It is possible (likely) that some of the steps I describe may be overkill, poorly/inaccurately/incompletely explained, redundant, or sub-optimal. If that's the case, please help me improve this page by submitting a pull request!

3.  The tools I recommend installing are, therefore, biased towards those used for working with microbial genomes. However, you can (should!) follow theses instructions if you work in another area of genomics/bioinformatics since many popular tools can be installed safely with [Homebrew](http://brew.sh).

4.  The procedure relies on making a clean installation (NOT an upgrade from an existing OS X) on an empty hard drive. This guarantees that bits of detritus from previous versions don't get carried along and gum up the works. [Daniel](http://thebondlab.org/people) discovered after several upgrades that OS X had somehow maintained drivers for his early-2000s Palm Pilot. A clean installation guarantees that no programs get installed and no files get transferred without your explicit permission.

5.  Installation recommendations should **NOT** be misinterpreted as either implicit or explicit endorsements of certain tools or packages over others. As with any bioinformatics analysis, [Vince Buffalo](http://vincebuffalo.com) says it best: never trust your tools (or your data). This post simply describes how to *install* software in a well-controlled, organized fashion and makes no recommendations for use of said software, as no single tool or set of parameters will apply universally to every question or dataset.

## Before you begin

**Back up all data!** This procedure will erase your hard drive so make sure you have a readable copy (ideally, copies) of your important files on a separate disk.

## Create bootable OS X installer

Find a USB stick or external hard drive with at least 8 GB capacity, and transfer any important data off of it as this procedure will erase the disk.

From the App Store, download OS X El Capitain *but do not begin the installation process*. In Finder, open Applications. You should see an app called "Install OS X El Capitan." Insert your (bootable) USB stick or external hard drive and open Finder to determine the name of your external disk. If you prefer, you can also erase the disk using Disk Utility and simply select the default "Untitled" name. The command below assumes the disk is named "Untitled" but modify as needed. Open Terminal and type: (**NOTE:** this will erase the disk!):

    sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app --nointeraction

and replace `/Volumes/Untitled` with the name of your external disk.

## Install OS X

Reboot the computer and, with your installer disk connected, hold down the Option key. You will see a menu with different boot options. Select the yellow icon labeled "Install OS X El Capitan."

When the installer loads, run Disk Utility. Select your internal hard drive and click "erase." Give the disk a name, and select GUID partition table and OS X Extended (Journaled). When the command completes, quit Disk Utility.

Select "Install OS X" from the menu and select your hard disk (i.e. the one you just erased and reformatted) as the installation target. The system will reboot and eventually ask you for configuration/preferences. **DO NOT transfer any files at this time.**

## Disable System Integrity Protection

El Capitan comes with a feature called System Integrity Protection, which locks down several low-level operating system directories (such as `/usr`) from being written to, even with root access. This becomes important later when installing things like Perl modules as root, for example. You can disable SIP at any time; however, it's best to get it out of the way before we begin installing anything. To disable SIP:

1.  Reboot the computer and hold down Option + R. This will boot into Recovery Mode.

2.  From the Utilties menu, open Terminal and type

```
csrutil disable
```
    
Then reboot the computer.

## Prepare the OS for Homebrew

1.  In System Preferences --> Energy Saver, uncheck "turn off hard disks whenever possible" and slide the "put computer to sleep" bar to "never." (Some software installations take > 10 minutes and this guarantees that they will complete.) Once software is installed, you can re-adjust Energy Saver options to your liking.

2.  Install OS updates from App store.

3.  Install Xcode from the App store (be patient as it can take several minutes).

4.  Launch Xcode and agree to the terms and conditions.

5.  Install the command line developer tools. Open Terminal and type `xcode-select --install`.

6.  Install XQuartz. Go to [http://www.xquartz.org](http://www.xquartz.org) and download the latest version. The installer will guide you through the setup process (be patient).

7.  Install the latest Java SE Runtime Environment (JRE). Go [here](http://www.oracle.com/technetwork/java/javase/downloads) and click on 'JRE download', then select `jre-8u66-macosx-x64.dmg`. Download and run the installer.

8.  Install the latest Java Development Kit (JDK). Go [here](http://www.oracle.com/technetwork/java/javase/downloads) and click on 'JDK download', then select `jdk-8u66-macosx-x64.dmg`. Download and run the installer.

9. Install legacy Java (v. 1.6). This version is required to run [Artemis](http://www.sanger.ac.uk/science/tools/artemis) as well as older versions of the Adobe Creative Suite. It will not conflict with your existing Java 1.8 installations. Click [here](https://support.apple.com/kb/DL1572?locale=en_US) and click on the 'Download' button, then run the installer.

10.  Open TextEdit. You will use TextEdit to modify some plain text files below, so you will need to change preferences for it to behave properly. Under 'Format', select 'plain text'. Uncheck all boxes *except* 'check spelling as you type'. On the 'Open and Save' tab, uncheck 'add .txt extension to plain text files'. Quit TextEdit.

11.  Reboot.

## Install Homebrew

Homebrew is a wonderful package manager for OS X. Most Linux distributions ship with package managers that oversee proper installation (and uninstallation) of software, and Homebrew behaves similarly on OS X. Open a Terminal and type

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

Enter your administrator password when prompted.

### Verify Homebrew

    brew update
    brew doctor

To learn more about Homebrew's features, type `brew help` or `man brew`. The following are some of the most common commands:

    brew update # scans the Homebrew GitHub repo for updates; always run this first
    brew install <package> OPTIONS # install packages, with optional arguments
    brew uninstall --force <package> # completely uninstalls a package
    brew cleanup # remove outdated software and source code
    brew prune # trim unused/dead-end symbolic links
    brew doctor # scan your system and Homebrew installation for errors and provide recommended fixes
    
## Optional: Upgrade bash

Upgrading bash will get you on par with the versions shipped with the latest Linux distributions, and should be at least more on par with versions you might encounter on compute clusters. OS X ships with version 3.2; the latest is version 4.3.

    brew install bash

Now we have `bash` in two places: `/bin/bash` (default OS X, version 3.2) and `/usr/local/bin/bash` (Homebrew, version 4.3). We need to tell the system that we want the more recent bash version as our default shell:

    sudo chsh -s /usr/local/bin/bash $(whoami)

Finally, we need to tell the system that the new bash is an acceptable shell for the system to use:
    
    open -a /Applications/TextEdit.app/ /etc/shells
    
Add the following line to the end of the file:

    /usr/local/bin/bash

Save the file and quit TextEdit.

## Create a bash profile

Now that Homebrew is installed, we want to use Homebrew to install other software such as the GNU flavors of certain shell commands. To do this, we need to tell `bash` where to look for these executables by modifying our `$PATH` variable. On OS X, we need to create a file called `.bash_profile` in our home directory (`~`):

    touch ~/.bash_profile
    open -a /Applications/TextEdit.app/ ~/.bash_profile

The `.bash_profile` contains user-specific configuration for your `bash` shell sessions (on Linux, it is called `~/.bashrc`). You can make it as simple or complex as you wish; however, if you want to be able to invoke any program from anywhere on your computer, you must explicitly append your `$PATH` variable in your `.bash_profile`. It can also be useful to alias certain long or frequently executed commands to simpler ones. My `.bash_profile` is provided [here](https://github.com/jbadomics/install_help/blob/master/example_bash_profile) for some examples.

This [reference](https://www.topbug.net/blog/2013/04/14/install-and-use-gnu-command-line-tools-in-mac-os-x/) describes how to install GNU core utilities and how to set your `.bash_profile` to call GNU commands before the default BSD flavors of the same commands that ship with OS X. In TextEdit, add these lines to your `.bash_profile`:

    export PATH=/usr/local/sbin:$PATH
    export PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
    alias edit='open -a /Applications/TextEdit.app/'
    alias rc='source ~/.bash_profile'
    alias man='_() { echo $1; man -M $(brew --prefix)/opt/coreutils/libexec/gnuman $1 1>/dev/null 2>&1;  if [ "$?" -eq 0 ]; then man -M $(brew --prefix)/opt/coreutils/libexec/gnuman $1; else man $1; fi }; _'

The `edit` and `rc` aliases are optional but helpful, and you can call them whatever you want. The `man` alias should be left alone, as it makes sure to call the appropriate `man` page for GNU commands. Make sure the file ends with a blank line and save the file. In terminal, type

    source ~/.bash_profile

From this point forward, you can accomplish the same command (because we have it aliased) by typing

    rc
    
to tell `bash` to refresh itself with the settings you just added to your `.bash_profile`.

## Install GCC

Apple's C/C++ compiler is called `clang` and is aliased to `gcc`, which means that any time you need to compile source C/C++, `clang` will run by default. Unfortunately, many programs will fail to build with `clang` and instead need the GNU Compiler Collection (`gcc`). We can install GNU gcc v. 6.1 with Homebrew:

    brew install gcc --without-multilib

Some programs won't compile with the latest version of `gcc`, but fortunately homebrew allows us to install multiple versions of the same thing:

    brew install homebrew/versions/gcc5 --without-multilib

Be patient as each step can take 30-60 minutes to complete the `make bootstrap` step. This installs the latest GNU `gcc` with [OpenMP](http://openmp.org/wp/) support. **It does not** erase or replace Apple's default `clang` installation; instead, GNU `gcc` can be invoked as necessary to compile certain programs.

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

Edit your `~/.bash_profile` and add the following:

```
alias find='gfind'
alias locate='glocate'
alias oldfind='goldfind'
alias updatedb='gupdatedb'
alias xargs='gxargs'
```

Installing `findutils --with-default-names` causes a `brew doctor` error about certain packages failing to build. We can silence this error by installing GNU `findutils` commands with a `g` prefix, and simply alias the GNU commands to their default names.

## Install/upgrade other utilities

Xcode now includes Perl 5.18 and so it is not necessary to `brew install perl` unless you specifically need the latest version (5.24 as of 2016-06-27).

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
brew install vim --with-custom-perl=/usr/bin/perl --without-perl
```

Installing `vim` with default settings will download Homebrew's Perl (v. 5.24) and cause problems. The above command installs the latest vim without upgrading to Perl 5.24.

## Configure Perl

Many of my most frustrating installations somehow always seemed to involve missing Perl modules. Perl ships with its own shell called `cpan` which can be used to intall Perl modules. While you *can* install modules as root, I've found that you're far less likely to encounter Perl problems across upgrades if you install modules in your home directory (local::lib).

We're going to use `cpan` to install a simpler Perl module installer called `cpanminus`, but before we get to that point, we need to configure `cpan` itself. Copy and paste the entire line to configure `cpan` and enter the `cpan` shell:

    PERL_MM_OPT="INSTALL_BASE=$HOME/perl5" cpan local::lib

You will be prompted to 'configure as much as possible automatically'. Type `yes`. Eventually `cpan` will ask to select appropriate mirror sites for FTP, and you will be shown a prompt that looks like

    cpan[1]>

When this pan prompt appears, type

    install CPAN # this and the following command will self-upgrade CPAN
    reload cpan
    install App::cpanminus
    exit

This will return you to a normal command prompt, except now you will have `cpanminus` installed. **Note** that you can also install `cpanminus` with Homebrew, but I've found that it's best to use Perl's native setup to install Perl modules (the same goes for Python - see below).

You need to add the following to your `~/.bash_profile` to tell Perl where to find your locally installed modules:

    echo 'eval "$(perl -I$HOME/perl5/lib/perl5 -Mlocal::lib)"' >> ~/.bash_profile
    source ~/.bash_profile

### Install Perl Modules

Now that we have `cpanminus` installed, we will use it to install several Perl modules. The following is a list of Perl modules I've collected as dependencies for several useful and/or popular bioinformatics programs, so it's best to install them upfront and not to have to worry about them later:

    cpanm --self-upgrade
    cpanm --force GD  # this module is notorious for failing to build, but will work here
    cpanm --force Bio::Perl  # some tests fail but these error messages can be disregarded

    cpanm Algorithm::Munkres Array::Compare Convert::Binary::C Graph HTML::TableExtract PostScript::TextBlock SOAP::Lite SVG::Graph Set::Scalar Sort::Naturally Spreadsheet::ParseExcel XML::DOM XML::DOM::XPath XML::Parser::PerlSAX XML::SAX::Writer YAML Carp Clone CPAN::Meta CPAN::Meta::Check CPAN::Meta::YAML Config::General Cwd Data::Dumper Devel::OverloadInfo Devel::StackTrace Digest::MD5 Exception::Class Exporter::Tiny ExtUtils::Install File::Basename File::Copy File::Spec::Functions File::Spec::Link File::Temp File::Find::Rule::Perl File::HomeDir Filesys::Df FindBin Font::TTF::Font Getopt::Long IO::File List::MoreUtils List::Util Mac::SystemDirectory Math::Bezier Math::BigFloat Math::Round Math::VecStat Memoize Module::Build Module::Build::Tiny Module::Metadata Module::Runtime Module::Runtime::Conflicts Moose Number::Format Parse::CPAN::Meta Params::Validate POSIX Readonly Regexp::Common SVG Set::IntSpan Statistics::Basic Statistics::Descriptive Storable Sub::Identify Sys::Hostname Test::CleanNamespaces Test::Harness Test::Most Test::Warnings Text::Balanced Text::Format Time::HiRes XML::Simple XML::Twig

If at any point you discover a Perl module that you're missing, installation *should* be as simple as

    cpanm <Module::Name>

Occasionally it can be helpful to audit which Perl modules you already have installed. I have a simple alias called `perlmodules` to a script [`.listmodules.pl`](https://github.com/jbadomics/install_help/blob/master/.listmodules.pl) which prints a list of installed modules to STDOUT. If you wish, clone the script into your home directory, make it executable, and add the following alias to your `.bash_profile`:

    chmod +x ~/.listmodules.pl
    alias perlmodules='~/.listmodules.pl 2> /dev/null'

## Configure Python

`pip` is a handy module installer for Python. Since we have both Python 2 and Python 3 installed (note that Python 3 is a *fork* of Python, not simply a newer version of Python 2), we will want to make sure that our Python modules are installed to support both versions. `pip2` will install Python 2 modules; `pip3` will install Python 3 modules. Note that not all modules are compatible with both Python versions.

    sudo -H pip2 install requests[security]
    sudo -H pip2 install --upgrade pip setuptools
    sudo -H pip2 install matplotlib
    sudo -H pip2 install alabaster appnope attrs Babel backports.shutil-get-terminal-size bcbio-gff beautifulsoup4 biopython bx-python bz2file cffi checkm-genome cryptography cssselect cutadapt cycler Cython deap decorator DendroPy docutils entrypoints gnureadline h5py html5lib idna imagesize ipykernel ipython ipython-genutils ipywidgets jcvi Jinja2 jsonschema jupyter jupyter-client jupyter-console jupyter-core khmer lxml MarkupSafe matplotlib mistune nbconvert nbformat ndg-httpsclient networkx notebook numpy oauthlib pandas parsel path.py pexpect pickleshare ptyprocess py pyasn1 pyasn1-modules pycparser PyDispatcher Pygments pyOpenSSL pyparsing pysam pytest python-dateutil pytz pyzmq qtconsole queuelib requests requests-oauthlib rpython scikit-learn scipy Scrapy ScreamingBackpack screed service-identity simplegeneric six snowballstemmer Sphinx sphinx-rtd-theme terminado tornado traitlets Twisted twython virtualenv w3lib widgetsnbextension zope.interface

### Optional: Install QIIME

    cd ~/Downloads
    wget http://sourceforge.net/projects/pycogent/files/PyCogent/1.5.3/PyCogent-1.5.3.tgz
    cp ~/Downloads/PyCogent-1.5.3/include/array_interface.h /usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/include/python2.7/
    cd /usr/local/Cellar/python/2.7.11/Frameworks/Python.framework/Versions/2.7/include/python2.7/
    ln -s /usr/local/lib/python2.7/site-packages/numpy/core/include/numpy
    sudo -H env CC=/usr/local/bin/gcc-6 pip install --global-option build_ext qiime

### Install Python3 modules

    sudo -H pip3 install requests[security]
    sudo -H pip3 install --upgrade pip setuptools
    sudo -H pip3 install matplotlib
    sudo -H pip3 install alabaster appnope attrs Babel backports.shutil-get-terminal-size bcbio-gff beautifulsoup4 biopython bx-python bz2file cffi checkm-genome cryptography cssselect cutadapt cycler Cython deap decorator DendroPy docutils entrypoints gnureadline h5py html5lib idna imagesize ipykernel ipython ipython-genutils ipywidgets jcvi Jinja2 jsonschema jupyter jupyter-client jupyter-console jupyter-core khmer lxml MarkupSafe matplotlib mistune nbconvert nbformat ndg-httpsclient networkx notebook numpy oauthlib pandas parsel path.py pexpect pickleshare ptyprocess py pyasn1 pyasn1-modules pycparser PyDispatcher Pygments pyOpenSSL pyparsing pysam pytest python-dateutil pytz pyzmq qtconsole queuelib requests requests-oauthlib rpython scikit-learn scipy Scrapy ScreamingBackpack screed service-identity simplegeneric six snowballstemmer Sphinx sphinx-rtd-theme terminado tornado traitlets Twisted twython virtualenv w3lib widgetsnbextension zope.interface snakemake

## Install other Homebrew packages

    brew install boost --cc=gcc-6
    brew install source-highlight --cc=gcc-6
    brew install highlight --cc=gcc-6
    brew install hdf5 --cc=gcc-6
    brew install blasr --cc=gcc-5
    brew install clustal-w --cc=gcc-6
    brew install glimmer3 --cc=gcc-6
    brew install amos --cc=gcc-5
    brew install diamond --cc=gcc-5
    brew install mysql # follow post-installation instructions

    brew install a5 apple-gcc42 aragorn arb barrnap bdw-gc bedtools bioawk bison blast bowtie bowtie2 cabal-install cairo canu cd-hit cloog cowsay ddate docbook docbook-xsl docker doxygen emboss entr exonerate fasta fastqc fastx_toolkit figlet flex flint fortune gdbm ghc glew gperftools gsl guile hmmer htslib igv igvtools imagemagick imlib2 infernal kmergenie less libvpx lua lzip mash megahit mercurial minced most mummer nettle pandoc picard-tools pixman prodigal prokka quast r raxml readline repeatmasker rmblast rsync s-lang s3cmd samtools screen seqtk sqlite subversion szip tbb tbl2asn tinyxml2 tmux toilet trf trimmomatic trnascan wxmac xmlstarlet xmlto xmltoman
