### [Read in English here](https://github.com/DivanX10/cat-bowl-with-scales/blob/main/README_EN.md)

# Дорабатываем автокормушку. Переделываем миску и встраиваем в миску весы

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)





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

### Смотреть [видео](https://youtu.be/qWqOF85e7Kk)

