# Open Field Node / Открытый полевой узел

Открытая модульная headless‑платформа для полевой связи, радионаблюдения, survival‑сценариев и аудита собственных сетей.

У устройства нет экрана и клавиатуры. Основной интерфейс — смартфон на Android: постановка задач, live‑мониторинг, анализ и отчёты. Сам узел умеет работать автономно, хранить логи и синхронизироваться позже.

Язык: [English](README.md) | **Русский**

## Статус

Архитектурная база v0.2 готова: дерево питания, классы разъёмов, поэтапные схемы питания и carrier, BOM без покупок. Решения D1–D10 зафиксированы в SPEC v0.2. Покупки не утверждены до прохождения review gates.

## Ключевые документы

- [Полная документация проекта](docs/ru/PROJECT-DOCUMENTATION-ru.md)
- [SPEC v0.2 — архитектурная база до покупок](docs/ru/SPEC-v0.2.md)
- [Архитектура питания v0.2](docs/ru/POWER-ARCHITECTURE-v0.2.md)
- [Сравнение разъёмов модулей v0.2](docs/ru/MODULE-CONNECTOR-v0.2.md)
- [Схема питания backplane v0.2](docs/ru/BACKPLANE-SCHEMATIC-v0.2-ru.md)
- [Схема MVP carrier v0.2](docs/ru/CARRIER-SCHEMATIC-v0.2-ru.md)
- [BOM v0.2 (без покупок)](docs/ru/BOM-v0.2-ru.md)
- [Бюджет мощности v0.2 (Gate 1)](docs/ru/POWER-BUDGET-v0.2-ru.md)
- [Дескриптор модуля v0.2](docs/ru/MODULE-DESCRIPTOR-v0.2-ru.md)
- [Стандарт модуля v0.2](docs/ru/MODULE-STANDARD-v0.2-ru.md)
- [Заметки по поднятию Wi-Fi v0.2](docs/ru/WIFI-BRINGUP-v0.2-ru.md)
- [Шпаргалка по сборке P0 (день получения)](docs/ru/P0-ASSEMBLY-CHECKLIST-ru.md)
- [Критерии приёмки P0 v0.2](docs/ru/P0-ACCEPTANCE-v0.2-ru.md)
- [Идентичность узла и RTC v0.2](docs/ru/NODE-IDENTITY-v0.2-ru.md)
- [LED и кнопка: интерфейс v0.2](docs/ru/HMI-LED-BUTTON-v0.2-ru.md)
- [Механизм обновления ПО v0.2](docs/ru/UPDATE-MECHANISM-v0.2-ru.md)
- [Бэклог v0.2 (проработка)](docs/ru/BACKLOG-v0.2-ru.md)
- [Программная архитектура v0.2](docs/ru/SOFTWARE-ARCHITECTURE-v0.2-ru.md)
- [Список закупки dev-станции](docs/ru/DEV-STATION-BUYLIST-ru.md)
- [Соответствие готовым модулям v0.2](docs/ru/READY-MODULES-v0.2-ru.md)
- [RF plan v0.2](docs/ru/RF-PLAN-v0.2-ru.md)
- [Механический конверт v0.2](docs/ru/MECHANICAL-v0.2-ru.md)
- [Выбор сетевого модуля v0.2](docs/ru/NETWORK-MODULE-v0.2-ru.md)
- [Сборка прототипа P0 v0.2](docs/ru/P0-PROTOTYPE-v0.2-ru.md)
- [Архитектура](docs/ru/ARCHITECTURE.md)
- [Стандарт модуля](docs/ru/MODULE-STANDARD.md)
- [Capability API](docs/ru/CAPABILITY-API.md)

## Область применения

- полевая связь;
- радиопрослушивание и SDR;
- LoRa/Meshtastic;
- DMR hotspot как опциональный модуль;
- два Ethernet / два Wi‑Fi / LTE‑SIM;
- offline survival‑режим;
- лаборатория аудита собственных сетей.

## Важно

Security‑ и RF‑передающие функции рассчитаны только на собственные устройства, собственную сеть или изолированный стенд.

## Лицензии

Структура лицензий описана в `LICENSES/README.md`.
