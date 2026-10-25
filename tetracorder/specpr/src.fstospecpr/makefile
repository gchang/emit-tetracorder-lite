# @(#) makefile 2.9@(#)

SHELL=/bin/sh
# install into $(SP_LOCAL)/bin like the other specpr programs (default $(SP_LOCAL)/bin)
SP_LOCAL ?= /usr/local

OBJ= \
specio.o \
io_specpr.o


PROG=fstospecpr

${PROG}:	${OBJ}
	${CC} ${SP_CFLAGS} ${OBJ} -o $@ -lm

clean:
	-rm -f ${OBJ} ${PROG}

install:	${OBJ} ${PROG}
	cp -a ${PROG} $(SP_LOCAL)/bin/${PROG}
	

.SUFFIXES: .o .c
	
specio.o: specio.c
	check_env SP_CFLAGS
	$(CC) $(SP_CFLAGS) -c specio.c

io_specpr.o: io_specpr.c
	check_env SP_CFLAGS
	$(CC) $(SP_CFLAGS) -c io_specpr.c
