AC_INIT([DOSBox-X-rewrite], 0.01)
AM_INIT_AUTOMAKE([subdir-objects])
AC_PROG_RANLIB
AC_PROG_CC
AC_PROG_CXX
AC_CONFIG_HEADER([config.h])
AC_PREFIX_DEFAULT(/usr/local)

if test $host != $build; then
    AC_CHECK_PROGS(BUILD_CXX, g++ c++)
    AC_CHECK_PROGS(BUILD_CC, gcc cc)
    AC_CHECK_PROGS(BUILD_CPP, cpp)
else
    BUILD_CXX=$CXX
    AC_SUBST(BUILD_CXX)

    BUILD_CPP=$CPP
    AC_SUBST(BUILD_CPP)

    BUILD_CC=$CC
    AC_SUBST(BUILD_CC)
fi

# largefile support.
# NTS: creation of config.h means you must include config.h to get _FILE_OFFSET_BITS=64 or whatever your platform needs.
#      Failure to do so will prevent that code from accessing files larger than 2GB on 32-bit systems (where off_t is 32-bit wide)
AC_SYS_LARGEFILE

# this code uses POSIX functions
POSIX_DEF=" -D_POSIX_C_SOURCE=200809L -D_BSD_SOURCE"
CFLAGS+="$POSIX_DEF"
CPPFLAGS+="$POSIX_DEF"
CXXFLAGS+="$POSIX_DEF"

# this code refers to itself
SELFINCLUDE_DEF=" -I$srcdir"
CFLAGS+="$SELFINCLUDE_DEF"
CPPFLAGS+="$SELFINCLUDE_DEF"
CXXFLAGS+="$SELFINCLUDE_DEF"

# this code uses C++11 features (TODO: gnu++11 if GCC, else c++11)
c11=0
AC_MSG_CHECKING([to make sure your C++ compiler supports C++11])
old_CXXFLAGS=$CXXFLAGS
old_CFLAGS=$CFLAGS
CXXFLAGS+=" -std=gnu++11"
CFLAGS+=" -std=gnu11"
AC_TRY_COMPILE([ ], [ ],
    [
        AC_MSG_RESULT([yes])
        c11=1
    ],
    [
        AC_MSG_RESULT([no])
        CXXFLAGS=$old_CXXFLAGS
        CFLAGS=$old_CFLAGS
    ])

# maybe gnu++0x?
if test $c11 = 0; then
    AC_MSG_CHECKING([to make sure your C++ compiler supports C++11 (gnu++0x)])
    CXXFLAGS+=" -std=gnu++0x"
    CFLAGS+=" -std=gnu1x"
    AC_TRY_COMPILE([ ], [ ],
        [
            AC_MSG_RESULT([yes])
            c11=1
        ],
        [
            AC_MSG_RESULT([no])
            CXXFLAGS=$old_CXXFLAGS
            CFLAGS=$old_CFLAGS
        ])
fi

# sorry
if test $c11 = 0; then
    AC_MSG_ERROR([This code base requires a C++11 capable compiler])
fi

# debug
AC_ARG_ENABLE(debug,AS_HELP_STRING([--enable-debug],[Enable debug information]),[ENABLE_DEBUG=$enableval],[ENABLE_DEBUG=no])
AC_SUBST(ENABLE_DEBUG)

# debug messages
AC_ARG_ENABLE(debug-msg,AS_HELP_STRING([--enable-debug-msg],[Enable debug code in the libraries]),[ENABLE_DEBUG_MSG=$enableval],[ENABLE_DEBUG_MSG=no])
AC_SUBST(ENABLE_DEBUG_MSG)

# stack checking
AC_ARG_ENABLE(stack-check,AS_HELP_STRING([--enable-stack-check],[Enable stack checking (default)]),[ENABLE_STACK_CHECK=$enableval],[ENABLE_STACK_CHECK=yes])
AC_SUBST(ENABLE_STACK_CHECK)

# work harder to optimize
AC_ARG_ENABLE(expensive-opts,AS_HELP_STRING([--enable-expensive-opts],[Enable expensive optimizations (default)]),[ENABLE_EXPENSIVE_OPTS=$enableval],[ENABLE_EXPENSIVE_OPTS=yes])
AC_SUBST(ENABLE_EXPENSIVE_OPTS)

# GCC also supports expensive optimizations
if test "$ENABLE_EXPENSIVE_OPTS" = yes; then
    CFLAGS+=" -fexpensive-optimizations";
    CXXFLAGS+=" -fexpensive-optimizations";
fi

# Open Watcom
HAVE_OPENWATCOM=0
WATCOM_ROOT=
AC_ARG_WITH(openwatcom,AS_HELP_STRING([--with-openwatcom],[Use Open Watcom C]),[with_openwatcom=$withval],[with_openwatcom=yes])
AC_SUBST(HAVE_OPENWATCOM)
AC_SUBST(WATCOM_ROOT)
AC_SUBST(WATCOM_EDPATH)
AC_SUBST(WATCOM_PATH)
AC_SUBST(WATCOM_INCLUDE)
AC_SUBST(DOSLIB3_INCLUDE)
AC_SUBST(WATCOM_ENV)
AC_SUBST(WATCOM_CC16)
AC_SUBST(WATCOM_CC32)
AC_SUBST(WATCOM_WLINK)
AC_SUBST(WATCOM_STRIP)
AC_SUBST(WATCOM_BIND)
AC_SUBST(WATCOM_WASM)
AC_SUBST(WATCOM_MAKE)
AC_SUBST(WATCOM_WLIB)
AC_SUBST(WATCOM_RC)
AC_SUBST(BUILDDIR)
AC_MSG_CHECKING([for openwatcom])
if test "$with_openwatcom" = "yes"; then
    dnl TODO: If Windows host, path needs to use ;
    dnl       Else (Linux and Mac OS X) use :
    for i in \
        /usr/src/open-watcom-v2/rel \
        /usr/watcom \
        /usr/watcom-1.9 \
        /usr/watcom-1.8; do
        if [[ -x "$i/binl/wcc" ]]; then
            HAVE_OPENWATCOM=1;
            WATCOM_ROOT=$i;
            break;
        fi;
    done
elif test -n "$with_openwatcom"; then
    if [[ -x "$with_openwatcom/binl/wcc" ]]; then
        HAVE_OPENWATCOM=1;
        WATCOM_ROOT=$with_openwatcom;
    fi
fi

if test "$HAVE_OPENWATCOM" = 1; then
    WATCOM_EDPATH="$i/eddat";
    WATCOM_PATH="$i/binl:$i/binw";
    WATCOM_INCLUDE="$i/h/nt;$i/h/nt/directx;$i/h/nt/ddk;$i/h";
    WATCOM_ENV="env \"EDPATH=\$(WATCOM_EDPATH)\" \"PATH=\$(WATCOM_PATH):\$(PATH)\" \"INCLUDE=\$(WATCOM_INCLUDE)\"";
    WATCOM_CC16="$WATCOM_ENV wcc -e=2 -zq -oilrtm -wx -q";
    WATCOM_CC32="$WATCOM_ENV wcc386 -e=2 -zq -oilrtm -wx -q";
    WATCOM_WLINK="$WATCOM_ENV wlink option quiet";
    WATCOM_MAKE="$WATCOM_ENV wmake -h -e";
    WATCOM_STRIP="$WATCOM_ENV wstrip -q";
    WATCOM_BIND="$WATCOM_ENV wbind -q";
    WATCOM_WASM="$WATCOM_ENV wasm -zq";
    WATCOM_WLIB="$WATCOM_ENV wlib -q";
    WATCOM_RC="$WATCOM_ENV wrc -q";
    DOSLIB3_INCLUDE="-i\$(abs_top_builddir)";

    if test "$ENABLE_DEBUG_MSG" = yes; then
        WATCOM_WASM+=" -dDOSLIB_DEBUG_MSG=1";
        WATCOM_CC16+=" -dDOSLIB_DEBUG_MSG=1";
        WATCOM_CC32+=" -dDOSLIB_DEBUG_MSG=1";
        WATCOM_RC+=" -DDOSLIB_DEBUG_MSG=1";
    fi

    if test "$ENABLE_DEBUG" = yes; then
        # full debug, browse
        WATCOM_CC16+=" -d9 -db";
        WATCOM_CC32+=" -d9 -db";
    else
        # no debug info
        WATCOM_CC16+=" -d0";
        WATCOM_CC32+=" -d0";
    fi

    if test "$ENABLE_STACK_CHECK" = yes; then
        # stack frames
        WATCOM_CC16+=" -of";
        WATCOM_CC32+=" -of";
    else
        # disable stack checking
        WATCOM_CC16+=" -s";
        WATCOM_CC32+=" -s";
    fi

    if test "$ENABLE_EXPENSIVE_OPTS" = yes; then
        WATCOM_CC16+=" -oh";
        WATCOM_CC32+=" -oh";
    fi

    AC_MSG_RESULT([yes: $WATCOM_ROOT])
else
    AC_MSG_RESULT([no])
	AC_MSG_ERROR([Open Watcom C is required to build this source code])
fi

# note builddir
BUILDDIR="\$(abs_top_builddir)/build"

# we also want -Wall
AC_MSG_CHECKING([support for -Wall])
CXXFLAGS+=" -Wall"
CFLAGS+=" -Wall"
AC_TRY_COMPILE([ ], [ ],
    [
        AC_MSG_RESULT([yes])
        c11=1
    ],
    [
        AC_MSG_RESULT([no])
        CXXFLAGS=$old_CXXFLAGS
        CFLAGS=$old_CFLAGS
    ])

# we also want -Wextra
AC_MSG_CHECKING([support for -Wextra])
CXXFLAGS+=" -Wextra"
CFLAGS+=" -Wextra"
AC_TRY_COMPILE([ ], [ ],
    [
        AC_MSG_RESULT([yes])
        c11=1
    ],
    [
        AC_MSG_RESULT([no])
        CXXFLAGS=$old_CXXFLAGS
        CFLAGS=$old_CFLAGS
    ])

# we also want -pedantic
AC_MSG_CHECKING([support for -pedantic])
CXXFLAGS+=" -pedantic"
CFLAGS+=" -pedantic"
AC_TRY_COMPILE([ ], [ ],
    [
        AC_MSG_RESULT([yes])
        c11=1
    ],
    [
        AC_MSG_RESULT([no])
        CXXFLAGS=$old_CXXFLAGS
        CFLAGS=$old_CFLAGS
    ])

# we also want -Wno-unused-label
# our opcode compiler output triggers this warning too much.
AC_MSG_CHECKING([support for -Wno-unused-label])
CXXFLAGS+=" -Wno-unused-label"
CFLAGS+=" -Wno-unused-label"
AC_TRY_COMPILE([ ], [ ],
    [
        AC_MSG_RESULT([yes])
        c11=1
    ],
    [
        AC_MSG_RESULT([no])
        CXXFLAGS=$old_CXXFLAGS
        CFLAGS=$old_CFLAGS
    ])

# use MMX, SSE, SSE2, AVX if available (TODO: Only if target is i686 or x86_64)
# not for ALL code, just the optimized rendering code.
old_CXXFLAGS=$CXXFLAGS
old_CFLAGS=$CFLAGS
for ext in mmx sse sse2 sse3 avx avx2; do
    AC_MSG_CHECKING([to make sure your C++ compiler supports the $ext instruction set])
    EXT=`echo $ext | tr [[:lower:]] [[:upper:]]`
    CXXFLAGS+=" -m$ext"
    CFLAGS+=" -m$ext"
    AC_TRY_COMPILE([ ],[ ],
        [
            AC_MSG_RESULT([yes])
            _nm=HAVE_CPU_$EXT
            eval $_nm=1
        ],
        [
            AC_MSG_RESULT([no])
            _nm=HAVE_CPU_$EXT
            eval $_nm=0
        ])
    CXXFLAGS=$old_CXXFLAGS
    CFLAGS=$old_CFLAGS
done
AC_SUBST(HAVE_CPU_MMX)
AC_SUBST(CPU_MMX_CFLAGS)
AM_CONDITIONAL([HAVE_CPU_MMX],[test $HAVE_CPU_MMX = 1])
AC_DEFINE_UNQUOTED([HAVE_CPU_MMX],[$HAVE_CPU_MMX],[Has MMX CPU extension])
if test $HAVE_CPU_MMX = 1; then CPU_MMX_CFLAGS+=-mmmx; fi

AC_SUBST(HAVE_CPU_SSE)
AC_SUBST(CPU_SSE_CFLAGS)
AM_CONDITIONAL([HAVE_CPU_SSE],[test $HAVE_CPU_SSE = 1])
AC_DEFINE_UNQUOTED([HAVE_CPU_SSE],[$HAVE_CPU_SSE],[Has SSE CPU extension])
if test $HAVE_CPU_SSE = 1; then CPU_SSE_CFLAGS+=-msse; fi

AC_SUBST(HAVE_CPU_SSE2)
AC_SUBST(CPU_SSE2_CFLAGS)
AM_CONDITIONAL([HAVE_CPU_SSE2],[test $HAVE_CPU_SSE2 = 1])
AC_DEFINE_UNQUOTED([HAVE_CPU_SSE2],[$HAVE_CPU_SSE2],[Has SSE2 CPU extension])
if test $HAVE_CPU_SSE2 = 1; then CPU_SSE2_CFLAGS+=-msse2; fi

AC_SUBST(HAVE_CPU_SSE3)
AC_SUBST(CPU_SSE3_CFLAGS)
AM_CONDITIONAL([HAVE_CPU_SSE3],[test $HAVE_CPU_SSE3 = 1])
AC_DEFINE_UNQUOTED([HAVE_CPU_SSE3],[$HAVE_CPU_SSE3],[Has SSE3 CPU extension])
if test $HAVE_CPU_SSE3 = 1; then CPU_SSE3_CFLAGS+=-msse3; fi

AC_SUBST(HAVE_CPU_AVX)
AC_SUBST(CPU_AVX_CFLAGS)
AM_CONDITIONAL([HAVE_CPU_AVX],[test $HAVE_CPU_AVX = 1])
AC_DEFINE_UNQUOTED([HAVE_CPU_AVX],[$HAVE_CPU_AVX],[Has AVX CPU extension])
if test $HAVE_CPU_AVX = 1; then CPU_AVX_CFLAGS+=-mavx; fi

AC_SUBST(HAVE_CPU_AVX2)
AC_SUBST(CPU_AVX2_CFLAGS)
AM_CONDITIONAL([HAVE_CPU_AVX2],[test $HAVE_CPU_AVX2 = 1])
AC_DEFINE_UNQUOTED([HAVE_CPU_AVX2],[$HAVE_CPU_AVX2],[Has AVX2 CPU extension])
if test $HAVE_CPU_AVX2 = 1; then CPU_AVX2_CFLAGS+=-mavx2; fi

AC_SUBST(CFLAGS)
AC_SUBST(CXXFLAGS)
AC_SUBST(LDFLAGS)

# important functions
AC_CHECK_FUNCS([nanosleep usleep lseek64 lseeki64 truncate ftruncate openat renameat symlink link stat fstat fstatat statfs fstatfs realpath sysconf mmap munmap getpagesize mincore mlock munlock mprotect mremap msync getrlimit setrlimit shmat shm_open remap_file_pages fallocate readlink readlinkat signal sigaction alarm pause sigprocmask sigaddset sigemptyset sigismember posix_memalign])
AC_CHECK_DECLS([isinf, isnan],[],[],[[#include <math.h>]])
AC_CHECK_DECLS([le16toh, le32toh],[],[],[[#include <endian.h>]])

# we use yasm
AC_CHECK_PROGS(YASM,$YASM yasm,error)
if test x"$YASM" = x"error"; then
	AC_MSG_ERROR([Yasm is required to build this source code])
fi

# we need Perl
AC_CHECK_PROGS(PERL,$PERL perl,error)
if test x"$PERL" = x"error"; then
	AC_MSG_ERROR([Perl is required to build this source code])
fi

# we need GNU iconv
AC_CHECK_PROGS(ICONV,$ICONV iconv,error)
if test x"$ICONV" = x"error"; then
	AC_MSG_ERROR([iconv is required to build this source code])
fi

# we use pkg-config
AC_CHECK_PROGS(PKGCONFIG,$PKGCONFIG pkg-config,error)
if test x"$PKGCONFIG" = x"error"; then
	AC_MSG_ERROR([pkg-config is required to build this source code])
fi

# autoconf script provides "target_alias"
# what OS are we targeting?
AC_MSG_CHECKING([host os])
build_os=$target_alias
if test "$build_os" = ""; then build_os=$build_alias; fi
if test "$build_os" = ""; then build_os=$host_alias; fi
if test "$build_os" = ""; then build_os=`uname -o 2>/dev/null`; fi
if test "$build_os" = ""; then build_os=`uname -s 2>/dev/null`; fi
AC_MSG_RESULT([$build_os])

IS_MAC_OSX=0
IS_MINGW=0
IS_LINUX=0
IS_WINDOWS=0
NEED_FPIC=0
NEED_M64=0
case "$build_os" in
    *linux*)
        IS_LINUX=1
        ;;
    *GNU\/Linux*)
        IS_LINUX=1
        ;;
    *Darwin*)
        IS_MAC_OSX=1
        ;;
    *Msys*)
        IS_MINGW=1
        IS_WINDOWS=1
        ;;
    *x86_64-pc-msys*)
        IS_MINGW=1
        IS_WINDOWS=1
        NEED_M64=1
        ;;
    *)
        ;;
esac

# the target may need -fPIC (such as x86_64)
if test "$NEED_FPIC" = 1; then
    FPIC_DEF=" -fPIC"
    CFLAGS+="$FPIC_DEF"
    CPPFLAGS+="$FPIC_DEF"
    CXXFLAGS+="$FPIC_DEF"
fi

# the target may need -m64 (such as x86_64)
if test "$NEED_M64" = 1; then
    M64_DEF=" -m64"
    CFLAGS+="$M64_DEF"
    CPPFLAGS+="$M64_DEF"
    CXXFLAGS+="$M64_DEF"
fi

# Apple in their wisdom use their own typedefs that don't show up by default
# if you define POSIX_C_SOURCE, but that many important syscalls use anyway.
# we need these syscalls, so we have to define DARWIN_C_SOURCE to get them.
if test "$IS_MAC_OSX" = 1; then
    DARWIN_DEF=" -D_DARWIN_C_SOURCE"
    CFLAGS+="$DARWIN_DEF"
    CPPFLAGS+="$DARWIN_DEF"
    CXXFLAGS+="$DARWIN_DEF"
fi

# Mac OS X may have XQuartz, with X11 headers, in a nonstandard location
if test "$IS_MAC_OSX" = 1; then
    if [[ -d "/opt/X11" ]]; then
        DARWIN_DEF=" -I/opt/X11/include"
        CFLAGS+="$DARWIN_DEF"
        CPPFLAGS+="$DARWIN_DEF"
        CXXFLAGS+="$DARWIN_DEF"
        LDFLAGS+=" -L/opt/X11/lib"
    fi
fi

# MinGW please statically link libgcc and libstdc++
if test "$IS_MINGW" = 1; then
    STATIC_DEF=" -Wl,-Bstatic -static-libgcc"
    CFLAGS+="$STATIC_DEF"
    CPPFLAGS+="$STATIC_DEF"
    CXXFLAGS+="$STATIC_DEF"
fi

# Mac OS X test programs require the use of Objective C++
AC_SUBST(OBJCXXFLAGS)
OBJCXXFLAGS=0
AC_SUBST(HAS_OBJCXX)
HAS_OBJCXX=0
if test "$IS_MAC_OSX" = 1; then
    old_CXXFLAGS=$CXXFLAGS
    old_CFLAGS=$CFLAGS
    AC_MSG_CHECKING([that your C++ compiler can handle Objective C++])
    EXT=" -ObjC++"
    CXXFLAGS+="$ext"
    CFLAGS+="$ext"
    AC_TRY_COMPILE([ ],[ ],
        [
            AC_MSG_RESULT([yes])
            OBJCXXFLAGS=" -ObjC++"
            HAS_OBJCXX=1
        ],
        [
            AC_MSG_RESULT([no])
        ])
    CXXFLAGS=$old_CXXFLAGS
    CFLAGS=$old_CFLAGS
fi
AM_CONDITIONAL([HAS_OBJCXX],[test $HAS_OBJCXX = 1])
AC_DEFINE_UNQUOTED([HAS_OBJCXX],[$HAS_OBJCXX],[Has Objective C++])

AC_SUBST(IS_LINUX)
AM_CONDITIONAL([IS_LINUX],[test $IS_LINUX = 1])
AC_DEFINE_UNQUOTED([IS_LINUX],[$IS_LINUX],[OS is Linux])

AC_SUBST(IS_MAC_OSX)
AM_CONDITIONAL([IS_MAC_OSX],[test $IS_MAC_OSX = 1])
AC_DEFINE_UNQUOTED([IS_MAC_OSX],[$IS_MAC_OSX],[OS is Mac OS X (Darwin)])

AC_SUBST(IS_WINDOWS)
AM_CONDITIONAL([IS_WINDOWS],[test $IS_WINDOWS = 1])
AC_DEFINE_UNQUOTED([IS_WINDOWS],[$IS_WINDOWS],[OS is Windows])

AC_SUBST(IS_MINGW)
AM_CONDITIONAL([IS_MINGW],[test $IS_MINGW = 1])
AC_DEFINE_UNQUOTED([IS_MINGW],[$IS_MINGW],[OS is Windows (MinGW/Msys)])

# GCC __builtin_cpu_init and other functions.
# Note that Clang doesn't provide this, yet.
HAVE_GCC_BUILTIN_CPU=0
AC_SUBST(HAVE_GCC_BUILTIN_CPU)
AC_MSG_CHECKING([for GCC builtin_cpu functions])
AC_TRY_LINK([#include <cpuid.h>],
    [
        __builtin_cpu_init();
        __builtin_cpu_supports("sse2");
    ],
    [
        AC_MSG_RESULT([yes])
        HAVE_GCC_BUILTIN_CPU=1
    ],
    [
        AC_MSG_RESULT([no])
    ])
AM_CONDITIONAL([HAVE_GCC_BUILTIN_CPU],[test $HAVE_GCC_BUILTIN_CPU = 1])
AC_DEFINE_UNQUOTED([HAVE_GCC_BUILTIN_CPU],[$HAVE_GCC_BUILTIN_CPU],[Has GCC builtin_cpu functions])

# we have to work at checking for clock_gettime
# note that on Linux this requires librt.
old_CXXFLAGS=$CXXFLAGS
old_CFLAGS=$CFLAGS
CFLAGS+=" -lrt"
CXXFLAGS+=" -lrt"
HAVE_CLOCK_GETTIME=0
AC_SUBST(HAVE_CLOCK_GETTIME)
AC_MSG_CHECKING([for clock_gettime])
AC_TRY_LINK([#include <time.h>],
    [
            struct timespec tv;
            clock_gettime(CLOCK_MONOTONIC,&tv);
    ],
    [
        AC_MSG_RESULT([yes])
        HAVE_CLOCK_GETTIME=1
    ],
    [
        AC_MSG_RESULT([no])
        CXXFLAGS=$old_CXXFLAGS
        CFLAGS=$old_CFLAGS
    ])
AM_CONDITIONAL([HAVE_CLOCK_GETTIME],[test $HAVE_CLOCK_GETTIME = 1])
AC_DEFINE_UNQUOTED([HAVE_CLOCK_GETTIME],[$HAVE_CLOCK_GETTIME],[Has clock_gettime])

# O_BINARY?
HAVE_O_BINARY=0
AC_SUBST(HAVE_O_BINARY)
AC_MSG_CHECKING([for O_BINARY])
AC_TRY_LINK(
    [
        #include <unistd.h>
    #include <fcntl.h>
    ],
    [
        int fd = open("test",O_BINARY|O_RDONLY);
    ],
    [
        AC_MSG_RESULT([yes])
        HAVE_O_BINARY=1
    ],
    [
        AC_MSG_RESULT([no])
    ])
AM_CONDITIONAL([HAVE_O_BINARY],[test $HAVE_O_BINARY = 1])
AC_DEFINE_UNQUOTED([HAVE_O_BINARY],[$HAVE_O_BINARY],[Has O_BINARY])

# __asm__?
HAVE_GCC_ASM=0
AC_SUBST(HAVE_GCC_ASM)
AC_MSG_CHECKING([for GCC_ASM])
AC_TRY_LINK(
    [
    ],
    [
        __asm__("");
    ],
    [
        AC_MSG_RESULT([yes])
        HAVE_GCC_ASM=1
    ],
    [
        AC_MSG_RESULT([no])
    ])
AM_CONDITIONAL([HAVE_GCC_ASM],[test $HAVE_GCC_ASM = 1])
AC_DEFINE_UNQUOTED([HAVE_GCC_ASM],[$HAVE_GCC_ASM],[Has GCC __asm__])

# should we use __asm__?
USE_ASM=$HAVE_GCC_ASM # by default, yes, if the compiler provides it
AC_ARG_WITH(gcc-asm,AS_HELP_STRING([--with-asm],[Use inline assembly]),[],[with_asm=yes])
AC_SUBST(USE_ASM)
if test "$with_asm" = "no"; then USE_ASM=0; fi
AC_DEFINE_UNQUOTED([USE_ASM],[$USE_ASM],[Use inline assembly])
AM_CONDITIONAL([USE_ASM],[test $USE_ASM = 1])

# Does the system have windows.h?
HAVE_WINDOWS_H=0
AC_SUBST(HAVE_WINDOWS_H)
AC_CHECK_HEADER([windows.h],[HAVE_WINDOWS_H=1],[HAVE_WINDOWS_H=0])
AM_CONDITIONAL([HAVE_WINDOWS_H],[test $HAVE_WINDOWS_H = 1])
AC_DEFINE_UNQUOTED([HAVE_WINDOWS_H],[$HAVE_WINDOWS_H],[Has windows.h])

# Does Linux have sys/vm86.h for virtual 8086 mode?
HAVE_SYS_VM86_H=0
AC_SUBST(HAVE_SYS_VM86_H)
AC_CHECK_HEADER([sys/vm86.h],[HAVE_SYS_VM86_H=1],[HAVE_SYS_VM86_H=0])
AM_CONDITIONAL([HAVE_SYS_VM86_H],[test $HAVE_SYS_VM86_H = 1])
AC_DEFINE_UNQUOTED([HAVE_SYS_VM86_H],[$HAVE_SYS_VM86_H],[Has sys/vm86.h])

# Does Linux have sys/ptrace.h?
HAVE_SYS_PTRACE_H=0
AC_SUBST(HAVE_SYS_PTRACE_H)
AC_CHECK_HEADER([sys/ptrace.h],[HAVE_SYS_PTRACE_H=1],[HAVE_SYS_PTRACE_H=0])
AM_CONDITIONAL([HAVE_SYS_PTRACE_H],[test $HAVE_SYS_PTRACE_H = 1])
AC_DEFINE_UNQUOTED([HAVE_SYS_PTRACE_H],[$HAVE_SYS_PTRACE_H],[Has sys/ptrace.h])

# Does Linux have asm/ldt.h for the modify_ldt syscall?
HAVE_ASM_LDT_H=0
AC_SUBST(HAVE_ASM_LDT_H)
AC_CHECK_HEADER([asm/ldt.h],[HAVE_ASM_LDT_H=1],[HAVE_ASM_LDT_H=0])
AM_CONDITIONAL([HAVE_ASM_LDT_H],[test $HAVE_ASM_LDT_H = 1])
AC_DEFINE_UNQUOTED([HAVE_ASM_LDT_H],[$HAVE_ASM_LDT_H],[Has asm/ldt.h])

# Does Linux have linux/kvm.h for KVM support?
HAVE_LINUX_KVM_H=0
AC_SUBST(HAVE_LINUX_KVM_H)
AC_CHECK_HEADER([linux/kvm.h],[HAVE_LINUX_KVM_H=1],[HAVE_LINUX_KVM_H=0])
AM_CONDITIONAL([HAVE_LINUX_KVM_H],[test $HAVE_LINUX_KVM_H = 1])
AC_DEFINE_UNQUOTED([HAVE_LINUX_KVM_H],[$HAVE_LINUX_KVM_H],[Has linux/kvm.h])

# Does Linux have linux/fb.h for the framebuffer?
HAVE_LINUX_FB_H=0
AC_SUBST(HAVE_LINUX_FB_H)
AC_CHECK_HEADER([linux/fb.h],[HAVE_LINUX_FB_H=1],[HAVE_LINUX_FB_H=0])
AM_CONDITIONAL([HAVE_LINUX_FB_H],[test $HAVE_LINUX_FB_H = 1])
AC_DEFINE_UNQUOTED([HAVE_LINUX_FB_H],[$HAVE_LINUX_FB_H],[Has linux/fb.h])

# Does Linux have termios.h?
HAVE_TERMIOS_H=0
AC_SUBST(HAVE_TERMIOS_H)
AC_CHECK_HEADER([termios.h],[HAVE_TERMIOS_H=1],[HAVE_TERMIOS_H=0])
AM_CONDITIONAL([HAVE_TERMIOS_H],[test $HAVE_TERMIOS_H = 1])
AC_DEFINE_UNQUOTED([HAVE_TERMIOS_H],[$HAVE_TERMIOS_H],[Has termios.h])

# Does Linux have sys/prctl.h?
HAVE_SYS_PRCTL_H=0
AC_SUBST(HAVE_SYS_PRCTL_H)
AC_CHECK_HEADER([sys/prctl.h],[HAVE_SYS_PRCTL_H=1],[HAVE_SYS_PRCTL_H=0])
AM_CONDITIONAL([HAVE_SYS_PRCTL_H],[test $HAVE_SYS_PRCTL_H = 1])
AC_DEFINE_UNQUOTED([HAVE_SYS_PRCTL_H],[$HAVE_SYS_PRCTL_H],[Has sys/prctl.h])

# Does your compiler have mmintrin.h (MMX intrinsics)?
HAVE_MMINTRIN_H=0
AC_SUBST(HAVE_MMINTRIN_H)
AC_CHECK_HEADER([mmintrin.h],[HAVE_MMINTRIN_H=1],[HAVE_MMINTRIN_H=0])
AM_CONDITIONAL([HAVE_MMINTRIN_H],[test $HAVE_MMINTRIN_H = 1])
AC_DEFINE_UNQUOTED([HAVE_MMINTRIN_H],[$HAVE_MMINTRIN_H],[Has mmintrin.h])

# Does your compiler have xmmintrin.h (SSE intrinsics)?
HAVE_XMMINTRIN_H=0
AC_SUBST(HAVE_XMMINTRIN_H)
AC_CHECK_HEADER([xmmintrin.h],[HAVE_XMMINTRIN_H=1],[HAVE_XMMINTRIN_H=0])
AM_CONDITIONAL([HAVE_XMMINTRIN_H],[test $HAVE_XMMINTRIN_H = 1])
AC_DEFINE_UNQUOTED([HAVE_XMMINTRIN_H],[$HAVE_XMMINTRIN_H],[Has xmmintrin.h])

# Does your compiler have immintrin.h (AVX intrinsics)?
HAVE_IMMINTRIN_H=0
AC_SUBST(HAVE_IMMINTRIN_H)
AC_CHECK_HEADER([immintrin.h],[HAVE_IMMINTRIN_H=1],[HAVE_IMMINTRIN_H=0])
AM_CONDITIONAL([HAVE_IMMINTRIN_H],[test $HAVE_IMMINTRIN_H = 1])
AC_DEFINE_UNQUOTED([HAVE_IMMINTRIN_H],[$HAVE_IMMINTRIN_H],[Has immintrin.h])

# Does your compiler have avxintrin.h (AVX intrinsics)?
HAVE_AVXINTRIN_H=0
AC_SUBST(HAVE_AVXINTRIN_H)
AC_CHECK_HEADER([avxintrin.h],[HAVE_AVXINTRIN_H=1],[HAVE_AVXINTRIN_H=0])
AM_CONDITIONAL([HAVE_AVXINTRIN_H],[test $HAVE_AVXINTRIN_H = 1])
AC_DEFINE_UNQUOTED([HAVE_AVXINTRIN_H],[$HAVE_AVXINTRIN_H],[Has avxintrin.h])

# Does your compiler have limits.h
HAVE_LIMITS_H=0
AC_SUBST(HAVE_LIMITS_H)
AC_CHECK_HEADER([limits.h],[HAVE_LIMITS_H=1],[HAVE_LIMITS_H=0])
AM_CONDITIONAL([HAVE_LIMITS_H],[test $HAVE_LIMITS_H = 1])
AC_DEFINE_UNQUOTED([HAVE_LIMITS_H],[$HAVE_LIMITS_H],[Has limits.h])

# Does your compiler have endian.h?
HAVE_ENDIAN_H=0
AC_SUBST(HAVE_ENDIAN_H)
AC_CHECK_HEADER([endian.h],[HAVE_ENDIAN_H=1],[HAVE_ENDIAN_H=0])
AM_CONDITIONAL([HAVE_ENDIAN_H],[test $HAVE_ENDIAN_H = 1])
AC_DEFINE_UNQUOTED([HAVE_ENDIAN_H],[$HAVE_ENDIAN_H],[Has endian.h])

# ZLIB
HAVE_ZLIB=0
AC_ARG_WITH(zlib,AS_HELP_STRING([--with-zlib],[Enable ZLIB support]),[],[with_zlib=yes])
AC_SUBST(HAVE_ZLIB)
AC_SUBST(ZLIB_CFLAGS)
AC_SUBST(ZLIB_LDFLAGS)
if test "$with_zlib" = "yes"; then
    AC_MSG_CHECKING([for ZLIB])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=1.2.6 zlib; then
            HAVE_ZLIB=1
            ZLIB_CFLAGS=`pkg-config --cflags zlib`
            ZLIB_LDFLAGS=`pkg-config --libs zlib`
        fi
    fi

    if test "$HAVE_ZLIB" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_ZLIB],[$HAVE_ZLIB],[Has ZLIB])
AM_CONDITIONAL([HAVE_ZLIB],[test $HAVE_ZLIB = 1])

# LZMA
HAVE_LZMA=0
AC_ARG_WITH(lzma,AS_HELP_STRING([--with-lzma],[Enable LZMA support]),[],[with_lzma=yes])
AC_SUBST(HAVE_LZMA)
AC_SUBST(LZMA_CFLAGS)
AC_SUBST(LZMA_LDFLAGS)
if test "$with_lzma" = "yes"; then
    AC_MSG_CHECKING([for LZMA])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=5.0.3 liblzma; then
            HAVE_LZMA=1
            LZMA_CFLAGS=`pkg-config --cflags liblzma`
            LZMA_LDFLAGS=`pkg-config --libs liblzma`
        fi
    fi

    if test "$HAVE_LZMA" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_LZMA],[$HAVE_LZMA],[Has LZMA])
AM_CONDITIONAL([HAVE_LZMA],[test $HAVE_LZMA = 1])

# FFMPEG avcodec
HAVE_FFMPEG_AVCODEC=0
AC_ARG_WITH(ffmpeg,AS_HELP_STRING([--with-ffmpeg],[Enable FFMPEG support]),[],[with_ffmpeg=yes])
AC_SUBST(HAVE_FFMPEG_AVCODEC)
AC_SUBST(FFMPEG_AVCODEC_CFLAGS)
AC_SUBST(FFMPEG_AVCODEC_LDFLAGS)
if test "$with_ffmpeg" = "yes"; then
    AC_MSG_CHECKING([for FFMPEG avcodec])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=57.24.102 libavcodec; then
            HAVE_FFMPEG_AVCODEC=1
            FFMPEG_AVCODEC_CFLAGS=`pkg-config --cflags libavcodec`
            FFMPEG_AVCODEC_LDFLAGS=`pkg-config --libs libavcodec`
        fi
    fi

    if test "$HAVE_FFMPEG_AVCODEC" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_FFMPEG_AVCODEC],[$HAVE_FFMPEG_AVCODEC],[Has FFMPEG_AVCODEC])
AM_CONDITIONAL([HAVE_FFMPEG_AVCODEC],[test $HAVE_FFMPEG_AVCODEC = 1])

# FFMPEG avformat
HAVE_FFMPEG_AVFORMAT=0
AC_SUBST(HAVE_FFMPEG_AVFORMAT)
AC_SUBST(FFMPEG_AVFORMAT_CFLAGS)
AC_SUBST(FFMPEG_AVFORMAT_LDFLAGS)
if test "$with_ffmpeg" = "yes"; then
    AC_MSG_CHECKING([for FFMPEG avformat])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=57.25.100 libavformat; then
            HAVE_FFMPEG_AVFORMAT=1
            FFMPEG_AVFORMAT_CFLAGS=`pkg-config --cflags libavformat`
            FFMPEG_AVFORMAT_LDFLAGS=`pkg-config --libs libavformat`
        fi
    fi

    if test "$HAVE_FFMPEG_AVFORMAT" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_FFMPEG_AVFORMAT],[$HAVE_FFMPEG_AVFORMAT],[Has FFMPEG_AVFORMAT])
AM_CONDITIONAL([HAVE_FFMPEG_AVFORMAT],[test $HAVE_FFMPEG_AVFORMAT = 1])

# FFMPEG avutil
HAVE_FFMPEG_AVUTIL=0
AC_SUBST(HAVE_FFMPEG_AVUTIL)
AC_SUBST(FFMPEG_AVUTIL_CFLAGS)
AC_SUBST(FFMPEG_AVUTIL_LDFLAGS)
if test "$with_ffmpeg" = "yes"; then
    AC_MSG_CHECKING([for FFMPEG avutil])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=55.17.103 libavutil; then
            HAVE_FFMPEG_AVUTIL=1
            FFMPEG_AVUTIL_CFLAGS=`pkg-config --cflags libavutil`
            FFMPEG_AVUTIL_LDFLAGS=`pkg-config --libs libavutil`
        fi
    fi

    if test "$HAVE_FFMPEG_AVUTIL" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_FFMPEG_AVUTIL],[$HAVE_FFMPEG_AVUTIL],[Has FFMPEG_AVUTIL])
AM_CONDITIONAL([HAVE_FFMPEG_AVUTIL],[test $HAVE_FFMPEG_AVUTIL = 1])

# FFMPEG swscale
HAVE_FFMPEG_SWSCALE=0
AC_ARG_WITH(ffmpeg-swscale,AS_HELP_STRING([--with-ffmpeg-swscale],[Enable FFMPEG swscale support]),[],[with_ffmpeg_swscale=yes])
AC_SUBST(HAVE_FFMPEG_SWSCALE)
AC_SUBST(FFMPEG_SWSCALE_CFLAGS)
AC_SUBST(FFMPEG_SWSCALE_LDFLAGS)
if test "$with_ffmpeg_swscale" = "yes"; then
    AC_MSG_CHECKING([for FFMPEG swscale])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=4.0.100 libswscale; then
            HAVE_FFMPEG_SWSCALE=1
            FFMPEG_SWSCALE_CFLAGS=`pkg-config --cflags libswscale`
            FFMPEG_SWSCALE_LDFLAGS=`pkg-config --libs libswscale`
        fi
    fi

    if test "$HAVE_FFMPEG_SWSCALE" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_FFMPEG_SWSCALE],[$HAVE_FFMPEG_SWSCALE],[Has FFMPEG_SWSCALE])
AM_CONDITIONAL([HAVE_FFMPEG_SWSCALE],[test $HAVE_FFMPEG_SWSCALE = 1])

# Simple Directmedia Layer 1.x
HAVE_SDL1=0
AC_ARG_WITH(sdl1,AS_HELP_STRING([--with-sdl1],[Enable SDL 1.x]),[],[with_sdl1=yes])
AC_SUBST(HAVE_SDL1)
AC_SUBST(SDL1_CFLAGS)
AC_SUBST(SDL1_LDFLAGS)
if test "$with_sdl1" = "yes"; then
    AC_MSG_CHECKING([for SDL 1.x])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=1.2.15 sdl; then
            HAVE_SDL1=1
            SDL1_CFLAGS=`pkg-config --cflags sdl`
            SDL1_LDFLAGS=`pkg-config --libs sdl`
        fi
    fi

    if test "$HAVE_SDL1" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_SDL1],[$HAVE_SDL1],[Has SDL1])
AM_CONDITIONAL([HAVE_SDL1],[test $HAVE_SDL1 = 1])

# Simple Directmedia Layer 2.x
HAVE_SDL2=0
AC_ARG_WITH(sdl2,AS_HELP_STRING([--with-sdl2],[Enable SDL 2.x]),[],[with_sdl2=yes])
AC_SUBST(HAVE_SDL2)
AC_SUBST(SDL2_CFLAGS)
AC_SUBST(SDL2_LDFLAGS)
if test "$with_sdl2" = "yes"; then
    AC_MSG_CHECKING([for SDL 2.x])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=2.0.3 sdl2; then
            HAVE_SDL2=1
            SDL2_CFLAGS=`pkg-config --cflags sdl2`
            SDL2_LDFLAGS=`pkg-config --libs sdl2`
        fi
    fi

    if test "$HAVE_SDL2" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_SDL2],[$HAVE_SDL2],[Has SDL2])
AM_CONDITIONAL([HAVE_SDL2],[test $HAVE_SDL2 = 1])

# libpng
HAVE_LIBPNG=0
AC_ARG_WITH(libpng,AS_HELP_STRING([--with-libpng],[Enable libpng]),[],[with_libpng=yes])
AC_SUBST(HAVE_LIBPNG)
AC_SUBST(LIBPNG_CFLAGS)
AC_SUBST(LIBPNG_LDFLAGS)
if test "$with_libpng" = "yes"; then
    AC_MSG_CHECKING([for libpng])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=1.2.46 libpng; then
            HAVE_LIBPNG=1
            LIBPNG_CFLAGS=`pkg-config --cflags libpng`
            LIBPNG_LDFLAGS=`pkg-config --libs libpng`
        fi
    fi

    if test "$HAVE_LIBPNG" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_LIBPNG],[$HAVE_LIBPNG],[Has LIBPNG])
AM_CONDITIONAL([HAVE_LIBPNG],[test $HAVE_LIBPNG = 1])

# libjpeg
HAVE_LIBJPEG=0
AC_ARG_WITH(libjpeg,AS_HELP_STRING([--with-libjpeg],[Enable libjpeg]),[],[with_libjpeg=yes])
AC_SUBST(HAVE_LIBJPEG)
AC_SUBST(LIBJPEG_CFLAGS)
AC_SUBST(LIBJPEG_LDFLAGS)
if test "$with_libjpeg" = "yes"; then
    AC_MSG_CHECKING([for libjpeg])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=1.0.0 jpeg; then
            HAVE_LIBJPEG=1
            LIBJPEG_CFLAGS=`pkg-config --cflags jpeg`
            LIBJPEG_LDFLAGS=`pkg-config --libs jpeg`
        fi
    fi

    if test "$HAVE_LIBJPEG" = 0; then
        if [[[ -f /usr/lib/libjpeg.a || -f /usr/lib64/libjpeg.a ]]] ; then
            HAVE_LIBJPEG=1
            LIBJPEG_CFLAGS=""
            LIBJPEG_LDFLAGS="-ljpeg";
        fi
    fi

    if test "$HAVE_LIBJPEG" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_LIBJPEG],[$HAVE_LIBJPEG],[Has LIBJPEG])
AM_CONDITIONAL([HAVE_LIBJPEG],[test $HAVE_LIBJPEG = 1])

# freetype2
HAVE_FREETYPE2=0
AC_ARG_WITH(freetype2,AS_HELP_STRING([--with-freetype2],[Enable freetype 2]),[],[with_freetype2=yes])
AC_SUBST(HAVE_FREETYPE2)
AC_SUBST(FREETYPE2_CFLAGS)
AC_SUBST(FREETYPE2_LDFLAGS)
if test "$with_freetype2" = "yes"; then
    AC_MSG_CHECKING([for freetype2])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=9.20.3 freetype2; then
            HAVE_FREETYPE2=1
            FREETYPE2_CFLAGS=`pkg-config --cflags freetype2`
            FREETYPE2_LDFLAGS=`pkg-config --libs freetype2`
        fi
    fi

    if test "$HAVE_FREETYPE2" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_FREETYPE2],[$HAVE_FREETYPE2],[Has FREETYPE2])
AM_CONDITIONAL([HAVE_FREETYPE2],[test $HAVE_FREETYPE2 = 1])

# fontconfig
HAVE_FONTCONFIG=0
AC_ARG_WITH(fontconfig,AS_HELP_STRING([--with-fontconfig],[Enable fontconfig]),[],[with_fontconfig=yes])
AC_SUBST(HAVE_FONTCONFIG)
AC_SUBST(FONTCONFIG_CFLAGS)
AC_SUBST(FONTCONFIG_LDFLAGS)
if test "$with_fontconfig" = "yes"; then
    AC_MSG_CHECKING([for fontconfig])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=2.7.3 fontconfig; then
            HAVE_FONTCONFIG=1
            FONTCONFIG_CFLAGS=`pkg-config --cflags fontconfig`
            FONTCONFIG_LDFLAGS=`pkg-config --libs fontconfig`
        fi
    fi

    if test "$HAVE_FONTCONFIG" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_FONTCONFIG],[$HAVE_FONTCONFIG],[Has FONTCONFIG])
AM_CONDITIONAL([HAVE_FONTCONFIG],[test $HAVE_FONTCONFIG = 1])

# cairo
HAVE_CAIRO=0
AC_ARG_WITH(cairo,AS_HELP_STRING([--with-cairo],[Enable cairo]),[],[with_cairo=yes])
AC_SUBST(HAVE_CAIRO)
AC_SUBST(CAIRO_CFLAGS)
AC_SUBST(CAIRO_LDFLAGS)
if test "$with_cairo" = "yes"; then
    AC_MSG_CHECKING([for cairo])
    if test "$HAVE_PKGCONFIG" = 1; then
        if pkg-config --atleast-version=1.11.2 cairo; then
            HAVE_CAIRO=1
            CAIRO_CFLAGS=`pkg-config --cflags cairo`
            CAIRO_LDFLAGS=`pkg-config --libs cairo`
        fi
    fi

    if test "$HAVE_CAIRO" = 1; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
fi
AC_DEFINE_UNQUOTED([HAVE_CAIRO],[$HAVE_CAIRO],[Has CAIRO])
AM_CONDITIONAL([HAVE_CAIRO],[test $HAVE_CAIRO = 1])

# FIXME: correct src vs build dir?
./Makeflag.gen; X=$?
if test "$X" != 0; then
    AC_MSG_ERROR([Makeflag.gen failed])
    exit 1
fi

AC_CONFIG_FILES(Makefile)
AC_OUTPUT

