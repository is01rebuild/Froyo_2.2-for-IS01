UNUBI_MTD:=mtd2.bin
FROYO_DIR:=is01rebuild2_2_r1_3_20110119_01
WRITE_PARTITION:=recovery_wr
#
MKBOOTIMG:=$(FROYO_DIR)/tools_linux/mkbootimg
MKBOOTFS:=$(FROYO_DIR)/tools_linux/mkbootfs
UNUBINIZE:=./unubinize.pl
SPLIT_BOOTIMG:=./split_bootimg.pl
##############
KERNEL:=$(UNUBI_MTD).out-kernel
EXECDIR:=/sqlite_journals/is01root
SU:=$(EXECDIR)/_su
BUSYBOX:=$(EXECDIR)/busybox

##############
TARGET:=froyo_boot.img
FILES:=./ramdisk/init.rc
BOOTIMG:=my_mkboot.img
RAMDISKDIR:=ramdisk
RAMDISK:=my_ramdisk.img
UBICFG:=ubi.cfg
CMDLINE:="console=ttyMSM2,115200n8 androidboot.hardware=qcom"
##############

.PHONY:all
all:
	@echo is01rebuild froyo installer. Web site for more information.
#pull patch ubinize copy push_froyo copy_froyo ch push_flash_image 
#push_bootimage
#reboot


#ubi
.PHONY:ubinize
ubinize:$(TARGET)

$(TARGET): $(BOOTIMG) $(UBICFG)
	ubinize -o $(TARGET) -p 128KiB -m 2048 -O 256 $(UBICFG)

#mkboot
$(BOOTIMG):$(RAMDISK) $(KERNEL)
	$(MKBOOTIMG) --kernel $(KERNEL) --ramdisk $(RAMDISK) --cmdline $(CMDLINE) --base 0x20000000 -o $@

$(RAMDISK):$(FILES)
	rm -rf ramdisk/*~
	$(MKBOOTFS) $(RAMDISKDIR) > $(RAMDISK)

$(FILES):

.PHONY:copy_froyo
copy_froyo:
	adb shell $(SU) 0 $(BUSYBOX) cp -fa /data/backup0/system /data/system_ex


.PHONY:ch
ch:
	adb shell $(SU) 0 chmod 550        /data/system_ex/system/etc/dhcpcd/dhcpcd-run-hooks
	adb shell $(SU) 0 chown dhcp.shell /data/system_ex/system/etc/dhcpcd/dhcpcd-run-hooks
	adb shell $(SU) 0 chmod 755        /data/system_ex/system/etc/dhcpcd/dhcpcd-hooks
	adb shell $(SU) 0 chown root.root  /data/system_ex/system/etc/dhcpcd/dhcpcd-hooks
	adb shell $(SU) 0 chmod 644        /data/system_ex/system/etc/dhcpcd/dhcpcd.conf
	adb shell $(SU) 0 chown root.root  /data/system_ex/system/etc/dhcpcd/dhcpcd.conf



.PHONY:push_froyo
push_froyo:
	adb push $(FROYO_DIR)/system /data/backup0/system/
	adb shell $(SU) 0 $(BUSYBOX) chown -R 0.0 /data/backup0/system

.PHONY:copy
copy:
	adb shell $(SU) 0 mkdir /data/data_ex
	adb shell $(SU) 0 mkdir /data/data_ex/data
	adb shell $(SU) 0 $(BUSYBOX) cp -a /data/misc /data/data_ex/data/
	adb shell $(SU) 0 $(BUSYBOX) sed -i  '1a ctrl_interface=DIR=/data/misc/wifi/sockets'  /data/data_ex/data/misc/wifi/wpa_supplicant.conf
	adb shell $(SU) 0 mkdir /data/system_ex
	adb shell $(SU) 0 mkdir /data/system_ex/system
	adb shell $(SU) 0 $(BUSYBOX) cp -a /system/lib /data/system_ex/system/
	adb shell $(SU) 0 $(BUSYBOX) cp -a /system/bin /data/system_ex/system/
	adb shell $(SU) 0 $(BUSYBOX) cp -a /system/etc /data/system_ex/system/
	adb shell $(SU) 0 $(BUSYBOX) cp -a /system/usr /data/system_ex/system/


.PHONY:pull
pull:pull_clean
	adb shell mkdir /data/backup0
	adb shell chmod 777 /data/backup0
	adb shell $(SU) 0 dd if=/dev/mtd/mtd0 of=/data/backup0/mtd0.bin
	adb shell $(SU) 0 dd if=/dev/mtd/mtd2 of=/data/backup0/mtd2.bin
	adb shell $(SU) 0 chmod 444 /data/backup0/mtd0.bin
	adb shell $(SU) 0 chmod 444 /data/backup0/mtd2.bin
	adb pull /data/backup0/$(UNUBI_MTD)
	cp -v $(UNUBI_MTD) $(basename $(UNUBI_MTD))_`md5sum $(UNUBI_MTD) | awk '{print $$1}'`.bin
	adb shell $(SU) 0 $(BUSYBOX)  cp /data/backup0/$(UNUBI_MTD) "/data/backup0/$(basename $(UNUBI_MTD))_\`$(BUSYBOX) md5sum /data/backup0/$(UNUBI_MTD) | $(BUSYBOX) awk '{print \$$1}'\`.bin"

.PHONY:patch
patch:
	$(UNUBINIZE) $(UNUBI_MTD)
	$(SPLIT_BOOTIMG) $(UNUBI_MTD).out
	mkdir -p ramdisk
	( cd ramdisk; \
	cat ../$(UNUBI_MTD).out-ramdisk.cpio | cpio -i; \
	mv init.rc init.rc.bak; \
	cat init.rc.bak | grep -v '#' > init.rc; \
	chmod 750 init.rc; \
	patch init.rc < ../init.rc.patch; \
	mv system system0; \
	mv initlogo.rle initlogo.rle.bak; )
	cp -v $(FROYO_DIR)/ramdisk/init ramdisk/init
	cp -v $(FROYO_DIR)/ramdisk/default.prop ramdisk/default.prop
	cp -v $(FROYO_DIR)/ramdisk/sbin/adbd ramdisk/adbd

.PHONY:push_flash_image
push_flash_image:
	adb push flash_image /sqlite_journals/is01root

.PHONY:push_bootimage
push_bootimage:ubinize
	adb push $(TARGET) /data/backup0/
	adb shell $(SU) 0 $(EXECDIR)/flash_image $(WRITE_PARTITION) /data/backup0/$(TARGET)

.PHONY:reboot
reboot:
	adb shell $(SU) 0 reboot recovery


.PHONY:wpa
wpa:
	adb push wpa_supplicant.conf /data/misc/wifi/
	adb shell /system/bin/chown wifi.wifi /data/misc/wifi/wpa_supplicant.conf
	adb shell /system/bin/chmod 660 /data/misc/wifi/wpa_supplicant.conf

.PHONY:push_froyo_clean
push_froyo_clean:
	adb shell $(SU) 0 $(BUSYBOX) rm -rf /data/backup0/system 

.clean:target_froyo_clean
target_froyo_clean:
	adb shell $(SU) 0 $(BUSYBOX) rm -rf /data/data_ex/data
	adb shell $(SU) 0 $(BUSYBOX) rm -rf /data/system_ex/system
	@echo
	@echo
	@echo All froyo files of the target was deleted. 
	@echo !!!! Please change recovery partition image !!!!!
	@echo
	@echo


.PHONY:pull_clean
pull_clean:
	rm -f $(UNUBI_MTD) $(UNUBI_MTD).out $(UNUBI_MTD).out-ramdisk.cpio $(UNUBI_MTD).out-kernel
	rm -rf ramdisk

.PHONY:local_clean
local_clean:
	rm -f $(RAMDISK)
	rm -f $(BOOTIMG)
	rm -f $(TARGET)
	rm -rf *~

.PHONY:clean
clean: local_clean pull_clean push_froyo_clean target_froyo_clean 
