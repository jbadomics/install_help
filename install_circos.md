## Installing Circos

This repo contains instructions for installing [Circos](http://circos.ca) on Linux (Ubuntu 14.04 and CentOS 6.5) and Mac (OS 10.11 El Capitan).

In all cases you should consider adding the /your/path/to/circos-0.69/bin to your $PATH variable in `~/.bashrc` (Linux) or `~/.bash_profile` (Mac).

**NOTE:** These instructions assume root privileges.

### Mac OS X

See instructions [here](https://github.com/jbadomics/install_help/blob/master/OSX_for_bioinformatics.md) for setting up Homebrew and your Perl installation. Though you *can* use Homebrew to install Circos, it now bundles some Perl modules along and I've found this approach to be cludgy. 

Assuming Homebrew, Perl, and cpanminus are already installed:

~~~
brew tap homebrew/science
brew uninstall --force gd
brew install gd --with-freetype
brew install wget

cd /path/to/where/you/want/circos
wget http://circos.ca/distribution/circos-0.69.tgz
tar -xvzf circos-0.69.tgz

cd circos-0.69/bin

sudo cpanm --force GD
sudo cpanm Compress::Zlib $(./circos -modules | grep missing | cut -c20- | tr '\n' ' ')

./circos -modules  # at this point all should say 'ok'

cd ../example
rm circos.* run.out
./run # image takes 45-60 seconds to generate

ls -lrt  # should show circos.png, circos.svg, and run.out
~~~

### Ubuntu 14.04

~~~
sudo apt-get update
sudo apt-get -y install freetype* libgd-dev
sudo apt-get autoremove

sudo ln -s /usr/bin/env /bin/env

cd /path/to/where/you/want/circos
wget http://circos.ca/distribution/circos-0.69.tgz
tar -xvzf circos-0.69.tgz
cd circos-0.69/bin

sudo cpan App::cpanminus

sudo cpanm --force GD
sudo cpanm Compress::Zlib $(./circos -modules | grep missing | cut -c20- | tr '\n' ' ')
./circos -modules  # at this point all should say 'ok'

cd ../example
rm circos.* run.out
./run # image takes 45-60 seconds to generate

ls -lrt  # should show circos.png, circos.svg, and run.out
~~~

### CentOS 6.5

~~~
sudo yum update
which gcc  # if no gcc found: sudo yum install gcc
sudo yum install freetype* perl-CPAN gd gd-devel

sudo cpan
	configure automatically? yes

	install App::cpanminus	# yes to prompts, if any; disregard 'YAML not installed'
	exit

cd /path/to/where/you/want/circos
wget http://circos.ca/distribution/circos-0.69.tgz
tar -xvzf circos-0.69.tgz
cd circos-0.69/bin

sudo /usr/local/bin/cpanm GD --force
sudo /usr/local/bin/cpanm Compress::Zlib $(./circos -modules | grep missing | cut -c20- | tr '\n' ' ')

./circos -modules  # at this point all should say 'ok'

cd ../example
rm circos.* run.out
./run

ls -lrt  # should show circos.png, circos.svg, and run.out
~~~

