# 02. Модульная система Odoo

> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: architectural foundation  
> Владеет понятиями: module/addon; App; `addons_path`; manifest; dependencies; `auto_install`; module lifecycle  
> Preview: Python import chain; module data  
> Отложено: import/loading mechanics; external IDs; inheritance; migrations  
> Edition scope: platform semantics; edition-specific modules фиксируются отдельно

## Цель

Понять module как техническую единицу композиции Odoo и развести четыре состояния:

```text
code exists
→ server can discover addon
→ module is available
→ module is installed in a particular database
```

---

## 1. Module и addon

**[ODOO]** В Odoo термины `module` и `addon` используются для одного архитектурного семейства: packaged server/client extension, направленного на определённую цель.

Module может содержать:

- business objects;
- views;
- data;
- controllers;
- static web data;
- или только часть этих элементов.

**[ODOO]** Ни один из перечисленных типов содержимого не является обязательным сам по себе.

**[ВЫВОД]** Module не обязан создавать отдельную model, table, menu или business domain.

---

## 2. `addons_path`

**[ODOO]** Odoo ищет modules/addons в каталогах `addons_path`.

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

**[ODOO]** Additional addon directories можно подключать через server configuration / CLI `--addons-path`.

**[ВЫВОД]** Physical directory на server и module availability для Odoo — разные уровни: parent directory должен входить в configured addon search path.

---

## 3. Odoo module является Python package с manifest

**[ODOO]** Server Framework 101 создаёт addon как Python package с как минимум:

```text
module/
├── __init__.py
└── __manifest__.py
```

`__init__.py` может быть пустым, если Python files ещё не подключены.

**[ODOO]** Manifest `__manifest__.py` содержит Python dictionary и объявляет metadata/properties module.

**[ВЫВОД]** Нельзя формулировать «module становится Python package только если в нём есть business objects». Package structure существует с самого addon/module skeleton.

Import role `__init__.py` — owner следующего урока.

---

## 4. Manifest — декларативный контракт module

Для текущей архитектуры важны:

### `name`
Human-readable name.

### `version`
Module version.

### `depends`
Modules, которые должны быть загружены раньше текущего, потому что current module использует их capabilities или изменяет определённые ими resources.

### `data`
Files module data, загружаемые при install/update.

### `demo`
Demo data files.

### `application`
Признак полноценного user-facing application.

### `auto_install`
Условие автоматической installation module при выполнении dependency conditions.

### `external_dependencies`
Внешние Python/binary dependencies.

**[ВЫВОД]** Manifest — не просто карточка Apps UI. Он участвует в module dependency/data/lifecycle semantics.

---

## 5. App — специальный user-facing module

**[ODOO]** Architecture Overview говорит: main user-facing modules выделяются как Apps, но большинство modules не являются Apps.

**[ODOO]** Manifest field `application` указывает, следует ли считать module полноценным application.

```text
Odoo module
├── application = True  → App
└── application = False → supporting/technical module
```

**[ВЫВОД]** `App` не используется в курсе как синоним произвольной functional area. Functional area может включать несколько modules, включая supporting/link modules.

---

## 6. Apps dashboard — UI управления apps/modules, не модель архитектуры

**[ODOO]** User Documentation показывает Apps dashboard как интерфейс установки и управления apps/modules. По умолчанию используется Apps filter; technical/extra modules можно находить после изменения фильтрации.

**[ВЫВОД]** Dashboard не следует описывать как «полный dependency graph» или как «экран только Apps». Это user-facing management UI над частью module system.

Архитектуру module graph определяет manifest dependency semantics, не раскладка карточек в UI.

---

## 7. Dependencies образуют graph

```python
'depends': ['base', 'module_a', 'module_b']
```

**[ODOO]** Dependencies загружаются раньше dependent module; при installation требуемые dependencies также устанавливаются.

```text
          base
         /    \
        ▼      ▼
   module_a  module_b
        \      /
         ▼    ▼
        module_c
```

**[ВЫВОД]** Database configuration формируется dependency graph modules, а не плоским списком Apps.

---

## 8. Dependency — не только Python import

**[ODOO]** `depends` нужен и когда current module использует capabilities другого, и когда изменяет resources, определённые другим module.

```text
module A defines resource X
          │
module B depends on A
          │
          └── changes / extends X
```

**[ВЫВОД]** Dependency задаёт межмодульный архитектурный порядок, а Python imports внутри package — другой механизм.

---

## 9. `base`

**[ODOO]** `base` является обязательным установленным module Odoo.

**[ВЫВОД]** Это системная module foundation, но не универсальный business parent всех records и не предметный корень ERP.

---

## 10. `auto_install` и link modules

**[ODOO]** Manifest reference прямо приводит link module как типичный use case `auto_install`: module интегрирует два иначе независимых modules и устанавливается при наличии необходимых dependencies.

```text
Module A       Module B
    \           /
     \         /
      Link module
```

**[ВЫВОД]** Это официальный integration pattern Odoo: cross-module behavior может жить в отдельном link addon, не заставляя одну базовую сторону всегда зависеть от другой.

Не называем это единственным или универсальным «каноническим паттерном» всей Odoo.

---

## 11. `data` и `demo`: только preview

**[ODOO]** Data files перечисляются в manifest и загружаются module lifecycle.

**[ODOO]** Odoo официально называет содержимое `data` **master data** module и отличает его от demo data.

Важно не смешивать:

```text
Odoo module master data
= data, нужные module / устанавливаемые с ним
  включая technical data вроде views/actions

ERP master data
= будущая бизнес-классификация предметных данных
```

Полное определение Odoo module master data, external IDs и `noupdate` принадлежит уроку Module Data.

---

## 12. Availability и installed state

Различаем:

```text
1. addon code exists
2. parent directory is in addons_path
3. Odoo discovers/lists module
4. module is available for installation
5. module is installed in Database X
```

**[ODOO]** Для нового addon User Documentation предусматривает Update Apps List в developer mode.

**[ВЫВОД]** Available и installed — разные состояния.

---

## 13. Install / upgrade / uninstall

**[ODOO]** Odoo поддерживает отдельные lifecycle operations:

```text
install
upgrade
uninstall
```

- install вводит module/dependencies в installed configuration database;
- upgrade применяет обновлённую module/data version;
- uninstall удаляет module из installed configuration и может удалять связанные records/затрагивать dependencies.

**[ВЫВОД]** Module lifecycle — изменение database configuration/data, а не UI toggle.

---

## 14. Community / Enterprise на module level

**[ODOO]** Official architecture/source-install documentation описывает Enterprise repository как additional addons поверх Community server/modules.

**[ВЫВОД]** Edition boundary технически во многом определяется доступным module set.

Но status конкретного module/feature **не выводится автоматически** из этой общей архитектуры; он хранится в edition ledger.

---

## 15. Минимальная модель после урока

```text
filesystem
    │
addons_path
    │
available addon / Python package
    │
__manifest__.py
    │
dependency graph
    │
Database X
    │
installed modules
    │
install / upgrade / uninstall
```

Python import chain и construction effective models — следующий owner-урок.

---

## Что нельзя заключать

- addon on filesystem = installed module — нет;
- App = любой module — нет;
- Apps dashboard = complete architecture — нет;
- dependency = только Python import — нет;
- module обязан создать model — нет;
- `master data` всегда означает ERP reference/master objects — нет;
- Enterprise = отдельный independent server engine — нет.

## Контрольные вопросы

1. Что такое module/addon?
2. Что делает `addons_path`?
3. Почему Odoo module является Python package уже на уровне skeleton?
4. Какую роль выполняет manifest?
5. Чем App отличается от module?
6. Почему dependencies образуют graph?
7. Что такое link module?
8. Чем Odoo module master data отличается от будущей ERP master-data classification?
9. Какие состояния существуют между addon code и installed module?

## Официальные источники

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
2. A New Application  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/02_newapp.html
3. Module Manifests  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
4. Apps and modules  
   https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html
5. Define module data  
   https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html
6. Source install  
   https://www.odoo.com/documentation/19.0/administration/on_premise/source.html