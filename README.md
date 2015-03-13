README of the VS2013 solution for FFMPEG 2.5.3

Disclaimer
==========

- Some of the sources of FFMPEG 2.5.3 were slightly modified to be able to make the build, nothing serious though. Typical changes are:
	- some additional defines
		#define HAVE_CL_CL_H 1
		#define inline __inline
		#define HAVE_STRUCT_POLLFD 1
	- some additional includes
		#include "libavutil/cpu.h"
		#include "libavutil/internal.h"
		#include "libavutil/intmath.h"
		#include "avcodec.h"
		#include "config.h"
- So you need not worry about those, just keep them in mind if you want to commit to Git, as it might break the build on other platforms (especially the defines, which would have been better incorporated in the project properties; I leave that to you).



Prerequisites
=============

- Visual Studio 2013, Update 4

- yasm.exe is not included in this distribution.
- You should have yasm.exe available somewhere in the PATH.
- If you have MinGW or MSYS2, then run mingw32_shell.bat, then type
	pacman -S yasm
at the mingw prompt. That should install the yasm package and produce yasm.exe in C:\msys32\usr\bin (depending on your msys directory)

- You should have OpenCL SDK for your CPU. For AMD, you need AMD APP SDK. For Intel, the equivalent, don't ask me.
- After having it installed, go to the Property Manager of VS2013 and open the property page Microsoft.Cpp.Win32.user and ensure the following paths point to your AMD APP SDK (or Intel equivalent):
	- VC++ Directories
		-> Include Directories = ..;C:\Program Files (x86)\AMD APP SDK\2.9-1\include;$(IncludePath)
		-> Library Directories = C:\Program Files (x86)\AMD APP SDK\2.9-1\lib\x86;$(LibraryPath)



The build
=========

The solution contains several projects. The preferred build order is:
libavutil
libswscale
libswresample
libavcodec
libavformat
libavfilter
libavdevice
ffmpeg

That's it. It should work.



Details of the solution configuration
=====================================

- Only bother to read this if you plan to make some further solution changes, which could affect the build
- The property page Microsoft.Cpp.Win32.user
	- VC++ Directories
		-> Include Directories = ..;C:\Program Files (x86)\AMD APP SDK\2.9-1\include;$(IncludePath)
		-> Library Directories = C:\Program Files (x86)\AMD APP SDK\2.9-1\lib\x86;$(LibraryPath)
	- C/C++
		-> General
			-> Debug Information Format = C7 compatible (/Z7)
			-> SDL checks = No
		-> Optimization
			-> Optimization = Maximize Speed (/O2)
		-> Preprocessor -> Preprocessor Definitions = HAVE_AV_CONFIG_H;_ISOC99_SOURCE;_USE_MATH_DEFINES;_CRT_SECURE_NO_WARNINGS;_WINDLL;%(PreprocessorDefinitions)
		-> Output Files -> Object File Name = $(IntDir)%(RelativeDir)\
	- Linker
		-> Input -> Additional Dependencies = opencl.lib;...
- The properties of each lib project
	- VC++ Directories
		-> Include Directories = <inherit>
		-> Library Directories = <inherit>
	- C/C++
		-> General
			-> Debug Information Format = <inherit>
			-> SDL checks = <inherit>
		-> Optimization
			-> Optimization = <inherit>
		-> Preprocessor -> Preprocessor Definitions = <inherit>
		-> Output Files -> Object File Name = <inherit>
	- Linker
		-> Input -> Additional Dependencies = <inherit>
- The properties of the project ffmpeg
	- VC++ Directories
		-> Include Directories = C:\Program Files (x86)\AMD APP SDK\2.9-1\include;$(IncludePath)
		-> Library Directories = $(Configuration);C:\Program Files (x86)\AMD APP SDK\2.9-1\lib\x86;$(LibraryPath)
	- Linker
		-> Input -> Additional Dependencies = vfw32.lib;strmiids.lib;ws2_32.lib;libavutil.lib;libswscale.lib;libswresample.lib;libavcodec.lib;libavformat.lib;libavfilter.lib;libavdevice.lib;opencl.lib;...
- The properties of each .asm source
	- General -> Item Type -> Custom Build Tool
	- Custom Build Tool -> General
		-> Command Line:
			yasm -f win32 -DPREFIX -P ..\config.asm -o $(IntDir)%(RelativeDir)\%(Filename).obj -i $(SolutionDir)/ -i $(ProjectDir)/x86 %(FullPath)
		-> Outputs:
			$(IntDir)%(RelativeDir)\%(Filename).obj
- Remember PATH must contain the path to yasm.exe, see above
