AC_INIT(dash, 0.5.7)
AM_INIT_AUTOMAKE([foreign])
AC_CONFIG_SRCDIR([src/main.c])

AC_CONFIG_HEADERS(config.h)

dnl Checks for programs.
AC_PROG_CC
AC_GNU_SOURCE
AC_PROG_YACC

AC_MSG_CHECKING([for build system compiler])
if test "$cross_compiling" = yes; then
	CC_FOR_BUILD=${CC_FOR_BUILD-cc}
else
	CC_FOR_BUILD=${CC}
fi
AC_MSG_RESULT(${CC_FOR_BUILD})
AC_SUBST(CC_FOR_BUILD)

AC_MSG_CHECKING([for __attribute__((__alias__()))])
dash_cv_have_attribute_alias=no
AC_LINK_IFELSE([AC_LANG_PROGRAM([void t() {}
                                 void a() __attribute__((__alias__("t")));],
                                [a();])],
               [dash_cv_have_attribute_alias=yes])
AC_MSG_RESULT($dash_cv_have_attribute_alias)
if test "x$dash_cv_have_attribute_alias" = xyes; then
  AC_DEFINE([HAVE_ALIAS_ATTRIBUTE], 1,
            [Define if __attribute__((__alias__())) is supported])
fi

AC_ARG_ENABLE(static, AS_HELP_STRING(--enable-static, \
				     [Build statical linked program]))
if test "$enable_static" = "yes"; then
	export LDFLAGS="-static -Wl,--fatal-warnings"
fi

AC_ARG_ENABLE(fnmatch, AS_HELP_STRING(--enable-fnmatch, \
				      [Use fnmatch(3) from libc]))
AC_ARG_ENABLE(glob, AS_HELP_STRING(--enable-glob, [Use glob(3) from libc]))

dnl Checks for libraries.

dnl Checks for header files.
AC_CHECK_HEADERS(alloca.h paths.h)

dnl Check for declarations
AC_CHECK_DECL([_PATH_BSHELL],,AC_DEFINE_UNQUOTED([_PATH_BSHELL], "/bin/sh", [Define to system shell path]),[
#ifdef HAVE_PATHS_H
#include <paths.h>
#endif
])
AC_CHECK_DECL([_PATH_DEVNULL],,AC_DEFINE_UNQUOTED([_PATH_DEVNULL], "/dev/null", [Define to devnull device node path]),[
#ifdef HAVE_PATHS_H
#include <paths.h>
#endif
])
AC_CHECK_DECL([_PATH_TTY],,AC_DEFINE_UNQUOTED([_PATH_TTY], "/dev/tty", [Define to tty device node path]),[
#ifdef HAVE_PATHS_H
#include <paths.h>
#endif
])

dnl Some systems lack isblank
AC_CHECK_DECLS([isblank],,,[#include <ctype.h>])

dnl Check for sizes of types
AC_CHECK_SIZEOF([intmax_t])
AC_CHECK_SIZEOF([long long int])

dnl Select a fallback format string for intmax_t in case we don't find PRIdMAX
if test "x$ac_cv_sizeof_intmax_t" = "x$ac_cv_sizeof_long_long_int"; then
  intmax_fstr="lld"
else
  intmax_fstr="jd"
fi

dnl Check for PRIdMAX and define it to a fallback if not found
AC_CHECK_DECL([PRIdMAX],,
	[AC_DEFINE_UNQUOTED([PRIdMAX], "$intmax_fstr",
				       [Define to printf format string for intmax_t])],
        [
#include <inttypes.h>
])

dnl Checks for library functions.
AC_CHECK_FUNCS(bsearch faccessat getpwnam getrlimit isalpha killpg \
	       mempcpy \
	       sigsetmask stpcpy strchrnul strsignal strtod strtoimax \
	       strtoumax sysconf)

if test "$enable_fnmatch" = yes; then
	use_fnmatch=
	AC_CHECK_FUNCS(fnmatch, use_fnmatch=yes)
fi

if test "$use_fnmatch" = yes && test "$enable_glob" = yes; then
	AC_CHECK_FUNCS(glob)
fi

dnl Check for klibc signal.
AC_CHECK_FUNC(signal)
if test "$ac_cv_func_signal" != yes; then
	AC_CHECK_FUNC(bsd_signal,
		      [AC_DEFINE(signal, bsd_signal,
				 [klibc has bsd_signal instead of signal])])
fi

dnl Check for stat64 (dietlibc/klibc).
AC_CHECK_FUNC(stat64,, [
	AC_DEFINE(fstat64, fstat, [64-bit operations are the same as 32-bit])
	AC_DEFINE(lstat64, lstat, [64-bit operations are the same as 32-bit])
	AC_DEFINE(stat64, stat, [64-bit operations are the same as 32-bit])
	AC_DEFINE(readdir64, readdir, [64-bit operations are the same as 32-bit])
])

AC_CHECK_FUNC(open64,, [
	AC_DEFINE(open64, open, [64-bit operations are the same as 32-bit])
])

AC_ARG_WITH(libedit, AS_HELP_STRING(--with-libedit, [Compile with libedit support]))
use_libedit=
if test "$with_libedit" = "yes"; then
	AC_CHECK_LIB(edit, history_init, [
		AC_CHECK_HEADER([histedit.h], [use_libedit="yes"],
				AC_MSG_ERROR(
					[Can't find required header files.]))])
fi
if test "$use_libedit" != "yes"; then
	AC_DEFINE([SMALL], 1, [Define if you build with -DSMALL])
else
	export LIBS="$LIBS -ledit"
fi
AC_ARG_ENABLE(lineno, AS_HELP_STRING(--disable-lineno, \
				     [Disable LINENO support]))
if test "$enable_lineno" != "no"; then
	AC_DEFINE([WITH_LINENO], 1, [Define if you build with -DWITH_LINENO])
fi
AC_CONFIG_FILES([Makefile src/Makefile])
AC_OUTPUT
