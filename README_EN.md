# Modernization of the Pettix S36 automatic feeder

Project for ESPHome and Home Assistant

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)


***

### Watch video [Automatic feeder with bowl on scales](https://youtu.be/qWqOF85e7Kk)
### Watch the video [Automatic feeder with a rotating bowl on a scale](https://youtu.be/ATh8qVqtzHo?si=UaMHs_tmDmoli1Qk)

***

## We untie the feeder from the Tuya cloud and transfer it to ESPHome

<details>
  <summary><b>Connecting ESP to the feeder</b></summary>

Unsolder the WBR2 chip and connect the ESP [WBR2 Module Datasheet](https://developer.tuya.com/en/docs/iot/wbr2-datasheet?id=K989h4vonmsey)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c1ad69c7-c963-4932-bf9b-0d4a6b19d0ea)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/533b0f16-4dcd-42ce-8f7d-36d0fdd44692)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/ae929434-ed82-4fbf-bc39-5bfa4d290a13)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/87fc1946-cf70-4b3f-ae72-8fb07e55289a)
  
</details>

<details>
  <summary><b>Decoding the protocol for controlling the feeder</b></summary>

  
**Enable slow feed**
```
55:AA:00:06:00:05:06:01:00:01:01:13
```

**Turn off slow feed**
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

1 serving
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:01:17
```

2 servings
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:02:18
```

3 servings
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:03:19
```

4 servings
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:04:1A
```

5 servings
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:05:1B
```

6 servings
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


</details>


<details>
  <summary><b>ESPHome</b></summary>


### View configurations [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/ESPHome/en)
***  
Calibrate your scale before using all the code. Remove these lines from the code and enable logging in DEBUG mode. This is how we will receive raw data. Record the weight without a load, copy the numbers from the logs as they are, then take a 500 gram load and put it on the scales, record the numbers. Write all these numbers into a linear filter

An example of a filter, where `-169085` is the raw value and this is the value without a weight on the scale, so I indicated that this value has a weight of 0 grams, and the value `-92230` was displayed in the logs after I set the weight to 500 grams and then indicated that this value has a weight of 500 grams
```
filters:
  - calibrate_linear:
      - -169085 -> 0
      - -92230 -> 500
```

This is what the code looks like with logging in debug mode and without using a linear calibration filter. This will allow you to get the raw values
```
#Logging
logger:
  level: DEBUG #Debug mode

sensor:
  # Cat bowl scales
  - platform: hx711
    name: "${node_name} Weight"
    icon: mdi:scale
    id: idWeight
    dout_pin: D7 #DT
    clk_pin: D6  #SCK
    gain: 64
    update_interval: 1s
    unit_of_measurement: g
    accuracy_decimals: 0
    device_class: weight
    state_class: measurement
    entity_category: diagnostic
    internal: False
```

If the readings are unstable, fluctuate frequently and strongly, then replace the HX711 controller with a working one, otherwise the filters will not help. I personally encountered this myself and wasted a lot of time setting up filters. If the HX711 controller is working properly, the scale readings at rest may change slightly and this is normal, but they do not fluctuate greatly and constantly. In this case, an additional filter will help, for example the median filter, which can stabilize the scale readings. [Read more in the ESPHome documentation](https://esphome.io/components/sensor/index.html#median)

```
      - median:
          window_size: 7
          send_every: 4
          send_first_at: 3
```
                
</details>


## Automatic feeder with bowl on scales

<details>
  <summary><b>What you need to assemble a bowl scale</b></summary>
  
* Sensitive strain gauges with 1 gram accuracy. You can find them in electronic kitchen scales with round legs. We take any kitchen scale with round legs, not sticks. This can be easily understood if you turn the scales over. [I took these kitchen scales](https://ozon.ru/t/zewBN6W)
* ESP8266 Wemos Mini D1
* HX711 scale controller
* Print the platform. You can download [here](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/files/STL%20Scales)
</details>

<details>
  <summary><b>Photos</b></summary>
  
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
  <summary><b>Home Assistant</b></summary>

![feeder_pettix_s36_control_panel_scales_ru](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/210d583e-eb97-494c-9856-877aa98ce18d)

**For the card to work, you need to install components**
* [Fold Entity Row](https://github.com/thomasloven/lovelace-fold-entity-row)
* [Multiple Entity Row](https://github.com/benct/lovelace-multiple-entity-row)

**Card and templates**
* You can get the card code [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/en/Card.%20Bowl%20with%20scales.yaml)
* The template code can be found [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/en/Template.yaml)
  
</details>

<details>
  <summary><b>3D model</b></summary>
  
The platform was designed in FreeCAD. Download FreeCAD [available here](https://www.freecad.org/?lang=ru). I have included 3 files, two STL files and one for FreeCAD where you can edit if necessary. I designed it so that the strain gauges would hold tightly and made the clips in the form of an arc, which is why the strain gauges hardly fall into place; you need to pry them off with a thin flat-head screwdriver, but they stand clearly and it will be very difficult to remove them without damaging the case.

Ready-made models can be downloaded [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/STL%20Scales)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/0c233383-4d06-4839-b33a-e1bf852fab4e)


</details>


## Automatic feeder with rotating bowl with scales

<details>
  <summary><b>What you need to assemble a rotating bowl with scales</b></summary>
  
* Sensitive strain gauges with 1 gram accuracy. You can find them in electronic kitchen scales with round legs. We take any kitchen scale with round legs, not sticks. This can be easily understood if you turn the scales over. [I took these kitchen scales](https://ozon.ru/t/zewBN6W)

<img src="https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/b2f3df54-8d77-4d6a-98a2-83872547ea16" width=30%>

* ESP8266 Wemos Mini D1
* HX711 scale controller
* ULN2003 driver module and 28YBJ 48 stepper motor
* DS1307 Real Time Clock (RTS) Module
* Bearing 6814 2RS (61814) SLZ. Took [here](https://ozon.ru/t/6M8ZB3Y)
   *Outer diameter: 90mm
   *Inner diameter: 70mm
   *Height: 10mm
<img src="https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/ef17c186-4428-45d1-8580-bf0b5b19b3b0" width=30%>

* Liquid rubber KUDO COLOR FLEX, Transparent. Took [here](https://ozon.ru/t/wz6gR7n)
  
<img src="https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/080146eb-a8f7-4a47-86b8-b1077bf3b672" width=30%>

* Print out the bowl and platform. You can download [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/STL%20Rotating%20bowl%20with%20scales)

</details>

<details>
  <summary><b>Photos</b></summary>
  
![1696536595687](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/e5091fb4-d05b-433d-994f-a0790e9496eb)
![1696536595643](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/75efbf59-bb92-4c0e-910c-0f0f82f0b723)
![1696035017452](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/c7887c34-0a1d-4a8c-a42e-9700d2406eb4)
![1696536595635](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/739142d2-9f8e-4311-9748-1940fcb5a2c5)
![1695752533054](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/235adf0a-dac5-4fb2-9a26-18c1388e870a)
![1695752533058](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/1a2c6325-2bed-4af7-8e3d-39b1822b20b8)
![1695752533041](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/08a2d907-a161-480d-9c9b-c7cdcb255c00)




</details>

<details>
  <summary><b>Connection diagram</b></summary>

We take power from the automatic feeder itself, which produces 5V and 1A, which will be enough to power the entire feeder completely. For full operation of the feeder, a current of up to 800mA is required.

![Connection diagram 01](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/82ead812-b407-4f3a-81b9-9df1173ad573)


</details>




<details>
  <summary><b>Home Assistant</b></summary>


![feeder_pettix_s36_control_panel_ru](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/acfdeed5-26c6-4066-af82-268f88884132)

**For the card to work, you need to install components**
* [History explorer card](https://github.com/alexarch21/history-explorer-card)
* [Button Card](https://github.com/custom-cards/button-card)

**Card and templates**
* You can get the card code [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/en/Card.%20Rotating%20bowl%20with%20scales.yaml)
* The template code can be found [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/en/Template.yaml)

</details>

<details>
  <summary><b>3D model</b></summary>


The bowl consists of several parts. Made to save printing time and filament in case the part breaks and to avoid printing the entire bowl again. Read about bearing 6814 2RS (61814) SLZ below

Ready-made models can be downloaded [here](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/STL%20Rotating%20bowl%20with%20scales)


![image](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/1300658d-b24c-4751-83ff-323fc8251d8f)

> The orange circle is bearing 6814 2RS (61814) SLZ, which is inserted into a round platform under a steel bowl (blue in the screenshot). To prevent the steel bowl from sliding on the top of the platform, I recommend spraying it with liquid rubber. Provides excellent grip on the bowl

![image](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/69a98ae4-d7f7-41f9-9f3f-d81ba4872770)



</details>
