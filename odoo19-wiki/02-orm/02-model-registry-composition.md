# ORM-02. Per-database model registry и model composition

> Lesson ID: `ORM-02`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ARCH-01`, `ARCH-02`, `ARCH-03`, `ORM-01`  
> Canonical owner: backend model registry context; per-database model construction; effective model class  
> Aspect owner: inheritance mechanics → `EXT-01`  
> Preview: model inheritance  
> Отложено: `_inherit`; `_inherits`; metaclasses; schema synchronization; registry internal lifecycle; workers  
> Edition scope: platform ORM semantics  
> Sources: `S1`, `S2`

## Цель

Теперь, когда `Model`, recordset и Environment уже определены, можно корректно связать Python declarations с per-database ORM model set.

Главная граница:

```text
Python class/declaration known to runtime
        ≠
ORM model available in every database
```

---

## 1. Models существуют в database context

**[ODOO][S1]** ORM автоматически instantiates available models once per database. Набор available models зависит от modules, установленных на этой database.

```text
same Odoo runtime
      │
      ├── DB A → installed modules A → models A
      └── DB B → installed modules B → models B
```

**[ВЫВОД]** Model availability — per-database concept, а не просто список Python files/classes server filesystem.

---

## 2. Почему ARCH-03 не мог владеть этим concept

`ARCH-03` объясняет только:

```text
addon package
→ __init__.py import chain
→ Python declarations execute
```

Но только после `ORM-01` у нас есть definitions:

- `Model`;
- recordset;
- Environment;
- `env['model.name']`.

Поэтому registry/model composition находится здесь, а не до ORM Core.

---

## 3. Backend model registry context

**[ODOO][S1]** Environment предоставляет mapping-like access к models по technical model name:

```python
env['res.partner']
```

ORM documentation описывает registry/model mapping в database/environment context.

В курсе термин **backend model registry** означает:

> per-database backend mapping/context доступных ORM models.

Мы не используем это выражение как обещание exact undocumented internal implementation.

---

## 4. Backend registry не равен frontend registry

**[ODOO][S2]** Frontend framework отдельно использует registries для расширения web client.

```text
backend model registry context
        ≠
frontend JavaScript registries
```

Frontend registries принадлежат `RUN-03`.

---

## 5. Effective model class может быть composite

**[ODOO][S1]** ORM Reference говорит, что actual class model instance строится из Python classes, которые создают и наследуют соответствующую model.

Концептуально:

```text
module A Python class creates model X
              │
module B Python class inherits/extends X
              │
module C Python class inherits/extends X
              │
              ▼
effective model X class in database context
```

**[ВЫВОД]** Нельзя считать effective ORM model эквивалентом единственного Python class definition одного addon.

Exact `_inherit`/`_inherits` mechanics принадлежат `EXT-01`.

---

## 6. Modules и database configuration связывают declarations с effective models

Из предыдущих owners:

```text
ARCH-02: database-bound installed module graph
ARCH-03: Python import chain
ORM-01: Model / Environment
```

Теперь можно собрать:

```text
addon Python declarations
        │
installed module graph of DB X
        │
        ▼
per-database effective models
        │
        ▼
Environment model access / recordsets
```

Это conceptual model, не exact loader trace.

---

## 7. Multi-database consequence

**[ODOO][S1]** Один Odoo process может обслуживать databases с разными installed modules/models.

**[ВЫВОД]** Module-dependent/model-dependent state нельзя мыслить как универсальное mutable process-global property.

Это связывает `ARCH-01` multi-database principle с ORM runtime semantics.

---

## 8. Что сознательно не моделируем

Не строим undocumented sequence:

```text
scan → import → metaclass hook → registry mutation → schema sync → ...
```

До отдельного official evidence нам достаточно contracts:

- imports execute declarations;
- models are available per database;
- Environment maps technical names to models;
- effective model class may compose multiple Python classes.

---

## Минимальная модель

```text
Python addon declarations
        │
        ▼
Database X installed module set
        │
        ▼
backend per-database model mapping
        │
        ▼
effective ORM model classes
        │
        ▼
Environment / recordsets
```

## Что нельзя заключать

- every Python model declaration is available in every database — нет;
- backend registry = frontend registry — нет;
- one ORM model = one Python class — нет;
- ORM-02 уже объясняет `_inherit` mechanics — нет;
- схема выше является exact internal loader implementation — нет.

## Контрольные вопросы

1. Почему model availability является per-database concept?
2. Почему ORM registry нельзя корректно объяснить до ORM Core?
3. Что означает backend model registry в этом курсе?
4. Почему backend и frontend registries нельзя смешивать?
5. В каком смысле effective model class может быть composite?
6. Почему process-global installed-module state опасен при multi-database runtime?

## Официальные источники

- `S1` — ORM API  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- `S2` — Frontend Registries  
  https://www.odoo.com/documentation/19.0/developer/reference/frontend/registries.html