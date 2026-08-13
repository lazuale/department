# ARCH-01. Архитектурный фундамент Odoo

> Lesson ID: `ARCH-01`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `GOV`  
> Canonical owner: three-tier architecture; deployment/runtime/process distinction; multi-database property; data-driven architecture  
> Aspect owner: workers → `RUN-06`; per-database models → `ORM-02`  
> Preview: module, App, model, record, UI resources  
> Отложено: module lifecycle, imports, ORM details, workers, security, business models  
> Edition scope: Odoo 19.0 Community self-hosted platform baseline; concrete feature entitlement не определяется здесь  
> Sources: `S1`–`S6`

## Цель

Построить первую непротиворечивую mental model Odoo, не превращая overview в выдуманную internal implementation diagram.

---

## 1. Odoo — three-tier application

**[ODOO][S1]** Architecture Overview описывает Odoo как three-tier application:

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

**[ВЫВОД]** UI, server-side logic и persistence — разные architectural layers. Экран Odoo нельзя считать самой business model.

---

## 2. Server/runtime нельзя приравнивать к одному process

Нельзя закреплять формулу:

```text
Odoo server = один Python process
```

**[ODOO][S6]** CLI поддерживает multiprocessing mode: при `--workers > 0` используются worker subprocesses; runtime behavior зависит от server mode.

Минимально:

```text
Odoo deployment / runtime
        │
        ├── single-process mode
        └── multiprocessing / workers
```

Exact worker architecture принадлежит `RUN-06`.

---

## 3. Один Python process может обслуживать несколько databases

**[ODOO][S2]** Server Framework 101 предупреждает, что один Odoo process может обслуживать несколько databases, на которых установлены разные modules.

```text
one Python process
      │
      ├── Database A → module set A
      ├── Database B → module set B
      └── Database C → module set C
```

Это не означает, что production deployment всегда состоит из одного process.

**[ВЫВОД]** Database context и OS/Python process — разные dimensions architecture.

---

## 4. Функциональность Odoo композиционно поставляется modules

Понятие previewed; canonical owner — `ARCH-02`.

**[ODOO][S1]** Server/client extensions packaged as modules/addons, которые могут загружаться в Odoo database/runtime context.

**[ВЫВОД]** User-facing functionality нельзя мыслить как набор полностью independent monolithic apps; она собирается поверх общей platform/module system.

---

## 5. Database имеет собственную installed configuration

**[ODOO][S2]** Different databases одного runtime могут иметь different installed module sets.

Per-database ORM model mapping будет определён только после ORM Core в `ORM-02`.

На текущем уровне достаточно:

```text
server has addon code
        │
        ▼
Database X has installed configuration
        │
        ▼
runtime behavior of Database X differs from Database Y
```

**[ВЫВОД]** Нельзя выводить database functionality только из files, лежащих на server.

---

## 6. Odoo в значительной степени data-driven

**[ODOO][S3][S5]** Developer documentation описывает Odoo как greatly/highly data-driven. Records/data используются не только для ordinary business rows: views, actions/menus, security/configuration и другие system resources во многом представлены data records.

**[ODOO][S4]** Views themselves хранятся как records.

```text
Odoo data records
├── business/configuration data
├── views
├── actions/menus
├── security-related data
└── other system resources
```

**[ВЫВОД]** Граница `Python code` / `database data` не совпадает с простой схемой «code = system, rows = user data».

Detailed owners появятся позже.

---

## 7. App, module, model и record — разные levels

Здесь только preview:

```text
MODULE → technical extension/package unit
APP    → user-facing application module
MODEL  → ORM concept
RECORD → concrete ORM data record
```

**[ВЫВОД]** Mental model `1 App = 1 module = 1 model = 1 table = 1 screen` неверна.

Canonical owners: `ARCH-02`, `ORM-01`.

---

## 8. UI resources не являются business model

На архитектурном уровне:

```text
models / records
      │
      ▼
actions / views / menus
      │
      ▼
web client
```

**[ВЫВОД]** Separate menu or screen не доказывает existence отдельной independent domain entity.

---

## 9. Community и Enterprise: только общая boundary

**[ODOO][S1]** Odoo имеет Community и Enterprise editions; technical Enterprise functionality описывается как additional modules поверх Community modules/server.

**[ВЫВОД]** Общая edition boundary модульна, но concrete module/feature entitlement определяется только edition ledger.

---

## Минимальная модель

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
                    installed configuration
                              │
                 ORM/data/UI layers later
```

Эта схема conceptual, не UML internal implementation.

## Что нельзя заключать

- Odoo server = one process — нет;
- one database = one process — нет;
- addon code on disk = installed functionality — нет;
- App = module = model — нет;
- menu/view = business object — нет;
- documentation page = Community entitlement — нет;
- ARCH-01 уже определяет backend model registry — нет.

## Контрольные вопросы

1. Какие three tiers официально выделяет Odoo?
2. Почему runtime нельзя автоматически приравнять к одному process?
3. Что означает multi-database property одного process?
4. Почему database context и process architecture различаются?
5. Что означает data-driven характер Odoo?
6. Почему UI resource не равен business model?

## Официальные источники

- `S1` — Architecture Overview  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- `S2` — Models and Basic Fields  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
- `S3` — Data Files  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/data.html
- `S4` — View Records  
  https://www.odoo.com/documentation/19.0/developer/reference/user_interface/view_records.html
- `S5` — Define module data  
  https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html
- `S6` — Command-line interface  
  https://www.odoo.com/documentation/19.0/developer/reference/cli.html