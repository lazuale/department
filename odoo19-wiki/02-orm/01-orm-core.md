# 01. ORM Core

> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: architecture foundation; module loading  
> Владеет понятиями: Model/TransientModel/AbstractModel; record/recordset/singleton; Environment basic semantics; browse/exists/ensure_one; search/basic domain; CRUD  
> Preview: fields; relations; security context; transactions  
> Отложено: field taxonomy; relations; computed behavior; `sudo`; multi-company; cache/prefetch; raw SQL; transactions details  
> Edition scope: platform ORM semantics

## Цель

Понять минимальный рабочий язык backend Odoo до углубления в fields, security, transactions и performance.

---

## 1. ORM — основной server-side data API Odoo

**[ODOO]** ORM является ключевым компонентом Odoo framework. Business objects объявляются Python classes, наследующими ORM model classes.

```text
Python model declarations
        │
        ▼
      Odoo ORM
        │
        ▼
     PostgreSQL
```

**[ВЫВОД]** Нормальная server-side business logic строится вокруг ORM models/recordsets, а не вокруг manual SQL как основного API.

---

## 2. Три базовых model classes

**[ODOO]** Основные ORM base classes:

```text
BaseModel
├── Model
├── TransientModel
└── AbstractModel
```

### `Model`
Regular database-persisted model superclass.

### `TransientModel`
Temporary model: records хранятся в database, но periodically vacuumed.

### `AbstractModel`
Abstract superclass для reusable behavior.

**[ВЫВОД]** «Документ», «справочник», «ERP master data» и «transaction» не являются альтернативными ORM base classes.

---

## 3. Model — не просто SQL table

**[ODOO]** Model definition включает fields и methods; ORM обеспечивает persistence/framework behavior. Actual model class может быть составлен extensions нескольких Python classes/modules.

Минимально:

```text
MODEL
├── technical identity
├── fields
├── methods
└── ORM behavior / extensions
```

**[ВЫВОД]** `ORM model = SQL table` недостаточно как архитектурное объяснение.

Physical field/storage semantics принадлежат следующему owner-уроку.

---

## 4. Recordset — основной рабочий объект ORM

**[ODOO]** Interactions with models and records выполняются через recordsets — ordered collections of records одной model.

Model methods работают на recordset; `self` является recordset.

```python
class Example(models.Model):
    _name = 'example.model'

    def do_something(self):
        # self is a recordset
        ...
```

`self` может содержать:

```text
0 records
1 record
N records
```

Implementation caveats, не необходимые для core mental model, будут помещаться в advanced/reference notes, а не в фундамент.

---

## 5. Один record — singleton recordset

**[ODOO]** Record не имеет отдельного explicit object representation: один record представлен recordset из одного элемента.

```text
one record
=
singleton recordset
```

При итерации:

```python
for record in records:
    ...
```

каждый `record` — singleton recordset.

**[ВЫВОД]** В Odoo не существует второго независимого ORM API «для одной строки» поверх recordset API.

---

## 6. `ids`, `exists()` и `ensure_one()`

### `ids`

**[ODOO]** `records.ids` возвращает identifiers records текущего recordset.

### `exists()`

**[ODOO]** Возвращает subset records, которые существуют.

```python
records = model.browse([7, 18, 999999])
existing = records.exists()
```

**[ВЫВОД]** `browse(ids)` создаёт ORM representation для identifiers; наличие id в browsed recordset само по себе не является проверкой существования corresponding row/record.

### `ensure_one()`

**[ODOO]** Проверяет, что recordset содержит ровно один record, иначе raises `ValueError`.

```python
self.ensure_one()
```

**[ВЫВОД]** Если method требует singleton, это должно быть выражено явно, а не предполагаться из слова `self`.

---

## 7. Field access — только preview

Fields имеет отдельный owner.

На текущем уровне достаточно:

**[ODOO]** Fields объявляются на model и доступны через recordset API.

```python
record.name
```

**[ВЫВОД]** ORM field не равен visual input/widget form view.

Taxonomy, metadata, storage и automatic/reserved fields будут в `02-fields.md`.

---

## 8. Environment — runtime context ORM

**[ODOO]** Environment хранит contextual data, используемые ORM:

- `cr` — database cursor;
- `uid` — current user id;
- `context` — context dictionary;
- `su` — superuser-mode state.

Environment также предоставляет access к models по model name и содержит runtime structures ORM.

Минимальная mental model:

```text
Environment
├── database cursor
├── current user id
├── context
└── superuser-state flag
```

Security meaning `uid/su`, company context, cache и recomputation **не определяются здесь**.

---

## 9. Model access через Environment

**[ODOO]** Environment поддерживает mapping-like access:

```python
partners = self.env['res.partner']
```

Это даёт recordset указанной model в текущем Environment.

Далее:

```python
partners.browse(...)
partners.search(...)
partners.create(...)
```

**[ВЫВОД]** Полезная mental model runtime recordset:

```text
model + record identities + Environment
```

Это conceptual aid, не formal internal object layout.

---

## 10. `browse()` — recordset по известным identifiers

**[ODOO]** `browse(ids)` возвращает recordset для supplied ids в current Environment.

```python
partners = self.env['res.partner'].browse([7, 18, 12])
```

```text
known identifiers
      │
      ▼
   browse()
      │
      ▼
  recordset
```

`browse()` не выполняет business-criteria search. Существование проверяется `exists()`.

---

## 11. `search()` и basic domain semantics

**[ODOO]** `search(domain)` возвращает records model, удовлетворяющие search domain.

```python
records = self.env['example.model'].search([
    ('active', '=', True),
])
```

Пустой domain:

```python
[]
```

означает отсутствие filter criteria; фактическая доступность records также зависит от security/runtime context, owner которого будет позже.

### Criterion

```python
(field_name, operator, value)
```

Примеры:

```python
('active', '=', True)
('amount', '>', 1000)
```

Несколько обычных criteria комбинируются логическим AND; более сложные logical expressions поддерживаются domain operators.

**[ВЫВОД]** Domain — ORM expression language, не SQL WHERE string.

Relation traversal и domains в UI принадлежат будущим owner-урокам.

---

## 12. `create()`

**[ODOO]** `Model.create(vals_list)` создаёт records и возвращает created recordset.

```python
records = model.create([
    {'name': 'A'},
    {'name': 'B'},
])
```

Для compatibility допустим один dictionary.

**[ВЫВОД]** Base API изначально поддерживает multi-record/batch semantics.

Defaults/computed behavior разбираются позже.

---

## 13. `read()`

**[ODOO]** `read(fields)` возвращает requested values records как list of dictionaries.

```python
values = records.read(['name', 'active'])
```

Концептуально:

```python
[
    {'id': 1, 'name': 'A', 'active': True},
    {'id': 2, 'name': 'B', 'active': False},
]
```

**[ВЫВОД]** `read()` structured representation и normal attribute field access — разные API forms ORM.

---

## 14. `write()`

**[ODOO]** `write(vals)` обновляет все records текущего `self`.

```python
records.write({'active': False})
```

**[ВЫВОД]** Model methods и CRUD naturally работают с recordsets, не обязательно с single record.

---

## 15. `unlink()`

**[ODOO]** `unlink()` удаляет records current `self`.

```python
records.unlink()
```

Security/business restrictions возможны, но их semantics имеют другие owner-уроки.

---

## 16. CRUD не равен четырём SQL statements

Условная аббревиатура CRUD удобна, но:

```text
ORM create/read/write/unlink
        ≠
ручные INSERT/SELECT/UPDATE/DELETE
```

**[ВЫВОД]** ORM operations существуют внутри framework model/environment semantics. Transactions, security, computed behavior и cache будут добавляться к этой mental model по мере прохождения owners.

---

## 17. Минимальный рабочий цикл

```text
Environment
     │
     └── env['model.name']
              │
              ▼
          recordset
              │
      ┌───────┼────────┐
      │       │        │
   browse   search   create
      │       │        │
      └───────┼────────┘
              ▼
          recordset
              │
       read/write/unlink
```

---

## Что нельзя заключать

- `self` всегда один record — нет;
- `browse(id)` проверяет existence — нет;
- model = SQL table — нет;
- field = form widget — нет;
- domain = SQL WHERE string — нет;
- CRUD = raw SQL — нет;
- Environment = process-global state — нет;
- `sudo`, company, cache и transaction semantics уже понятны после этого урока — нет.

## Контрольные вопросы

1. Какие три base model classes используются ORM?
2. Что такое recordset?
3. Почему single record — singleton recordset?
4. Что делает `ensure_one()`?
5. Чем `browse()` отличается от `search()`?
6. Зачем нужен `exists()`?
7. Что такое basic domain criterion?
8. Что минимально содержит Environment?
9. Что делает `env['model.name']`?
10. На сколько records действует `write()`?
11. Почему CRUD нельзя свести к raw SQL?

## Официальные источники

1. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
2. Models and Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html