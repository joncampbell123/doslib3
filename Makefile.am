
include $(abs_top_builddir)/Makeflag.tgt

all_targets =

$(BUILDDIR):
	mkdir -p $(BUILDDIR)

include $(abs_top_builddir)/*/Makefile.sub

all: $(BUILDDIR) $(all_targets)

