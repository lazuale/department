# Урок 2. Модульная система: addons path, manifest, dependencies и lifecycle

## Цель урока

Понять жизненный цикл Odoo module как технической единицы платформы:

```text
code exists
→ server can discover it
→ manifest declares it
→ database can install it
→ dependencies affect installed module graph
→ upgrade/uninstall change database state
```

Этот урок **не разбирает** Python import/loading mechanics, model inheritance, external IDs и внутреннюю загрузку data files. Это следующие уровни курса.

---

## 1. Module и addon

**[ODOO]** Odoo modules также называются addons.

**[ODOO]** Каталоги, в которых server ищет modules, образуют `addons_path`.

```text
Odoo server
   │
   └── addons_path
        ├── directory A
        │    ├── module_x/
        │    └── module_y/
        └── directory B
             └── module_z/
```

**[ВЫВОД]** Addon на filesystem — ещё не installed module конкретной database.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 2. `addons_path` определяет, где server ищет modules

**[ODOO]** Module directories задаются через `--addons-path` или соответствующую configuration option.

**[ODOO]** Дополнительные addon directories можно подключать к server, не изменяя core Odoo source tree.

Модель:

```text
module directory exists
        │
        ▼
is parent directory in addons_path?
        │
    ┌───┴───┐
   no      yes
    │        │
    ▼        ▼
not found   can be discovered
by server   as an available addon
```

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

---

## 3. Manifest объявляет Python package как Odoo module

**[ODOO]** Manifest file называется:

```text
__manifest__.py
```

Он содержит один Python dictionary и служит для:

- объявления package как Odoo module;
- задания module metadata;
- dependencies;
- data/demo files;
- application/auto-install semantics;
- external dependencies и других module properties.

Минимальный пример:

```python
{
    'name': 'Example',
    'depends': ['base'],
    'data': [],
}
```

**[ВЫВОД]** Имя каталога и наличие Python files сами по себе не заменяют manifest как декларативную точку Odoo module.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 4. Ключевые поля manifest

Для архитектуры курса сейчас нужны следующие поля.

### `name`

**[ODOO]** Human-readable name модуля.

### `version`

**[ODOO]** Version модуля.

### `depends`

**[ODOO]** Odoo modules, которые должны быть загружены раньше текущего module, потому что текущий module использует их возможности или изменяет определённые ими resources.

### `data`

**[ODOO]** Data files, которые загружаются при installation/update module.

### `demo`

**[ODOO]** Demonstration data files, используемые в demo mode.

### `application`

**[ODOO]** Признак того, является ли module полноценным user-facing application.

### `auto_install`

**[ODOO]** Условие автоматической установки module при наличии соответствующих dependencies.

### `external_dependencies`

**[ODOO]** Внешние Python/binary dependencies, которые могут быть необходимы для installation.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 5. App является специальным случаем module

**[ODOO]** Main user-facing modules выставляются как Apps; большинство modules не являются Apps.

**[ODOO]** Manifest field `application` определяет, следует ли рассматривать module как полноценное application.

```text
Odoo modules
├── App modules
└── supporting / technical modules
```

**[ВЫВОД]** Apps dashboard показывает только верхний user-facing слой dependency graph.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 6. `depends` формирует граф modules

Пример:

```python
'depends': ['base', 'module_a', 'module_b']
```

**[ODOO]** Dependencies загружаются до зависимого module.

**[ODOO]** При установке зависимого module необходимые dependencies также устанавливаются.

Граф может быть многоуровневым:

```text
          base
         /    \
        ▼      ▼
   module_a  module_b
        \      /
         ▼    ▼
        module_c
```

**[ВЫВОД]** Состав базы — dependency graph modules, а не плоский перечень Apps.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

---

## 7. Dependency означает больше, чем Python import

**[ODOO]** Manifest reference объясняет `depends` двумя основными причинами:

1. current module использует capabilities другого module;
2. current module изменяет resources, определённые другим module.

```text
module A
  └── defines resource X

module B
  ├── depends on A
  └── changes/extends resource X
```

**[ВЫВОД]** Module dependency в Odoo задаёт архитектурный порядок композиции функциональности, а не только библиотечную зависимость.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 8. `base` — особая обязательная module dependency

**[ODOO]** Module `base` всегда установлен в Odoo instance.

Документация рекомендует явно указывать его в `depends`, когда module зависит непосредственно от base/platform resources.

**[ВЫВОД]** `base` является системной module foundation, но это не означает, что она является родителем всех business records или предметной иерархией ERP.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 9. `auto_install` и link modules

**[ODOO]** `auto_install` может автоматически установить module при выполнении dependency conditions.

**[ODOO]** Manifest reference прямо приводит link module как типичный сценарий: отдельный module реализует integration между двумя иначе независимыми modules.

```text
Module A      Module B
    \          /
     \        /
      ▼      ▼
      Link module
```

**[ВЫВОД]** Официально документированный integration pattern Odoo допускает вынесение связи двух предметных modules в отдельный bridge/link module, вместо жёсткой зависимости одной стороны от другой.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 10. Module не обязан создавать model

**[ODOO]** Architecture Overview перечисляет возможные части module:

- business objects;
- object views;
- data files;
- web controllers;
- static web data.

**[ODOO]** Ни один из этих элементов не является обязательным.

Следовательно, module может:

```text
create models
extend existing resources
load only data
add UI/resources
add integration behavior
combine several of these
```

**[ВЫВОД]** Нельзя по названию addon автоматически заключать, что он создаёт новую independent ORM model или SQL table.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 11. `data` и `demo` пока рассматриваем только как manifest dependencies

**[ODOO]** XML/CSV files должны быть перечислены в manifest, чтобы Odoo загрузила их как module data.

**[ODOO]** `data` загружается при normal installation/update; `demo` относится к demonstration data.

На текущем уровне достаточно модели:

```text
module
├── Python behavior
└── declared data files
```

External IDs, XML record operations, `noupdate` и точный data load order будут отдельным уроком 08.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html

---

## 12. Filesystem availability, Apps list и installed state — разные уровни

Полезно различать:

```text
1. addon code exists on filesystem
        │
2. server can see parent directory through addons_path
        │
3. Odoo can discover/list module
        │
4. module is available for installation
        │
5. module is installed in a particular database
```

**[ODOO]** Apps/modules устанавливаются в database.

**[ODOO]** Для появления нового module в Apps list documentation использует `Update Apps List` в developer mode.

**[ВЫВОД]** Available module и installed module — не одно состояние.

Официальный источник:
https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

---

## 13. Install, upgrade и uninstall меняют состояние database

**[ODOO]** Odoo отдельно поддерживает:

```text
install
upgrade
uninstall
```

### Install

Вводит module и необходимые dependencies в installed configuration database.

### Upgrade

Применяет новую версию module/data к database.

### Uninstall

Удаляет module из installed configuration; documentation предупреждает о возможном удалении связанных database records и влиянии dependency relations.

**[ВЫВОД]** Module lifecycle — операция над структурой и данными database, а не просто toggle элемента меню.

Официальный источник:
https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

---

## 14. CLI подтверждает module lifecycle как server capability

**[ODOO]** Odoo CLI имеет module operations для install, uninstall и upgrade, а также database/addons-path parameters.

**[ВЫВОД]** Apps dashboard является UI над частью модульного механизма, но сам lifecycle существует на server level.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/cli.html

---

## 15. Community и Enterprise как наборы addons

**[ODOO]** Architecture Overview и source installation documentation описывают Enterprise functionality как дополнительные modules поверх Community modules/server.

```text
Community server + Community addons
              │
              ├── Community database configuration
              │
              └── + Enterprise addon path/modules
                         ↓
                 Enterprise functionality
```

**[ВЫВОД]** Edition boundary технически выражается прежде всего доступным набором modules, поэтому наличие feature в общей User Documentation не является самостоятельным доказательством Community availability.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

---

## 16. Минимальная модель после урока

```text
                         ODOO SERVER
                              │
                         addons_path
                              │
                     available addons
                              │
                       __manifest__.py
                              │
                      dependency graph
                              │
                              ▼
                         DATABASE
                              │
                      installed modules
                              │
                    install / upgrade /
                         uninstall
```

В следующем уроке будет разобрано, **как Python package и dependency order превращаются в загруженные model definitions конкретной database**.

---

## 17. Что считать ошибкой

### «Addon находится на сервере, значит работает в базе»

Нет. Availability и installed state различаются.

### «App = module»

Не полностью. App — user-facing module; большинство modules не Apps.

### «Dependency — только импорт чужого Python package»

Нет. Module может зависеть от другого, чтобы изменять определённые им resources.

### «Module обязательно создаёт отдельный model»

Нет.

### «Integration обязательно надо встроить в одну из сторон»

Нет. Odoo официально документирует link modules через `auto_install`.

### «Enterprise — отдельный независимый server engine»

Нет. Официальная архитектура описывает дополнительные Enterprise modules поверх Community modules/server.

---

## 18. Контрольные вопросы

1. Что такое module/addon?
2. Что определяет `addons_path`?
3. Какую роль выполняет `__manifest__.py`?
4. Чем App отличается от обычного module?
5. Что означает `depends`?
6. Почему dependencies формируют graph?
7. Почему dependency может быть нужна для изменения чужого resource?
8. Что делает `auto_install`?
9. Что такое link module?
10. Может ли module не создавать ни одной model?
11. Чем `data` отличается от `demo` на уровне manifest?
12. Чем available module отличается от installed module?
13. Что изменяют install/upgrade/uninstall?
14. Как связаны Community и Enterprise на module level?

---

## Официальные источники урока

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

2. Module Manifests  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

3. Apps and modules  
   https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

4. Source install  
   https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

5. Define module data  
   https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html

6. CLI  
   https://www.odoo.com/documentation/19.0/developer/reference/cli.html
