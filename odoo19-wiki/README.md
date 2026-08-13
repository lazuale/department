# Odoo 19 Community: архитектура и идеология

Последовательный учебный курс по **Odoo 19.0 Community**.

Цель — досконально понять Odoo как платформу и ERP-систему: от runtime/module/ORM architecture до штатных business models и end-to-end процессов.

## Governance

Перед уроками действуют четыре обязательных правила:

1. только официальные материалы Odoo 19.0;
2. один owner для каждого фундаментального понятия;
3. Community/Enterprise status не угадывается;
4. новый уровень не использует необъяснённые prerequisites.

Документы governance:

- [Метод курса](00-governance/method.md)
- [Политика источников](00-governance/source-policy.md)
- [Владение понятиями](00-governance/concept-ownership.md)
- [Реестр редакций](00-governance/edition-ledger.md)
- [Глоссарий](00-governance/glossary.md)

## Маркировка утверждений

- **[ODOO]** — непосредственно подтверждено official Odoo 19.0 documentation;
- **[ВЫВОД]** — логическое следствие документированных механизмов;
- **[ERP-КЛАССИФИКАЦИЯ]** — внешний business/ERP term, а не внутренний platform/ORM type Odoo.

## Текущие уроки

### 01. Architecture

- [01. Архитектурный фундамент](01-architecture/01-architecture-foundations.md)
- [02. Модульная система](01-architecture/02-module-system.md)
- [03. Загрузка модулей и per-database models](01-architecture/03-module-loading.md)

### 02. ORM

- [01. ORM Core](02-orm/01-orm-core.md)

## Согласованная архитектура курса

```text
00-governance/

01-architecture/
    architecture foundation
    module system
    module loading

02-orm/
    ORM Core
    Fields
    Relations
    Computed / related / inverse / constraints
    Transactions

03-data-security-ui/
    Module data / external IDs / noupdate
    Security
    Actions and menus
    Views
    Onchange

04-extension/
    Model inheritance
    View inheritance
    Mixins
    Multi-company

05-runtime/
    Advanced ORM
    HTTP / RPC
    Web client / Owl / assets
    Testing
    Upgrades / migrations
    Deployment / workers

10-business-model/
11-domain-apps/
12-end-to-end/
```

Эта структура разделяет платформенные понятия по владельцам и не позволяет одному каталогу `platform` превратиться в свалку всех последующих тем.

## Важные терминологические границы

### Odoo module master data ≠ ERP master data

Официальная Odoo documentation использует **Master Data** для data, устанавливаемых с module и необходимых для его работы, включая technical data вроде views/actions.

Будущая **ERP master data** — отдельная business classification курса. Эти понятия всегда квалифицируются.

### App ≠ functional area

App — user-facing Odoo module. Functional area — аналитическая business category и может складываться из нескольких modules.

### Odoo server/runtime ≠ один process

Odoo может работать в multiprocessing mode. При этом official tutorial отдельно показывает multi-database property одного Python process. Process architecture и database context не смешиваются.

## Community scope

Official custom-module examples используются для изучения architecture/framework, но **не считаются частью канонического Community ERP baseline**.

Concrete module/feature попадает в Community-карту только после отдельного official edition evidence.

## Официальные отправные точки

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- Server Framework 101: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- Define module data: https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html
- CLI: https://www.odoo.com/documentation/19.0/developer/reference/cli.html
- Coding Guidelines: https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html