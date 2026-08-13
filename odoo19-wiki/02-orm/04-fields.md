# ORM-04. Fields

> Lesson ID: `ORM-04`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ORM-03`  
> Canonical owner: field definition; generic field attributes; non-relational field types; automatic fields; field storage semantics  
> Aspect owner: field access security → `SEC-01`; company-dependent fields → `EXT-04`; field performance/index tuning → `RUN-01`; field extension/incremental definition → `EXT-01`  
> Preview: relational fields; computed/related fields  
> Отложено: Many2one/One2many/Many2many/Command; compute/related/inverse/depends; `selection_add` extension mechanics; company consistency; ACL/group semantics; cache/prefetch  
> Edition scope: platform ORM semantics; concrete application entitlement здесь не определяется  
> Sources: `S1`, `S2`

## Цель

Понять поле как часть ORM-модели, а не как элемент формы.

После `ORM-03` мы уже различаем model metadata и SQL/schema declarations. Теперь добавляем следующий слой:

```text
ORM MODEL
   │
   ├── model metadata / schema       → ORM-03
   │
   └── FIELDS                        → ORM-04
         ├── type
         ├── label/help
         ├── required/default
         ├── storage/index/copy
         ├── access/company hooks
         └── value semantics
```

Relational и derived fields будут отдельными owners.

---

## 1. Field — ORM descriptor

**[ODOO][S1]** `odoo.fields.Field` содержит definition поля и управляет доступом и присваиванием соответствующего значения на records.

Поле объявляется class attribute модели:

```python
from odoo import fields, models

class Example(models.Model):
    _name = 'example.model'

    name = fields.Char()
    active = fields.Boolean(default=True)
```

Из `ORM-01` уже известно, что values читаются через record/recordset API:

```python
record.name
record['name']
```

**[ВЫВОД]** Field одновременно является частью Python ORM definition и контрактом работы значения record. Он не равен HTML input, XML `<field>` или JavaScript widget.

---

## 2. Field name — Python class attribute и ORM field name

**[ODOO][S1]** Fields определяются как attributes model class.

```python
name = fields.Char()
```

Здесь `name` становится technical field name.

**[ODOO][S1]** Нельзя определять field и method с одинаковым именем: позднее class attribute silently overwrite предыдущий.

```python
name = fields.Char()

def name(self):
    ...
```

— плохая и фактически конфликтующая definition.

**[ВЫВОД]** Namespace fields и methods внутри model class требует дисциплины: technical field name является реальной частью Python class namespace.

---

## 3. `string` — пользовательская метка, не technical name

**[ODOO][S1]** Если `string` не указан, ORM формирует user-visible label из field name; параметр `string` позволяет его переопределить.

```python
internal_code = fields.Char(string='Internal Code')
```

Различаем:

```text
internal_code
= technical field name

Internal Code
= user-facing label
```

**[ВЫВОД]** Переименование `string` не переименовывает technical field в ORM.

---

## 4. `help`

**[ODOO][S1]** `help` задаёт user-visible tooltip/описание поля.

```python
code = fields.Char(
    string='Code',
    help='Stable external code',
)
```

`help` является metadata field, а не validation rule.

---

## 5. `readonly`: особенно опасная граница

**[ODOO][S1]** Generic field parameter `readonly=True` влияет только на UI. Программное присваивание всё равно возможно для stored или inversable field.

```python
code = fields.Char(readonly=True)
```

не означает:

```text
server-side immutable field
```

**[ВЫВОД]** `readonly` нельзя использовать как security/business-integrity barrier.

Полные security semantics принадлежат `SEC-01`.

---

## 6. `required`

**[ODOO][S1]** `required=True` означает, что значение field обязательно.

```python
name = fields.Char(required=True)
```

Это ORM field requirement, а не просто визуальная пометка формы.

При этом конкретное business правило вида:

> поле обязательно только при определённом состоянии

не следует автоматически путать с постоянным field-level `required=True`.

Conditional UI behavior будет изучаться вместе с Views.

---

## 7. `default`: значение или callable

**[ODOO][S1]** Default field value можно задать literal value или callable/function.

```python
active = fields.Boolean(default=True)
```

или:

```python
def _default_name(self):
    return self._compute_initial_name()

name = fields.Char(default=lambda self: self._default_name())
```

**[ВЫВОД]** Default — способ получить исходное значение при создании, а не database/business invariant.

Наличие default не означает, что пользователь/код впоследствии не сможет записать другое значение.

---

## 8. `store`: stored и non-stored — разные semantics

**[ODOO][S1]** Generic `store` определяет, хранится ли field в database. Для обычных fields default — `True`; для computed fields default другой и будет разобран в `ORM-06`.

Минимально:

```text
stored field
→ value имеет database storage semantics

non-stored field
→ value не читается как обычная stored column value
```

**[ВЫВОД]** Нельзя считать каждый ORM field физической колонкой model table.

Это продолжает принцип из `ORM-03`:

```text
ORM model ≠ SQL table
ORM field ≠ обязательно SQL column
```

---

## 9. `index`: field-level index declaration

**[ODOO][S1]** Generic field parameter `index` управляет database index для field. Документация перечисляет варианты, включая:

- `True` / `"btree"`;
- `"btree_not_null"`;
- `"trigram"`;
- `False` / `None`.

Для non-stored/virtual fields параметр не даёт обычного database index effect.

```python
code = fields.Char(index=True)
```

**[ВЫВОД]** Field-level `index=` и schema-level `models.Index(...)` из `ORM-03` — связанные, но разные declaration mechanisms.

Как выбирать index в production — owner `RUN-01`, а не этот урок.

---

## 10. `copy`

**[ODOO][S1]** `copy` определяет, копируется ли значение field при duplication record.

Default зависит от характера field: для normal fields обычно `True`, для некоторых relational/computed/property/related cases — `False`.

Полную специфику этих типов мы не переносим сюда раньше их owners.

**[ВЫВОД]** Duplicate record — не raw clone каждой ORM value.

---

## 11. `groups`: только архитектурная граница

**[ODOO][S1]** Generic `groups` принимает comma-separated XML IDs groups и ограничивает field access указанными groups.

```python
secret = fields.Char(groups='base.group_system')
```

Но полное определение:

- group;
- user security principal;
- ACL;
- record rule;
- field access interaction

принадлежит `SEC-01`.

**[ВЫВОД]** Field может иметь собственный security-access aspect, но `groups=` не является полной моделью безопасности Odoo.

---

## 12. `company_dependent`: только preview

**[ODOO][S1]** `company_dependent=True` означает, что value поля зависит от current company; Odoo 19 ORM Reference документирует хранение таких values в `jsonb` dict с company id как key и fallback через `ir.default`.

Это важный факт, но ownership full semantics — `EXT-04`.

```text
same record
+ same technical field
+ different company context
→ potentially different effective value
```

**[ВЫВОД]** `company_dependent` не означает «просто добавить company_id column к record».

Multi-company semantics здесь сознательно не раскрываются дальше.

---

## 13. `search`: field может участвовать в search semantics не только как column

**[ODOO][S1]** Generic field parameter `search` может указывать method, который переписывает search condition field в replacement domain.

Это особенно важно для fields, чьи values не сводятся к прямому SQL-column lookup.

Full computed/search mechanics будут в `ORM-06`.

**[ВЫВОД]** Domain criterion по field не обязательно означает буквальный SQL predicate по одноимённой колонке.

---

## 14. `aggregator` и `group_expand`

**[ODOO][S1]** Field metadata содержит aggregation/grouping hooks.

`aggregator` задаёт default aggregate function, используемую web client при группировке. Documentation перечисляет, например:

```text
count
count_distinct
bool_and
bool_or
max
min
avg
sum
```

`group_expand` позволяет расширять набор groups для grouped views; для Selection поддерживается специальное поведение.

**[ВЫВОД]** Field metadata может влиять не только на storage, но и на generic grouping/aggregation behavior.

Detailed UI/read_group mechanics остаются будущим owners.

---

# Часть II. Типы полей

## 15. Учебная таксономия этого урока

Official ORM Reference документирует несколько families field mechanics. Для текущего owner мы разбираем **non-relational fields**, а не пытаемся захватить будущие уроки.

```text
Field
│
├── scalar/basic
│   ├── Boolean
│   ├── Char
│   ├── Text
│   ├── Integer
│   └── Float
│
├── semantic/specialized scalar
│   ├── Monetary
│   ├── Selection
│   ├── Date
│   └── Datetime
│
├── content
│   ├── Binary
│   ├── Image
│   └── Html
│
├── relational              → ORM-05
│
└── computed / related      → ORM-06
```

**[ВЫВОД]** Это учебная карта ownership, а не утверждение, что Python class hierarchy Odoo обязана буквально иметь такое дерево.

---

## 16. `Boolean`

**[ODOO][S1]** `fields.Boolean` инкапсулирует `bool`.

```python
active = fields.Boolean(default=True)
```

Использовать Boolean логично для бинарного признака, а не для lifecycle с множеством состояний.

Последнее уже business-model decision, а не правило framework.

---

## 17. `Char`

**[ODOO][S1]** `fields.Char` — string field для относительно короткого текста, обычно single-line в clients.

Специфические parameters включают:

- `size`;
- `trim`;
- `translate`.

```python
code = fields.Char(size=32, trim=True)
```

**[ODOO][S1]** В Odoo 19 trim behavior согласован между web client create/write flow и server import behavior.

**[ВЫВОД]** `Char` — data type/field semantics; конкретный widget отображения является отдельным UI layer.

---

## 18. `Text`

**[ODOO][S1]** `fields.Text` похож на Char, но предназначен для более длинного содержимого, не имеет `size` и обычно отображается multiline.

```python
notes = fields.Text()
```

`Text` также может использовать translation semantics.

---

## 19. `Integer`

**[ODOO][S1]** `fields.Integer` инкапсулирует Python `int`.

```python
priority = fields.Integer()
```

**[ВЫВОД]** Integer field не означает автоматически sequence, priority или count. Business meaning определяется model semantics.

---

## 20. `Float` и precision

**[ODOO][S1]** `fields.Float` инкапсулирует float; precision настраивается через `digits`.

```python
quantity = fields.Float(digits=(16, 3))
```

ORM Reference также предоставляет helpers:

```text
fields.Float.round()
fields.Float.is_zero()
fields.Float.compare()
```

**[ODOO][S1]** Для quantity, связанной с unit of measure, документация рекомендует сравнивать/округлять с корректной precision/rounding соответствующей UoM.

**[ВЫВОД]** Сравнение business quantities простым `a == b` может быть семантически неправильным, если domain использует собственную precision.

---

## 21. `Monetary`

**[ODOO][S1]** `fields.Monetary` представляет float amount в определённой currency. Decimal precision и currency symbol берутся через `currency_field`, по умолчанию `currency_id`.

```python
amount = fields.Monetary(currency_field='currency_id')
```

`currency_field` ссылается на Many2one к currency model; relation semantics будут в `ORM-05`.

**[ВЫВОД]** Денежное значение в Odoo — не просто Float с символом валюты в UI: field semantics явно связывают amount с currency field.

---

## 22. `Selection`

**[ODOO][S1]** `fields.Selection` представляет exclusive choice между допустимыми values.

```python
state = fields.Selection([
    ('draft', 'Draft'),
    ('done', 'Done'),
])
```

Различаем:

```text
'draft'
= stored/technical selection value

'Draft'
= user-facing label
```

`selection` может быть list, callable или method name.

`selection_add` и `ondelete` нужны при расширении существующего Selection; detailed extension mechanics принадлежат `EXT-01`.

**[ВЫВОД]** Selection удобен для закрытого набора values, но наличие Selection само по себе не делает универсальный workflow engine.

---

## 23. `Date` и `Datetime` — не одно и то же

**[ODOO][S1]** ORM Reference отдельно подчёркивает корректное использование Dates и Datetimes.

Допустимые assignments включают:

- Python `date` / `datetime`;
- server-format string;
- `False` / `None`.

Formats:

```text
Date      YYYY-MM-DD
Datetime  YYYY-MM-DD HH:MM:SS
```

**[ODOO][S1]** Date следует сравнивать с date objects, Datetime — с datetime objects; comparison строк Date и Datetime documentation прямо не рекомендует из-за неожиданных результатов.

---

## 24. Datetime и timezone

**[ODOO][S1]** Datetime values хранятся в database как `timestamp without timezone`, в UTC; timezone conversion управляется client side.

```text
DATABASE DATETIME
        ↓
UTC storage convention
        ↓
client-side timezone conversion
```

**[ВЫВОД]** Нельзя сохранять разные local-time conventions в Datetime field и ожидать, что PostgreSQL timezone column сам решит semantics Odoo.

Timezone-aware user behavior будет использовано позже, но этот storage fact является частью field semantics.

---

## 25. Date/Datetime helpers

**[ODOO][S1]** Date и Datetime classes предоставляют helpers для conversion и period operations, включая:

```text
today
context_today (Date)
to_date
to_datetime
to_string
start_of
end_of
add
subtract
```

Не все helpers одинаковы для обоих classes; при использовании всегда проверяется конкретный API Reference.

**[ВЫВОД]** Для Odoo date/time logic предпочтительно опираться на documented field/date helpers, а не изобретать собственные string manipulations.

---

## 26. `Binary`

**[ODOO][S1]** `fields.Binary` хранит binary content, например file.

Ключевой parameter:

```text
attachment=True  (default)
```

определяет, хранится ли content через `ir_attachment` либо в column model table.

**[ВЫВОД]** Даже stored ORM field не обязательно означает, что payload физически хранится прямо в table текущей model.

Это ещё один пример, почему ORM storage semantics нельзя сводить к «одно поле = одна колонка с value».

---

## 27. `Image`

**[ODOO][S1]** `fields.Image` расширяет Binary и добавляет image-specific validation/resizing parameters, включая:

- `max_width`;
- `max_height`;
- `verify_resolution`.

```python
image = fields.Image(max_width=1920, max_height=1080)
```

**[ВЫВОД]** Image — не только UI widget для Binary; field type имеет собственную server-side content semantics.

---

## 28. `Html`

**[ODOO][S1]** `fields.Html` представляет HTML content и имеет sanitation parameters.

Например:

```text
sanitize
sanitize_overridable
sanitize_tags
sanitize_attributes
sanitize_style
...
```

Default sanitation включена.

**[ВЫВОД]** Html field отличается от plain Text не только способом отображения: type содержит специальные content-sanitization semantics.

Security implications sanitation не заменяют общий Security owner.

---

# Часть III. Automatic и reserved fields

## 29. `id`

**[ODOO][S1]** `Model.id` — identifier field record.

Для singleton recordset можно получить:

```python
record.id
```

Если recordset не singleton, обращение к `id` не является способом получить список IDs; для этого из `ORM-01` используется:

```python
records.ids
```

---

## 30. `display_name`

**[ODOO][S1]** `display_name` — automatic name field, показываемое web client по умолчанию.

Default behavior связан с `_rec_name`, уже рассмотренным в `ORM-03`, и может кастомизироваться через `_compute_display_name`.

```text
_rec_name metadata      → ORM-03
         │
         ▼
display_name field      → ORM-04
```

**[ВЫВОД]** `_rec_name` и `display_name` нельзя считать одним и тем же объектом.

---

## 31. Access Log fields

**[ODOO][S1]** Если `_log_access` enabled, ORM автоматически ведёт access-log fields:

```text
create_date
create_uid
write_date
write_uid
```

`ORM-03` уже определил `_log_access` как model-level switch.

Теперь связь:

```text
_log_access
    │
    ▼
automatic Access Log fields
```

- `create_date` / `write_date` — Datetime semantics;
- `create_uid` / `write_uid` — relations к users, relation details → `ORM-05`.

**[ВЫВОД]** Эти fields являются framework-maintained technical metadata records, а не бизнес-полями автора/исполнителя документа.

---

## 32. Reserved field names создают framework behavior

**[ODOO][S1]** ORM Reference документирует ряд reserved field names, наличие которых включает specific framework behavior.

К ним относятся, среди прочего:

```text
active
state
parent_id
parent_path
company_id
```

Это не означает, что все models обязаны иметь эти fields.

---

## 33. `active`

**[ODOO][S1]** Field name `active` используется framework для active/archive behavior; соответствующие model helpers умеют архивировать/разархивировать records.

**[ВЫВОД]** `active=False` — framework archival pattern, а не универсальное business state «закрыт» или «удалён».

---

## 34. `state`

**[ODOO][S1]** Reserved `state` связывается с lifecycle stages object и Selection semantics.

Но это **не** доказательство существования единой универсальной Odoo state machine для всех models.

**[ВЫВОД]** У разных applications могут быть собственные lifecycle semantics даже при использовании field name `state`.

---

## 35. `parent_id` / `parent_path`

**[ODOO][S1]** `parent_id` является default field для `_parent_name`, а `parent_path` используется вместе с `_parent_store=True` для optimized hierarchy и domain operators `child_of` / `parent_of`.

Model-level attributes уже разобраны в `ORM-03`.

Relation/domain details принадлежат `ORM-05`.

---

## 36. `company_id`

**[ODOO][S1]** `company_id` является основным field name, используемым Odoo multi-company behavior и consistency checks.

Но полная semantics:

- shared vs company-specific record;
- allowed/active companies;
- `check_company`;
- company-dependent values

принадлежит `EXT-04`.

**[ВЫВОД]** В этом уроке `company_id` — reserved framework hook, а не разрешение преждевременно строить multi-company model.

---

# Часть IV. Границы со следующими owners

## 37. Relational fields — уже видим, но не определяем

ORM Reference документирует:

```text
Many2one
One2many
Many2many
Command
```

Canonical owner — `ORM-05`.

Здесь достаточно понимать:

> relational field является Field, но его value semantics выражаются recordsets/relations, поэтому он требует отдельного урока.

---

## 38. Computed и related — не часть текущего owner

Generic Field имеет parameters вроде:

```text
compute
precompute
compute_sudo
recursive
inverse
related
```

Они официально документированы в Field API, но canonical semantics принадлежат `ORM-06`.

**[ВЫВОД]** Reference page организована по API, а учебный курс — по dependency ownership. Мы не обязаны преподавать всё, что находится рядом на одной странице документации.

---

## 39. Field-level `groups` не заменяет Security

Ещё раз граница:

```text
field groups metadata
        │
        ▼
security aspect of field

НО

ACL + record rules + users/groups + sudo + public methods
        │
        ▼
SEC-01
```

Поле не владеет общей security model.

---

## 40. `company_dependent` не заменяет Multi-company

```text
Field.company_dependent
        │
        └── storage/value hook

Multi-company
        │
        └── broader Environment/record/company semantics
```

Owner второго — `EXT-04`.

---

## 41. Field и View field node — разные concepts

```python
name = fields.Char(required=True)
```

— ORM field definition.

```xml
<field name="name"/>
```

— UI view node, который только ссылается на model field.

**[ВЫВОД]** Один и тот же technical field может отображаться по-разному в разных views; его ORM identity при этом не меняется.

Views принадлежат `UI-02`.

---

## 42. Минимальная mental model

```text
                         ORM FIELD
                            │
        ┌───────────────────┼────────────────────┐
        │                   │                    │
   definition metadata   value type          storage/runtime
        │                   │                    │
 string/help             Char/Text          store
 required/default        Integer/Float      index
 readonly                Boolean            copy
 groups                  Selection          company hook
 ...                     Date/Datetime      search/group hooks
                         Monetary
                         Binary/Image/Html
                            │
                            ├── relations → ORM-05
                            └── derived    → ORM-06
```

---

## Что нельзя заключать

- Field = form widget — нет;
- `string` = technical field name — нет;
- `readonly=True` = server-side immutability/security — нет;
- default = invariant — нет;
- every field = SQL column — нет;
- stored Binary payload всегда находится прямо в model table — нет;
- Float equality можно всегда проверять обычным Python `==` без domain precision — нет;
- Monetary = обычный Float с символом валюты — нет;
- Date = Datetime без времени — недостаточно как semantics;
- `state` = универсальный Odoo workflow engine — нет;
- `active=False` = business deletion — нет;
- `company_id` уже объясняет multi-company — нет;
- `groups=` уже объясняет security — нет;
- этот урок уже определяет relations/computed fields — нет.

## Контрольные вопросы

1. Что такое `Field` в ORM?
2. Почему field и method не должны иметь одинаковое имя?
3. Чем `string` отличается от technical field name?
4. Почему `readonly=True` не является security barrier?
5. Чем default отличается от invariant?
6. Что означает `store`?
7. Чем field-level `index=` отличается от schema-level `models.Index`?
8. Какие semantics несёт `company_dependent` и почему full owner другой?
9. В чём разница Char и Text?
10. Почему Float требует внимания к precision?
11. Чем Monetary отличается от Float?
12. Что хранит Selection: technical value или label?
13. Как Odoo 19 хранит Datetime относительно timezone?
14. Чем Binary storage может отличаться от обычной table column?
15. Что добавляет Image поверх Binary?
16. Почему Html имеет отдельные sanitation semantics?
17. Что такое `display_name` и чем он связан с `_rec_name`?
18. Какие Access Log fields создаются framework при `_log_access`?
19. Какие reserved field names важно знать на этом уровне?
20. Почему relational и computed fields вынесены в следующие owners?

## Официальные источники

- `S1` — ORM API / Fields  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- `S2` — Server Framework 101 / Models and Basic Fields  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
