# emacs-w11-arm64
Port of GNU emacs to Windows 11 on ARM64 machine

The starting point for building is to fetch msys2 for ARM64.  You can download from here https://www.msys2.org/

You will need to open the msys2 shell, and install some packages:

pacman.exe -S --needed base-devel mingw-w64-aarch64-toolchain autoconf \
	   mingw-w64-clang-aarch64-imagemagick mingw-w64-clang-aarch64-gnutls

You will need to open a new msys2 shell, configured for arm64 compilation:

C:\msys64\msys2_shell.cmd -clangarm64

------------------------------

Now we can turn our attention to the emacs source tree.  I have version
30.2 downloaded.

Before we run autogen.sh, we have a few tweaks.  The file emacs.diff contains some context diffs which resolve a few issues that would come up during the build process.  Some of this is just because emacs for mingw32 on ARM is not well supported yet.

The changes aren't all that clean, but indicate where the problem areas are.  The compiler
we are using doesn't have __builtin_longjmp or __builtin_setjmp, and there was a conflict
involving _lseek.  There is no nanosleep(), but to sleep 10ms, we can use the Windows
Sleep(10) call instead.

Once you have applied these changes, then you can run

autogen.sh

which will re-generate the configure script.  This won't take all that long to run, and when complete, it should indicate that configure is ready to be run.

Now you can run the configure script.  I used this:

Now run the configure script:

./configure \
    --prefix=/c/eric/emacs-arm64 \
    --without-native-compilation \
    --host=arm64-w64-mingw32 \
    --target=arm64-w64-mingw32 \
    --build=arm64-w64-mingw32 \
    --with-gnutls \
    --with-jpeg \
    --with-png \
    --with-rsvg \
    --with-tiff \
    --with-wide-int \
    --with-xft \
    --with-xml2 \
    --with-xpm \
    'CFLAGS=-O3 -I/mingw64/include/noX'

This will take a considerable amount of time to run.  You can alter the 'prefix' to be the desired install directory once everything is complete.

You need to place a copy of libgmp-10.dll in the 'src' directory so that temacs can run correctly.

I found a copy in /msys64/clangarm64/bin/libgmp-10.dll.

This by itself won't be sufficient to get everything to build
But this by itself will not be sufficient.  You *also* need to tweak the makefile in 'src' and 'lisp'.

What was happening was that the Makefiles were running temacs.exe, and Make was not correctly detecting that the process exited normally.  This *may* have to do with the fact that Temacs is a subsystem windows application.  That's really just a guess on my part - it could be something else.  But the tools I ran indicate that temacs exited and returned a success code, so the fix was to make it so that make no longer paid attention to the return code.

What I did to work around this was to patch the makefiles.  The changes are in make.diff
