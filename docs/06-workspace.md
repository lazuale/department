# Настройка Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [16 Модули](16-community-modules-context.md) · [19 Master data](19-master-data.md) · [26 Task ↔ master data](26-task-master-data-relations.md)

---

Это последовательный пилот текущей архитектуры Odoo 19 Community.

Его задача — проверить принятые master data, связи с `project.task`, реальные рабочие сценарии, права и аналитику до появления custom addon.

## 1. Установить согласованный базовый контур

Master data уже определили обязательные предметные Apps:

```text
Project      project
Contacts     contacts
Fleet        fleet
Maintenance  maintenance
Inventory    stock
```

`Discuss / mail` входит в рабочую и техническую инфраструктуру выбранных приложений.

Назначение:

```text
project.task             → работа
res.partner              → люди
fleet.vehicle            → путевая техника
maintenance.equipment    → сканирующие рамки
stock.package            → терминал Nobilis как комплект
product.product          → тип комплектующей Nobilis
stock.lot                → конкретная серийная комплектующая
stock.location / quant   → положение Nobilis
stock.move / move.line   → движение и изменение состава
```

Employees / HR не устанавливается ради справочника людей.

Purchase, Repair, Calendar, Website и прочие кандидаты не входят в baseline только потому, что они существуют.

Полное решение по модулям: [16 — Реестр модулей](16-community-modules-context.md).

## 2. Проверить автоматически установленный технический слой

После установки базовых Apps проверить фактический список модулей.

Из выбранной архитектуры ожидаются, в частности:

```text
product
barcodes
barcodes_gs1_nomenclature
project_todo
project_stock
stock_maintenance
api_doc
и другие dependencies выбранных Apps
```

Критичные bridges:

```text
Project + Inventory
→ project_stock

Inventory + Maintenance
→ stock_maintenance
```

`project_stock` связывает `stock.picking` с `project.project`; он не заменяет связь Task с Package/Lot.

`stock_maintenance` добавляет Equipment связь с `stock.location` и сопоставление с `stock.lot` по serial. Это не означает автоматического решения использовать Stock для всех рамок и участков.

## 3. Загрузить master data

Сначала загружаются предметные справочники, только затем строятся Tasks.

```text
люди             → res.partner
путевая техника  → fleet.vehicle
рамки             → maintenance.equipment
Nobilis           → Stock models
```

Для импорта:

- проверять уникальность;
- сохранять External ID при повторных обновлениях;
- не создавать дубли;
- заранее определить источник истины;
- проверить права на импорт и редактирование.

Один человек должен соответствовать одной записи `res.partner`. Выдача доступа через `res.users` не создаёт второго человека.

Task не копирует master data текстом.

## 4. Проверить Nobilis в нативном Stock

На реальных тестовых данных проверить:

```text
Терминал T-001
→ stock.package

Основной блок SN-...
Компонент A SN-...
Компонент B SN-...
→ stock.lot
```

Сценарии:

1. собрать Package из серийных частей;
2. переместить весь Package между locations;
3. снять одну часть;
4. поставить другую часть;
5. переставить часть между двумя терминалами;
6. хранить часть отдельно;
7. вернуть часть производителю;
8. вернуть целый комплект производителю;
9. восстановить traceability по конкретному Serial Number.

Если эти сценарии закрываются штатным Stock, собственная terminal model не вводится.

## 5. Проверить рамки в Maintenance

Для реальной рамки проверить:

```text
maintenance.equipment
├── идентичность рамки
├── инвентарный номер
├── состояние
└── maintenance requests
```

Отдельно проверить автоматически появившийся `location_id → stock.location` из `stock_maintenance`, но не принимать его как общую модель участка до отдельного решения по master data участков.

## 6. Проверить путевую технику во Fleet

В один предметный реестр входят:

```text
самосвалы
строительная техника
малая механизация
→ fleet.vehicle
```

Проверить импорт фактического списка техники, уникальные идентификаторы, поиск и пригодность категорий/моделей.

Fleet не превращается в систему путевых листов. Путевые листы остаются в действующей учётной системе; Odoo связывает управляемую работу с нужной единицей техники.

## 7. Настроить связь Task с master data

Согласованный пилотный механизм — relational Properties.

```text
Исполнитель
→ штатный project.task.user_ids → res.users

Заявитель
→ Many2one(res.partner)

Связанные люди
→ Many2many(res.partner)

Техника
→ Many2one / Many2many(fleet.vehicle)

Рамка
→ Many2one(maintenance.equipment)

Терминал
→ Many2one(stock.package)

Комплектующие
→ Many2many(stock.lot)
```

`project.task.partner_id` не переименовывается в «Заявитель»: в штатном Project это Customer.

Подробно: [26 — Task ↔ master data](26-task-master-data-relations.md).

## 8. Проверить Properties до решения о custom fields

На реальном объёме проверить:

- autocomplete и выбор records;
- открытие target record;
- права target model;
- List / Kanban;
- Search / Filter / Group By;
- Task Analysis;
- импорт / массовое заполнение;
- JSON-2;
- несколько Projects;
- производительность;
- историю изменений, если она нужна процессу.

Обычные ORM-поля добавляются только при доказанном gap.

## 9. Определить границы Projects

Не использовать автоматически правила:

```text
один отдел = один Project
один процесс = один Project
```

Новый Project оправдан реальной границей:

- visibility;
- отдельный lifecycle;
- substantially different workflow;
- milestones / project-level reporting;
- самостоятельная инициатива.

Если различие нужно только для классификации — использовать Properties / Tags / Views.

Важно: Properties определяются на уровне Project, поэтому сценарий нескольких Projects обязательно входит в runtime-проверку.

## 10. Настроить workflow только по реальным процессам

Для каждого используемого Stage определить:

```text
что означает вход
что означает нахождение
что означает выход
```

Не создавать Stages только ради копирования старой методики.

Не дублировать без причины системные `Done`, `Canceled` и dependency-driven `Waiting`.

## 11. Исполнители, сроки и приоритет

Assignees остаются `res.users`.

Не создавать fake users для очередей, групп или людей без доступа.

Deadline используется только при реальном сроке результата.

Activity Due Date — срок следующего действия, а не второй Deadline Task.

Priority используется только после определения смысла уровней в конкретном процессе.

## 12. Activities, Dependencies и Subtasks

```text
Activity
→ следующее действие / follow-up

Dependency
→ реальная блокировка одной Task другой

Subtask
→ самостоятельная управляемая часть результата
```

Не превращать технические шаги в лишние Tasks/Subtasks.

## 13. Templates и Recurrence

```text
типовая разовая работа
→ Task Template

календарно повторяемая работа
→ Recurring Task
```

Если процесс зависит от копирования Properties/Subtasks в recurrence, проверить это runtime на стенде.

## 14. Права

Сначала описать фактические действия пользователя:

```text
read
create
write
delete
какие records видит
какие master data может менять
```

Только после этого настраивать:

```text
Groups
Access Rights
Record Rules
Project visibility
```

Domain у relational Property ограничивает выбор, но не заменяет security target model.

## 15. Входящие каналы

Сначала должна работать ручная модель.

Затем по реальной потребности проверяются:

- Email Alias;
- Website + `website_project`;
- JSON-2;
- Automation / webhook.

Не добавлять Website только ради того, что Portal технически существует.

## 16. List, Kanban и аналитика

На реальном потоке проверить:

```text
List
Kanban
Shared Views
Task Analysis
Pivot / Graph
My Dashboard
```

Views и KPI создаются под устойчивые управленческие вопросы, а не под все доступные поля.

## 17. Следующие кандидаты на модули

После принятой master data остаются только процессные кандидаты:

```text
base_automation
website + website_project
calendar
data_recycle
survey
purchase
```

Каждый включается только после появления конкретного требования.

Не считать ближайшими кандидатами без нового процесса:

```text
hr
repair
mrp
account
crm / sale
hr_timesheet
hr_attendance
hr_work_entry
```

## 18. Runtime checklist

Перед выводом «нужна доработка» проверить минимум:

### Master data

- реальные люди;
- реальную выборку путевой техники;
- несколько рамок;
- несколько терминалов Nobilis и их компонентов;
- External IDs;
- уникальность;
- права;
- архивирование/активность records.

### Stock

- Package contents;
- serial traceability;
- internal transfers;
- замена компонентов;
- возврат производителю;
- `project_stock`;
- `stock_maintenance`.

### Tasks

- реальные Tasks;
- Assignees;
- Stages / State;
- Deadline;
- Activities;
- Dependencies;
- Subtasks;
- Templates;
- Recurrence.

### Relations

- `res.partner`;
- `fleet.vehicle`;
- `maintenance.equipment`;
- `stock.package`;
- `stock.lot`;
- Many2one / Many2many Properties;
- поиск / фильтрация / группировка;
- API / импорт;
- несколько Projects.

### Security

- обычный пользователь;
- руководитель, если ему нужны отдельные полномочия;
- администратор;
- master-data read/write;
- видимость Tasks и target records.

## 19. Когда писать custom addon

Последовательность:

```text
1. Правильно ли выбрана штатная сущность?
2. Есть ли нативная relation / Property?
3. Есть ли auto-install bridge?
4. Решается ли вопрос стандартным workflow / Activity / View?
5. Подтверждён ли gap реальными данными на стенде?
```

Если gap подтверждён — делается минимальная доработка именно на него.

Не нужно избегать custom addon любой ценой. Нужно избегать собственной системы поверх уже работающих моделей Odoo.

---

[← 05 — Процессы](05-processes.md) · [16 — Модули](16-community-modules-context.md) · [19 — Master data](19-master-data.md) · [26 — Task ↔ master data](26-task-master-data-relations.md) · [Главная](../README.md)
