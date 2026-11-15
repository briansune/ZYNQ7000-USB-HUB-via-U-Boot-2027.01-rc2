# Zynq-7000 ARM U-Boot USB Hub setup

## Description

It is found that many old posts reports Zynq-7000 having issue on USB HUB detect.

1) Zynq ARM uses Chipidea ULPI IP
2) ULPI PHY chip could be touchy on time and dts compatibles
3) GPIO reset order timing is very touchy on some system

Fail / SoCFPGA system HUB issue

[1: 2013 ZYNQ U-Boot report](https://community.element14.com/products/devtools/avnetboardscommunity/avnetboard-forums/f/software-application-development/34440/how-to-enable-usb-support-in-u-boot-for-zynq)

[2: Xilinx ULPI Chip](https://adaptivesupport.amd.com/s/article/53450?language=en_US)

[3: SoCFPGA Cyclone](https://community.altera.com/discussions/soc-system/cyclone5-soc-u-boot-not-detecting-usb-hub/349878)

Based on the above background it is very sensitive on System PHY CHIP <-> ULPI IP

For ZYNQ CH132C+CH334P the stable setup:

U-Boot .config
```
CONFIG_USB_ONBOARD_HUB=y
CONFIG_USB_HUB_DEBOUNCE_TIMEOUT=2000
```

DTS
```
	usb_phy0: phy0 {
		#phy-cells = <0>;
		compatible = "usb-nop-xceiv";
		reset-gpios = <&gpio0 46 GPIO_ACTIVE_LOW>;
	};

&usb0 {
	status = "okay";
	dr_mode = "host";
	usb-phy = <&usb_phy0>;

	#address-cells = <1>;
	#size-cells = <0>;

	hub@1 {
		compatible = "CH132C,WCH", "usb424,2514";
		reg = <1>;
		reset-gpios = <&gpio0 48 GPIO_ACTIVE_LOW>;
	};
};
```

LOG
```

Zynq> usb start
starting USB...
USB EHCI 1.00
Bus usb@e0002000: 4 USB Device(s) found
       scanning usb for storage devices... 1 Storage Device(s) found


Zynq> usb tree
USB device tree:
  1  Hub (480 Mb/s, 0mA)
  |  u-boot EHCI Host Controller
  |
  +-2  Hub (480 Mb/s, 100mA)
    |   USB HUB
    |
    +-3   (480 Mb/s, 500mA)
    |    Realtek 802.11n  WLAN Adapter
    |
    +-4  Mass Storage (480 Mb/s, 224mA)
         SanDisk Ultra Fit
```
