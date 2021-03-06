# ===========================================================================
# Kernel configuration targets
# These targets are used from top-level makefile

.PHONY: oldconfig xconfig gconfig menuconfig config silentoldconfig

xconfig: $(obj)/qconf
	$< arch/$(ARCH)/Kconfig

gconfig: $(obj)/gconf
	$< arch/$(ARCH)/Kconfig

menuconfig: $(obj)/mconf
	$(Q)$(MAKE) $(build)=scripts/lxdialog
	$< arch/$(ARCH)/Kconfig

config: $(obj)/conf
	$< arch/$(ARCH)/Kconfig

oldconfig: $(obj)/conf
	$< -o arch/$(ARCH)/Kconfig

silentoldconfig: $(obj)/conf
	$< -s arch/$(ARCH)/Kconfig

.PHONY: randconfig allyesconfig allnoconfig allmodconfig defconfig

randconfig: $(obj)/conf
	$< -r arch/$(ARCH)/Kconfig

allyesconfig: $(obj)/conf
	$< -y arch/$(ARCH)/Kconfig

allnoconfig: $(obj)/conf
	$< -n arch/$(ARCH)/Kconfig

allmodconfig: $(obj)/conf
	$< -m arch/$(ARCH)/Kconfig

defconfig: $(obj)/conf
	$< -d arch/$(ARCH)/Kconfig

%_defconfig: $(obj)/conf
	$(Q)$< -D arch/$(ARCH)/configs/$@ arch/$(ARCH)/Kconfig

# Help text used by make help
help:
	@echo  '  oldconfig	  - Update current config utilising a line-oriented program'
	@echo  '  menuconfig	  - Update current config utilising a menu based program'
	@echo  '  xconfig	  - Update current config utilising a QT based front-end'
	@echo  '  gconfig	  - Update current config utilising a GTK based front-end'
	@echo  '  defconfig	  - New config with default answer to all options'
	@echo  '  allmodconfig	  - New config selecting modules when possible'
	@echo  '  allyesconfig	  - New config where all options are accepted with yes'
	@echo  '  allnoconfig	  - New minimal config'

# ===========================================================================
# Shared Makefile for the various kconfig executables:
# conf:	  Used for defconfig, oldconfig and related targets
# mconf:  Used for the mconfig target.
#         Utilizes the lxdialog package
# qconf:  Used for the xconfig target
#         Based on QT which needs to be installed to compile it
# gconf:  Used for the gconfig target
#         Based on GTK which needs to be installed to compile it
# object files used by all kconfig flavours

libkconfig-objs := zconf.tab.o

hostprogs-y	:= conf mconf qconf gconf
conf-objs	:= conf.o  libkconfig.so
mconf-objs	:= mconf.o libkconfig.so

ifeq ($(MAKECMDGOALS),xconfig)
	qconf-target := 1
endif
ifeq ($(MAKECMDGOALS),gconfig)
	gconf-target := 1
endif


ifeq ($(qconf-target),1)
qconf-cxxobjs	:= qconf.o
qconf-objs	:= kconfig_load.o
endif

ifeq ($(gconf-target),1)
gconf-objs	:= gconf.o kconfig_load.o
endif

clean-files	:= libkconfig.so lkc_defs.h qconf.moc .tmp_qtcheck \
		   .tmp_gtkcheck zconf.tab.c zconf.tab.h lex.zconf.c

# generated files seem to need this to find local include files
HOSTCFLAGS_lex.zconf.o	:= -I$(src)
HOSTCFLAGS_zconf.tab.o	:= -I$(src)

HOSTLOADLIBES_qconf	= -L$(QTLIBPATH) -Wl,-rpath,$(QTLIBPATH) -l$(QTLIB) -ldl
HOSTCXXFLAGS_qconf.o	= -I$(QTDIR)/include 

HOSTLOADLIBES_gconf	= `pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --libs`
HOSTCFLAGS_gconf.o	= `pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --cflags`

$(obj)/conf.o $(obj)/mconf.o $(obj)/qconf.o $(obj)/gconf.o: $(obj)/zconf.tab.h

$(obj)/qconf.o: $(obj)/.tmp_qtcheck

ifeq ($(qconf-target),1)
MOC = $(QTDIR)/bin/moc
QTLIBPATH = $(QTDIR)/lib
-include $(obj)/.tmp_qtcheck

# QT needs some extra effort...
$(obj)/.tmp_qtcheck:
	@set -e; for d in $$QTDIR /usr/share/qt* /usr/lib/qt*; do \
	  if [ -f $$d/include/qconfig.h ]; then DIR=$$d; break; fi; \
	done; \
	if [ -z "$$DIR" ]; then \
	  echo "*"; \
	  echo "* Unable to find the QT installation. Please make sure that the"; \
	  echo "* QT development package is correctly installed and the QTDIR"; \
	  echo "* environment variable is set to the correct location."; \
	  echo "*"; \
	  false; \
	fi; \
	LIBPATH=$$DIR/lib; LIB=qt; \
	$(HOSTCXX) -print-multi-os-directory > /dev/null 2>&1 && \
	  LIBPATH=$$DIR/lib/$$($(HOSTCXX) -print-multi-os-directory); \
	if [ -f $$LIBPATH/libqt-mt.so ]; then LIB=qt-mt; fi; \
	echo "QTDIR=$$DIR" > $@; echo "QTLIBPATH=$$LIBPATH" >> $@; \
	echo "QTLIB=$$LIB" >> $@; \
	if [ ! -x $$DIR/bin/moc -a -x /usr/bin/moc ]; then \
	  echo "*"; \
	  echo "* Unable to find $$DIR/bin/moc, using /usr/bin/moc instead."; \
	  echo "*"; \
	  echo "MOC=/usr/bin/moc" >> $@; \
	fi
endif

$(obj)/gconf.o: $(obj)/.tmp_gtkcheck

ifeq ($(gconf-target),1)
-include $(obj)/.tmp_gtkcheck

# GTK needs some extra effort, too...
$(obj)/.tmp_gtkcheck:
	@if `pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --exists`; then		\
		if `pkg-config gtk+-2.0 --atleast-version=2.0.0`; then			\
			touch $@;								\
		else									\
			echo "*"; 							\
			echo "* GTK+ is present but version >= 2.0.0 is required.";	\
			echo "*";							\
			false;								\
		fi									\
	else										\
		echo "*"; 								\
		echo "* Unable to find the GTK+ installation. Please make sure that"; 	\
		echo "* the GTK+ 2.0 development package is correctly installed..."; 	\
		echo "* You need gtk+-2.0, glib-2.0 and libglade-2.0."; 		\
		echo "*"; 								\
		false;									\
	fi
endif

$(obj)/zconf.tab.o: $(obj)/lex.zconf.c

$(obj)/kconfig_load.o: $(obj)/lkc_defs.h

$(obj)/qconf.o: $(obj)/qconf.moc $(obj)/lkc_defs.h

$(obj)/gconf.o: $(obj)/lkc_defs.h

$(obj)/%.moc: $(src)/%.h
	$(MOC) -i $< -o $@

$(obj)/lkc_defs.h: $(src)/lkc_proto.h
	sed < $< > $@ 's/P(\([^,]*\),.*/#define \1 (\*\1_p)/'


###
# The following requires flex/bison
# By default we use the _shipped versions, uncomment the following line if
# you are modifying the flex/bison src.
# LKC_GENPARSER := 1

ifdef LKC_GENPARSER

$(obj)/zconf.tab.c: $(obj)/zconf.y 
$(obj)/zconf.tab.h: $(obj)/zconf.tab.c

%.tab.c: %.y
	bison -t -d -v -b $* -p $(notdir $*) $<

lex.%.c: %.l
	flex -P$(notdir $*) -o$@ $<

endif
