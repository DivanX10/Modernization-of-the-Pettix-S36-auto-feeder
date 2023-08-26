# We are finalizing the autocormer. We redo the bowl and embed the scales in the bowl

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)



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

### Watch [video](https://youtu.be/qWqOF85e7Kk)
