ACLOCAL_AMFLAGS = -I m4 -I ${LIBVARNISHAPI_DATAROOTDIR}/aclocal

SUBDIRS = src

DISTCHECK_CONFIGURE_FLAGS = \
	VMOD_DIR=@VMOD_DIR@

EXTRA_DIST = README.rst LICENSE CHANGES.rst

doc_DATA = README.rst LICENSE CHANGES.rst

dist_man_MANS = vmod_saintmode.3
MAINTAINERCLEANFILES = $(dist_man_MANS)

vmod_saintmode.3: README.rst

%.1 %.2 %.3 %.4 %.5 %.6 %.7 %.8 %.9:
if HAVE_RST2MAN
	${RST2MAN} $< $@
else
	@echo "========================================"
	@echo "You need rst2man installed to make dist"
	@echo "========================================"
	@false
endif

