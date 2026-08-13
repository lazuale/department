# Урок 2. Модульная система Odoo: addons path, manifest, зависимости и загрузка

## Цель урока

Понять, как Odoo обнаруживает расширения, чем addon отличается от App, какую роль играет `__manifest__.py`, как зависимости формируют порядок установки и загрузки и почему наличие кода на диске ещё не означает наличие функциональности в конкретной базе.

Этот урок намеренно не уходит глубоко в ORM-поля, views, security и XML-синтаксис. Здесь нас интересует **жизненный цикл модуля как единицы архитектуры**.

---

## 1. Module и addon — одно архитектурное семейство понятий

**[ODOO]** В Architecture Overview Odoo говорит, что modules также могут называться **addons**, а каталоги, в которых сервер ищет их, образуют `addons_path`.

Минимальная модель:

```text
Odoo server
    │
    └── addons_path
          ├── path A
          │    ├── module_1/
          │    └── module_2/
          │
          ├── path B
          │    └── module_3/
          │
          └── ...
```

**[ВЫВОД]** Addon — это не «установленное приложение». Сначала сервер должен иметь возможность обнаружить модуль в одном из каталогов `addons_path`, и только затем этот модуль может участвовать в жизненном цикле конкретной базы.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

---

## 2. `addons_path` — граница видимости кода для сервера

**[ODOO]** При source installation Odoo запускается через `odoo-bin`, а дополнительные каталоги модулей можно передавать через `--addons-path`.

Официальный пример для Community:

```bash
cd /CommunityPath
python3 odoo-bin --addons-path=addons -d mydb
```

**[ODOO]** Документация отдельно говорит, что custom addon paths можно добавлять сверх стандартных значений, чтобы сервер мог загружать custom modules.

Следовательно:

```text
модуль существует на файловой системе
              │
              ▼
его каталог находится в addons_path?
      │                     │
     нет                   да
      │                     │
      ▼                     ▼
сервер его             модуль может быть
не обнаруживает         обнаружен Odoo
```

**[ВЫВОД]** Физическое присутствие каталога с кодом и доступность addon для экземпляра Odoo — разные состояния.

Официальный источник:
https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

---

## 3. Community — это не «Enterprise без лицензии», а серверная основа

Это важно зафиксировать именно на уровне модульной архитектуры.

**[ODOO]** Официальная source-install документация прямо говорит:

- основной server code находится в Community edition;
- Enterprise repository не содержит полного исходного кода Odoo;
- Enterprise repository представляет собой набор дополнительных addons;
- Enterprise запускается поверх Community server с добавлением Enterprise addon path.

Концептуально:

```text
COMMUNITY
│
├── main server code
└── Community addons

            +

ENTERPRISE repository
└── additional addons

            ↓

same Community server
+ additional Enterprise addons_path
```

**[ВЫВОД]** Граница Community/Enterprise в архитектуре Odoo проходит прежде всего через **набор доступных addons**, а не через существование отдельного независимого ERP-движка Enterprise.

Это объясняет, почему общая пользовательская документация Odoo сама по себе не является надёжной матрицей Community/Enterprise: одна и та же серверная платформа может работать с разными наборами addons.

Официальный источник:
https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

---

## 4. Что делает каталог Python-пакета Odoo-модулем

**[ODOO]** Manifest file объявляет Python package как Odoo module и задаёт metadata модуля.

Файл называется:

```text
__manifest__.py
```

и содержит один Python dictionary.

Минимально концептуально:

```python
{
    'name': 'A Module',
    'depends': ['base'],
    'data': [],
}
```

**[ВЫВОД]** Само имя каталога ещё не определяет архитектурный смысл addon. Ключевая декларативная точка модуля — manifest.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 5. Manifest — контракт модуля с платформой

В manifest Odoo описывает свойства модуля.

Для архитектурного понимания особенно важны следующие поля.

### `name`

**[ODOO]** Человекочитаемое имя модуля.

Это не обязательно техническое имя addon. Техническая идентичность модуля связана с его package/addon name, а `name` служит человекочитаемым названием.

### `version`

**[ODOO]** Версия модуля.

### `depends`

**[ODOO]** Список Odoo modules, которые должны быть загружены раньше данного модуля, потому что текущий модуль использует созданные ими возможности или изменяет определённые ими ресурсы.

### `data`

**[ODOO]** Файлы данных, которые устанавливаются или обновляются вместе с модулем.

### `demo`

**[ODOO]** Файлы данных, устанавливаемые/обновляемые только в demonstration mode.

### `application`

**[ODOO]** Признак того, должен ли модуль считаться полноценным пользовательским application (`True`) либо техническим модулем, добавляющим функциональность существующему application (`False`).

### `auto_install`

**[ODOO]** Механизм автоматической установки модуля при выполнении условий по зависимостям.

### `external_dependencies`

**[ODOO]** Внешние Python или binary dependencies, отсутствие которых может препятствовать установке модуля.

Минимальная архитектурная схема:

```text
__manifest__.py
│
├── identity / metadata
├── dependency graph
├── data files
├── demo files
├── app-vs-technical marker
├── auto-install rule
└── external runtime dependencies
```

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 6. `application=True` объясняет, почему App и Module нельзя считать синонимами

**[ODOO]** Architecture Overview прямо отмечает: основные пользовательские modules помечаются и показываются как Apps, но большинство modules не являются Apps.

Manifest reference дополняет это полем:

```python
'application': True | False
```

Следовательно:

```text
MODULE
│
├── application=True
│      └── может позиционироваться как полноценное App
│
└── application=False
       └── технический / дополнительный module
```

**[ВЫВОД]** App — это пользовательский уровень представления части модульной системы, а не базовая техническая единица Odoo.

Поэтому выражение:

```text
"установить одно приложение"
```

не означает:

```text
"установить ровно один module"
```

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 7. Dependencies образуют граф, а не плоский список приложений

Рассмотрим:

```python
'depends': ['base', 'module_a', 'module_b']
```

**[ODOO]** При установке модуля его dependencies устанавливаются раньше него. При загрузке dependencies также загружаются раньше зависимого модуля.

То есть зависимость задаёт отношение:

```text
module_a ─────┐
              │
module_b ─────┼──► current_module
              │
base ─────────┘
```

Но `module_a` и `module_b` сами могут иметь зависимости:

```text
                base
               /    \
              ▼      ▼
         module_a  module_b
              \      /
               ▼    ▼
            module_c
```

**[ВЫВОД]** Установленная конфигурация Odoo формируется **графом модулей**, а не линейным перечнем приложений из главного меню.

Это одно из центральных объяснений того, почему установка одного пользовательского App может существенно менять модели и интерфейс других частей системы.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

---

## 8. `base` занимает особое положение

**[ODOO]** Manifest reference говорит, что module `base` всегда установлен в любой Odoo instance.

При этом документация рекомендует всё равно явно указывать его в `depends`, чтобы модуль корректно обновлялся при обновлении `base`.

Концептуально:

```text
base
 │
 ├── module A
 ├── module B
 └── module C
```

Это не означает, что весь граф модулей непосредственно зависит только от `base`: реальные зависимости могут быть многоуровневыми.

**[ВЫВОД]** `base` — платформенная зависимость особого уровня, но не «универсальная бизнес-модель» и не корень предметной иерархии данных.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 9. Dependencies нужны не только чтобы «использовать чужой код»

Официальное определение `depends` содержит два важных случая.

Модуль зависит от другого, если он:

1. использует возможности, которые другой module создаёт;
2. изменяет resources, которые другой module определяет.

Это второй случай особенно важен для идеологии Odoo.

```text
module A
   │
   └── defines Model X / View X / other resource

module B
   │
   ├── depends on module A
   └── extends / alters resources of module A
```

**[ВЫВОД]** Зависимость в Odoo — это не просто package dependency в стиле «нужна библиотека». Она также задаёт архитектурную последовательность расширения существующей функциональности.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 10. Auto-install modules — официальный механизм интеграционных мостов

**[ODOO]** `auto_install` может автоматически установить module при наличии его dependencies.

Официальная документация прямо приводит типичный вариант: **link modules**, которые реализуют интеграцию между двумя иначе независимыми modules.

Пример из manifest reference:

```text
sale
   \
    \       sale_crm
     ─────► integration module
    /
   /
crm
```

Когда необходимые зависимости установлены, link module может добавиться автоматически.

**[ВЫВОД]** Это важнейшая часть идеологии Odoo:

> интеграция двух предметных контуров не обязательно должна быть жёстко встроена в любой из них.

Вместо этого архитектура допускает отдельный bridge/addon, который знает сразу об обеих сторонах.

Это уменьшает прямую связанность базовых modules.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 11. Отсюда возникает канонический паттерн Odoo

```text
MODULE A                    MODULE B
   │                           │
   │ независимая логика        │ независимая логика
   │                           │
   └────────────┐ ┌────────────┘
                ▼ ▼
             BRIDGE MODULE
                  │
                  └── интеграционное поведение
```

**[ВЫВОД]** Это архитектурно чище, чем заставлять `MODULE A` всегда зависеть от `MODULE B`, если интеграция нужна лишь тогда, когда установлены оба.

Мы будем искать этот паттерн позже при разборе реальных предметных приложений Odoo.

---

## 12. Module может вообще не создавать новый бизнес-объект

**[ODOO]** Architecture Overview перечисляет возможные элементы module:

- business objects;
- object views;
- data files;
- web controllers;
- static web data.

И отдельно подчёркивает: **ни один из этих элементов не является обязательным**.

Developer tutorial также замечает, что некоторые modules существуют исключительно для добавления data в Odoo.

Следовательно, модуль может:

```text
A. создать новые models
B. расширить чужие models
C. добавить только data
D. добавить только UI/resources
E. добавить integration behavior
F. сочетать несколько вариантов
```

**[ВЫВОД]** По названию модуля нельзя автоматически заключить, что в базе появится отдельная новая предметная сущность.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

---

## 13. Data files — часть установки модуля

**[ODOO]** Данные module могут объявляться в CSV или XML. Чтобы Odoo загрузила file, он должен быть указан в manifest.

Основные manifest keys:

```python
'data': [
    'security/ir.model.access.csv',
    'views/example_views.xml',
    'data/master_data.xml',
],
'demo': [
    'demo/demo_data.xml',
],
```

**[ODOO]** В tutorial `data` описывается как master data, всегда устанавливаемая вместе с module, а `demo` — как demonstration data.

**[ODOO]** Views и actions также являются data.

Это важно:

```text
MODULE INSTALLATION
       │
       ├── Python definitions / extensions
       │
       └── declared data files
             ├── security
             ├── views/actions
             ├── configuration/master data
             └── demo data (when enabled)
```

**[ВЫВОД]** Установка addon — это не только «подключить Python-код». Она может изменять структуру и содержимое базы посредством декларативно загружаемых records.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html

---

## 14. Порядок data files тоже является зависимостью

**[ODOO]** В importable modules tutorial Odoo требует перечислять файлы в `data` section manifest **в порядке зависимости**, обычно начиная с model files в соответствующем сценарии.

Общий смысл:

```text
file A creates something
        │
        ▼
file B references it
```

значит, `file A` должен быть загружен раньше `file B`.

**[ВЫВОД]** У модуля существует два уровня зависимости:

```text
INTER-MODULE
module dependencies

INTRA-MODULE
order/dependencies of declared data files
```

Это полезное различие для дальнейшего понимания external IDs, views и security.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/importable_modules.html

---

## 15. Код доступен серверу ≠ module установлен в базе

Это один из самых важных выводов урока.

Можно иметь:

```text
addons_path
   │
   ├── module_a/   ← существует на диске
   ├── module_b/   ← существует на диске
   └── module_c/   ← существует на диске
```

но конкретная база может иметь установленными только часть:

```text
DATABASE X
   ├── module_a installed
   └── module_b installed

module_c available to server,
but not installed in DATABASE X
```

Основания:

**[ODOO]** Apps and modules устанавливаются в database.

**[ODOO]** ORM автоматически создаёт model instances один раз для каждой database, и доступные модели зависят от modules, установленных именно в этой database.

Поэтому:

```text
FILESYSTEM AVAILABILITY
        ≠
DATABASE INSTALLATION STATE
        ≠
AVAILABLE MODEL SET IN THAT DATABASE
```

**[ВЫВОД]** Когда мы позже будем говорить «Odoo содержит модель X», всегда надо уточнять уровень утверждения:

- код модели существует в доступном addon;
- module установлен;
- model реально входит в итоговый набор моделей этой базы.

Официальные источники:

- https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html
- https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 16. App list — не сам `addons_path`

**[ODOO]** Если новый app/module не отображается в интерфейсе Apps, документация предлагает включить developer mode и выполнить:

```text
Apps → Update Apps List → Update
```

**[ВЫВОД]** Значит, надо различать:

```text
addon присутствует в доступном server path
        ↓
Odoo знает/обновила список доступных modules
        ↓
module доступен для установки
        ↓
module установлен в конкретной database
```

Это разные этапы жизненного цикла.

Официальный источник:
https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

---

## 17. Install, upgrade и uninstall — разные операции над состоянием базы

**[ODOO]** Odoo отдельно документирует операции:

```text
install
upgrade
uninstall
```

### Install

Вводит module и его необходимые dependencies в установленную конфигурацию database.

### Upgrade

Применяет обновлённую версию module к базе.

### Uninstall

Удаляет module из установленной конфигурации.

**[ODOO]** Документация отдельно предупреждает, что uninstall приложения удаляет связанные database records и может затронуть другие modules из-за зависимостей.

**[ВЫВОД]** Module lifecycle — это операция над структурой и данными конкретной базы, а не просто включение/выключение пункта меню.

Официальный источник:
https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

---

## 18. CLI подтверждает, что modules являются управляемой единицей платформы

**[ODOO]** Odoo 19 CLI имеет отдельные module subcommands:

```bash
odoo-bin module install <modules>
odoo-bin module uninstall <modules>
odoo-bin module upgrade <modules>
odoo-bin module forcedemo <modules>
```

Для этих операций доступны параметры database и addons path.

**[ВЫВОД]** Модульная система — не только визуальная функция Apps dashboard. Она является серверным механизмом управления составом базы.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/cli.html

---

## 19. Что происходит с моделями после установки modules

Здесь важно не придумать лишнего.

**[ODOO]** ORM reference устанавливает следующие факты:

1. Odoo автоматически инстанцирует каждую model один раз для каждой database;
2. эти instances представляют модели, доступные в конкретной database;
3. их набор зависит от установленных modules;
4. фактический Python class каждого model instance строится из Python classes, которые создают и наследуют соответствующую модель.

Концептуально:

```text
installed modules in DB
        │
        ├── class creates model X
        ├── class extends model X
        ├── another class extends model X
        └── class creates model Y
                │
                ▼
      effective model definitions
                │
                ▼
    models available in that DB
```

**[ВЫВОД]** Нельзя анализировать итоговую model только по одному Python class из одного addon. Её действительное поведение в базе может быть составлено из нескольких module extensions.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 20. Осторожно со словом `registry`

В Odoo существует несколько контекстов слова registry, включая отдельные registries frontend framework. Поэтому нельзя использовать слово «registry» без уточнения.

В этом курсе пока используем точную формулировку:

> **набор моделей, доступных в конкретной database после учёта установленных modules и их model inheritance/extension**.

До отдельного разбора server internals мы не будем приписывать этому механизму детали, которых ещё не установили официальной документацией.

**[ВЫВОД]** Это защищает нас от смешения backend model machinery с frontend registries, которые в Odoo являются отдельным механизмом web client.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- https://www.odoo.com/documentation/19.0/developer/reference/frontend/registries.html

---

## 21. Полный жизненный цикл addon в минимальной модели

```text
1. ADDON CODE EXISTS
        │
        ▼
2. DIRECTORY IS IN addons_path
        │
        ▼
3. __manifest__.py DECLARES ODOO MODULE
        │
        ▼
4. ODOO DISCOVERS / LISTS AVAILABLE MODULE
        │
        ▼
5. INSTALL REQUEST FOR DATABASE
        │
        ▼
6. RESOLVE DEPENDENCIES
        │
        ▼
7. INSTALL / LOAD DEPENDENCIES FIRST
        │
        ▼
8. LOAD MODULE PYTHON + DECLARED DATA
        │
        ▼
9. EFFECTIVE MODELS / DATA / UI / SECURITY
   BECOME PART OF THAT DATABASE CONFIGURATION
```

Некоторые шаги здесь являются **[ВЫВОД]** — это архитектурная последовательность, собранная из нескольких официально документированных механизмов, а не буквальная блок-схема из документации Odoo.

---

## 22. Что означает установка одного App с точки зрения архитектуры

Пользователь видит:

```text
[Activate Sales]
```

Архитектурно это может означать:

```text
user-facing application module
          │
          ├── dependency 1
          ├── dependency 2
          ├── dependency 3
          │
          └── auto-install bridges may appear
                    │
                    ▼
       several models are created/extended
                    │
       several data files are loaded
                    │
       views/actions/security may change
```

**[ВЫВОД]** Поэтому изучать Odoo исключительно через карточки Apps — значит видеть верхушку dependency graph, но не саму архитектуру.

---

## 23. Идеологический вывод: Odoo строится композиционно

На текущем уровне мы уже можем сформулировать первый сильный архитектурный вывод.

**[ВЫВОД]** Odoo предпочитает **композицию функциональности через небольшие взаимозависимые modules и extensions**, а не один монолитный application package со всей логикой внутри.

На это указывают сразу несколько официально документированных механизмов:

- большинство modules не являются Apps;
- module может расширять resources другого module;
- dependencies задают порядок;
- auto-install link modules соединяют независимые модули;
- итоговый Python class модели строится из создающих и наследующих её classes.

Это одна из ключевых идей, которую нужно сохранить для всех дальнейших уроков.

---

## 24. Ошибочные модели мышления

### Ошибка 1. «Addon лежит на сервере — значит функция уже работает»

Нет.

```text
available on filesystem
≠ installed in database
```

### Ошибка 2. «Одно App = один technical module»

Нет. Основные modules могут быть Apps, но большинство modules не Apps, а зависимости могут устанавливать дополнительные modules.

### Ошибка 3. «Dependency нужна только для импорта Python-кода»

Нет. Dependency также используется, когда module изменяет resources, определённые другим module.

### Ошибка 4. «Integration надо писать внутри одного из двух приложений»

Не обязательно. Odoo официально поддерживает auto-install link modules для интеграции независимых modules.

### Ошибка 5. «Manifest — просто описание для Apps Store»

Нет. Manifest объявляет Python package как Odoo module и задаёт dependencies, data, demo, application/auto-install semantics и другие свойства.

### Ошибка 6. «Module обязательно создаёт свою таблицу или model»

Нет. Odoo прямо говорит, что элементы module не обязательны; некоторые modules существуют только для добавления data.

### Ошибка 7. «Community и Enterprise — два разных серверных продукта»

Нет. Официальная source-install документация описывает Enterprise как дополнительные addons поверх Community server code.

---

## 25. Минимальная модель в голове после урока

```text
                    ODOO SERVER
                        │
                   addons_path
                        │
            ┌───────────┴───────────┐
            │                       │
       available addon         available addon
            │                       │
      __manifest__.py          __manifest__.py
            │                       │
            └───────────┬───────────┘
                        │
                 dependency graph
                        │
                        ▼
                    DATABASE
                        │
                installed modules
                        │
       ┌────────────────┼─────────────────┐
       │                │                 │
    models           data              UI/security/
 create/extend     records             other resources
       │
       ▼
 effective models available
 in this particular database
```

---

## 26. Что мы пока сознательно НЕ разбираем

После этого урока ещё нельзя считать понятыми:

- Python import mechanics Odoo modules;
- `_name`, `_inherit`, `_inherits`;
- Environment;
- model metaclasses/server internals;
- exact model registry implementation;
- fields;
- external IDs;
- XML record operations;
- `noupdate`;
- ACL/record rules;
- views inheritance;
- frontend asset loading;
- controllers;
- upgrade scripts/migrations.

Эти темы будут вынесены в отдельные уроки, чтобы не смешивать разные уровни платформы.

---

## 27. Контрольные вопросы

Перед следующим уроком нужно уметь ответить:

1. Что такое `addons_path`?
2. Чем физическое наличие addon отличается от установки module в database?
3. Какую роль выполняет `__manifest__.py`?
4. Почему App и module не являются синонимами?
5. Что задаёт `depends`?
6. В каком порядке устанавливаются dependencies?
7. Почему dependency может быть нужна для расширения чужого resource?
8. Что такое `auto_install`?
9. Какую архитектурную проблему решают link modules?
10. Почему один пользовательский App может привести к установке нескольких modules?
11. Что хранится в `data` и `demo` manifest sections?
12. Почему порядок data files может иметь значение?
13. Чем доступный module отличается от установленного module?
14. Что происходит с набором моделей конкретной database при изменении установленных modules?
15. Как технически связаны Community server и Enterprise addons?

---

## Официальные источники урока

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

2. Module Manifests  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

3. Source install  
   https://www.odoo.com/documentation/19.0/administration/on_premise/source.html

4. Apps and modules  
   https://www.odoo.com/documentation/19.0/applications/general/apps_modules.html

5. Define module data  
   https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html

6. Building a Module  
   https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

7. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

8. Command-line interface (CLI)  
   https://www.odoo.com/documentation/19.0/developer/reference/cli.html

9. Write importable modules  
   https://www.odoo.com/documentation/19.0/developer/tutorials/importable_modules.html

10. Frontend Registries — только для разграничения терминов  
    https://www.odoo.com/documentation/19.0/developer/reference/frontend/registries.html
