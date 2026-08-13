# Урок 3. ORM: модели, поля, recordsets, Environment и работа с данными

## Цель урока

Понять центральный механизм backend Odoo — ORM — не как удобную обёртку над SQL, а как основной способ существования и обработки данных внутри серверной части Odoo.

После урока должны быть понятны:

- чем ORM-модель отличается от таблицы PostgreSQL;
- почему `recordset` является базовым рабочим объектом методов Odoo;
- почему отдельный record в Odoo представлен singleton-recordset;
- что такое field на уровне ORM;
- как читаются и изменяются поля;
- зачем нужен `Environment`;
- чем `browse()` отличается от `search()`;
- что такое search domain;
- как работают `create()`, `write()`, `read()` и `unlink()`;
- зачем ORM содержит cache и prefetch;
- почему прямой SQL является сознательным выходом за обычную модель ORM.

В этом уроке **не разбираются глубоко** inheritance, computed fields, relational commands, constraints и multi-company. Они будут отдельными темами после того, как базовая механика ORM станет очевидной.

---

# 1. ORM — центр серверной модели Odoo

**[ODOO]** Официальная документация называет ORM ключевым компонентом Odoo. Бизнес-объекты объявляются как Python-классы, наследующие один из базовых ORM-классов.

Базовая картина:

```text
Python model declaration
        │
        ▼
      Odoo ORM
        │
        ├── model registry
        ├── fields
        ├── recordsets
        ├── access through Environment
        ├── cache / recomputation
        └── persistence
                │
                ▼
           PostgreSQL
```

**[ВЫВОД]** В Odoo PostgreSQL является слоем хранения, но нормальная серверная бизнес-логика строится не вокруг ручной работы с таблицами, а вокруг ORM models и recordsets.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 2. Model — не просто таблица

**[ODOO]** Odoo автоматически создаёт экземпляр каждой доступной модели для каждой базы данных. Набор моделей зависит от установленных модулей, а фактический класс модели собирается из Python-классов, которые создают и наследуют эту модель.

Минимальная модель в голове:

```text
Installed modules
       │
       ▼
Python model classes
       │
       ▼
Model registry for database
       │
       ▼
Available ORM models
```

Модель содержит не только описание хранимых колонок.

Она объединяет как минимум:

```text
MODEL
├── identity / technical name
├── fields
├── methods
├── ORM behavior
├── constraints / indexes
├── relations
└── extension points
```

**[ВЫВОД]** Формула:

```text
ORM Model = Database Table
```

неверна как архитектурное описание.

У обычной persisted-модели действительно может существовать собственная таблица, но модель Odoo является более высоким объектом: она задаёт поля, методы, поведение, отношения и может быть расширена другими модулями.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

---

# 3. Три базовых класса моделей

Повторим формальное разделение, потому что дальше оно становится практическим.

```text
BaseModel
├── Model
├── TransientModel
└── AbstractModel
```

## 3.1. `Model`

**[ODOO]** Обычная модель, данные которой сохраняются в базе данных.

## 3.2. `TransientModel`

**[ODOO]** Модель временных данных. Records сохраняются в базе, но автоматически очищаются с течением времени.

## 3.3. `AbstractModel`

**[ODOO]** Абстрактный суперкласс для поведения, которое должно переиспользоваться другими моделями.

**[ВЫВОД]** Это формальные категории ORM. Термины вроде «документ», «справочник» или «master data» к этому уровню не относятся.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 4. Fields: из чего состоит модель данных

**[ODOO]** Fields объявляются атрибутами Python-класса модели.

Простейший пример:

```python
from odoo import fields, models

class Example(models.Model):
    _name = 'example.model'

    name = fields.Char()
    quantity = fields.Integer()
    active = fields.Boolean()
```

Поле имеет не только тип данных. Оно может иметь параметры, определяющие, например:

- пользовательскую подпись;
- default value;
- обязательность;
- хранение;
- вычисление;
- поиск;
- индексирование;
- копирование;
- company-dependent поведение;
- relationship с другой моделью.

**[ODOO]** Значение `string` задаёт пользовательское название поля; если оно не указано, label выводится из технического имени поля.

Пример:

```python
quantity = fields.Integer(string='Quantity')
```

**[ВЫВОД]** Field в Odoo — часть метамодели ORM, а не только колонка интерфейса формы.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 5. Простые и relational fields

**[ODOO]** В Server Framework 101 поля разделяются на две широкие категории:

```text
FIELDS
├── simple
└── relational
```

Примеры simple fields:

```text
Boolean
Integer
Float
Char
Text
Date
Selection
```

Relational fields связывают records одной или разных моделей.

К ним относятся:

```text
Many2one
One2many
Many2many
```

Связи подробно будут разобраны отдельным уроком.

Сейчас важно понять только архитектурный смысл:

```text
field value
   │
   ├── atomic value
   │
   └── relation to records
```

**[ВЫВОД]** Нормальная предметная модель Odoo строится не только через набор колонок внутри одного объекта, но через сеть связанных ORM-моделей.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

---

# 6. Хранимое поле и вычисляемое поле — разные понятия

**[ODOO]** Параметр `store` определяет, хранится ли значение поля в базе. Для обычных fields хранение по умолчанию включено, а для computed fields по умолчанию выключено.

Это сразу разрушает слишком простую модель:

```text
field = database column
```

Правильнее:

```text
ORM FIELD
    │
    ├── может быть stored
    │      └── значение хранится в БД
    │
    └── может быть non-stored
           └── значение вычисляется ORM
```

Computed fields и dependency graph будут разобраны отдельно.

**[ВЫВОД]** Пользователь может видеть поле в интерфейсе, но из этого нельзя заключать, что оно является отдельной физически хранимой колонкой с самостоятельным вводом данных.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 7. Automatic fields

**[ODOO]** ORM предоставляет ряд автоматических полей.

Самое очевидное:

```text
id
```

Это идентификатор record.

Также существует:

```text
display_name
```

которое используется web client как отображаемое имя record и может вычисляться на основе `_rec_name`.

При включённом access logging ORM также ведёт:

```text
create_date
create_uid
write_date
write_uid
```

где:

- `create_date` — время создания;
- `create_uid` — пользователь, создавший record;
- `write_date` — время последнего изменения;
- `write_uid` — пользователь, последним изменивший record.

**[ВЫВОД]** Часть общесистемной семантики records обеспечивается ORM автоматически и не обязана вручную реализовываться каждым прикладным модулем.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 8. Reserved field names несут специальное поведение

**[ODOO]** Некоторые имена fields зарезервированы для стандартного поведения Odoo.

Например:

```text
name
```

по умолчанию используется как `_rec_name` для представления record.

Поле:

```text
active
```

включает стандартную семантику архивирования: records с `active=False` скрываются из большинства обычных searches/listings.

**[ВЫВОД]** В Odoo техническое имя поля иногда является частью framework contract. Нельзя считать все fields нейтральными пользовательскими атрибутами.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 9. Recordset — главный рабочий объект ORM

Это одна из самых важных идей Odoo.

**[ODOO]** Взаимодействие с моделями и records выполняется через **recordsets** — упорядоченные коллекции records одной модели.

Методы модели выполняются на recordset.

Следовательно:

```python
class Example(models.Model):
    _name = 'example.model'

    def do_something(self):
        ...
```

`self` внутри метода — не обязательно одна запись.

Это может быть:

```text
0 records
1 record
N records
```

Условно:

```text
example.model()
example.model(7)
example.model(7, 12, 18)
```

**[ODOO]** Документация отдельно предупреждает: несмотря на название, recordset в текущей реализации может содержать duplicates.

**[ВЫВОД]** Нельзя мыслить методом Odoo как обычным object method, который автоматически работает с одной сущностью. Семантика множества records заложена в API изначально.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 10. Отдельный record — singleton recordset

**[ODOO]** Record не имеет отдельного самостоятельного представления: один record представлен recordset из одного элемента.

То есть концептуально:

```text
record
=
singleton recordset
```

При итерации по recordset Odoo выдаёт новые singleton-recordsets:

```python
for record in records:
    ...
```

Если `records` содержит пять records, каждый `record` в цикле всё равно является recordset длины 1.

Модель в голове:

```text
example.model(1, 2, 3)
       │
       ├── example.model(1)
       ├── example.model(2)
       └── example.model(3)
```

**[ВЫВОД]** Термины `record` и `recordset` полезно различать по смыслу, но технически ORM не создаёт второй отдельный объектный API для singleton-record.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 11. Active Record interface

**[ODOO]** Recordsets предоставляют Active Record interface: fields читаются через Python attributes.

Пример:

```python
record.name
record.company_id
```

Изменение значения через attribute assignment обновляет поле:

```python
record.name = 'New name'
```

Для динамического имени field поддерживается dictionary-style access:

```python
field_name = 'name'
value = record[field_name]
```

**[ODOO]** Для non-relational field попытка обычного attribute read на multi-record recordset вызывает ошибку. Если нужно получить values с нескольких records, можно использовать `mapped()`.

Пример:

```python
records.mapped('name')
```

**[ODOO]** Relational field access всегда возвращает recordset — пустой, singleton или multi-record в зависимости от типа и значения связи.

**[ВЫВОД]** ORM старается сохранить единый объектный способ навигации по данным: переход по relation не выбрасывает разработчика в отдельный SQL/join API, а возвращает следующий recordset.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 12. Recordsets можно комбинировать

**[ODOO]** Recordsets immutable: set operations создают новые recordsets.

Поддерживаются, среди прочего:

```text
union        |
intersection &
difference   -
subset       <= / <
superset     >= / >
membership   in / not in
```

Например концептуально:

```python
all_records = set_a | set_b
common = set_a & set_b
only_a = set_a - set_b
```

**[ВЫВОД]** Recordset — полноценный абстрактный контейнер предметных records, а не просто список числовых `id`.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 13. `browse()` не выполняет поиск по условию

**[ODOO]** `browse(ids)` возвращает recordset для переданных identifiers в текущем Environment.

Пример:

```python
partners = self.env['res.partner'].browse([7, 18, 12])
```

Это означает:

```text
known ids
   │
   ▼
browse(ids)
   │
   ▼
recordset
```

`browse()` не является поиском по бизнес-условию.

**[ВЫВОД]** `browse()` отвечает на вопрос:

> «Дай ORM-представление records с этими identifiers».

Он не отвечает на вопрос:

> «Найди records, удовлетворяющие таким условиям».

Для второго существует `search()`.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 14. `search()` работает через domain

**[ODOO]** `search(domain)` ищет records текущей модели, удовлетворяющие search domain.

Пример:

```python
records = self.env['example.model'].search([
    ('active', '=', True),
])
```

У `search()` также есть параметры:

```text
offset
limit
order
```

Пустой domain:

```python
[]
```

соответствует поиску всех доступных records модели.

**[ODOO]** `search()` может завершиться `AccessError`, если пользователь не имеет права на требуемую информацию.

**[ВЫВОД]** Поиск ORM — не просто SQL WHERE. Он выполняется внутри model/environment/security контекста Odoo.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 15. Domain — язык условий над records

**[ODOO]** Search domain кодирует условия выбора records.

Базовый criterion имеет форму:

```python
(field_name, operator, value)
```

Например:

```python
('active', '=', True)
```

или:

```python
('amount', '>', 1000)
```

Несколько обычных критериев по умолчанию объединяются логическим AND.

Пример:

```python
[
    ('active', '=', True),
    ('amount', '>', 1000),
]
```

означает:

```text
active = True
AND
amount > 1000
```

**[ODOO]** Для явных логических комбинаций поддерживаются prefix operators:

```text
&  AND
|  OR
!  NOT
```

**[ВЫВОД]** Domain — отдельный декларативный язык фильтрации Odoo, который используется далеко шире одного Python `search()`: тот же принцип позже встречается в relational fields, actions и UI-фильтрации.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

---

# 16. Domain может проходить по отношениям

**[ODOO]** Search domain позволяет ссылаться на relationship traversal через dot notation для соответствующих relational paths.

Концептуальный пример:

```python
('partner_id.country_id.code', '=', 'NL')
```

Модель мышления:

```text
current record
   │
   └── partner_id
          │
          └── country_id
                 │
                 └── code
```

**[ВЫВОД]** Предметные связи ORM являются не просто способом показать dropdown в UI; они входят непосредственно в язык поиска данных.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 17. `create()` создаёт records, а не одну обязательную строку

**[ODOO]** `Model.create(vals_list)` создаёт новые records модели.

Нормальная сигнатура принимает список dictionaries:

```python
records = model.create([
    {'name': 'A'},
    {'name': 'B'},
])
```

Для обратной совместимости допустим один dictionary:

```python
record = model.create({'name': 'A'})
```

В таком случае он интерпретируется как singleton list.

**[ODOO]** При создании используются переданные values и, при необходимости, defaults через `default_get()`.

Метод возвращает созданные records.

**[ВЫВОД]** Batch semantics встроена уже в базовый create API Odoo. Это согласуется с общей ориентацией ORM на recordsets, а не на последовательную ручную обработку строк по одной.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 18. `write()` изменяет весь `self`

**[ODOO]** `write(vals)` изменяет все records текущего recordset `self` переданными values.

Пример:

```python
records.write({
    'active': False,
})
```

Если `records` содержит 100 records, операция семантически направлена на весь recordset.

**[ВЫВОД]** В Odoo естественная единица серверной операции часто является набором records, а не одиночной строкой.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 19. `read()` возвращает сериализованное представление values

**[ODOO]** `read(fields)` читает requested fields для records в `self` и возвращает list of dictionaries — по одному dictionary на record.

Концептуально:

```python
values = records.read(['name', 'active'])
```

результат имеет форму:

```python
[
    {'id': 1, 'name': 'A', 'active': True},
    {'id': 2, 'name': 'B', 'active': False},
]
```

**[ВЫВОД]** Это отличается от Active Record access:

```python
record.name
```

Первое получает structured values, второе работает непосредственно через recordset API.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 20. `unlink()` удаляет records

**[ODOO]** `unlink()` удаляет records в `self`.

Это снова recordset operation:

```python
records.unlink()
```

**[ODOO]** Операция может завершиться `AccessError`, если пользователь не имеет соответствующего доступа, и может блокироваться бизнес-ограничениями.

Позже отдельно будет разобран `@api.ondelete`, потому что Odoo учитывает не только обычное удаление records, но и корректность поведения при uninstall модулей.

**[ВЫВОД]** CRUD в Odoo является частью model API и связан с security/business behavior, а не просто с четырьмя SQL-командами.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 21. CRUD в модели Odoo

Теперь можно собрать базовый цикл:

```text
CREATE
  │
  ▼
recordset
  │
  ├── browse known ids
  ├── search by domain
  ├── read fields
  ├── access fields directly
  ├── write values
  └── unlink records
```

Или в терминах методов:

```text
create()
browse()
search()
read()
write()
unlink()
```

Это не полный ORM API, но это минимальный операционный скелет.

---

# 22. `Environment` — контекст выполнения ORM

Это второй фундаментальный объект после recordset.

**[ODOO]** `Environment` хранит contextual data, которые использует ORM:

```text
cr
uid
context
su
```

Где:

- `cr` — текущий database cursor;
- `uid` — id текущего пользователя, используемый в том числе для access rights checks;
- `context` — dictionary произвольных contextual metadata;
- `su` — режим superuser.

Environment также:

- предоставляет доступ к registry по model name;
- держит cache records;
- содержит структуру управления recomputations.

Минимальная схема:

```text
ENVIRONMENT
├── database cursor
├── current user id
├── context
├── superuser state
├── model registry access
├── record cache
└── recomputation state
```

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 23. `self.env['model.name']` — доступ к модели через Environment

**[ODOO]** Environment реализует mapping от model names к models.

Пример:

```python
partners = self.env['res.partner']
```

Результат — пустой recordset соответствующей модели в текущем Environment.

Дальше можно выполнить:

```python
partners.search(...)
partners.create(...)
partners.browse(...)
```

Концептуально:

```text
Environment
     │
     └── ['res.partner']
              │
              ▼
       res.partner()
       empty recordset
              │
              ├── search()
              ├── browse()
              └── create()
```

**[ВЫВОД]** Доступ к модели в runtime идёт не через глобальный singleton Python-класс, а через model registry в конкретном Environment.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 24. Recordset наследует Environment

**[ODOO]** При создании нового recordset из другого recordset Environment наследуется.

Поэтому цепочка:

```python
records = self.env['example.model'].search(domain)
```

не теряет текущие:

```text
user
context
cursor
superuser state
cache context
```

**[ВЫВОД]** Recordset нельзя корректно рассматривать только как пару:

```text
model + ids
```

Практически он существует внутри Environment.

Полезнее мыслить:

```text
recordset
=
model + records + environment
```

Это не формальная сигнатура Odoo, а архитектурная модель для понимания.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 25. Context — не произвольный глобальный storage

**[ODOO]** Environment содержит `context` как dictionary contextual metadata.

На разных участках framework context используется для изменения поведения без изменения identity самой модели или records.

Например в ORM существуют специальные context keys, влияющие на поведение framework.

**[ВЫВОД]** Context нужно мыслить как **контекст выполнения операции**, а не как нормальное место долговременного хранения бизнес-данных.

Если значение должно быть устойчивым бизнес-фактом, его место нужно искать в fields/records соответствующей модели, а не автоматически помещать в context.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 26. `sudo()` меняет режим доступа, а не текущего пользователя

**[ODOO]** `sudo(flag=True)` возвращает новую версию recordset с включённым или выключенным superuser mode.

Документация отдельно уточняет: superuser mode не меняет текущего пользователя, а bypasses access rights checks.

Также Odoo предупреждает, что `sudo` может пересечь границы record rules и смешать records, которые должны быть изолированы, включая multi-company данные.

**[ВЫВОД]** `sudo()` — не безобидный технический shortcut. Он меняет security semantics ORM-операции.

Глубокий security-разбор будет позже.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 27. Cache встроен в Environment/ORM

**[ODOO]** Odoo кеширует field values records, чтобы каждый field access не создавал отдельный database query.

Пример семантики:

```python
record.name  # первый доступ может прочитать из БД
record.name  # повторный доступ может прийти из cache
```

Модель:

```text
record field access
      │
      ├── cache hit ─────────► value
      │
      └── cache miss
              │
              ▼
           database
              │
              ▼
            cache
              │
              ▼
            value
```

**[ВЫВОД]** Чтение field через ORM нельзя сводить к модели «каждое обращение = SELECT». ORM сознательно абстрагирует это за cache.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 28. Prefetch — чтение пакетами

**[ODOO]** Чтобы не читать один field одного record за раз, ORM использует prefetch heuristics.

Когда field требуется для одного record, ORM обычно читает этот field для более широкого recordset, из которого record происходит, и помещает values в cache.

Кроме того, stored scalar-like fields, которые эффективно читаются из колонок той же таблицы, могут загружаться вместе.

Пример:

```python
for partner in partners:
    print(partner.name)
    print(partner.lang)
```

Без prefetch такой код мог бы породить множество SQL-запросов.

С prefetch ORM получает values пакетно для recordset.

Архитектурно:

```text
partners = many records
       │
       ▼
first field access
       │
       ▼
prefetch broader set
       │
       ▼
cache populated
       │
       ▼
subsequent field accesses
mostly use cache
```

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 29. Prefetch распространяется на secondary records

**[ODOO]** Prefetch работает и при проходе по relational fields. Records, полученные через relations, могут подписываться на последующий prefetch.

Официальная документация приводит принципиальный пример:

```text
partners
   │
   └── country_id
            │
            └── country.name
```

ORM способен сначала пакетно прочитать partners, а затем пакетно прочитать связанные countries, вместо SQL-запроса на каждую пару.

**[ВЫВОД]** Relational object traversal и prefetch спроектированы совместно. Это одна из причин, почему нативная модель Odoo предпочитает работать через ORM relations вместо ручного поштучного SQL.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 30. `fetch()` и `search_fetch()` позволяют управлять загрузкой

**[ODOO]** `fetch(field_names)` обеспечивает наличие requested fields в памяти/cache для records текущего recordset.

**[ODOO]** `search_fetch(domain, field_names, ...)` сочетает поиск records и загрузку fields с минимальным количеством SQL queries.

Эти методы предназначены для случаев, где обычный prefetch не даёт желаемой схемы доступа.

**[ВЫВОД]** Cache/prefetch — не скрытая деталь реализации, о которой разработчику никогда не нужно знать. ORM предоставляет явные инструменты оптимизации поверх этой модели.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 31. Batch operations — нативная модель производительности

**[ODOO]** Performance documentation рекомендует работать batch operations над recordsets.

Антипаттерн:

```python
for record_id in record_ids:
    record = model.browse(record_id)
    print(record.foo)
```

В таком варианте ORM теряет возможность нормально prefetch records одной группой.

Предпочтительно:

```python
records = model.browse(record_ids)
for record in records:
    print(record.foo)
```

То же относится к create: документация рекомендует по возможности создавать records batch-операцией, а не по одному вызову в цикле.

**[ВЫВОД]** Recordset semantics — одновременно предметная и performance-модель Odoo.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/performance.html

---

# 32. Cache означает необходимость согласованности с БД

Пока все изменения проходят через ORM, framework управляет своим состоянием.

Проблема возникает при прямом SQL.

**[ODOO]** Environment использует cache. Если database изменяется raw SQL-командой `CREATE`, `UPDATE` или `DELETE`, cache необходимо корректно invalidated, иначе последующая ORM-работа может увидеть несогласованное состояние.

Модель проблемы:

```text
ORM cache: state = old

raw SQL:
UPDATE ... state = new

Database: state = new
Cache:    state = old
```

Без invalidation ORM может продолжить жить с устаревшим cached value.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 33. Raw SQL — сознательный выход из обычной безопасности ORM

**[ODOO]** `env.cr` предоставляет database cursor и позволяет выполнять SQL напрямую.

Но документация прямо предупреждает: raw SQL bypasses ORM и, следовательно, security rules Odoo.

Поэтому:

```text
ORM operation
   │
   ├── model semantics
   ├── security integration
   ├── cache/recompute integration
   └── framework behavior

raw SQL
   │
   └── direct database operation
```

Raw SQL может быть оправдан для сложных запросов или производительности, но требует ручного внимания к:

- security;
- sanitization;
- flushing;
- cache invalidation;
- recomputation consistency.

**[ВЫВОД]** SQL не является «более нативным низким уровнем Odoo». Это более низкий уровень PostgreSQL, использование которого сознательно обходит часть гарантий ORM.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

# 34. ORM связывает данные, security и runtime-context

После всех предыдущих частей можно собрать более точную схему:

```text
                     ENVIRONMENT
              ┌─────────┼─────────┐
              │         │         │
            user      context   cursor
              │         │         │
              └─────────┼─────────┘
                        │
                        ▼
                     REGISTRY
                        │
                        ▼
                      MODEL
                        │
         ┌──────────────┼──────────────┐
         │              │              │
       fields         methods       relations
         │              │              │
         └──────────────┼──────────────┘
                        │
                        ▼
                    RECORDSET
                        │
        ┌───────────────┼────────────────┐
        │               │                │
     search/read     create/write      unlink
        │               │                │
        └───────────────┼────────────────┘
                        │
                        ▼
                 CACHE / PREFETCH
                        │
                        ▼
                    PostgreSQL
```

Это значительно ближе к реальной серверной архитектуре Odoo, чем схема:

```text
Python object → SQL table
```

---

# 35. Что ORM НЕ означает

## Не означает 1: «каждая model = одна таблица и ничего больше»

Model содержит framework behavior и может быть составлена расширениями нескольких modules.

## Не означает 2: «record = обычный Python instance строки»

Record представлен singleton recordset.

## Не означает 3: «self всегда один record»

`self` может содержать от нуля до множества records.

## Не означает 4: «field = обязательно stored column»

Fields могут быть computed/non-stored и обладать другой ORM-семантикой.

## Не означает 5: «search = обычный SQL WHERE»

Search работает через domain внутри Environment и security context.

## Не означает 6: «каждый field access вызывает SQL»

ORM использует cache и prefetch.

## Не означает 7: «sudo = другой пользователь»

Superuser mode меняет security behavior, но не подменяет текущего user сам по себе.

## Не означает 8: «raw SQL — эквивалент ORM, только быстрее»

Raw SQL bypasses ORM security и требует ручного управления consistency.

---

# 36. Почему этот урок важен для понимания идеологии Odoo

Теперь можно впервые сформулировать несколько архитектурных выводов.

### [ВЫВОД] 1. Odoo мыслит наборами records

API методов, batch create/write и prefetch ориентированы на recordsets.

### [ВЫВОД] 2. Business object существует внутри runtime context

Recordset несёт Environment, а Environment содержит user, context, cursor, cache и registry access.

### [ВЫВОД] 3. Relations являются частью самого языка ORM

Связанные models не просто отображаются в UI; через них происходит навигация, filtering и prefetch.

### [ВЫВОД] 4. ORM является границей гарантий framework

При работе через ORM Odoo может применять свои security, cache, recomputation и model behavior. При прямом SQL часть этих гарантий приходится поддерживать вручную.

### [ВЫВОД] 5. Производительность заложена в объектную модель

Recordsets, batch operations и prefetch — не независимые оптимизации после проектирования. Они следуют из того, как сам ORM представляет данные.

---

# 37. Минимальная модель в голове после урока

```text
Database
   │
   ▼
Registry
   │
   ▼
Model
   │
   ├── fields
   ├── methods
   └── relations
   │
   ▼
Recordset
   │
   ├── 0..N records
   ├── singleton = one record
   ├── active-record field access
   ├── CRUD/search
   └── set operations
   │
   ▼
Environment
   ├── cursor
   ├── user id
   ├── context
   ├── superuser state
   ├── registry access
   ├── cache
   └── recomputation state
   │
   ▼
Cache / Prefetch
   │
   ▼
PostgreSQL
```

При этом порядок стрелок не означает, что Environment «ниже» recordset технически. Схема показывает связи понятий, а не UML-классы.

---

# 38. Контрольные вопросы

Перед следующим уроком нужно уметь объяснить без подсказки:

1. Почему ORM model нельзя приравнять к таблице PostgreSQL?
2. Какие три базовых типа models есть в Odoo?
3. Чем field отличается от UI-поля формы?
4. Какие две широкие категории fields выделяет базовый tutorial?
5. Что означает `store`?
6. Какие automatic access-log fields предоставляет ORM?
7. Что такое recordset?
8. Почему отдельный record называется singleton-recordset?
9. Может ли `self` содержать несколько records?
10. Что происходит при итерации по recordset?
11. Почему non-relational field нельзя просто прочитать как attribute на multi-record set?
12. Чем `browse()` отличается от `search()`?
13. Что такое domain?
14. Как обычные criteria domain комбинируются по умолчанию?
15. Что делает `create()`?
16. На сколько records действует `write()`?
17. Что возвращает `read()`?
18. Что хранит `Environment`?
19. Что означает `self.env['res.partner']`?
20. Почему recordset нельзя корректно мыслить только как список ids?
21. Зачем ORM cache?
22. Что такое prefetch?
23. Почему полезно browse/search records batch-операцией?
24. Почему raw SQL требует cache invalidation после изменений?
25. Почему raw SQL и ORM не эквивалентны по security semantics?

---

# 39. Что изучаем дальше

После этого урока уже можно безопасно углубиться в **поля и отношения**.

Следующий урок должен разобрать:

```text
Fields
├── scalar/basic
├── Many2one
├── One2many
├── Many2many
├── relational commands
├── computed fields
├── dependencies
├── inverse
├── related fields
├── store
└── recomputation
```

Только после этого имеет смысл подробно разбирать inheritance и то, как несколько modules составляют одну итоговую предметную модель.

---

# Официальные источники урока

1. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

2. Chapter 3: Models And Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

3. Building a Module — Domains  
   https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

4. Performance — Good practices / Batch operations  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/performance.html
