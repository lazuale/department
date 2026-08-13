# Реестр редакций Odoo 19.0

Этот файл хранит только **доказанный** edition status. Он не является списком всех приложений Odoo.

Статусы:

- **Community подтверждено**;
- **Enterprise подтверждено**;
- **Редакция не установлена**.

## Платформенная граница

| Объект / утверждение | Статус | Official evidence | Проверено |
|---|---|---|---|
| Community edition существует как open-source основа Odoo | Community подтверждено | Architecture Overview / Licenses | 2026-08-13 |
| Enterprise functionality поставляется дополнительными modules поверх Community modules/server | Enterprise подтверждено как дополнительный слой | Architecture Overview; Source install | 2026-08-13 |
| Official developer tutorial custom addon examples входят в канонический Community ERP baseline | **Нет** — это учебные примеры, не baseline modules | Governance rule | 2026-08-13 |

## Как добавлять записи

Новая строка появляется только когда module/feature реально встречается в курсе.

Шаблон:

| Feature / module | Статус | Official evidence | Проверено |
|---|---|---|---|
| `<name>` | Community / Enterprise / не установлена | official Odoo 19 URL/page | YYYY-MM-DD |

## Запрещённый вывод

```text
страница есть в Odoo 19 Documentation
        ≠
feature подтверждена как Community
```

Если documentation не даёт edition evidence, статус остаётся **«Редакция не установлена»**.