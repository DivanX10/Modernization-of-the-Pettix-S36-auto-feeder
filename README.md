### [Read in English here](https://github.com/DivanX10/cat-bowl-with-scales/blob/main/README_EN.md)

# Модернизация автокормушки Pettix S36

Проект для ESPHome и Home Assistant

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/680f93cf-808a-4fb4-938e-c62c3f006a86)


***

### Смотреть видео [Автокормушка с миской на весах](https://youtu.be/qWqOF85e7Kk)
### Смотреть видео [Автокормушка с вращающейся миской на весах](https://youtu.be/ATh8qVqtzHo?si=UaMHs_tmDmoli1Qk)

***

## Отвязываем кормушку от облака Tuya и переводим на ESPHome

<details>
  <summary><b>Подключаем ESP к кормушке</b></summary>

  > Используйте платы ESP8266 и ESP32 на свое усмотрение, я использовал ESP32 по той причине, что оно у меня было свободным
 
Выпаиваем чип WBR2 и подключаем ESP. [WBR2 Module Datasheet](https://developer.tuya.com/en/docs/iot/wbr2-datasheet?id=K989h4vonmsey)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/c1ad69c7-c963-4932-bf9b-0d4a6b19d0ea)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/533b0f16-4dcd-42ce-8f7d-36d0fdd44692)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/ae929434-ed82-4fbf-bc39-5bfa4d290a13)
![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/87fc1946-cf70-4b3f-ae72-8fb07e55289a)
  
</details>

<details>
  <summary><b>Расшифровка протокола для управления кормушкой</b></summary>

  
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
55:AA:00:06:00:05:66:01:00:01:01:73
```

**Выключить 24 часа**
```
55:AA:00:06:00:05:66:01:00:01:00:72
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

**Время воспроизведения голоса**

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


</details>



## Автокормушка с миской на весах

<details>
  <summary><b>Что нужно для сборки весов для миски</b></summary>
  
* Чувствительные тензодатчики с точностью 1 грамм. Найти можно в электронных кухонных весах с круглыми ножками. Берем любые кухонные весы с круглыми ножками, а не с палками. Это легко можно понять, если перевернуть весы. [Я брал такие кухонные весы](https://ozon.ru/t/zewBN6W)
* ESP8266 Wemos Mini D1
* Контроллер весов HX711
* Распечатать платформу. Скачать можно [здесь](https://github.com/DivanX10/cat-bowl-with-scales/tree/main/files/STL%20Scales)
</details>

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
  <summary><b>ESPHome</b></summary>

### Полный код можно посмотреть [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/ESPHome/ru/feeder-pettix-s36%20(весы).yaml)
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
  <summary><b>Home Assistant</b></summary>

![feeder_pettix_s36_control_panel_scales_ru](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/210d583e-eb97-494c-9856-877aa98ce18d)

**Для работы карточки необходимо установить компоненты**
* [Fold Entity Row](https://github.com/thomasloven/lovelace-fold-entity-row)
* [Multiple Entity Row](https://github.com/benct/lovelace-multiple-entity-row)

**Карточка и шаблоны**
* Код карточки можно взять [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/ru/Карточка.%20Миска%20с%20весами.yaml)
* Код шаблона можно взять [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/ru/Шаблон.yaml)
  
</details>

<details>
  <summary><b>3D модель</b></summary>
  
Платформу спроектировал в программе FreeCAD. Скачать FreeCAD [можно здесь](https://www.freecad.org/?lang=ru). Я вложил 3 файла, два файла STL и один для FreeCAD, где вы сможете отредактировать при необходимости. Я спроектировал так, чтобы тензодатчики держались крепко и сделал клипсы в виде дуги из-за чего тензодатчики с трудом встают на свои места, нужно тоненькой плоской отверткой поддеть, но зато стоят четко и очень трудно их будет демонтировать без повреждения корпуса.

Готовые модели можно скачать [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/STL%20Scales)

![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/0c233383-4d06-4839-b33a-e1bf852fab4e)


</details>


## Автокормушка с вращающейся миской с весами

<details>
  <summary><b>Что нужно для сборки вращающейся миски с весами</b></summary>
  
* Чувствительные тензодатчики с точностью 1 грамм. Найти можно в электронных кухонных весах с круглыми ножками. Берем любые кухонные весы с круглыми ножками, а не с палками. Это легко можно понять, если перевернуть весы. [Я брал такие кухонные весы](https://ozon.ru/t/zewBN6W)
* ESP8266 Wemos Mini D1
* Контроллер весов HX711
* Модуль драйвера ULN2003 и шаговый двигатель 28YBJ 48
* Модуль часов реального времени (RTS) DS1307
* Подшипник 6814 2RS (61814) SLZ Подшипник. Брал [здесь](https://ozon.ru/t/6M8ZB3Y)
<img src="https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/ef17c186-4428-45d1-8580-bf0b5b19b3b0" width=40%>

* Распечатать миску и платформу. Скачать можно [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/STL%20Rotating%20bowl%20with%20scales)

</details>

<details>
  <summary><b>Cхема подключения</b></summary>


  ![image](https://github.com/DivanX10/cat-bowl-with-scales/assets/64090632/00910005-fac4-4c2f-b378-c91905fcea85)

</details>

<details>
  <summary><b>ESPHome</b></summary>


### Полный код можно посмотреть [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/ESPHome/ru/feeder-pettix-s36%20(вращающаяся%20миска%20и%20весы).yaml)
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
  <summary><b>Home Assistant</b></summary>


![feeder_pettix_s36_control_panel_ru](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/acfdeed5-26c6-4066-af82-268f88884132)

**Для работы карточки необходимо установить компоненты**
* [History explorer card](https://github.com/alexarch21/history-explorer-card)
* [Button Card](https://github.com/custom-cards/button-card)

**Карточка и шаблоны**
* Код карточки можно взять [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/ru/Карточка.%20Вращающаяся%20миска%20с%20весами.yaml)
* Код шаблона можно взять [здесь](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/blob/main/files/HomeAssistant/ru/Шаблон.yaml)

</details>

<details>
  <summary><b>3D модель</b></summary>


Миска состоит из нескольких деталей. Сделано для экономии времени печати и филамента на случай, если деталь сломалась и чтобы не печатать всю миску целиком заново

Готовые модели можно скачать [тут](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/tree/main/files/STL%20Rotating%20bowl%20with%20scales)


![3d model rotating bowl with scales](https://github.com/DivanX10/Modernization-of-the-Pettix-S36-auto-feeder/assets/64090632/5ea1d579-6f5d-48d0-b5c5-f5021d340d0c)


</details>



