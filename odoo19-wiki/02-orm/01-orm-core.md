# ORM-01. ORM Core

> Lesson ID: `ORM-01`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ARCH-01`, `ARCH-02`, `ARCH-03`  
> Canonical owner: Model/TransientModel/AbstractModel; record/recordset/singleton; Environment basic semantics; browse/exists/ensure_one; search/basic domain; CRUD  
> Aspect owner: —  
> Preview: fields; relations; transactions; security context  
> Отложено: backend model registry/composition; field taxonomy; relations; computed behavior; `sudo`; multi-company; cache/prefetch; raw SQL; transaction details  
> Edition scope: platform ORM semantics  
> Sources: `S1`, `S2`

## Цель

Понять минимальный рабочий язык backend Odoo, не забегая в registry composition, fields, security, transactions и performance.

---

## 1. ORM — основной server-side data API

**[ODOO][S1]** ORM является ключевым компонентом Odoo framework; business objects объявляются Python classes, наследующими ORM model classes.

```text
Python model declaration
        │
        ▼
      Odoo ORM
        │
        ▼
     PostgreSQL
```

**[ВЫВОД]** Normal server-side business logic строится вокруг ORM models/recordsets, а manual SQL не является базовой application API model.

---

## 2. Три основных model classes

**[ODOO][S1]** Основные ORM base classes:

```text
BaseModel
├── Model
├── TransientModel
└── AbstractModel
```

- `Model` — regular database-persisted model superclass;
- `TransientModel` — temporary records, stored in database and periodically vacuumed;
- `AbstractModel` — abstract superclass for reusable behavior.

**[ВЫВОД]** «Документ», «справочник», `ERP master data` и business transaction не являются альтернативными ORM base classes.

---

## 3. Model — ORM concept, не просто SQL table

**[ODOO][S1][S2]** Model definitions содержат fields и methods; ORM обеспечивает persistence/framework semantics.

На этом уровне достаточно:

```text
MODEL
├── technical identity
├── fields
├── methods
└── ORM behavior
```

**[ВЫВОД]** `ORM model = SQL table` недостаточно как architecture explanation.

Как несколько Python classes/modules составляют effective model, принадлежит `ORM-02` и `EXT-01`.

---

## 4. Recordset — основной рабочий объект ORM

**[ODOO][S1]** Interactions with models and records выполняются через recordsets — ordered collections of records одной model. Model methods работают на recordset; `self` является recordset.

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

Implementation caveats не входят в core mental model.

---

## 5. Один record — singleton recordset

**[ODOO][S1]** Record не имеет отдельного explicit object representation: один record представлен recordset из одного элемента.

```text
one record = singleton recordset
```

При итерации по recordset каждый `record` в цикле является singleton recordset.

---

## 6. `ids`, `exists()` и `ensure_one()`

**[ODOO][S1]**

- `records.ids` возвращает record identifiers;
- `records.exists()` возвращает subset существующих records;
- `ensure_one()` проверяет singleton и иначе raises `ValueError`.

```python
records = model.browse([7, 18, 999999])
existing = records.exists()
```

**[ВЫВОД]** `browse(ids)` создаёт ORM representation identifiers; это не independent existence check.

---

## 7. Fields — только preview

**[ODOO][S1]** Fields объявляются на model и доступны через recordset API.

```python
record.name
```

**[ВЫВОД]** ORM field не равен visual input/widget form view.

Field taxonomy/storage принадлежат `ORM-03`.

---

## 8. Environment — runtime context ORM

**[ODOO][S1]** Environment хранит contextual data ORM, включая:

- `cr` — database cursor;
- `uid` — current user id;
- `context` — context dictionary;
- `su` — superuser-mode state.

Environment также предоставляет model access по technical model name.

Минимальная mental model:

```text
Environment
├── database cursor
├── current user id
├── context
└── superuser-state flag
```

Security/company/cache aspects принадлежат другим owners.

---

## 9. Model access через Environment

**[ODOO][S1]** Mapping-like access:

```python
partners = self.env['res.partner']
```

даёт recordset указанной model в текущем Environment.

Далее возможны:

```python
partners.browse(...)
partners.search(...)
partners.create(...)
```

Backend registry/per-database composition за этим access будет разобран в `ORM-02`.

---

## 10. `browse()`

**[ODOO][S1]** `browse(ids)` возвращает recordset для supplied identifiers в current Environment.

```text
known identifiers
      │
      ▼
   browse()
      │
      ▼
  recordset
```

`browse()` не выполняет business-criteria search.

---

## 11. `search()` и basic domain

**[ODOO][S1]** `search(domain)` возвращает records model, удовлетворяющие search domain.

```python
records = self.env['example.model'].search([
    ('active', '=', True),
])
```

Basic criterion:

```python
(field_name, operator, value)
```

Multiple plain criteria комбинируются AND; logical operators позволяют строить более сложные expressions.

**[ВЫВОД]** Domain — ORM expression language, не SQL WHERE string.

Relation traversal и UI domains принадлежат будущим aspect owners.

---

## 12. `create()`

**[ODOO][S1]** `Model.create(vals_list)` создаёт records и возвращает created recordset.

```python
records = model.create([
    {'name': 'A'},
    {'name': 'B'},
])
```

Single dictionary также поддерживается для compatibility.

---

## 13. `read()`

**[ODOO][S1]** `read(fields)` возвращает requested values как list of dictionaries.

```python
values = records.read(['name', 'active'])
```

---

## 14. `write()`

**[ODOO][S1]** `write(vals)` обновляет все records текущего `self`.

```python
records.write({'active': False})
```

---

## 15. `unlink()`

**[ODOO][S1]** `unlink()` удаляет records current `self`.

Security/business restrictions существуют, но их semantics имеют другие owners.

---

## 16. CRUD не равен raw SQL

```text
ORM create/read/write/unlink
        ≠
manual INSERT/SELECT/UPDATE/DELETE
```

**[ВЫВОД]** ORM operations выполняются внутри model/environment/framework semantics. Transactions, security, computed behavior и cache будут добавлены следующими layers.

---

## Минимальная модель

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

## Что нельзя заключать

- `self` всегда один record — нет;
- `browse(id)` проверяет existence — нет;
- model = SQL table — нет;
- field = form widget — нет;
- domain = SQL WHERE string — нет;
- CRUD = raw SQL — нет;
- Environment = process-global state — нет;
- per-database registry/model composition уже объяснены — нет.

## Контрольные вопросы

1. Какие три base model classes используются ORM?
2. Что такое recordset?
3. Почему one record — singleton recordset?
4. Что делает `ensure_one()`?
5. Чем `browse()` отличается от `search()`?
6. Зачем нужен `exists()`?
7. Что такое basic domain criterion?
8. Что минимально содержит Environment?
9. Что делает `env['model.name']`?
10. Почему CRUD нельзя свести к raw SQL?

## Официальные источники

- `S1` — ORM API  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- `S2` — Models and Basic Fields  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html