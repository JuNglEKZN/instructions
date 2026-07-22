# Настенное управление кондиционером: ArtGallery + Shelly i4 Gen3 + Home Assistant

Эта инструкция продолжает настройку кондиционера AUX после его успешного подключения к Home Assistant через UMDU AC.

Сначала выполните:

[Подключение AUX к Home Assistant через UMDU AC](../aux-umdu-home-assistant/README.md)

Итоговая архитектура:

```text
ArtGallery ▲ / ▼
        │ два входа
        ▼
 Shelly i4 Gen3
        │ Wi‑Fi / локальная интеграция Shelly
        ▼
  Home Assistant
        │ ESPHome
        ▼
     UMDU AC
        │ UART
        ▼
AUX ASW-H18A4/BA-R2DI
```

Кнопка не отключает питание кондиционера. Она отправляет события в Home Assistant, а Home Assistant передаёт штатные команды кондиционеру через UMDU AC.

## Что потребуется

1. Уже работающая сущность кондиционера в Home Assistant, например:

   ```text
   climate.aux_living_room
   ```

2. Shelly i4 Gen3, модель `S3SN-0024X`.
3. Двухклавишная кнопка для жалюзи ArtGallery без фиксации:
   - белая версия — `GAL000119`;
   - для другого цвета — аналогичный механизм ArtGallery в нужной отделке.
4. Рамка ArtGallery.
5. Монтажный провод 0,75–1 мм².
6. Наконечники НШВИ для гибкого провода.
7. Индикатор напряжения или мультиметр.
8. Желательно подрозетник глубиной 60 мм.

Нативный механизм ArtGallery устанавливается непосредственно в рамку ArtGallery. Адаптер AtlasDesign не требуется. Кнопка имеет две клавиши со стрелками, самовозврат и механическую блокировку одновременного нажатия.

## Проверка глубины подрозетника

Ориентировочная установочная глубина механизма ArtGallery — около 23 мм, толщина Shelly i4 Gen3 — около 16 мм. Формально получается 39 мм, но дополнительно требуется место для:

- клемм;
- изгиба проводов;
- фазных перемычек;
- безопасной укладки соединений.

Подрозетник 45 мм будет на грани. Рекомендуется глубина 60 мм.

## Логика управления

| Жест | Действие |
|---|---|
| ▲ коротко | Температура +1 °C |
| ▼ коротко | Температура −1 °C |
| ▲ двойное | Следующая скорость вентилятора |
| ▼ двойное | Включить или выключить качание жалюзи |
| ▲ долго | Сценарий «Комфорт»: Cool, 23 °C, Auto |
| ▼ долго | Выключить кондиционер |

Дополнительное правило:

- если кондиционер выключен, короткое ▲ запускает «Комфорт»;
- короткое ▼ при выключенном кондиционере ничего не делает.

## 1. Проверить электрику в подрозетнике

Shelly i4 Gen3 требует:

```text
L — постоянная фаза
N — нейтраль
```

Устройство работает от 110–240 В AC, имеет четыре входа `SW1–SW4` и не имеет силовых выходов. Оно только считывает нажатия кнопок.

До монтажа проверить:

1. Есть ли в коробке постоянная фаза.
2. Есть ли нейтраль.
3. Не является ли провод возвратом от освещения вместо постоянной фазы.
4. Есть ли место для Shelly, механизма и изгиба проводов.

> [!WARNING]
> Если нейтрали нет, обычный Shelly i4 Gen3 в этом месте работать не будет.

## 2. Электрическое подключение

Все работы выполнять при выключенном автомате и проверенном отсутствии напряжения.

### Общая схема

```text
Сеть 230 В

N ───────────────────────────── Shelly N

L ───────────────┬───────────── Shelly L
                 │
                 ├───────────── общий контакт кнопки ▲
                 │
                 └───────────── общий контакт кнопки ▼

выход кнопки ▲ ───────────────── Shelly SW1
выход кнопки ▼ ───────────────── Shelly SW2

Shelly SW3 — не используется
Shelly SW4 — не используется
```

У Shelly есть два контакта `L`. Один можно использовать для входящей фазы, второй — для перемычки к кнопке. Также фазу можно разветвить отдельной клеммой.

### Контакты ArtGallery

У механизма жалюзи две независимые контактные группы. Перед подключением:

1. Посмотреть схему на корпусе механизма.
2. Либо прозвонить каждую кнопку мультиметром.
3. Найти пару контактов, замыкающуюся при нажатии ▲.
4. Найти пару контактов, замыкающуюся при нажатии ▼.
5. Одну сторону каждой пары соединить с `L`.
6. Вторую сторону вывести на `SW1` и `SW2`.

Контакты кнопки неполярные, но после подключения на них присутствует сетевое напряжение.

### Провода и укладка

Для перемычек удобно использовать гибкий провод 0,75 или 1 мм² с НШВИ. Рекомендуемый порядок:

1. Уложить Shelly в заднюю часть коробки.
2. Оставить доступ к винтам клемм до окончательной установки.
3. Не делать резких перегибов.
4. Не зажимать провода распорными лапками механизма.
5. Проверить свободный возврат клавиш.

## 3. Первичная настройка Shelly

### 3.1. Подать питание

1. Пока не устанавливать рамку окончательно.
2. Включить автомат.
3. Убедиться, что Shelly запустился.
4. Найти сеть вида:

```text
ShellyI4G3-XXXXXXXXXXXX
```

### 3.2. Подключить Shelly к Wi‑Fi

1. Подключиться к точке доступа Shelly.
2. Открыть:

   ```text
   http://192.168.33.1
   ```

3. Перейти в `Settings → Wi‑Fi`.
4. Выбрать домашнюю сеть 2,4 ГГц.
5. Ввести пароль и сохранить настройки.

В Keenetic рекомендуется закрепить IP, например:

```text
192.168.1.61
```

Имя устройства:

```text
shelly-aux-wall-control
```

### 3.3. Обновить прошивку

Открыть:

```text
Settings → Firmware
→ Check for updates
→ Update
```

После обновления дождаться перезапуска.

### 3.4. Отключить облако

Shelly Cloud для этой схемы не требуется. Home Assistant работает с Shelly локально. Облако можно оставить отключённым.

### 3.5. Настроить входы

Для `Input 1`:

```text
Name: Temperature Up
Input mode: Button
Enable input: On
Invert input: Off
```

Для `Input 2`:

```text
Name: Temperature Down
Input mode: Button
Enable input: On
Invert input: Off
```

`Input 3` и `Input 4` можно отключить.

Режим `Button` нужен для событий:

```text
single_push
double_push
triple_push
long_push
btn_down
btn_up
```

## 4. Добавить Shelly в Home Assistant

Открыть:

```text
Настройки
→ Устройства и службы
```

Обычно Shelly появляется в разделе «Обнаружено».

Если устройство не появилось:

```text
Добавить интеграцию
→ Shelly
→ указать IP 192.168.1.61
```

Переименовать устройство:

```text
AUX Wall Control
```

## 5. Определить каналы и события

Не вставлять автоматизацию до проверки фактических номеров каналов.

Открыть:

```text
Инструменты разработчика
→ События
```

Тип события:

```text
shelly.click
```

Нажать «Начать прослушивание» и выполнить:

1. Короткое ▲.
2. Двойное ▲.
3. Долгое ▲.
4. Короткое ▼.
5. Двойное ▼.
6. Долгое ▼.

Пример события:

```yaml
event_type: shelly.click
data:
  device_id: 0123456789abcdef0123456789abcdef
  device: shellyi4g3-aabbccddeeff
  channel: 1
  click_type: single_push
```

Скопировать:

- `device_id`;
- канал верхней кнопки;
- канал нижней кнопки;
- точные значения `click_type`.

В примерах ниже принято:

```text
channel 1 = ▲
channel 2 = ▼
```

## 6. Создать сценарий «Комфорт»

Создать сценарий с ID:

```text
script.aux_comfort
```

YAML:

```yaml
alias: AUX — Комфорт
sequence:
  - action: climate.set_hvac_mode
    target:
      entity_id: climate.aux_living_room
    data:
      hvac_mode: cool

  - delay: "00:00:01"

  - action: climate.set_temperature
    target:
      entity_id: climate.aux_living_room
    data:
      temperature: 23

  - choose:
      - conditions:
          - condition: template
            value_template: >
              {{ 'auto' in
                 (state_attr('climate.aux_living_room', 'fan_modes') or []) }}
        sequence:
          - action: climate.set_fan_mode
            target:
              entity_id: climate.aux_living_room
            data:
              fan_mode: auto

mode: restart
icon: mdi:snowflake-thermometer
```

## 7. Создать сценарий переключения вентилятора

Создать сценарий с ID:

```text
script.aux_cycle_fan
```

YAML:

```yaml
alias: AUX — Следующая скорость вентилятора
sequence:
  - variables:
      supported_modes: >
        {{ state_attr('climate.aux_living_room', 'fan_modes') or [] }}
      current_mode: >
        {{ state_attr('climate.aux_living_room', 'fan_mode') or '' }}

  - variables:
      next_mode: >-
        {% if current_mode == 'auto' and 'low' in supported_modes %}
          low
        {% elif current_mode == 'low' and 'medium' in supported_modes %}
          medium
        {% elif current_mode == 'medium' and 'high' in supported_modes %}
          high
        {% elif 'auto' in supported_modes %}
          auto
        {% elif supported_modes | count > 0 %}
          {{ supported_modes[0] }}
        {% else %}
          {{ '' }}
        {% endif %}

  - condition: template
    value_template: >
      {{ (next_mode | trim) in supported_modes }}

  - action: climate.set_fan_mode
    target:
      entity_id: climate.aux_living_room
    data:
      fan_mode: "{{ next_mode | trim }}"

mode: restart
icon: mdi:fan
```

Пример рассчитан на последовательность:

```text
auto → low → medium → high → auto
```

Если UMDU использует названия `mid`, `quiet`, `turbo` или другие, заменить значения на фактические из атрибута `fan_modes`.

## 8. Создать сценарий качания жалюзи

Создать сценарий с ID:

```text
script.aux_toggle_swing
```

YAML:

```yaml
alias: AUX — Переключить качание жалюзи
sequence:
  - variables:
      supported_modes: >
        {{ state_attr('climate.aux_living_room', 'swing_modes') or [] }}
      current_mode: >
        {{ state_attr('climate.aux_living_room', 'swing_mode') or 'off' }}

  - variables:
      active_mode: >-
        {% set candidates =
           supported_modes | reject('equalto', 'off') | list %}
        {% if candidates | count > 0 %}
          {{ candidates[0] }}
        {% else %}
          {{ '' }}
        {% endif %}

  - variables:
      next_mode: >-
        {% if current_mode == 'off' %}
          {{ active_mode }}
        {% else %}
          off
        {% endif %}

  - condition: template
    value_template: >
      {{ (next_mode | trim) in supported_modes }}

  - action: climate.set_swing_mode
    target:
      entity_id: climate.aux_living_room
    data:
      swing_mode: "{{ next_mode | trim }}"

mode: restart
icon: mdi:swap-vertical
```

Сценарий выбирает первый доступный режим качания, отличный от `off`.

## 9. Создать основную автоматизацию

Заменить:

```text
PASTE_SHELLY_DEVICE_ID
```

на фактический `device_id` из события `shelly.click`.

Проверить номера каналов.

```yaml
alias: AUX — Настенная кнопка ArtGallery
description: Управление кондиционером через Shelly i4 Gen3
triggers:
  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: PASTE_SHELLY_DEVICE_ID
      channel: 1
      click_type: single_push
    id: up_single

  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: PASTE_SHELLY_DEVICE_ID
      channel: 2
      click_type: single_push
    id: down_single

  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: PASTE_SHELLY_DEVICE_ID
      channel: 1
      click_type: double_push
    id: up_double

  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: PASTE_SHELLY_DEVICE_ID
      channel: 2
      click_type: double_push
    id: down_double

  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: PASTE_SHELLY_DEVICE_ID
      channel: 1
      click_type: long_push
    id: up_long

  - trigger: event
    event_type: shelly.click
    event_data:
      device_id: PASTE_SHELLY_DEVICE_ID
      channel: 2
      click_type: long_push
    id: down_long

conditions:
  - condition: template
    value_template: >
      {{ states('climate.aux_living_room')
         not in ['unknown', 'unavailable'] }}

actions:
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'up_single' }}"
        sequence:
          - choose:
              - conditions:
                  - condition: state
                    entity_id: climate.aux_living_room
                    state: "off"
                sequence:
                  - action: script.aux_comfort
            default:
              - action: climate.set_temperature
                target:
                  entity_id: climate.aux_living_room
                data:
                  temperature: >-
                    {% set current =
                       state_attr('climate.aux_living_room',
                                  'temperature') | float(23) %}
                    {% set maximum =
                       state_attr('climate.aux_living_room',
                                  'max_temp') | float(30) %}
                    {{ [current + 1, maximum] | min }}

      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'down_single' }}"
        sequence:
          - condition: template
            value_template: >
              {{ not is_state('climate.aux_living_room', 'off') }}
          - action: climate.set_temperature
            target:
              entity_id: climate.aux_living_room
            data:
              temperature: >-
                {% set current =
                   state_attr('climate.aux_living_room',
                              'temperature') | float(23) %}
                {% set minimum =
                   state_attr('climate.aux_living_room',
                              'min_temp') | float(16) %}
                {{ [current - 1, minimum] | max }}

      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'up_double' }}"
        sequence:
          - action: script.aux_cycle_fan

      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'down_double' }}"
        sequence:
          - action: script.aux_toggle_swing

      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'up_long' }}"
        sequence:
          - action: script.aux_comfort

      - conditions:
          - condition: template
            value_template: "{{ trigger.id == 'down_long' }}"
        sequence:
          - action: climate.set_hvac_mode
            target:
              entity_id: climate.aux_living_room
            data:
              hvac_mode: "off"

mode: queued
max: 10
```

> [!IMPORTANT]
> Перед сохранением проверить фактический `entity_id` кондиционера и трёх сценариев. В этой инструкции используются стабильные английские ID:
>
> ```text
> climate.aux_living_room
> script.aux_comfort
> script.aux_cycle_fan
> script.aux_toggle_swing
> ```

## 10. Приёмочные испытания

Проверять по одному действию.

### Короткое ▲

- при включённом кондиционере: `23 °C → 24 °C`;
- при выключенном: запускается `Cool`, `23 °C`, `Fan Auto`.

### Короткое ▼

- при включённом: `23 °C → 22 °C`;
- при выключенном действие отсутствует.

### Двойное ▲

```text
Auto → Low → Medium → High → Auto
```

### Двойное ▼

```text
Swing Off → Swing On → Swing Off
```

### Долгое ▲

```text
Cool
23 °C
Fan Auto
```

### Долгое ▼

Кондиционер выключается штатной командой без обрыва электропитания.

После каждого теста открыть:

```text
Настройки
→ Автоматизации
→ AUX — Настенная кнопка ArtGallery
→ Трассировки
```

Трассировка покажет, какой триггер сработал и на каком шаге возникла ошибка.

## 11. Проброс в Алису

В Алису экспортируется только:

```text
climate.aux_living_room
```

Shelly и кнопочные event-сущности экспортировать не нужно.

## Что будет работать при сбоях

| Сбой | Что останется доступно |
|---|---|
| Home Assistant выключен | Штатный пульт AUX |
| Shelly недоступен | Home Assistant, Алиса и штатный пульт |
| UMDU недоступен | Только штатный пульт |
| Интернет недоступен | Локальные UMDU, Shelly и Home Assistant продолжают работать |
| Wi‑Fi полностью выключен | Штатный пульт AUX |

Настенная кнопка зависит от Home Assistant, но питание кондиционера никогда не разрывается реле, а штатный пульт остаётся независимым резервным способом управления.

[← Вернуться к списку инструкций](../README.md)
