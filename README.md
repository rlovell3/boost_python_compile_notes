# boost_python_compile_notes
My travels with building Boost.Python


Objective:  Build Boost.Python on Ubuntu Linux against an existing, specific conda environment that is using Python3.11.  This is in complete disregard for any default Python installation that may be on your machine, and it also does not require you to sudo apt install anything.

Steps: 
1.  Download boost.  As of this writing:  boost_1_82_0.tar.gz
2.  tar -xzvf boost_1_82_0.tar.gz
3.  sudo mv boost_1_82_0 /usr/local/boost

Decide which python environment you want to build against.  In this case, I want to build against my conda xxx environment, which uses Python 3.11.

4.  Find these files within the specified conda environment, and record their paths:  
    libpython3.11.so     ~/miniconda3/envs/xxx/lib/libpython3.11.so
    
    Python.h             ~/miniconda3/envs/xxx/include/python3.11/Python.h
    
    python3.11 executable   ~/miniconda3/envs/xxx/bin/python3.11
    
5.  Create a directory to hold all your freshly-built Boost Python stuff:
```bash
mkdir ~/1Boost
```

#### NOTE:  
When you run bootstrap, it will backup but also write over any work you may have done to project-config.jam.  If you already made the changes below to project-config.jam, it will likely now be renamed project-config.jam.1.  Or some other number.  In all regards, at this point, you must make sure that the file, project-config.jam is the file with your paths and notes.  You know wnat to do....  

Location:  project-config.jam:  /usr/local/boost/project-config.jam

#### project-config.jam file contents:
```bash

cat project-config.jam
# B2 Configuration
# Automatically generated by bootstrap.sh, but we make changes below before running ./b2 ....

import option ;
import feature ;

# Compiler configuration. This definition will be used unless
# you already have defined some toolsets in your user-config.jam
# file.
if ! gcc in [ feature.values <toolset> ]
{
    using gcc ; 
}

project : default-build <toolset>gcc ;

# Python configuration  (I hard-coded my home/rl path.  Maybe ~/ works.  Fix path to suit your proper /home/whatever/ path.
import python ;
if ! [ python.configured ]
{
    using python 
    : 3.11 
    : /home/rl/miniconda3/envs/xxx/bin/python3.11
    : /home/rl/miniconda3/envs/xxx/include/python3.11
    : /home/rl/miniconda3/envs/xxx/lib
    ;
}

path-constant ICU_PATH : /usr ;


# List of --with-<library> and --without-<library>
# options. If left empty, all libraries will be built.
# Options specified on the command line completely
# override this variable.
libraries = python ;

# These settings are equivalent to corresponding command-line
# options.
option.set prefix : ~/1Boost ;
option.set exec-prefix : ~/1Boost ;
option.set libdir : ~/1Boost/lib ;
option.set includedir : ~/1Boost/include ;

# Stop on first error
option.set keep-going : false ;
##### END of file 
```

### Run these commands to compile Boost.Python:
```bash
cd /usr/local/boost
./bootstrap.sh --with-python=~/Miniconda3/envs/otcrs/bin/python3.11
#  REMEMBER:  you just overwrote project-config.jam.  Assert your project-config.jam is as written above before proceeding.
# Then, ....
./b2 --with-python --debug-configuration --prefix=~/1Boost  --stagedir=~/1Boost/stage --build-type=minimal  --build-dir=~/1Boost-build --layout=versioned --python-buildid=311 --variant=release --link=shared --threading=multi > ~/1Boost/compile_output.txt 2>&1
```

Running ./b2 as listed above will produce a debug output file:   ~/1Boost/compile_output.txt
You can read that file to look for any errors in the build.  


