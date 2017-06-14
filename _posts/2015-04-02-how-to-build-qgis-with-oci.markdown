---
layout: post
title: How to build QGIS with OCI on OSX 10.10
date: 09:48 19-02-2015
headline: Build QGIS from source with OCI baked right in!
taxonomy:
    category: blog
    tag: [grav]
---

As opposed to its Windows counterpart, the **OSX** version of **QGIS** doesn't come with a built-in OCI library which is sad when you have to work with **Oracle Spatial** databases. Fortunately, you can build your own version from source and add whatever dependency you'd like to use, including the OCI library, so let's get started! 
## Prerequisites
We will need several things:

*   Homebrew
*   CMake
*   Git
*   Oracle Instant Client:download it [here][1]

**Homebrew** is a fanstastic package manager for OSX (more info [here][2]), to install it launch your Terminal and run:

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
     
Now were are ready to install **CMake** and **Git**:

    brew install cmake
    brew install git
    

## Install Oracle Instant Client 

Go to **[Oracle Instant Client webpage][1]**, download **both** *OCI SDK* and *OCI Basic* (make sure to download the right version for your OS: 32-bit or 64-bit) and unpack the archives. (For the sake of the tutorial, let's say both files are unzipped in `/Users/loudtyping/Downloads/oci`). 

    cd /Users/loudtyping/Downloads/oci
    
Rename files and copy them to `/usr/local/lib/`:

    mv libclntsh.dylib.11.1 libclntsh.dylib
    sudo cp libclntsh.dylib libnnz11.dylib /usr/local/lib/ 
     
 Change install path:

    sudo install_name_tool -id /usr/local/lib/libclntsh.dylib -change /ade/b/2649109290/oracle/ldap/lib/libnnz11.dylib /usr/local/lib/libnnz11.dylib /usr/local/lib/libclntsh.dylib
    

## Install QGIS dependencies

Follow the steps in the [QGIS documentation][3] to install the required dependencies. Check that you have installed the following:
*   GDAL ([via installer][4])
*   GRASS ([via installer][5])
*   Postgres ([via installer][6])
*   SpatialIndex
*   Python 2.x
*   Qwt 6.0.2
*   SIP
*   QScintilla2
*   PyQt
*   Bison 2.4

## Download and build QGIS
Create a folder to store QGIS source:

    mkdir qgis
    cd qgis
     
Clone the latest version of QGIS:

    git init
    git remote add -f -t master -m master qgisupstream https://github.com/qgis/QGIS.git
    git merge qgisupstream

Go into the unpacked folder and create a `build` folder:

    mkdir build
    cd build
     
It's time to build our installation, run the `cmake` command with the path to your *OCI SDK* in `OCI_INCLUDE_DIR` (don't forget the `..` at the end):
 
    cmake -D CMAKE_INSTALL_PREFIX=~/Applications \
    -D CMAKE_BUILD_TYPE=MINSIZEREL -D ENABLE_TESTS=FALSE \
    -D WITH_PYSPATIALITE=FALSE \
    -D SPATIALINDEX_LIBRARY=/usr/local/lib/libspatialindex.dylib \
    -D SPATIALINDEX_INCLUDE_DIR=/usr/local/include/spatialindex \
    -D QWT_LIBRARY=/usr/local/qwt-6.0.2/lib/libqwt.dylib \
    -D QWT_INCLUDE_DIR=/usr/local/qwt-6.0.2/include \
    -D BISON_EXECUTABLE=/usr/local/bin/bison \
    -D WITH_ORACLE=true \
    -D OCI_LIBRARY=/usr/local/lib/libclntsh.dylib \
    -D OCI_INCLUDE_DIR=/Users/loudtyping/Downloads/oci/sdk/include \
    ..
     
If anything goes wrong, check the `cmake` trace, you might be missing a dependency. Finally, run `make` (you can go grab a cup of coffee) and `make install`:

    make
    sudo make install
     
That's it, you're all set! You will find the app in the `Applications` folder, launch it and enjoy QGIS with its new OCI powers!

 [1]: http://www.oracle.com/technetwork/database/features/instant-client/
 [2]: http://brew.sh/
 [3]: http://htmlpreview.github.io/?https://raw.github.com/qgis/QGIS/master/doc/INSTALL.html#toc20
 [4]: http://www.kyngchaos.com/software/frameworks#gdal_complete
 [5]: http://www.kyngchaos.com/software/grass
 [6]: http://www.kyngchaos.com/software/postgres