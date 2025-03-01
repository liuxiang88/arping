#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ(2.61)
AC_INIT(arping, 2.24, thomas@habets.se)
AC_CANONICAL_SYSTEM
AC_CONFIG_SRCDIR([src/arping.c])
AM_INIT_AUTOMAKE
AM_MAINTAINER_MODE
AC_CONFIG_HEADER(config.h)


# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_MAKE_SET

# Required to define __USE_MISC, which in turn typedefs uint, which
# is used by libnet headers.
# This "just works" until running GCC with -std=c99, where uint is
# no longer defined by default.
CPPFLAGS="$CPPFLAGS -D_DEFAULT_SOURCE=1"

AC_DEFUN([CHECK_COMPILER_OPTION], [
    AC_MSG_CHECKING([compiler option $1])
    OLD_CFLAGS="$CFLAGS"
    CFLAGS="$CFLAGS $1"
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[]], [[]])],
    [AC_MSG_RESULT(yes)],
    [
        AC_MSG_RESULT(no)]
        CFLAGS="$OLD_CFLAGS"
    )
])

CHECK_COMPILER_OPTION([-std=c99])

# Checks for libraries.
AC_CHECK_LIB([m], [sqrt])
AC_CHECK_LIB([socket], [socket])
AC_CHECK_LIB([nsl], [gethostbyname])
AC_CHECK_LIB([cap], [cap_init])
AC_CHECK_LIB([seccomp], [seccomp_init])
AC_CHECK_LIB([rt], [clock_gettime])
AC_CHECK_LIB([net], [libnet_name_resolve],
      [AC_MSG_ERROR([libnet 1.0.x found. Arping 2.x requires libnet 1.1 or newer])])
AC_CHECK_LIB([net], [libnet_init], ,[AC_MSG_ERROR([libnet 1.1.x not found])])
AC_CHECK_LIB([pcap], [pcap_open_live], ,[AC_MSG_ERROR([libpcap not found])])
AC_SEARCH_LIBS([clock_gettime], [rt])

AC_ARG_ENABLE([seccomp], [Enable seccomp priv drops by default (-z to turn on, -Z for off)], [
   if test x"$enableval" = x"yes"; then
      if test "$ac_cv_lib_seccomp_seccomp_init" = "no"; then
         AC_MSG_ERROR([--enable-seccomp given but seccomp libraries not present])
      fi
      AC_DEFINE([DEFAULT_SECCOMP], [1], [Enable seccomp by default])
   else
      AC_DEFINE([DEFAULT_SECCOMP], [0], [Enable seccomp by default])
   fi
], [
   AC_DEFINE([DEFAULT_SECCOMP], [0], [Disable seccomp by default])
])
# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([\
arpa/inet.h \
getopt.h \
netinet/in.h \
seccomp.h \
stdlib.h \
sys/socket.h \
time.h \
grp.h \
sys/time.h \
sys/types.h \
sys/param.h \
sys/capability.h \
stdint.h \
libnet.h \
win32/libnet.h \
sys/random.h \
net/bpf.h \
pwd.h \
unistd.h])

# Libnet include file is not optional
if test ! x$ac_cv_header_libnet_h = xyes; then
  AC_MSG_ERROR([Can't find libnet.h. See INSTALL file for where to get Libnet])
fi

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_HEADER_TIME
AC_C_VOLATILE
AC_TYPE_UINT8_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T

# Checks for library functions.
AC_PROG_GCC_TRADITIONAL
AC_FUNC_SELECT_ARGTYPES
AC_FUNC_SETVBUF_REVERSED
AC_TYPE_SIGNAL
AC_CHECK_FUNCS([gettimeofday memset select strchr strdup strerror strstr \
getifaddrs cap_init pcap_create pcap_list_tstamp_types pcap_set_immediate_mode \
pledge unveil pcap_findalldevs getrandom strtoll])

if test x$ac_cv_func_getifaddrs = xyes; then
    AC_LIBOBJ([findif_getifaddrs])
else
    case "$target_os" in
        *linux*)
            AC_LIBOBJ([findif_linux])
            ;;
        *freebsd*|*openbsd*|*darwin*)
            AC_LIBOBJ([findif_sysctl])
            ;;
        *solaris*)
            AC_LIBOBJ([findif_bsdroute])
            ;;
        *)
            AC_LIBOBJ([findif_other])
            ;;
    esac
fi

AC_CACHE_CHECK([for libnet_init signature has const],
		    ac_cv_have_libnet_init_const, [
		    AC_TRY_LINK([#include<libnet.h>],
[libnet_t*libnet_init(int a,const char* b,char* c);],
  [ ac_cv_have_libnet_init_const="yes" ],
  [ ac_cv_have_libnet_init_const="no" ])
])
if test x$ac_cv_have_libnet_init_const = xyes; then
   AC_DEFINE([HAVE_LIBNET_INIT_CONST], [1], [Libnet init takes const char device])
fi

AC_CACHE_CHECK([for libnet_name2addr4 signature has const],
		    ac_cv_have_libnet_name2addr4_const, [
		    AC_TRY_LINK([#include<libnet.h>],
[uint32_t libnet_name2addr4(libnet_t*a, const char* b,uint8_t c);],
  [ ac_cv_have_libnet_name2addr4_const="yes" ],
  [ ac_cv_have_libnet_name2addr4_const="no" ])
])
if test x$ac_cv_have_libnet_name2addr4_const = xyes; then
   AC_DEFINE([HAVE_LIBNET_NAME2ADDR4_CONST], [1], [Libnet name2addr4 takes const char* addr])
fi


# check for CLOCK_MONOTONIC
AC_CHECK_DECL([CLOCK_MONOTONIC], [], [],
              [
                #include <time.h>
                #include <sys/time.h>
              ]
)
AM_CONDITIONAL(HAVE_CLOCK_MONOTONIC, test x$ac_cv_have_decl_CLOCK_MONOTONIC = xyes)
if test x$ac_cv_have_decl_CLOCK_MONOTONIC = xyes; then
   AC_DEFINE([HAVE_CLOCK_MONOTONIC], [1], [Monotonic clock])
fi

AC_ARG_ENABLE(hardened,
    AS_HELP_STRING([--enable-hardened], [Enable security build options]),
    [
        CHECK_COMPILER_OPTION([-fstack-clash-protection])
        CHECK_COMPILER_OPTION([-fstack-protector-strong])
        CHECK_COMPILER_OPTION([-mbranch-protection=standard])
        CHECK_COMPILER_OPTION([-fcf-protection=full])
        CHECK_COMPILER_OPTION([-fstrict-flex-arrays=3])
        CHECK_COMPILER_OPTION([-fPIE -pie])
        CHECK_COMPILER_OPTION([-Wl,-z,nodlopen])
        CHECK_COMPILER_OPTION([-Wl,-z,noexecstack])
        CHECK_COMPILER_OPTION([-Wl,-z,relro])
        CHECK_COMPILER_OPTION([-Wl,-z,noexecheap])
        CHECK_COMPILER_OPTION([-Wl,-z,now])
        CHECK_COMPILER_OPTION([-fstack-protector-all])
        CHECK_COMPILER_OPTION([--param ssp-buffer-size=1])
        CHECK_COMPILER_OPTION([-U _FORTIFY_SOURCE -D_FORTIFY_SOURCE=3])
])
AC_ARG_ENABLE(warnings,
    AS_HELP_STRING([--enable-warnings], [Enable build warnings]),
    [
        CHECK_COMPILER_OPTION([-Wall])
        CHECK_COMPILER_OPTION([-Wextra])
        CHECK_COMPILER_OPTION([-Wpedantic])
        CHECK_COMPILER_OPTION([-Wstack-protector])
        CHECK_COMPILER_OPTION([-Wformat])
        CHECK_COMPILER_OPTION([-Wformat-security])
        CHECK_COMPILER_OPTION([-Wformat-y2k])
        CHECK_COMPILER_OPTION([-Werror=format-security])
        CHECK_COMPILER_OPTION([-Wconversion])
        CHECK_COMPILER_OPTION([-Wsign-conversion])
        CHECK_COMPILER_OPTION([-Wimplicit-fallthrough])
        CHECK_COMPILER_OPTION([-Wtrampolines])
])

AC_CONFIG_FILES([Makefile])
AC_CONFIG_FILES([src/Makefile])
AC_CONFIG_FILES([doc/Makefile])
AC_OUTPUT

echo "
  $PACKAGE_NAME version $PACKAGE_VERSION
  Prefix.........: $prefix
  Debug Build....: $debug
  C Compiler.....: $CC $CFLAGS $CPPFLAGS
  Linker.........: $LD $LDFLAGS $LIBS
"
