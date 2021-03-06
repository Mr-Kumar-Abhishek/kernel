default: build

target = x86_64-unknown-intermezzos-gnu
libcore_rlib_path = target/libcore/target/x86_64-unknown-intermezzos-gnu/release/
libcore_rlib = $(libcore_rlib_path)/libcore.rlib
libcore_sha = 02e41cd5b925a1c878961042ecfb00470c68296b
libintermezzos_a = target/x86_64-unknown-intermezzos-gnu/release/libintermezzos.a
o_files = target/multiboot_header.o target/boot.o
	
build: target/kernel.bin

.PHONY: clean

target/libcore:
	git clone http://github.com/intermezzos/libcore target/libcore
	cd target/libcore && git reset --hard $(libcore_sha)

$(libcore_rlib): target/libcore
	cp $(target).json target/libcore
	cd target/libcore && cargo build --release --features disable_float --target=$(target).json

cargo: $(libcore_rlib)
	RUSTFLAGS="-L $(libcore_rlib_path)" cargo build --release --target $(target).json

target/multiboot_header.o: src/asm/multiboot_header.asm
	mkdir -p target
	nasm -f elf64 src/asm/multiboot_header.asm -o target/multiboot_header.o

target/boot.o: src/asm/boot.asm
	mkdir -p target
	nasm -f elf64 src/asm/boot.asm -o target/boot.o

target/kernel.bin: $(o_files) src/asm/linker.ld cargo
	ld -n -o target/kernel.bin -T src/asm/linker.ld $(o_files) $(libintermezzos_a)

target/os.iso: target/kernel.bin src/asm/grub.cfg
	mkdir -p target/isofiles/boot/grub
	cp src/asm/grub.cfg target/isofiles/boot/grub
	cp target/kernel.bin target/isofiles/boot/
	grub-mkrescue -o target/os.iso target/isofiles

run: target/os.iso
	qemu-system-x86_64 -cdrom target/os.iso

test: build
	cargo test

clean: 
	cargo clean
