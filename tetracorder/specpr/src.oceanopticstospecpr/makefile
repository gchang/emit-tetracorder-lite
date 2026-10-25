
DBG=../debug
# install into $(SP_LOCAL)/bin like the other specpr programs (default $(SP_LOCAL)/bin)
SP_LOCAL ?= /usr/local

OBJ=		\
	$(SP_LIB)/sputil.a


all:		oceanopticstospecpr

install:	oceanopticstospecpr
		cp oceanopticstospecpr $(SP_LOCAL)/bin/oceanopticstospecpr
		size $(SP_LOCAL)/bin/oceanopticstospecpr
		cp oceanopticstospecpr-multiple $(SP_LOCAL)/bin/

oceanopticstospecpr:	oceanopticstospecpr.o $(OBJ)
		$(F77) $(SP_FFLAGS) -o oceanopticstospecpr oceanopticstospecpr.o \
			$(OBJ) $(SP_LIB)/specpr.a $(SP_LDLIBS)

.SUFFIXES:
.SUFFIXES: .o .r

oceanopticstospecpr.o:	oceanopticstospecpr.r
		sspp -r -$(SSPPFLAGS) oceanopticstospecpr.r > oceanopticstospecpr.x
		$(RF) $(SP_RFLAGS) oceanopticstospecpr.x > oceanopticstospecpr.f
		rm oceanopticstospecpr.x
		$(F77) $(SP_FFLAGS) -c  oceanopticstospecpr.f
		rm oceanopticstospecpr.f


