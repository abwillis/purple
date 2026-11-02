EXTRA_DIST = \
	Makefile.mingw \
	README

pkgdir = $(libdir)/purple-$(PURPLE_MAJOR_VERSION)

NULLSOURCES = nullprpl.c

AM_CFLAGS = $(st)

libnull_la_LDFLAGS = -module -avoid-version $(PURPLE_DLL_LDFLAGS)

# nullprpl isn't built by default. when it is built, it's dynamically linked.
st =
pkg_LTLIBRARIES    = libnull.la
libnull_la_SOURCES = $(NULLSOURCES)
libnull_la_LIBADD  = $(GLIB_LIBS) $(PURPLE_DLL_LIBS)

AM_CPPFLAGS = \
	-I$(top_srcdir)/libpurple \
	-I$(top_builddir)/libpurple \
	$(GLIB_CFLAGS) \
	$(DEBUG_CFLAGS)
