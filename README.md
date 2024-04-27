This is for Eddy V1 USB ONLY


1. Push and hold boot button on Eddy (Its next to where the cable plugs in) and at the same time, plug in the cable to your Raspberry Pi
2. SSH into raspberry PI
3. Type ```lsusb``` into the command line. You should see eddy. 

4. Type  cd klipper into command line
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
10. Place Eddy Approx. 20mm above the bed.
11. From Mainsail or Fluidd run command  ```LDC_CALIBRATE_DRIVE_CURRENT CHIP=btt_eddy```
12. Type ```SAVE_CONFIG``` to save the drive currant to your config
13. Home X and Y axes with command G28 X Y
14. Make sure you dont have a bed heightmap loaded.
15. Move Nozzle to Centre of the bed with ```G0 X125 Y125 F6000``` (adjust for your bed size)
16. Start Manual Z-Offset Calibration by typing ```PROBE_EDDY_CURRENT_CALIBRATE CHIP=btt_eddy ```
17. Once completed use ```SAVE_CONFIG```
18. Home All Axes
19. Use command ```BED_MESH_CALIBRATE METHOD=scan SCAN_MODE=rapid```
20. Once completed use ```SAVE_CONFIG```
21. Home All Axes and move Z 30 above bed
22. Set idle timeout ```SET_IDLE_TIMEOUT TIMEOUT=36000```
23. Record ambient temp of the BTT Eddy Sensor
24. Set max temp for bed (i.e 100c) and set typical temperature for hotend (200c)
25. Wait for BTT Eddy temp to stabilize then record temp.
26. Return to room temp by turning off bed and hotend
27. Run ```PROBE_DRIFT_CALIBRATE PROBE=btt_eddy TARGET=50 STEP=3```  (target should be the temp you recorded of the max recorded temp from step 25.
28. Quickly set Z offset
29. Heat bed and nozzle
30. As Eddy temp rises at each 5c increment set the z-offset when prompted





