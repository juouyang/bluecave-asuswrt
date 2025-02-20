include ../../common.mak

CFLAGS += -Os -Wall $(EXTRACFLAGS) -fPIC
CFLAGS += -I. -I$(TOP)/shared -I$(TOP)/nvram$(BCMEX)$(EX7) -I$(SRCBASE)/include -I$(TOP)/httpd
CFLAGS += -I$(TOP)/bwdpi_source/src/tdts_udb_sample
CFLAGS += -I$(TOP)/bwdpi_source/src/tdts_udb_sample/linux
CFLAGS += -I$(TOP)/bwdpi_source/include
CFLAGS += -I$(TOP)/bwdpi_source/asus_include

LDFLAGS += -L$(TOP)/nvram$(BCMEX)$(EX7) -lnvram -L$(TOP)/shared -lshared

ifeq ($(RTCONFIG_QTN),y)
LDFLAGS += -L$(TOP)/libqcsapi_client -lqcsapi_client
endif

ifeq ($(HND_ROUTER),y)
CFLAGS += -idirafter$(LINUXDIR)/include
CFLAGS += -I$(HND_SRC)/userspace/public/include -I$(HND_SRC)/userspace/public/include/linux -I$(HND_SRC)/userspace/private/include
LDFLAGS += -L$(TOP)/wlcsm -lwlcsm -L$(TOP)/bcm_flashutil -lbcm_flashutil -L$(TOP)/bcm_util -lbcm_crc -L$(TOP)/bcm_boardctl -lbcm_boardctl
endif

ifeq ($(RTCONFIG_BCMARM),y)
CFLAGS += -I$(SRCBASE)/shared/bcmwifi/include -DTYPEDEF_FLOAT_T
endif

ifeq ($(RTCONFIG_PERMISSION_MANAGEMENT),y)
CFLAGS += -I$(TOP)/sqlCipher
CFLAGS += -I$(TOP)/PMS_DBapis
CFLAGS += -I$(TOP)/openssl/include/openssl
LDFLAGS += -L$(TOP)/openssl -lcrypto -lssl
LDFLAGS += -L $(TOP)/sqlCipher/.libs -lsqlcipher
LDFLAGS += -L$(TOP)/PMS_DBapis -lpms_sql
endif

# BUILD_NAME
ifneq ($(findstring 4G-,$(BUILD_NAME)),)
TARGET_MODEL = RT$(subst -,,$(BUILD_NAME))
else ifneq ($(findstring DSL,$(BUILD_NAME)),)
TARGET_MODEL = $(subst -,_,$(BUILD_NAME))
else
TARGET_MODEL = $(subst -,,$(subst +,P,$(BUILD_NAME)))
endif
CFLAGS  += -D$(TARGET_MODEL)

# for QCA toolchain compile issue, xxxx-ld has no pthead and dl in openwrt-toolchain
ifeq ($(RTCONFIG_QCA),y)
EXTRACC += -lpthread -ldl
EXTRALD =
else
EXTRACC = 
EXTRALD = 
endif

LD1 += $(LDFLAGS) $(EXTRACC)
LD2 += $(LDFLAGS) $(EXTRALD)

# ASUSWRT
OBJS += iqos.o wrs.o stat.o dpi.o wrs_app.o data_collect.o watchdog_check.o
OBJS += dpi_support.o  # dpi_support feature to enhance security
OBJS += wrs_wbl.o      # wrs - white and black list

# TrendMicro
OBJS += conf_app.o ioc_anomaly.o ioc_common.o ioc_qos.o ioc_vp.o ioc_wbl.o ioc_wrs.o ioc_mesh.o
ifneq ($(RTCONFIG_QCA956X),y)
OBJS += ioc_patrol_tq.o
endif

ifeq ($(RTCONFIG_BCMARM),y)
vpath %.c $(SRCBASE)/shared
endif

vpath %.c $(TOP)/bwdpi_source/src/tdts_udb_sample

ifeq ($(wildcard $(SRCBASE)/router/bwdpi_source/asus/*.c),)
all:
	-cp -f prebuild/libbwdpi.so libbwdpi.so
	-cp -f prebuild/hwinfo hwinfo
else
all: hwinfo libbwdpi.so dpi_support.prep
endif

libbwdpi.so: $(OBJS)
	@echo " [bwdpi] LD $@"
	@$(LD) $(LD2) -shared -o $@ $^

hwinfo: hwinfo.o
	@$(CC) -o $@ $^ $(LD1) -L. -lgcc_s

# this command is for debug only
dbg_dpi_support: dbg_dpi_support.o dpi_support.o
	@$(CC) -o $@ $^ $(LD1) -L. -lgcc_s

# this command is for debug only
bwdpi_cmd: main.o $(OBJS)
	@echo " [bwdpi] CC $@"
	@$(CC) -o $@ $^ $(LD1) -L. -lgcc_s
	@$(STRIP) bwdpi_cmd

install: all
	@echo "[bwdpi] Installing..."
	@install -d $(INSTALLDIR)/bwdpi_source/usr/lib
	@install -m 755 libbwdpi.so $(INSTALLDIR)/bwdpi_source/usr/lib
	@$(STRIP) $(INSTALLDIR)/bwdpi_source/usr/lib/libbwdpi.so
	@install -d $(INSTALLDIR)/bwdpi_source/usr/sbin
	@install -m 755 hwinfo $(INSTALLDIR)/bwdpi_source/usr/sbin/hwinfo
	@$(STRIP) $(INSTALLDIR)/bwdpi_source/usr/sbin/hwinfo

%.o: %.c
	@echo " [bwdpi] CC $@"
	@$(CC) $(CFLAGS) -c $<

dpi_support.prep: dpi_support.c
	@echo " [bwdpi] compile prep $@"
	@$(CC) $(CFLAGS) -E $< > $@

clean:
	rm -f *.so *.o *.prep hwinfo bwdpi_cmd dbg_dpi_support
