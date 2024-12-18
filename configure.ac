# git2r, R bindings to the libgit2 library.
# Copyright (C) 2013-2024 The git2r contributors
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License, version 2,
# as published by the Free Software Foundation.
#
# git2r is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License along
# with this program; if not, write to the Free Software Foundation, Inc.,
# 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

AC_PREREQ([2.69])
AC_INIT([git2r], [see.DESCRIPTION.file], [https://github.com/ropensci/git2r/issues])
AC_CONFIG_SRCDIR([src/git2r.c])
AC_CONFIG_AUX_DIR([tools])
AC_CANONICAL_HOST

m4_include([tools/pkg.m4])

## Check for brew on macOS
AC_DEFUN([AC_PROG_BREW], [AC_CHECK_PROG(BREW,brew,yes)])

# Check for pkg-config
PKG_PROG_PKG_CONFIG

# Check for R
: ${R_HOME=`R RHOME`}
if test -z "${R_HOME}"; then
    AC_MSG_FAILURE([Could not determine R_HOME])
fi
RBIN="${R_HOME}/bin/R"

# Library settings
m4_pattern_allow([PKG_CONFIG_NAME])
m4_pattern_allow([PKG_BREW_NAME])
PKG_CONFIG_NAME="libgit2"
PKG_LIBS="-lgit2"
PKG_CFLAGS=""

# The minimum version of libgit2 that is compatible with git2r.
LIBGIT2_MIN_VERSION=1.0

# Check if building against a custom installation of libgit2 or system
# libgit2
if test "x${INCLUDE_DIR}${LIB_DIR}" = x; then
    if test [ -n "$PKG_CONFIG" ] ; then
        # Check if libgit2 is installed and have a version that is
        # compatible with git2r.
        if $PKG_CONFIG ${PKG_CONFIG_NAME} --atleast-version=${LIBGIT2_MIN_VERSION}; then
            PKGCONFIG_CFLAGS=`"${PKG_CONFIG}" --cflags "${PKG_CONFIG_NAME}"`
            PKGCONFIG_LIBS=`"${PKG_CONFIG}" --libs "${PKG_CONFIG_NAME}"`
        fi
    fi

    if test "x${PKGCONFIG_CFLAGS}${PKGCONFIG_LIBS}" = x; then
        case "${host_os}" in
            darwin*)
                AC_PROG_BREW
                if test "x${BREW}" = xyes; then
                    BREWDIR=`brew --prefix`
                    PKG_CFLAGS="-I${BREWDIR}/opt/libgit2/include"
                    PKG_LIBS="-L${BREWDIR}/opt/libgit2/lib ${PKG_LIBS}"
                elif test -z "${DISABLE_AUTOBREW}"; then
                    # Try to download a static build of libgit2 and use that.
                    bottle_ver="1.8.1"
                    bottle_sha256="9419deece3c3b7bec2ed8ed1caa5f4a3f3f1b049a6d17b0a75b2e05128c10fdf"
                    bottle_url="https://github.com/autobrew/bundler/releases/download"
                    bottle="${bottle_url}/libgit2-${bottle_ver}/libgit2-${bottle_ver}-universal.tar.xz"

                    # Download the bottle
                    echo "Download autobrew bundle $(basename ${bottle})"
                    curl -sSL $bottle -o libs.tar.xz

                    # Verify the checksum of the downloaded bottle
                    AC_MSG_CHECKING([SHA256 checksum of downloaded $(basename ${bottle})])
                    : ${bottle_download_sha256=`openssl sha256 libs.tar.xz | sed -e 's:.*= ::'`}
                    if test "x$bottle_download_sha256" != "x$bottle_sha256"; then
                        AC_MSG_FAILURE([The SHA256 checksum of $(basename ${bottle}) didn't match])
                    fi
                    AC_MSG_RESULT([yes])

                    # General setup
                    BREWDIR="$PWD/.deps"
                    mkdir -p $BREWDIR
                    tar -xf libs.tar.xz --strip 1 -C $BREWDIR
                    rm -f libs.tar.xz

                    PKG_CFLAGS="-I${BREWDIR}/include"
                    PKG_LIBS="-L${BREWDIR}/lib ${PKG_LIBS} -lssh2 -lcrypto -lcurl -lz -liconv -framework Security"

                    # Prevent linking against other libs in /usr/local/lib
                    for FILE in $BREWDIR/lib/*.a; do
                        BASENAME=$(basename $FILE)
                        LIBNAME=$(echo "${BASENAME%.*}" | cut -c4-)
                        cp -f $FILE $BREWDIR/lib/libbrew$LIBNAME.a
                        PKG_LIBS=$(echo $PKG_LIBS | sed "s/-l$LIBNAME /-lbrew$LIBNAME /g")
                    done
                fi
            ;;
        esac
    else
        echo "Found pkg-config cflags and libs!"
        PKG_CFLAGS="${PKGCONFIG_CFLAGS}"
        PKG_LIBS="${PKGCONFIG_LIBS}"
    fi
else
    echo "Found INCLUDE_DIR and/or LIB_DIR!"
    PKG_CFLAGS="-I${INCLUDE_DIR} ${PKG_CFLAGS}"
    PKG_LIBS="-L${LIB_DIR} ${PKG_LIBS}"
fi

# Find the compiler and compiler flags to use
CC=`"${RBIN}" CMD config CC`
CFLAGS=`"${RBIN}" CMD config CFLAGS`
CPPFLAGS=`"${RBIN}" CMD config CPPFLAGS`

# check that the version of libgit2 works with git2r.
AC_MSG_CHECKING([whether the libgit2 version will work in git2r])
libgit2_ver_ok=no
${CC} ${CPPFLAGS} ${PKG_CFLAGS} ${CFLAGS} -E tools/version.c >/dev/null 2>&1 && libgit2_ver_ok=yes
AC_MSG_RESULT([${libgit2_ver_ok}])

if test "x${libgit2_ver_ok}" = xno; then
    echo "
  -----------------------------------------------------------------------

   Configuration failed to find libgit2 library. Try installing:
     * brew: libgit2 (MacOS)
     * deb: libgit2-dev   (Debian, Ubuntu, etc)
     * rpm: libgit2-devel (Fedora, CentOS, RHEL)
   and try again.

   If the libgit2 library is installed on your system but the git2r
   configuration is unable to find it, you can specify the include and
   lib path to libgit2 with:

   given you downloaded a tar-gz archive:
   R CMD INSTALL git2r-.tar.gz --configure-vars='INCLUDE_DIR=/path/to/include LIB_DIR=/path/to/lib'

   or cloned the GitHub git2r repository into a directory:
   R CMD INSTALL git2r/ --configure-vars='INCLUDE_DIR=/path/to/include LIB_DIR=/path/to/lib'

   or download and install git2r in R using
   install.packages('git2r', type='source', configure.vars='LIB_DIR=-L/path/to/libs INCLUDE_DIR=-I/path/to/headers')

  -----------------------------------------------------------------------

"

    AC_MSG_FAILURE([package dependency requirement 'libgit2 >= ${LIBGIT2_MIN_VERSION}' could not be satisfied.])
fi

# For debugging
echo "----- Results of the git2r package configure -----"
echo ""
echo "  PKG_CFLAGS: ${PKG_CFLAGS}"
echo "  PKG_LIBS: ${PKG_LIBS}"
echo ""
echo "--------------------------------------------------"

AC_SUBST([PKG_CFLAGS], ["${PKG_CFLAGS}"])
AC_SUBST([PKG_LIBS], ["${PKG_LIBS}"])
AC_CONFIG_FILES([src/Makevars])

AC_OUTPUT
