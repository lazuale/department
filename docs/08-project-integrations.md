# Штатные интеграции Project в Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [07 Аудит](07-deep-community-audit.md) · **08 Интеграции Project** · [09 Project и безопасность](09-project-productivity-security.md) · [10 API](10-external-integrations.md) · [11 Properties](11-relational-properties.md)

---

Project в Odoo 19 Community связан с другими приложениями не только через ручные ссылки. Значимая часть интеграции появляется через базовые relations, relational Properties и auto-install bridge modules.

## 1. Три уровня связи

### Уровень A — Task ↔ предметный объект

Для click-only ссылки сначала использовать relational Property:

```text
Task → fleet.vehicle
Task → hr.employee
Task → maintenance.equipment
Task → res.partner
Task → stock.lot [если модель доступна и сценарий подтверждён]
```

### Уровень B — документ ↔ Project

Штатные bridge modules добавляют прямые поля, например:

```text
Purchase Order → Project
Stock Picking  → Project
```

### Уровень C — Project ↔ analytic/financial layer

Base Project уже имеет Analytic Account, а bridges добавляют profitability/cost integrations.

## 2. Project ↔ Analytic Account

Public `project.project` содержит:

```python
account_id = fields.Many2one('account.analytic.account', ...)
```

Project зависит от `analytic`.

Использовать этот слой, если существует реальный финансовый вопрос:

- затраты проекта;
- доходы;
- материалы;
- расходы;
- закупки;
- profitability.

Не использовать Analytic Account вместо Process Property или Task classification.

## 3. `project_account`

Community bridge `project_account` добавляет profitability-интеграцию с accounting/analytic data.

Подтверждены секции/источники типа:

- Vendor Bills;
- Other Costs;
- Other Revenues.

Полнота Project Profitability зависит от установленных приложений и bridge modules.

Поэтому корректная формулировка:

> Community имеет модульную public-инфраструктуру Project profitability, а не гарантированно одинаковый полный dashboard на любой установке.

## 4. Project ↔ Purchase

`project_purchase` добавляет в `purchase.order`:

```python
project_id = fields.Many2one('project.project', ...)
```

Если Purchase Order относится к самостоятельной инициативе, связь уже штатная.

Не создавать текстовое Property `Проект закупки` ради той же связи.

Purchase Order при этом не становится Task.

## 5. Project ↔ Inventory

`project_stock` добавляет в `stock.picking` прямой `project_id`.

Это позволяет связать материальное перемещение с Project.

При этом отдельная Task может ссылаться на конкретный Equipment/Serial/Vehicle через relational Property, если процессу нужен именно такой уровень детализации.

Не путать:

```text
Picking → Project
Task → предметный record через Property
```

Это разные связи.

## 6. Project ↔ Stock Accounting

`project_stock_account` добавляет analytic handling stock movements в контексте Project.

Если отдельная инициатива использует материалы и стоимость движения важна, сначала тестировать штатную связку.

## 7. Project ↔ Expenses

`project_hr_expense` связывает Expenses с Project/analytic account.

Не создавать Project Task на каждый Expense только ради отображения в проекте.

Task нужна лишь при отдельном обязательстве поверх expense workflow.

## 8. Relational Properties дополняют bridges, а не конкурируют с ними

Пример:

```text
Stock Picking.project_id
= к какой инициативе относится перемещение

Task Property[Оборудование]
= по какому конкретному Equipment выполняется работа
```

Или:

```text
Purchase Order.project_id
= к какому Project относится закупка

Task Property[ТС]
= какое ТС является предметом отдельного обязательства
```

Не создавать вторую связь там, где bridge уже отвечает на тот же вопрос.

## 9. Inventory ↔ Maintenance bridge

`stock_maintenance` уже связывает Equipment со stock location и совпавшим serial/lot.

Поэтому для asset/process architecture сначала проверить:

```text
Inventory
+ Maintenance
+ stock_maintenance
+ Task Property[Оборудование]
```

а не писать собственную цепочку relations.

## 10. Employees ↔ Fleet / Maintenance

`hr_fleet` даёт Employee ↔ Vehicle history.

`hr_maintenance` даёт Employee ↔ Equipment allocation.

Task может при этом иметь:

```text
Property[Сотрудник]
Property[ТС]
Property[Оборудование]
```

только там, где эти dimensions нужны самому обязательству.

## 11. Import relational data

Официальный Import Odoo 19 умеет работать с relational fields через names/Database ID/External ID.

Для master data сохранять стабильный External ID.

Это важно для Fleet/Employees/Equipment/Contacts.

Для **dynamic Properties внутри Tasks** импорт нужно проверить отдельно: pseudo-field Properties не следует автоматически считать эквивалентным обычной schema Many2one при bulk import.

## 12. JSON-2 для cross-app integration

Если системная интеграция должна читать/писать Project и связанные модели, использовать JSON-2/API keys и фактическую `/doc` конкретной базы.

До custom API-module проверить:

- Project Task;
- task_properties;
- target model;
- bridge-generated fields;
- user ACL/record rules.

## 13. Cross-app security

Связь record с Project не отменяет права исходного приложения.

Пример:

```text
Project User видит Project
но не имеет Purchase access
```

Он не должен автоматически получать полный доступ к Purchase Order только из-за `project_id`.

Для каждого bridge тестировать под реальными ролями:

- видит ли section/smart button;
- может ли открыть record;
- может ли изменить;
- какие target records доступны relational Property.

## 14. Что больше не считаем gap

После обнаружения relational Properties не считать gap по умолчанию:

```text
Task → Fleet Vehicle
Task → Employee
Task → Maintenance Equipment
```

Они сначала реализуются Property Many2one/Many2many.

Также уже штатно подтверждены:

```text
Purchase Order → Project
Stock Picking → Project
Project → Analytic Account
Employee ↔ Fleet history
Employee ↔ Maintenance allocation
Inventory ↔ Maintenance [частично]
```

## 15. Возможные реальные cross-model gaps

Остаются кандидатами только после пилота:

- Property неудобен/медленен на нужном масштабе;
- требуется reverse relation на предметной model;
- API/BI требует обычного schema field;
- нужна server constraint;
- требуется специальная cross-model transaction logic;
- standard profitability/reporting не отвечает на нужный вопрос;
- нужна агрегированная аналитика, которой нет в standard report models.

## 16. Budget Management

Analytic Accounting подтверждён Community.

Budget Management не включается в гарантированный baseline: общая документация и setting существуют, но public `account_budget` не подтверждён в `odoo/odoo:19.0`.

## 17. Операционный Project vs инициатива

### Операционный Project отдела

Основные вопросы:

- backlog;
- Deadline;
- Waiting;
- external waiting;
- Assignee;
- предметные Properties;
- throughput.

Финансовые bridges могут быть не нужны вообще.

### Самостоятельная инициатива

При наличии закупок/материалов/расходов/экономического результата можно подключать штатный analytic/project integration layer.

## 18. Новый порядок проверки cross-model потребности

```text
1. предметная model
2. relational Property Task → record
3. bridge modules между приложениями
4. прямые model relations
5. standard views/reports
6. ACL/record rules
7. Import/External ID
8. JSON-2/API
9. реальный объём
10. residual gap
```

Только пункт 10 является основанием для custom integration module.

---

[← 07 — Аудит](07-deep-community-audit.md) · [09 — Project и безопасность →](09-project-productivity-security.md)
