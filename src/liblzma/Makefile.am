##
## Author: Lasse Collin
##
## This file has been put into the public domain.
## You can do whatever you want with this file.
##

SUBDIRS = api

EXTRA_DIST =
CLEANFILES =
doc_DATA =

lib_LTLIBRARIES = libflzma.la
libflzma_la_SOURCES =
libflzma_la_CPPFLAGS = \
	-I$(top_srcdir)/src/liblzma/api \
	-I$(top_srcdir)/src/liblzma/common \
	-I$(top_srcdir)/src/liblzma/check \
	-I$(top_srcdir)/src/liblzma/lz \
	-I$(top_srcdir)/src/liblzma/rangecoder \
	-I$(top_srcdir)/src/liblzma/lzma \
	-I$(top_srcdir)/src/liblzma/radix \
	-I$(top_srcdir)/src/liblzma/delta \
	-I$(top_srcdir)/src/liblzma/simple \
	-I$(top_srcdir)/src/common \
	-DTUKLIB_SYMBOL_PREFIX=lzma_
libflzma_la_LDFLAGS = -no-undefined -version-info 0:0:0

EXTRA_DIST += liblzma.map validate_map.sh
if COND_SYMVERS
libflzma_la_LDFLAGS += \
	-Wl,--version-script=$(top_srcdir)/src/liblzma/liblzma.map
endif

libflzma_la_SOURCES += ../common/tuklib_physmem.c

if COND_THREADS
libflzma_la_SOURCES += ../common/tuklib_cpucores.c
endif

include $(srcdir)/common/Makefile.inc
include $(srcdir)/check/Makefile.inc

if COND_FILTER_LZ
include $(srcdir)/lz/Makefile.inc
endif

if COND_FILTER_LZMA1
include $(srcdir)/lzma/Makefile.inc
include $(srcdir)/rangecoder/Makefile.inc
endif

if COND_FILTER_LZMA2
include $(srcdir)/radix/Makefile.inc
endif

if COND_FILTER_DELTA
include $(srcdir)/delta/Makefile.inc
endif

if COND_FILTER_SIMPLE
include $(srcdir)/simple/Makefile.inc
endif


## Windows-specific stuff

# Windows resource compiler support. libtool knows what to do with .rc
# files, but Automake (<= 1.11 at least) doesn't know.
#
# We want the resource file only in shared libflzma. To avoid linking it into
# static libflzma, we overwrite the static object file with an object file
# compiled from empty input. Note that GNU-specific features are OK here,
# because on Windows we are compiled with the GNU toolchain.
.rc.lo:
	$(LIBTOOL) --mode=compile $(RC) $(DEFS) $(DEFAULT_INCLUDES) \
		$(INCLUDES) $(libflzma_la_CPPFLAGS) $(CPPFLAGS) $(RCFLAGS) \
		-i $< -o $@
	echo > empty.c
	$(COMPILE) -c empty.c -o $(*D)/$(*F).o

# Remove ordinals from the generated .def file. People must link by name,
# not by ordinal, because no one is going to track the ordinal numbers.
libflzma.def: libflzma.la libflzma.def.in
	sed 's/ \+@ *[0-9]\+//' libflzma.def.in > libflzma.def

# Creating libflzma.def.in is a side effect of linking the library.
libflzma.def.in: libflzma.la

if COND_W32
CLEANFILES += libflzma.def libflzma.def.in empty.c
libflzma_la_SOURCES += liblzma_w32res.rc
libflzma_la_LDFLAGS += -Xlinker --output-def -Xlinker libflzma.def.in

## libflzma.def.in is created only when building shared libflzma, so don't
## try to create libflzma.def when not building shared libflzma.
if COND_SHARED
doc_DATA += libflzma.def
endif
endif


## pkg-config
pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = libflzma.pc
EXTRA_DIST += liblzma.pc.in

pc_verbose = $(pc_verbose_@AM_V@)
pc_verbose_ = $(pc_verbose_@AM_DEFAULT_V@)
pc_verbose_0 = @echo "  PC      " $@;

libflzma.pc: $(srcdir)/liblzma.pc.in
	$(AM_V_at)rm -f $@
	$(pc_verbose)sed \
		-e 's,@prefix[@],$(prefix),g' \
		-e 's,@exec_prefix[@],$(exec_prefix),g' \
		-e 's,@libdir[@],$(libdir),g' \
		-e 's,@includedir[@],$(includedir),g' \
		-e 's,@PACKAGE_URL[@],$(PACKAGE_URL),g' \
		-e 's,@PACKAGE_VERSION[@],$(PACKAGE_VERSION),g' \
		-e 's,@PTHREAD_CFLAGS[@],$(PTHREAD_CFLAGS),g' \
		-e 's,@LIBS[@],$(LIBS),g' \
		< $(srcdir)/liblzma.pc.in > $@ || { rm -f $@; exit 1; }

clean-local:
	rm -f libflzma.pc
