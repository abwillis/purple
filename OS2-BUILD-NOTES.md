# Pidgin / libpurple 2.14.14 on OS/2 (IPv4-only, no GUI)

These notes cover the complete Pidgin source tree.  `configure` is run from the
top of the tree, not from `libpurple/`.

## Configure and build

```sh
autoreconf -fi
./configure --host=i386-pc-os2-emx --disable-gui --disable-nls \
            --disable-dbus --disable-perl --disable-tcl \
            --disable-gstreamer --disable-vv
make
```

`--disable-gui` is shorthand for `--disable-gtkui --disable-consoleui`; the
`pidgin/` and `finch/` directories are skipped and only libpurple, its
protocols and its plugins are built.  `autoreconf -fi` is required once after
applying the patch because `configure.ac` and several `Makefile.am` files
changed; the `Makefile.in` files and `configure` already in a tree are stale.

Plugin support must stay enabled (do not pass `--disable-plugins`).  SSL/TLS is
provided by the `ssl*` plugins, so XMPP and IRC over TLS depend on it.

## What the OS/2 changes do

### IPv6 removed

- `configure` gains `--enable-ipv6` / `--disable-ipv6`.  The default is
  *disabled on OS/2 hosts* (`host_os` matching `os2*`) and *enabled elsewhere*
  when `struct sockaddr_in6` and `struct in6_addr` exist.  A disabled build
  defines `PURPLE_DISABLE_IPV6` in `config.h`.  `--enable-ipv6` on OS/2 defines
  `PURPLE_ENABLE_IPV6` and keeps the IPv6 code.
- `internal.h` also turns IPv6 off when the compiler defines `__OS2__`,
  `__EMX__` or `OS2`, even if `config.h` is not used, and derives
  `PURPLE_HAS_IPV6` only when `AF_INET6` exists and IPv6 is not disabled.
- The gate covers the shared socket-address union, DNS resolution (`AF_INET`
  only), listening sockets (`AF_UNSPEC` becomes `AF_INET`, other families are
  refused), interface enumeration, address-to-text conversion, proxy and
  NAT-PMP handling, Jabber STUN discovery, and Bonjour.

### DLL linking

An OS/2 DLL cannot contain unresolved symbols.  As on other DLL platforms,
libtool needs `-no-undefined` (and the module has to link against libpurple) to
build a shared module, otherwise the protocol and SSL plugins are not produced
as loadable DLLs.  On OS/2 hosts `configure` therefore sets:

- `PURPLE_DLL_LDFLAGS = -no-undefined`
- `PURPLE_DLL_LIBS = $(top_builddir)/libpurple/libpurple.la`

Every protocol and plugin `Makefile.am` appends them.  Both variables are empty
on every other host, so nothing changes there.

`libpurple/Makefile.am` builds libpurple before the plugins on OS/2
(`ciphers . plugins protocols`).  With `--with-static-prpls=...` the static
protocols are built first because they become part of libpurple
(`protocols ciphers . plugins`).  Mixing static and dynamic protocols is not
supported on OS/2.

### Other configure changes

- `inet_aton` is also searched for in `-lsocket` before configure gives up.

## Applying the patch

The patch is rooted at the top of the tree:

```sh
git apply --check pidgin-2.14.14-os2.patch
git apply pidgin-2.14.14-os2.patch
```

If the tree was exported with tabs replaced by spaces, or the reverse, add
`--ignore-whitespace`.  GNU patch: `patch -p1 -l < pidgin-2.14.14-os2.patch`.

Run `git apply` from the top of the tree.  From a subdirectory it silently
skips every path outside that subdirectory.

## Validation status

Performed on Linux with GCC:

- All 61 libpurple core translation units plus `marshallers.c`, the IRC, Jabber,
  Jabber/Google, Simple, Novell and Null protocols, Bonjour, and the example
  plugins compile with no errors in three modes: normal IPv6, explicit
  `PURPLE_DISABLE_IPV6`, and `__OS2__` / `__EMX__` with automatic detection.
  The one exception is a test artifact: Bonjour's own `INET6_ADDRSTRLEN`
  fallback (`#ifndef INET6_ADDRSTRLEN`) is bypassed by the poisoning below.
  Bonjour is not built on OS/2 without Avahi or the Bonjour SDK.
- In the two IPv4-only modes every IPv6-only identifier (`AF_INET6`,
  `struct sockaddr_in6`, `struct in6_addr`, `INET6_ADDRSTRLEN`, `IN6_IS_*`,
  `IPV6_V6ONLY`) was poisoned, so any unguarded use fails the compile.
- The core was also compiled without `getaddrinfo`, `getifaddrs` and `inet_ntop`
  and without `__unix__` (the blocking resolver path).
- `-Wdeclaration-after-statement` is clean for the files changed.

Not performed: running autoconf, automake or libtool (not installed in the
validation environment), and any build on OS/2 itself.  The `configure.ac` and
`Makefile.am` edits were reviewed by hand only.  The first `autoreconf -fi` on
the OS/2 system is the first real test of them.
