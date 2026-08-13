# Odoo 19 Community: архитектура и идеология

Последовательный учебный курс по **Odoo 19.0 Community self-hosted**.

Цель — досконально понять Odoo как платформу и ERP-систему: от runtime/module/ORM architecture до штатных business models и end-to-end процессов.

## Baseline

```text
Baseline ID: B1
Зафиксирован: 2026-08-13
Status: frozen foundation
```

Baseline B1 фиксирует architecture курса, governance, prerequisite DAG и ownership. Он не перестраивается при каждом новом lesson; основания для изменения перечислены в [baseline.md](00-governance/baseline.md).

## Governance

Перед lessons действуют обязательные controls:

1. только official Odoo 19.0 documentation;
2. stable lesson IDs;
3. один canonical prerequisite graph;
4. один canonical owner concept + явные aspect owners;
5. coverage map против пропусков;
6. Community/Enterprise status не угадывается;
7. каждый `[ODOO]` claim имеет однозначный `S#`, причём `1 S# = 1 official page`.

Документы:

- [Метод курса](00-governance/method.md)
- [Канонический prerequisite DAG](00-governance/course-dag.md)
- [Политика источников](00-governance/source-policy.md)
- [Владение понятиями](00-governance/concept-ownership.md)
- [Coverage map](00-governance/coverage-map.md)
- [Реестр редакций](00-governance/edition-ledger.md)
- [Глоссарий](00-governance/glossary.md)
- [Baseline B1](00-governance/baseline.md)

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

- **[ODOO][S#]** — claim подтверждается указанной одной official documentation page;
- **[ВЫВОД]** — логическое следствие documented mechanisms;
- **[ERP-КЛАССИФИКАЦИЯ]** — внешний business/ERP term.

## Текущие owner-lessons

### Architecture

- [`ARCH-01` Архитектурный фундамент](01-architecture/01-architecture-foundations.md)
- [`ARCH-02` Модульная система](01-architecture/02-module-system.md)
- [`ARCH-03` Python package и import chain](01-architecture/03-module-loading.md)
- [`ARCH-04` Request / RPC execution boundary](01-architecture/04-request-rpc-boundary.md)

### ORM

- [`ORM-01` ORM Core](02-orm/01-orm-core.md)
- [`ORM-02` Per-database model registry и composition](02-orm/02-model-registry-composition.md)

Следующий planned owner: `ORM-03` — Model metadata / SQL storage / schema declarations.

## Prerequisite graph

**Единственный нормативный graph находится в [`00-governance/course-dag.md`](00-governance/course-dag.md).**

README не определяет собственные зависимости между lessons.

Текущий основной путь после существующих owners:

```text
ORM-01
  ↓
ORM-02
  ↓
ORM-03 Model metadata / schema
  ↓
ORM-04 Fields
  ↓
ORM-05 Relations
  ↓
ORM-06 Derived fields / Python constraints
```

`ORM-07 Transactions` имеет отдельные direct prerequisites, зафиксированные только в canonical DAG.

## Архитектура каталогов

```text
00-governance/
    method
    course DAG
    source policy
    concept ownership
    coverage map
    edition ledger
    glossary
    baseline

01-architecture/
    ARCH-01 foundation
    ARCH-02 module system
    ARCH-03 Python import chain
    ARCH-04 request/RPC boundary

02-orm/
    ORM-01 Core
    ORM-02 registry/composition
    ORM-03 Model metadata / SQL storage / schema declarations
    ORM-04 Fields
    ORM-05 Relations
    ORM-06 Derived fields / Python constraints
    ORM-07 Transactions

03-data-security-ui/
    DATA-01 Module data
    SEC-01 Security
    UI-01 Actions and menus
    UI-02 Views
    UI-03 Onchange
    UI-04 QWeb reports / report actions

04-extension/
    EXT-01 Model inheritance
    EXT-02 View inheritance
    EXT-03 Mixins
    EXT-04 Multi-company

05-runtime/
    RUN-01 Advanced ORM
    RUN-02 HTTP / controllers
    RUN-03 Web client / Owl / assets / frontend QWeb
    RUN-04 Testing
    RUN-05 Upgrades / migrations
    RUN-06 Deployment / workers / --load runtime mechanics

10-business-model/
11-domain-apps/
12-end-to-end/
```

## Критические терминологические границы

### Odoo module master data ≠ ERP master data

Первое — official module-data terminology; второе — future business classification курса.

### App ≠ functional area

App — user-facing module. Functional area может состоять из нескольких modules.

### Database installation ≠ server-wide loading

Это **две независимые axes**, а не два непересекающихся класса modules:

```text
module
├── database aspect: installed / not installed in DB
└── runtime aspect: may / may not participate in --load
```

### Multi-database ≠ multi-company

```text
multi-database → несколько PostgreSQL database contexts
multi-company  → company semantics внутри Odoo database/environment
```

### Odoo server/runtime ≠ один process

Process architecture и database context не смешиваются.

### user/company security-context ≠ data-model semantics

`user` как security principal и `res.users` как model — разные aspects. То же относится к company context и `res.company`.

### RPC transaction claim ≠ generic HTTP transaction claim

Current architecture lesson фиксирует только прямо документированную framework-managed transaction semantics RPC calls. Generic HTTP/controller aspect принадлежит future `RUN-02` и должен подтверждаться отдельно.

## Официальные отправные точки

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- Server Framework 101: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- CLI: https://www.odoo.com/documentation/19.0/developer/reference/cli.html
- Security: https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html
- Coding Guidelines: https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html