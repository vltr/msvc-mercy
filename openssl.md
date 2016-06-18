# OpenSSL - "Cryptography and SSL/TLS Toolkit"

Well, this is a tricky one. OpenSSL is the main toolkit for cryptography these days (and for a long time), but to create binaries suitable for Windows ...

The greatest effort I've seen was from [this article](http://hostagebrain.blogspot.com.br/2015/04/build-openssl-on-windows.html), but I just found it when I was polishing linking errors.

## Requirements

There are some requirements to compile OpenSSL under Windows:

* The [OpenSSL source code](https://www.openssl.org/source/) (obviously). We will be working with version 1.0.2h, the latest stable release ATM (Jun 18, 2016);
* Perl: the main configuration file for OpenSSL is written in Perl, so the same is needed to run and configure OpenSSL (something like the `./configure` script);
* [NASM](http://www.nasm.us/pub/nasm/releasebuilds/?C=M;O=D) (optional): compile assembly speedups for the x86 platform only. Download it from the link provided and make sure `nasm.exe` is found in your `%PATH%`.

## First steps

We will be creating builds with and without ASM, for release or debug. The following table explains paths and what will be installed into each one of them:

| Path | [N]ASM | Debug or Release |
|------|--------|------------------|
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm` | Yes | Debug |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm` | Yes | Release |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-no_asm` | No | Debug |
| `%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-no_asm` | No | Release |

## Setup

I always like to compile source code from a clean, fresh copy of its distribution. So, I always delete (if path already existent) and unpack the source code package.

We'll start always from a fresh source code directory. For now, I'll just leave the snippets and **all libraries will be build as DLL**.

## Compile - ASM + Debug

```batch
perl Configure debug-VC-WIN32 --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-nasm"
ms\do_nasm
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

## Compile - ASM + Release

```batch
perl Configure VC-WIN32 --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-nasm"
ms\do_nasm
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

## Compile - NO_ASM + Debug

```batch
perl Configure debug-VC-WIN32 no-asm --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-debug-no_asm"
ms\do_ms
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

## Compile - NO_ASM + Release

```batch
perl Configure VC-WIN32 no-asm --prefix="%CUSTOM_LIBPATH%\openssl-1.0.2h-msvc2013-x86-release-no_asm"
ms\do_ms
nmake -f ms\ntdll.mak
nmake -f ms\ntdll.mak install
```

# Quick notes

There are some important items I still have to provide, like binary compatibility with Windows XP.
