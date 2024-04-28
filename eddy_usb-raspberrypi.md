This is for Eddy V1 USB ONLY


> [!WARNING]  
> [KAMP aka Klipper-Adaptive-Meshing-Purging](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) should be removed from your klipper prior to using Eddy. Complete removal including any added BED_MESH_CALIBRATION macros should be removed or sufficiently altered (outside the scope of this guide) if you want eddy to RAPID scan bed meshing. 
>
> Instead KAMP has been integrated into klipper as of January 2024 and you should use the ADAPTIVE=1 option in your BED_MESH_CALIBRATION calls. You can find more [Information on Adaptive Mesh Here](https://www.klipper3d.org/Bed_Mesh.html#adaptive-meshes)

> [!WARNING]
> As it stands, Eddy requires the use of BTT's fork of klipper found [HERE](https://github.com/bigtreetech/klipper). This is included in the guide under steps 13.
> 
> This will be merged into mainline klipper at some stage and the guide will be updated once it happens. Until then this is a STRICT REQUIREMENT.

> [!IMPORTANT]
>I placed my eddy configuration from step 9 into its own file eddy.cfg, however this caused an unseen problem later when live adjusting z offset. See [NOTES](https://github.com/krautech/vyper-klipper/blob/main/eddy_usb-raspberrypi.md#notes)
> 
>If not using seperate .cfg files and only using printer.cfg you can disregard live z save offset issues.

## Please read [NOTES](https://github.com/krautech/vyper-klipper/blob/main/eddy_usb-raspberrypi.md#notes)

# Installation of EDDY USB V1 and Raspberry Pi
## Compiling Firmware
1. SSH into raspberry PI
2. Type
```
cd ~/klipper
make menuconfig
```
3. Use these settings to compile the firmware.
![Firmware Image](https://github.com/krautech/vyper-klipper/blob/main/images/eddy-pi/compile.png?raw=true)
4. Once set, hit 'Q' and when asked, select yes to save.
5. Type ```make``` to compile.
6. Disconnect power to Eddy
7. Push and hold boot button on Eddy (Its next to where the cable plugs in) and at the same time, plug in the cable to your Raspberry Pi
![Boot Image](https://github.com/krautech/vyper-klipper/blob/main/images/eddy-pi/boot.png?raw=true)
8. SSH into raspberry Pi
9. Type ```lsusb``` into the command line. You should see eddy. 

![LSUSB Image](https://github.com/krautech/vyper-klipper/blob/main/images/eddy-pi/lsusb.png?raw=true)

10. Type  ```cd ~/klipper``` into command line
11. Type ```make flash FLASH_DEVICE=2e8a:0003```
Remember to change 2e8a:0003 to your device ID you found in step 9
12. Type  ```ls /dev/serial/by-id/*```  into the command line. The found device will be what you enter into your klipper config under [mcu eddy] for the Serial variable.
> [!NOTE]
> You need to change from the main branch of klipper to BTTs branch as discussed in the warning at the top of the page. This is only temporary and will be updated accordingly.
> 
> Still accurate as of **28-04-2024**.
13. Change to BTT klipper by entering the following via SSH
```
git remote add eddy https://github.com/bigtreetech/klipper
git fetch eddy
git checkout eddy/eddy
```
14. Type into command line ```sudo reboot```

## Printer Configuration

15. Add the following to your printer.cfg making sure to adjust for your bed size and probe position
> [!IMPORTANT]
> Adjust your **x_offset** and **y_offset** to match your probe position relative to your nozzle. You can do that following these steps found [HERE](https://www.klipper3d.org/Probe_Calibrate.html)

```
[mcu eddy]
serial: /dev/serial/by-id/usb-Klipper_rp2040_4550357129142D58-if00

[temperature_sensor btt_eddy_mcu]
sensor_type: temperature_mcu
sensor_mcu: eddy
min_temp: 10
max_temp: 100

[probe_eddy_current btt_eddy]
sensor_type: ldc1612
z_offset: 1.0
#i2c_address:
i2c_mcu: eddy
i2c_bus: i2c0f
x_offset: 0 # Set according to the actual offset relative to the nozzle
y_offset: 20 # Set according to the actual offset relative to the nozzle
data_rate: 500

[temperature_probe btt_eddy]
sensor_type: Generic 3950
sensor_pin: eddy:gpio26
horizontal_move_z: 2

[bed_mesh]
horizontal_move_z: 2
speed: 300
mesh_min: 10, 10
mesh_max: 220, 220
probe_count: 9, 9
algorithm: bicubic


[safe_z_home]
home_xy_position: 125, 125
z_hop: 10
z_hop_speed: 25
speed: 200
```
## 2. Live Current Calibration
16. Place Eddy Approx. 20mm above the bed.
17. From Mainsail or Fluidd run command  ```LDC_CALIBRATE_DRIVE_CURRENT CHIP=btt_eddy```
18. Type ```SAVE_CONFIG``` to save the drive currant to your config
## 3. Z Offset Calibration
> [!IMPORTANT]
> If using a printer with Quick Gantry Leveling (Voron etc) perform it now to ensure the gantry is level and to prevent the nozzle rubbing into the bed.
 
19. Home X and Y axes with command ```G28 X Y```
20. Make sure you dont have a bed heightmap loaded.
21. Move Nozzle to Centre of the bed with ```G0 X125 Y125 F6000``` (adjust for your bed size)
22. Start Manual Z-Offset Calibration by typing ```PROBE_EDDY_CURRENT_CALIBRATE CHIP=btt_eddy ```
> [!IMPORTANT]
> Perform another Quick Gantry Leveling (Voron etc)
23. Once completed use ```SAVE_CONFIG```
## 4. Bed Mesh Calibration
24. Home All Axes
25. Use command ```BED_MESH_CALIBRATE METHOD=scan SCAN_MODE=rapid```
26. Once completed use ```SAVE_CONFIG```
## 5. Temperature Compensation Calibration (Eddy USB ONLY)
> [!CAUTION]
> The following steps (27-36) are for Eddy USB Only. Eddy Coil doesnt have temperature compensation so these steps should be disregarded.

27. Home All Axes and move Z 10 above bed
28. Set idle timeout ```SET_IDLE_TIMEOUT TIMEOUT=36000```
29. Record ambient temp of the BTT Eddy Sensor
![Eddy Temperature](https://github.com/krautech/vyper-klipper/blob/main/images/eddy-pi/eddy-temp.jpg?raw=true)
30. Set max temp for bed (i.e 100c) and set typical temperature for hotend (200c)
31. Wait for BTT Eddy temp to stabilize then record temp.
32. Return to room temp by turning off bed and hotend
> [!TIP]
> If you have a high range to test between ambient and max eddy temp from step 25, you can change the value of STEP=3 to STEP=5 to save you some time. Ideally you want as many calibration points as possible for the best use of eddy but I found for range between 30c-50c a STEP value of 3 was sufficient

33. Run ```PROBE_DRIFT_CALIBRATE PROBE=btt_eddy TARGET=50 STEP=3```  (target should be the temp you recorded of the max recorded temp from step 31.
34. Using [the paper method](https://www.klipper3d.org/Bed_Level.html#the-paper-test) adjust your Z offset.
35. Turn on your heat bed and nozzle to same values as step 30
36. As Eddy temp rises at each 3c (STEP=3) increment, you will automatically be asked to set the z-offset when prompted using the paper test.
> [!NOTE]
> By default the calibration procedure will request a manual probe every 2C between samples until the TARGET is reached. The temperature delta between samples can be customized by setting the STEP parameter in PROBE_DRIFT_CALIBRATE.
>
> The following additional gcode commands are available during drift calibration.
>
>    ```PROBE_DRIFT_NEXT``` may be used to force a new sample before the step delta has been reached.
>
>    ```PROBE_DRIFT_COMPLETE``` may be used to complete calibration before the TARGET has been reached.
>
>    ```ABORT``` may be used to end calibration and discard results.
>

> [!TIP]
> You might not reach your target temperature, thats okay. You can end the test by using command ```PROBE_DRIFT_COMPLETE``` to finish.

37. Youre all done! :)

Make sure you LIVE ADJUST your z-offset with your first print to really home it in.

# Extras & Notes

- Klipper seems to only be able to save Z_Offset to printer.cfg so having ```z_offset: 1``` under ```[probe_eddy_current btt_eddy]``` results in unable to ```SAVE_CONFIG``` if its in its own .cfg file. I have moved these 2 sections into seperate files. This allows klipper to be able to save the z offset automatically.

Printer.cfg

```
[probe_eddy_current btt_eddy]
z_offset: 1.0
```

eddy.cfg

```
[probe_eddy_current btt_eddy]
sensor_type: ldc1612
#z_offset: 1.0 ## THIS IS SET IN PRINTER.CFG INSTEAD
#i2c_address:
i2c_mcu: eddy
i2c_bus: i2c0f
x_offset: 0 # Set according to the actual offset relative to the nozzle
y_offset: 20 # Set according to the actual offset relative to the nozzle
data_rate: 500
```

### START PRINT Macro

- Add the following to your start print macro to enable adaptive bed mesh using Eddy
```
BED_MESH_CALIBRATE SCAN_MODE=rapid METHOD=scan ADAPTIVE=1
```

# FAQ - Frequently Asked Questions

### Eddy is performing Z Hops when running Bed Mesh
- Make sure you are using the correct macro call.
```BED_MESH_CALIBRATE SCAN_MODE=rapid METHOD=scan```
- Remove or alter KAMP - Adaptive Bed Mesh and any custom BED_MESH_CALIBRATE macros. Use klipper adaptive mesh instead. 
[Information on Adaptive Mesh Here](https://www.klipper3d.org/Bed_Mesh.html#adaptive-meshes)

### Which Eddy version should I use?
- It depends on your needs. Eddy USB and Eddy Coil are nearly identical, however Eddy Coil is more for toolhead boards and connects via I2C connectors.
- Eddy Coil cannot be used for z-homing as a z-endstop as it doesnt feature temperature compensation.



