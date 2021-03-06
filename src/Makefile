# Version of echoprint, as a list. Is expanded out
# for various version numbers.
EP_VERSION := 4 1 1
VERSION := $(word 1, $(EP_VERSION)).$(word 2, $(EP_VERSION)).$(word 3, $(EP_VERSION))
VERSION_MAJ := $(word 1, $(EP_VERSION))
VERSION_COMPAT := $(word 1, $(EP_VERSION)).$(word 2, $(EP_VERSION))
UNAME := $(shell uname -s)
CXX=g++
CC=gcc
#OPTFLAGS=-g -O0
OPTFLAGS=-O3 -DBOOST_UBLAS_NDEBUG -DNDEBUG
BOOST_CFLAGS=-I/usr/local/include/boost-1_35
TAGLIB_CFLAGS=`taglib-config --cflags`
TAGLIB_LIBS=`taglib-config --libs`
CXXFLAGS=-Wall $(BOOST_CFLAGS) $(TAGLIB_CFLAGS) -fPIC $(OPTFLAGS)
CFLAGS=-Wall -fPIC $(OPTFLAGS)
LDFLAGS=$(TAGLIB_LIBS) -lz -lpthread $(OPTFLAGS)
ifeq ($(UNAME),Darwin)
LIBNAME=libcodegen.$(VERSION).dylib
else
LIBNAME=libcodegen.so
endif
SONAME=$(LIBNAME).$(VERSION_MAJ)
EXENAME=../echoprint-codegen

MODULES_LIB = \
    AudioBufferInput.o \
    AudioStreamInput.o \
    Base64.o \
    Codegen.o \
    Fingerprint.o \
    MatrixUtility.o \
    SubbandAnalysis.o \
    Whitening.o
MODULES = $(MODULES_LIB) Metadata.o

all: $(LIBNAME) $(EXENAME)

$(LIBNAME): $(MODULES_LIB)
ifeq ($(UNAME),Darwin)
	libtool -dynamic -flat_namespace -install_name libcodegen.$(VERSION).dylib -lSystem -compatibility_version $(VERSION_COMPAT) \
		-macosx_version_min 10.6 -current_version $(VERSION) -o $(LIBNAME) -undefined suppress \
	    $(MODULES_LIB) -framework Accelerate
else
	$(CXX) -shared -fPIC -Wl,-soname,$(SONAME) -o $(LIBNAME).$(VERSION) $(MODULES_LIB) -lz
endif

$(EXENAME): $(MODULES) main.o
	$(CXX) $(MODULES) main.o $(LDFLAGS) -o $(EXENAME)

%.o: %.c %.h
	$(CC) $(CFLAGS) -c -o $@ $<

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

%.o: %.cxx %.h
	$(CXX) $(CXXFLAGS) -c -o $@ $<

%.o: %.cxx
	$(CXX) $(CXXFLAGS) -c -o $@ $<

clean:
	rm -f *.o $(EXENAME)
	rm -f libcodegen.so*
ifeq ($(UNAME),Darwin)
	rm -f *.dylib
endif

PREFIX ?= /usr/local
DESTDIR ?= $(PREFIX)
BINDIR ?= /bin
LIBDIR ?= /lib
INCLUDEDIR ?= /include

install: all
	mkdir -p $(DESTDIR)$(BINDIR)
	install $(EXENAME) $(DESTDIR)$(BINDIR)
	install -d $(DESTDIR)$(INCLUDEDIR)/echoprint
	install -pm 644 Codegen.h $(DESTDIR)$(INCLUDEDIR)/echoprint/
	mkdir -p $(DESTDIR)$(LIBDIR)
ifeq ($(UNAME),Darwin)
	install -m 755 libcodegen.$(VERSION).dylib $(DESTDIR)$(LIBDIR)
	cd $(DESTDIR)$(LIBDIR) && ln -fs libcodegen.$(VERSION).dylib libcodegen.$(VERSION_MAJ).dylib
	cd $(DESTDIR)$(LIBDIR) && ln -fs libcodegen.$(VERSION_MAJ).dylib libcodegen.dylib
else
	install -m 755 $(LIBNAME).$(VERSION) $(DESTDIR)$(LIBDIR)
	cd $(DESTDIR)$(LIBDIR) && ln -fs $(LIBNAME).$(VERSION) $(SONAME)
	cd $(DESTDIR)$(LIBDIR) && ln -fs $(SONAME) $(LIBNAME)
endif

.PHONY: clean all install
