GET= get
GFLAGS=-s
# install into $(SP_LOCAL)/bin like the other specpr programs (default $(SP_LOCAL)/bin)
SP_LOCAL ?= /usr/local

OBJ=sspp.o 

sspp:	$(OBJ)
	cc $(CFLAGS) -o sspp $(OBJ)

install:	
	mv -f sspp $(SP_LOCAL)/bin
	chmod 555 $(SP_LOCAL)/bin/sspp

manual:		manual.1
	cp manual.1 /usr/man/cat1/sspp.1

.SUFFIXES:
.SUFFIXES: .o .c
	
sspp.o:		sspp.c
	cc $(CFLAGS) -c sspp.c
