
Skip to content
Navigation Menu

    krautech
    /
    btt-eddy-guide

Code
Issues 1
Pull requests
Actions
Projects
Wiki
Security
Insights

    Settings

Files
t

config
images
video
.gitignore
README.md

    eddy_usb-raspberrypi.md

Editing README.md in btt-eddy-guide
Breadcrumbs

    btt-eddy-guide

/
in
main

Indent mode
Indent size
Line wrap mode
Editing README.md file contents
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
# Installation of EDDY USB V1 and Raspberry Pi


> [!WARNING]  
> [KAMP aka Klipper-Adaptive-Meshing-Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) should be removed from your klipper prior to using Eddy. Please comment out the include line. ie ```#[include ./KAMP/adaptive_meshing.cfg]``` from your KAMP_SETTINGS.cfg
>
> Instead KAMP has been integrated into klipper as of January 2024 and you should use the ADAPTIVE=1 option in your BED_MESH_CALIBRATION calls. You can find more [Information on Adaptive Mesh Here](https://www.klipper3d.org/Bed_Mesh.html#adaptive-meshes)

> [!WARNING]
> As it stands, Eddy requires the use of BTT's fork of klipper found [HERE](https://github.com/bigtreetech/klipper). This is included in the guide under steps 13.
> 
> This will be merged into mainline klipper at some stage and the guide will be updated once it happens. Until then this is a STRICT REQUIREMENT.

## Please read [NOTES](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#notes)

# Index
- [Compiling Firmware](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#compiling-firmware)
- [Printer Configuration](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#printer-configuration)
- - [Z Endstop](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#z-endstop)
- [Live Current Calibration](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#2-live-current-calibration)
- [Z-Offset Calibration](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#3-z-offset-calibration)
- [Bed Mesh Calibration](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#4-bed-mesh-calibration)
- [Temperature Compensation Calibration](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#5-temperature-compensation-calibration-eddy-usb-only)
>
- [Bed Mesh Calibration Parameters](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#bed-mesh-calibrate-parameters)
- [Bed Mesh Scan Height](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#bed-mesh-scan-height)
- [Bed Mesh Rapid Scanning](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#rapid-continuous-scanning)
- [Extras & Notes](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#extras--notes)
- - Includes Print Start Macro Adjustment
- [FAQ - Frequently Asked Questions](https://github.com/krautech/btt-eddy-guide/blob/main/eddy_usb-raspberrypi.md#faq---frequently-asked-questions)

## Compiling Firmware
1. SSH into raspberry PI
2. Type
```
cd ~/klipper
make menuconfig
```
3. Use these settings to compile the firmware.
![Firmware Image](https://github.com/krautech/btt-eddy-guide/blob/main/images/eddy-pi/compile.png?raw=true)
4. Once set, hit 'Q' and when asked, select yes to save.
5. Type ```make``` to compile.
6. Disconnect power to Eddy
7. Push and hold boot button on Eddy (Its next to where the cable plugs in) and at the same time, plug in the cable to your Raspberry Pi
![Boot Image](https://github.com/krautech/btt-eddy-guide/blob/main/images/eddy-pi/boot.png?raw=true)
8. SSH into raspberry Pi
9. Type ```lsusb``` into the command line. You should see eddy. 

![LSUSB Image](https://github.com/krautech/btt-eddy-guide/blob/main/images/eddy-pi/lsusb.png?raw=true)

10. Type  ```cd ~/klipper``` into command line
11. Type ```make flash FLASH_DEVICE=2e8a:0003```
Remember to change 2e8a:0003 to your device ID you found in step 9
12. Type  ```ls /dev/serial/by-id/*```  into the command line. The found device will be what you enter into your klipper config under [mcu eddy] for the Serial variable.
> [!NOTE]
> You need to change from the main branch of klipper to BTTs branch as discussed in the warning at the top of the page. This is only temporary and will be updated accordingly.
> 
> Still accurate as of **13-05-2024**.
13. Change to BTT klipper by entering the following via SSH
```
git remote add eddy https://github.com/bigtreetech/klipper
git fetch eddy
git checkout eddy/eddy
```
14. Type into command line ```sudo reboot```

## Printer Configuration

Use Control + Shift + m to toggle the tab key moving focus. Alternatively, use esc then tab to move to the next interactive element on the page.
Attach files by dragging & dropping, selecting or pasting them.
Editing btt-eddy-guide/README.md at main · krautech/btt-eddy-guide
