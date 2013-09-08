OUYA/Android/Win32 port of Second Reality
=========================================

This is a port of Second Reality by Future Crew updated to run on present-day platforms.  The OUYA is the focus, but should work fine on any other Android 2.3 device or higher.  See below for Win32 support.

**Current status**:

- Part 1 (alku): Renders and executes fully.  Bugs: Text does not scroll properly (needs ascrolltext() function implemented).
- Part 2 (u2a): Not yet implemented.
- Part 3 (pam): Screen flash and fade renders fully, explosion effect partially works.  Bugs: Graphic corruption, and incorrect page flipping.
- Part 4 (beg): Renders and executes fully.
- Part 15 (dots): Renders and executes fully.
- No audio playback code has been written yet.
- Timing and synchronization is not perfect.

![dots part running on OUYA](http://falken42.github.com/sr6.jpg)

*Commit 9f74af7 running on the OUYA, after adding dots part*

To build the Android version, run 'make' in the source folder.  You will need the Android NDK with API 10 installed.  The built .apk will automatically be uploaded to an attached device (via adb install) if it is connected.

The Win32 version is currently in an experimental state, but is functional.  To build with Visual Studio 2010, open the solution in the source/win32 folder and execute.

If you are interested in helping, Pull Requests are welcomed.


Porting Notes
=============

This port implements a VGA hardware emulation layer through function calls that are made by the demo at the C library level.  This allows the demo to run natively while making minimal changes to the original source code.  Most of the VGA emulation is already complete, however, converting the demo's assembly code into functionally equivalent C code is taking the most time.

The emulation layer works by providing functions that were used on DOS, but are no longer found in today's C libraries.  For example, the macro MK\_FP() which is used to create a far pointer to a memory block is now a function, and detects if the memory block being requested is in the DOS VGA memory area (A000:0000) or not.  If it is, it returns a pointer to an emulated VGA memory buffer instead, which is later used in rendering.

Another function, outport(), is used to directly access the VGA hardware.  This function is provided by the emulation layer, and holds the state of the various VGA registers.  When a frame is to be displayed, the stored VGA register values control how the VGA memory buffers should be interpreted.

To actually render a frame, the demo calls an internal function called dis\_exit() to determine if the ESC key has been pressed and the demo should quit.  This function is called throughout the entire code, and is the main hook used for rendering.  After a simulated VBlank has passed (16.667ms / 60fps), the current VGA buffers and palette are combined into a frame buffer, uploaded as a texture to OpenGL, and rendered as a fullscreen polygon.

Instead of handling each part as a separate executable, this port compiles and statically links all of the parts together and calls them directly in the order of the original demo.


Multiplatform Support
=====================

All of the platform-specific code has been abstracted away in its own interface.  If you wish to add a new platform, implement the functions in source/jni/platform.h in a separate source file and add that source file to your build system.  Implement main() or whatever entry point function is required for your platform, and call the demo\_execute() function listed in u2-port.h to run the demo.

Note that the Second Reality code was originally developed for DOS and is non-reentrant.  Once you call demo\_execute(), it will **not return** until after the entire demo has completed.  The only idle callback function provided is through platform\_handle\_events(), but if your platform cannot render properly without returning from its caller, then this port will not work on your platform.  It might be possible to work around this limitation by using fibers, however.  Note that there are no plans to overhaul the original code to support reentrancy.


Older Screenshots
=================

![Running on a Kindle Fire HD](http://falken42.github.com/sr.jpg)

*Commit a762b8f running on a Kindle Fire HD, before Mode X support*

![Running on a Kindle Fire HD](http://falken42.github.com/sr2.jpg)

*Commit 2e7c444 running on a Kindle Fire HD, with basic Mode X support*

![Running on a Kindle Fire HD](http://falken42.github.com/sr3.jpg)

*Commit fec75a6 running on a Kindle Fire HD, with improved VGA support and scrolling*

![Running on a Kindle Fire HD](http://falken42.github.com/sr4.jpg)

*Commit 52a9fac running on a Kindle Fire HD, after various bug fixes*

![Running on a Kindle Fire HD](http://falken42.github.com/sr5.jpg)

*Commit c8da9a9 running on a Kindle Fire HD, after adding beg part*


License
=======

The license for this port is the Unlicense, and is the same as the original Second Reality source ( https://github.com/mtuomi/SecondReality ):

This is free and unencumbered software released into the public domain.

Anyone is free to copy, modify, publish, use, compile, sell, or
distribute this software, either in source code form or as a compiled
binary, for any purpose, commercial or non-commercial, and by any
means.

In jurisdictions that recognize copyright laws, the author or authors
of this software dedicate any and all copyright interest in the
software to the public domain. We make this dedication for the benefit
of the public at large and to the detriment of our heirs and
successors. We intend this dedication to be an overt act of
relinquishment in perpetuity of all present and future rights to this
software under copyright law.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS BE LIABLE FOR ANY CLAIM, DAMAGES OR
OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

For more information, please refer to <http://unlicense.org/>
