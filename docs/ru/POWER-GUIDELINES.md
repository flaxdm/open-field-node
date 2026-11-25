# Power Guidelines

## Основные идеи

- основной field bus: предпочтительно 12 V;
- локальные DC/DC в модулях;
- контролируемые линии питания модулей;
- battery backup;
- solar input;
- safe shutdown;
- измерение тока;
- отдельный power domain для LTE.

## Замечания по LTE

LTE‑модемы имеют заметные импульсы потребления. Power tree должен проектироваться под burst load, а не только среднюю мощность.

## Survival mode

Узел должен поддерживать sleep, scheduled wake и low‑power logging.
