# 01. Архитектурный фундамент Odoo

> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: governance  
> Владеет понятиями: three-tier architecture; deployment/runtime/process distinction; multi-database property; data-driven architecture  
> Preview: module, App, model, record, UI resources  
> Отложено: module lifecycle, imports, ORM details, workers, security, business models  
> Edition scope: платформенная архитектура; Community/Enterprise boundary только на общем уровне

## Цель

Построить первую непротиворечивую mental model Odoo, не превращая верхнеуровневую схему в выдуманную internal implementation diagram.

---

## 1. Odoo — three-tier application

**[ODOO]** Architecture Overview описывает Odoo как three-tier application:

```text
Presentation tier
HTML5 / JavaScript / CSS
        │
        ▼
Logic tier
Python / Odoo server
        │
        ▼
Data tier
PostgreSQL
```

**[ВЫВОД]** UI, server-side logic и persistence — разные архитектурные уровни. Экран Odoo нельзя считать самой business model.

---

## 2. Server/runtime нельзя приравнивать к одному process

На раннем уровне удобно говорить «Odoo server», но нельзя закреплять ошибочную формулу:

```text
Odoo server = один Python process
```

**[ODOO]** CLI Odoo поддерживает multiprocessing mode: при `--workers > 0` запускаются несколько HTTP worker subprocesses. Cron workers/threads также зависят от server mode.

**[ВЫВОД]** Для курса различаем:

```text
Odoo deployment / server runtime
        │
        ├── может работать в single-process mode
        └── может включать несколько worker processes
```

Exact worker/runtime architecture принадлежит будущему deployment-уроку.

---

## 3. Один Python process может обслуживать несколько databases

**[ODOO]** Server Framework 101 отдельно предупреждает: один Odoo instance/process может обслуживать несколько databases, причём на них могут быть установлены разные modules.

```text
один Python process
        │
        ├── Database A → module set A
        ├── Database B → module set B
        └── Database C → module set C
```

Это утверждение **не означает**, что production deployment всегда состоит из одного process. Оно описывает multi-database property runtime.

**[ВЫВОД]** Database context и OS/Python process — разные измерения архитектуры.

Из этого позже станет понятно, почему installed-module-dependent mutable globals являются плохой моделью: один process не обязательно соответствует одной database configuration.

---

## 4. Функциональность Odoo композиционно поставляется modules

На этом уроке понятие только previewed; owner — следующий урок.

**[ODOO]** Server/client extensions Odoo packaged as modules (addons), которые могут загружаться в database.

**[ВЫВОД]** Пользовательская функциональность Odoo не должна мыслиться как набор полностью независимых монолитных приложений. Она собирается поверх общей платформы из modules и их dependencies/extensions.

Подробная module semantics будет в `02-module-system.md`.

---

## 5. Database определяет installed configuration

**[ODOO]** Набор доступных ORM models конкретной database зависит от modules, установленных именно в ней.

Минимальная схема:

```text
server has addon code
        │
        ▼
Database X has installed module set
        │
        ▼
models available in Database X
```

**[ВЫВОД]** Нельзя корректно утверждать «Odoo содержит model X» без уточнения уровня:

- addon code существует;
- module доступен server;
- module установлен в database;
- model реально доступна в runtime этой database.

Module discovery/loading будут owner-уроками 02–03.

---

## 6. Odoo в значительной степени data-driven

**[ODOO]** Developer documentation прямо характеризует Odoo как greatly/highly data-driven. Module data records используются не только для business data.

В частности, через records/data определяются или хранятся такие механизмы, как:

```text
views
menus/actions
security/configuration data
reports and other system resources
business / technical records
```

**[ODOO]** Views сами хранятся как records и могут редактироваться отдельно от моделей, которые они представляют.

**[ВЫВОД]** В Odoo граница «код против данных» не совпадает с простой схемой «Python = логика, PostgreSQL = только пользовательские строки». Значительная часть описания самой системы является data records.

Подробные owners:

- module data → `03-data-security-ui/01-module-data.md`;
- security → отдельный security lesson;
- actions/menus/views → UI lessons.

---

## 7. App, module, model и record — разные уровни

Здесь только preview; definitions принадлежат следующим owner-урокам.

```text
MODULE
= technical extension/package unit

APP
= user-facing application module

MODEL
= ORM-level object definition/runtime model

RECORD
= concrete ORM data record
```

**[ВЫВОД]** Нельзя строить mental model:

```text
1 App = 1 module = 1 model = 1 table = 1 screen
```

Odoo не устроена так.

---

## 8. UI resources не являются business model

На архитектурном уровне достаточно схемы:

```text
records/models
      │
      ▼
actions / views / menus
      │
      ▼
web client
      │
      ▼
user
```

**[ВЫВОД]** Отдельный menu item или отдельный экран не доказывает существование отдельной независимой предметной сущности.

---

## 9. Community и Enterprise: только общая граница

**[ODOO]** Odoo имеет Community и Enterprise editions.

**[ODOO]** Официальная архитектурная/source-install documentation описывает Enterprise functionality как дополнительные modules/addons поверх Community server/modules.

```text
Community server + Community addons
                +
Enterprise additional addons
```

**[ВЫВОД]** Edition boundary в значительной степени выражается составом доступных modules, но из этого **не следует**, что любая страница общей документации доступна в Community.

Конкретный edition status фиксируется только в governance edition ledger.

---

## 10. Минимальная модель после урока

```text
                     ODOO DEPLOYMENT / RUNTIME
                              │
                 ┌────────────┴────────────┐
                 │                         │
        one or more processes      addon code available
                 │                         │
                 └────────────┬────────────┘
                              │
                       DATABASE CONTEXT
                              │
                       installed modules
                              │
                       available models
                              │
                      records / behavior
                              │
                  data-driven UI/security
                              │
                          web client
```

Эта схема концептуальная. Она не является UML internal classes и не описывает exact worker/registry implementation.

---

## 11. Что нельзя заключать

- `Odoo server = один Python process` — нет;
- `one database = one process` — нет;
- `addon code on disk = installed functionality` — нет;
- `App = module = model` — нет;
- `menu/view = business object` — нет;
- `Community/Enterprise status определяется наличием страницы в docs` — нет.

---

## Контрольные вопросы

1. Какие three tiers официально выделяет Odoo?
2. Почему server runtime нельзя автоматически приравнять к одному process?
3. В чём состоит documented multi-database property одного process?
4. Почему database context и process architecture — разные понятия?
5. Что означает data-driven характер Odoo?
6. Почему UI resource не равен business model?
7. Почему наличие addon code не доказывает model availability в конкретной database?

## Официальные источники

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
2. Models and Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
3. Data Files / Define module data  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/data.html  
   https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html
4. View records  
   https://www.odoo.com/documentation/19.0/developer/reference/user_interface/view_records.html
5. CLI / multiprocessing  
   https://www.odoo.com/documentation/19.0/developer/reference/cli.html