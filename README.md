### [Read in English here](https://github.com/DivanX10/cat-bowl-with-scales/blob/main/README_EN.md)

# Дорабатываем автокормушку. Переделываем миску и встраиваем в миску весы

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)


***

### Смотреть [видео](https://youtu.be/qWqOF85e7Kk)

***

### Что нужно для сборки весов для миски:
* Чувствительные тензодатчики с точностью 1 грамм. Найти можно в электронных кухонных весах с круглыми ножками. Берем любые кухонные весы с круглыми ножками, а не с палками. Это легко можно понять, если перевернуть весы. [Я брал такие кухонные весы](https://ozon.ru/t/zewBN6W)
* Контроллер HX711
* ESP8266
* Распечатать платформу 

***

<details>
  <summary><b>Фотографии</b></summary>
  
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/df7389fe-d94a-468a-a0af-940cf160bc81)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/f5922b16-2881-4e63-9c3f-eff8ddc1fa62)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/abe8e139-9b38-483d-9db3-028f81224551)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/9f6fc135-7c15-4b94-b5d5-03907ad124ab)


</details>


<details>
  <summary><b>Схема подключения и сборка</b></summary>


![Схема подключения весов к контроллеру HX711 и к ESP8266](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/bde19c1b-f528-445c-9f29-a02ab361cd80)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/bbecdcee-01e7-4d82-b56b-de997552f5fb)
![1692211420683](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/fed69521-62d4-44f0-bd97-e9a33ec976a5)
![1692211420675](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/f258478b-e6c0-4592-86f6-8c3d846ef2f2)
![1692296894910](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/24c2ed5a-f6fc-49f3-ae14-95871bf6a00d)
![1692299489836](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/dea4d793-994e-4d57-b99e-a52308ee41eb)




  
</details>


<details>
  <summary><b>Код для ESPHome</b></summary>

### Полный код можно посмотреть [здесь](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/config)
***  
Перед тем как использовать весь код, откалибруйте свои весы. Уберите из кода эти строчки и включите журналирование в режиме DEBUG. Так мы будем получать сырые данные. Зафиксируйте вес без груза, скопируйте цифры с логов как есть, потом возъмите груз на 500 грамм и поставьте на весы, зафиксируйте цифры. Все эти цифры запишите в линейный фильтр

Пример фильтра, где `-169085` это сырое значение и это значение без груза на весах, поэтому я указал что данное значение имеет вес 0 грамм, а значение `-92230` отобразилось в логах после того, как я установил груз весом 500 грамм и после указал, что данное значение имеет вес 500 грамм
```
filters:
  - calibrate_linear:
      - -169085 -> 0
      - -92230 -> 500
```

Так выглядит код с журналированием в режиме отладки и без использования фильтра с линейной калибровкой. Это позволит вам получить сырые значения
```
#Журналирование
logger:
  level: DEBUG #Режим отладки

sensor:
  # Весы кошачьей миски
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

Если показания нестабильны и сильно скачут, то можно использовать дополнительный фильтр, например медиана, что уменьшит частое изменение показании. [Подробнее читаем в документации ESPHome](https://esphome.io/components/sensor/index.html#median)

```
      - median:
          window_size: 7
          send_every: 5
          send_first_at: 4
```


</details>

<details>
  <summary><b>3D модель корпуса для весов под миску</b></summary>
  
Платформу спроектировал в программе FreeCAD. Скачать FreeCAD [можно здесь](https://www.freecad.org/?lang=ru). Я вложил 3 файла, два файла STL и один для FreeCAD, где вы сможете отредактировать при необходимости. Я спроектировал так, чтобы тензодатчики держались крепко и сделал клипсы в виде дуги из-за чего тензодатчики с трудом встают на свои места, нужно тоненькой плоской отверткой поддеть, но зато стоят четко и очень трудно их будет демонтировать без повреждения корпуса.

Готовые модели можно скачать [тут](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/files)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/0c233383-4d06-4839-b33a-e1bf852fab4e)


</details>

<details>
  <summary><b>Карточка управления автокормушкой</b></summary>
  
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c761cc49-fe44-45ce-95d7-375ef393cc4a)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/24ff4dbf-113b-410a-813d-a3d76ea75304)


Для работы карточки необходимо установить компоненты
* [fold-entity-row](https://github.com/thomasloven/lovelace-fold-entity-row)
* [multiple-entity-row](https://github.com/benct/lovelace-multiple-entity-row)


```
type: entities
entities:
  - type: custom:fold-entity-row
    head:
      entity: sensor.kukhnia_avto_kormushka_statusy
      name: Миска
      icon: mdi:cat
      secondary_info:
        attribute: Вес корма
        name: Корм
        unit: g
      type: custom:multiple-entity-row
      show_state: false
      state_header: Статус
      entities:
        - entity: group.kitchen_auto_feeder_info_and_menu
          name: Меню
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
        name: Насыпать кошкам корм
        secondary_info: last-changed
      - entity: number.manual_feed
        name: Насыпать корм
      - entity: switch.slow_feed
      - entity: input_boolean.smartfeeder_pour_the_feed_automatically
        name: Автокормежка
      - entity: input_number.smartfeeder_serving_quantity
        name: Количество порции
title: Автокормушка

```

  
</details>


<details>
  <summary><b>Шаблоны: сенсоры, выключатели, группы</b></summary>
  

```
#Используем сенсоры нового образца от 2023
#Документация https://www.home-assistant.io/integrations/template/
#Это пример нового образца шаблонов
#template:
#  - sensor:
#      ...
#  - binary_sensor:


template:
#Кухня: Авто кормушка. Статусы
#Объект: sensor.kukhnia_avto_kormushka_statusy
  - sensor:
      - name: 'Кухня: Автокормушка. Статусы'
        unique_id: kitchen auto feeder status
        icon: mdi:cat
        state: '{{ states("input_boolean.smartfeeder_pour_the_feed_automatically") }}'
        attributes:
          Вес миски: '{{ states("sensor.scales_cat_bowl_weight") }}'
          Вес корма: '{{ states("sensor.scales_cat_bowl_weight_food") }}'
          Наличие миски: '{{ states("binary_sensor.scales_cat_bowl_bowl") }}'
          Наличие корма: '{{ states("binary_sensor.scales_cat_bowl_food") }}'


#Вспомогательный элемент: Input Boolean
#https://www.home-assistant.io/integrations/input_boolean/
input_boolean:
#Автокормушка: Сыпать корм автоматически
#Объект: input_boolean.smartfeeder_pour_the_feed_automatically
  smartfeeder_pour_the_feed_automatically:
    name: "Автокормушка: Сыпать корм автоматически"
    icon: mdi:cat

#Группы
#https://www.home-assistant.io/integrations/group/
group:
#Автокормушка: Инфо и меню
#Объект: group.kitchen_auto_feeder_info_and_menu
  kitchen_auto_feeder_info_and_menu:
    name: "Автокормушка: Инфо и меню"
    icon: mdi:information-outline
    all: false
    entities:
      - button.scales_cat_bowl_restart #Перезагрузить
      - binary_sensor.scales_cat_bowl_bowl #Наличие миски
      - binary_sensor.scales_cat_bowl_food #Наличие корма
      - number.scales_cat_bowl_set_weight_for_bowl #Указать вес миски
      - input_number.kitchen_auto_feeder_min_feed_threshold #Минимальный порог корма
      - sensor.scales_cat_bowl_weight #Вес миски
      - sensor.scales_cat_bowl_weight_food #Вес корма

#Вспомогательный элемент: Число
#https://www.home-assistant.io/integrations/input_number
input_number:
#Автокормушка: Минимальный порог корма
#Объект: input_number.kitchen_auto_feeder_min_feed_threshold
  kitchen_auto_feeder_min_feed_threshold:
    name: "Минимальный порог корма"
    min: 5
    max: 30
    step: 1
    mode: slider #box
    icon: mdi:weight-gram

```
  
</details>


***


### Отвязываем кормушку от облака Tuya и переводим на ESPHome


<details>
  <summary><b>Подключаем ESP к кормушке</b></summary>

  > Используйте платы ESP8266, а хотите ESP32 на свое усмотрение, я использовал ESP32 по той причине, что оно у меня было свободным
 
Выпаиваем чип WBR2 и подключаем ESP

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c1ad69c7-c963-4932-bf9b-0d4a6b19d0ea)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/533b0f16-4dcd-42ce-8f7d-36d0fdd44692)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/ae929434-ed82-4fbf-bc39-5bfa4d290a13)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/87fc1946-cf70-4b3f-ae72-8fb07e55289a)
  
</details>

<details>
  <summary><b>Расшифровка протокола для управления кормушкой</b></summary>

> Примечание. На текущий момент сенсоры пока что не работает из-за отсутствия этой кормушки в компоненте TuyaMCU для ESPHome. В Tasmota эта кормушка в компоненте TuyaMCU доступна и сенсоры там работают. Возможно в будущем добавят кормушку в компонент TuyaMCU для ESPHome. Следите за Issues [здесь](https://github.com/esphome/issues/issues/4844)
  
**Включить меделенную подачу корма**
```
55:AA:00:06:00:05:06:01:00:01:01:13
```

**Выключить меделенную подачу корма**
```
55:AA:00:06:00:05:06:01:00:01:00:12
```
***

**Включить 24 часа**
```
55:AA:03:07:00:05:66:01:00:01:01:77
```

**Выключить 24 часа**
```
55:AA:03:07:00:05:66:01:00:01:00:76
```
***

**Подача порции корма**

1 порция
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:01:17
```

2 порции
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:02:18
```

3 порции
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:03:19
```

4 порции
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:04:1A
```

5 порции
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:05:1B
```

6 порции
```
55:AA:00:06:00:08:03:02:00:04:00:00:00:06:1C
```

***

**Длительность воспроизведения речи**

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

**Сенсор наличия корма в баке**

В контейнере корм имеется
```
55:AA:03:07:00:05:0E:05:00:01:00:22
```

В контейнере корм закончился
```
55:AA:03:07:00:05:0E:05:00:01:01:23
```

</details>

<details>
  <summary><b>Код для ESPHome</b></summary>
  
> Используйте платы ESP8266, а хотите ESP32 на свое усмотрение, я использовал ESP32 по той причине, что оно у меня было свободным

Я выложу два варианта кода, один только для управления кормушкой, а второй код, где будет управление кормушкой и миска с весами.

<details>
  <summary>Управление только кормушкой</summary>
  

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

#Учетные данные Wi-Fi для подключения платы к домашней сети
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: off
  reboot_timeout: 5min

#Если не будет связи с WiFi, то поднимется точка доступа
  ap:
    ssid: ESP Feeder S36 Tuya
    password: !secret ap_esp_password

#Компонент captive portal в ESPHome является резервным механизмом на случай сбоя подключения к настроенному Wi-Fi
captive_portal:

#Веб сервер выключен специально, так как кофемашина выключается самопроизвольно в любой момент
web_server:
  port: 80

# Enable logging
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

#Включить компонент Tuya MCU
tuya:
  time_id: sntp_time


#####################################################################################
########################### Глобальные переменные ###################################
globals:
#Состояние выключателя Slow Feed
  - id: idSavedSwitchSlowFeed
    type: bool
    restore_value: yes
    initial_value: 'false'

#Состояние выключателя 24 Hours
  - id: idSavedSwitch24Hours
    type: bool
    restore_value: yes
    initial_value: 'true'


#####################################################################################
################################## Выключатель ######################################
switch:
#Медленная подача корма
  - platform: template
    name: "Slow Feed"
    icon: mdi:speedometer-slow
    optimistic: true
    lambda: !lambda 'return id(idSavedSwitchSlowFeed);'
    turn_on_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x01, 0x13]
    turn_off_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x00, 0x12]

#Включить отображение на часах 24 часовой формат времени 
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
################################## Сенсор ###########################################
sensor:
#Сенсор уровня сигнала WiFi
  - platform: wifi_signal
    name: "RSSI WiFi"
    icon: mdi:wifi
    update_interval: 60s

#Скрытый сенсор безотказной работы в секундах
  - platform: uptime
    name: "Uptime sec"
    icon: mdi:clock-outline
    id: uptime_sensor
    internal: True #Скрыть - true \показать - false
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
################################### Текстовый сенсор ################################
text_sensor:
#Сенсор IP
  - platform: wifi_info
    ip_address:
      name: IP

#ESPHome Version
  - platform: version
    name: "ESPHome Version"
    hide_timestamp: true
    

#Время безотказной работы
  - platform: template
    name: "Uptime ESP"
    icon: mdi:clock-start
    id: uptime_esp
    entity_category: diagnostic


#####################################################################################
####################################### Кнопка ######################################
button:
#Перезагрузка
  - platform: restart
    name: "Restart"
    icon: mdi:restart

#Кнопка кормежки. Выдает порции корма столько, сколько будет выставлено в ползунке количество корма
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
####################################### Число #######################################
number:
#Выставляем количество подаваемой порции корма
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

#Продолжительность записи речи
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
####################################### Время #######################################
time:
  - platform: sntp
    id: sntp_time
    timezone: Europe/Moscow

```
  
</details>

<details>
  <summary>Управление кормушкой и миской с весами</summary>
  

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

#Учетные данные Wi-Fi для подключения платы к домашней сети
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: off
  reboot_timeout: 5min

#Если не будет связи с WiFi, то поднимется точка доступа
  ap:
    ssid: ESP Feeder S36 Tuya
    password: !secret ap_esp_password

#Компонент captive portal в ESPHome является резервным механизмом на случай сбоя подключения к настроенному Wi-Fi
captive_portal:

#Веб сервер выключен специально, так как кофемашина выключается самопроизвольно в любой момент
web_server:
  port: 80

# Enable logging
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

#Включить компонент Tuya MCU
tuya:
  time_id: sntp_time

#####################################################################################
########################### Глобальные переменные ###################################
globals:
#Состояние выключателя Slow Feed
  - id: idSavedSwitchSlowFeed
    type: bool
    restore_value: yes
    initial_value: 'false'

#Состояние выключателя 24 Hours
  - id: idSavedSwitch24Hours
    type: bool
    restore_value: yes
    initial_value: 'true'


#####################################################################################
################################## Выключатель ######################################
switch:
#Медленная подача корма
  - platform: template
    name: "Slow Feed"
    icon: mdi:speedometer-slow
    optimistic: true
    lambda: !lambda 'return id(idSavedSwitchSlowFeed);'
    turn_on_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x01, 0x13]
    turn_off_action:
      - uart.write: [0x55, 0xAA, 0x00, 0x06, 0x00, 0x05, 0x06, 0x01, 0x00, 0x01, 0x00, 0x12]

#Включить отображение на часах 24 часовой формат времени 
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
################################## Сенсор ###########################################
sensor:
#Сенсор общего веса
  - platform: hx711
    name: "Weight"
    icon: mdi:scale
    id: idWeight
    dout_pin: GPIO15 # DT зеленый провод
    clk_pin: GPIO16  # SCK белый провод
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
      #Если миска извлечена, то вес корма будет 0
      - lambda: !lambda |-
          if (x < 0) return 0;
          return x;
    on_value:
      then:
      - if:
          condition:
              #Если вес миски ниже 20, значит миски нет
              - lambda: 'return id(idWeight).state < 20;'
          then:
              #Опубликовать статус OFF
              - binary_sensor.template.publish:
                  id: idBowl
                  state: OFF
      - if:
          condition:
              #Если вес миски выше 60, значит миска на месте
              - lambda: 'return id(idWeight).state > 60;'
          then:
              #Опубликовать статус ON
              - binary_sensor.template.publish:
                  id: idBowl
                  state: ON

#Сенсор веса корма в миске
  - platform: template
    name: "Weight Food"
    icon: mdi:weight-gram
    id: idWeightFood
    update_interval: 1s
    unit_of_measurement: g
    accuracy_decimals: 0
    device_class: weight
    state_class: measurement
    lambda: 'return id(idWeight).state - id(idSetWeightBowl).state;' #Вычитаем вес миски и получаем вес корма
    filters:
        #Если миска извлечена, то вес корма будет 0
        - lambda: !lambda |-
            if (x < 0) return 0;
            return x;
        #Используем фильтр медиану
        - median:
            window_size: 7
            send_every: 5
            send_first_at: 4
    on_value:
      then:
      - if:
          condition:
              #Если вес миски ниже 1, значит корма в миске нет
              - lambda: 'return id(idWeightFood).state < 1;'
          then:
              #Опубликовать статус OFF
              - binary_sensor.template.publish:
                  id: idFood
                  state: OFF
      - if:
          condition:
              #Если вес миски выше 1, значит корм в миске есть
              - lambda: 'return id(idWeightFood).state > 1;'
          then:
              #Опубликовать статус ON
              - binary_sensor.template.publish:
                  id: idFood
                  state: ON

#Сенсор уровня сигнала WiFi
  - platform: wifi_signal
    name: "RSSI WiFi"
    icon: mdi:wifi
    update_interval: 60s

#Скрытый сенсор безотказной работы в секундах
  - platform: uptime
    name: "Uptime sec"
    icon: mdi:clock-outline
    id: uptime_sensor
    internal: True #Скрыть - true \показать - false
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
################################## Бинарный сенсор ##################################
binary_sensor:
#Наличие миски
  - platform: template
    name: "Bowl"
    icon: mdi:bowl
    id: idBowl
    internal: false #Скрыть - true \показать - false

#Наличие корма в миске
  - platform: template
    name: "Food"
    icon: mdi:bowl
    id: idFood
    internal: false #Скрыть - true \показать - false

#Наличие корма в баке
  - platform: template
    name: "Food level"
    icon: mdi:food-drumstick
    id: idFoodLevel
    internal: false #Скрыть - true \показать - false

#####################################################################################
################################### Текстовый сенсор ################################
text_sensor:
#Сенсор IP
  - platform: wifi_info
    ip_address:
      name: IP

#ESPHome Version
  - platform: version
    name: "ESPHome Version"
    hide_timestamp: true
    

#Время безотказной работы
  - platform: template
    name: "Uptime ESP"
    icon: mdi:clock-start
    id: uptime_esp
    entity_category: diagnostic


#####################################################################################
####################################### Кнопка ######################################
button:
#Перезагрузка
  - platform: restart
    name: "Restart"
    icon: mdi:restart

#Кнопка кормежки. Выдает порции корма столько, сколько будет выставлено в ползунке количество корма
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
####################################### Число #######################################
number:
#Выставляем вес миски чтобы не отображать вес, а отображать вес корма
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

#Выставляем количество подаваемой порции корма
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

#Продолжительность записи речи
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
####################################### Время #######################################
time:
  - platform: sntp
    id: sntp_time
    timezone: Europe/Moscow

```
  
</details>


</details>




