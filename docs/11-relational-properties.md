# Relational Properties в Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [07 Аудит](07-deep-community-audit.md) · [08 Интеграции Project](08-project-integrations.md) · [10 API](10-external-integrations.md) · **11 Relational Properties**

---

Properties Odoo 19 — не только текст/Selection. Это штатный click-only механизм, который в том числе умеет ссылки на записи других моделей.

Это существенно отодвигает необходимость custom module, но Properties не идентичны обычным schema fields и имеют важные ограничения.

## 1. Подтверждённые типы Property

Официальная документация Odoo 19 подтверждает:

- Text;
- Multiline Text;
- HTML;
- Checkbox;
- Integer;
- Decimal;
- Monetary;
- Date;
- Date & Time;
- Selection;
- Tags;
- **Many2one**;
- **Many2many**;
- Separator.

Источник: [Property fields](https://www.odoo.com/documentation/19.0/applications/essentials/property_fields.html).

## 2. Many2one / Many2many выбирают настоящую Odoo-модель

Для relational Property задаются:

```text
Label
Field Type
Model
Domain
Default Value
```

Примеры:

```text
ТС           → fleet.vehicle
Сотрудник    → hr.employee
Оборудование → maintenance.equipment
Контрагент   → res.partner
```

Это не вручную набитый Selection, а ссылка на существующий record.

## 3. Public Community source подтверждает relations

`odoo/orm/fields_properties.py` поддерживает `type`, `comodel`, `domain` и relational values.

Web client содержит:

- `ModelSelector`;
- `DomainSelector`;
- `Many2XAutocomplete`;
- переход к выбранной записи.

Источники:

- [`fields_properties.py`](https://github.com/odoo/odoo/blob/19.0/odoo/orm/fields_properties.py);
- [`property_definition.js`](https://github.com/odoo/odoo/blob/19.0/addons/web/static/src/views/fields/properties/property_definition.js);
- [`property_value.js`](https://github.com/odoo/odoo/blob/19.0/addons/web/static/src/views/fields/properties/property_value.js).

## 4. Project Task Properties — штатная часть Community

`project.task` содержит:

```python
task_properties = fields.Properties(
    'Properties',
    definition='project_id.task_properties_definition',
    copy=True,
)
```

Definition хранится на Project.

Набор Properties одного операционного Project можно настроить через UI без Studio.

## 5. Search / Group By подтверждены

Standard Task Search View содержит:

```text
task_properties
Group By → Properties
```

Следовательно, Property участвует в оперативном поиске/группировке Tasks.

Источник: [`project_task_views.xml`](https://github.com/odoo/odoo/blob/19.0/addons/project/views/project_task_views.xml).

## 6. Правильная архитектура master data

```text
Fleet Vehicle
= источник истины по ТС

Task Property[ТС]
= ссылка на Fleet Vehicle
```

Не делать:

```text
Property Selection[ТС]
= тысячи вручную внесённых госномеров
```

То же для Employee, Equipment и Contacts.

## 7. Что Many2one Property даёт штатно

- выбор существующего record;
- единый display name;
- переход к target record;
- Domain;
- target-model ACL/record rules;
- filter/group в Project;
- `Show in cards`;
- отсутствие дублируемого справочника.

## 8. Почему Property всё же не обычный schema field

Property — pseudo-field.

Values/definition используют Properties/JSONB-механику, а не отдельную колонку:

```sql
project_task.vehicle_id
```

Поэтому нельзя автоматически предполагать обычные свойства DB Many2one.

## 9. Не предполагать без runtime-теста

- производительность на любом объёме;
- наличие Property в каждом SQL report model;
- DB foreign key/index;
- reverse relation;
- удобство любой server constraint;
- одинаково простой API/BI;
- одинаково простой bulk import.

Это предмет пилота.

## 10. Domain не является security

Domain ограничивает варианты выбора в UI.

Доступ к target record определяется:

- ACL;
- record rules;
- company/security context.

## 11. Create/Create Edit target records

Relational Property использует стандартный autocomplete и может позволить создание target record, если пользователь имеет create access соответствующей модели.

Для master data определить отдельно:

```text
кто создаёт Vehicles/Employees/Equipment
кто только выбирает существующие records
```

## 12. Критичное ограничение: Project Task Properties не включены в Chatter tracking по умолчанию

Это важная цена click-only Properties.

`project.task.task_properties` в public source объявлен без:

```python
tracking=True
```

А `mail.thread._track_get_fields()` включает в tracking только поля, у которых установлен атрибут `tracking`.

Источники:

- [`project_task.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task.py);
- [`mail_thread.py`](https://github.com/odoo/odoo/blob/19.0/addons/mail/models/mail_thread.py).

### Следствие

Не следует рассчитывать, что изменение:

```text
Property[ТС]: Vehicle A → Vehicle B
```

или:

```text
Property[Сотрудник]: Employee A → Employee B
```

автоматически появится в Chatter Project Task как обычное tracked-field изменение.

Это принципиально отличается, например, от `stage_id`, `user_ids`, `date_deadline`, `priority`, где tracking включён.

## 13. Mail engine технически умеет tracking Properties

При этом public `mail.tracking.value` **специально поддерживает property values**, включая:

- Many2one;
- Many2many;
- Tags;
- другие Property types.

Источник: [`mail_tracking_value.py`](https://github.com/odoo/odoo/blob/19.0/addons/mail/models/mail_tracking_value.py).

То есть ограничение не в том, что Odoo вообще не умеет отображать old/new Property values. Ограничение — стандартный `project.task.task_properties` не включён в tracked fields.

### Методический вывод

Если доказуемая история изменения предметной ссылки обязательна, это уже обоснованный критерий для:

1. отдельного правила ручной фиксации причины в Chatter; либо
2. минимальной доработки tracking; либо
3. обычного schema field с tracking, если одновременно есть другие основания.

Не писать большое приложение только ради этого.

## 14. Общего CRUD Audit Trail для Project Community не подтверждено

Официальный общий Audit Trail report документирован в Accounting и относится к изменениям, влияющим на бухгалтерский учёт.

Public Community repo не подтверждает отдельный универсальный audit-log app для всех Project fields/records.

Chatter tracking в Odoo работает по конкретным tracked fields.

Источник: [Chatter / field tracking](https://www.odoo.com/documentation/19.0/developer/reference/backend/mixins.html), [Accounting Audit Trail](https://www.odoo.com/documentation/19.0/applications/finance/accounting/reporting.html).

### Следствие

Не обещать:

> «Odoo штатно хранит полный audit всех изменений любого поля Task».

Корректно:

> Odoo хранит Chatter/history для отслеживаемых полей и коммуникаций; полный audit конкретного бизнес-поля нужно проверять отдельно.

## 15. Рекомендуемый Property `ТС`

```text
Label: ТС
Type: Many2one
Model: fleet.vehicle
Domain: <при необходимости>
Show in cards: <если полезно>
```

Пилот:

1. загрузить реальный тестовый Fleet;
2. создать Tasks;
3. autocomplete;
4. Filter;
5. Group By Properties;
6. List/Kanban;
7. ACL/create rights;
8. Export/API;
9. изменить Vehicle и проверить фактическую историю/Chatter;
10. решить, требуется ли audit этого изменения.

## 16. Property `Сотрудник`

```text
Type: Many2one
Model: hr.employee
```

Использовать когда Employee — предмет Task, а не исполнитель.

Если история изменения Employee-связи существенна, применяются те же ограничения tracking.

## 17. Property `Оборудование`

```text
Type: Many2one
Model: maintenance.equipment
```

Если процесс является обычным Maintenance Request, отдельная Project Task может не требоваться.

## 18. Property на Inventory Serial/Lot

Технически Many2one Property может ссылаться на `stock.lot`, если модель доступна в Model selector и ACL пользователя разрешает records.

Обязательно проверить на конкретной базе до объявления custom gap.

## 19. Когда Many2many оправдан

Many2many использовать только если один результат действительно относится к нескольким объектам.

Он усложняет:

- трактовку;
- группировку;
- интеграцию;
- аудит изменений.

Сначала проверить, не нужны ли отдельные Tasks.

## 20. Definition зависит от Project

### Плюс

Разные Projects могут иметь разные Properties без custom module.

### Минус

Если одно поле должно иметь одинаковую schema и tracking во множестве Projects, Property требует дисциплины конфигурации и может стать менее удобным, чем обычное field.

Для одного операционного Project это в основном плюс.

## 21. Properties и Task Templates

`task_properties` имеет `copy=True`.

Template-based copying делает Properties кандидатом на перенос в созданную Task.

Но динамический предметный объект (Vehicle/Employee) не нужно зашивать в template, если он меняется в каждом случае.

Проверить runtime.

## 22. Properties и Recurring Tasks

Source recurrence использует `copy_data()`, а `task_properties` имеет `copy=True`.

Официальная документация при этом не перечисляет Properties среди гарантированно копируемых полей.

Поэтому проверить runtime:

- Selection Property;
- Many2one;
- Many2many;
- нужность переноса конкретного значения в следующий период.

## 23. Properties и API

Через JSON-2 `/doc` проверить фактическое `task_properties` конкретной базы.

Для BI/API проверить:

- read format;
- relational values;
- search/domain;
- стабильность definitions;
- преобразование в витрину.

Если это объективно слишком сложно для критичной интеграции, schema field может быть оправдан.

## 24. Properties и Import

Обычный Odoo Import умеет relational fields, но bulk import **внутрь pseudo-field Properties** необходимо отдельно проверить.

Не считать поведение обычного Many2one и dynamic Property идентичным без теста.

## 25. Properties и Task Analysis

Operational Task Search умеет Group By Properties.

Но `project.report` не следует автоматически считать содержащим каждое dynamic Property как отдельную SQL-report dimension.

Различать:

```text
оперативный filter/group
vs
агрегированный reporting / BI
```

## 26. Критерии перехода к custom field

Обычный `fields.Many2one` / другая минимальная доработка рассматривается, если пилот доказывает:

- неприемлемую производительность;
- невозможность нужного report;
- нестабильность/сложность API;
- необходимость server constraint;
- reverse relation;
- единое поле across Projects;
- DB-level index/FK;
- сложность bulk import;
- **обязательную историю изменения, которой стандартный Project Property не даёт в нужном виде**.

Не является причиной само по себе:

```text
в Python-модели project.task нет vehicle_id
```

## 27. Новый baseline

```text
master data
→ relational Property
→ filter/group/view
→ security + tracking test
→ real-volume test
→ Import/API/report test
→ residual gap
→ минимальная доработка
```

Именно так Properties становятся сильным click-only механизмом, а не очередным источником скрытых ограничений.

---

[← 10 — API](10-external-integrations.md) · [12 — Коммуникации и вложения →](12-communication-documents.md)
