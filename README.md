# mscv-mercy

Sometimes, all you need is a little mercy. Why, why oh why things should be so tricky and complicated to do by using Windows, specially for C/C++ development? "Oh, Windows 10 has bash now". But ... There are still people that needs to build applications compatible with Windows XP.

Yep. Windows XP. If I recall, I was a kid when Microsoft launched Windows XP.

Well, let's say you had a situation probably resembling mine:

* You'll be using a mature C++ framework to develop cross-platform solutions. I chose Qt;
* You have been developing for a long time now to know that you don't need to reinvent the wheel (it doesn't mean you won't suffer);
* You're familiar with Linux / Unix, but got yourself in the tricky world of compiling things ... for Windows;
* All you want is a little mercy.

# A few notes before we begin

I'll make a brief list of what I have installed in my development box:

* Windows 7 Professional x86;
* Visual Studio 2013 Express update 5;
* Qt 5.[6.1|7.0];
* Windows SDK 7.1 [download ISO here - GRMSDK_EN_DVD is for x86](https://www.microsoft.com/en-us/download/details.aspx?id=8442);
* Windows Driver Kit 7.1 [download iso here - GRMWDK_EN_7600_1](https://www.microsoft.com/en-us/download/details.aspx?id=11800);

_NOTE_: Microsoft tends to change links as the years go by. Searching those ISO files in Google might help you, or even torrent. Be warned that you might end downloading something else - sometimes harmful - from untrusted sources.

## Visual Studio 2010 notes

To be able to compile things with Qt versions prior to 5.6 in VS2010, you need to have things installed in a specific order. Things might blow if you don't (might, seriously):

0. Visual Studio 2010 Express [download ISO here](http://download.microsoft.com/download/1/E/5/1E5F1C0A-0D5B-426A-A603-1798B951DDAE/VS2010Express1.iso);
0. Windows SDK 7.1 [download ISO here - GRMSDK_EN_DVD is for x86](https://www.microsoft.com/en-us/download/details.aspx?id=8442);
0. Visual Studio 2010 SP1 [download ISO here](http://go.microsoft.com/fwlink/?LinkId=210710);
0. Visual C++ 2010 SP1 Compiler Update for the Windows SDK 7.1 [download installer here](https://www.microsoft.com/en-us/download/details.aspx?id=4422);

_REMEMBER_: I used VS2010 in the past. I just let this as a note. This document is absolutely based on my box config!

# Let's create binaries!

... well, not so fast. First, let's organize ourselves by defining some user / environment variables, so when I refer to `WIN_DDK_DIR` you'll know what I'm talking about.

I'll put here some of my system variables with a brief explanation about each one:

| Variable name | Value | Example |
|---------------|-------|---------|
| `DIRECTX_SDK_DIR` | DirectX SDK Directory, _if needed_ | `C:\Program Files\Microsoft DirectX SDK (June 2010)` |
| `CUSTOM_BINPATH` | The path to my own set of third party executables | `E:\bin` |
| `CUSTOM_IMPORTPATH` | The path to my own set of third party source (code) imports | `E:\includes` |
| `CUSTOM_LIBPATH` | The path to my own set of compiled libs | `E:\lib` |
| `WIN_DDK_DIR` | The path to the Windows Driver Development Kit, _if needed_ | `C:\WinDDK\7600.16385.1` |
| `WIN_SDK_DIR` | The path to the Windows Software Development Kit, _if needed_ | `C:\Program Files\Microsoft SDKs\Windows\v7.1` |

Some other tools I have installed at `%CUSTOM_BINPATH%` or `%PROGRAMFILES%` and with its binary file in my `%PATH%`:

* [ActivePerl x86](http://www.activestate.com/activeperl)
* [ActiveTCL x86](http://www.activestate.com/activetcl/downloads)
* [NASM](http://www.nasm.us/pub/nasm/releasebuilds/?C=M;O=D)
* [Python 2.7 x86](http://www.python.org/download/)
* [Jom](https://wiki.qt.io/Jom) (optional)

## You shall not forget ...

... that, in order to successfully compile something for Windows XP, you will be using these flags **A LOT**:

`/SUBSYSTEM:WINDOWS,5.01`
`/SUBSYSTEM:CONSOLE,5.01`

Why? Because the binary signature created using newer versions [of Windows / Visual C++ / whatever] will not work with Windows XP, you'll probably get an error like this:

![Not a valid WHAT?](https://raw.githubusercontent.com/vltr/msvc-mercy/master/assets/not_a_valid_win32_app.png)

`<whatever_bin.exe> is not a valid Win32 application`.

Sweeeeeeeeeet ... Not. The solution, however, is quite interesting. There are two ways to "fix" this:

0. You do it right and add the valid `/SUBSYSTEM:...,5.01` in your flags; or ...
0. You can also edit your executable using any hexeditor. Really.

It is quite funny, because you just need to change a couple of bytes and it'll work (probably). Using an editor with 16 bytes per row, offset in hex, probably around row 140, you find two pairs of three bytes, quite near to each other, probably with these values: `06 00 00`. This means that at least Windows API 6.00 is required to run the executable.
So, the fix is simple: under Windows XP x86, you'll change from `06 00 00` to `05 00 01`, which will be written to the binary if you use the `/SUBSYSTEM:..,5.01` flag. I would say this is the best approach :) Yet, here's what it looks like in the hex editor:

![Before edit](https://raw.githubusercontent.com/vltr/msvc-mercy/master/assets/hex_edit_01.png)

I have highlighted the areas where the signature matches. After editing:

![After edit](https://raw.githubusercontent.com/vltr/msvc-mercy/master/assets/hex_edit_02.png)

Now, by having these changes, your executable will run under Windows XP (5.01 is for x86, 5.02 is for Windows XP 64). Cool, huh?

# "The time has come to shine"

The list of what I already had compiled under Windows is quite big, so I'll keep here a list to some of the tricky ones:

* ICU[4C] - "International Components for Unicode"
* [OpenSSL](https://github.com/vltr/msvc-mercy/blob/master/openssl.md) - "Cryptography and SSL/TLS Toolkit"
* Qt 5.X
* [SQLCipher](https://github.com/vltr/msvc-mercy/blob/master/sqlcipher.md) - "Full Database Encryption for SQLite"
* [SQLCipher Qt5 SQL plugin](https://github.com/vltr/msvc-mercy/blob/master/qsqlcipher_plugin.md) - or see [this](https://github.com/vltr/qt5-sqlcipher) repository.

## Would you like to have instructions to compile something?

Please, open an issue or contribute by giving tips to issues. *Remember*, I'm just documenting the process of creating binaries under MSVC 2013, not MSVC 20XX nor MinGW.

## Would you like to contribute?

Just drop here a pull request :) Remember to see if you agree to this repository LICENSE!

# License

All documents created by me are released here [under the GNU GENERAL PUBLIC LICENSE Version 3](https://github.com/vltr/msvc-mercy/blob/master/LICENSE).
