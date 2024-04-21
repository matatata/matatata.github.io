# matatata's notes on building and using opentrack on macOS

**Including PointTracker, NeuralNet(Face)Tracker and xplane-plugin**

*Tested on Sonoma 14.4*

## Prebuilt (unsigned) binaries

Get them here https://github.com/matatata/opentrack/releases

## General notes on using it with X-Plane
AFAIK you have three choices:

1. Set opentrack's output to "X-Plane" and install opentrack's xplane-plugin ~/Desktop/opentrack/build/instal/xplane/opentrack.xpl by copying it into `<X-Plane 12>/Resources/plugins` before starting X-Plane.
2. Set opentrack's to "UDP over network". Configure the remote address to be 127.0.0.1 Port 4242. Install the x-plane plugin [https://github.com/amyinorbit/headtrack](https://github.com/amyinorbit/headtrack). Start X-Plane and Fly! In X-Plane's Plugin-Menu activate HeadTracking. It should now pick up da Head-Position from opentrack.
3. I guess the X-Camera xplane-plugin is also able to receive UDP position data so the configuration would be similar as in (2)

For the start I highly recommend using (1) since it saves you some configuration work and you can solely work in opentrack.

Also remember to configure a "Center" Keyboard Binding in opentrack's Options. You'll need it. Tweak the mappings and so on. Have fun.

## Notes on installing x-plane plugins
Installing unsigned X-Plane plugins often requires removing the qurantine flags by executing `sudo xattr -c opentrack.xpl` in this case or `sudo xattr -cr .` inside the plugin's folder in case it comes in a folder which is quite common. Only do that for software you trust! Alternatively let macOS warn you and you'll have to trust the plugin in the system preferences.

## Building

First install https://www.macports.org for your architecture (Intel or arm64), I did not get it to work with homebrew.
If you use homebrew, it may be necessary to temporarily comment out (#) the line that read something like this: `eval "$(/opt/homebrew/bin/brew shellenv)"` in case you have it in your ~/.zprofile or ~/.bash_profile file.

Open a Terminal:

    cd ~/Desktop/
    
    # Unfortunately there's no onnxruntime in macports so we'll donwload it:

    curl -L https://github.com/microsoft/onnxruntime/releases/download/v1.17.3/onnxruntime-osx-universal2-1.17.3.tgz > onnxruntime-osx.tgz
    
    tar -xzf onnxruntime-osx.tgz 

	 # We also want to compile opentack's xplane-plugin which is very convenient and need to download the X-lane SDK. In this case for X-Plane 12. **For X-Plane 11 change the SDK version to 303, but note that you cannot only build for x86_64**
	 
	 curl -L https://developer.x-plane.com/wp-content/plugins/code-sample-generation/sample_templates/XPSDK401.zip > XPSDK.zip
	 
	 unzip XPSDK.zip

    git clone https://github.com/matatata/opentrack.git

    

    export PATH=$PATH:/opt/local/bin:/opt/local/libexec/qt5/bin
    
    sudo port selfupdate

In case you're on a Apple-Silicon Mac: `export OTR_OSX_ARCH=arm64`


    sudo port install cmake qt5 opencv4 libomp create-dmg ImageMagick
    
    cd ~/Desktop/opentrack
    
    cmake \
    -DOpenCV_DIR=/opt/local/libexec/opencv4/cmake \
    -DONNXRuntime_LIBRARY=~/Desktop/onnxruntime-osx-universal2-1.17.3/lib/libonnxruntime.dylib \
    -DONNXRuntime_INCLUDE_DIR=~/Desktop/onnxruntime-osx-universal2-1.17.3/include \
    -DOpenMP_CXX_FLAG="-Xclang -fopenmp" \
    -DOpenMP_CXX_INCLUDE_DIR=/opt/local/include/libomp \
    -DOpenMP_CXX_LIB_NAMES=libomp \
    -DOpenMP_C_FLAG="-Xclang -fopenmp" \
    -DOpenMP_C_INCLUDE_DIR=/opt/local/include/libomp \
    -DOpenMP_C_LIB_NAMES=libomp \
    -DOpenMP_libomp_LIBRARY=/opt/local/lib/libomp/libomp.dylib \
    -DSDK_XPLANE=~/Desktop/SDK \
    -S . -B build --toolchain cmake/apple.cmake
    
    cd build
    make install

Have a cup of tea.

    open install
        
If everything went fine You'll now see the opentrack.app in ~/Desktop/opentrack/build/install. You'll also find a .dmg-File in ~/Desktop/build/. If you open the opentrack.app directly from the install-Folder it probably crashes on Apple-Silicon with something like  or similar.

In that case you'll have to sign it locally to be able to run it. I wonder why this error messages sucks so much.

    cd install
    codesign --force --deep --sign - opentrack.app
        
It should say `opentrack.app: replacing existing signature`

**Update: Note that I already do the above in macosx/make-app-bundle.sh, so it should work fine from the start.**
        
Now you should be able to start it. It will ask for permissions to access the Documents folder, because it wants to creatE/store the profile files. If you start tracking using trackers like PointTracker and neuralnet-Face that want to use a Webcam then macOS asks for permission. Grant permission, stop Tracking and restart again. Or maybe restart the app.




  




