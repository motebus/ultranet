# Setup RPi for boot from LAN

1.  Prepare one SD card.
  ```
  sudo apt-get update
  sudo apt-get full-upgrade
  sudo apt-get install rpi-eeprom
  ```
2.  Configure the Rasperry Pi 4 bootloader to PXE boot <br />
  Next lets examine your boot loader configuration using this command:<br />
  ```
  vcgencmd bootloader_config
  ```
  We need to modify the boot loader config to boot off the network using the BOOT_ORDER parameter. To do that we must extract it from the EEPROM image. Once extracted, make our modifications to enable PXE boot. Finally install it back into the boot loader.<br />
  -  Go to the directory where the bootloader images are stored:<br />
    `cd /lib/firmware/raspberrypi/bootloader/beta/`<br />
  -  Make a copy of the latest firmware image file.<br />
    `sudo cp pieeprom-<latest>.bin new-pieeprom.bin`<br />
  -  Extract the config from the eeprom image<br />
    `sudo rpi-eeprom-config new-pieeprom.bin > ~/bootconf.txt`<br />
  -  In bootconf.txt, change the BOOT_ORDER variable to BOOT_ORDER=0x21. In my case it had defaulted to BOOT_ORDER=0x1. 0X1 means only boot from SD card. 0x21 means attempt SD card boot first, then network boot. See this Raspberry Pi Bootloader page for more details on the values and what they control.<br />
  -  Now save the new bootconf.txt file to the firmware image we copied earlier:<br />
    `sudo rpi-eeprom-config --out netboot-pieeprom.bin --config ~/bootconf.txt new-pieeprom.bin`<br />
  -  Now install the new boot loader:<br />
    `sudo rpi-eeprom-update -d -f ./netboot-pieeprom.bin`<br />
  -  If you get an error with the above command, double check that your apt-get full-upgrade completed successfully.<br />
3.  Disabling automatic rpi-eeprom-update<br />
  As pointed out by a reddit user, rpi-update will update itself by default. The rpi-eeprom-update job does this. Considering that we are using beta features, a firmware update could disable PXE boot in the eeprom. You can disable automatic updates by masking the rpi-eeprom-update via systemctl. You can manually update the eeprom by running rpi-eeprom-update when desired. See the Raspberry Pi docs on rpi-eeprom-update for more<br /> details.
  ```
  sudo systemctl mask rpi-eeprom-update
  ```
4.  Please make note of ethernet interface MAC address.
  ```
  ip addr show eth0
  ```
