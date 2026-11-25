# SPEC v0.2 — Базовая архитектура до покупок

Статус: черновик на ревью

Цель: определить платформу так, чтобы не покупать лишнее железо.

## Главное правило

Пока не пройдены decision gates из этого документа, покупки не делаем.

Проект всё ещё в стадии архитектуры. v0.2 — это не список покупок.

## Решения

| ID | Решение | Статус |
|----|----------|--------|
| D1 | Ориентир по ядру — Raspberry Pi CM5‑класс, для первого узла предпочтительно 8 GB Lite / no wireless | принято |
| D2 | Backplane должен давать 12 V bus, controlled rails, USB 3.0, Gigabit Ethernet, I2C, UART, GPIO и module ID bus | принято |
| D3 | В backplane желательно заложить опциональный PCIe lane под будущие network/AI/storage модули | принято |
| D4 | Модуль — карточная заменяемая единица с keying и front panel под RF/разъёмы | принято |
| D5 | MVP purchase scope: core carrier, power, status, один network module, один LoRa module | принято |
| D6 | DMR, LTE/SIM, второй Wi‑Fi, dual Ethernet и SDR host — пост‑MVP модули | принято |
| D7 | Low-speed модули на ключуемом IDC 2×8; high-speed — один PCIe x1 edge слот и один USB3 слот | принято |
| D8 | Двойной вход питания: полевой XT30 (8–18 В, приоритет) + USB-C с PD‑триггером на 12 В, объединение по схеме ИЛИ | принято |
| D9 | Два выхода для зарядки: USB-C PD‑источник (9 В/2 А или 12 В/1.5 А) для быстрой зарядки телефона + простой USB-A 5 В/2.4 А | принято |

## Почему core reference — CM5‑класс

Причины:
- сильная экосистема и документация;
- хороший Linux support;
- USB 3.0 и Gigabit Ethernet;
- хорошая периферийная поддержка;
- долгий жизненный цикл семейства;
- большая экосистема carrier boards;
- нормальный upgrade path на более мощные ядра.

Практическое замечание: 16 GB варианты не являются целью MVP из‑за цены и текущей дорогой памяти.

## Требования к backplane v0.2

Backplane должен давать:

- основной field input 12 V;
- защиту от переполюсовки;
- контролируемые линии питания модулей;
- локальные 5 V / 3.3 V;
- USB 3.0;
- Gigabit Ethernet;
- I2C для ID и телеметрии;
- UART service;
- GPIO для кнопок/статуса;
- опциональный footprint PCIe lane.

В первом carrier PCIe можно оставить незаполненным, если это удорожает или повышает риск.

## Базовый уровень разъёмов модуля

Два класса разъёмов:

- Class L: ключуемый IDC 2×8 box header для low-speed слотов (LoRa, GPS, статус, DMR control). 16 контактов: 5 V ×3, GND ×3, I2C, UART, SPI ×3, CS/ID, EN/PG, 3.3 V.
- Class H: один PCIe x1 edge слот (H1) под SDR/NVMe/2nd GbE и один USB3 слот (H2) под USB3 SDR/LTE/накопитель.

См. [Сравнение разъёмов модулей v0.2](MODULE-CONNECTOR-v0.2.md).

## Базовый уровень питания

См. [Архитектура питания v0.2](POWER-ARCHITECTURE-v0.2.md). Кратко:

- основа — вход 12 V, 8–18 В: полевой XT30 (приоритет) или USB-C PD‑триггер на 12 В;
- buck 5 V / 6 A;
- buck 3.3 V / 2 A;
- на каждый слот коммутируемые 5 V с ограничением 2 А и измерением;
- мониторинг INA219 на входе и шине 5 V;
- два выхода для зарядки устройств (D9): USB-C PD‑источник + USB-A 5 В/2.4 А;
- аккумулятор — отдельным gate.

## Что входит в MVP purchase scope

Разрешено только после gates:

1. compute core carrier;
2. power module;
3. status/service module;
4. один network module;
5. один LoRa module.

Что не входит в MVP purchase scope:

- DMR hotspot module;
- LTE modem;
- SIM hardware beyond connector planning;
- second Wi‑Fi module;
- dual Ethernet beyond one initial port unless already planned on the same module;
- SDR host module;
- 5G hardware;
- final enclosure production.

## Gates перед покупкой

Gate 1 — power budget

Черновик есть в «Архитектура питания v0.2». Нужен ревью.

Gate 2 — freeze module interface

Классы разъёмов описаны в «Сравнение разъёмов модулей v0.2». Распиновка backplane и module descriptor ещё открыты.

Gate 3 — core carrier reference

Первый carrier должен быть проверен как схема/мокап до покупки дорогих compute модулей.

Gate 4 — RF plan

До покупки антенн, фильтров, DMR, LTE или SDR нужен региональный RF plan.

## Открытые вопросы

- финальный MCU companion;
- химия батареи и power path;
- протокол Android ↔ node;
- версия схемы module descriptor;
- габариты корпуса;
- региональные RF правила;
- точный part number ключуемого IDC разъёма;
- ограничитель на слот: ИС eFuse vs дискрет;
- выбор PD‑триггера (CH224K / IP2721 / аналог);
- PD‑контроллер источника для выхода зарядки (IP2723T / CH235 / аналог);
- объединение входов: диоды Шоттки vs ideal-diode MOSFET.

## Прогресс

- Дерево питания и шины: черновик готов.
- Двойной вход питания (D8: XT30 приоритет + USB-C PD 12 В): принято.
- Выходы зарядки (D9: USB-C PD‑источник + USB-A 5 В): принято.
- Классы разъёмов модулей: черновик готов.
- Блок-схема MVP carrier: черновик готов.

## Следующие инженерные действия

1. Зафиксировать JSON schema module descriptor.
2. Нарисовать схему backplane power tree по «Архитектуре питания v0.2».
3. Нарисовать схему MVP carrier (CM5 + распиновка backplane).
4. Подтвердить part numbers ключуемого IDC и его токовый рейтинг.
5. Подготовить no‑purchase BOM для ревью.
