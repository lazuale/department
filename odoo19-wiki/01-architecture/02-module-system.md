# ARCH-02. Модульная система Odoo

> Lesson ID: `ARCH-02`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ARCH-01`  
> Canonical owner: module/addon; App; `addons_path`; manifest; dependencies; `auto_install`; module lifecycle; server-wide module distinction  
> Aspect owner: runtime details `--load` → `RUN-06`  
> Preview: Python import chain; module data  
> Отложено: imports; external IDs; inheritance; migrations; exact worker/runtime behavior  
> Edition scope: platform semantics; concrete module availability фиксируется отдельно  
> Sources: `S1`–`S5`

## Цель

Понять module как техническую единицу композиции Odoo и не смешивать:

```text
addon code доступен server
        ≠
database-bound module установлен в Database X
        ≠
server-wide module загружен runtime через --load
```

---

## 1. Module и addon

**[ODOO][S1]** Server/client extensions Odoo packaged as modules; в документации также используется термин addon.

Module может содержать business objects, views, data, controllers, static web data или только часть этих элементов.

**[ВЫВОД]** Module не обязан создавать отдельную model, table, menu или business domain.

---

## 2. `addons_path`

**[ODOO][S5]** `--addons-path` задаёт directories, которые Odoo scans for modules.

```text
Odoo runtime
   │
   └── addons_path
        ├── dir A
        │    ├── module_x/
        │    └── module_y/
        └── dir B
             └── module_z/
```

**[ВЫВОД]** Наличие directory где-то на filesystem не означает, что Odoo рассматривает его как addon: его parent directory должен находиться в configured addon search path.

---

## 3. Minimal addon skeleton

**[ODOO][S2]** Current Server Framework 101 вводит минимальный addon shell с двумя files:

```text
module/
├── __init__.py
└── __manifest__.py
```

`__init__.py` может быть пустым. `__manifest__.py` описывает module и не является пустым.

**[ВЫВОД]** Для classic addon в `addons_path` package/module skeleton существует до появления business objects.

Python import role `__init__.py` принадлежит `ARCH-03`.

---

## 4. Manifest

**[ODOO][S3]** `__manifest__.py` содержит Python dictionary с metadata/properties module.

Для архитектуры курса важны:

- `name` — human-readable name;
- `version` — module version;
- `depends` — module dependencies;
- `data` — files, загружаемые при install/update;
- `demo` — demonstration data;
- `application` — признак полноценного application;
- `auto_install` — conditional automatic installation;
- `external_dependencies` — external Python/binary requirements.

**[ВЫВОД]** Manifest — декларативный контракт module system, а не только карточка Apps UI.

---

## 5. App — специальный user-facing module

**[ODOO][S1][S3]** Main user-facing modules выделяются как Apps; manifest `application` указывает, следует ли считать module полноценным application. Большинство modules не обязаны быть Apps.

```text
Odoo module
├── application = True  → App
└── application = False → supporting/technical module
```

**[ВЫВОД]** `App` не является синонимом arbitrary functional area.

---

## 6. Apps dashboard — UI, а не dependency graph

**[ODOO][S4]** Apps UI позволяет устанавливать/управлять apps/modules. Default Apps filter можно менять, чтобы искать extra/technical modules; Update Apps List используется для обновления списка обнаруживаемых addons.

**[ВЫВОД]** Раскладка карточек в Apps dashboard не является model/module architecture.

---

## 7. `depends` формирует graph

**[ODOO][S3]** Dependencies загружаются до dependent module; при installation dependent module требуемые dependencies также устанавливаются.

```text
          base
         /    \
        ▼      ▼
   module_a  module_b
        \      /
         ▼    ▼
        module_c
```

**[ВЫВОД]** Database-bound installed configuration формируется dependency graph modules, а не плоским списком Apps.

---

## 8. Dependency — не Python import

**[ODOO][S3]** Dependency нужна и когда module использует capabilities другого module, и когда изменяет resources, которые тот определяет.

```text
module A defines X
        │
module B depends on A
        └── changes / extends X
```

Python imports внутри одного addon package — другой mechanism и owner `ARCH-03`.

---

## 9. `base`

**[ODOO][S3]** `base` всегда установлен в Odoo instance/database context согласно manifest reference.

**[ВЫВОД]** `base` — platform foundation module, но не business parent всех records.

---

## 10. `auto_install` и link modules

**[ODOO][S3]** Manifest reference приводит link module как типичный use case `auto_install`: отдельный module интегрирует иначе независимые modules и устанавливается при выполнении dependency conditions.

```text
Module A       Module B
    \           /
     \         /
      Link module
```

**[ВЫВОД]** Cross-module integration может поставляться отдельным addon. Это documented pattern, но не единственный универсальный pattern Odoo.

---

## 11. Module data — только preview

**[ODOO][S3]** Manifest `data` и `demo` перечисляют data files module.

В документации термин **master data** имеет отдельный Odoo module-data смысл. Полное определение принадлежит `DATA-01`.

> Здесь достаточно помнить: `Odoo module master data` не следует автоматически читать как `ERP master data`.

Это не входит в контрольные вопросы ARCH-02.

---

## 12. Основная database-bound lifecycle model

Для большинства addons полезна модель:

```text
addon code находится в searchable addons_path
        │
        ▼
Odoo может обнаружить/показать module
        │
        ▼
module installed in Database X
```

**[ODOO][S4][S5]** Apps List обновляет обнаруживаемые modules; install/update operations работают с выбранной database.

**[ВЫВОД]** Не вводим отдельную фундаментальную сущность между «Odoo обнаружила addon» и «module доступен к install», если documentation не требует такого concept.

---

## 13. Install / upgrade / uninstall

**[ODOO][S4][S5]** Odoo поддерживает install, upgrade и uninstall module operations.

- install вводит database-bound module и dependencies в installed configuration;
- upgrade применяет обновлённую module/data version;
- uninstall удаляет module из installed configuration и может затрагивать связанные records/dependencies.

**[ВЫВОД]** Lifecycle — изменение database configuration/data, а не UI toggle.

---

## 14. Важное исключение: server-wide modules

**[ODOO][S5]** CLI option `--load` задаёт **server-wide modules**. Documentation прямо говорит, что они предоставляют features, не обязательно связанные с конкретной database, в отличие от modules, которые устанавливаются и привязаны к specific database; последние составляют большинство Odoo addons. Default `--load` — `base,web`.

Поэтому нельзя превращать правило:

```text
module → installed in database
```

в универсальное описание всех runtime modules.

Минимально:

```text
majority of addons
→ database-bound when installed

server-wide modules
→ runtime-level loading via --load
```

Exact runtime implications принадлежат `RUN-06`.

---

## 15. Community / Enterprise на module level

**[ODOO][S1]** Enterprise technical functionality документируется как additional modules поверх Community modules/server.

**[ВЫВОД]** Общая edition architecture модульна, но status конкретного module/feature определяется только edition ledger.

---

## Минимальная модель

```text
filesystem
   │
addons_path
   │
discoverable addon
   │
__manifest__.py
   │
module dependencies
   │
   ├── majority → installed per database
   └── special server-wide modules → --load runtime path
```

## Что нельзя заключать

- addon on filesystem = installed module — нет;
- App = любой module — нет;
- Apps dashboard = complete architecture — нет;
- dependency = Python import — нет;
- каждый module обязательно database-bound — нет;
- `master data` всегда означает ERP master data — нет;
- Enterprise = independent server engine — нет.

## Контрольные вопросы

1. Что такое module/addon?
2. Что делает `addons_path`?
3. Какие два files current tutorial требует для минимального addon shell?
4. Какую роль выполняет manifest?
5. Чем App отличается от module?
6. Почему dependencies образуют graph?
7. Что такое link module?
8. Как выглядит упрощённый database-bound lifecycle?
9. Чем server-wide modules через `--load` отличаются от большинства installed addons?

## Официальные источники

- `S1` — Architecture Overview  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- `S2` — A New Application  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/02_newapp.html
- `S3` — Module Manifests  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- `S4` — Apps and modules  
  https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html
- `S5` — Command-line interface  
  https://www.odoo.com/documentation/19.0/developer/reference/cli.html