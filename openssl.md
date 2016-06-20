# OpenSSL - "Cryptography and SSL/TLS Toolkit"

Well, this is a tricky one. OpenSSL is the main toolkit for cryptography these days (and for a long time), but to create binaries suitable for Windows ...

The greatest effort I've seen was from [this article](http://hostagebrain.blogspot.com.br/2015/04/build-openssl-on-windows.html), but I just found it when I was polishing linking errors with NASM. I will provide some more tips from this article as they're needed.

## Requirements

There are some requirements to compile OpenSSL under Windows:

* The [OpenSSL source code](https://www.openssl.org/source/) (obviously). We will be working with version 1.0.2h, the latest stable release ATM (Jun 18, 2016);
* Perl: the main configuration file for OpenSSL is written in Perl, so the same is needed to run and configure OpenSSL (something like the `./configure` script);
* [NASM](http://www.nasm.us/pub/nasm/releasebuilds/?C=M;O=D) (optional): compile assembly speedups for the x86 platform only. Download it from the link provided and make sure `nasm.exe` is found in your `%PATH%`.

## First steps

We will be creating builds with and without ASM, for release or debug. The following table explains paths and what will be installed into each one of them:

| Path | [N]ASM | Debug or Release | Static or Dynamic |
|------|--------|------------------|-------------------|
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm` | Yes | Debug | Dynamic |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm` | Yes | Release | Dynamic |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-no_asm` | No | Debug | Dynamic |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-no_asm` | No | Release | Dynamic |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm-static` | Yes | Debug | Static |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm-static` | Yes | Release | Static |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-no_asm-static` | No | Debug | Static |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-no_asm-static` | No | Release | Static |

## Setup

I always like to compile source code from a clean, fresh copy of its (official source) distribution. So, I always delete (if path already existent) and unpack the source code package.

We'll start always from a fresh source code directory. First, we'll compile dynamic libraries (DLL) as they are, IMHO, what developers most want (in most of the libs I would say, OpenSSL might be an exception).

## Creating DLLs

Ok, in here we will be creating an OpenSSL binary distribution to use as dynamic linking.

### "Remember, remember the 5th of November"

As I already told in the [README](https://github.com/vltr/msvc-mercy), all binaries here are required to run in a Windows XP environment, x86.

### ASM + Debug

```batch
perl Configure debug-VC-WIN32 --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm"
ms\do_nasm
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

### ASM + Release

```batch
perl Configure VC-WIN32 --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm"
ms\do_nasm
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

### NO_ASM + Debug

```batch
perl Configure debug-VC-WIN32 no-asm --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-no_asm"
ms\do_ms
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

### NO_ASM + Release

```batch
perl Configure VC-WIN32 no-asm --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-no_asm"
ms\do_ms
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

## Creating static binaries

TODO:
- this
- Windows XP compat
