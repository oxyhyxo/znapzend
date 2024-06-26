# Author: Tobi Oetiker <tobi@oetiker.ch>
# License: Public Domain

AC_INIT([znapzend],[m4_esyscmd(tr -d '\n' < VERSION)],[tobi@oetiker.ch])
AC_PREREQ([2.69])
AC_CONFIG_AUX_DIR(conftools)

AC_MSG_CHECKING(in to see how you are doing)
AC_MSG_RESULT(keep fighting man!)

# need this to allow long path names
AM_INIT_AUTOMAKE([1.9 tar-ustar foreign no-dependencies no-installinfo no-texinfo.tex nostdinc])
AM_MAINTAINER_MODE

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_PREFIX_DEFAULT(/opt/$PACKAGE_NAME-$PACKAGE_VERSION)
dnl Fix this early so we can expand with eval later; note literal single quotes in exec_prefix
AS_IF([test x"${prefix}" = x"NONE"], [prefix="${ac_default_prefix}"])
AS_IF([test x"${exec_prefix}" = x"NONE"], [exec_prefix='${prefix}'])

AC_ARG_VAR(PERL,   [Path to local perl binary])
AC_PATH_PROG(PERL, perl, no)
AC_PATH_PROG(CURL, curl, no)
AC_PATH_PROG(WGET, wget, no)
AC_PATH_PROG(POD2MAN, pod2man, no)
AC_PATH_PROG(RM, rm, no)

AC_MSG_CHECKING([for CONFIG_YEAR and CONFIG_DATE])
CONFIG_YEAR="`date +%Y`"
CONFIG_DATE="`date +%Y-%m-%d`"
AC_MSG_RESULT([got ${CONFIG_YEAR} and ${CONFIG_DATE} respectively])
AC_SUBST(CONFIG_YEAR)
AC_SUBST(CONFIG_DATE)

AC_ARG_ENABLE([deb_build],
AS_HELP_STRING([--enable-deb_build    Enable debian package build specifics]),
[deb_build=true],
[deb_build=false])

AM_CONDITIONAL([DEB_BUILD], [test x$deb_build = xtrue])

# TODO: with some respect to legacy, split such settings to be named
# after respective service management framework; code supports several
AC_ARG_ENABLE(svcinstall,
        AS_HELP_STRING([--enable-svcinstall=DIR],[install svc manifest into given directory
        ]),
        [SVCINSTALLDIR=$enableval],
        [SVCINSTALLDIR=no]
)
AC_SUBST(SVCINSTALLDIR)
AM_CONDITIONAL(ENABLE_SVCINSTALL,[test $SVCINSTALLDIR != no])

SVCNAME_SMF="oep/znapzend"
AC_ARG_WITH(svcname-smf,
        AS_HELP_STRING([--with-svcname-smf=SMF_FMRI],[use specified SMF FMRI in manifest (defaults to oep/znapzend)
        ]),
        [AC_MSG_CHECKING([user-provided SMF service name of '$withval'])
         SVCNAME_SMF="`echo "$withval" | sed -e 's,^svc:,,' -e 's,^/*,,' -e 's,:.*$,,'`"
        ],[AC_MSG_CHECKING([default SMF service name])]
)
AC_MSG_RESULT([$SVCNAME_SMF])
AC_SUBST(SVCNAME_SMF)

AC_MSG_CHECKING(for way to cat URLs)
URL_CAT=""
DEFAULT_URL_CAT_CURL="$CURL --location --insecure"
DEFAULT_URL_CAT_WGET="$WGET -O -"

AC_ARG_WITH(url-cat,
    AS_HELP_STRING([--with-url-cat=(curl|wget)],[use specified program (with further predefined arguments added as needed) to cat URLs
    ]),
    [AS_CASE("$withval",
        [curl],[AS_IF([test -n "$CURL" && test -x "$CURL"],
                    [URL_CAT="$DEFAULT_URL_CAT_CURL"],
                    [AC_MSG_ERROR([Asked to preconfigure use of $withval but one is not in PATH])])],
        [wget],[AS_IF([test -n "$WGET" && test -x "$WGET"],
                    [URL_CAT="$DEFAULT_URL_CAT_WGET"],
                    [AC_MSG_ERROR([Asked to preconfigure use of $withval but one is not in PATH])])],
        [*],[AC_MSG_ERROR([Asked to preconfigure use of $withval which is not an URL cat-er we support])])],
    [AC_ARG_WITH(url-cat-program,
        AS_HELP_STRING([--with-url-cat-program=(/path/name --args)],[use specified custom program with arguments to cat URLs
            ]),
        [URL_CAT="$withval"],
        [
        AS_IF([test -x "$WGET"],[URL_CAT="$DEFAULT_URL_CAT_WGET"],
            [AS_IF([test -x "$CURL"],[URL_CAT="$DEFAULT_URL_CAT_CURL"])]
        )])
    ])

AS_IF([test -z "$URL_CAT"],
    [URL_CAT="false : neither curl nor wget found, nor --with-url-cat-program=... specified"])

AC_MSG_RESULT([$URL_CAT])
AC_SUBST(URL_CAT)


# Note: current Mojolicious as of mid-2020 requires even Perl 5.16+
# Older versions needed 5.10.1+
ac_perl_version="5.16.0"

if test "x$PERL" != "x"; then
  AC_MSG_CHECKING(for perl version greater than or equal to $ac_perl_version)
  $PERL -e "use $ac_perl_version;" >/dev/null 2>&1
  if test $? -ne 0; then
    AC_MSG_RESULT(no);
    AC_MSG_WARN(at least version $ac_perl_version is required to run modern mojolicious)
  fi
else
  AC_MSG_ERROR(could not find perl)
fi
AC_SUBST(MOJOLICIOUS_VERSION_CONSTRAINT)
# if test -n "$MOJOLICIOUS_VERSION_CONSTRAINT" ; then
#    AC_MSG_WARN(The available Perl version limits mojolicious version$MOJOLICIOUS_VERSION_CONSTRAINT)
# fi

AC_MSG_CHECKING(is perl reasonably complete)
if $PERL -MExtUtils::MakeMaker -e '' 2>/dev/null; then
  AC_MSG_RESULT(yes. ExtUtils::MakeMaker is available);
else
  AC_MSG_RESULT(no)
  AC_MSG_ERROR([a complete perl ('perl-core' in the redhat world) installation is required])
fi

AC_MSG_CHECKING([if require a c compiler to get perl modules compiled])
if $PERL -MIO::Socket::IP -e 'exit($IO::Socket::IP::VERSION < 0.37)' 2>/dev/null; then
    AC_MSG_RESULT(no)
else
    AC_MSG_RESULT(yes)
    perl_cc=`$PERL -MConfig -e 'print $Config{cc}'`
    AC_PATH_PROG(PERL_CC_PATH, $perl_cc, no)
    AC_MSG_CHECKING(is perls favorite c compiler ($perl_cc) available)
    if test x$PERL_CC_PATH = xno; then
        AC_MSG_RESULT(no)
        AC_MSG_ERROR([perl needs the '$perl_cc' compiler package to build dependencies])
    else
        AC_MSG_RESULT(yes)
    fi
fi

AC_PROG_GREP

dnl We tried to make te Makefile.am syntax portable, but hiccups are possible
AC_ARG_VAR(GMAKE,   [Path to local GNU Make binary])
AC_PATH_PROGS(GMAKE, [gnumake gmake make])

AC_MSG_CHECKING([for gnu make availability])
if  ( $GMAKE --version 2> /dev/null | $GREP GNU  > /dev/null 2>&1 );  then
    AC_MSG_RESULT([$GMAKE is GNU make])
else
    AC_MSG_WARN([GNU make not found, build recipes might fail. Try setting the GMAKE environment variable.])
fi

AC_ARG_ENABLE(pkgonly,
        AS_HELP_STRING([--enable-pkgonly],[Skip all checking]))
AC_SUBST(enable_pkgonly)

# TODO: Is this block needed? No code seems to refer this varname...
actual_prefix=$prefix
if test x$actual_prefix = xNONE; then
    actual_prefix=$ac_default_prefix
fi

AC_MSG_CHECKING(the price for bergulian eckels)
AC_MSG_RESULT(way to expensive!)

AC_ARG_VAR(PERL5LIB,   [Colon separated list of perl library directories])
AC_SUBST(PERL5LIB)

AC_MSG_CHECKING([for PERL_MODULES_EXTRA_DIST])
PERL_MODULES_EXTRA_DIST="`find "${srcdir}/lib/" -name "*.pm" | sed "s,^${srcdir}/,"'$(top_srcdir)/', | tr '\n' ' '`"
AC_MSG_RESULT([${PERL_MODULES_EXTRA_DIST}])
AC_SUBST(PERL_MODULES_EXTRA_DIST)

AC_MSG_CHECKING([for PERL_MODULES])
PERL_MODULES="`cd "${srcdir}/lib/" && ( find . -name "*.pm" | sed 's,^\./,,' | tr '\n' ' ' )`"
AC_MSG_RESULT([${PERL_MODULES}])
AC_SUBST(PERL_MODULES)

AC_MSG_CHECKING([for PERL_CONFIG_VERSION])
PERL_CONFIG_VERSION="`${PERL} -MConfig -e 'my $v = $Config{version}; $v =~ s/\.\d+$//; print $v;'`"
AC_MSG_RESULT([${PERL_CONFIG_VERSION}])
AC_SUBST(PERL_CONFIG_VERSION)

AC_MSG_CHECKING([for PERL_THIRDPARTY_DIST cache])
PERL_THIRDPARTY_DIST="`cd thirdparty && test -d cache && find cache -type f | tr '\n' ' '`"
AS_IF([test x"${PERL_THIRDPARTY_DIST}" = x], [AC_MSG_RESULT([empty])], [AC_MSG_RESULT([populated])])
AC_SUBST(PERL_THIRDPARTY_DIST)

dnl Autotools default expansion of path settings results in further $variables
dnl so for practical use in static configs this yarn has to be fully unrolled:
dnl expand ${prefix} and write it out
conftemp="${prefix}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
PREFIX=${conftemp}
AC_DEFINE_UNQUOTED(PREFIX, "${conftemp}", [Default base path for architecture-independent product files])
AC_SUBST(PREFIX)

dnl same for exec_prefix
conftemp="${exec_prefix}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
EXEC_PREFIX=${conftemp}
AC_DEFINE_UNQUOTED(EXEC_PREFIX, "${conftemp}", [Default base path for architecture-dependent product files])
AC_SUBST(EXEC_PREFIX)

dnl same for datadir
conftemp="${datadir}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
DATADIR=${conftemp}
AC_DEFINE_UNQUOTED(DATADIR, "${conftemp}", [Default path for data files])
AC_SUBST(DATADIR)

dnl same for sysconfdir
conftemp="${sysconfdir}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
SYSCONFDIR=${conftemp}
AC_DEFINE_UNQUOTED(SYSCONFDIR, "${conftemp}", [Default path for configuration files])
AC_SUBST(SYSCONFDIR)

dnl same for bindir
conftemp="${bindir}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
BINDIR=${conftemp}
AC_DEFINE_UNQUOTED(BINDIR, "${conftemp}", [Default path for user executables])
AC_SUBST(BINDIR)

dnl same for sbindir
conftemp="${sbindir}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
SBINDIR=${conftemp}
AC_DEFINE_UNQUOTED(SBINDIR, "${conftemp}", [Default path for system executables])
AC_SUBST(SBINDIR)

dnl same for libdir
conftemp="${libdir}"
eval conftemp=\"${conftemp}\"
eval conftemp=\"${conftemp}\"
LIBDIR=${conftemp}
AC_DEFINE_UNQUOTED(LIBDIR, "${conftemp}", [Default path for system libraries])
AC_SUBST(LIBDIR)

AC_MSG_NOTICE([Generating "data" files from templates, see below for executable scripts])
AC_CONFIG_FILES([
    Makefile
    thirdparty/Makefile
    lib/Makefile
    debian/Makefile
    debian/znapzend.links
    init/znapzend.service
    init/znapzend.xml
    init/org.znapzend.plist
])

AC_MSG_NOTICE([Generating templated script files that should be marked executable])
m4_foreach_w([SCRIPTFILE], [
    init/znapzend.freebsd
    init/znapzend.sysv
    init/znapzend.upstart
], [
    dnl Autoconf substitutes the token above specified in plain text,
    dnl e.g. the brace below is empty and bracket gives verbatim varname
    dnl AC_MSG_NOTICE([Script: SCRIPTFILE brace:(${SCRIPTFILE}) bracket:([SCRIPTFILE])])
    AC_CONFIG_FILES(SCRIPTFILE, chmod +x "SCRIPTFILE")
])

AC_SUBST(VERSION)

AC_OUTPUT

if test x$mod_ok = x0; then
    cat <<NOTES

** SOME PERL MODULES ARE MISSING ******************************

If you know where perl can find the missing modules, set
the PERL5LIB environment variable accordingly. Otherwise
just proceed and the missing modules will be installed
automatically.
NOTES

fi

cat <<NOTES

** CONFIGURE DONE **********************************************

Settings:

  PERL5LIB = ${PERL5LIB:-"not set"}
  PERL = $PERL
  SVCINSTALLDIR = $SVCINSTALLDIR
  URL_CAT = $URL_CAT
  DEB_BUILD = $deb_build

The Makefiles should be portable to the best extent of our testing,
but their recipes should most certainly succeed with GNU make.
It is recommended that now you continue installation with

  ${GMAKE:-make} install

NOTES
