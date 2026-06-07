# Building opentrack on and for macOS

In the years 2024/2025/2026

Every time I come back here to build a new release I have to fix new problems! Everything breaks all the time. Everything is crap... After spending 10 hours to get things working on Tahoe I'm too annoyed to write it all down. The solutions won't last long anywhay. I never experience this much pain on other unix systems. macOS is horrible for open source development. Maybe I'm not experienced or clever enough, but it really sucks.

First install https://www.macports.org for your architecture (Intel or arm64) (version >= 2.10.3 because of a bug https://trac.macports.org/ticket/71052). I did not get it to work with homebrew, but there's a contribution in this PR: https://github.com/matatata/opentrack/pull/1 . I probably won't be able to maintain both build system so I'll stick with macports for now.

Open a Terminal:

	# not sure we need to set it here because we have it the toolchain apple.cmake!
 	# export MACOSX_DEPLOYMENT_TARGET=15
	# optionally set a SDKROOT	although not recommended by Apple
	# export SDKROOT=/Library/Developer/CommandLineTools/SDKs/MacOSX15.sdk


	SRC_DIR=~/Git/opentrack

	## choose the opentrack branch
    git clone --single-branch --branch master https://github.com/matatata/opentrack.git "SRC_DIR"

	BUILD_DIR=~/Desktop/opentrack_build
    mkdir -p "$BUILD_DIR"
	cd "$BUILD_DIR"
    
    # Unfortunately there's no onnxruntime in macports so we'll donwload it:
	ONXVERSION=1.23.2
    curl -L https://github.com/microsoft/onnxruntime/releases/download/v$ONXVERSION/onnxruntime-osx-universal2-$ONXVERSION.tgz > onnxruntime-osx.tgz
    
    tar -xzf onnxruntime-osx.tgz 

    # We also want to compile opentacks xplane-plugin which is very convenient and need to download the X-lane SDK. In this case for X-Plane 12. **For X-Plane 11 change the SDK version to 303, but note that you can only build for x86_64**
    curl -L http://developer.x-plane.com/wp-content/plugins/code-sample-generation/sdk_zip_files/XPSDK401.zip > XPSDK.zip
    unzip XPSDK.zip
	

    sudo port selfupdate
    
    # SKIP this if you build for x86_64
    export OTR_OSX_ARCH=arm64
    #create-dmg
    sudo port -N install cmake qt6 opencv4 libomp  ImageMagick7 pandoc

    
    # becuase of picky openmp apparently we need to install and use non-Apple clang.
    sudo port -N install clang-21
    sudo port select --set clang mp-clang-21

    # For LEGACY WINE integration you'll need to have to install the 'dev' variant of wine (sudo port install wine-stable +dev)
    # and then add the -DSDK_WINE=1 option. To skip the time consuming installation of wine dev
    # also add -DPREBUILT_WINE_WRAPPER_LOCATION=/directory/where/thefileis/ to use a prebuilt opentrack-wrapper-wine.exe.so.
	# Note that since at least wine 10.x compiling winelibs is no longer supported on unix with clang, therefore I created a different
	# solution. For that clone wine and set SDK_WINE_PATH accordingly:
 	# clone wine
    # git clone --single-branch --branch stable https://github.com/wine-mirror/wine.git
    export PATH=$PATH:/opt/local/bin:/opt/local/lib/ImageMagick7/bin
    
	# For the OSC protocol clone this fork
	git clone https://github.com/matatata/oscpack.git "$BUILD_DIR/oscpack"

	# For osx optional: TODO make 'make install-local' work (again)
    #pushd "$BUILD_DIR/oscpack"
    #sudo make install
	#popd
    
 	

	# These are experimental old or deprecated (at least from my point of view)
	OPTIONAL_OPTS="-DSDK_OSCPACK=/usr/local"
	WINE_OPTS="-DSDK_WINE_PATH=\"$BUILD_DIR/wine\" "
	DEPRECATED_OPTS="-DSDK_WINE=1"
	
	cmake \
	-DCMAKE_BUILD_TYPE=RELEASE \
 	-DCMAKE_C_COMPILER=/opt/local/bin/clang -DCMAKE_CXX_COMPILER=/opt/local/bin/clang++ \
	-DOpenCV_DIR=/opt/local/libexec/opencv4/cmake \
	-DONNXRuntime_LIBRARY="$BUILD_DIR/onnxruntime-osx-universal2-$ONXVERSION/lib/libonnxruntime.dylib" \
	-DONNXRuntime_INCLUDE_DIR="$BUILD_DIR/onnxruntime-osx-universal2-$ONXVERSION/include" \
	-DOpenMP_CXX_FLAG="-fopenmp" \
	-DOpenMP_CXX_INCLUDE_DIR=/opt/local/include/libomp \
	-DOpenMP_CXX_LIB_NAMES=libomp \
	-DOpenMP_C_FLAG="-fopenmp" \
	-DOpenMP_C_INCLUDE_DIR=/opt/local/include/libomp \
	-DOpenMP_C_LIB_NAMES=libomp \
	-DOpenMP_libomp_LIBRARY=/opt/local/lib/libomp/libomp.dylib \
	-DSDK_XPLANE="$BUILD_DIR/SDK" \
    $OPTIONAL_OPTS \
  	--toolchain "$SRC_DIR"/cmake/apple.cmake \
	-S "$SRC_DIR" -B "$BUILD_DIR/cmake_build"
    
    cd "$BUILD_DIR/cmake_build"
    make -j5 install

	# use PACKAGE=1 DEPLOY=1 and CODESIGN_IDENTITY="Developer ID Application" for redistribution
	# But note that some libs cannot be relocated and youu get errors sayinng you need to link those libs with -headerpad_max_install_names
	# In that case you need to rebuild them and add -Wl,-headerpad_max_install_names 
	# fpr macports this should look something like this:
	#configure.ldflags-append \
    #                -Wl,-headerpad_max_install_names
	# I had this for librsvg. Uninstall it sudo port uninstall librsvg, edit the Portfile sudo port edit librsvg, sudo port install -s librsvg
    
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




  




