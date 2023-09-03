# We are finalizing the autocormer. We redo the bowl and embed the scales in the bowl

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)

***

### Watch [video](https://youtu.be/qWqOF85e7Kk)

***

### What you need to assemble the scales for the bowl:
* Sensitive load cells with an accuracy of 1 gram. You can find it in electronic kitchen scales with round legs. We take any kitchen scales with round legs, not with sticks. This can be easily understood if you flip the scales.
* HX711 Controller
* ESP8266
* Print the platform

***

<details>
  <summary><b>Photo</b></summary>
  
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/df7389fe-d94a-468a-a0af-940cf160bc81)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/f5922b16-2881-4e63-9c3f-eff8ddc1fa62)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/abe8e139-9b38-483d-9db3-028f81224551)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/9f6fc135-7c15-4b94-b5d5-03907ad124ab)


</details>


<details>
  <summary><b>Connection diagram and assembly</b></summary>


![Connection diagram of the scales to the HX711 controller and to the ESP8266](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/bde19c1b-f528-445c-9f29-a02ab361cd80)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/bbecdcee-01e7-4d82-b56b-de997552f5fb)
![1692211420683](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/fed69521-62d4-44f0-bd97-e9a33ec976a5)
![1692211420675](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/f258478b-e6c0-4592-86f6-8c3d846ef2f2)
![1692296894910](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/24c2ed5a-f6fc-49f3-ae14-95871bf6a00d)
![1692299489836](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/dea4d793-994e-4d57-b99e-a52308ee41eb)



</details>


<details>
  <summary><b>Code for ESPHome</b></summary>

### The full code can be viewed [here](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/config)
***
Before using all the code, calibrate your scales. Remove these lines from the code and enable logging in DEBUG mode. This way we will get raw data. Fix the weight without the load, copy the numbers from the logs as is, then take the load for 500 grams and put it on the scales, fix the numbers. Write all these numbers into a linear filter

An example of a filter where `-169085` is a raw value and it is a value without a load on the scale, so I indicated that this value has a weight of 0 grams, and the value `-92230` was displayed in the logs after I set the load weighing 500 grams and then indicated that this value has a weight of 500 grams
```
filters:
  - calibrate_linear:
      - -169085 -> 0
      - -92230 -> 500
```

This is how the code looks with logging in debug mode and without using a filter with linear calibration. This will allow you to get raw values
```
#Logging
logger:
  level: DEBUG #Debugging mode

sensor:
  # Cat Bowl Scales
  - platform: hx711
    name: "${node_name} Weight"
    icon: mdi:scale
    id: idWeight
    dout_pin: D7 # DT
    clk_pin: D6  # SCK
    gain: 64
    update_interval: 2s
    unit_of_measurement: g
    accuracy_decimals: 0
    device_class: weight
    state_class: measurement
    entity_category: diagnostic
    internal: False
```

If the readings are unstable and jump a lot, then you can use an additional filter, such as the median, which will reduce the frequent change in the reading. [Read more in the ESPHome documentation](https://esphome.io/components/sensor/index.html#median)

```
      - median:
          window_size: 7
          send_every: 5
          send_first_at: 4
```


</details>

<details>
  <summary><b>3D model of the body for the scales under the bowl</b></summary>

The platform was designed in the FreeCAD program. Download FreeCAD [here](https://www.freecad.org/?lang=ru). I have attached 3 files, two STL files and one for FreeCAD, where you can edit if necessary. I designed the load cells to hold tight and made clips in the form of an arc, which is why the load cells hardly fall into place, you need to pry with a thin flat screwdriver, but they stand clearly and it will be very difficult to dismantle them without damaging the case.

Ready-made models can be downloaded [here](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/files)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/0c233383-4d06-4839-b33a-e1bf852fab4e)


</details>

<details>
  <summary><b>Automatic feeder control card</b></summary>
  
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c761cc49-fe44-45ce-95d7-375ef393cc4a)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/24ff4dbf-113b-410a-813d-a3d76ea75304)


For the card to work, you need to install the components
* [fold-entity-row](https://github.com/thomasloven/lovelace-fold-entity-row)
* [multiple-entity-row](https://github.com/benct/lovelace-multiple-entity-row)


```
type: entities
entities:
  - type: custom:fold-entity-row
    head:
      entity: sensor.kukhnia_avto_kormushka_statusy
      name: Bowl
      icon: mdi:cat
      secondary_info:
        attribute: Feed weight
        name: Feed
        unit: g
      type: custom:multiple-entity-row
      show_state: false
      state_header: Status
      entities:
        - entity: group.kitchen_auto_feeder_info_and_menu
          name: Menu
          state_color: true
          icon: mdi:information-outline
          styles:
            height: 60px
            width: 50px
        - entity: switch.slow_feed
          name: Slow Feed
          type: button
          state_color: true
          icon: mdi:speedometer-slow
          tap_action:
            action: toggle
          styles:
            height: 60px
            width: 50px
        - entity: input_boolean.smartfeeder_pour_the_feed_automatically
          name: Auto Feed
          type: button
          state_color: true
          icon: mdi:auto-mode
          tap_action:
            action: toggle
          styles:
            height: 60px
            width: 50px
        - entity: number.manual_feed
          name: Feed
          type: button
          state_color: true
          styles:
            height: 60px
            width: 50px
    entities:
      - entity: input_button.smartfeeder_pour_cat_food
        name: Pour cat food
        secondary_info: last-changed
      - entity: number.manual_feed
        name: Pour the feed
      - entity: switch.slow_feed
      - entity: input_boolean.smartfeeder_pour_the_feed_automatically
        name: Auto feeder
      - entity: input_number.smartfeeder_serving_quantity
        name: Serving quantity
title: Car Feeder

```


</details>


<details>
  <summary><b>Templates: sensors, switches, groups</b></summary>


```
#We use sensors of a new sample from 2023
#Documentation https://www.home-assistant.io/integrations/template/
#This is an example of a new template sample
#template:
#  - sensor:
#      ...
#  - binary_sensor:


template:
#Kitchen: Auto feeder. Statuses
#Object: sensor.kukhnia_avto_kormushka_statusy
  - sensor:
      - name: 'Kitchen: Auto feeder. Statuses'
        unique_id: kitchen auto feeder status
        icon: mdi:cat
        state: '{{ states("input_boolean.smartfeeder_pour_the_feed_automatically") }}'
        attributes:
          Bowl weight: '{{ states("sensor.scales_cat_bowl_weight") }}'
          Feed weight: '{{ states("sensor.scales_cat_bowl_weight_food") }}'
          Availability of a bowl: '{{ states("binary_sensor.scales_cat_bowl_bowl") }}'
          Availability of feed: '{{ states("binary_sensor.scales_cat_bowl_food") }}'


#Auxiliary Element: Input Boolean
#https://www.home-assistant.io/integrations/input_boolean/
input_boolean:
#Auto Feeder: Pour feed automatically
#Object: input_boolean.smartfeeder_pour_the_feed_automatically
  smartfeeder_pour_the_feed_automatically:
    name: "Auto Feeder: Pour feed automatically"
    icon: mdi:cat

#Groups
#https://www.home-assistant.io/integrations/group/
group:
#Auto Feeder: Info and menu
#Object: group.kitchen_auto_feeder_info_and_menu
  kitchen_auto_feeder_info_and_menu:
    name: "Auto Feeder: Info and menu"
    icon: mdi:information-outline
    all: false
    entities:
      - button.scales_cat_bowl_restart #Restart
      - binary_sensor.scales_cat_bowl_bowl #Availability of a bowl
      - binary_sensor.scales_cat_bowl_food #Availability of feed
      - number.scales_cat_bowl_set_weight_for_bowl #Specify the weight of the bowl
      - input_number.kitchen_auto_feeder_min_feed_threshold #Minimum feed threshold
      - sensor.scales_cat_bowl_weight #Bowl weight
      - sensor.scales_cat_bowl_weight_food #Feed weight

#Auxiliary element: Number
#https://www.home-assistant.io/integrations/input_number
input_number:
#Auto Feeder: Minimum feed threshold
#Object: input_number.kitchen_auto_feeder_min_feed_threshold
  kitchen_auto_feeder_min_feed_threshold:
    name: "Minimum feed threshold"
    min: 5
    max: 30
    step: 1
    mode: slider #box
    icon: mdi:weight-gram

```

</details>

***

### We untie the feeder from the Tuya cloud and transfer it to ESPHome


<details>
  <summary><b>Connect the ESP to the feeder</b></summary>

  > Use the ESP8266 and ESP32 boards at your discretion, I used ESP32 for the reason that I had it free
 
We solder the WBR2 chip and connect the ESP. [WBR2 Module Datasheet](https://developer.tuya.com/en/docs/iot/wbr2-datasheet?id=K989h4vonmsey)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c1ad69c7-c963-4932-bf9b-0d4a6b19d0ea)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/533b0f16-4dcd-42ce-8f7d-36d0fdd44692)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/ae929434-ed82-4fbf-bc39-5bfa4d290a13)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/87fc1946-cf70-4b3f-ae72-8fb07e55289a)
  
</details>

<details>
  <summary><b>Decoding the protocol for controlling the feeder</b></summary>

> Note. At the moment, the sensors are not working yet due to the absence of this feeder in the TuyaMCU component for ESPHome. In Tasmota, this feeder is available in the TuyaMCU component and the sensors work there. Perhaps in the future they will add a feeder to the TuyaMCU component for ESPHome. Follow issues [here](https://github.com/esphome/issues/issues/4844)
  
**Turn on the slow feed feed**
```
55:AA:00:06:00:05:06:01:00:01:01:13
```

**Turn off the slow feed feed**
```
55:AA:00:06:00:05:06:01:00:01:00:12
```
***

**Enable 24 hours**
```
55:AA:00:06:00:05:66:01:00:01:01:73
```

**Turn off 24 hours**
```
55:AA:00:06:00:05:66:01:00:01:00:72
```
***

**Feed serving**

1 serving of feed
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:01:17
```

2 servings of feed
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:02:18
```

3 servings of feed
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:03:19
```

4 servings of feed
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:04:1A
```

5 servings of feed
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:05:1B
```

6 servings of feed
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:06:1C
```

***

**Voice playback time**

0
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:00:25
```

1
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:01:26
```

2
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:02:27
```

3
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:03:28
```

4
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:04:29
```

5
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:05:2A
```

6
```
55:AA:00:06:00:08:12:02:00:04:00:00:00:06:2B
```

***

**Sensor for the presence of feed in the tank**

There is food in the container
```
55:AA:03:07:00:05:0E:05:00:01:00:22
```

The container has run out of food
```
55:AA:03:07:00:05:0E:05:00:01:01:23
```

</details>

<details>
  <summary><b>Code for ESPHome</b></summary>

I posted two versions of the code, one only for controlling the feeder, and the second code where the feeder and the bowl with scales will be controlled

<details>
  <summary>Control of the feeder only</summary>
  

```
substitutions:
  board_name: ESP Feeder S36 Tuya
  node_name: feeder-s36-tuya

esphome:
  name: feeder-s36-tuya
  friendly_name: feeder-s36-tuya
  comment: ESP Feeder S36 Tuya

esp32:
  board: esp32dev
  framework:
    type: arduino

#Wi-Fi credentials for connecting the board to the home network
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: off
  reboot_timeout: 5min

#If there is no connection with WiFi, then the access point will rise
  ap:
    ssid: ESP Feeder S36 Tuya
    password: !secret ap_esp_password

#The captive portal component in ESPHome is a backup mechanism in case of a connection failure to the configured Wi-Fi
captive_portal:

#Web server
web_server:
  port: 80

#Enable logging
logger:
  level: ERROR
  baud_rate: 0

#Enable Home Assistant API
api:

#Enable OTA
ota:
  password: "esphome"


#####################################################################################
######################################### UART ######################################
uart:
  tx_pin: GPIO1
  rx_pin: GPIO3
  baud_rate: 9600
  stop_bits: 1
  data_bits: 8
  parity: NONE

#Enable the TuyaMCU component
tuya:
  time_id: sntp_time


#####################################################################################
############################## Global variables #####################################
globals:
#Status of the Slow Feed switch
  - id: idSavedSwitchSlowFeed
    type: bool
    restore_value: yes
    initial_value: 'false'

#Switch Status 24 Hours
  - id: idSavedSwitch24Hours
    type: bool
    restore_value: yes
    initial_value: 'true'


#####################################################################################
##################################### switch ########################################
switch:
#Slow feed feed
  - platform: template
    name: "Slow Feed"
    icon: mdi:speedometer-slow
    optimistic: true
    lambda: !lambda 'return id(idSavedSwitchSlowFeed);'
    turn_on_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x01, 0x13]
    turn_off_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x00, 0x12]

#Enable the display on the clock 24 hour time format
  - platform: template
    name: "24 Hours"
    icon: mdi:hours-24
    optimistic: true
    lambda: !lambda 'return id(idSavedSwitch24Hours);'
    turn_on_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x66, 0x01, 0x00, 0x01, 0x01, 0x73]
    turn_off_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x66, 0x01, 0x00, 0x01, 0x00, 0x72]


#####################################################################################
################################## Sensor ###########################################
sensor:
#WiFi Signal Strength Sensor
  - platform: wifi_signal
    name: "RSSI WiFi"
    icon: mdi:wifi
    update_interval: 60s

#Hidden sensor uptime in seconds
  - platform: uptime
    name: "Uptime sec"
    icon: mdi:clock-outline
    id: uptime_sensor
    internal: True #Hide - true \show - false
    update_interval: 60s
    on_raw_value:
      then:
        - text_sensor.template.publish:
            id: uptime_esp
            state: !lambda |-
              int seconds = round(id(uptime_sensor).raw_state);
              int days = seconds / (24 * 3600);
              seconds = seconds % (24 * 3600);
              int hours = seconds / 3600;
              seconds = seconds % 3600;
              int minutes = seconds /  60;
              seconds = seconds % 60;
              return (
                (days ? String(days) + "d " : "") +
                (hours ? String(hours) + "h " : "") +
                (String(minutes) + "m")
              ).c_str();


#####################################################################################
##################################### Text sensor ###################################
text_sensor:
#IP sensor
  - platform: wifi_info
    ip_address:
      name: IP

#ESPHome Version
  - platform: version
    name: "ESPHome Version"
    hide_timestamp: true
    

#Uptime
  - platform: template
    name: "Uptime ESP"
    icon: mdi:clock-start
    id: uptime_esp
    entity_category: diagnostic


#####################################################################################
####################################### Button ######################################
button:
#Reboot
  - platform: restart
    name: "Restart"
    icon: mdi:restart

#The feeding button. Gives out portions of feed as much as the amount of feed will be set in the slider
  - platform: template
    name: "Feed"
    icon: mdi:food-drumstick
    on_press:
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 1;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x01, 0x17]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 2;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x02, 0x18]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 3;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x03, 0x19]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 4;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x04, 0x1A]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 5;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x05, 0x1B]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 6;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x06, 0x1C]


#####################################################################################
###################################### Number #######################################
number:
#We set the amount of feed served
  - platform: template
    name: "Feed Portions"
    icon: mdi:wall-sconce-round-variant
    id: idFeedPortions
    min_value: 1
    max_value: 6
    step: 1
    mode: slider #slider/box
    optimistic: true
    restore_value: true

#Voice playback time
  - platform: template
    name: "Voice Times"
    id: idVoiceTimes
    min_value: 1
    max_value: 6
    step: 1
    mode: slider #slider/box
    optimistic: true
    restore_value: true
    on_value:
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 0;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x00, 0x25]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 1;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x01, 0x26]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 2;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x02, 0x27]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 3;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x03, 0x28]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 4;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x04, 0x29]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 5;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x05, 0x2A]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 6;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x06, 0x2B]


#####################################################################################
######################################## Time #######################################
time:
  - platform: sntp
    id: sntp_time
    timezone: Europe/Moscow

```
  
</details>

<details>
  <summary>Control of the feeder and the bowl with scales</summary>
  

```
substitutions:
  board_name: ESP Feeder S36 Tuya
  node_name: feeder-s36-tuya

esphome:
  name: feeder-s36-tuya
  friendly_name: feeder-s36-tuya
  comment: ESP Feeder S36 Tuya

esp32:
  board: esp32dev
  framework:
    type: arduino

#Wi-Fi credentials for connecting the board to the home network
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: off
  reboot_timeout: 5min

#If there is no connection with WiFi, then the access point will rise
  ap:
    ssid: ESP Feeder S36 Tuya
    password: !secret ap_esp_password

#The captive portal component in ESPHome is a backup mechanism in case of a connection failure to the configured Wi-Fi
captive_portal:

#Web server
web_server:
  port: 80

#Enable logging
logger:
  level: ERROR
  baud_rate: 0

#Enable Home Assistant API
api:

#Enable OTA
ota:
  password: "esphome"

#####################################################################################
######################################### UART ######################################
uart:
  tx_pin: GPIO1
  rx_pin: GPIO3
  baud_rate: 9600
  stop_bits: 1
  data_bits: 8
  parity: NONE

#Enable the TuyaMCU component
tuya:
  time_id: sntp_time

#####################################################################################
############################## Global variables #####################################
globals:
#Status of the Slow Feed switch
  - id: idSavedSwitchSlowFeed
    type: bool
    restore_value: yes
    initial_value: 'false'

#Состояние выключателя 24 Hours
  - id: idSavedSwitch24Hours
    type: bool
    restore_value: yes
    initial_value: 'false'


#####################################################################################
#################################### Switch #########################################
switch:
#Slow feed feed
  - platform: template
    name: "Slow Feed"
    icon: mdi:speedometer-slow
    optimistic: true
    lambda: !lambda 'return id(idSavedSwitchSlowFeed);'
    turn_on_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x01, 0x13]
    turn_off_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x00, 0x12]

#Enable the display on the clock 24 hour time format
  - platform: template
    name: "24 Hours"
    icon: mdi:hours-24
    optimistic: true
    lambda: !lambda 'return id(idSavedSwitch24Hours);'
    turn_on_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x66, 0x01, 0x00, 0x01, 0x01, 0x73]
    turn_off_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x66, 0x01, 0x00, 0x01, 0x00, 0x72]


#####################################################################################
################################## Sensor ###########################################
sensor:
#Total weight sensor
  - platform: hx711
    name: "Weight"
    icon: mdi:scale
    id: idWeight
    dout_pin: GPIO15 # DT
    clk_pin: GPIO16  # SCK
    gain: 64
    update_interval: 1s
    unit_of_measurement: g
    accuracy_decimals: 0
    device_class: weight
    state_class: measurement
    entity_category: diagnostic
    internal: False
    filters:
      - calibrate_linear:
          - 56194 -> 0
          - 127044 -> 500
      - median:
          window_size: 7
          send_every: 5
          send_first_at: 4
      #If the bowl is removed, the feed weight will be 0
      - lambda: !lambda |-
          if (x < 0) return 0;
          return x;
    on_value:
      then:
      - if:
          condition:
              #If the weight of the bowl is below 20, then there is no bowl
              - lambda: 'return id(idWeight).state < 20;'
          then:
              #Publish the OFF status
              - binary_sensor.template.publish:
                  id: idBowl
                  state: OFF
      - if:
          condition:
              #If the weight of the bowl is above 60, then the bowl is in place
              - lambda: 'return id(idWeight).state > 60;'
          then:
              #Publish the status ON
              - binary_sensor.template.publish:
                  id: idBowl
                  state: ON

#Feed weight sensor in a bowl
  - platform: template
    name: "Weight Food"
    icon: mdi:weight-gram
    id: idWeightFood
    update_interval: 1s
    unit_of_measurement: g
    accuracy_decimals: 0
    device_class: weight
    state_class: measurement
    lambda: 'return id(idWeight).state - id(idSetWeightBowl).state;' #Subtract the weight of the bowl and get the weight of the feed
    filters:
        #If the bowl is removed, the feed weight will be 0
        - lambda: !lambda |-
            if (x < 0) return 0;
            return x;
        #We use the median filter
        - median:
            window_size: 7
            send_every: 5
            send_first_at: 4
    on_value:
      then:
      - if:
          condition:
              #If the weight of the bowl is below 1, then there is no food in the bowl
              - lambda: 'return id(idWeightFood).state < 1;'
          then:
              #Publish the OFF status
              - binary_sensor.template.publish:
                  id: idFood
                  state: OFF
      - if:
          condition:
              #If the weight of the bowl is higher than 1, then there is food in the bowl
              - lambda: 'return id(idWeightFood).state > 1;'
          then:
              #Publish the status ON
              - binary_sensor.template.publish:
                  id: idFood
                  state: ON

#WiFi Signal Strength Sensor
  - platform: wifi_signal
    name: "RSSI WiFi"
    icon: mdi:wifi
    update_interval: 60s

#Hidden sensor uptime in seconds
  - platform: uptime
    name: "Uptime sec"
    icon: mdi:clock-outline
    id: uptime_sensor
    internal: True #Hide - true \show - false
    update_interval: 60s
    on_raw_value:
      then:
        - text_sensor.template.publish:
            id: uptime_esp
            state: !lambda |-
              int seconds = round(id(uptime_sensor).raw_state);
              int days = seconds / (24 * 3600);
              seconds = seconds % (24 * 3600);
              int hours = seconds / 3600;
              seconds = seconds % 3600;
              int minutes = seconds /  60;
              seconds = seconds % 60;
              return (
                (days ? String(days) + "d " : "") +
                (hours ? String(hours) + "h " : "") +
                (String(minutes) + "m")
              ).c_str();


#####################################################################################
################################### Binary sensor ###################################
binary_sensor:
#Availability of a bowl
  - platform: template
    name: "Bowl"
    icon: mdi:bowl
    id: idBowl
    internal: false #Hide - true \show - false

#The presence of food in the bowl
  - platform: template
    name: "Food"
    icon: mdi:bowl
    id: idFood
    internal: false #Hide - true \show - false


#####################################################################################
###################################### Text Sensor ##################################
text_sensor:
#IP sensor
  - platform: wifi_info
    ip_address:
      name: IP

#ESPHome Version
  - platform: version
    name: "ESPHome Version"
    hide_timestamp: true
    

#Uptime
  - platform: template
    name: "Uptime ESP"
    icon: mdi:clock-start
    id: uptime_esp
    entity_category: diagnostic


#####################################################################################
####################################### Button ######################################
button:
#Reboot
  - platform: restart
    name: "Restart"
    icon: mdi:restart

#The feeding button. Gives out portions of feed as much as the amount of feed will be set in the slider
  - platform: template
    name: "Feed"
    icon: mdi:food-drumstick
    on_press:
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 1;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x01, 0x17]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 2;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x02, 0x18]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 3;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x03, 0x19]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 4;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x04, 0x1A]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 5;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x05, 0x1B]
      - if:
          condition:
              - lambda: 'return id(idFeedPortions).state == 6;'
          then:
              - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x03, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x06, 0x1C]


#####################################################################################
###################################### Number #######################################
number:
#We set the weight of the bowl so as not to display the weight, but to display the weight of the feed
  - platform: template
    name: "Set weight for bowl"
    icon: mdi:bowl
    id: idSetWeightBowl
    min_value: 70
    max_value: 100
    step: 1
    mode: slider #slider/box
    optimistic: true
    restore_value: true

#We set the amount of feed served
  - platform: template
    name: "Feed Portions"
    icon: mdi:wall-sconce-round-variant
    id: idFeedPortions
    min_value: 1
    max_value: 6
    step: 1
    mode: slider #slider/box
    optimistic: true
    restore_value: true

#Voice playback time
  - platform: template
    name: "Voice Times"
    id: idVoiceTimes
    min_value: 1
    max_value: 6
    step: 1
    mode: slider #slider/box
    optimistic: true
    restore_value: true
    on_value:
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 0;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x00, 0x25]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 1;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x01, 0x26]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 2;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x02, 0x27]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 3;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x03, 0x28]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 4;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x04, 0x29]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 5;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x05, 0x2A]
      - if:
          condition:
              - lambda: 'return id(idVoiceTimes).state == 6;'
          then:
               - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x08, 0x12, 0x02, 0x00, 0x04, 0x00, 0x00, 0x00, 0x06, 0x2B]


#####################################################################################
######################################## Time #######################################
time:
  - platform: sntp
    id: sntp_time
    timezone: Europe/Moscow

```
  
</details>


</details>


