# Политика источников

## Scope

Для фактических утверждений используются только официальные материалы **Odoo 19.0**.

Допустимые семейства источников:

- Developer Reference;
- актуальный Server Framework 101 и другие current official tutorials;
- Administration documentation;
- User Documentation;
- official Contributing / Coding Guidelines, когда речь о development/runtime conventions.

Не используются как основание курса:

- блоги;
- форумы;
- Stack Overflow;
- статьи интеграторов;
- сторонние addons;
- документация других версий, если claim относится к Odoo 19.0;
- `saas-19.x` как замена ветке 19.0;
- deprecated tutorial, если current Reference или Server Framework 101 покрывает тот же вопрос.

## Product scope

Baseline курса — **Odoo 19.0 Community self-hosted**.

Odoo Online и Odoo.sh могут изучаться позже как отдельные deployment/product contexts, но их поведение не переносится автоматически на Community self-hosted.

## Приоритет по типу вопроса

### API / ORM / backend semantics

Основной источник: **Developer Reference**.

Tutorial используется для объяснения и learning flow, но не переопределяет более точную current Reference semantics.

### Архитектурная mental model

Основной источник: **current Server Framework 101** плюс relevant Developer Reference.

### Deployment / CLI / workers / server-wide modules

Основной источник: **Administration documentation** и **Developer CLI Reference**.

### User-facing business behavior

Основной источник: **User Documentation** конкретного application/feature.

### Development conventions / transaction discipline

Основной источник: official **Contributing / Coding Guidelines**, если Developer Reference не описывает вопрос полнее.

## Если official sources различаются по детализации

Для API semantics:

```text
более точный current Reference
        >
упрощённый current Tutorial
```

Tutorial при этом может быть лучшим источником объяснения purpose/learning sequence.

## Если documentation не позволяет установить факт

Пишем:

> Официальная документация Odoo 19.0 не позволяет надёжно установить это утверждение.

Пробел не заменяется предположением.

## Community / Enterprise

Наличие общей documentation page не является edition evidence.

Для конкретных module/feature edition evidence хранится в [edition-ledger.md](edition-ledger.md).

## Ограничение documentation-only

Current governance сознательно не использует official source code/manifests как normative evidence.

Следствие:

- documented platform semantics можно изучать глубоко;
- user-facing documented behavior можно проверять;
- но exhaustive Community addon inventory **не гарантируется**.

Если потребуется технически полный Community whitelist, policy должна быть отдельно изменена до использования official source/manifests.

## Source IDs в уроках

Каждый урок назначает локальные source IDs:

```text
S1 — ORM API
S2 — Security Reference
S3 — Coding Guidelines
```

Claims оформляются:

```text
**[ODOO][S1]** ...
```

Один source ID может подтверждать несколько claims.

Для `[ВЫВОД]` source ID не обязателен прямо в marker, но supporting `[ODOO]` claims должны быть traceable.

## Дата проверки

Каждый урок содержит:

```text
Проверено: YYYY-MM-DD
```

При существенной правке дата меняется только после повторной проверки ключевых claims по current Odoo 19.0 documentation.

## Coverage

Источниковая полнота отслеживается не количеством ссылок в уроке, а [coverage-map.md](coverage-map.md), который сопоставляет major official documentation surfaces с owner-уроками и статусами.

## Исходный код Odoo

Official Odoo source code **не используется как основной источник фактов** при current policy.

Любое изменение этого правила должно сначала изменить governance, source policy и scope evidence в edition ledger.