Organized files into folders, so that you can simply add a few lines to your YAML file and ESPHome will automatically download the files from this repo and use them when compiling your firmware. Tested with ESPHome 2024.6.6. This fork is hard coded for 500mAh cell capacity.

Add this towards the top of your YAML file:
```
external_components:
  - source:
      type: git
      url: https://github.com/jakea333/lc709203f_500mAh
    components: [ lc709203f ]
```
    
In addition to the instructions below, you may need to add this to your YAML file:
```
i2c:
  id: bus_a
```
  
## initial support for lc709203f on  ESP32_Bat_Pro 
 This is an attempt to add wrappers to the code provided by EzSBC in https://github.com/EzSBC/ESP32_Bat_Pro by
adding required functions and renaming some subroutines. These were modelled on the esphome ads1115 and ina219 components

```
captured with mosquitto_sub  -F "%I %t %p" -v -t ezmin/debug
...
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:119]: LC709203F:
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:120]:   Address: 0x0B
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:126]:   Update Interval: 30.0s
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:128]:   Cell Voltage 'ezmin battery V'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:128]:     Unit of Measurement: 'V'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:128]:     Accuracy Decimals: 2
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:129]:   Cell Rem Pct 'ezmin battery lvl'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:129]:     Unit of Measurement: '%'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:129]:     Accuracy Decimals: 1
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:130]:   Cell StateCharge 'ezmin cell charge'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:130]:     Unit of Measurement: '%'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:130]:     Accuracy Decimals: 0
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:131]:   IC version 'ezmin ic'
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:131]:     Unit of Measurement: ''
2021-04-04T11:38:24-0400 ezmin/debug [C][lc709203f.sensor:131]:     Accuracy Decimals: 0
...
2021-04-04T11:38:42-0400 ezmin/debug [D][lc709203f.sensor:092]: Got Battery values: cellVoltage_mV=4149 cellRemainingPercent10=964 cellStateOfCharge=96 ic=0x2717
2021-04-04T11:38:42-0400 ezmin/debug [D][sensor:092]: 'ezmin battery V': Sending state 4.14900 V with 2 decimals of accuracy
2021-04-04T11:38:42-0400 ezmin/debug [D][sensor:092]: 'ezmin battery lvl': Sending state 96.40000 % with 1 decimals of accuracy
2021-04-04T11:38:42-0400 ezmin/debug [D][sensor:092]: 'ezmin ic': Sending state 10007.00000  with 0 decimals of accuracy
2021-04-04T11:38:42-0400 ezmin/debug [D][sensor:092]: 'ezmin cell charge': Sending state 96.00000 % with 0 decimals of accuracy
2021-04-04T11:39:23-0400 ezmin/debug [I][deep_sleep:067]: Beginning Deep Sleep
...
```

works to retreive these 4 values. I have hard coded my battery config in setup routine



#### Required code changes:
- revised [ESPHOME]/custom_components/lc709203f/lc709203f.h
- revised [ESPHOME]/custom_components/lc709203f/lc709203f.cpp
- revised [ESPHOME]/custom_components/lc709203f/sensor.py



#### add stanza to yaml for RGB led
```
switch:
  - platform: gpio
    id: led_red
    name: LED Red
    pin:
      number: GPIO16
      inverted: true
  - platform: gpio
    id: led_green
    name: LED Green
    pin:
      number: GPIO17
      inverted: true
  - platform: gpio
    id: led_blue
    name: LED Blue
    pin:
      number: GPIO18
      inverted: true
```


#### add stanza to yaml for fuel gauge 
```
i2c:
  id: bus_a

sensor:
  - platform: lc709203f
    i2c_id: bus_a
    address: "0x0B"
    battery_voltage:
      name: Battery Volt
    battery_level:
      name: Battery Level
    icversion:
      name: IC Version
    cell_charge:
      name: Cell charge
    update_interval: 30s
```






#### TODO
- accept config for setPowerMode, setCellCapacity, setCellProfile  ( to replace hard coded setup )
- accept value for lc709203f_current_direction_t mentioned at : https://github.com/RIOT-OS/RIOT/blob/df3ab9f16135dca610d81c0286eb23791170b21d/drivers/include/lc709203f.h
- TODO  low priority because no resource/application for me---------------------
- capture values for getThermistorBeta, getCellTemperature 
- accept config for setThermistorB, setTemperatureMode, setAlarmRSOC, setAlarmVoltage

