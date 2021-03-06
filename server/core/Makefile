# This file is distributed as part of the SkySQL Gateway.  It is free
# software: you can redistribute it and/or modify it under the terms of the
# GNU General Public License as published by the Free Software Foundation,
# version 2.
#
# This program is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
# FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more
# details.
#
# You should have received a copy of the GNU General Public License along with
# this program; if not, write to the Free Software Foundation, Inc., 51
# Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
#
# Copyright SkySQL Ab 2013
#
# Revision History
# Date		Who			Description
# 13/06/13	Mark Riddoch		Addition of -rdynamic to allow libraries
# 					to resolve symbols in the main executable
# 17/06/13	Mark Riddoch		Addition of dependency generation
# 24/06/13	Massimiliano Pinto	Addition of libmysqlclient and its 
#                                       includes do this for launching the 
#                                       gateway: export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH:/packages/mariadb-5.5.25/libmysql					
# 27/06/13      Vilho Raatikka          Added logmanager-related libs and 
#                                       headers so that liblog_manager.so can 
#                                       be linked in.
# 28/06/13      Vilho Raatikka          Added query classifier-related libs and
#                                       commented out mysql client headers and 
#                                       lib to avoid conflicts. MARIADB_SRC_PATH
#                                       is set in build_gateway.inc. Examples
#                                       are behind SS_DEBUG macros.
# 29/06/13       Vilho Raatikka         Reverted Query classifier changes because
#                                       gateway needs mysql client lib, not qc.
# 24/07/13	Mark Ridoch		Addition of encryption routines

include ../../build_gateway.inc

LOGPATH := $(ROOT_PATH)/log_manager
UTILSPATH := $(ROOT_PATH)/utils

CC=cc

CFLAGS=-c -I/usr/include -I../include -I../inih \
	$(MYSQL_HEADERS) \
	-I$(LOGPATH) -I$(UTILSPATH) \
	-Wall -g

include ../../makefile.inc

LDFLAGS=-rdynamic -L$(LOGPATH) \
	-Wl,-rpath,$(DEST)/lib \
	-Wl,-rpath,$(LOGPATH) -Wl,-rpath,$(UTILSPATH) \
	-Wl,-rpath,$(EMBEDDED_LIB)

SRCS= atomic.c buffer.c spinlock.c gateway.c \
	gw_utils.c utils.c dcb.c load_utils.c session.c service.c server.c \
	poll.c config.c users.c hashtable.c dbusers.c thread.c gwbitmask.c \
	monitor.c adminusers.c secrets.c

HDRS= ../include/atomic.h ../include/buffer.h ../include/dcb.h \
	../include/gw.h ../include/mysql_protocol.h \
	../include/session.h ../include/spinlock.h ../include/thread.h \
	../include/modules.h ../include/poll.h ../include/config.h \
	../include/users.h ../include/hashtable.h ../include/gwbitmask.h \
	../include/adminusers.h

OBJ=$(SRCS:.c=.o)

KOBJS=maxkeys.o secrets.o utils.o
POBJS=maxpasswd.o secrets.o utils.o

LIBS=-L../inih/extra -linih -lssl -lstdc++ \
	-L$(EMBEDDED_LIB) \
	-lz -lm -lcrypt -lcrypto -ldl -laio -lrt -pthread -llog_manager \
	-lmysqld

all:	maxscale maxkeys maxpasswd

maxscale: $(OBJ)
	$(CC) $(LDFLAGS) $(OBJ) $(UTILSPATH)/skygw_utils.o $(LIBS) -o $@

maxkeys: $(KOBJS)
	$(CC) $(LDFLAGS) $(KOBJS) $(UTILSPATH)/skygw_utils.o $(LIBS) -o $@

maxpasswd: $(POBJS)
	$(CC) $(LDFLAGS) $(POBJS) $(UTILSPATH)/skygw_utils.o $(LIBS) -o $@

.c.o:
	$(CC) $(CFLAGS) $< -o $@

clean:
	rm -f $(OBJ) maxscale
	- rm *.so

tags:
	ctags $(SRCS) $(HDRS)

depend:	
	@rm -f depend.mk
	cc -M $(CFLAGS) $(SRCS) > depend.mk

install: maxscale maxkeys maxpasswd
	@mkdir -p $(DEST)/bin
	install -D maxscale maxkeys maxpasswd  $(DEST)/bin
	install -D $(EMBEDDED_LIB)/$(LIB) $(DEST)/lib
	install -D $(ERRMSG)/errmsg.sys $(DEST)/MaxScale/mysql

include depend.mk
