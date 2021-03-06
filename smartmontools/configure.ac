#
# $Id: configure.ac 4549 2017-10-10 19:09:54Z chrfranke $
#
dnl Process this file with autoconf to produce a configure script.
AC_PREREQ([2.60])
AC_INIT(smartmontools, 6.6, smartmontools-support@listi.jpberlin.de)
AM_INIT_AUTOMAKE([1.10 foreign])

smartmontools_cvs_tag=`echo '$Id: configure.ac 4549 2017-10-10 19:09:54Z chrfranke $'`
smartmontools_release_date=2016-05-07
smartmontools_release_time="11:17:46 UTC"

AC_DEFINE_UNQUOTED(SMARTMONTOOLS_CONFIGURE_ARGS, "$ac_configure_args",            [smartmontools Configure Arguments])
AC_DEFINE_UNQUOTED(SMARTMONTOOLS_RELEASE_DATE,   "$smartmontools_release_date",   [smartmontools Release Date])
AC_DEFINE_UNQUOTED(SMARTMONTOOLS_RELEASE_TIME,   "$smartmontools_release_time",   [smartmontools Release Time])
AC_DEFINE_UNQUOTED(CONFIG_H_CVSID,               "$smartmontools_cvs_tag",        [smartmontools CVS Tag])
AC_DEFINE_UNQUOTED(PACKAGE_HOMEPAGE,             "http://www.smartmontools.org/", [smartmontools Home Page])

AC_CONFIG_SRCDIR([smartctl.cpp])
AC_CONFIG_HEADER([config.h])

AM_MAINTAINER_MODE

AC_LANG([C++])
dnl Checks for programs.
AC_PROG_CXX
AM_PROG_AS
AC_PROG_INSTALL

m4_pattern_forbid([^PKG_PROG_])
if test "$cross_compiling" = "no"; then
  m4_ifdef([PKG_PROG_PKG_CONFIG], [PKG_PROG_PKG_CONFIG],
    [AC_MSG_WARN([m4/pkg.m4 missing, systemd detection disabled])])
fi

AC_ARG_VAR(WINDMC, [Windows message compiler command])
AC_ARG_VAR(WINDRES, [Windows resource compiler command])
AC_ARG_VAR(MAKENSIS, [NSIS compiler command])

AC_CANONICAL_HOST
case "${host}" in
  *-*-mingw*)
    AC_CHECK_TOOL(WINDMC, [windmc])
    AC_CHECK_TOOL(WINDRES, [windres])

    AC_MSG_CHECKING([for makensis])
    if test -z "$MAKENSIS"; then
      if test -n "$PROGRAMFILES" && "$PROGRAMFILES/NSIS/makensis" -VERSION >/dev/null 2>&1; then
        MAKENSIS="$PROGRAMFILES/NSIS/makensis"
      elif makensis -VERSION >/dev/null 2>&1; then
        MAKENSIS=makensis
      fi
    fi
    AC_MSG_RESULT([${MAKENSIS:-no}])
    ;;
esac

# Check for SVN.
AC_MSG_CHECKING([whether this is a build from SVN])
is_svn_build=no
svn_deps=
if test -f "$srcdir/.svn/wc.db"; then
  # SVN 1.7, 1.8 working copy
  svn_deps='${srcdir}/.svn/wc.db'
elif test -f "${srcdir}/.svn/entries"; then
  # SVN <= 1.6 working copy (SVN 1.7 has empty entries file)
  svn_deps='${srcdir}/.svn/entries'
fi
if test -n "$svn_deps"; then
  is_svn_build=unknown
  if (cd "$srcdir" && svn --version && svnversion && svn info) >/dev/null 2>&1; then
    is_svn_build=yes
  fi
fi
AC_SUBST([svn_deps])
AM_CONDITIONAL(IS_SVN_BUILD, [test "$is_svn_build" = "yes"])
AC_MSG_RESULT([$is_svn_build])

# Note: On Linux, clock_gettime() requires -lrt which implies -lpthreads
# Check ommitted for now, gettimeofday() provides reasonable precision
# AC_SEARCH_LIBS(clock_gettime, rt)

dnl Checks for header files.
AC_CHECK_HEADERS([locale.h])
AC_CHECK_HEADERS([dev/ata/atavar.h])
dnl we need [u]int64_t and friends.
AC_CHECK_HEADERS([inttypes.h])		dnl C99, UNIX98, solaris 2.6+
AC_CHECK_HEADERS([stdint.h])		dnl C99
AC_CHECK_HEADERS([sys/inttypes.h])	dnl pre-UNIX98
AC_CHECK_HEADERS([sys/int_types.h])	dnl pre-UNIX98, solaris 2.6+
dnl Check for FreeBSD twe include files...currently missing on 5.2, but should be there
AC_CHECK_HEADERS([sys/tweio.h])
AC_CHECK_HEADERS([sys/twereg.h])
dnl Check for FreeBSD twa include files...
AC_CHECK_HEADERS([sys/tw_osl_ioctl.h])
dnl This header file is needed for cciss_ioctl.h at least on SuSE LINUX
AC_CHECK_HEADERS([linux/compiler.h])
dnl Check for the FreeBSD CCISS system header and use internal one if not found
AC_CHECK_HEADERS([dev/ciss/cissio.h],
    [AC_DEFINE([CISS_LOCATION],[<dev/ciss/cissio.h>],[freebsd ciss header location])],
    [AC_DEFINE([CISS_LOCATION],["cissio_freebsd.h"],[freebsd ciss header location])]
    )
dnl Check for Linux CCISS include file
AC_CHECK_HEADERS([linux/cciss_ioctl.h], [], [], [AC_INCLUDES_DEFAULT
#ifdef HAVE_LINUX_COMPILER_H
# include <linux/compiler.h>
#endif
])
dnl Check for Windows DDK and WMI header files
AC_CHECK_HEADERS([ntdddisk.h ddk/ntdddisk.h], [], [], [AC_INCLUDES_DEFAULT
#include <windows.h>
])
AC_CHECK_HEADERS([wbemcli.h])

dnl Checks for typedefs, structures, and compiler characteristics.
AC_CHECK_TYPES([int64_t, uint64_t])

dnl Checks for library functions.
AC_CHECK_FUNCS([getopt_long], [need_getopt_long=no], [need_getopt_long=yes])
AM_CONDITIONAL(NEED_GETOPT_LONG, [test "$need_getopt_long" = "yes"])
AC_CHECK_FUNCS([regcomp], [need_regex=no], [need_regex=yes])
AM_CONDITIONAL(NEED_REGEX, [test "$need_regex" = "yes"])

AC_CHECK_FUNCS([sigset])
AC_CHECK_FUNCS([strtoull])
AC_CHECK_FUNCS([uname])
AC_CHECK_FUNCS([clock_gettime ftime gettimeofday])

# Check byte ordering (defines WORDS_BIGENDIAN)
AC_C_BIGENDIAN

# check for __attribute__((packed))
# (sizeof() check is required to avoid false positives if other
# __attribute__((x)) are supported)
AC_MSG_CHECKING([whether $CXX supports __attribute__((packed))])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(, [[
    struct s { char a; short b; } __attribute__((packed));
    typedef char t[sizeof(struct s) == 3 ? 1 : -1];]])],
  [gcc_have_attr_packed=yes], [gcc_have_attr_packed=no])
AC_SUBST(gcc_have_attr_packed)
if test "$gcc_have_attr_packed" = "yes"; then
  AC_DEFINE(HAVE_ATTR_PACKED, 1, [Define to 1 if C++ compiler supports __attribute__((packed))])
fi
AC_MSG_RESULT([$gcc_have_attr_packed])

AC_SUBST(CPPFLAGS)
AC_SUBST(LDFLAGS)
AC_SUBST(ASFLAGS)

AC_ARG_WITH(systemdsystemunitdir,
  [AS_HELP_STRING([--with-systemdsystemunitdir@<:@=DIR|auto|yes|no@:>@], [Location of systemd service files [auto]])],
  [], [with_systemdsystemunitdir=auto])

systemdsystemunitdir=
case "$with_systemdsystemunitdir" in
 auto|yes)
   if test -n "$PKG_CONFIG"; then
     AC_MSG_CHECKING([for systemdsystemunitdir])
     systemdsystemunitdir=`$PKG_CONFIG --variable=systemdsystemunitdir systemd 2>/dev/null`
     AC_MSG_RESULT([${systemdsystemunitdir:-no}])
   fi
   case "$with_systemdsystemunitdir:$sysconfdir:$systemdsystemunitdir" in
     yes:*:) AC_MSG_ERROR([Location of systemd service files not found]) ;;
     yes:*:*|auto:*:|auto:/etc:*) ;;
     *) systemdsystemunitdir='${prefix}'$systemdsystemunitdir ;;
   esac ;;
 no) ;;
 *) systemdsystemunitdir="$with_systemdsystemunitdir" ;;
esac
AC_SUBST(systemdsystemunitdir)
AM_CONDITIONAL(INSTALL_SYSTEMDUNIT, [test -n "$systemdsystemunitdir"])

AC_ARG_WITH(systemdenvfile,
  [AS_HELP_STRING([--with-systemdenvfile@<:@=FILE|auto|yes|no@:>@], [Path of systemd EnvironmentFile [auto]])],
  [], [with_systemdenvfile=auto])

systemdenvfile=
case "$with_systemdenvfile:$cross_compiling:$systemdsystemunitdir" in
  auto:no:?*|yes:*:?*)
    AC_MSG_CHECKING([for path of systemd EnvironmentFile])
    for dir in sysconfig default; do
      if test -d /etc/$dir; then
        systemdenvfile='${sysconfdir}'/$dir/smartmontools
        break
      fi
    done
    AC_MSG_RESULT([${systemdenvfile:-no}])
    case "$with_systemdenvfile:$systemdenvfile" in
      yes:) AC_MSG_ERROR([Path of systemd EnvironmentFile not found]) ;;
    esac ;;
  auto:*|no:*) ;;
  *:*:) AC_MSG_ERROR([Location of systemd service files not found]) ;;
  *) systemdenvfile="$with_systemdenvfile"
esac
AC_SUBST(systemdenvfile)

AC_ARG_WITH(initscriptdir,
  [AS_HELP_STRING([--with-initscriptdir@<:@=DIR|auto|yes|no@:>@], [Location of init scripts [auto]])],
  [], [with_initscriptdir=auto])

initddir=
case "$with_initscriptdir:$cross_compiling:$systemdsystemunitdir" in
  auto:no:|yes:*)
    AC_MSG_CHECKING([for init (rc) directory])
    for dir in rc.d/init.d init.d rc.d; do
      if test -d /etc/$dir; then
        initddir='${sysconfdir}'/$dir
        break
      fi
    done
    AC_MSG_RESULT([${initddir:-no}])
    case "$with_initscriptdir:$initddir" in
      yes:) AC_MSG_ERROR([Location of init scripts not found]) ;;
    esac ;;
  auto:*|no:*) ;;
  *) initddir="$with_initscriptdir" ;;
esac
AC_SUBST(initddir)
AM_CONDITIONAL(INSTALL_INITSCRIPT, [test -n "$initddir"])

# use different init script templates for different OS
case "${host}" in
  *-*-freebsd*)
    initdfile="smartd.freebsd.initd"
    ;;
  *-apple-darwin*)
    initdfile="com.smartmontools.smartd.plist"
    ;;
  *)
    initdfile="smartd.initd"
    ;;
esac
AC_SUBST(initdfile)

AC_ARG_WITH(exampledir,
  [AS_HELP_STRING([--with-exampledir=DIR], [Location of example scripts [DOCDIR/examplescripts]])],
  [exampledir="$withval"], [exampledir='${docdir}/examplescripts'])
AC_SUBST(exampledir)

drivedbdir='${datadir}/${PACKAGE}'
AC_ARG_WITH(drivedbdir,
  [AS_HELP_STRING([--with-drivedbdir@<:@=DIR|yes|no@:>@], [Location of drive database file [DATADIR/smartmontools]])],
  [case "$withval" in yes) ;; no) drivedbdir= ;; *) drivedbdir="$withval" ;; esac])
AC_SUBST(drivedbdir)
AM_CONDITIONAL(ENABLE_DRIVEDB, [test -n "$drivedbdir"])

drivedb_version=$VERSION
AC_ARG_WITH(update-smart_drivedb,
  [AS_HELP_STRING([--with-update-smart-drivedb@<:@=yes|no|X.Y@:>@],
    [Install update-smart-drivedb script (and backport it to branches/RELEASE_X_Y_DRIVEDB) [yes]])],
  [ case "$withval" in
      yes|no) ;;
      5.4[[0-3]]|6.[[0-9]]) drivedb_version=$withval; with_update_smart_drivedb=yes ;;
      *) AC_MSG_ERROR([Invalid drivedb branch version: $withval]) ;;
    esac
  ],
  [with_update_smart_drivedb=yes])
test -n "$drivedbdir" || with_update_smart_drivedb=no
AC_SUBST(with_update_smart_drivedb)
AM_CONDITIONAL(ENABLE_UPDATE_SMART_DRIVEDB, [test "$with_update_smart_drivedb" = "yes"])

gnupg="gpg"
# Also check for '--with-gnupg[=yes]' because 'yes' is a valid command with infinite output
AC_ARG_WITH(gnupg,
  [AS_HELP_STRING([--with-gnupg@<:@=FILE|yes|no@:>@], [GnuPG used to verify drivedb.h [gpg]])],
  [case "$withval" in yes) ;; no) gnupg= ;; *) gnupg="$withval" ;; esac], [])
AC_SUBST(gnupg)

case "$with_update_smart_drivedb:$gnupg" in
  no:?*)
    AC_MSG_ERROR([
'--without-update-smart-drivedb' now requires '--without-gnupg'.
NEWS: update-smart-drivedb now verifies the downloaded drivedb.h file with GnuPG.])
    ;;
esac

AC_ARG_WITH(smartdscriptdir,
  [AS_HELP_STRING([--with-smartdscriptdir=DIR], [Location of smartd_warning.sh script [SYSCONFDIR]])],
  [smartdscriptdir="$withval"], [smartdscriptdir='${sysconfdir}'])
AC_SUBST(smartdscriptdir)

AC_ARG_WITH(smartdplugindir,
  [AS_HELP_STRING([--with-smartdplugindir=@<:@DIR|no@:>@],
    [Location of smartd_warning.sh plugin scripts [SMARTDSCRIPTDIR/smartd_warning.d]])],
  [smartdplugindir=; test "$withval" != "no" && smartdplugindir="$withval"],
  [smartdplugindir='${smartdscriptdir}/smartd_warning.d'])
AC_SUBST(smartdplugindir)

AC_ARG_WITH(scriptpath,
  [AS_HELP_STRING([--with-scriptpath=@<:@PATH|no@:>@],
    [PATH variable set within scripts [/usr/bin:/bin]])],
  [scriptpath=; test "$withval" != "no" && scriptpath="$withval"],
  [scriptpath="/usr/bin:/bin"])
AC_SUBST(scriptpath)
AM_CONDITIONAL(ENABLE_SCRIPTPATH, [test -n "$scriptpath"])

savestates=
AC_ARG_WITH(savestates,
  [AS_HELP_STRING([--with-savestates@<:@=PREFIX|yes|no@:>@],
    [Enable default smartd state files [no] (yes=LOCALSTATEDIR/lib/smartmontools/smartd.)])],
  [case "$withval" in yes) savestates='${localstatedir}/lib/${PACKAGE}/smartd.' ;;
                      no) ;; *) savestates="$withval" ;; esac])
savestatesdir="${savestates%/*}"
AC_SUBST(savestates)
AC_SUBST(savestatesdir)
AM_CONDITIONAL(ENABLE_SAVESTATES, [test -n "$savestates"])

attributelog=
AC_ARG_WITH(attributelog,
  [AS_HELP_STRING([--with-attributelog@<:@=PREFIX|yes|no@:>@],
    [Enable default smartd attribute log files [no] (yes=LOCALSTATEDIR/lib/smartmontools/attrlog.)])],
  [case "$withval" in yes) attributelog='${localstatedir}/lib/${PACKAGE}/attrlog.' ;;
                      no) ;; *) attributelog="$withval" ;; esac])
attributelogdir="${attributelog%/*}"
AC_SUBST(attributelog)
AC_SUBST(attributelogdir)
AM_CONDITIONAL(ENABLE_ATTRIBUTELOG, [test -n "$attributelog"])

AC_ARG_ENABLE(sample,
  [AS_HELP_STRING([--enable-sample], [Enables appending .sample to the installed smartd rc script and configuration file])],
  [smartd_suffix=; test "$enableval" = "yes" && smartd_suffix=".sample"],
  [smartd_suffix=;])
AC_SUBST(smartd_suffix)

AC_ARG_WITH(os-deps,
  [AS_HELP_STRING([--with-os-deps='os_module.o ...'], [Specify OS dependent module(s) [guessed]])],
  [ for x in $with_os_deps; do
      case $x in
        *.o) ;;
        *) AC_MSG_ERROR([non-object file specified by --with-os-deps]) ;;
      esac
    done
  ],[])

AC_ARG_WITH(selinux,
  [AS_HELP_STRING([--with-selinux@<:@=yes|no@:>@], [Enables SELinux support [no]])],
  [ if test "$withval" = "yes"; then
      AC_CHECK_HEADERS([selinux/selinux.h], [], [AC_MSG_ERROR([Missing SELinux header files])])
      AC_CHECK_LIB(selinux, matchpathcon, [], [AC_MSG_ERROR([Missing or incorrect SELinux library files])])
    fi
  ],[])
AC_SUBST(with_selinux)
if test "$with_selinux" = "yes"; then
  AC_DEFINE(WITH_SELINUX, 1, [Define to 1 if SELinux support is enabled])
fi

AC_ARG_WITH(libcap-ng,
  [AS_HELP_STRING([--with-libcap-ng@<:@=auto|yes|no@:>@], [Add Libcap-ng support to smartd [auto]])],
  [], [with_libcap_ng=auto])

use_libcap_ng=no
if test "$with_libcap_ng" != "no"; then
  AC_CHECK_LIB(cap-ng, capng_clear,
    [AC_DEFINE(HAVE_LIBCAP_NG, 1, [Define to 1 if you have the `cap-ng' library (-lcap-ng).]) dnl `vim syntax
     CAPNG_LDADD="-lcap-ng"; use_libcap_ng=yes])

  if test "$use_libcap_ng" = "yes"; then
    AC_CHECK_HEADER(cap-ng.h, [], [AC_MSG_ERROR([libcap-ng libraries found but headers are missing])])
  elif test "$with_libcap_ng" = "yes"; then
    AC_MSG_ERROR([libcap-ng support was requested but the library was not found])
  fi
fi

AC_MSG_CHECKING([whether to use libcap-ng])
AC_SUBST(CAPNG_LDADD)
AC_MSG_RESULT([$use_libcap_ng])

# TODO: Remove when NVMe support is no longer EXPERIMENTAL
AC_ARG_WITH(nvme-devicescan,
  [AS_HELP_STRING([--with-nvme-devicescan@<:@=yes|no@:>@],
    [Include NVMe devices in smartd DEVICESCAN [no]])])
AC_SUBST(with_nvme_devicescan)
if test "$with_nvme_devicescan" = "yes"; then
  AC_DEFINE(WITH_NVME_DEVICESCAN, 1, [Define to 1 to include NVMe devices in smartd DEVICESCAN.])
fi

AC_ARG_WITH(solaris-sparc-ata,
  [AS_HELP_STRING([--with-solaris-sparc-ata@<:@=yes|no@:>@],
    [Enable legacy ATA support on Solaris SPARC (requires os_solaris_ata.s from SVN repository) [no]])])

case "$host:$with_solaris_sparc_ata" in
  sparc-*-solaris*:yes)
    if test ! -f "$srcdir/os_solaris_ata.s"; then
      AC_MSG_ERROR([Missing source file: $srcdir/os_solaris_ata.s
This file is no longer included in the source tarball but still
available in the SVN repository.])
    fi
    AC_DEFINE(WITH_SOLARIS_SPARC_ATA, 1, [Define to 1 to enable legacy ATA support on Solaris SPARC.])
    ;;
esac

# Assume broken snprintf only on Windows with MSVCRT (MinGW without ANSI stdio support)
libc_have_working_snprintf=yes

case "$host" in
  *-*-mingw*)
    case " $CPPFLAGS $CXXFLAGS" in
      *\ -[[DU]]__USE_MINGW_ANSI_STDIO*)
        ;;
      *)
        # Older MinGW do not properly define PRI?64 if __USE_MINGW_ANSI_STDIO is set
        # Newer MinGW set __USE_MINGW_ANSI_STDIO in first C++ include which may be too late
        AC_MSG_CHECKING([whether __USE_MINGW_ANSI_STDIO is defined by C++ includes])
        AC_PREPROC_IFELSE([AC_LANG_SOURCE([[
          #undef __USE_MINGW_ANSI_STDIO
          #include <iostream>
          #ifndef __USE_MINGW_ANSI_STDIO
            #error false
          #endif]])],
        [CXXFLAGS="-D__USE_MINGW_ANSI_STDIO $CXXFLAGS"],
        [libc_have_working_snprintf=no])
        AC_MSG_RESULT([$libc_have_working_snprintf])
        ;;
    esac ;;
esac

AC_ARG_WITH(working-snprintf,
  [AS_HELP_STRING([--with-working-snprintf@<:@=yes|no@:>@],
    [Function snprintf() handles output truncation as specified by C99 [MinGW:guessed,others:yes]])],
  [libc_have_working_snprintf=$withval])

if test "$libc_have_working_snprintf" = "yes"; then
  AC_DEFINE(HAVE_WORKING_SNPRINTF, 1, [Define to 1 if the `snprintf' function is sane.]) dnl `vim syntax
fi

AC_ARG_WITH(mingw-aslr,
  [AS_HELP_STRING([--with-mingw-aslr@<:@=auto|yes|low|no@:>@], [Enable ASLR for MinGW executables [auto]])],
  [], [with_mingw_aslr=auto])

case "$host:${LDFLAGS+set}" in
  *-*-mingw*:) # MinGW defaults: link statically and indicate DEP and TS compatibility
    LDFLAGS="-static -Wl,--nxcompat,--tsaware" ;;
esac

case "$host:$with_mingw_aslr" in
  x86_64-*-mingw*:auto)
    AC_MSG_CHECKING([whether $CXX supports --high-entropy-va])
    save_LDFLAGS=$LDFLAGS
    LDFLAGS="$LDFLAGS -pie -Wl,--dynamicbase,-emainCRTStartup,--high-entropy-va,--image-base,0x140000000"
    # Link libstdc++ to detect MinGW 6.3.0 problems with high --image-base
    AC_LINK_IFELSE([AC_LANG_PROGRAM([[
        #include <string>
        std::string s(42, '.');]])],
      [with_mingw_aslr=yes], [with_mingw_aslr=no])
    LDFLAGS=$save_LDFLAGS
    AC_MSG_RESULT([$with_mingw_aslr])
    test "$with_mingw_aslr" = "yes" || with_mingw_aslr=low
    ;;
esac

case "$host:$with_mingw_aslr" in
  x86_64-*-mingw*:yes)
    LDFLAGS="$LDFLAGS -pie -Wl,--dynamicbase,-emainCRTStartup,--high-entropy-va,--image-base,0x140000000" ;;
  x86_64-*-mingw*:low)
    LDFLAGS="$LDFLAGS -pie -Wl,--dynamicbase,-emainCRTStartup" ;;
  *-*-mingw*:auto|*-*-mingw*:yes|*-*-mingw*:low)
    LDFLAGS="$LDFLAGS -pie -Wl,--dynamicbase,-e_mainCRTStartup" ;;
esac

os_win32_manifest=
case "$host" in
  *-*-mingw*)
    # Newer MinGW may add a default manifest
    AC_MSG_CHECKING([whether $CXX adds an application manifest])
    cc_adds_manifest=no
    AC_LINK_IFELSE([AC_LANG_PROGRAM()], [
        if "$WINDRES" -O rc conftest.exe 2>/dev/null | grep '^1.*RT_MANIFEST' >/dev/null 2>&1; then
          cc_adds_manifest=incomplete
          # Manifest must provide a Win 10 compatibility ID
          if "$WINDRES" -O rc conftest.exe 2>/dev/null | grep '{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}' >/dev/null 2>&1; then
            cc_adds_manifest=yes
          fi
        fi],
      [AC_MSG_ERROR([test compile failed])])
    AC_MSG_RESULT([$cc_adds_manifest])
    test "$cc_adds_manifest" = "yes" || os_win32_manifest='os_win32/default.manifest'
    ;;
esac

AC_ARG_WITH(cxx11-option,
  [AS_HELP_STRING([--with-cxx11-option@<:@=OPTION|auto|no@:>@],
    [Compiler option to enable C++11 support for future versions of smartmontools, 'no' if unsupported [auto]])],
  [], [with_cxx11_option=auto])

check_cxx11_support()
{
  save_CXXFLAGS=$CXXFLAGS
  CXXFLAGS=$1
  AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[
      #if __cplusplus < 201103L
      #error false
      #endif
      // use some C++11 features (and return v * 42 :-)
      auto cxx11(long v) noexcept -> decltype(v) {
        typedef decltype(v) t; t r = v;
        static const t a[] = { -7, -1, 1, 2, 3 };
        static_assert(sizeof(r) == sizeof(a[0]), "fail");
        auto f = [](t x, t y){ return x * y; };
        for (const auto & e : a) r = f(r, e);
        return r;
      }]])],
    [CXXFLAGS=$save_CXXFLAGS; return 0], [CXXFLAGS=$save_CXXFLAGS; return 1])
}

case "$with_cxx11_option" in
  no) ;;
  auto)
    AC_MSG_CHECKING([for $CXX option to accept C++11])
    with_cxx11_option=unknown
    for option in "" "-std=gnu++11" "-std=gnu++0x" "-std=c++11" "-std=c++0x"; do
      if check_cxx11_support "$option"; then with_cxx11_option=$option; break; fi
    done
    AC_MSG_RESULT([${with_cxx11_option:-none needed}])
    ;;
  *)
    AC_MSG_CHECKING([whether $CXX $with_cxx11_option supports C++11])
    res=no; check_cxx11_support "$with_cxx11_option" && res=yes
    AC_MSG_RESULT([$res])
    test "$res" = "yes" || AC_MSG_ERROR([$CXX $with_cxx11_option does support C++11])
    ;;
esac


AC_SUBST(releaseversion,['${PACKAGE}-${VERSION}'])
AC_SUBST(smartmontools_release_date)
AC_SUBST(smartmontools_release_time)

# Set platform-specific modules and symbols
os_libs=
os_dltools='curl wget lynx svn'
os_mailer=mail
os_hostname="'hostname'"
os_dnsdomainname=
os_nisdomainname="'domainname'"
os_darwin=no
os_solaris=no
os_win32=no
os_win32_mingw=no
os_win64=no
os_man_filter=
case "${host}" in
  *-*-linux*)
    os_deps='os_linux.o cciss.o dev_areca.o'
    os_dnsdomainname="'dnsdomainname' 'hostname -d'"
    os_nisdomainname="'nisdomainname' 'hostname -y' 'domainname'"
    os_man_filter=Linux
    ;;
  *-*-freebsd*|*-*-dragonfly*|*-*-kfreebsd*-gnu*)
    os_deps='os_freebsd.o cciss.o dev_areca.o'
    os_libs='-lcam'
    os_dltools='curl wget lynx fetch svn'
    AC_CHECK_LIB(usb, libusb20_dev_get_device_desc)
    os_man_filter=FreeBSD
    ;;
  sparc-*-solaris*)
    os_deps='os_solaris.o'
    test "$with_solaris_sparc_ata" = "yes" \
      && os_deps="$os_deps os_solaris_ata.o"
    os_mailer='mailx'
    os_solaris=yes
    os_man_filter=Solaris
    ;;
  *-pc-solaris*)
    os_deps='os_solaris.o'
    os_mailer='mailx'
    os_solaris=yes
    os_man_filter=Solaris
    ;;
  *-*-netbsd*)
    os_deps='os_netbsd.o'
    os_libs='-lutil'
    os_man_filter=NetBSD
    ;;
  *-*-openbsd*)
    os_deps='os_openbsd.o'
    os_libs='-lutil'
    os_dltools='curl wget lynx ftp svn'
    os_man_filter=OpenBSD
    ;;
  *-*-cygwin*)
    os_deps='os_win32.o dev_areca.o'
    os_mailer='email'
    os_hostname="'hostname' 'echo "'"${HOSTNAME?unset}"'"'"
    os_dnsdomainname="'dnsdomainname' 'hostname -d' 'echo "'"${USERDNSDOMAIN?unset}"'"'"
    os_nisdomainname=
    os_win32=yes
    os_man_filter=Cygwin
    ;;
  x86_64-*-mingw*)
    os_deps='os_win32.o dev_areca.o'
    os_win32=yes
    os_win32_mingw=yes
    os_win64=yes
    os_man_filter=Windows
    ;;
  *-*-mingw*)
    os_deps='os_win32.o dev_areca.o'
    os_win32=yes
    os_win32_mingw=yes
    os_man_filter=Windows
    ;;
  *-*-darwin*)
    os_deps='os_darwin.o'
    os_libs='-framework CoreFoundation -framework IOKit'
    os_darwin=yes
    os_man_filter=Darwin
    ;;
  *-*-nto-qnx*)
    os_deps='os_qnxnto.o'
    ;;
  *-*-os2-*)
    os_deps='os_os2.o'
    ;;
  *)
    os_deps='os_generic.o'
    ;;
esac

# Replace if '--with-os-deps' was specified
test -z "$with_os_deps" || os_deps="$with_os_deps"

# Check if we need adapter to old interface (dev_legacy.cpp)
os_src=`echo "${os_deps}"|sed -n 's,^\([[^ .]]*\)\.o.*$,\1.cpp,p'`
AC_MSG_CHECKING([whether ${os_src} uses new interface])
if grep "smart_interface" "${srcdir}/${os_src}" >/dev/null 2>&1; then
  os_new_interface=yes
else
  os_new_interface=no
  os_deps="${os_deps} dev_legacy.o"
  AC_DEFINE(OLD_INTERFACE, 1, [Define to 1 if os_*.cpp still uses the old interface])
fi
AC_MSG_RESULT([$os_new_interface])

AC_SUBST([os_deps])
AC_SUBST([os_libs])
AC_SUBST([os_dltools])
AC_SUBST([os_mailer])
AC_SUBST([os_hostname])
AC_SUBST([os_dnsdomainname])
AC_SUBST([os_nisdomainname])
AC_SUBST([os_man_filter])
AC_SUBST([os_win32_manifest])

# Create drivedb.h update branch name from version: 5.41[.X] -> RELEASE_5_41_DRIVEDB
DRIVEDB_BRANCH=`echo "$drivedb_version" | sed 's,^\([[0-9]]*\.[[0-9]]*\)\..*$,\1,' \
                | sed -n 's,^\([[0-9]][[0-9]]*\)\.\([[0-9]][[0-9]]*\)$,RELEASE_\1_\2_DRIVEDB,p'`
if test -z "$DRIVEDB_BRANCH"; then
  AC_MSG_ERROR([Unable to create DRIVEDB_BRANCH for version: $drivedb_version])
fi
AC_SUBST([DRIVEDB_BRANCH])

# Enable platform-specific makefile sections
AM_CONDITIONAL(OS_DARWIN, [test "$os_darwin" = "yes"])
AM_CONDITIONAL(OS_SOLARIS, [test "$os_solaris" = "yes"])
AM_CONDITIONAL(OS_WIN32, [test "$os_win32" = "yes"])
AM_CONDITIONAL(OS_WIN32_MINGW, [test "$os_win32_mingw" = "yes"])
AM_CONDITIONAL(OS_WIN32_NSIS, [test -n "$MAKENSIS"])
AM_CONDITIONAL(OS_WIN64, [test "$os_win64" = "yes"])

if test "$GXX" = "yes"; then
  # Add -Wall and -W[extra] if its not already specified
  case " $CXXFLAGS " in
    *\ -Wall\ *) ;;
    *) CXXFLAGS="$CXXFLAGS -Wall" ;;
  esac
  case " $CXXFLAGS " in
    *\ -W\ *|*\ -Wextra\ *) ;;
    *) CXXFLAGS="$CXXFLAGS -W" ;;
  esac
  # Add -Wformat=2 (GCC 3.0) -fstack-protector[-strong] (GCC 4.1[4.9]) if supported
  # and CXXFLAGS was not set in configure cmdline (TODO: -Wformat-signedness)
  case "${ac_test_CXXFLAGS+set}:$ac_test_CXXFLAGS" in
    set:)
      for option in "-Wformat=2" "-fstack-protector-strong" "-fstack-protector"; do
        case " $CXXFLAGS:$option" in *\ -fstack-p*:-fstack-p*) continue ;; esac
        AC_MSG_CHECKING([whether $CXX supports $option])
        save_CXXFLAGS=$CXXFLAGS
        CXXFLAGS="$CXXFLAGS $option"
        AC_COMPILE_IFELSE([AC_LANG_PROGRAM()], [res=yes], [res=no; CXXFLAGS=$save_CXXFLAGS])
        AC_MSG_RESULT([$res])
      done
      ;;
  esac
else
  # We are NOT using gcc, so enable host-specific compiler flags
  case "${host}" in
    sparc*-*-solaris*)
      # Tell the Solaris/SPARC C++ compiler about packed ATA structures
      case " $CXXFLAGS" in
        *\ -xmemalign*) ;;
        *) CXXFLAGS="-xmemalign=1i $CXXFLAGS" ;;
      esac ;;
  esac
  case "${host}" in
    *-*-solaris*)
      # Turn on optimization if user has not explicitly set its value
      case " $CXXFLAGS" in
        *\ -xO*) ;;
        *) CXXFLAGS="-xO2 $CXXFLAGS" ;;
      esac
      # Suppress trivial warnings
      case " $CXXFLAGS" in
        *\ -erroff*) ;;
        *) CXXFLAGS="-erroff=%none,wbadinitl,wbadasgl,badargtypel2w,badargtype2w $CXXFLAGS" ;;
      esac ;;
  esac
fi

AC_DEFINE_UNQUOTED(SMARTMONTOOLS_BUILD_HOST,     "${host}",                       [smartmontools Build Host])

AC_SUBST(CXXFLAGS)

AC_CONFIG_FILES(Makefile)
AC_OUTPUT
AC_PROG_MAKE_SET

# Note: Use `...` here as some shells do not properly parse '$(... case $x in X) ...)'
info=`
  echo "-----------------------------------------------------------------------------"
  echo "${PACKAGE}-${VERSION} configuration:"
  echo "host operating system:  $host"
  echo "C++ compiler:           $CXX"
  echo "C compiler:             $CC"
  echo "preprocessor flags:     $CPPFLAGS"
  echo "C++ compiler flags:     $CXXFLAGS"
  echo "C compiler flags:       $CFLAGS"
  echo "linker flags:           $LDFLAGS"
  echo "OS specific modules:    $os_deps $os_libs $LIBS"

  case "$host_os" in
    mingw*)
      echo "application manifest:   ${os_win32_manifest:-built-in}"
      echo "resource compiler:      $WINDRES"
      echo "message compiler:       $WINDMC"
      echo "NSIS compiler:          $MAKENSIS"
      if test -n "$drivedbdir"; then
        echo "drive database file:    EXEDIR/drivedb.h"
        if test -n "$MAKENSIS"; then
          echo "database update tool:   EXEDIR/update-smart-drivedb.exe"
        fi
      else
        echo "drive database file:    [[disabled]]"
      fi
      if test -n "$savestates"; then
        echo "smartd save files:      \`eval eval eval echo $savestates\`MODEL-SERIAL.TYPE.state"
      fi
      if test -n "$attributelog"; then
        echo "smartd attribute logs:  \`eval eval eval echo $attributelog\`MODEL-SERIAL.TYPE.csv"
      fi
      echo "NVMe DEVICESCAN:        ${with_nvme_devicescan-no}"
      ;;

    *)
      echo "binary install path:    \`eval eval eval echo $sbindir\`"
      echo "man page install path:  \`eval eval eval echo $mandir\`"
      echo "doc file install path:  \`eval eval eval echo $docdir\`"
      echo "examples install path:  \`eval eval eval echo $exampledir\`"
      if test -n "$drivedbdir"; then
        echo "drive database file:    \`eval eval eval echo $drivedbdir\`/drivedb.h"
        if test "$with_update_smart_drivedb" = "yes"; then
          echo "database update script: \`eval eval eval echo $sbindir\`/update-smart-drivedb"
          if test "$drivedb_version" != "$VERSION"; then
            echo "... backported to:      branches/$DRIVEDB_BRANCH"
          fi
          echo "download tools:         \`eval eval eval echo $os_dltools\`"
          if test -n "$gnupg"; then
            echo "GnuPG for verification: \`eval eval eval echo $gnupg\`"
          else
            echo "GnuPG for verification: [[disabled]]"
          fi
        else
          echo "database update script: [[disabled]]"
        fi
      else
        echo "drive database file:    [[disabled]]"
      fi
      echo "local drive database:   \`eval eval eval echo $sysconfdir\`/smart_drivedb.h"
      echo "smartd config file:     \`eval eval eval echo $sysconfdir\`/smartd.conf${smartd_suffix}"
      echo "smartd warning script:  \`eval eval eval echo $smartdscriptdir\`/smartd_warning.sh"
      if test -n "$smartdplugindir"; then
        echo "smartd plugin path:     \`eval eval eval echo $smartdplugindir\`"
      else
        echo "smartd plugin path:     [[disabled]]"
      fi
      if test -n "$scriptpath"; then
        echo "PATH within scripts:    \`eval eval eval echo $scriptpath\`"
      else
        echo "PATH within scripts:    [[inherited]]"
      fi
      if test -n "$initddir"; then
        echo "smartd initd script:    \`eval eval eval echo $initddir\`/${initdfile}"
      elif test -z "$systemdsystemunitdir"; then
        echo "smartd initd script:    [[disabled]]"
      fi
      if test -n "$systemdsystemunitdir"; then
        echo "smartd systemd file:    \`eval eval eval echo $systemdsystemunitdir\`/smartd.service"
        if test -n "$systemdenvfile"; then
          echo "smartd environ file:    \`eval eval eval echo $systemdenvfile\`"
        else
          echo "smartd environ file:    [[disabled]]"
        fi
      fi
      if test -n "$savestates"; then
        echo "smartd save files:      \`eval eval eval echo $savestates\`MODEL-SERIAL.TYPE.state"
      else
        echo "smartd save files:      [[disabled]]"
      fi
      if test -n "$attributelog"; then
        echo "smartd attribute logs:  \`eval eval eval echo $attributelog\`MODEL-SERIAL.TYPE.csv"
      else
        echo "smartd attribute logs:  [[disabled]]"
      fi
      echo "libcap-ng support:      $use_libcap_ng"
      case "$host_os" in
        linux*) echo "SELinux support:        ${with_selinux-no}" ;;
      esac
      case "$host_os" in
        linux*|darwin*|netbsd*|cygwin*)
          echo "NVMe DEVICESCAN:        ${with_nvme_devicescan-no}" ;;
      esac
      ;;
  esac
  echo "-----------------------------------------------------------------------------"
`

AC_MSG_NOTICE([
$info
])

# TODO: Remove when NVMe support is no longer EXPERIMENTAL
case "$host_os:${with_nvme_devicescan+set}" in
  linux*:|darwin*:|netbsd*:|cygwin*:|mingw*:)
    AC_MSG_WARN([
This version of smartmontools provides NVMe support which is still
EXPERIMENTAL.  NVMe devices are not yet included in smartd.conf
'DEVICESCAN' and 'smartctl --scan' unless '-d nvme' is specified.
Use option '--with-nvme-devicescan' to include NVMe devices.
Use option '--without-nvme-devicescan' to suppress this warning.
])
    ;;
esac

# TODO: Remove after smartmontools 6.6
case "$host:${with_solaris_sparc_ata+set}" in
  sparc-*-solaris*:)
    AC_MSG_WARN([
Legacy ATA support is no longer enabled by default on Solaris SPARC.
The required source file 'os_solaris_ata.s' is no longer included in
the source tarball but still available in the SVN repository.
Use option '--with-solaris-sparc-ata' to enable legacy ATA support.
Use option '--without-solaris-sparc-ata' to suppress this warning.
])
    ;;
esac

# TODO: Turn to AC_MSG_ERROR after smartmontools 6.6
case "$with_cxx11_option" in
  unknown)
    AC_MSG_WARN([
This version of smartmontools does not use C++11 features, but future
versions possibly will.
This script was unable to determine a compiler option to enable C++11.
Use option '--with-cxx11-option=OPTION' to specify the compiler option
(it will be checked, but not used in the actual build).
Use option '--without-cxx11-option' to suppress this warning if the
compiler lacks C++11 support.
In both cases, please send info about compiler and platform to
$PACKAGE_BUGREPORT - Thanks!
])
    ;;
esac
