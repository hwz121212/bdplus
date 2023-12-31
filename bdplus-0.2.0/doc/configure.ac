dnl library version number
m4_define([bdplus_major], 0)
m4_define([bdplus_minor], 2)
m4_define([bdplus_micro], 0)
m4_define([bdplus_version],[bdplus_major.bdplus_minor.bdplus_micro])

dnl library (.so) version
#
dnl update when making new release
#
dnl - If there are no ABI changes, increase revision.
dnl - If ABI is changed backwards-compatible way, increase current and age. Set revision to 0.
dnl - If ABI is broken, increase current and set age and revision to 0.
#
dnl Library file name will be libbdplus.(current-age).age.revision
#
m4_define([lt_current],  2)
m4_define([lt_age],      2)
m4_define([lt_revision], 0)

dnl initilization
AC_INIT([libbdplus], bdplus_version, [http://www.videolan.org/developers/libbdplus.html])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_SYSTEM
AM_INIT_AUTOMAKE([foreign tar-ustar dist-bzip2 no-dist-gzip subdir-objects])
AC_CONFIG_HEADERS(config.h)

dnl Enable silent rules only when available (automake 1.11 or later).
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

SET_FEATURES="-D_ISOC99_SOURCE -D_POSIX_C_SOURCE=200112L"

case "${host_os}" in
  "")
    SYS=unknown
    ;;
  *mingw* | *cygwin* | *wince* | *mingwce*)
    case "${host_os}" in
      *wince* | *mingwce* | *mingw32ce*)
        SYS=mingwce
        ;;
      *mingw*)
        SYS=mingw32
        AC_DEFINE([_WIN32_WINNT], 0x0501, [Define to '0x0500' for Windows XP APIs.])
        AC_DEFINE([_WIN32_IE], 0x0501, [Define to '0x0501' for IE 5.01.])
        ;;
    esac
    ;;
  *darwin*)
    SYS=darwin
    ;;
  *)
    SYS="${host_os}"
    ;;
esac

AM_CONDITIONAL(HAVE_WIN32,   test "${SYS}" = "mingw32")
AM_CONDITIONAL(HAVE_DARWIN,  test "${SYS}" = "darwin")

dnl messages
library_not_found="Could not find required library!"
function_not_found="Could not find required function!"

dnl configure options
AC_ARG_ENABLE([werror],
  [AS_HELP_STRING([--enable-werror], [set warnings as errors via -Werror @<:@default=disabled@:>@])])

AC_ARG_ENABLE([extra-warnings],
  [AS_HELP_STRING([--disable-extra-warnings], [set extra warnings @<:@default=enabled@:>@])])

AC_ARG_ENABLE([optimizations],
  [AS_HELP_STRING([--disable-optimizations], [disable optimizations @<:@default=enabled@:>@])])

dnl override platform specific check for dependent libraries
dnl otherwise libtool linking of shared libraries will
dnl fail on anything other than pass_all.
AC_CACHE_VAL(lt_cv_deplibs_check_method,
    [lt_cv_deplibs_check_method=pass_all])

dnl required programs
AC_PROG_CC
AC_PROG_LIBTOOL
AC_LIBTOOL_WIN32_DLL

dnl required types
AC_TYPE_SIGNAL

dnl required headers
AC_CHECK_HEADERS([stdarg.h sys/types.h dirent.h errno.h libgen.h malloc.h])
AC_CHECK_HEADERS([stdlib.h mntent.h linux/cdrom.h inttypes.h])
AC_CHECK_HEADERS([sys/time.h time.h])

dnl required structures
AC_STRUCT_DIRENT_D_TYPE

dnl required system services
AC_SYS_LARGEFILE

dnl required functions
AC_CHECK_FUNC([snprintf],, [AC_MSG_ERROR($function_not_found)])

dnl pthread check (not on win32)
AS_IF([test "${SYS}" != "mingw32"], [
  AC_CHECK_HEADERS([pthread.h], ,
    [AC_MSG_ERROR([pthread required])])
  AC_SEARCH_LIBS([pthread_create], [pthread], ,
    [AC_MSG_ERROR([pthread required])])
])

dnl gcrypt check

AC_ARG_WITH(libgcrypt-prefix,
            AC_HELP_STRING([--with-libgcrypt-prefix=PFX],
                           [prefix where libgcrypt is installed (optional)]),
     libgcrypt_config_prefix="$withval", libgcrypt_config_prefix="")
if test x$libgcrypt_config_prefix != x ; then
   if test x${LIBGCRYPT_CONFIG+set} != xset ; then
      LIBGCRYPT_CONFIG=$libgcrypt_config_prefix/bin/libgcrypt-config
   fi
fi

AC_PATH_PROG(LIBGCRYPT_CONFIG, libgcrypt-config, no)
if test x"$LIBGCRYPT_CONFIG" = xno; then
  AC_MSG_ERROR([libgcrypt not found on system])
else
  LIBGCRYPT_CFLAGS=`$LIBGCRYPT_CONFIG --cflags`
  LIBGCRYPT_LIBS=`$LIBGCRYPT_CONFIG --libs`
  AC_SUBST(LIBGCRYPT_CFLAGS)
  AC_SUBST(LIBGCRYPT_LIBS)
fi
AC_FUNC_STRERROR_R

dnl use re-entrant version of gcrypt_error() from gpg-error

AC_ARG_WITH(gpg-error-prefix,
            AC_HELP_STRING([--with-gpg-error-prefix=PFX],
                           [prefix where gpg-error is installed (optional)]),
     gpg_error_config_prefix="$withval", gpg_error_config_prefix="")
if test x$gpg_error_config_prefix != x ; then
   if test x${GPG_ERROR_CONFIG+set} != xset ; then
      GPG_ERROR_CONFIG=$gpg_error_config_prefix/bin/gpg-error-config
   fi
fi

AC_PATH_PROG(GPG_ERROR_CONFIG, gpg-error-config, no)
if test x"$GPG_ERROR_CONFIG" = xno; then
  AC_MSG_ERROR([gpg-error not found on system])
else
  GPG_ERROR_CFLAGS=`$GPG_ERROR_CONFIG --cflags`
  GPG_ERROR_LIBS=`$GPG_ERROR_CONFIG --libs`
  AC_SUBST(GPG_ERROR_CFLAGS)
  AC_SUBST(GPG_ERROR_LIBS)
  AC_DEFINE(HAVE_LIBGPG_ERROR, 1, [Define to 1 if you have the gpg-error library])
fi

dnl libaacs (for bdplus_test)
AC_ARG_WITH([libaacs],
  [AS_HELP_STRING([--without-libaacs], [build bdplus_test without libaacs support @<:@default=with@:>@])])
AS_IF([test "x$with_libaacs" != "xno"], [
  PKG_CHECK_MODULES([LIBAACS], [libaacs >= 0.7.0],
    [with_libaacs=yes; AC_DEFINE([HAVE_LIBAACS], [1],
        [Define to 1 to use libaacs with bdplus_test])],
    [with_libaacs=no])
])

CC_CHECK_CFLAGS_APPEND([-Wall -Wdisabled-optimization -Wpointer-arith ]dnl
[-Wredundant-decls -Wcast-qual -Wwrite-strings -Wtype-limits -Wundef ]dnl
[-Wmissing-prototypes -Wshadow])

CC_CHECK_CFLAGS_APPEND([-Werror=implicit-function-declaration ]dnl
                       [-Werror-implicit-function-declaration],
                       [break;])

CC_CHECK_WERROR
AS_IF([test "x$enable_werror" = "xyes"], [
  CFLAGS="${CFLAGS} $cc_cv_werror"
])

AS_IF([test "x$enable_extra_warnings" != "xno"], [
  CC_CHECK_CFLAGS_APPEND([-Wextra -Winline])
])

AS_IF([test "x$enable_optimizations" != "xno"], [
  CC_CHECK_CFLAGS_APPEND([-O3 -fomit-frame-pointer])
])


dnl export library version number
BDPLUS_VERSION_MAJOR=bdplus_major()
AC_SUBST(BDPLUS_VERSION_MAJOR)
BDPLUS_VERSION_MINOR=bdplus_minor()
AC_SUBST(BDPLUS_VERSION_MINOR)
BDPLUS_VERSION_MICRO=bdplus_micro()
AC_SUBST(BDPLUS_VERSION_MICRO)

dnl export library (.so) version
LT_VERSION_INFO="lt_current:lt_revision:lt_age"
AC_SUBST(LT_VERSION_INFO)

dnl generate output files
AC_SUBST(SET_FEATURES)
AC_CONFIG_FILES([Makefile src/libbdplus.pc src/libbdplus/bdplus-version.h])
AC_OUTPUT
