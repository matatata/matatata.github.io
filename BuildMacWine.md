# Wine on macOS

I once was able to install wine 9.0 via macports in the +dev variant, which includes dev-tools such as winegcc in order to use te winelib functionality. With wine 10.x this apparently is no longer possible. Well it is possible, but the build does not build the various *.a files under lib/wine/x86-64-unix/. Withouut them winegcc and wineg++ will fail, because it's missing libadvapi32.a for instance.

The problem apparently is very much related to the fact that `./configure` is checking for the `-mabi=ms` compiler option. I think it checks wether the Microsoft ABI (Application Binary Interface) can be emitted. I think the test does make sence since we need to cross-compile code (dlls) that actually needs the Microsoft ABI since it will be excecuted by wine. But the test is being applied to the host compiled and the cross-compiler. I get that it's actually required when checking the cross-compiler, but I don't understand why it has the following effect when applied to the host-compiler. When it fails then no libXYZ.a files will be produced in libe/wine/x86_62-unix/. But those are necessary for winegcc.

On wine 9.0 the check apparently is not applied to the host-compiler and the build is a success (apart from various other problems I won't just write down just yet).


