# Odoo 19 Community: архитектура и идеология

Последовательный учебный курс по **Odoo 19.0 Community self-hosted**.

Цель — досконально понять Odoo как платформу и ERP-систему: от runtime/module/ORM architecture до штатных business models и end-to-end процессов.

## Governance

Перед уроками действуют обязательные controls:

1. только official Odoo 19.0 documentation;
2. stable lesson IDs и явные prerequisites;
3. один canonical owner concept + явные aspect owners;
4. coverage map против пропусков;
5. Community/Enterprise status не угадывается;
6. новый уровень не использует необъяснённые prerequisites.

Документы:

- [Метод курса](00-governance/method.md)
- [Политика источников](00-governance/source-policy.md)
- [Владение понятиями](00-governance/concept-ownership.md)
- [Coverage map](00-governance/coverage-map.md)
- [Реестр редакций](00-governance/edition-ledger.md)
- [Глоссарий](00-governance/glossary.md)

## Scope

Baseline:

```text
Odoo 19.0
Community
self-hosted
```

Не переносятся автоматически в baseline:

- Enterprise-only behavior;
- Odoo Online-specific behavior;
- Odoo.sh-specific behavior;
- `saas-19.x` documentation.

Current docs-only policy позволяет глубоко изучать documented semantics, но **не гарантирует exhaustive Community addon whitelist**. Для полного technical inventory может потребоваться отдельное разрешение official source/manifests через governance change.

## Маркировка

- **[ODOO][S#]** — claim подтверждается указанным official source урока;
- **[ВЫВОД]** — наше логическое следствие documented mechanisms;
- **[ERP-КЛАССИФИКАЦИЯ]** — внешний business/ERP term.

## Текущие уроки

### Architecture

- [`ARCH-01` Архитектурный фундамент](01-architecture/01-architecture-foundations.md)
- [`ARCH-02` Модульная система](01-architecture/02-module-system.md)
- [`ARCH-03` Python package и import chain](01-architecture/03-module-loading.md)
- [`ARCH-04` Request / RPC execution boundary](01-architecture/04-request-rpc-boundary.md)

### ORM

- [`ORM-01` ORM Core](02-orm/01-orm-core.md)
- [`ORM-02` Per-database model registry и composition](02-orm/02-model-registry-composition.md)

## Course DAG

```text
ARCH-01
   │
   ├──► ARCH-02 ─► ARCH-03 ─► ORM-01 ─► ORM-02 ─► ORM-03 Fields
   │                                      │
   └──► ARCH-04                           ├──► ORM-04 Relations
            │                             └──► ...
            ├──► ORM-06 Transactions
            ├──► SEC-01 Security
            └──► UI-03 Onchange (after UI-02 Views)
```

Full planned sequence находится в `00-governance/method.md`.

## Архитектура каталогов

```text
00-governance/

01-architecture/
    ARCH-01 foundation
    ARCH-02 module system
    ARCH-03 Python import chain
    ARCH-04 request/RPC boundary

02-orm/
    ORM-01 Core
    ORM-02 registry/composition
    ORM-03 Fields
    ORM-04 Relations
    ORM-05 Derived fields / constraints
    ORM-06 Transactions

03-data-security-ui/
    DATA-01 Module data
    SEC-01 Security
    UI-01 Actions and menus
    UI-02 Views
    UI-03 Onchange

04-extension/
    EXT-01 Model inheritance
    EXT-02 View inheritance
    EXT-03 Mixins
    EXT-04 Multi-company

05-runtime/
    RUN-01 Advanced ORM
    RUN-02 HTTP / controllers
    RUN-03 Web client / Owl / assets
    RUN-04 Testing
    RUN-05 Upgrades / migrations
    RUN-06 Deployment / workers

10-business-model/
11-domain-apps/
12-end-to-end/
```

## Критические терминологические границы

### Odoo module master data ≠ ERP master data

Первое — official module-data terminology; второе — future business classification курса.

### App ≠ functional area

App — user-facing module. Functional area может состоять из нескольких modules.

### database-bound addon ≠ любой runtime module

Большинство installed addons bound to a specific database, но CLI Odoo также имеет server-wide modules через `--load`.

### Odoo server/runtime ≠ один process

Process architecture и database context не смешиваются.

### user/company security-context ≠ data-model semantics

`user` как security principal и `res.users` как model — разные aspects. То же относится к company context и `res.company`.

## Официальные отправные точки

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- Server Framework 101: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- CLI: https://www.odoo.com/documentation/19.0/developer/reference/cli.html
- Security: https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html
- Coding Guidelines: https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html