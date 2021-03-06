AC_INIT([yara], [3.7.0], [vmalvarez@virustotal.com])

AM_SILENT_RULES([yes])
AC_CONFIG_SRCDIR([yara.c])

AM_INIT_AUTOMAKE([-Wall -Werror foreign subdir-objects])

AC_CONFIG_MACRO_DIR([m4])

# AC_PROG_CC sets CFLAGS to "-g -O2" if it wasn't previously set. Let's set
# an empty CFLAGS
: ${CFLAGS=""}

# automake 1.12 seems to require AM_PROG_AR, but automake 1.11 doesn't
# recognize it
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])

AC_PROG_CC
AM_PROG_CC_C_O
AM_PROG_LEX
AC_PROG_YACC
AC_SYS_LARGEFILE

LT_INIT
AC_PROG_LIBTOOL

AC_CANONICAL_HOST

case $host_alias in
     i?86-*-mingw*) CFLAGS="$CFLAGS -D__MINGW_USE_VC2005_COMPAT" ;;
esac

case $host_os in
    darwin*) CFLAGS="$CFLAGS -I/opt/local/include"
             # Starting with Mac OS X 10.11 (El Capitan) the OpenSSL headers
             # are in /usr/local/opt/openssl/include
             CFLAGS="$CFLAGS -DUSE_MACH_PROC -I/usr/local/opt/openssl/include"
             LDFLAGS="$LDFLAGS -L/usr/local/opt/openssl/lib"
             proc_interface=mach
             jemalloc_prefix=je_ ;;
    mingw*) CFLAGS="$CFLAGS -DUSE_WINDOWS_PROC"
            proc_interface=windows
            jemalloc_prefix= ;;
    linux*|netbsd*|dragonfly*|kfreebsd*)
            CFLAGS="$CFLAGS -DUSE_LINUX_PROC"
            proc_interface=linux
            jemalloc_prefix=  ;;
    freebsd*)
            CFLAGS="$CFLAGS -DUSE_FREEBSD_PROC"
            proc_interface=freebsd
            jemalloc_prefix= ;;
    openbsd*)
            CFLAGS="$CFLAGS -DUSE_OPENBSD_PROC"
            proc_interface=openbsd
            jemalloc_prefix= ;;
    *)
            CFLAGS="$CFLAGS -DUSE_NO_PROC"
            proc_interface=none
            jemalloc_prefix= ;;
esac

AC_C_BIGENDIAN

ACX_PTHREAD(
    [LIBS="$PTHREAD_LIBS $LIBS"
     CFLAGS="$CFLAGS $PTHREAD_CFLAGS"
     CC="$PTHREAD_CC"],
    [AC_MSG_ERROR([pthread API support is required.])])

AC_CHECK_LIB(m, isnan)
AC_CHECK_LIB(m, log2)
AC_CHECK_FUNCS([strlcpy strlcat memmem timegm _mkgmtime])

AC_ARG_ENABLE([debug],
  [AS_HELP_STRING([--enable-debug], [compiles with -g option])],
  [if test x$enableval = xyes; then
    debug=true
  fi])

AC_ARG_ENABLE([optimization],
  [AS_HELP_STRING([--disable-optimization], [disable compiler optimizations with -O0])],
  [if test x$enableval = xyes; then
    optimization=false
  fi],
  [optimization=true])

AC_ARG_ENABLE([address-sanitizer],
  [AS_HELP_STRING([--enable-address-sanitizer], [compiles with -fsanitize=address])],
  [if test x$enableval = xyes; then
    address_sanitizer=true
  fi])

AC_ARG_ENABLE([profiling],
  [AS_HELP_STRING([--enable-profiling], [enable profiling support])],
  [if test x$enableval = xyes; then
    profiling_enabled=true
    CFLAGS="$CFLAGS -DPROFILING_ENABLED"
  fi])

AC_ARG_ENABLE([dmalloc],
  [AS_HELP_STRING([--enable-dmalloc],
    [enable dmalloc to debug heap-related issues])],
  [if test x$enableval = xyes; then
    AC_CHECK_LIB(dmalloc, dmalloc_malloc,,
      AC_MSG_ERROR([please install dmalloc library]))
    CFLAGS="$CFLAGS -DDMALLOC"
  fi])

AC_ARG_ENABLE([jemalloc],
  [AS_HELP_STRING([--enable-jemalloc],
    [enable jemalloc to debug heap-related issues])],
  [if test x$enableval = xyes; then
      mallctl=mallctl
      AC_CHECK_LIB(jemalloc, $jemalloc_prefix$mallctl,,
        AC_MSG_ERROR([please install jemalloc library]))
    CFLAGS="$CFLAGS -DJEMALLOC -Dmallctl=$jemalloc_prefix$mallctl"
  fi])

AC_ARG_ENABLE([cuckoo],
  [AS_HELP_STRING([--enable-cuckoo], [enable cuckoo module])],
  [if test x$enableval = xyes; then
    build_cuckoo_module=true
    AC_CHECK_HEADERS([jansson.h],,
      AC_MSG_ERROR([please install Jansson library]))
    AC_CHECK_LIB(jansson, json_loadb,,
      AC_MSG_ERROR([please install Jansson library]))
    CFLAGS="$CFLAGS -DCUCKOO_MODULE"
    PC_REQUIRES_PRIVATE="$PC_REQUIRES_PRIVATE jansson"
  fi])

AC_ARG_ENABLE([magic],
  [AS_HELP_STRING([--enable-magic], [enable magic module])],
  [if test x$enableval = xyes; then
    build_magic_module=true
    AC_CHECK_HEADERS([magic.h],,
      AC_MSG_ERROR([please install libmagic library]))
    AC_CHECK_LIB(magic, magic_open,,
      AC_MSG_ERROR([please install libmagic library]))
    CFLAGS="$CFLAGS -DMAGIC_MODULE"
    PC_LIBS_PRIVATE="$PC_LIBS_PRIVATE -lmagic"
  fi])

AC_ARG_ENABLE([dotnet],
  [AS_HELP_STRING([--enable-dotnet], [enable dotnet module])],
  [if test x$enableval = xyes; then
    build_dotnet_module=true
    CFLAGS="$CFLAGS -DDOTNET_MODULE"
  fi])

AC_ARG_ENABLE([macho],
  [AS_HELP_STRING([--enable-macho], [enable macho module])],
  [if test x$enableval = xyes; then
    build_macho_module=true
    CFLAGS="$CFLAGS -DMACHO_MODULE"
  fi])

AC_ARG_WITH([crypto],
  AS_HELP_STRING([--without-crypto],
    [ignore presence of OpenSSL and disable it]))

AC_ARG_ENABLE([dex],
  [AS_HELP_STRING([--enable-dex], [enable dex module])],
  [if test x$enableval = xyes; then
    build_dex_module=true
    CFLAGS="$CFLAGS -DDEX_MODULE"
  fi])

AS_IF([test "x$with_crypto" != "xno"],
  [
    AC_CHECK_HEADERS([openssl/md5.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/sha.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/asn1.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/crypto.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/bio.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/pkcs7.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/x509.h],, [have_crypto=no])
    AC_CHECK_HEADERS([openssl/safestack.h],, [have_crypto=no])
    AC_CHECK_LIB(crypto, MD5_Init,, [have_crypto=no])
    AC_CHECK_LIB(crypto, MD5_Update,, [have_crypto=no])
    AC_CHECK_LIB(crypto, MD5_Final,, [have_crypto=no])
    AC_CHECK_LIB(crypto, SHA256_Init,, [have_crypto=no])
    AC_CHECK_LIB(crypto, SHA256_Update,, [have_crypto=no])
    AC_CHECK_LIB(crypto, SHA256_Final,, [have_crypto=no])
  ],
  [
    have_crypto=no
  ])


AS_IF([test "x$have_crypto" = "xno"],
  [
    AS_IF([test "x$with_crypto" = "xyes"],
      [
        AC_MSG_ERROR([please install OpenSSL library])
      ],
      [
        AC_MSG_WARN([

*****************************************************************
  Could not find OpenSSL library. Some features in "pe" module
  have been disabled. If you want to enable all features please
  install OpenSSL and run this script again.
*****************************************************************
])
	      AC_MSG_CHECKING([for Microsoft Crypto API])
        AC_CHECK_HEADERS([wincrypt.h],
          [
	          AC_MSG_RESULT([The "hash" module functions will be provided through the Microsoft Crypto API])
            # FIXME: Add PC_LIBS_PRIVATE entries?
            build_hash_module=true
          ],
          [],
          [#include <windows.h>])
	      AC_MSG_CHECKING([for MacOSX Common Crypto API])
        AC_CHECK_HEADERS([CommonCrypto/CommonCrypto.h],
          [
	          AC_MSG_RESULT([
*****************************************************************
  As an alternative to OpenSSL the "hash" module functions will
  be provided through the MacOSX Common Crypto API.
*****************************************************************
])
            # FIXME: Add PC_LIBS_PRIVATE entries?
            build_hash_module=true
          ])
        AS_IF([test x$build_hash_module = xtrue],
          [
            build_hash_module=true
            CFLAGS="$CFLAGS -DHASH_MODULE"
          ],
          [
            AC_MSG_WARN([

*****************************************************************
  Could not find alternative APIs for hash functions. The "hash"
  module has been disabled.
*****************************************************************
            ])
          ])
      ])
  ],
  [
    build_hash_module=true
    CFLAGS="$CFLAGS -DHASH_MODULE"
    PC_REQUIRES_PRIVATE="$PC_REQUIRES_PRIVATE libcrypto"
  ])

AM_CONDITIONAL([DEBUG], [test x$debug = xtrue])
AM_CONDITIONAL([PROFILING_ENABLED], [test x$profiling_enabled = xtrue])
AM_CONDITIONAL([OPTIMIZATION], [test x$optimization = xtrue])
AM_CONDITIONAL([ADDRESS_SANITIZER], [test x$address_sanitizer = xtrue])
AM_CONDITIONAL([CUCKOO_MODULE], [test x$build_cuckoo_module = xtrue])
AM_CONDITIONAL([MAGIC_MODULE], [test x$build_magic_module = xtrue])
AM_CONDITIONAL([HASH_MODULE], [test x$build_hash_module = xtrue])
AM_CONDITIONAL([DOTNET_MODULE], [test x$build_dotnet_module = xtrue])
AM_CONDITIONAL([MACHO_MODULE], [test x$build_macho_module = xtrue])
AM_CONDITIONAL([GCC], [test "x$GCC" = xyes])
AM_CONDITIONAL([DEX_MODULE], [test x$build_dex_module = xtrue])

AM_CONDITIONAL([USE_WINDOWS_PROC], [test x$proc_interface = xwindows ])
AM_CONDITIONAL([USE_LINUX_PROC], [test x$proc_interface = xlinux ])
AM_CONDITIONAL([USE_FREEBSD_PROC], [test x$proc_interface = xfreebsd ])
AM_CONDITIONAL([USE_OPENBSD_PROC], [test x$proc_interface = xopenbsd ])
AM_CONDITIONAL([USE_MACH_PROC], [test x$proc_interface = xmach ])
AM_CONDITIONAL([USE_NO_PROC], [test x$proc_interface = xnone ])
AS_IF(
  [test x$proc_interface != xnone],[AC_DEFINE([HAVE_SCAN_PROC_IMPL],[1])],
  [test x$proc_interface = xnone],[AC_DEFINE([HAVE_SCAN_PROC_IMPL],[0])])

AC_SUBST([PC_REQUIRES_PRIVATE])
AC_SUBST([PC_LIBS_PRIVATE])

AC_CONFIG_FILES([Makefile])
AC_CONFIG_FILES([libyara/Makefile])
AC_CONFIG_FILES([libyara/yara.pc])

AC_OUTPUT
