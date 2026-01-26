[Switch to English version](readme.md)

# 🌫️ Xiaomi Air Purifier в Home Assistant (LAN)

Я подключил свои **Xiaomi Purifier 4 Compact** к **Home Assistant** по LAN.  
Функционально всё работало, но организация стоковых карточек на дашборде меня не устроила. В итоге я сделал кастомную карточку — предсказуемую, компактную и информативную.

![Карточки Air Purifier для Dashboard](assets/air-cleaner-tiles-ru.png)
- иконка меняет цвет в зависимости от уровня загрязнения 🟢🟠🔴
- отображается текущий уровень PM2.5
- когда фильтр заканчивается, отображается ресурс фильтра в днях 
  -  менее 60 дней — оранжевый 🟧
  -  менее 30 дней — красный 🟥
- по клику на карточку можно регулировать скорость работы очистителя
- по клику на иконку отображается график чистоты воздуха — удобно сопоставлять изменения качества воздуха со своими действиями

**Да, это не ракетостроение.  
Но если кому-то это сэкономит пару часов жизни — значит, всё было не зря.**

🐲 Кстати, для Кракова тема очистителей особенно актуальна: на момент написания он попал в топ-4 самых загрязнённых городов мира (в моменте). Это кратковременное зимнее явление но факт неприятный.
![Качество воздуха, Краков, зима 2026](assets/air-quality-krk.png)

## 🛠 Установка и настройка
> Я не стал делать из этого кода пакет потому что на мой взгляд в Home Assistant дистрибуция Lovelace пакетов построена не лучше чем простое копирование вручную, поэтому текст, который вы читаете — статья а не описание инсталляции. Но не переживайте, всё очень просто. Если у вас не Xiaomi устройство, пропустите то что касается Xiaomi.

### 1. Установите необходимые модули
Все модули устанавливаются через HACS. Установите вначале HACS если его у вас нет. Это вообще база для Home Assistant, которая наверняка понадобится вам ещё много раз.
Далее перечислены модули, которые следует установить найдя их по очереди в строке поиска HACS, выбрав каждый и нажав кнопку загрузки:
- ```Xiaomi Miot``` - Для локального подключения Xiaomi Devices;
- ```card-mod``` - Для поддержки стилизации стоковых карточек;
- ```Decluttering card``` - Шаблонизатор, чтобы не дублировать весь код карточки при использовании нескольких устройств.


### 2. Подключите Xiaomi устройства по LAN

> Пожалуйста, пропустите этот пункт если вы уже настроили своё устройство по LAN. Если нет — прочитать будет полезно.

#### Почему полезно подключение именно по LAN а не через облако Xiaomi 🤔
1. Вы не зависите от китайского сервера, управление вашим прибором происходит на расстоянии 10 метров а не через пол мира;
2. Вы не зависите от отключения интернета и перебоев вашего интернет-провайдера;
3. Скорость реакции устройства в вашей локальной сети существенно выше чем через интернет;
4. Вашим устройством по-настоящему управляете вы.

#### Как настроить подключение 
1. Добавьте [интеграцию Xiaomi Miot](https://my.home-assistant.io/redirect/config_flow_start?domain=xiaomi_miot) 
2. Рекомендую авторизоваться вашим MI аккаунтом чтобы получить локальный доступ ко всем вашим устройствам в аккаунте и выбрать тип подключения Local.

### 3. Добавление карточек
На нужном вам dashboard нажмите в правом верхнем углу ⋮ → Редактировать панель (Edit dashboard) → ⋮ → Редактор конфигурации (Raw configuration editor)

> ⚠️ Вышеописанного пункта не будет, если дашборд создан в UI mode. В этом случае выполните следующее действие:  
> Настройки (Settings) → Панели управления (Dashboards) → выбрать панель → Перевести в режим YAML (Take control / Switch to YAML mode)

#### Добавление шаблона
Добавьте следующий код, рекомендую в самый низ:
```yaml
decluttering_templates:
  xiaomi_air_cleaner_tile:
    default:
      name: Air      
    card:
      type: tile
      entity: '[[entity_pm25]]'
      name: '[[name]]'
      icon: mdi:air-filter
      vertical: false
      features_position: bottom
      tap_action:
        action: more-info
        entity: '[[entity_fan]]'
      icon_tap_action:
        action: more-info
      card_mod:
        style: |
          ha-card {
            position: relative;            
            {% set d = states('[[entity_filter]]')|float(999) %}
            {% if d < 30 %}
              --badge-text: "{{ d|round(0) }}d";
              --badge-color: #e74c3c;
            {% elif d < 60 %}
              --badge-text: "{{ d|round(0) }}d";
              --badge-color: #f39c12;
            {% else %}
              --badge-text: "";
            {% endif %}
            {% set v = states('[[entity_pm25]]')|float(0) %}
            {% if v < 12 %}
              --tile-color: #2ecc71 !important;
            {% elif v < 35 %}
              --tile-color: #f39c12 !important;
            {% else %}
              --tile-color: #e74c3c !important;
            {% endif %}
          }
                 
          ha-card::after {
            content: var(--badge-text);
            position: absolute;
            top: 3px;
            right: 6px;
            font-size: 12px;
            font-weight: 400;
            color: var(--badge-color);
            pointer-events: none;
          }
```

#### Добавление карточки:
Есть 2 варианта, можете отредактировать карточку в том же Raw Configuration Editor или же прямо на дашборде переключившись в режим YAML (кода):

```yaml
- type: custom:decluttering-card
  template: xiaomi_air_cleaner_tile
  variables:
    - name: Воздух
    - entity_pm25: sensor.xiaomi_cpa4_9c14_pm25_density
    - entity_filter: sensor.xiaomi_cpa4_9c14_filter_left_time
    - entity_fan: fan.xiaomi_cpa4_9c14_air_purifier
```

- name - необязательное поле, если не указать, карточка будет называться "Air"
- Остальные поля обязательны, требуется указать соответствующие сенсоры вашего устройства для конкретной карточки. Таких карточек вы можете создать любое количество.


## 🛍 Приложение 1 — Что такое HACS?

## 🛍 Appendix 1 — What is HACS?

HACS (Home Assistant Community Store) is a community-driven extension system for Home Assistant.
It allows you to install third-party blueprints, integrations, dashboards, and custom components directly from GitHub — with update notifications and version management.

### How to install HACS

> During the first-time setup, HACS will ask you to sign in to GitHub and authorize access.  
> You need a GitHub account for this (create one if needed: [https://github.com/signup](https://github.com/signup)).  
> 🧘‍♂️ The authorization is read-only — HACS can only download public repositories and cannot modify your GitHub account or data.  

👉 [Follow the official HACS setup instruction](https://hacs.xyz/docs/use/#getting-started-with-hacs)
