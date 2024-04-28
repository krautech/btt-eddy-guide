This is for Eddy V1 USB ONLY


## Please read [NOTES](https://github.com/krautech/vyper-klipper/blob/main/eddy_usb-raspberrypi.md#notes)

> [!IMPORTANT]
>I placed my eddy configuration from step 9 into its own file eddy.cfg, however this caused an unseen problem later when live adjusting z offset. See [NOTES](https://github.com/krautech/vyper-klipper/blob/main/eddy_usb-raspberrypi.md#notes)
> 
>If not using seperate .cfg files and only using printer.cfg you can disregard live z save offset issues.

## Installation of EDDY USB V1 and Raspberry Pi

1. Push and hold boot button on Eddy (Its next to where the cable plugs in) and at the same time, plug in the cable to your Raspberry Pi
2. SSH into raspberry PI
3. Type ```lsusb``` into the command line. You should see eddy. 

![LSUSB Image](https://github.com/krautech/vyper-klipper/blob/main/images/eddy-pi/lsusb.png?raw=true)

4. Type  ```cd ~/klipper``` into command line
5. Type ```make flash FLASH_DEVICE=2e8a:0003```
Remember to change 2e8a:0003 to your device ID you found in step 3
6. Type  ```ls /dev/serial/by-id/*```  into the command line. The found device will be what you enter into your klipper config under [mcu eddy] for the Serial variable.

7. Type into command line 
```
git remote add eddy https://github.com/bigtreetech/klipper
git fetch eddy
git checkout eddy/eddy
```
8. Type into command line ```sudo reboot```


9. Add the following to your printer.cfg making sure to adjust for your bed size and probe position

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
## Live Current Calibration
10. Place Eddy Approx. 20mm above the bed.
11. From Mainsail or Fluidd run command  ```LDC_CALIBRATE_DRIVE_CURRENT CHIP=btt_eddy```
12. Type ```SAVE_CONFIG``` to save the drive currant to your config
## Z Offset Calibration
13. Home X and Y axes with command ```G28 X Y```
14. Make sure you dont have a bed heightmap loaded.
15. Move Nozzle to Centre of the bed with ```G0 X125 Y125 F6000``` (adjust for your bed size)
16. Start Manual Z-Offset Calibration by typing ```PROBE_EDDY_CURRENT_CALIBRATE CHIP=btt_eddy ```
17. Once completed use ```SAVE_CONFIG```
## Bed Mesh Calibration
18. Home All Axes
19. Use command ```BED_MESH_CALIBRATE METHOD=scan SCAN_MODE=rapid```
20. Once completed use ```SAVE_CONFIG```
## Temperature Compensation Calibration (Eddy USB ONLY)
> [!CAUTION]
> The following steps (21-31) are for Eddy USB Only. Eddy Coil doesnt have temperature compensation so these steps should be disregarded.

21. Home All Axes and move Z 10 above bed
22. Set idle timeout ```SET_IDLE_TIMEOUT TIMEOUT=36000```
23. Record ambient temp of the BTT Eddy Sensor
![Eddy Temperature](https://github.com/krautech/vyper-klipper/blob/main/images/eddy-pi/eddy-temp.jpg?raw=true)
24. Set max temp for bed (i.e 100c) and set typical temperature for hotend (200c)
25. Wait for BTT Eddy temp to stabilize then record temp.
26. Return to room temp by turning off bed and hotend
> [!TIP]
> If you have a high range to test between ambient and max eddy temp from step 25, you can change the value of STEP=3 to STEP=5 to save you some time. Ideally you want as many calibration points as possible for the best use of eddy but I found for range between 30c-50c a STEP value of 3 was sufficient

27. Run ```PROBE_DRIFT_CALIBRATE PROBE=btt_eddy TARGET=50 STEP=3```  (target should be the temp you recorded of the max recorded temp from step 25.
29. Using [the paper method](https://www.klipper3d.org/Bed_Level.html#the-paper-test) adjust your Z offset.
30. Heat bed and nozzle to same values as step 24
31. As Eddy temp rises at each 3c (STEP=3) increment set the z-offset when prompted using the paper test.
32. Youre all done! :)

Make sure you LIVE ADJUST your z-offset with your first print to really home it in.

## Extras & Notes

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





