# Урок 3. Загрузка модулей: Python package, imports и per-database model construction

## Цель урока

Связать два предыдущих уровня:

```text
module существует и установлен
        ↓
его Python package импортируется
        ↓
model classes становятся частью Odoo model machinery
        ↓
available models формируются для конкретной database
```

Этот урок описывает только то, что официальная документация Odoo 19 позволяет установить надёжно. Мы сознательно **не моделируем внутреннюю реализацию registry/metaclasses глубже, чем она документирована**.

---

## 1. Module с Python-кодом является Python package

**[ODOO]** Если Odoo module содержит business objects в Python, они организуются как Python package.

У module есть верхний:

```text
__init__.py
```

а у подпакетов, например `models/`, — собственный `__init__.py`.

Официальный Architecture Overview показывает структуру:

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

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 2. `__manifest__.py` и `__init__.py` решают разные задачи

```text
__manifest__.py
= declares Odoo module metadata/dependencies/data

__init__.py
= Python import structure
```

**[ODOO]** Manifest declares Python package as Odoo module.

**[ODOO]** `__init__.py` содержит import instructions для Python files/packages module.

Пример:

```python
from . import models
```

а внутри `models/__init__.py`:

```python
from . import estate_property
from . import estate_property_offer
```

**[ВЫВОД]** Manifest делает package частью Odoo module system, а Python imports делают model declarations реально импортируемыми server process.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/02_newapp.html

---

## 3. Python file с model class сам по себе недостаточен

Если создать:

```text
models/example.py
```

но не подключить его через Python import chain, наличие file в module tree само по себе не означает, что Python interpreter выполнил его declarations.

Типичная import chain:

```text
module/__init__.py
        │
        ▼
from . import models
        │
        ▼
models/__init__.py
        │
        ▼
from . import example
        │
        ▼
models/example.py
        │
        ▼
class Example(models.Model): ...
```

**[ВЫВОД]** File organization и actual Python import graph — разные вещи.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 4. Dependencies задают порядок между modules

**[ODOO]** Manifest `depends` указывает modules, которые должны быть загружены раньше текущего module.

Это нужно, когда current module:

- использует capabilities dependency;
- изменяет resources, которые dependency определяет.

```text
module A
   │
   ▼
loaded before
   │
module B
```

**[ВЫВОД]** У Python import order внутри одного module и у Odoo dependency order между modules разные уровни:

```text
INTRA-MODULE
__init__.py import chain

INTER-MODULE
depends graph
```

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

---

## 5. Installed module graph различается между databases

**[ODOO]** Один Odoo instance может обслуживать несколько databases в одном Python process.

**[ODOO]** На разных databases могут быть установлены разные modules.

```text
same Python process
        │
        ├── DB A: base + module_x
        └── DB B: base + module_y
```

Это важное ограничение архитектуры.

**[ODOO]** Server Framework 101 прямо предупреждает не использовать mutable global variables, зависящие от installed modules, именно из-за multi-database runtime.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

---

## 6. Odoo creates model instances per database

**[ODOO]** ORM автоматически instantiates every model once per database.

**[ODOO]** Эти instances представляют models, доступные в данной database, и зависят от modules, установленных на ней.

```text
DB A installed modules
        │
        ▼
available models A

DB B installed modules
        │
        ▼
available models B
```

**[ВЫВОД]** Model availability — per-database runtime concept, а не просто перечень Python classes, существующих на filesystem.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 7. Actual model class может быть составлен из нескольких Python classes

**[ODOO]** ORM reference говорит: actual class каждого model instance строится из Python classes, которые создают и наследуют соответствующую model.

Концептуально:

```text
module A
└── Python class creates model x

module B
└── Python class inherits/extends model x

module C
└── another Python class inherits/extends model x

        ↓
actual model x class in database
```

Подробная семантика `_inherit` и `_inherits` будет только в уроке 11.

**[ВЫВОД]** Уже сейчас достаточно понимать: итоговая model является результатом module composition и model inheritance, а не обязательно единственного class definition.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 8. Почему нельзя использовать слово `registry` небрежно

ORM Environment предоставляет access to registry by mapping model names to models.

Но Odoo также использует термин registries во frontend framework.

Поэтому до отдельного разбора internals в курсе используем точную формулировку:

> **per-database set/mapping of available ORM models**

когда речь идёт о backend ORM.

Мы не приписываем registry внутреннюю implementation detail, если она не нужна для текущего уровня.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- https://www.odoo.com/documentation/19.0/developer/reference/frontend/registries.html

---

## 9. Что module loading не означает

### Не означает: «все Python models на server доступны всем databases»

Нет. Available models зависят от installed modules конкретной database.

### Не означает: «manifest importирует Python files»

Нет. Manifest и Python `__init__.py` имеют разные роли.

### Не означает: «каждый file в `models/` автоматически исполняется только из-за местоположения»

Python package должен импортировать нужный module file.

### Не означает: «одна ORM model = один Python class»

Actual class может быть составлен из classes, создающих и наследующих model.

### Не означает: «backend registry = frontend registry»

Это разные контексты одного слова.

---

## 10. Минимальная модель после урока

```text
ADDON DIRECTORY
      │
      ├── __manifest__.py
      │      └── dependency declaration
      │
      └── __init__.py
             └── Python imports
                    │
                    ▼
              model classes
                    │
        installed module graph
                    │
                    ▼
                 DATABASE
                    │
                    ▼
        available/effective models
                    │
                    ▼
              ORM recordsets
```

---

## 11. Что пока сознательно не разбираем

- `_name`, `_inherit`, `_inherits` подробно;
- model metaclass implementation;
- registry internal lifecycle;
- field registration;
- database schema synchronization;
- data file loading details;
- module upgrade migrations;
- server workers/process architecture.

Каждая из этих тем требует своего контекста.

---

## 12. Контрольные вопросы

1. Чем `__manifest__.py` отличается от `__init__.py`?
2. Зачем нужен `models/__init__.py`?
3. Почему наличие Python file не равно выполненному model declaration?
4. Чем intra-module import order отличается от inter-module dependency order?
5. Почему installed module graph может различаться между databases одного process?
6. Что ORM instantiates once per database?
7. От чего зависит available model set database?
8. Почему actual model class может собираться из нескольких Python classes?
9. Почему слово registry надо употреблять с контекстом?

---

## Официальные источники урока

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

2. A New Application  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/02_newapp.html

3. Models and Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

4. Module Manifests  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html

5. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
