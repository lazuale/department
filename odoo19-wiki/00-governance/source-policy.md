# Политика источников

## Scope

Для фактических утверждений используются только официальные материалы Odoo 19.0.

Допустимые семейства источников:

- Developer Reference;
- актуальный Server Framework 101 и другие актуальные official tutorials;
- Administration documentation;
- User Documentation;
- официальные Contributing / Coding Guidelines, когда речь о development/runtime conventions.

Не используются как основание курса:

- блоги;
- форумы;
- Stack Overflow;
- статьи интеграторов;
- сторонние addons;
- документация других версий, если утверждение относится к Odoo 19.0;
- deprecated tutorial, если актуальный Reference или Server Framework 101 покрывает тот же вопрос.

## Приоритет по типу вопроса

### API / ORM / backend semantics

Основной источник: **Developer Reference**.

Tutorial используется для объяснения и учебной последовательности, но не должен переопределять более точную reference semantics.

### Архитектурная mental model и учебная последовательность

Основной источник: **актуальный Server Framework 101**.

### Deployment / CLI / workers / installation

Основной источник: **Administration documentation** и соответствующая **Developer CLI Reference**.

### User-facing business behavior

Основной источник: **User Documentation** конкретного приложения/feature.

### Development conventions и transactional discipline

Основной источник: **официальные Contributing / Coding Guidelines**, если Reference не описывает это полнее.

## Если официальные источники различаются по детализации

Используем правило:

```text
более точный current Reference
        >
упрощённый current Tutorial
```

для API semantics.

При этом tutorial может оставаться лучшим источником объяснения **зачем** механизм существует.

## Если документация не позволяет установить факт

Пишем:

> Официальная документация Odoo 19.0 не позволяет надёжно установить это утверждение.

Не заменяем пробел предположением.

## Community / Enterprise

Общая страница документации не является доказательством edition availability.

Для каждого спорного module/feature edition evidence хранится отдельно в [edition-ledger.md](edition-ledger.md).

## Дата проверки

Каждый урок содержит `Проверено: YYYY-MM-DD`.

Это важно, потому что документация ветки Odoo 19.0 продолжает обновляться. При существенном обновлении урока дата проверки обновляется вместе с повторной верификацией ключевых claims.

## Прямые ссылки

Внизу каждого урока хранится компактный список official source pages. Не нужно дублировать один и тот же URL после каждого абзаца внутри wiki, если принадлежность утверждений `[ODOO]` и источников урока однозначна.

## Исходный код Odoo

На текущем этапе курса официальный source code **не используется как основной источник фактов**, потому что scope согласован как official documentation only.

Если это правило когда-либо изменится, source-level evidence вводится отдельной правкой governance, а не молча.