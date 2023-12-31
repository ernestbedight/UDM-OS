
#Untethered Data Manager Operating System

#INCLUDED ABSTRACTION
all:clear clone_directories limine_header compile_kernel limine_build

#SECURE RULES
.PHONY:                     \
compile_kernel              \
run                         \
debug                       \
runusb                      \
push                        \
clear                       \
write                       \
limine_build                \
limine_download             \
limine_header               \




#GENERAL SPECIFICATIONS

IMAGE_NAME                      :=    udmos.iso
USB_MOUNT_POINT                 :=    /dev/sda
BUILD_DIRECTORY                 :=    ./Build
CROSS_COMPILER_DESTINATION      :=    ./cross-compiler


#KERNEL SPECIFICATIONS

KERNEL_NAME                     :=     udmos.elf

ifneq ($(wildcard ./cross-compiler/compiler/bin/x86_64-elf-gcc),)
KERNEL_COMPILER                 :=     ./cross-compiler/compiler/bin/x86_64-elf-gcc
else
KERNEL_COMPILER                 :=     gcc
endif

KERNEL_LINKER                   :=     ld
KERNEL_ASSEMBLER                :=     gcc

# All the flags for linker and compiler

INCLUDE_DIRECTORIES       :=                                                 \
                            -I src/kernel/common/                            \
                            -I src/kernel/sys_lib/                           \
                            -I $(BUILD_DIRECTORY)/limine_header              

KERNEL_C_FLAGS            :=                                                 \
                            -Wall                                            \
                            -Wextra                                          \
                            -Wno-unused-parameter                            \
                            -std=gnu11                                       \
                            -ffreestanding                                   \
                            -fno-stack-protector                             \
                            -fno-stack-check                                 \
                            -fno-lto                                         \
                            -fno-PIE                                         \
                            -fno-PIC                                         \
                            -m64                                             \
                            -march=x86-64                                    \
                            -mabi=sysv                                       \
                            -mno-80387                                       \
                            -mno-mmx                                         \
                            -msoft-float                                     \
                            -mgeneral-regs-only                              \
                            -mno-sse                                         \
                            -mno-red-zone                                    \
                            -mcmodel=kernel                                  \
                            $(INCLUDE_DIRECTORIES)        


KERNEL_ASSEMBLER_FLAGS      :=                                               \
                            $(KERNEL_C_FLAGS)                                \
                            -no-pie                                          
                            
KERNEL_LINKER_FLAGS         :=                                               \
                            -nostdlib                                        \
                            -static                                          \
                            -m elf_x86_64                                    \
                            -z                                               \
                            max-page-size=0x1000                             \
                            -no-pie                                          \
                            -T src/kernel/linker_scripts/linker.ld



#RUN TOOLS
QEMU                        :=     qemu-system-x86_64

QEMU_FLAGS                  :=                                               \
                            -d int,guest_errors,cpu_reset                    \
                            -no-reboot                                       \
                            -no-shutdown                                     \
                            -device                                          \
                            VGA,edid=on,xres=1920,yres=1080                  \
                            -cpu Skylake-Client-v4                           \
                            -machine type=q35                                \
                            -m 2G                
                
#,-sse,-sse2,-sse3,-ssse3,-sse4.1,-sse4.2,-fpu                
#add to -cpu to test without certain features                
                
QEMU_DEBUG_FLAG            :=    -s -S                
                
QEMU_LOCAL_SOURCE          :=    -hda $(BUILD_DIRECTORY)/$(IMAGE_NAME)                
QEMU_USB_SOURCE            :=    -hda $(USB_MOUNT_POINT)
                
run:                
	$(QEMU)                                                                  \
	$(QEMU_FLAGS)                                                            \
	$(QEMU_LOCAL_SOURCE)                                                     \
                
debug:                                                    
	$(QEMU)                                                                  \
	$(QEMU_FLAGS)                                                            \
	$(QEMU_LOCAL_SOURCE)                                                     \
	$(QEMU_DEBUG_FLAG)                                                    
                
runusb:                                                    
	$(QEMU)                                                                  \
	$(QEMU_FLAGS)                                                            \
	$(QEMU_USB_SOURCE)

#GIT HUB UPLOAD
push:clear
	git add .                                                               ;\
	read -p "commit text : " COMMIT                                          \
	&& echo "commit : $${COMMIT}"                                           ;\
	git commit -a -m "$$COMMIT"                                             ;\
	git push -f

#CLEANING TOOLS
clear:
	clear
	@echo -e "\n=>\e[0;31mCLEANING...\e[0m"
	@-mkdir -p ./Build
	@rm -rf $(BUILD_DIRECTORY)/*

#WRITING ONTO A MOUNTING POINT
write:
	dd if=$(BUILD_DIRECTORY)/$(IMAGE_NAME) of=$(USB_MOUNT_POINT) obs=1M oflag=direct status=none
#it might brick your mouse if the usb devuce is not plugged, to fix close qemu windows through shortcuts

#LIMINE SETUP

LIMINE_CONFIGURATION_FILE     := ./src/kernel/limine_configuration/limine.cfg
LIMINE_DIRECTORY              := ./limine

limine: limine_download

limine_build:
	@echo -e "\n=>\e[0;31mBUILDING LIMINE...\e[0m"
	@-mkdir -p $(BUILD_DIRECTORY)/IsoRoot
	@cp                                                  \
	$(BUILD_DIRECTORY)/$(KERNEL_NAME)                    \
	$(LIMINE_CONFIGURATION_FILE)                         \
	$(LIMINE_DIRECTORY)/limine.sys                       \
	$(LIMINE_DIRECTORY)/limine-cd.bin                    \
	$(LIMINE_DIRECTORY)/limine-cd-efi.bin                \
	$(BUILD_DIRECTORY)/IsoRoot            	
	@xorriso                                             \
        -as mkisofs                                          \
        -b limine-cd.bin                                     \
        -no-emul-boot                                        \
        -boot-load-size 4                                    \
        -boot-info-table                                     \
        --efi-boot limine-cd-efi.bin                         \
        -efi-boot-part                                       \
        --efi-boot-image                                     \
        --protective-msdos-label                             \
        $(BUILD_DIRECTORY)/IsoRoot                           \
        -o $(BUILD_DIRECTORY)/$(IMAGE_NAME)

	@$(LIMINE_DIRECTORY)/limine-deploy $(BUILD_DIRECTORY)/$(IMAGE_NAME)


limine_download:
	@echo -e "\n=>\e[0;31mDOWNLOADING LIMINE...\e[0m"
	rm -rf ./limine
	git clone https://github.com/limine-bootloader/limine.git $(LIMINE_DIRECTORY) --branch=v4.x-branch-binary --depth=1 && make -C $(LIMINE_DIRECTORY)

limine_header:
	cp $(LIMINE_DIRECTORY)/limine.h $(BUILD_DIRECTORY)/limine_header/



build_cross_compiler:build_cross_compiler_stubb


#INCLUDING RULES
include ./src/kernel/GNUmakefile
include ./cross-compiler/GNUmakefile


#STUBB FOR FUTURE FEATURES

compile_kernel: build_kernel


#CREATE A CLONE OF THE SOURCE DIRECTORIES TO BUILD IN

clone_directories:
	@echo -e "\n=>\e[0;31mCREATING DIRECTORIES...\e[0m"
	@rsync -av -f"+ */" -f"- *" "./src" "$(BUILD_DIRECTORY)"
	@mkdir -p $(BUILD_DIRECTORY)/limine_header

