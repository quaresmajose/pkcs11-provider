#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([pkcs11-provider], [0.1], [simo@redhat.com])
AC_CONFIG_SRCDIR([src/provider.c])
AC_CONFIG_HEADERS([src/config.h])
AM_INIT_AUTOMAKE([foreign dist-xz -Wall -Werror])
AC_CONFIG_MACRO_DIRS([m4])

# Checks for programs.
AC_PROG_CC
AM_PROG_AR
LT_INIT
PKG_PROG_PKG_CONFIG

AX_CHECK_COMPILE_FLAG([-std=c11],
                      [CFLAGS="$CFLAGS -std=c11"],
                      AC_MSG_ERROR([C compiler must support at least C11 standard])
)

STD_CFLAGS="-Wall -Wextra -Wwrite-strings -Wpointer-arith -Wno-missing-field-initializers -Wformat -Wshadow"

# Temporarily disable unused parameter until the implementation is complete
STD_CFLAGS="$STD_CFLAGS -Wno-unused-parameter"

# These should be always errors
STD_CFLAGS="$STD_CFLAGS -Werror=implicit-function-declaration -Werror=missing-prototypes -Werror=format-security -Werror=parentheses -Werror=implicit -Werror=strict-prototypes"

# Don't enable warnings on VLA yet, but I would avoid using VLAs
# STD_CFLAGS"$STD_CFLAGS -Werror=vla"

AX_CHECK_COMPILE_FLAG([-fno-strict-aliasing],
		      [STD_CFLAGS="$STD_CFLAGS -fno-strict-aliasing"])
AX_CHECK_COMPILE_FLAG([-Werror -fno-delete-null-pointer-checks],
		      [STD_CFLAGS="$STD_CFLAGS -fno-delete-null-pointer-checks"])
AX_CHECK_COMPILE_FLAG([-fdiagnostics-show-option],
		      [STD_CFLAGS="$STD_CFLAGS -fdiagnostics-show-option"])

AC_SUBST([STD_CFLAGS])

AC_ARG_WITH([openssl],
  [AS_HELP_STRING([--with-openssl],[the path to the OpenSSL files])],
  [
      OPENSSL_DIR=${withval}
      case "$OPENSSL_DIR" in
        # Relative paths
        ./*|../*)  OPENSSL_DIR="`pwd`/$OPENSSL_DIR"
      esac
      if test -d "$OPENSSL_DIR/lib"; then
        libcrypto_path="$OPENSSL_DIR/lib"
      elif test -d "$OPENSSL_DIR/lib64"; then
        libcrypto_path="$OPENSSL_DIR/lib64"
      else
        # Built but not installed
        libcrypto_path="$OPENSSL_DIR"
      fi
      CFLAGS="-I$OPENSSL_DIR/include $CFLAGS"
      LDFLAGS="-L${libcrypto_path} ${LDFLAGS}"
      LIBS="$LIBS -L${libcrypto_path}"
      AC_MSG_WARN([Custom openssl located in $OPENSSL_DIR is used, lib version check is skipped])
  ],
  [
      # Checks for libraries.
      PKG_CHECK_MODULES(
        [OPENSSL],
        [libcrypto >= 3.0.5, libssl],
        ,
        [AC_MSG_ERROR([libcrypto >= 3.0.5 is required])]
      )
  ])

AC_SUBST([SHARED_EXT], $(eval echo "${shrext_cmds}"))

# Check whether we have a p11-kit to use as a default PKCS#11 module
PKG_CHECK_EXISTS([p11-kit-1],
                 [PKG_CHECK_VAR([DEFAULT_PKCS11_MODULE],
                                [p11-kit-1],
                                [proxy_module])],
                 [AC_MSG_WARN([The P11-kit proxy is not available. No fallback PKCS11 module used.])])
if test "x$DEFAULT_PKCS11_MODULE" != "x"; then
    AC_DEFINE_UNQUOTED([DEFAULT_PKCS11_MODULE], "$DEFAULT_PKCS11_MODULE",
              [Default PKCS11 module])
fi


# Try nss-softoken first as used on Fedora,
# fallback to "nss" as used on Debian
PKG_CHECK_EXISTS(
        [nss-softokn],
        [PKG_CHECK_VAR([SOFTOKENDIR], [nss-softokn], [libdir])],
        [PKG_CHECK_EXISTS(
                [nss],
                [PKG_CHECK_VAR([SOFTOKENDIR], [nss], [libdir])]
        )]
)

AC_CHECK_FILE($SOFTOKENDIR/libsoftokn3$SHARED_EXT, [],
    [AC_CHECK_FILE($SOFTOKENDIR/nss/libsoftokn3$SHARED_EXT,
        [AC_SUBST([SOFTOKEN_SUBDIR], "nss/")],
        [AC_MSG_WARN([Softoken library missing, tests will fail!])])
    ])

# find p11-kit-client to separate softhsm openssl context from our tests
PKG_CHECK_EXISTS([p11-kit-1],
                 [PKG_CHECK_VAR([P11_MODULE_PATH],
                                [p11-kit-1],
                                [p11_module_path])],
                 [AC_MSG_WARN([The p11-kit client not found. Can not run SoftHSM tests])])
if test "$P11_MODULE_PATH" != "" ; then
	# p11-kit-client is a module, so its name ends with .so also on macOS!
    AC_SUBST([P11KITCLIENTPATH], "$P11_MODULE_PATH/p11-kit-client.so")
fi

# Checks for header files.
AC_CHECK_HEADERS([string.h dlfcn.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_CHECK_HEADER_STDBOOL
AC_C_BIGENDIAN
AC_C_INLINE
AC_TYPE_SIZE_T

# Checks for library functions.
AC_CHECK_FUNCS([strpbrk])

AC_CONFIG_FILES([
	Makefile
	src/Makefile
	tests/Makefile
])
AC_OUTPUT
