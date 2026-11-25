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

## Базовый механический/разъёмный уровень модуля

Первый стандарт модуля должен опираться на:

- карточные модули;
- keyed insertion;
- front panel под разъёмы;
- сменную RF panel;
- поддержку 3D‑печатного корпуса;
- обслуживание без полной разборки.

Точный part number разъёма ещё не выбран. Выбор разъёма зависит от:

- тока;
- числа контактов;
- RF/импедансных требований;
- доступности;
- цены;
- сложности сборки.

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
До покупки силовых частей должен быть полный power budget.

Gate 2 — freeze module interface  
До покупки разъёмов надо зафиксировать backplane pinout, семейство разъёмов и module descriptor format.

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
- региональные RF правила.

## Следующие инженерные действия

1. Зафиксировать JSON schema module descriptor.
2. Нарисовать draft power tree backplane.
3. Сделать block diagram MVP carrier.
4. Сравнить 2–3 семейства разъёмов модулей.
5. Подготовить no‑purchase BOM для ревью.
