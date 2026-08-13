# 03. Загрузка модулей и per-database models

> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: architecture foundation; module system  
> Владеет понятиями: Python import chain; backend model registry context; per-database model construction  
> Preview: model inheritance  
> Отложено: `_inherit`/`_inherits`; metaclasses; schema synchronization; workers; migrations  
> Edition scope: platform semantics

## Цель

Понять границу между:

```text
Python code/package exists
        ≠
Python declaration imported
        ≠
model available in every database
```

и не придумывать undocumented internal loader mechanics.

---

## 1. `__manifest__.py` и `__init__.py` имеют разные роли

Из предыдущего урока:

```text
__manifest__.py
= Odoo module metadata / dependencies / data

__init__.py
= Python package import structure
```

**[ODOO]** Official tutorial создаёт module package с обоими файлами.

**[ODOO]** Когда появляются Python subpackages/files, `__init__.py` импортирует их.

Пример:

```python
# module/__init__.py
from . import models
```

```python
# module/models/__init__.py
from . import example
```

---

## 2. Наличие `.py` file не равно выполненному declaration

```text
models/example.py exists
        │
        ▼
Python import chain reaches file?
     ┌──────┴──────┐
    no             yes
    │               │
file is not      declarations execute
imported via     in Python runtime
package chain
```

**[ВЫВОД]** Filesystem organization и executed Python declarations — разные уровни.

---

## 3. Intra-module imports и inter-module dependencies — разные механизмы

```text
INTRA-MODULE
__init__.py → Python imports

INTER-MODULE
manifest `depends` → Odoo dependency order
```

**[ODOO]** Dependencies должны быть loaded before dependent module.

**[ВЫВОД]** Нельзя объяснять `depends` как обычный Python import и нельзя заменять manifest dependency импортом соседнего addon package.

---

## 4. Python runtime и database configuration не совпадают

**[ODOO]** Один Odoo Python process может обслуживать databases с разными installed module sets.

**[ODOO]** ORM instantiates every available model once per database; available model set зависит от installed modules этой database.

Это требует жёстко удерживать различие:

```text
Python code is present/importable in server runtime
                ≠
model is available in Database X
```

**[ВЫВОД]** Database-level model availability нельзя выводить только из того, какие Python files существуют или уже известны process.

---

## 5. Per-database backend model mapping / registry context

**[ODOO]** `Environment` позволяет обращаться к models по technical name:

```python
env['res.partner']
```

и ORM documentation говорит о registry/model mapping в контексте database/environment.

В курсе термин **backend model registry** означает:

> backend context/mapping моделей, доступных конкретной database/runtime Environment.

Мы сознательно **не утверждаем** здесь детали внутреннего lifecycle/metaclass implementation, если они не нужны для documented semantics.

Также не смешиваем этот термин с frontend registries Odoo web client — у них другой owner.

---

## 6. Actual model class может быть composite

**[ODOO]** ORM reference указывает, что actual class model instance строится из Python classes, которые создают и наследуют эту model.

Концептуально:

```text
module A class creates model X
          │
module B class extends/inherits X
          │
module C class extends/inherits X
          │
          ▼
effective model X in database context
```

Подробная semantics `_inherit` и `_inherits` принадлежит model-inheritance lesson.

**[ВЫВОД]** Итоговую model нельзя считать одним единственным Python class из одного addon.

---

## 7. Почему mutable globals опасны

**[ODOO]** Server Framework 101 предупреждает не использовать mutable global variables, зависящие от installed modules, поскольку один process может обслуживать databases с разными module configurations.

Пример неправильной mental model:

```text
global variable says module X installed
        ↓
assumed true for whole process
```

но process может одновременно обслуживать:

```text
DB A → X installed
DB B → X not installed
```

**[ВЫВОД]** Module/database-dependent state должен мыслиться в database/runtime context, а не как универсальное process-global property.

---

## 8. Что мы не моделируем сейчас

Не утверждаем exact sequence вроде:

```text
scan → import → metaclass hook → registry mutation → schema sync → ...
```

как официальную внутреннюю диаграмму, пока документация курса этого не установила.

Нам достаточно документированных contracts:

- module package/import structure;
- dependency order;
- per-database available models;
- composite actual model class.

---

## 9. Минимальная модель после урока

```text
ADDON PACKAGE
    │
    ├── __manifest__.py → Odoo dependency/data declaration
    └── __init__.py     → Python import chain
                              │
                              ▼
                       Python model classes
                              │
                   installed module graph
                              │
                              ▼
                         Database X
                              │
                              ▼
                 backend available models
                              │
                              ▼
                         ORM recordsets
```

---

## Что нельзя заключать

- every `.py` file in addon is automatically executed — нет;
- manifest imports Python files — нет;
- every model class known to process is available in every database — нет;
- one ORM model = one Python class — нет;
- backend model registry = frontend registry — нет;
- этот урок описывает exact internal registry/metaclass lifecycle — нет.

## Контрольные вопросы

1. Чем manifest отличается от `__init__.py`?
2. Что такое intra-module import chain?
3. Чем он отличается от manifest dependencies?
4. Почему Python code presence не равно model availability в database?
5. Что означает per-database model construction?
6. В каком смысле actual model class может быть composite?
7. Почему installed-module-dependent globals опасны в multi-database process?
8. Как в курсе употребляется backend model registry?

## Официальные источники

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
6. Frontend registries (для разграничения терминов)  
   https://www.odoo.com/documentation/19.0/developer/reference/frontend/registries.html