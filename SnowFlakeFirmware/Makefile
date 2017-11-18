ifeq ($(CMSIS),)
	CMSIS = $(error You must specify the CMSIS location in the CMSIS environment variable)
endif

ifeq ($(SAMD20),)
	SAMD20 = $(error You must specify the SAMD20 DFP location in the SAMD20 environment variable)
endif

CROSS_COMPILE=arm-none-eabi-

CC=$(CROSS_COMPILE)gcc
CXX=$(CROSS_COMPILE)g++
OBJCOPY=$(CROSS_COMPILE)objcopy
OBJDUMP=$(CROSS_COMPILE)objdump
SIZE=$(CROSS_COMPILE)size
LD=$(CROSS_COMPILE)g++
GDB=$(CROSS_COMPILE)gdb

INC += -I $(CMSIS)/Include -I $(SAMD20)/include

CFLAGS += -mthumb -D__SAMD20E17__ $(INC)
CFLAGS += -ffunction-sections -Wall -mcpu=cortex-m0plus -c -std=gnu99

CXXFLAGS += -mthumb -D__SAMD20E17__ $(INC)
CXXFLAGS += -ffunction-sections -funsafe-math-optimizations
CXXFLAGS += -ffast-math -fno-rtti -fno-exceptions -Wall -mcpu=cortex-m0plus -c

LDSCRIPT = Device_Startup/samd20e17_flash.ld

LDFLAGS += -mthumb -Wl,-Map=main.map --specs=nano.specs --specs=nosys.specs
LDFLAGS += -Wl,--start-group -lm -Wl,--end-group -L Device_Startup -Wl,--gc-sections
LDFLAGS += -funsafe-math-optimizations -ffast-math -mcpu=cortex-m0plus -T $(LDSCRIPT)

SRCS = $(wildcard src/*.cpp) $(wildcard src/scene/*.cpp) $(wildcard Device_Startup/*.c)
OBJS = $(addsuffix .o, $(basename $(SRCS)))

ifeq ($(DEBUG),)
	OBJCOPY_STRIP = --strip-unneeded
	CFLAGS += -DNDEBUG -Os
	CXXFLAGS += -DNDEBUG -O3
else
	OBJCOPY_STRIP =
	CFLAGS += -DDEBUG -g -F dwarf -Og
	CXXFLAGS += -DDEBUG -g -F dwarf -Og
endif

ifeq ($(BMP_PORT),)
	BMP_PORT_CANDIDATES := $(wildcard \
		/dev/serial/by-id/usb-Black_Sphere_Technologies_Black_Magic_Probe_*-if00 \
		/dev/cu.usbmodem*1)
	ifeq ($(words $(BMP_PORT_CANDIDATES)),1)
		BMP_PORT := $(BMP_PORT_CANDIDATES)
	else
		BMP_PORT = $(error Black Magic Probe gdb serial port not found, please provide the device name via the BMP_PORT variable parameter$( if \
			$(BMP_PORT_CANDIDATES), (found $(BMP_PORT_CANDIDATES))))
	endif
endif

all: main.hex main.srec size

main.elf: $(OBJS) $(LDSCRIPT)
	$(LD) $(LDFLAGS) $(LIBS) -o $@ $(OBJS)

%.bin: %.elf
	$(OBJCOPY) $(OBJCOPY_STRIP) -O binary $< $@

%.hex: %.elf
	$(OBJCOPY) -O ihex -R .eeprom -R .fuse -R .lock -R .signature $< $@

%.srec: %.elf
	$(OBJCOPY) -O srec -R .eeprom -R .fuse -R .lock -R .signature $< $@

size: main.elf
	$(SIZE) $<

JLINKSPEED = 15000

flash: main.bin segger.script
	JLinkExe -if swd -device ATSAMD20E17 -speed $(JLINKSPEED) -CommanderScript segger.script

flash-bmp: main.elf
	$(GDB) --batch \
		-ex 'target extended-remote $(BMP_PORT)' \
		-ex 'set confirm off' \
		-ex 'monitor version' \
		-ex 'monitor tpwr enable' \
		-ex 'shell sleep 0.1' \
		-ex 'monitor swdp_scan' \
		-ex 'attach 1' \
		-ex 'load' \
		-ex 'compare-sections' \
		-ex 'kill' \
		$<

headers: main.elf
	$(OBJDUMP) -Ch $^

clean:
	rm -f {,src/,src/scene/,Device_Startup/}*.{o,d} main.{bin,elf,hex,map,srec}

list: main.elf
	$(OBJDUMP) -C -S $^

.PHONY: all flash flash-bmp headers clean list
