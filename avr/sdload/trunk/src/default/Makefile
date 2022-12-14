###############################################################################
# Makefile for the project pentevo
###############################################################################

## General Flags
PROJECT = core
MCU = atmega128
TARGET = core.elf
CC = avr-gcc

## Options common to compile, link and assembly rules
COMMON = -mmcu=$(MCU)

## Compile options common for all C compilation units.
CFLAGS = $(COMMON)
CFLAGS += -Wall -Wextra -gdwarf-2 -std=gnu99 -Os -DF_CPU=11059200UL -funsigned-char -funsigned-bitfields -fpack-struct -fshort-enums
CFLAGS += -MD -MP -MT $(*F).o -MF dep/$(@F).d

## Assembly specific flags
ASMFLAGS = $(COMMON)
ASMFLAGS += $(CFLAGS)
ASMFLAGS += -x assembler-with-cpp -Wa,-gdwarf2

## Linker flags
LDFLAGS = $(COMMON)
LDFLAGS +=  -Wl,-Map=core.map


## Intel Hex file production flags
HEX_FLASH_FLAGS = -R .eeprom

HEX_EEPROM_FLAGS = -j .eeprom
HEX_EEPROM_FLAGS += --set-section-flags=.eeprom="alloc,load"
HEX_EEPROM_FLAGS += --change-section-lma .eeprom=0 --no-change-warnings


## Objects that must be built in order to link
OBJECTS = zx.o interrupts.o main.o test.o pff.o diskio.o ps2.o spi.o depacker_dirty.o rs232.o rtc.o atx.o joystick.o version.o tape.o kbmap.o

## Objects explicitly added by the user
#LINKONLYOBJECTS = fpga.o
FPGACORE = top.o

## Build
all: $(TARGET) core.hex core.eep core.lss size zxevo_beta.bin
	mv zxevo_beta.bin zxevo_fw.bin

release: $(TARGET) core.hex core.eep core.lss size zxevo_release.bin
	mv zxevo_release.bin zxevo_fw.bin

## Compile
zx.o: ../zx.c ../zx.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

interrupts.o: ../interrupts.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

main.o: ../main.c
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

diskio.o: ../pfs/diskio.c ../pfs/diskio.h ../pfs/pff.h ../pfs/pffconf.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<
	
pff.o: ../pfs/pff.c ../pfs/diskio.h ../pfs/pff.h ../pfs/pffconf.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<
	
test.o: ../pfs/test.c ../pfs/diskio.h ../pfs/pff.h ../pfs/pffconf.h ../pins.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<
	
ps2.o: ../ps2.c ../ps2.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

spi.o: ../spi.c ../spi.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

depacker_dirty.o: ../depacker_dirty.c ../depacker_dirty.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

rs232.o: ../rs232.c ../rs232.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

rtc.o: ../rtc.c ../rtc.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

atx.o: ../atx.c ../atx.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

joystick.o: ../joystick.c ../joystick.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

version.o: ../version.c ../version.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

tape.o: ../tape.c ../tape.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

kbmap.o: ../kbmap.c ../kbmap.h
	$(CC) $(INCLUDES) $(CFLAGS) -c  $<

##Link
$(TARGET): $(OBJECTS)
	 $(CC) $(LDFLAGS) $(OBJECTS) $(LIBDIRS) $(LIBS) -o $(TARGET)

top.rbf: ../../../fpga/current/quartus/top.rbf
	cp ../../../fpga/current/quartus/top.rbf top.rbf

$(FPGACORE): top.rbf
	../../../tools/mhmt/mhmt -maxwin2048 top.rbf top.mlz
	avr-objcopy -I binary -O elf32-avr -B avr:51 --rename-section .data=.progmem.data,contents,alloc,load,readonly,data --redefine-sym _binary_top_mlz_start=fpga --redefine-sym _binary_top_mlz_end=fpga_end top.mlz top.o

%.hex: $(TARGET)
	avr-objcopy -O ihex $(HEX_FLASH_FLAGS)  $< $@

%.eep: $(TARGET)
	avr-objcopy $(HEX_EEPROM_FLAGS) -O ihex $< $@ || exit 0

%.lss: $(TARGET)
	avr-objdump -h -S $< > $@

size: ${TARGET}
	@echo .
	@avr-size -C --mcu=${MCU} ${TARGET}

zxevo_beta.bin: core.hex core.eep
	../../../../../../tools/make_fw/make_fw core.hex core.eep zxevo_beta.bin version.txt

zxevo_release.bin: core.hex core.eep
	../../../tools/make_fw/make_fw core.hex core.eep zxevo_release.bin version.txt o

## Clean target
.PHONY: clean
clean:
	-rm -rf $(OBJECTS) $(FPGACORE) core.* dep/* top.* zxevo_*.bin

## program chip
pgm: core.hex
	pgm.bat core.hex



## Other dependencies
-include $(shell mkdir dep 2>/dev/null) $(wildcard dep/*)
