# Compiling SQLCipher under Windows with MSVC (2013)

## First step: compiling OpenSSL

You can get more information [here](https://github.com/vltr/msvc-mercy/blob/master/openssl.md).

## General notes

* Download the latest version of SQLCipher [here](https://github.com/sqlcipher/sqlcipher/archive/master.zip);
* Install TCL in your machine - you can grab ActiveTCL [here](http://www.activestate.com/activetcl/downloads);
* Always compile versions starting from a fresh source directory;
* For this document, the OpenSSL path is `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm` (release version with assembly speed ups);
* Always use the Visual Studio native compiler for your arch; in this document MSVC 2013u5 was used.

## Building SQLCipher release version

Considering the working directory `%CUSTOM_IMPORTPATH%` and SQLCipher source code in `sqlcipher.zip`.

First, clean up (any mess) and create your SQLCipher "dist folder":

```batch
rmdir /S /Q sqlcipher
mkdir sqlcipher-dist
```

Now, you can extract SQLCipher and enter its directory:

```batch
"%PROGRAMFILES%\7-Zip\7z.exe" x sqlcipher.zip
cd sqlcipher
```

Optional: you can test if everything would work correctly by invoking SQLCipher tests:

```batch
xcopy "%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\bin\libeay32.dll" .
nmake -f Makefile.msc quicktest "NCC=""%VCINSTALLDIR%\bin\cl.exe""" DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=0 XCOMPILE=1 NO_TCL=1 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:CONSOLE,5.01 /LIBPATH:""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib"";""%CUSTOM_BINPATH%\tcl\lib""  ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib\libeay32.lib"" ""%CUSTOM_BINPATH%\tcl\lib\tcl86.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\include -I%CUSTOM_BINPATH%\tcl\include"
```

*TODO*: Some tests may fail because of the lack of SQLite extensions or other compiling flags, where I'll be working to run when I get some time.

First of, we can't compile SQLCipher shell and library altogether because of the `/SUBSYSTEM` link flag, which is required for this to work on Windows XP.
So, here we go compiling only the shell (patched version of sqlite3.exe with support to SQLCipher):

```batch
nmake -f Makefile.msc shell "NCC=""%VCINSTALLDIR%\bin\cl.exe""" USE_RC=0 OPTIMIZATIONS=9 DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=0 XCOMPILE=1 NO_TCL=1 PLATFORM=x86 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:CONSOLE,5.01 /LIBPATH:""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib"" ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib\libeay32.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\include"
```

If everything goes well, you should notice some new files, including `sqlite3.dll`, but you will not use it yet. Let's copy what we need to the lib/dist directory:

```batch
xcopy sqlite3.exe "..\sqlcipher-dist"
xcopy shell.obj "..\sqlcipher-dist"
```

Oki doki. Time to create the `libsqlite3.lib` file, always cleaning the build before:

```batch
nmake -f Makefile.msc clean
nmake -f Makefile.msc libsqlite3.lib "NCC=""%VCINSTALLDIR%\bin\cl.exe""" USE_RC=0 OPTIMIZATIONS=9 DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=0 XCOMPILE=1 NO_TCL=1 PLATFORM=x86 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:WINDOWS,5.01 /LIBPATH:""%CUSTOM_LIBBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib"" ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib\libeay32.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\include"
```

Now, you should have libsqlite3.lib in your path. Time to copy it to your dist:

```batch
xcopy libsqlite3.lib "..\sqlcipher-dist"
```

Nice! If everything went well so far, compiling the DLL should be as simple as this (let's not forget cleaning):

```batch
nmake -f Makefile.msc clean
nmake -f Makefile.msc dll "NCC=""%VCINSTALLDIR%\bin\cl.exe""" USE_RC=0 OPTIMIZATIONS=9 DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=0 XCOMPILE=1 NO_TCL=1 PLATFORM=x86 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:WINDOWS,5.01 /LIBPATH:""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib"" ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\lib\libeay32.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm\include"
```

Oh yeah! If you see a lot of `sqlite3.*` files, time to copy them and your dist is done!

```batch
xcopy sqlite3.* "..\sqlcipher-dist"
xcopy sqlite3ext.h "..\sqlcipher-dist"
```

Libraries, shell, headers ... Everything in one directory :)

Now, cleaning thing up ..

```batch
cd ..
```

## Building SQLCipher debug version

Nothing really changes between compiling the release version to the debug version, nevertheless the OpenSSL library used to compile the debug version of sqlcipher is in `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm`.

The commands are pretty straight-forward, like the release version:

```batch
rmdir /S /Q sqlcipher
"%PROGRAMFILES%\7-Zip\7z.exe" x sqlcipher.zip
cd sqlcipher
# building the shell
nmake -f Makefile.msc shell "NCC=""%VCINSTALLDIR%\bin\cl.exe""" SYMBOLS=1 OPTIMIZATIONS=0 DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=3 XCOMPILE=1 NO_TCL=1 PLATFORM=x86 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:CONSOLE,5.01 /LIBPATH:""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\lib"" ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\lib\libeay32.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\include"
# creating a directory for it
mkdir "%CUSTOM_LIBPATH%\sqlcipher-msvc2013-x86-debug"
xcopy sqlite3.exe "%CUSTOM_LIBPATH%\sqlcipher-msvc2013-x86-debug"
xcopy shell.obj "%CUSTOM_LIBPATH%\sqlcipher-msvc2013-x86-debug"
# clean up and build the lib
nmake -f Makefile.msc clean
nmake -f Makefile.msc libsqlite3.lib "NCC=""%VCINSTALLDIR%\bin\cl.exe""" SYMBOLS=1 OPTIMIZATIONS=0 DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=3 XCOMPILE=1 NO_TCL=1 PLATFORM=x86 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:WINDOWS,5.01 /LIBPATH:""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\lib"" ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\lib\libeay32.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\include"
xcopy libsqlite3.lib "%CUSTOM_LIBPATH%\sqlcipher-msvc2013-x86-debug"
# clean up and build the dll
nmake -f Makefile.msc clean
nmake -f Makefile.msc dll "NCC=""%VCINSTALLDIR%\bin\cl.exe""" SYMBOLS=1 OPTIMIZATIONS=0 DYNAMIC_SHELL=1 USE_NATIVE_LIBPATHS=1 TCLSH_CMD=tclsh USE_CRT_DLL=1 USE_ICU=0 DEBUG=3 XCOMPILE=1 NO_TCL=1 PLATFORM=x86 NSDKLIBPATH="%WIN_SDK_DIR%\Lib" UCRTLIBPATH="%WIN_DDK_DIR%\lib\Crt\i386" LTLINKOPTS="/SUBSYSTEM:WINDOWS,5.01 /LIBPATH:""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\bin"";""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\lib"" ""%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\lib\libeay32.lib""" "OPTS=-DSQLITE_HAS_CODEC -DSQLITE_TEMP_STORE=2 -D_USING_V120_SDK71_ -I%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm\include"
xcopy sqlite3.* "%CUSTOM_LIBPATH%\sqlcipher-msvc2013-x86-debug"
xcopy sqlite3ext.h "%CUSTOM_LIBPATH%\sqlcipher-msvc2013-x86-debug"
cd ..
```

Now you're probably done with some of the most weird command line options you've ever seen.

# TODO

This needs refinement.
