# Wine on macOS

I once was able to install wine 9.0 via macports in the +dev variant, which includes dev-tools such as winegcc in order to use te winelib functionality. With wine 10.x this apparently is no longer possible. Well it is possible, but the build does not build the various *.a files under lib/wine/x86-64-unix/. Withouut them winegcc and wineg++ will fail, because it's missing libadvapi32.a for instance.

The problem apparently is very much related to the fact that `./configure` is checking for the `-mabi=ms` compiler option. I think it checks wether the Microsoft ABI (Application Binary Interface) can be emitted. I think the test does make sence since we need to cross-compile code (dlls) that actually needs the Microsoft ABI since it will be excecuted by wine. But the test is being applied to the host compiled and the cross-compiler. I get that it's actually required when checking the cross-compiler, but I don't understand why it has the following effect when applied to the host-compiler. When it fails then no libXYZ.a files will be produced in libe/wine/x86_62-unix/. But those are necessary for winegcc.

On wine 9.0 the check apparently is not applied to the host-compiler and the build is a success (apart from various other problems I won't just write down just yet).

## Wine 9.0.1 without macports

I don't recall why but I used clang-mp-16 from macports

Use something like
`export MACOSX_DEPLOYMENT_TARGET=10.14`

Maybe
`LDFLAGS=-Wl,-rpath`

Problems with identifiers named bool ... -std=gnu23

Interesting now the CRT thing shows:

clang-mp-16 -m64 -c -o dlls/ucrtbase/printf.o dlls/ucrtbase/printf.c -Idlls/ucrtbase -Idlls/msvcrt -Iinclude -Iinclude/msvcrt \
  -D_UCRT -D__WINESRC__ -D_CRTIMP= -Wall -pipe -fcf-protection=none -fvisibility=hidden \
  -fno-stack-protector -fno-strict-aliasing -Wdeclaration-after-statement -Wempty-body \
  -Wignored-qualifiers -Winit-self -Wno-pragma-pack -Wstrict-prototypes -Wtype-limits \
  -Wunused-but-set-parameter -Wvla -Wwrite-strings -Wpointer-arith -gdwarf-4 -fPIC \
  -fasynchronous-unwind-tables -D_WIN32 -fno-builtin -fshort-wchar -Wno-format -mabi=ms -g -O2 -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=0
clang: warning: argument unused during compilation: '-mabi=ms' [-Wunused-command-line-argument]

Interestingly printf is when I get crashes when executing a compiled exe.so




