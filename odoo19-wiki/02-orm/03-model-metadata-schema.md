# ORM-03. Model metadata, SQL storage и schema declarations

> Lesson ID: `ORM-03`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ORM-02`  
> Canonical owner: model technical metadata; SQL storage options; schema-level `Constraint` / `Index` / `UniqueIndex` declarations  
> Aspect owner: migrations/schema changes → `RUN-05`; performance aspects of indexes → `RUN-01`; inheritance attributes → `EXT-01`; multi-company metadata → `EXT-04`  
> Preview: fields; automatic/access-log fields  
> Отложено: field taxonomy; relational fields; `_inherit`/`_inherits`; company consistency; migration mechanics; performance tuning; raw SQL  
> Edition scope: platform ORM semantics; concrete application entitlement здесь не определяется  
> Sources: `S1`, `S2`, `S3`

## Цель

После `ORM-01` мы знаем, что такое Model/recordset/Environment, а после `ORM-02` — что effective models существуют в per-database ORM context.

Теперь нужно понять ещё один отдельный слой:

```text
MODEL
  │
  ├── technical identity / metadata
  ├── SQL persistence policy
  ├── table-level behavior
  └── schema declarations
```

Этот слой нельзя смешивать ни с field API, ни с inheritance, ни с migration tooling.

---

## 1. Model class настраивается не только fields и methods

**[ODOO][S1]** ORM Reference документирует ряд model attributes, которые определяют technical identity, persistence и default behavior модели.

**[ODOO][S2]** Server Framework 101 отдельно подчёркивает, что models настраиваются attributes в class definition, а `_name` является ключевым атрибутом модели.

Минимально:

```python
from odoo import models

class Example(models.Model):
    _name = 'example.model'
    _description = 'Example Model'
```

**[ВЫВОД]** Model definition в Odoo — это не только набор `fields.*`: сама model имеет metadata/configuration attributes.

---

## 2. `_name` — technical model name

**[ODOO][S1]** `_name` задаёт имя model в dot-notation / module namespace.

**[ODOO][S2]** Для новой обычной model tutorial называет `_name` обязательным и использует его как имя model в системе.

```python
_name = 'example.model'
```

Это имя используется ORM/API как technical identity:

```python
env['example.model']
```

Из `ORM-01` уже известно, что `env['model.name']` возвращает recordset соответствующей model.

**[ВЫВОД]** `_name` — не UI label и не SQL table name как таковой. Это technical identity ORM model.

---

## 3. `_description` — informal/human-readable model name

**[ODOO][S1]** `_description` — informal name model.

```python
_description = 'Example Model'
```

Различаем:

```text
_name
= technical ORM identity

_description
= informal/human-readable description
```

**[ВЫВОД]** Нельзя использовать `_description` как replacement technical name модели.

---

## 4. `_auto` определяет automatic table creation

**[ODOO][S1]** Для regular `Model` `_auto` имеет значение `True`; этот атрибут определяет, должна ли ORM создавать database table автоматически.

При:

```python
_auto = True
```

обычная persistence model использует ORM-managed table creation.

При:

```python
_auto = False
```

ORM не должна автоматически создавать table; documentation указывает переопределить `init()` для создания database table, если table всё же нужна.

**[ODOO][S1]** Для model без собственной table documentation рекомендует `AbstractModel`.

**[ВЫВОД]** `_auto=False` не означает автоматически «эта model вообще не имеет storage». Это означает, что automatic table creation ORM отключена; дальнейшая storage strategy требует отдельного определения.

---

## 5. `_table` — SQL table name, когда model использует table

**[ODOO][S1]** `_table` задаёт SQL table name, используемое model при `_auto`.

Минимальная граница:

```text
ORM model name
_name = 'example.model'

SQL persistence name
_table = ...
```

**[ODOO][S2]** Tutorial показывает обычный случай: достаточно объявить `_name`, после чего ORM создаёт соответствующую database table для regular model.

**[ВЫВОД]** Model name и table name связаны framework conventions, но являются разными concepts. Нельзя строить универсальную формулу:

```text
ORM model = SQL table
```

Это уже было запрещено в `ORM-01`; здесь мы видим техническую причину глубже.

---

## 6. `_register` — registry visibility, а не installation state

**[ODOO][S1]** `_register` относится к registry visibility; documentation также говорит, что для class, которую не следует instantiate/register как model, `_register` можно установить в `False`.

Это **не** то же самое, что:

```text
module installed / not installed
```

и не то же самое, что database lifecycle module из `ARCH-02`.

**[ВЫВОД]** `_register` — model-class/runtime metadata. Не используем его как surrogate module state.

Exact internal registry mechanics остаются за пределами этого урока.

---

## 7. `_log_access` управляет access-log fields на уровне model

**[ODOO][S1]** `_log_access` определяет, должна ли ORM автоматически создавать/обновлять Access Log fields.

К ним относятся, например:

```text
create_date
create_uid
write_date
write_uid
```

**[ODOO][S1]** Значение `_log_access` по умолчанию связано с `_auto`; для `TransientModel` `_log_access` должен быть enabled.

**[ODOO][S2]** Tutorial показывает, что обычная persistent model получает дополнительные automatic fields в table, хотя разработчик их явно не объявлял.

Полная semantics automatic fields принадлежит `ORM-04 Fields`.

**[ВЫВОД]** Access-log columns — пример того, как model-level metadata влияет на physical schema без явного field declaration в конкретном class body.

---

## 8. `_rec_name` определяет representative field для labeling records

**[ODOO][S1]** `_rec_name` задаёт field, используемый для labeling records; default — `name`.

```python
_rec_name = 'code'
```

может изменить representative field model.

Важно:

```text
_rec_name
≠
technical model name `_name`
```

**[ВЫВОД]** Technical identity модели и human-facing identity конкретного record — разные уровни.

Exact `display_name`/field behavior будет подробно разобрано в `ORM-04`.

---

## 9. `_order` задаёт default ordering search results

**[ODOO][S1]** `_order` задаёт default order field/expression для search results; documentation указывает default `id`.

```python
_order = 'name'
```

**[ВЫВОД]** Порядок records по умолчанию — model metadata, а не свойство конкретного list view.

UI может иметь свои sorting aspects позже, но default ORM ordering существует отдельно.

---

## 10. Иерархическая metadata: `_parent_name` и `_parent_store`

**[ODOO][S1]** `_parent_name` определяет Many2one field, используемый как parent field; default — `parent_id`.

**[ODOO][S1]** `_parent_store=True` вместе с `parent_path` организует indexed storage tree structure и ускоряет hierarchical domain operators `child_of` / `parent_of`.

```text
_parent_name
→ какой relation field является parent

_parent_store
→ хранить optimized hierarchy path
```

Relations и domain traversal принадлежат `ORM-05`; field `parent_path` и indexing подробнее будут использованы там/в performance aspect.

**[ВЫВОД]** Tree/hierarchy behavior Odoo может включать model-level metadata и storage optimization, а не только визуальный tree widget.

---

## 11. `_fold_name` — metadata для folded groups

**[ODOO][S1]** `_fold_name` задаёт field, используемый для определения folded groups в kanban views; default — `fold`.

Это хороший пример cross-layer metadata:

```text
model attribute
      │
      ▼
field semantics
      │
      ▼
UI behavior
```

Полная kanban/view semantics принадлежит `UI-02`.

**[ВЫВОД]** Не вся UI-related behavior живёт исключительно внутри XML view definitions.

---

## 12. Что сознательно не разбираем среди model attributes

ORM Reference рядом документирует:

```text
_inherit
_inherits
_check_company_auto
_abstract
_transient
```

Но ownership уже установлен:

- `_inherit` / `_inherits` → `EXT-01`;
- company consistency / `_check_company_auto` → `EXT-04`;
- различие `Model` / `TransientModel` / `AbstractModel` → уже `ORM-01`.

**[ВЫВОД]** Наличие attribute в одном reference section не означает, что один lesson должен полностью владеть всеми его semantics.

---

## 13. Constraints и indexes в Odoo 19 объявляются как model attributes

**[ODOO][S1]** ORM Reference позволяет declaratively объявлять:

```text
Constraint
Index
UniqueIndex
```

Имена class attributes для таких declarations должны начинаться с `_`, чтобы не конфликтовать с field names.

Пример из documented pattern:

```python
class Example(models.Model):
    _name = 'example.model'

    _positive = models.Constraint(
        'CHECK (amount > 0)',
        'Amount must be positive',
    )

    _name_idx = models.Index('(name)')
```

**[ODOO][S3]** Official tutorial также разделяет Python constraints и SQL/schema constraints/indexes и использует `Constraint`, `Index`, `UniqueIndex` для последних.

---

## 14. Schema constraint ≠ Python constraint

В этом уроке owner — только schema-level declarations.

```text
models.Constraint(...)
= database/schema-level invariant declaration

@api.constrains(...)
= Python ORM validation method
```

Python constraints принадлежат `ORM-06`.

**[ВЫВОД]** Одинаковое слово «constraint» не означает один и тот же execution mechanism.

Это разграничение обязательно сохраняется дальше по курсу.

---

## 15. `Index` и `UniqueIndex` — schema tools, не business classification

**[ODOO][S1]** ORM Reference отдельно документирует index declarations наряду с constraints.

**[ODOO][S3]** Tutorial прямо говорит, что Odoo позволяет объявлять SQL constraints и более complex SQL indexes.

Минимально:

```text
Index
→ структура database для поиска/access patterns

UniqueIndex
→ index с uniqueness semantics
```

Performance trade-offs индексов принадлежат `RUN-01`.

Migration/update behavior schema declarations принадлежит `RUN-05`.

**[ВЫВОД]** Наличие index declaration в model class не означает, что текущий lesson уже определяет production indexing strategy.

---

## 16. Model metadata и Fields — разные owners

После этого урока схема должна быть такой:

```text
MODEL CLASS
│
├── model metadata / storage options       → ORM-03
│
├── fields / automatic fields              → ORM-04
│
├── relational fields                      → ORM-05
│
├── computed / related / Python constraints→ ORM-06
│
└── methods / runtime behavior             → previous/later owners
```

**[ВЫВОД]** Это разделение защищает курс от повторного превращения «Fields» в свалку всей ORM schema semantics.

---

## 17. Минимальная mental model

```text
                ORM MODEL
                    │
      ┌─────────────┼─────────────┐
      │             │             │
 technical       persistence    schema
 metadata         policy        declarations
      │             │             │
 _name          _auto          Constraint
 _description   _table         Index
 _rec_name      _log_access    UniqueIndex
 _order
 _register
 hierarchy/UI-related metadata
      │
      ▼
 PostgreSQL schema + ORM runtime semantics
```

Это conceptual map. Она не является exact internal metaclass/schema-update algorithm.

---

## Что нельзя заключать

- `_name` = SQL table name — нет;
- `_description` = technical model identifier — нет;
- `_auto=False` автоматически означает model без storage — нет;
- `_register` = module installation state — нет;
- `_order` = только list-view sorting — нет;
- `_parent_store` = визуальный tree widget — нет;
- `models.Constraint` = `@api.constrains` — нет;
- наличие `Index` означает, что performance strategy уже определена — нет;
- этот урок объясняет inheritance или multi-company semantics — нет.

## Контрольные вопросы

1. Чем `_name` отличается от `_description`?
2. Что контролирует `_auto`?
3. Что означает `_table`?
4. Почему `_register` нельзя смешивать с module installed state?
5. Что контролирует `_log_access`?
6. Чем `_rec_name` отличается от `_name`?
7. Что задаёт `_order`?
8. Для чего нужны `_parent_name` и `_parent_store`?
9. Чем `models.Constraint` отличается от `@api.constrains`?
10. Какие schema declarations Odoo 19 документирует вместе с constraints/indexes?
11. Почему performance и migration aspects indexes не принадлежат ORM-03?

## Официальные источники

- `S1` — ORM API  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- `S2` — Server Framework 101: Models And Basic Fields  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
- `S3` — Building a Module: Model constraints  
  https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html
