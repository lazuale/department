# Реестр редакций Odoo 19.0

Этот файл разделяет два разных типа evidence:

1. **platform edition facts** — общая архитектурная граница Community/Enterprise;
2. **feature/module availability** — entitlement конкретного module/feature.

Они не смешиваются.

## 1. Platform edition facts

| Утверждение | Official evidence | Проверено |
|---|---|---|
| Odoo имеет Community и Enterprise editions | Architecture Overview / Licenses | 2026-08-13 |
| Community является open-source основой | Architecture Overview / Licenses | 2026-08-13 |
| Enterprise technical functionality поставляется additional modules поверх Community modules/server | Architecture Overview / Source install | 2026-08-13 |
| Current course baseline: Odoo 19.0 Community self-hosted | Governance scope, не entitlement claim | 2026-08-13 |

Эта таблица **не доказывает** edition status конкретного application/feature.

## 2. Feature / module availability

Допустимые статусы только такие:

- **Community подтверждено**;
- **Enterprise подтверждено**;
- **Редакция не установлена**.

| Feature / module | Статус | Official evidence | Проверено |
|---|---|---|---|
| _пока нет предметных entries_ | — | — | — |

Новая строка появляется только когда конкретный module/feature реально встречается в курсе и есть official edition evidence.

## 3. Что не является edition entry

Не добавляются в availability table:

- custom-module examples из tutorials;
- governance rules;
- общие platform architecture facts;
- наличие documentation page без edition evidence.

Official tutorial custom addon — учебный пример architecture/framework, а не штатный baseline module и не объект entitlement classification.

## 4. Docs-only limitation

При current source policy official documentation не гарантирует exhaustive Community addon inventory.

Поэтому отсутствие module в этой таблице означает только:

> edition status ещё не зафиксирован курсом.

Это **не означает**, что module отсутствует в Community.

## Запрещённый вывод

```text
страница есть в Odoo 19 Documentation
        ≠
feature подтверждена как Community
```

Если documentation не даёт достаточного edition evidence, статус остаётся **«Редакция не установлена»**.