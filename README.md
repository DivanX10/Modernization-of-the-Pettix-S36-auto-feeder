# Кошачья миска с весами на ESP

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)

Дорабатываем автокормушку. Переделываем миску и встраиваем в миску весы



<details>
  <summary><b>Схема подключения и сборка</b></summary>
  

![Схема подключения весов к контроллеру HX711 и к ESP8266](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/bde19c1b-f528-445c-9f29-a02ab361cd80)

![1692211420683](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/fed69521-62d4-44f0-bd97-e9a33ec976a5)
![1692211420675](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/f258478b-e6c0-4592-86f6-8c3d846ef2f2)
![1692296894910](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/24c2ed5a-f6fc-49f3-ae14-95871bf6a00d)
![1692299489836](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/dea4d793-994e-4d57-b99e-a52308ee41eb)




  
</details>


<details>
  <summary><b>Код для ESPHome</b></summary>
  

```
substitutions:
  board_name: Scales for a cat bowl
  node_name: scales-cat-bowl

esphome:
  name: ${node_name}
  comment: WeMos D1 Scales for a cat bowl

esp8266:
  board: d1_mini
  framework:
    version: recommended

#Учетные данные Wi-Fi для подключения платы к домашней сети
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  fast_connect: off
  reboot_timeout: 5min

#Если не будет связи с WiFi, то поднимется точка доступа
  ap:
    ssid: ESP Сat Tray Hotspot
    password: !secret ap_esp_password

#Компонент captive portal в ESPHome является резервным механизмом на случай сбоя подключения к настроенному Wi-Fi.
captive_portal:

#Веб сервер
web_server:
  port: 80

#Журналирование
logger:
  level: ERROR

#Enable OTA
ota:
  password: "esphome"

#Enable Home Assistant API
#Шифрование выключил для снижения нагрузки на ESP
api:

#####################################################################################
################################## Сенсор ###########################################
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
    filters:
      - calibrate_linear:
          - -169085 -> 0
          - -92230 -> 500
      - delta: 2
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
      - if:
          condition:
              #Если вес миски ниже 99, значит корма в миске нет
              - lambda: 'return id(idWeight).state < 99;' 
          then:
              #Опубликовать статус OFF
              - binary_sensor.template.publish:
                  id: idFood
                  state: OFF
      - if:
          condition:
              #Если вес миски выше 99, значит корм в миске есть
              - lambda: 'return id(idWeight).state > 99;' 
          then:
              #Опубликовать статус ON
              - binary_sensor.template.publish:
                  id: idFood
                  state: ON

  - platform: template
    name: "${node_name} Weight Food"
    id: idWeightFood
    update_interval: 2s
    unit_of_measurement: g
    accuracy_decimals: 0
    device_class: weight
    state_class: measurement
    icon: mdi:weight-gram
    lambda: 'return id(idWeight).state - id(idSetWeightBowl).state;' #Вычитаем вес миски и получаем вес корма
    filters:
        #Если миска извлечена, то вес корма будет 0
        - lambda: !lambda |-
            if (x < 0) return 0;
            return x;

#Сенсор уровня сигнала WiFi
  - platform: wifi_signal
    name: ${node_name} RSSI WiFi
    icon: mdi:wifi
    update_interval: 60s

#Скрытый сенсор безотказной работы в секундах
  - platform: uptime
    name: "${node_name} Uptime sec"
    icon: mdi:clock-outline
    id: uptime_sensor
    internal: False #Скрыть - true \показать - false


#####################################################################################
################################## Бинарный сенсор ##################################
binary_sensor:
#Наличие миски
  - platform: template
    name: "${node_name} Bowl"
    icon: mdi:bowl
    id: idBowl
    internal: false #Скрыть - true \показать - false

#Наличие корма в миске
  - platform: template
    name: "${node_name} Food"
    icon: mdi:bowl
    id: idFood
    internal: false #Скрыть - true \показать - false

#####################################################################################
################################### Текстовый сенсор ################################
#Время безотказной работы
text_sensor:
  - platform: wifi_info
    ip_address:
      name: ${node_name} IP
#    ssid:
#      name: ${board_name} SSID
#    bssid:
#      name: ${board_name} BSSID
#    mac_address:
#      name: ${board_name} Mac
#    scan_results:
#      name: ${board_name} Latest_Scan_Results


#####################################################################################
####################################### Число #######################################
#Ползунок для управления сервоприводом
#Указать вес для миски
number:
  - platform: template
    name: "${node_name} Set weight for bowl"
    id: idSetWeightBowl
    min_value: 70
    max_value: 100
    step: 1
    mode: slider #slider/box
    optimistic: true
    restore_value: true
#####################################################################################
####################################### Кнопка ######################################
button:
  - platform: restart
    name: "${node_name} Restart"
    icon: mdi:restart

#####################################################################################
####################################### Время #######################################
time:
  - platform: sntp
    id: sntp_time
    timezone: Europe/Moscow
```
  
</details>

<details>
  <summary><b>3D модель корпуса для весов под миску</b></summary>
  

Готовые модели можно скачать [тут](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/files)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/0c233383-4d06-4839-b33a-e1bf852fab4e)


</details>

<details>
  <summary><b>Карточка управления автокормушкой</b></summary>
  
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c761cc49-fe44-45ce-95d7-375ef393cc4a)

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
      - sensor.scales_cat_bowl_weight #Вес миски
      - sensor.scales_cat_bowl_weight_food #Вес корма
      - sensor.auto_feeder_feed_per_day #Выдано корма в день
      - sensor.auto_feeder_feed_per_week #Выдано корма в неделю
      - sensor.auto_feeder_feed_per_month #Выдано корма в месяц





```
  
</details>
