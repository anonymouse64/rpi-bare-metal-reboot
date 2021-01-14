# Simple Raspberry Pi Bare Metal Reboot Utility

This program is a fork of an example program (https://github.com/bztsrc/raspi3-tutorial/tree/8ce0f75b47ccf233e2271dcf30f03fe7a0a5f56a/08_power) which simply reboots a Raspberry Pi.

It currently only is setup for 64-bit Pi's, but supports Raspberry Pi 4 and 3.

All it does is execute a reboot to a specific partition. It is meant to be used as a "kernel" which U-Boot executes, so that U-Boot based systems, such as Ubuntu Core 20, can have separate boot assets on different partitions.

## Building

Because bare metal programs are hard but SD cards are spacious (mostly), instead of making this a fully-baked kernel that actually supports things like I/O and dynamically choosing which partition to boot from, instead I have opted to just make which partition to boot from configurable at build-time. Thus you need to specify which partition the program should reboot into when compiling by defining `REBOOT_PARTITION` as in:

```
$ make CFLAGS=-DREBOOT_PARTITION=2
rm kernel8.elf *.o >/dev/null 2>/dev/null || true
clang --target=aarch64-elf -DREBOOT_PARTITION=2 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c start.S -o start.o
clang --target=aarch64-elf -DREBOOT_PARTITION=2 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c delays.c -o delays.o
clang --target=aarch64-elf -DREBOOT_PARTITION=2 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c uart.c -o uart.o
clang --target=aarch64-elf -DREBOOT_PARTITION=2 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c power.c -o power.o
clang --target=aarch64-elf -DREBOOT_PARTITION=2 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c main.c -o main.o
clang --target=aarch64-elf -DREBOOT_PARTITION=2 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c mbox.c -o mbox.o
ld.lld -m aarch64elf -nostdlib start.o delays.o uart.o power.o main.o mbox.o -T link.ld -o kernel8.elf
llvm-objcopy -O binary kernel8.elf kernel8.img
```

Also note that the compiled image is only valid for one Raspberry Pi SoC, defaulting to Raspberry Pi 4, so to change to be valid for Raspberry Pi 3 for example, you would define `RASPI_3`:

```
$ make CFLAGS=-DRASPI_3
rm kernel8.elf *.o >/dev/null 2>/dev/null || true
clang --target=aarch64-elf -DRASPI_3 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c start.S -o start.o
clang --target=aarch64-elf -DRASPI_3 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c delays.c -o delays.o
clang --target=aarch64-elf -DRASPI_3 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c uart.c -o uart.o
clang --target=aarch64-elf -DRASPI_3 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c power.c -o power.o
clang --target=aarch64-elf -DRASPI_3 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c main.c -o main.o
clang --target=aarch64-elf -DRASPI_3 -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c mbox.c -o mbox.o
ld.lld -m aarch64elf -nostdlib start.o delays.o uart.o power.o main.o mbox.o -T link.ld -o kernel8.elf
llvm-objcopy -O binary kernel8.elf kernel8.img
```


Finally, note that the provided default Makefile uses clang + llvm to compile, but you could also use gcc by downloading that toolchain and moving the symlink from Makefile -> Makefile.clang to Makefile.gcc like so:

```
$ rm Makefile && ln -s Makefile.gcc Makefile
```

## Usage

If all you want to do is see it do something, you can build the kernel8.img and stick it on the first partition of an SD card, and then boot your Raspberry Pi. You should see a message like:

```
�rebooting to partition 2
```

show up on the serial console.

This means that in order to use this project in an universal gadget snap for an Ubuntu Core image that boots on multiple different Raspberry Pi models, you would need to compile the program multiple times and stage the built binaries iteratively, and then at runtime choose which one to boot. Someone with more time and Makefile skills may be able to upgrade this project to compile all permutations using different compile definitions for different targets, etc.

## License

This project is a fork of https://github.com/bztsrc/raspi3-tutorial, which is licensed as MIT, so this project is also licensed with the MIT license.