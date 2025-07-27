# Building opentrack on and for macOS

In the year 2024

First install https://www.macports.org for your architecture (Intel or arm64) (version >= 2.10.3 because of a bug https://trac.macports.org/ticket/71052). I did not get it to work with homebrew, but there's a contribution in this PR: https://github.com/matatata/opentrack/pull/1 . I probably won't be able to maintain both build system so I'll stick with macports for now.

Open a Terminal:

    cd ~/Desktop/
    
    # Unfortunately there's no onnxruntime in macports so we'll donwload it:

    curl -L https://github.com/microsoft/onnxruntime/releases/download/v1.17.3/onnxruntime-osx-universal2-1.17.3.tgz > onnxruntime-osx.tgz
    
    tar -xzf onnxruntime-osx.tgz 

    # We also want to compile opentacks xplane-plugin which is very convenient and need to download the X-lane SDK. In this case for X-Plane 12. **For X-Plane 11 change the SDK version to 303, but note that you can only build for x86_64**
    curl -L http://developer.x-plane.com/wp-content/plugins/code-sample-generation/sdk_zip_files/XPSDK401.zip > XPSDK.zip
    unzip XPSDK.zip

    git clone https://github.com/matatata/opentrack.git

    sudo port selfupdate
    
    # SKIP this if you build for x86_64
    export OTR_OSX_ARCH=arm64

    sudo port -N install cmake qt5 opencv4 libomp create-dmg ImageMagick

    
    # becuase of picky openmp apparently we need to install and use non-Apple clang
    sudo port -N install clang-19
    sudo port select --set clang mp-clang-19

    # For WINE integration you'll need to have to install the 'dev' variant of wine (sudo port install wine-stable +dev)
    # and then add the -DSDK_WINE=1 option. To skip the time consuming installation of wine dev
    # also add -DPREBUILT_WINE_WRAPPER_LOCATION=/directory/where/thefileis/ to use a prebuilt opentrack-wrapper-wine.exe.so.

    export PATH=$PATH:/opt/local/bin:/opt/local/libexec/qt5/bin
    # For the OSC protocol clone this fork
    cd ~/Desktop
    git clone https://github.com/matatata/oscpack.git

    cd ~/Desktop/oscpack
    make install-local
    
    cd ~/Desktop/opentrack

Make sure following lines in cmake/apple.cmake are commented out and look like the following. I'd like to get rid of them comnpletely, but I'm not sure if this would break something it the github pipelines
#set(OpenCV_DIR ~/dev/opentrack-depends/opencv/build)
#set(Qt5_DIR ~/Qt5.6.0/5.6/clang_64/lib/cmake/Qt5)

    
    cmake \
	-DCMAKE_BUILD_TYPE=RELEASE \
 	-DCMAKE_C_COMPILER=/opt/local/bin/clang -DCMAKE_CXX_COMPILER=/opt/local/bin/clang++ \
	-DOpenCV_DIR=/opt/local/libexec/opencv4/cmake \
	-DONNXRuntime_LIBRARY=~/Desktop/onnxruntime-osx-universal2-1.17.3/lib/libonnxruntime.dylib \
	-DONNXRuntime_INCLUDE_DIR=~/Desktop/onnxruntime-osx-universal2-1.17.3/include \
	-DOpenMP_CXX_FLAG="-fopenmp" \
	-DOpenMP_CXX_INCLUDE_DIR=/opt/local/include/libomp \
	-DOpenMP_CXX_LIB_NAMES=libomp \
	-DOpenMP_C_FLAG="-fopenmp" \
	-DOpenMP_C_INCLUDE_DIR=/opt/local/include/libomp \
	-DOpenMP_C_LIB_NAMES=libomp \
	-DOpenMP_libomp_LIBRARY=/opt/local/lib/libomp/libomp.dylib \
	-DSDK_XPLANE=~/Desktop/SDK \
 	-DSDK_WINE=1 \
  	-DPREBUILT_WINE_WRAPPER_LOCATION=/directory/where/thefileis/ \
        -DSDK_OSCPACK=~/Desktop/oscpack/oscpack \
  	--toolchain cmake/apple.cmake \
	-S . -B ../opentrack_build
    
    cd ../opentrack_build
    make -j5 install
    
    # switch back to default compiler:
    sudo port select --set clang none

Have a cup of tea.

--------------

        
If everything went fine You'll now see the opentrack.app in ~/Desktop/opentrack_build/install. You'll also find a .dmg-File in ~/Desktop/opentrack_build/. If you open the opentrack.app directly from the install-Folder it probably crashes on Apple-Silicon with some nasty warning/error.

In that case you'll have to sign it locally to be able to run it. I wonder why this error messages sucks so much.

    cd install
    codesign --force --deep --sign - opentrack.app
        
It should say `opentrack.app: replacing existing signature`

**Update: Note that I already do the above in macosx/make-app-bundle.sh, so it should work fine from the start.**
        
Now you should be able to start it. It will ask for permissions to access the Documents folder, because it wants to creatE/store the profile files. If you start tracking using trackers like PointTracker and neuralnet-Face that want to use a Webcam then macOS asks for permission. Grant permission, stop Tracking and restart again. Or maybe restart the app.




  




