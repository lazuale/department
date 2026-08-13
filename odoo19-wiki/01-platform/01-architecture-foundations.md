# Урок 1. Что такое Odoo: архитектурный фундамент

## Цель урока

Построить минимальную, но непротиворечивую модель Odoo до знакомства с предметными приложениями.

После урока должны быть различимы:

- Odoo server process;
- database;
- module/addon;
- App;
- ORM model;
- record/recordset;
- action/view/menu как пользовательский слой.

Fields, security, inheritance, multi-company и внутренняя loading mechanics здесь только обозначаются и будут отдельными уроками.

---

## 1. Odoo — three-tier application

**[ODOO]** Официальный Architecture Overview описывает Odoo как three-tier application:

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

**[ВЫВОД]** Экран Odoo нельзя считать самой бизнес-моделью. UI, server-side business logic и persistence — разные уровни системы.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 2. Один Odoo instance может обслуживать несколько databases

**[ODOO]** Server Framework 101 отдельно предупреждает, что один Odoo instance может запускать несколько databases параллельно внутри одного Python process.

**[ODOO]** На этих databases могут быть установлены разные modules.

Минимальная модель:

```text
                 ODOO SERVER PROCESS
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
     Database A      Database B      Database C
          │              │              │
    module set A     module set B     module set C
```

**[ВЫВОД]** Набор прикладной функциональности — свойство не только файлов сервера, но и состояния конкретной database.

Отсюда позже станет понятным, почему Odoo запрещает опираться на mutable global variables, зависящие от установленных modules.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

---

## 3. Всё расширение Odoo упаковывается в modules

**[ODOO]** Официальная документация говорит: server и client extensions packaged as modules, которые optionally loaded in a database.

Module — набор функций и данных, направленный на определённую цель. Он может:

- добавить новую business logic;
- изменить существующую business logic;
- добавить data files;
- добавить views;
- добавить controllers;
- добавить static web data;
- содержать только часть этих элементов.

**[ODOO]** Термин `addon` используется как синоним module.

**[ВЫВОД]** Module — базовая техническая единица композиции Odoo, но не обязательно отдельный бизнес-объект или отдельный экран.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 4. App — не синоним module и не синоним функциональной области

**[ODOO]** Основные user-facing modules помечаются и показываются как Apps, но большинство Odoo modules не являются Apps.

Manifest имеет отдельный признак `application`.

Поэтому:

```text
MODULE
├── application=True
│      └── user-facing App
│
└── application=False
       └── technical/supporting module
```

В этом курсе:

```text
App
= user-facing Odoo module

Functional area
= наша аналитическая категория,
  которая может состоять из нескольких modules
```

**[ВЫВОД]** Фраза «Sales как функциональная область» и конкретный App/module — не обязательно одно и то же понятие.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 5. Module является Python package, когда содержит Python business objects

**[ODOO]** Odoo module объявляется manifest-файлом `__manifest__.py`.

Если module содержит Python business objects, они организуются как Python package с `__init__.py`, который импортирует Python files/module packages.

Упрощённая структура из Architecture Overview:

```text
module/
├── models/
│   ├── *.py
│   └── __init__.py
├── data/
│   └── *.xml
├── __init__.py
└── __manifest__.py
```

На практике module также может иметь `views/`, `security/`, `controllers/`, `static/` и другие каталоги в зависимости от содержания.

**[ВЫВОД]** `__manifest__.py` объявляет Odoo module, а `__init__.py` относится к Python import structure. Эти механизмы связаны, но не выполняют одну и ту же роль.

Подробная loading/import mechanics будет отдельным уроком.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 6. ORM model — основной серверный бизнес-объект

**[ODOO]** Business objects объявляются Python classes, наследующими ORM model classes.

Формальные базовые типы:

```text
BaseModel
├── Model
├── TransientModel
└── AbstractModel
```

- `Model` — обычные database-persisted models;
- `TransientModel` — временные records, которые сохраняются в БД, но автоматически vacuumed;
- `AbstractModel` — общие abstract superclasses для переиспользования behavior.

**[ВЫВОД]** Термины «справочник», «документ», «master data» и «transaction» не являются альтернативными формальными типами ORM.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 7. Available models зависят от database

**[ODOO]** Odoo автоматически instantiates every model once per database.

**[ODOO]** Эти instances представляют models, доступные в конкретной database, и зависят от modules, установленных на этой database.

**[ODOO]** Actual class каждой model instance строится из Python classes, которые создают и наследуют соответствующую model.

Минимальная схема:

```text
Database
   │
   ▼
Installed modules
   │
   ├── create Model X
   ├── extend Model X
   └── create Model Y
          │
          ▼
Effective models available
in this database
```

**[ВЫВОД]** Нельзя корректно описать итоговую model, посмотрев только на один Python class одного addon.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 8. Record и recordset

**[ODOO]** Model instance является recordset — ordered collection of records этой модели.

**[ODOO]** Record не имеет отдельного object representation: одна запись представляется recordset из одного record.

```text
Model
  │
  └── Recordset
       ├── 0 records
       ├── 1 record  = singleton recordset
       └── N records
```

Подробная recordset semantics будет в ORM Core.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 9. UI layer не равен предметной модели

Odoo module может загружать views, actions и menus как data/configuration records.

На минимальном уровне достаточно понимать:

```text
Menu / button
      │
      ▼
Action
      │
      ▼
Model + View
      │
      ▼
Web client
```

**[ВЫВОД]** Отдельный пункт меню не доказывает наличие отдельной независимой ORM-модели или отдельного business domain.

Actions/views/menus будут отдельным уроком после data files и security.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html

---

## 10. Community и Enterprise на уровне архитектуры

**[ODOO]** Odoo существует в Community и Enterprise editions.

**[ODOO]** С технической точки зрения Enterprise functionality представляет дополнительные modules, установленные поверх modules Community version.

**[ВЫВОД]** Общая серверная архитектура остаётся модульной; различие редакций в значительной степени выражается составом доступных modules.

Это не означает, что любая функция общей документации доступна в Community. Edition каждого спорного приложения/feature проверяется отдельно.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 11. Первая непротиворечивая модель Odoo

```text
                        ODOO SERVER PROCESS
                                │
                    ┌───────────┴───────────┐
                    │                       │
               Database A              Database B
                    │                       │
             installed modules        installed modules
                    │                       │
                    ▼                       ▼
             effective models         effective models
                    │
                    ▼
              records/recordsets
                    │
        ┌───────────┴───────────┐
        │                       │
   business behavior       data/UI/security
                            resources
                                │
                                ▼
                           web client
```

А на файловом уровне server видит modules через `addons_path`; это будет разобрано в уроке 2.

---

## 12. Что из этого урока нельзя заключать

### Нельзя: «Odoo — это набор независимых Apps»

Apps являются верхним user-facing слоем модульной системы.

### Нельзя: «один App = один model»

Module может создавать и расширять несколько models, а dependencies могут добавлять дополнительные modules.

### Нельзя: «одна database = один Odoo process»

Один instance может обслуживать несколько databases.

### Нельзя: «module лежит на диске — значит установлен»

Filesystem availability и database installation state — разные уровни.

### Нельзя: «menu = business object»

Menu — часть UI/navigation layer.

### Нельзя: «model = ровно одна SQL table»

Model — объект ORM с behavior, fields и extensions; физическое хранение будет разбираться отдельно.

---

## 13. Контрольные вопросы

1. Какие три tier выделяет официальная архитектура Odoo?
2. Может ли один Odoo instance обслуживать несколько databases?
3. Могут ли эти databases иметь разные installed modules?
4. Что такое module/addon?
5. Чем App отличается от module?
6. Зачем module нужны `__manifest__.py` и `__init__.py`?
7. Какие три базовых ORM model classes существуют?
8. Почему available models зависят от конкретной database?
9. Что означает singleton recordset?
10. Почему menu/action/view нельзя считать самой предметной моделью?

---

## Официальные источники урока

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

2. Models and Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

3. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

4. Module Manifests  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

5. Actions  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html
