# matatata's notes on building and using opentrack on macOS

*Tested on Sonoma 14.6.1, Opentrack 2024.1.1*

## Prebuilt (unsigned) binaries

Get them [https://github.com/matatata/opentrack/releases](https://github.com/matatata/opentrack/releases)

### Bugs
- Camera-Selection is totally confused when you have more than one camera. This related to https://github.com/opencv/opencv/issues/22901 and mainly because opencv does not allow you to simply enumerate devices https://github.com/opencv/opencv/issues/4269. Run with environment variable OPENCV_VIDEOIO_DEBUG=1 and you'll see that Opentrack uses a different enumeration than opencv. Opentrack uses QTCamera, but opencv AVFoundation. I'll fix that.
- No Camera-Settings Dialog (won't fix)

## General notes on Headtracking with X-Plane on macOS with or without using opentrack

If you only want to use it with X-Plane set opentrack's output to "X-Plane" and install opentrack's xplane-plugin opentrack.xpl by copying it into `<X-Plane 12>/Resources/plugins`. Disable wine in the options dialog.

You then have to decide what input method you want to use.
1. In case you have a IR-camera and a IR-Reflector or LED-HeadClip (recommended) use PointTracker.
2. If you have a FaceTime-Camera or regular WebCam - or even an iPhone choose Neuralnet-Tracker. This will track your face.
3. If you want to use the iOS or Android App like Smoothtrack choose "UDP over network". Configure smoothtrack to send data to your computers ip-address.
   
If you want to feed motion data into a game that's run via wine/crossover or apple's gameportingkit choose the Wine/X-Plabe output and enable the wine-option in its options dialog.

Also remember to configure a "Center" Keyboard Binding in opentrack's Options. You'll need it. Tweak the mappings and so on. Have fun.

Note that xou do not need to use opentrack at all in case you have some other tracker like the Smoothtrack mobile app. In that case use'll probably want to use the x-plane plugin [https://github.com/amyinorbit/headtrack](https://github.com/amyinorbit/headtrack) or perhaps X-Camera. Or use opentrack in the middle.


## Notes on installing x-plane plugins
Installing unsigned X-Plane plugins often requires removing the qurantine flags by executing `sudo xattr -c opentrack.xpl` in this case or `sudo xattr -cr .` inside the plugin's folder in case it comes in a folder which is quite common. Only do that for software you trust! Alternatively let macOS warn you and you'll have to trust the plugin in the system preferences.

## Building opentrack yourself
Note that the binaries I built use the original source code, but you'll have to trust me that I did not alter it. I encourage you to build the software yourself yet I still recommend to clone my [fork](https://github.com/matatata/opentrack) since I made some small tweaks so that the build actually works with the instructions I give. But please go ahead and review the [differences](https://github.com/opentrack/opentrack/compare/master...matatata:opentrack:master) between my fork and the original [repo](https://github.com/opentrack/opentrack) if there are any.

First install https://www.macports.org for your architecture (Intel or arm64) (version >= 2.10.3 because of a bug https://trac.macports.org/ticket/71052). I did not get it to work with homebrew, but there's a contribution in this PR: https://github.com/matatata/opentrack/pull/1 . I probably won't be able to maintain both build system so I'll stick with macports for now.

Open a Terminal:

    cd ~/Desktop/
    
    # Unfortunately there's no onnxruntime in macports so we'll donwload it:

    curl -L https://github.com/microsoft/onnxruntime/releases/download/v1.17.3/onnxruntime-osx-universal2-1.17.3.tgz > onnxruntime-osx.tgz
    
    tar -xzf onnxruntime-osx.tgz 

	 # We also want to compile opentack's xplane-plugin which is very convenient and need to download the X-lane SDK. In this case for X-Plane 12. **For X-Plane 11 change the SDK version to 303, but note that you cannot only build for x86_64**
	 
	 curl -L http://developer.x-plane.com/wp-content/plugins/code-sample-generation/sdk_zip_files/XPSDK401.zip > XPSDK.zip
	 
	 unzip XPSDK.zip

    git clone https://github.com/matatata/opentrack.git

    sudo port selfupdate
    
    # SKIP this if you build for x86_64
    export OTR_OSX_ARCH=arm64

    sudo port -N install cmake qt5 opencv4 libomp create-dmg ImageMagick

    
    # becuase of picky openmp we need to install and use non-Apple clang
    sudo port -N install clang-18
    sudo port select --set clang mp-clang-18

    # For WINE integration you'll need to have to install the 'dev' variant of wine (sudo port install wine-stable +dev)
    # and then add the -DSDK_WINE=1 option. To skip the time consuming installation of wine dev
    # also add -DPREBUILT_WINE_WRAPPER_LOCATION=/path/where/thefileis to use a prebuilt opentrack-wrapper-wine.exe.so.

    export PATH=$PATH:/opt/local/bin:/opt/local/libexec/qt5/bin
    cd ~/Desktop/opentrack
    
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
	-S . -B build_x86_64 --toolchain cmake/apple.cmake
    
    cd build
    make -j5 install
    
    # switch back to default compiler:
    # sudo port select --set clang none

Have a cup of tea.

--------------

        
If everything went fine You'll now see the opentrack.app in ~/Desktop/opentrack/build/install. You'll also find a .dmg-File in ~/Desktop/build/. If you open the opentrack.app directly from the install-Folder it probably crashes on Apple-Silicon with some nasty warning/error.

In that case you'll have to sign it locally to be able to run it. I wonder why this error messages sucks so much.

    cd install
    codesign --force --deep --sign - opentrack.app
        
It should say `opentrack.app: replacing existing signature`

**Update: Note that I already do the above in macosx/make-app-bundle.sh, so it should work fine from the start.**
        
Now you should be able to start it. It will ask for permissions to access the Documents folder, because it wants to creatE/store the profile files. If you start tracking using trackers like PointTracker and neuralnet-Face that want to use a Webcam then macOS asks for permission. Grant permission, stop Tracking and restart again. Or maybe restart the app.




  




