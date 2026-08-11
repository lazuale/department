# Relational Properties в Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [07 Углублённый аудит](07-deep-community-audit.md) · [08 Интеграции Project](08-project-integrations.md) · [10 API](10-external-integrations.md) · **11 Relational Properties**

---

Этот документ фиксирует главное техническое уточнение текущего аудита: **Properties Odoo 19 — не только текст/Selection, а полноценный click-only механизм ссылок на записи других моделей**.

Это существенно меняет границу между настройкой и custom module.

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

## 2. Many2one / Many2many выбирают реальную Odoo-модель

Для relational Property настраиваются:

```text
Label
Field Type
Model
Domain
Default Value
```

То есть Property может ссылаться не на вручную набитый список, а на существующий record нужной модели.

Примеры:

```text
ТС           → fleet.vehicle
Сотрудник    → hr.employee
Оборудование → maintenance.equipment
Контрагент   → res.partner
```

## 3. Public Community source это подтверждает

`odoo/orm/fields_properties.py` поддерживает definition с `type`, `comodel`, `domain` и relational values.

Web-клиент Property Definition содержит:

- `ModelSelector`;
- `DomainSelector`;
- настройку comodel.

Property Value использует стандартный `Many2XAutocomplete`, умеет открывать связанную запись и учитывает отсутствие доступа к target record.

Источники:

- [`fields_properties.py`](https://github.com/odoo/odoo/blob/19.0/odoo/orm/fields_properties.py);
- [`property_definition.js`](https://github.com/odoo/odoo/blob/19.0/addons/web/static/src/views/fields/properties/property_definition.js);
- [`property_value.js`](https://github.com/odoo/odoo/blob/19.0/addons/web/static/src/views/fields/properties/property_value.js).

## 4. Project Task Properties — штатная часть Community Project

`project.task` содержит:

```python
task_properties = fields.Properties(
    'Properties',
    definition='project_id.task_properties_definition',
    copy=True,
)
```

Definition хранится на Project.

Это означает: набор Properties зависит от Project и может быть настроен кликами для его Tasks.

## 5. Search / Group By подтверждены

Стандартный Task Search View содержит:

```text
field name="task_properties"
Group By → Properties
```

Следовательно, relational Property — не только поле для отображения. Оно участвует в штатном поиске/группировке Tasks.

Источник: [`project_task_views.xml`](https://github.com/odoo/odoo/blob/19.0/addons/project/views/project_task_views.xml).

## 6. Правильная архитектура master data

Relational Property **не заменяет** Fleet/Employees/Maintenance.

Правильно:

```text
Fleet Vehicle
= источник истины по ТС

Task Property[ТС]
= ссылка на Fleet Vehicle
```

Неправильно:

```text
Property Selection[ТС]
= вручную набитые тысячи госномеров
```

То же самое для Employees, Equipment и Contacts.

## 7. Почему это лучше текстового поля

Many2one Property даёт:

- выбор существующей записи;
- единый display name;
- переход к target record;
- Domain;
- штатные права target model;
- filter/group в Project;
- отсутствие ручного дублирования значений справочника.

## 8. Почему это всё-таки не обычный schema field

Property — pseudo-field.

Definition и values хранятся через Properties/JSONB-механику, а не как отдельная колонка вида:

```sql
project_task.vehicle_id
```

Поэтому свойства и эксплуатационные характеристики отличаются от обычного `fields.Many2one`.

## 9. Что нельзя предполагать без теста

Не считать автоматически, что relational Property:

- одинаково производителен на любом объёме;
- одинаково удобен во всех reports;
- автоматически появляется во всех SQL-report models;
- имеет обычный DB foreign key/index;
- обеспечивает reverse relation на target model;
- подходит для любой server constraint;
- одинаково прост для внешнего BI/API.

Это вопросы пилота, а не причины отказаться от Property заранее.

## 10. Domain не является security

Domain ограничивает варианты в UI.

Но доступ к target record определяется:

- ACL;
- record rules;
- company/security context.

Нельзя использовать Domain как замену правам.

## 11. Create/Create Edit target records

Стандартный relational Property использует Many2XAutocomplete и технически поддерживает create/create-edit, если пользователь имеет соответствующие права target model.

Это важно для master data governance.

Если исполнитель Project должен только выбирать существующий Vehicle, но не создавать Vehicles, create access Fleet должен быть ограничен на уровне Fleet ACL/groups.

## 12. Рекомендуемый Property `ТС`

```text
Label: ТС
Type: Many2one
Model: fleet.vehicle
Domain: <при необходимости>
Show in cards: <если полезно>
```

Пилот:

1. создать 100+ Fleet records либо загрузить тестовый набор;
2. создать Tasks с разными Vehicles;
3. проверить autocomplete;
4. проверить Filter;
5. проверить Group By Properties;
6. проверить List/Kanban;
7. проверить отсутствие create rights у обычного пользователя;
8. проверить экспорт/API.

## 13. Property `Сотрудник`

```text
Label: Сотрудник
Type: Many2one
Model: hr.employee
```

Использовать только когда Employee является **объектом** Task.

Не путать с Assignee.

## 14. Property `Оборудование`

```text
Label: Оборудование
Type: Many2one
Model: maintenance.equipment
```

Если процесс является Maintenance Request, отдельная Project Task может вообще не требоваться.

## 15. Property на Inventory Serial/Lot

Технически Many2one Property позволяет выбрать модель `stock.lot`, если она доступна пользователю и Model selector позволяет её выбрать в конкретной базе.

Это нужно **проверить на стенде**, прежде чем объявлять Task → Serial/Lot custom gap.

При этом Inventory + Maintenance bridge `stock_maintenance` уже частично связывает serial/location с Equipment.

## 16. Когда Many2many оправдан

Использовать, если один контролируемый результат действительно относится к нескольким records.

Но Many2many усложняет:

- трактовку ответственности;
- группировку;
- аналитику;
- интеграцию.

Поэтому сначала проверить, не являются ли это несколько отдельных Tasks.

## 17. Property definition зависит от Project

Это сильная и слабая сторона одновременно.

### Плюс

Разные Projects могут иметь разные предметные Properties без Studio/custom module.

### Минус

Если одно поле должно иметь абсолютно одинаковую схему во множестве Projects, требуется дисциплина конфигурации или другая модель.

Для нашей базовой методики с одним операционным Project это скорее плюс.

## 18. Properties и Task Templates

`task_properties` имеет `copy=True`.

Task Templates используют штатное копирование Task data, поэтому relational Properties являются кандидатом на перенос в созданную из template Task.

Но конкретный сценарий нужно прогнать на стенде, особенно для default/динамических предметных объектов.

В шаблон не следует зашивать конкретный Vehicle, если он меняется в каждом случае.

## 19. Properties и Recurring Tasks

Публичное поле `task_properties` имеет `copy=True`, но официальная user documentation Recurring Tasks перечисляет гарантированно копируемые поля не исчерпывающе относительно source implementation.

Для методики правило простое:

> Поведение нужных Properties в recurrence проверяется на пилоте до запуска критичной периодики.

Не строить процесс на непроверенном предположении.

## 20. Properties и API

JSON-2 `/doc` должен показать фактическое поле `task_properties` конкретной базы.

Для BI/API отдельно проверить:

- read result;
- формат relational Property values;
- поиск/domain по Property;
- стабильность Property definition names/IDs;
- удобство трансформации в аналитическую витрину.

Если API/BI по Properties оказывается существенно сложнее обычного schema field и это критично — тогда это нормальное основание обсуждать минимальную доработку.

## 21. Properties и Import

Import умеет relational fields в обычных моделях, но работу импорта **внутрь pseudo-field Properties** не следует считать равной обычному Many2one без практической проверки.

Для массового создания Tasks с relational Properties необходимо отдельно прогнать импорт на тестовом наборе.

Если стандартный Import неудобен, остаются JSON-2/Automation/custom minimal logic — выбор делается после теста.

## 22. Properties и Task Analysis

Project Task Search View умеет Group By Properties.

Но SQL report model `project.report` не следует автоматически считать содержащей каждое динамическое Property как полноценное аналитическое поле.

Поэтому различать:

```text
оперативная filter/group analytics по Tasks
vs
агрегированная reporting model / BI
```

Для первой relational Properties уже подходят штатно.

Для второй нужно проверить фактический вопрос и инструмент.

## 23. Критерии перехода к custom field

Обычный `fields.Many2one` проектируется только если пилот доказывает хотя бы один пункт:

- неприемлемая производительность Property;
- невозможность нужного отчёта;
- сложность/нестабильность API integration;
- нужна server constraint;
- нужен reverse relation;
- нужна одинаковая schema across Projects;
- нужен DB-level index/foreign key;
- стандартные Automation/import paths недостаточны.

Не является причиной:

```text
в Python-модели project.task нет vehicle_id
```

## 24. Новый baseline методики

```text
master data model
→ relational Property
→ filter/group/view
→ pilot at real volume
→ API/report/security test
→ только затем residual gap
```

Это значительно сдвигает границу Odoo 19 Community в сторону **реальной click-only адаптации**.

---

[← 10 — API](10-external-integrations.md) · [Главная](../README.md)
