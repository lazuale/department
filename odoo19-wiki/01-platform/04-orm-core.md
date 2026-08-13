# Урок 4. ORM Core: Model, recordset, Environment, search и CRUD

## Цель урока

Понять минимальный рабочий язык backend Odoo до углубления в fields, relations, security и performance.

После урока должны быть понятны:

- `Model`, `TransientModel`, `AbstractModel`;
- recordset и singleton;
- `self` как recordset;
- `Environment` как runtime-context ORM;
- `browse()`, `exists()`, `ensure_one()`;
- `search()` и domain;
- `create()`, `read()`, `write()`, `unlink()`.

Cache, prefetch, `sudo()`, multi-company, raw SQL, flush и recomputation здесь **не разбираются**.

---

## 1. ORM — основной server-side API данных

**[ODOO]** Odoo ORM является ключевым компонентом framework. Business objects объявляются Python classes, наследующими ORM model classes.

```text
Python model declarations
        │
        ▼
      Odoo ORM
        │
        ▼
     PostgreSQL
```

**[ВЫВОД]** Нормальная server-side business logic Odoo строится вокруг models и recordsets, а не вокруг ручного SQL как основного API.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 2. Три базовых model classes

**[ODOO]** Odoo models создаются наследованием одного из трёх основных классов:

```text
BaseModel
├── Model
├── TransientModel
└── AbstractModel
```

### `Model`

Regular database-persisted models.

### `TransientModel`

Temporary data, stored in database but automatically vacuumed periodically.

### `AbstractModel`

Abstract superclasses, предназначенные для переиспользования behavior другими models.

**[ВЫВОД]** «Документ», «справочник», «transaction» и «master data» не являются формальными альтернативами этим ORM classes.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 3. Model — объект ORM, а не просто SQL table

**[ODOO]** Model definitions содержат fields и methods; ORM обеспечивает persistence и framework behavior.

**[ODOO]** Actual class model instance может собираться из нескольких Python classes, создающих и наследующих model.

Минимально:

```text
MODEL
├── technical identity
├── fields
├── methods
├── constraints/indexes
├── relations
└── inherited/extended behavior
```

**[ВЫВОД]** Формула `ORM model = SQL table` недостаточна как архитектурная модель.

Физическая семантика fields/storage будет отдельным уроком.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 4. Recordset — основной рабочий объект ORM

**[ODOO]** Interactions with models and records выполняются через **recordsets** — ordered collections of records одной model.

Methods model выполняются на recordset; `self` является recordset.

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

**[ODOO]** В текущей реализации recordsets могут содержать duplicates, несмотря на слово set в названии.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 5. Отдельный record — singleton recordset

**[ODOO]** Records не имеют отдельного explicit representation: record представлен recordset из одного элемента.

```text
record
=
singleton recordset
```

При итерации:

```python
for record in records:
    ...
```

каждый `record` является singleton recordset.

**[ВЫВОД]** В Odoo нет второго отдельного object API «для одной строки» поверх recordset API.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 6. `ids`, `exists()` и `ensure_one()`

### `ids`

**[ODOO]** `records.ids` возвращает actual record ids текущего recordset.

### `exists()`

**[ODOO]** `records.exists()` возвращает subset records, которые существуют.

Это важно после `browse()`:

```python
records = model.browse([7, 18, 999999])
existing = records.exists()
```

**[ВЫВОД]** `browse(id)` создаёт ORM representation для identifiers, но само по себе не является доказательством существования каждого record в database.

### `ensure_one()`

**[ODOO]** Проверяет, что recordset содержит ровно один record, иначе raises `ValueError`.

```python
self.ensure_one()
```

**[ВЫВОД]** Метод должен явно заявлять singleton requirement, если его логика действительно требует одной записи.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 7. Field access пока понимаем только концептуально

**[ODOO]** Fields объявляются на model и доступны через recordset API.

Пример:

```python
record.name
```

Relational fields возвращают recordsets, но подробная field taxonomy и relation semantics будут в уроках 5–6.

На этом уровне достаточно:

```text
Model
  └── fields
       ├── values
       └── relations to other records
```

**[ВЫВОД]** Field модели не следует путать с визуальным input widget в form view.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 8. `Environment` — runtime-context recordset

**[ODOO]** Environment хранит contextual data ORM:

- `cr` — current database cursor;
- `uid` — current user id;
- `context` — context dictionary;
- `su` — superuser mode state.

Environment также предоставляет mapping от model names к models и содержит ORM runtime structures.

На текущем уроке используем минимальную модель:

```text
Environment
├── database cursor
├── current user id
├── context
└── superuser-state flag
```

Security meaning `uid/su`, company semantics и cache/recomputation будут разобраны позже.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 9. Recordsets наследуют Environment

**[ODOO]** Когда recordset создаётся из другого recordset, Environment наследуется.

**[ODOO]** Доступ к model через Environment выглядит так:

```python
partners = self.env['res.partner']
```

Это даёт recordset model в текущем Environment.

Дальше можно выполнять:

```python
partners.browse(...)
partners.search(...)
partners.create(...)
```

**[ВЫВОД]** Runtime recordset полезно мыслить не просто как `model + ids`, а как model records, существующие в определённом Environment.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 10. `browse()` — получить recordset по известным ids

**[ODOO]** `browse(ids)` возвращает recordset для переданных identifiers в current Environment.

```python
partners = self.env['res.partner'].browse([7, 18, 12])
```

```text
known ids
   │
   ▼
browse(ids)
   │
   ▼
recordset
```

`browse()` не выполняет поиск по business criteria.

Для проверки существования используется `exists()`.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 11. `search()` — выбрать records по domain

**[ODOO]** `search(domain)` возвращает records model, удовлетворяющие search domain.

```python
records = self.env['example.model'].search([
    ('active', '=', True),
])
```

Поддерживаются также `offset`, `limit`, `order`.

Пустой domain:

```python
[]
```

соответствует поиску всех records, доступных в данном runtime/security context.

Security semantics будут отдельно разобраны в уроке 9.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 12. Domain — декларативный язык условий

Базовый criterion:

```python
(field_name, operator, value)
```

Примеры:

```python
('active', '=', True)
('amount', '>', 1000)
```

Несколько обычных criteria объединяются AND.

Логические operators позволяют строить более сложные expressions.

**[ВЫВОД]** Domain — часть собственного языка ORM Odoo, а не строка SQL WHERE.

Relations inside domains будут подробно разобраны после Many2one/One2many/Many2many.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 13. `create()`

**[ODOO]** `Model.create(vals_list)` создаёт records model и возвращает созданный recordset.

Пример batch create:

```python
records = model.create([
    {'name': 'A'},
    {'name': 'B'},
])
```

Для compatibility также допустим один dictionary.

**[ВЫВОД]** Base API уже ориентирован на создание множества records, что согласуется с recordset semantics.

Defaults и computed behavior будут разобраны позже.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 14. `read()`

**[ODOO]** `read(fields)` читает requested fields records в `self` и возвращает list of dictionaries.

```python
values = records.read(['name', 'active'])
```

Концептуальный результат:

```python
[
    {'id': 1, 'name': 'A', 'active': True},
    {'id': 2, 'name': 'B', 'active': False},
]
```

**[ВЫВОД]** `read()` возвращает structured values, а attribute access работает через recordset object API.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 15. `write()`

**[ODOO]** `write(vals)` изменяет **все records** текущего recordset `self` переданными values.

```python
records.write({
    'active': False,
})
```

**[ВЫВОД]** Операции model API естественно направлены на recordsets, а не обязательно на одну record.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 16. `unlink()`

**[ODOO]** `unlink()` удаляет records текущего `self`.

```python
records.unlink()
```

Операция может быть запрещена access/security или business constraints. Подробности удаления, `@api.ondelete` и security будут позже.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 17. Минимальный рабочий цикл ORM

```text
Environment
    │
    └── env['model.name']
             │
             ▼
         recordset
             │
    ┌────────┼─────────┐
    │        │         │
 browse    search    create
    │        │         │
    └────────┼─────────┘
             ▼
         recordset
             │
      read / write / unlink
```

Это **не полный ORM API**. Это минимальный язык, достаточный для следующих уроков.

---

## 18. Что пока нельзя заключать

### «Каждый field access = SQL query»

Неизвестно на этом уровне. Cache/prefetch изучаются позже.

### «`sudo()` просто меняет user»

Тему не рассматриваем до Security.

### «Environment = global application state»

Нет. Это runtime context ORM operations.

### «browse(id) доказывает существование record»

Нет. Для этого есть `exists()`.

### «search domain = SQL WHERE»

Нет. Это ORM expression language.

### «CRUD = прямые INSERT/SELECT/UPDATE/DELETE»

Нет. Это model/recordset API Odoo с framework semantics.

---

## 19. Что сознательно вынесено дальше

### Урок 5 — Fields

- basic field types;
- parameters;
- stored/non-stored;
- automatic/reserved fields.

### Урок 6 — Relations

- Many2one;
- One2many;
- Many2many;
- commands;
- relation traversal in domains.

### Урок 7 — Derived behavior

- computed/related fields;
- depends;
- inverse;
- onchange;
- constraints;
- recomputation.

### Урок 9 — Security

- ACL;
- record rules;
- `sudo()` semantics.

### Урок 12 — Multi-company

- allowed companies;
- company context;
- company-dependent fields.

### Урок 13 — Advanced ORM

- cache;
- prefetch;
- batch performance;
- flush;
- raw SQL;
- cache invalidation;
- transaction/recomputation consistency.

---

## 20. Контрольные вопросы

1. Чем `Model`, `TransientModel` и `AbstractModel` отличаются формально?
2. Что такое recordset?
3. Что такое singleton recordset?
4. Что представляет `self` в model method?
5. Что возвращает `ids`?
6. Для чего нужен `exists()`?
7. Для чего нужен `ensure_one()`?
8. Что хранит Environment на минимальном уровне?
9. Что означает `self.env['res.partner']`?
10. Чем `browse()` отличается от `search()`?
11. Что такое domain?
12. Что возвращает `create()`?
13. Какой recordset изменяет `write()`?
14. Что возвращает `read()`?
15. Что делает `unlink()`?

---

## Официальные источники урока

1. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

2. Models and Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
