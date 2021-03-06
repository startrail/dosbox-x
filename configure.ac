dnl Init.
AC_INIT(dosbox-x,0.81,[https://github.com/joncampbell123/dosbox-x/issues],[],[http://dosbox-x.software])
AC_PREREQ(2.50)
AC_CONFIG_SRCDIR(README)

dnl Detect the canonical host and target build environment
AC_CANONICAL_HOST
AC_CANONICAL_BUILD

dnl Setup for automake
AM_INIT_AUTOMAKE([foreign])
AC_CONFIG_HEADER(config.h)

dnl Checks for programs.
AC_PROG_MAKE_SET
AC_PROG_CC
AC_PROG_CPP
AC_PROG_CXX
AC_PROG_INSTALL
AC_PROG_RANLIB

dnl TODO where else to put this?

# AC_CHECK_CPPFLAGS(ADDITIONAL-CPPFLAGS, ACTION-IF-FOUND, ACTION-IF-NOT-FOUND)
#
# checks whether the $(C) compiler accepts the ADDITIONAL-CPPFLAGS
# if so, they are added to the CPPFLAGS
AC_DEFUN([AC_CHECK_CPPFLAGS],
[
  AC_MSG_CHECKING([whether $CPP accepts "$1"])
  temp_check_cppflags="${CPPFLAGS}"
  CPPFLAGS="$1 ${CPPFLAGS}"
  AC_PREPROC_IFELSE(
        [AC_LANG_SOURCE([[int main(void){return 0;}]])],
        [AC_MSG_RESULT([yes])],
        [AC_MSG_RESULT([no]); CPPFLAGS="${temp_check_cppflags}"])
])# AC_CHECK_CPPFLAGS

# AC_CHECK_CXXFLAGS(ADDITIONAL-CXXFLAGS, ACTION-IF-FOUND, ACTION-IF-NOT-FOUND)
#
# checks whether the $(CXX) (c++) compiler accepts the ADDITIONAL-CXXFLAGS
# if so, they are added to the CXXFLAGS
AC_DEFUN([AC_CHECK_CXXFLAGS],
[
  AC_MSG_CHECKING([whether $CXX accepts "$1"])
cat > conftest.c++ << EOF
int main(){
  return 0;
}
EOF
if $CXX $CPPFLAGS $CXXFLAGS -o conftest.o conftest.c++ [$1] > /dev/null 2>&1
then
  AC_MSG_RESULT([yes])
  CXXFLAGS="${CXXFLAGS} [$1]"
  AC_CHECK_CPPFLAGS([$1])
  [$2]
else
  AC_MSG_RESULT([no])
  [$3]
fi
])# AC_CHECK_CXXFLAGS

dnl Need GCC to know this code is using C++ lamda functions
CXXFLAGS="$CXXFLAGS -Wall -D_FILE_OFFSET_BITS=64"

AH_TEMPLATE(C_MT32,[Define to 1 to enable MT32 emulation (x86/x86_64 only)])
AC_ARG_ENABLE(mt32,AC_HELP_STRING([--disable-mt32],[Disable MT32 emulation]),,enable_mt32=yes)
AC_MSG_CHECKING(whether MT32 emulation will be enabled) 
if test x$enable_mt32 = xyes ; then 
  case "$host_cpu" in
    x86_64 | amd64)
      AC_MSG_RESULT(yes)
      AC_DEFINE(C_MT32,1)
      ;;
    i?86)
      AC_MSG_RESULT(yes)
      AC_DEFINE(C_MT32,1)
      ;;
    *)
      enable_mt32=no
      AC_MSG_RESULT(no)
      ;;
  esac
else
  enable_mt32=no
  AC_MSG_RESULT(no)
fi 
AM_CONDITIONAL(C_MT32, test "x$enable_mt32" == "xyes")

dnl Scalars in the code use MMX/SSE intrinsics
case "$host_cpu" in
  x86_64 | amd64)
    CXXFLAGS="$CXXFLAGS -mmmx -msse -msse2"
    ;;
  i?86)
    CXXFLAGS="$CXXFLAGS -mmmx -msse -msse2"
    ;;
esac

dnl Stop reporting "type punning" warnings, I'm sick of hearing about it.
CXXFLAGS="$CXXFLAGS -Wno-strict-aliasing"

dnl Some needed libaries for OS2
dnl perharps join this with the other target depended checks. move them upwards
if test x$host = xi386-pc-os2-emx ; then
    CXXFLAGS="$CXXFLAGS -Zmt"
    LDFLAGS="$LDFLAGS -Zomf -Zmt"
    LIBS="$LIBS -los2me"
fi

dnl Check for SDL
SDL_VERSION=1.2.0
AM_PATH_SDL($SDL_VERSION,
            :,
	    AC_MSG_ERROR([*** SDL version $SDL_VERSION not found!])
)
LIBS="$LIBS $SDL_LIBS"
CPPFLAGS="$CPPFLAGS $SDL_CFLAGS"

dnl Check if SDL is 1.2.x (1.3 not supported)
AC_MSG_CHECKING([SDL version only being 1.2.X])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include "SDL.h"
void blah(){
#if SDL_MINOR_VERSION != 2
#error "Only SDL 1.2 supported"
#endif
;
}
])],AC_MSG_RESULT([yes]),[
 AC_MSG_RESULT([no]) 
 AC_MSG_ERROR([Only libSDL 1.2.X supported])])

dnl Checks for header files.

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_C_INLINE
AC_TYPE_SIZE_T
AC_STRUCT_TM
AC_CHECK_SIZEOF(unsigned char)
AC_CHECK_SIZEOF(unsigned short)
AC_CHECK_SIZEOF(unsigned int)
AC_CHECK_SIZEOF(unsigned long)
AC_CHECK_SIZEOF(unsigned long long)
AC_CHECK_SIZEOF(int *)

dnl some semi complex check for sys/socket so it works on darwin as well
AC_CHECK_HEADERS([stdlib.h sys/types.h])
AC_CHECK_HEADERS([sys/socket.h  netinet/in.h pwd.h], [], [],
[#include <stdio.h>
#ifdef STDC_HEADERS
# include <stdlib.h>
# include <stddef.h>
#else
# ifdef HAVE_STDLIB_H
#  include <stdlib.h>
# endif
#endif
#ifdef HAVE_SYS_TYPES_H
# include <sys/types.h>
#endif
])

dnl check for the socklen_t (darwin doesn't always have it)
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <stdio.h>
#ifdef STDC_HEADERS
# include <stdlib.h>
# include <stddef.h>
#else
# ifdef HAVE_STDLIB_H
#  include <stdlib.h>
# endif
#endif
#ifdef HAVE_SYS_TYPES_H
# include <sys/types.h>
#endif
#ifdef HAVE_SYS_SOCKET_H
#include <sys/socket.h>
#endif
])],[],[AC_DEFINE([socklen_t],[int],[Define to `int` if you don't have socklen_t])])

dnl AC_CHECK_CXXFLAGS will prepend gnu++11 to CXXFLAGS if supported, for us.
dnl our code adds -std=gnu++0x if not supported where the function would not do anything.
AC_MSG_CHECKING([checking for C++11 support])
AC_CHECK_CXXFLAGS([ -std=gnu++11 ], [  ], [ CXXFLAGS="$CXXFLAGS -std=gnu++0x" ])

AC_MSG_CHECKING(if environ can be included)
AC_LINK_IFELSE([AC_LANG_PROGRAM([[
#include <unistd.h>
#include <stdlib.h>]],[[*environ;]])],
[AC_MSG_RESULT(yes);AC_DEFINE(ENVIRON_INCLUDED,1,[environ can be included])],AC_MSG_RESULT(no))

AC_MSG_CHECKING(if environ can be linked)
AC_LINK_IFELSE([AC_LANG_PROGRAM([[extern char ** environ;]],[[*environ;]])],
[AC_MSG_RESULT(yes);AC_DEFINE(ENVIRON_LINKED,1,[environ can be linked])],AC_MSG_RESULT(no))

AC_MSG_CHECKING([if dirent includes d_type])
AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <sys/types.h>
#include <dirent.h>
void blah(){
struct dirent d_test;
d_test.d_type = 0;
}])],[AC_MSG_RESULT(yes);AC_DEFINE(DIRENT_HAS_D_TYPE,1,[struct dirent has d_type])],AC_MSG_RESULT(no))


dnl Check for powf
AC_MSG_CHECKING(for powf in libm);
LIBS_BACKUP=$LIBS;
LIBS="$LIBS -lm";
AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <math.h>]],[[
        powf(1.0f, 1.0f);
]])], [AC_MSG_RESULT(yes)], [AC_DEFINE([DB_HAVE_NO_POWF],[1],[libm doesn't include powf])])
LIBS=$LIBS_BACKUP


dnl Checks for libraries.

#Check if the compiler support attributes
AH_TEMPLATE([C_HAS_ATTRIBUTE],[Determines if the compilers supports attributes for structures.])
AC_MSG_CHECKING(if compiler allows __attribute__)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
typedef struct { } __attribute__((packed)) junk;]],
[[ ]])],[ AC_MSG_RESULT(yes);AC_DEFINE(C_HAS_ATTRIBUTE)],AC_MSG_RESULT(no))


#Check if the compiler supports certain attributes
OLDCFLAGS="$CFLAGS"
CFLAGS="-Werror"

AH_TEMPLATE([C_ATTRIBUTE_ALWAYS_INLINE],[Determines if the compilers supports always_inline attribute.])
AC_MSG_CHECKING(if compiler allows __attribute__((always_inline)) )
AC_COMPILE_IFELSE([AC_LANG_SOURCE([ void __attribute__((always_inline)) test(){}
])],[ AC_MSG_RESULT(yes);AC_DEFINE(C_ATTRIBUTE_ALWAYS_INLINE)],AC_MSG_RESULT(no))

AH_TEMPLATE([C_ATTRIBUTE_FASTCALL],[Determines if the compilers supports fastcall attribute.])
AC_MSG_CHECKING(if compiler allows __attribute__((fastcall)) )
AC_COMPILE_IFELSE([AC_LANG_SOURCE([ void __attribute__((fastcall)) test(){}
])],[ AC_MSG_RESULT(yes);AC_DEFINE(C_ATTRIBUTE_FASTCALL)],AC_MSG_RESULT(no))


CFLAGS="$OLDCFLAGS"


#Check if the compiler supports __builtin_expect
#Switch language to c++
AC_LANG_PUSH(C++)
AH_TEMPLATE([C_HAS_BUILTIN_EXPECT],[Determines if the compilers supports __builtin_expect for branch prediction.])
AC_MSG_CHECKING(if compiler allows __builtin_expect)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]],[[
int x=10;if( __builtin_expect ((x==1),0) ) ;
]])], [ AC_MSG_RESULT(yes);AC_DEFINE(C_HAS_BUILTIN_EXPECT)],AC_MSG_RESULT(no))
#switch language back
AC_LANG_POP(C++)

dnl enable disable alsa and pass it's cflags to CXXFLAGS
AC_ARG_ENABLE(alsa-midi,
AC_HELP_STRING([--enable-alsa-midi],[compile with alsa midi support (default yes)]),
[ case "${enableval}" in
 yes) alsa_midi=true;;
 no)  alsa_midi=false;;
esac],
[alsa_midi=true])
if test x$alsa_midi = xtrue ; then 
  AM_PATH_ALSA(0.9.0, AC_DEFINE(HAVE_ALSA,1,[Define to 1 to use ALSA for MIDI]) , : )
  CXXFLAGS="$CXXFLAGS $ALSA_CFLAGS"
fi

#Check for big endian machine, should #define WORDS_BIGENDIAN if so
AC_C_BIGENDIAN

#Features to enable/disable
AH_TEMPLATE(C_DEBUG,[Define to 1 to enable internal debugger, requires libcurses])
AH_TEMPLATE(C_HEAVY_DEBUG,[Define to 1 to enable heavy debugging, also have to enable C_DEBUG])
AC_ARG_ENABLE(debug,AC_HELP_STRING([--enable-debug],[Enable debug mode]),[
   AC_CHECK_HEADER(curses.h,have_curses_h=yes,)
   AC_CHECK_LIB(curses, initscr, have_curses_lib=yes, , )
   AC_CHECK_LIB(ncurses, initscr, have_ncurses_lib=yes, , )
   AC_CHECK_LIB(pdcurses, initscr, have_pdcurses_lib=yes, , )

   if test x$enable_debug = xno; then
     AC_MSG_RESULT([Debugger not enabled])
   elif test x$have_curses_lib = xyes -a x$have_curses_h = xyes ; then
     LIBS="$LIBS -lcurses"
     AC_DEFINE(C_DEBUG,1)
     if test x$enable_debug = xheavy ; then 
       AC_DEFINE(C_HEAVY_DEBUG,1)
     fi
   elif test x$have_ncurses_lib = xyes -a x$have_curses_h = xyes ; then
     LIBS="$LIBS -lncurses"
     AC_DEFINE(C_DEBUG,1)
     if test x$enable_debug = xheavy ; then 
       AC_DEFINE(C_HEAVY_DEBUG,1)
     fi
   elif test x$have_pdcurses_lib = xyes -a x$have_curses_h = xyes ; then
     LIBS="$LIBS -lpdcurses"
     AC_DEFINE(C_DEBUG,1)
     if test x$enable_debug = xheavy ; then 
       AC_DEFINE(C_HEAVY_DEBUG,1)
     fi
   else 
     AC_MSG_ERROR([Can't find curses, which is required for debug mode])
   fi
],)

AH_TEMPLATE(C_CORE_INLINE,[Define to 1 to use inlined memory functions in cpu core])
AC_ARG_ENABLE(core-inline,AC_HELP_STRING([--enable-core-inline],[Enable inlined memory handling in CPU Core]),[
  if test x$enable_core_inline = xyes ; then 
    AC_MSG_RESULT([enabling inlined memory handling in CPU Core])
    AC_DEFINE(C_CORE_INLINE,1)
  fi
],)

dnl The target cpu checks for dynamic cores
AH_TEMPLATE(C_TARGETCPU,[The type of cpu this target has])
AC_MSG_CHECKING(for target cpu type) 
case "$host_cpu" in
  x86_64 | amd64)
    AC_DEFINE(C_TARGETCPU,X86_64)
    AC_MSG_RESULT(x86-64 bit compatible)
    c_targetcpu="x86_64"
    c_unalignedmemory=yes
    ;;
  i?86)
    AC_DEFINE(C_TARGETCPU,X86)
    AC_MSG_RESULT(x86 compatible)
    c_targetcpu="x86"
    c_unalignedmemory=yes
    ;;
   powerpc*)
    AC_DEFINE(C_TARGETCPU,POWERPC)
    AC_MSG_RESULT(Power PC)
    c_targetcpu="powerpc"
    c_unalignedmemory=yes
    ;;
   m68k*)
    AC_DEFINE(C_TARGETCPU,M68K)
    AC_MSG_RESULT(Motorola 68000)
    c_targetcpu="m68k"
    c_unalignedmemory=yes
    ;;
   *)
    AC_DEFINE(C_TARGETCPU,UNKNOWN)
    AC_MSG_RESULT(unknown)
    c_unalignedmemory=no
    ;;
esac

AC_ARG_ENABLE(dynamic-core,AC_HELP_STRING([--disable-dynamic-core],[Disable all dynamic cores]),,enable_dynamic_core=yes)

AH_TEMPLATE(C_DYNAMIC_X86,[Define to 1 to use x86 dynamic cpu core])
AC_ARG_ENABLE(dynamic-x86,AC_HELP_STRING([--disable-dynamic-x86],[Disable x86 dynamic cpu core]),,enable_dynamic_x86=yes)
AC_MSG_CHECKING(whether x86 dynamic cpu core will be enabled) 
if test x$enable_dynamic_x86 = xno -o x$enable_dynamic_core = xno; then 
   AC_MSG_RESULT(no)
else
  if test x$c_targetcpu = xx86 ; then
      AC_DEFINE(C_DYNAMIC_X86,1)
      AC_MSG_RESULT(yes)
  else
      AC_MSG_RESULT(no)
  fi
fi

AH_TEMPLATE(C_FPU,[Define to 1 to enable floating point emulation])
AC_ARG_ENABLE(fpu,AC_HELP_STRING([--disable-fpu],[Disable fpu support]),,enable_fpu=yes)
AC_MSG_CHECKING(whether fpu emulation will be enabled) 
if test x$enable_fpu = xyes ; then 
  AC_MSG_RESULT(yes)
  AC_DEFINE(C_FPU,1)
else 
  AC_MSG_RESULT(no)
fi 

AH_TEMPLATE(C_UNALIGNED_MEMORY,[Define to 1 to use a unaligned memory access])
AC_ARG_ENABLE(unaligned_memory,AC_HELP_STRING([--disable-unaligned-memory],[Disable unaligned memory access]),,enable_unaligned_memory=yes)
AC_MSG_CHECKING(whether to enable unaligned memory access) 
if test x$enable_unaligned_memory = xyes -a x$c_unalignedmemory = xyes; then 
  AC_DEFINE(C_UNALIGNED_MEMORY,1)
  AC_MSG_RESULT(yes)
else
  AC_MSG_RESULT(no)
fi

dnl Some target detection and actions for them
case "$host" in
    *-*-darwin*)
       dnl El Capitan's refusal to allow sudo make install to /usr/include, and the failure of most open source to
       dnl include from /usr/local/include is irritating and tiresome...
       CFLAGS="$CFLAGS -I/usr/local/include"
       CPPFLAGS="$CPPFLAGS -I/usr/local/include"
       ;;
esac

AH_TEMPLATE(C_LIBPNG,[Define to 1 if you have libpng])
AH_TEMPLATE(C_SSHOT,[Define to 1 to enable screenshots, requires libpng])
AC_CHECK_HEADER(png.h,have_png_h=yes,)
AC_CHECK_LIB(png, png_get_io_ptr, have_png_lib=yes, ,-lz)
if test x$have_png_lib = xyes -a x$have_png_h = xyes ; then
  LIBS="$LIBS -lpng -lz"
  AC_DEFINE(C_LIBPNG,1)
  AC_DEFINE(C_SSHOT,1)
else
  AC_MSG_WARN([Can't find libpng, screenshot support disabled])
fi

AH_TEMPLATE(C_NE2000,[Define to 1 to enable NE2000 ethernet passthrough, requires libpcap])
AC_CHECK_HEADER(pcap.h,have_pcap_h=yes,)
AC_CHECK_LIB(pcap, pcap_open_live, have_pcap_lib=yes, ,-lz)
if test x$have_pcap_lib = xyes -a x$have_pcap_h = xyes ; then
  LIBS="$LIBS -lpcap";
  AC_DEFINE(C_NE2000,1)
else
  AC_MSG_WARN([Can't find libpcap, NE2000 ethernet passthrough disabled])
fi

AH_TEMPLATE(C_SDL_NET,[Indicate whether SDL_net is present])
AH_TEMPLATE(C_MODEM,[Define to 1 to enable internal modem support, requires SDL_net])
AH_TEMPLATE(C_IPX,[Define to 1 to enable IPX over Internet networking, requires SDL_net])
AC_CHECK_HEADER(SDL_net.h,have_sdl_net_h=yes,)

if test x$host = xi386-pc-os2-emx ; then
  AC_MSG_CHECKING(for SDLNet_Init in SDL_net);
  LIBS_BACKUP=$LIBS;
  LIBS="$LIBS -lSDL_Net";
  AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <SDL_Net.h>]],[[
	SDLNet_Init ();
  ]])], [AC_MSG_RESULT(yes); have_sdl_net_lib=yes], AC_MSG_RESULT(no))
  LIBS=$LIBS_BACKUP
else
AC_CHECK_LIB(SDL_net, SDLNet_Init, have_sdl_net_lib=yes, , )
fi
if test x$have_sdl_net_lib = xyes -a x$have_sdl_net_h = xyes ; then
   LIBS="$LIBS -lSDL_net"
   AC_DEFINE(C_SDL_NET,1)
   AC_DEFINE(C_MODEM,1)
   AC_DEFINE(C_IPX,1)
else 
   AC_MSG_WARN([Can't find SDL_net, internal modem and ipx disabled])
fi

AH_TEMPLATE(C_X11_XKB,[define to 1 if you have XKBlib.h and X11 lib])
AC_CHECK_LIB(X11, main, have_x11_lib=yes, have_x11_lib=no, )
AC_CHECK_HEADER(X11/XKBlib.h, have_x11_h=yes, have_x11_h=no, )
AC_MSG_CHECKING(for XKBlib support)
if test x$have_x11_lib = xyes -a x$have_x11_h = xyes ; then
   LIBS="$LIBS -lX11"
   AC_DEFINE(C_X11_XKB,1)
   AC_MSG_RESULT(yes)
else
   AC_MSG_RESULT(no)
fi

AH_TEMPLATE(C_FLUIDSYNTH,[Define to 1 to enable fluidsynth MIDI synthesis])
AC_CHECK_HEADER(fluidsynth.h,have_fluidsynth_h=yes,)
AC_CHECK_LIB(fluidsynth, new_fluid_synth, have_fluidsynth_lib=yes,,)
if test x$have_fluidsynth_lib = xyes -a x$have_fluidsynth_h = xyes ; then
  LIBS="$LIBS -lfluidsynth"
  AC_DEFINE(C_FLUIDSYNTH,1)
else
  AC_MSG_WARN([fluidsynth MIDI synthesis not available])
fi

# FFMPEG output
AH_TEMPLATE(C_AVCODEC,[Define to 1 to use FFMPEG libavcodec for video capture])
AC_ARG_ENABLE(avcodec,AC_HELP_STRING([--disable-avcodec],[Disable FFMPEG avcodec support]),,enable_ffmpeg=yes)
if test x$enable_avcodec = xyes; then
	CFLAGS="$CFLAGS "`pkg-config libavcodec --cflags`
	CPPFLAGS="$CPPFLAGS "`pkg-config libavcodec --cflags`
	LIBS="$LIBS -lavcodec -lavformat -lavutil -lswscale "`pkg-config libavcodec --libs`
fi
AC_CHECK_LIB(avcodec,avcodec_open2,have_ffmpeg=yes,have_ffmpeg=no,)
AC_CHECK_HEADER(libavcodec/avcodec.h,have_avcodec_h=yes,have_avcodec_h=no,)
if test x$enable_avcodec = xyes; then
    if test x$have_ffmpeg = xyes; then
        if test x$have_avcodec_h = xyes; then
            AC_MSG_RESULT(yes)
            AC_DEFINE(C_AVCODEC,1)
        fi
    fi
fi

AH_TEMPLATE(C_OPENGL,[Define to 1 to use opengl display output support])
AC_CHECK_LIB(GL, main, have_gl_lib=yes, have_gl_lib=no , )
AC_CHECK_LIB(opengl32, main, have_opengl32_lib=yes,have_opengl32_lib=no , )
AC_CHECK_HEADER(GL/gl.h, have_gl_h=yes , have_gl_h=no , )
AC_ARG_ENABLE(opengl,AC_HELP_STRING([--disable-opengl],[Disable opengl support]),,enable_opengl=yes)
AC_MSG_CHECKING(whether opengl display output will be enabled)
if test x$enable_opengl = xyes; then
case "$host" in
    *-*-darwin*)
       AC_MSG_RESULT(yes)
       LIBS="$LIBS -framework OpenGL"
       AC_DEFINE(C_OPENGL,1)
       ;;
    *)
       if test x$have_gl_h = xyes -a x$have_gl_lib = xyes ; then
         AC_MSG_RESULT(yes)
         LIBS="$LIBS -lGL"
         AC_DEFINE(C_OPENGL,1)
       elif test x$have_gl_h = xyes -a x$have_opengl32_lib = xyes ; then
         AC_MSG_RESULT(yes)
         LIBS="$LIBS -lopengl32"
         AC_DEFINE(C_OPENGL,1)
       else
         AC_MSG_RESULT(no)
       fi
       ;;
esac
fi

dnl Check for mprotect. Needed for 64 bits linux 
AH_TEMPLATE(C_HAVE_MPROTECT,[Define to 1 if you have the mprotect function])
AC_CHECK_HEADER([sys/mman.h], [
AC_CHECK_FUNC([mprotect],[AC_DEFINE(C_HAVE_MPROTECT,1)])
])

dnl Setpriority
AH_TEMPLATE(C_SET_PRIORITY,[Define to 1 if you have setpriority support])
AC_MSG_CHECKING(for setpriority support)
AC_LINK_IFELSE([AC_LANG_SOURCE([
#include <sys/resource.h>
int main(int argc,char * argv[]) {
	return setpriority (PRIO_PROCESS, 0,PRIO_MIN+PRIO_MAX);
};
])],AC_MSG_RESULT(yes);AC_DEFINE(C_SET_PRIORITY,1),AC_MSG_RESULT(no))

dnl Some target detection and actions for them
case "$host" in
    *-*-cygwin* | *-*-mingw32*)
       LIBS="$LIBS -lwinmm -limm32"
       CXXFLAGS="$CXXFLAGS -mno-ms-bitfields"

dnl @FIXME:
dnl
dnl   GCC 5.4 is producing binaries that segfault at runtime during DirectX initialization.
dnl
dnl
dnl       AH_TEMPLATE(C_D3DSHADERS,[Define to 1 to use Direct3D shaders, requires d3d9.h and libd3dx9])
dnl       AC_ARG_WITH(dx-headers,[  --with-dx-headers=DIR   DirectX include files location],
dnl		   dx_headers="$withval", dx_headers="")
dnl       AC_ARG_WITH(dx-libs,[  --with-dx-libs=DIR      DirectX library location],
dnl		   dx_libs="$withval", dx_libs="")
dnl       ac_save_CPPFLAGS=$CPPFLAGS
dnl       if test x$dx_headers != x ; then
dnl        CPPFLAGS="$CPPFLAGS -I$dx_headers"
dnl       fi
dnl       AC_LANG(C++)
dnl       AC_CHECK_HEADERS(ddraw.h)
dnl       AC_CHECK_HEADERS(d3d9.h,have_d3d9_h=yes,)
dnl       AC_MSG_CHECKING(whether direct3d display output will be enabled)
dnl       if test x$have_d3d9_h = xyes ; then
dnl        AC_MSG_RESULT(yes)
dnl        AC_ARG_ENABLE(shaders,AC_HELP_STRING([--disable-shaders],[Disable d3d pixelshader support (which requires libd3dx9)]),,[
dnl		AC_MSG_CHECKING(whether direct3d pixelshaders will be enabled)
dnl		ac_save_LIBS=$LIBS
dnl		if test x$dx_libs != x ; then
dnl		 dx_libs=" -L$dx_libs"
dnl		fi
dnl		LIBS="$LIBS$dx_libs -ld3dx9"
dnl		AC_TRY_LINK([#include <d3d9.h>
dnl			     #include <d3dx9.h>
dnl			     extern "C" int main();],D3DXCreateEffect(0, 0, 0, 0, 0, 0, 0, 0, 0), have_d3dx9_lib=yes,)
dnl		if test x$have_d3dx9_lib = xyes ; then
dnl		 AC_MSG_RESULT(yes)
dnl		 AC_DEFINE(C_D3DSHADERS,1)
dnl		else
dnl		 LIBS=$ac_save_LIBS
dnl		 AC_MSG_RESULT(no)
dnl		 AC_MSG_WARN([Can't find libd3dx9, pixelshader support disabled])
dnl		fi
dnl	],)
dnl       else
dnl       CPPFLAGS=$ac_save_CPPFLAGS
dnl        AC_MSG_RESULT(no)
dnl       fi
       AC_DEFINE(C_DIRECTSERIAL, 1, [ Define to 1 if you want serial passthrough support (Win32, Posix and OS/2 only).])
       AC_DEFINE(C_DIRECTLPT, 1, [ Define to 1 if you want parallel passthrough support (Win32 only).])
       if test x$have_sdl_net_lib = xyes -a x$have_sdl_net_h = xyes ; then
         LIBS="$LIBS -lws2_32"
       fi
       ;;
    *-*-darwin*)
       dnl We have a problem here: both Mac OS X and Darwin report 
       dnl the same signature "powerpc-apple-darwin*" - so we have
       dnl to do more to distinguish them.
       dnl For now I am lazy and do not add proper detection code.
       AC_DEFINE(MACOSX, 1, [Compiling on Mac OS X])
       LIBS="$LIBS -framework CoreMIDI -framework AudioUnit -framework AudioToolbox"
       AC_DEFINE(C_DIRECTSERIAL, 1, [ Define to 1 if you want serial passthrough support (Win32, Posix and OS/2).])
       ;;
    *-*-linux*)
       AC_DEFINE(LINUX, 1, [Compiling on GNU/Linux])
       AC_DEFINE(C_DIRECTSERIAL, 1, [ Define to 1 if you want serial passthrough support (Win32, Posix and OS/2).])
       AC_DEFINE(C_DIRECTLPT, 1, [ Define to 1 if you want parallel passthrough support (Win32, Linux).])
       CXXFLAGS="$CXXFLAGS -D_XOPEN_SOURCE=700 -D_POSIX_C_SOURCE=200809L"
       ;;
    *-*-freebsd* | *-*-dragonfly* | *-*-netbsd* | *-*-openbsd*)
       dnl Disabled directserial for now. It doesn't do anything without
       dnl specifying an extra ifdef in directserial_posix.*
       dnl directserial detection should be rewritten to test for the needed
       dnl functions and headers. I currently do not know 
       dnl which ones are needed for BSD
       AC_DEFINE(BSD, 1, [Compiling on BSD])
       AC_DEFINE(C_DIRECTSERIAL, 1, [ Define to 1 if you want serial passthrough support (Win32, Posix and OS/2).])
       ;;
    *-*-os2-emx*)
       AC_DEFINE(OS2, 1, [Compiling on OS/2 EMX])
       AC_DEFINE(C_DIRECTSERIAL, 1, [ Define to 1 if you want serial passthrough support (Win32, Posix and OS/2).])
       ;;
esac

dnl Some stuff for the icon.
case "$host" in
    *-*-cygwin* | *-*-mingw32*)
       dnl Some stuff for the ico
       AC_CHECK_TOOL(WINDRES, windres, :)
    ;;
    *)
       WINDRES=":"
    ;;
esac
       AM_CONDITIONAL(HAVE_WINDRES, test "x$WINDRES" != "x:")
       AC_SUBST(WINDRES)


AC_CONFIG_FILES([ 
Makefile
src/Makefile
src/cpu/Makefile
src/cpu/core_full/Makefile
src/cpu/core_normal/Makefile
src/cpu/core_dyn_x86/Makefile
src/debug/Makefile
src/dos/Makefile
src/fpu/Makefile
src/gui/Makefile
src/hardware/Makefile
src/hardware/serialport/Makefile
src/hardware/reSID/Makefile
src/hardware/parport/Makefile
src/ints/Makefile
src/libs/Makefile
src/libs/zmbv/Makefile
src/libs/gui_tk/Makefile
src/libs/porttalk/Makefile
src/builtin/Makefile
src/misc/Makefile
src/shell/Makefile
src/platform/Makefile
include/Makefile
])
AC_OUTPUT

