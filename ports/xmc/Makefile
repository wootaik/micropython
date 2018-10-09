BUILD ?= build

DEBUG = 1

include ../../py/mkenv.mk
-include mpconfigport.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

CROSS_COMPILE = arm-none-eabi-

INC += -I.
INC += -I$(TOP)
INC += -I$(BUILD)
INC += -I../../lib/mp-readline
INC += -I$(TOP)/ports/stm32
INC += -Ixmclib/inc
INC += -Icmsis/Infineon/XMC4700_series/Include
INC += -Icmsis/Include
INC += -Ilufa
INC += -Ilufa/USB
INC += -Ilufa/VirtualSerial

COPT += 
CFLAGS_CORTEX_M4 = -mthumb -mtune=cortex-m4 -mcpu=cortex-m4 -mfpu=fpv4-sp-d16 -mfloat-abi=hard -fsingle-precision-constant -Wdouble-promotion
CFLAGS = $(INC) -Wall -Wpointer-arith -Werror -std=gnu99 -nostdlib $(CFLAGS_CORTEX_M4) $(COPT) -O0 -std=c99 -Wall -fms-extensions -DUC_ID=4503 -DXMC4700_F144x2048 -DARM_MATH_CM4 -g3 -fmessage-length=0

OBJS  = $(SRC:.c=.o)
OBJS += src/system_XMC4700.o

# Remove uncalled code from the final image.
CFLAGS += -fdata-sections -ffunction-sections
#LIBGCC_FILE_NAME = $(shell $(CC) $(CFLAGS) -print-libgcc-file-name)
LDFLAGS += --gc-sections 
#LDFLAGS += -L$(dir $(LIBGCC_FILE_NAME)) -lgcc
#LDFLAGS += --gc-sections -L usr/lib/gcc/arm-none-eabi/7.3.1/thumb/v7e-m/fpv4-sp/hard/libgcc.a 

#Debugging/Optimization
ifeq ($(DEBUG), 1)
CFLAGS += -O0 -ggdb
COPT += -O0
else
COPT += -Os -DNDEBUG
endif

LDFLAGS += -nostdlib -T xmc4700.ld -Map=$@.map --cref 
LIBS = $(shell $(CC) $(CFLAGS) -print-libgcc-file-name)

SRC_C = \
	main.c \
	system_XMC4700.c \
	uart_core.c \
	uart_extra.c \
	lib/timeutils/timeutils.c \
	lib/utils/printf.c \
	lib/utils/pyexec.c \
	lib/libc/string0.c \
	lib/libm/math.c \
	lib/libm/acoshf.c \
	lib/libm/asinfacosf.c \
	lib/libm/asinhf.c \
	lib/libm/atan2f.c \
	lib/libm/atanf.c \
	lib/libm/atanhf.c \
	lib/libm/ef_rem_pio2.c \
	lib/libm/erf_lgamma.c \
	lib/libm/fmodf.c \
	lib/libm/kf_cos.c \
	lib/libm/kf_tan.c \
	lib/libm/kf_rem_pio2.c \
	lib/libm/kf_sin.c \
	lib/libm/log1pf.c \
	lib/libm/nearbyintf.c \
	lib/libm/sf_frexp.c \
	lib/libm/sf_sin.c \
	lib/libm/sf_cos.c \
	lib/libm/sf_ldexp.c \
	lib/libm/sf_tan.c \
	lib/libm/sf_erf.c \
	lib/libm/sf_modf.c \
	lib/libm/roundf.c \
	lib/libm/wf_tgamma.c \
	lib/libm/wf_lgamma.c \
	lib/libm/ef_sqrt.c \
	lib/mp-readline/readline.c \
	$(BUILD)/_frozen_mpy.c \
	xmclib/src/xmc_common.c \
	xmclib/src/xmc_gpio.c \
	xmclib/src/xmc_uart.c \
	xmclib/src/xmc_usic.c \
	xmclib/src/xmc4_scu.c \
	xmclib/src/xmc4_gpio.c \
	xmclib/src/xmc_usbd.c \
	lufa/USB/Class/Device/CDCClassDevice.c \
	lufa/USB/Common/memcpy.c \
	lufa/USB/Core/ConfigDescriptors.c \
	lufa/USB/Core/Events.c \
	lufa/USB/Core/XMC4000/Endpoint_XMC4000.c \
	lufa/USB/Core/XMC4000/USBController_XMC4000.c \
	lufa/USB/Core/XMC4000/EndpointStream_XMC4000.c \
	lufa/USB/Core/HostStandardReq.c \
	lufa/USB/Core/USBTask.c \
	lufa/VirtualSerial/VirtualSerial.c \
	lufa/VirtualSerial/Descriptors.c \

SRC_S = \
	startup_XMC4700.S \
    gchelper.S 

OBJ = $(PY_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o) $(SRC_S:.S=.o))

all: $(BUILD)/firmware.elf $(BUILD)/firmware.hex

firmware.elf: $(OBJS)
	mkdir -p
	$(CROSS_COMPILE)gcc -T xmc4700.ld $(LFLAGS) $(CFLAGS) -o firmware.elf $(OBJS) $(LIBS)
	$(CROSS_COMPILE)objcopy -O ihex firmware.elf

firmware: firmware.elf
	$(CROSS_COMPILE)objcopy $(CPFLAGS) firmware.elf bin/firmware.bin
	$(CROSS_COMPILE)objdump -S firmware.elf > bin/firmware.lst
	$(CROSS_COMPILE)size firmware.elf

$(BUILD)/_frozen_mpy.c: frozentest.mpy $(BUILD)/genhdr/qstrdefs.generated.h
	$(ECHO) "MISC freezing bytecode"
	$(Q)$(TOP)/tools/mpy-tool.py -f -q $(BUILD)/genhdr/qstrdefs.preprocessed.h $< > $@

$(BUILD)/firmware.elf: $(OBJ)
	$(ECHO) "LINK $@"
	$(Q)$(LD) $(LDFLAGS) -o $@ $^ $(LIBS)
	$(Q)$(SIZE) $@

$(BUILD)/firmware.hex: $(BUILD)/firmware.elf
	$(Q)$(OBJCOPY) -O ihex $< $@

install: $(BUILD)/firmware.elf
	$(CROSS_COMPILE)gdb $(BUILD)/firmware.elf -ex "target remote :2331" -ex "monitor reset" -ex "load" -ex "monitor reset" -ex "monitor go"

debug: $(BUILD)/firmware.elf
	$(CROSS_COMPILE)gdb $(BUILD)/firmware.elf -ex "target remote :2331" -ex "monitor reset" -ex "load" -ex "monitor reset" -ex "monitor go" -ex "layout asm" -ex "monitor reset" -ex "break main" -ex "until main"

include $(TOP)/py/mkrules.mk
