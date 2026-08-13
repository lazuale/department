# Граница возможностей Odoo 19 Community

[Главная](../README.md) · **00 Возможности** · [01 Модель](01-methodology.md) · [03 Аналитика](03-control.md) · [06 Настройка](06-workspace.md) · [15 Перепроверка](15-capability-recheck.md) · [17 Люди](17-master-data-people.md)

---

Этот документ — **авторитетная техническая граница методики**.

Если другой текст PR противоречит этому документу или [повторной проверке](15-capability-recheck.md), использовать более осторожную формулировку.

## 1. Как определяется Community-возможность

Функция считается пригодной для методики только после проверки:

1. официальной документации Odoo 19.0 — что функция делает;
2. public `odoo/odoo:19.0` — существует ли она в Community source;
3. стандартного UI — можно ли реально использовать её без собственного кода;
4. bridge modules — нет ли уже штатной связи между приложениями;
5. редакционных зависимостей — не требует ли общий workflow Enterprise-приложение.

Наличие документации или технического модуля **по отдельности недостаточно**.

## 2. Классы возможностей

| Класс | Значение | Решение |
|---|---|---|
| A | public Community + подтверждённый пользовательский UI | можно включать в click-only baseline |
| B | public Community, но административный/технический слой | использовать по потребности |
| C | infrastructure/source есть, полный workflow не подтверждён | только runtime/edition test |
| D | Community не подтверждена | не использовать в baseline |

---

## 3. Главная архитектура

Odoo не превращается в одну универсальную таблицу Tasks.

```text
предметная сущность
→ ссылка из Task при необходимости
→ управленческий контроль
```

| Реальный объект | Штатная модель / приложение |
|---|---|
| обязательство / контролируемый результат | `project.task` |
| человек / сотрудник компании | `res.partner` |
| пользователь-исполнитель | `res.users` |
| ТС | `fleet.vehicle` после проверки пригодности Fleet |
| оборудование | `maintenance.equipment` |
| обслуживание | `maintenance.request` |
| serial / lot / location / movement | Inventory (`stock.*`) |
| встреча | `calendar.event` |
| личная мысль | To-Do |
| следующее действие | `mail.activity` |
| обсуждение записи | Chatter |
| командное обсуждение | Discuss |
| присутствие | Attendances |
| отсутствие | Time Off |
| HR work intervals | Work Entries |
| навык / сертификат | Skills / Certifications |
| обучение | eLearning |
| тест / анкета | Surveys |
| закупка | Purchase |
| расход сотрудника | Expenses |

Для текущего рабочего контура master data людей зафиксирован отдельно: **[17 — Master data: люди](17-master-data-people.md)**. Employees / `hr.employee` является доступной Community-возможностью, но не входит в baseline только ради справочника сотрудников.

> **Task — работа над объектом, а не сам объект.**

---

# 4. Project / Tasks — класс A

Подтверждённое Community-ядро:

- Projects;
- Tasks;
- Task Stages;
- Task States;
- Assignees;
- Deadline;
- Allocated Time;
- Priority 0–3;
- Tags;
- Properties;
- Chatter / followers / attachments;
- Activities;
- Activity Types;
- Activity chaining;
- Activity Plans;
- Subtasks;
- Dependencies / `Blocked by`;
- computed `Waiting`;
- Recurring Tasks;
- Task Templates;
- Project Templates;
- Project Roles;
- Milestones;
- Project Updates;
- Project Dashboard / Burndown;
- Project Stages;
- Project-level Activities;
- Rotting / Days to rot;
- email alias;
- Project sharing/collaboration;
- Task sharing;
- Task Analysis;
- List / Kanban / Calendar / Activity / Pivot / Graph;
- mass edit в Task List;
- custom Project Top Bar views.

Стандартный Community Tasks action не включает Gantt.

---

## 5. Task States

Обычные пользовательские состояния:

```text
In Progress
Changes Requested
Approved
Done
Canceled
```

`Waiting` — вычисляемое состояние при незакрытой Task Dependency.

Не описывать `Waiting` как обычный шестой ручной State.

---

## 6. Priority

Public model подтверждает:

```text
0 Low
1 Medium
2 High
3 Urgent
```

Quick-create parser также поддерживает:

```text
!   → Medium
!!  → High
!!! → Urgent
```

Методика не обязана использовать все четыре значения.

---

# 7. Relational Properties — класс A

Odoo 19 Property поддерживает:

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

Для relational Property доступны:

```text
Model
Domain
Default Value
```

Поэтому click-only можно создать:

```text
Property[ТС]           → Many2one(fleet.vehicle)
Property[Сотрудник]    → Many2one(res.partner)
Property[Оборудование] → Many2one(maintenance.equipment)
```

`project.task` содержит `task_properties`, а стандартный Task search поддерживает поиск и `Group By → Properties`.

### Правильная схема

```text
Fleet Vehicle
= master data

Task Property[ТС]
= ссылка на master record
```

Не создавать Selection со списком тысяч госномеров.

### Граница

Properties — pseudo-fields/JSONB, а не обычные schema columns.

`task_properties` в standard Project не объявлен как обычное `tracking=True` поле.

Custom field обсуждается только после измеренного ограничения:

- производительность;
- обязательная history/audit;
- server constraint;
- reverse relation;
- API/BI;
- bulk import;
- DB index/FK;
- единая schema across Projects.

---

# 8. Shared Views и My Dashboard — класс A

## Shared Views

Официальный Project workflow подтверждает:

```text
настроить view
→ top bar sliders
→ Save View
→ Shared
```

Это основной способ создавать общие руководительские и исполнительские срезы без размножения Projects.

## My Dashboard

Public module:

```text
board
```

предназначен для пользовательских dashboards и зависит от `spreadsheet_dashboard`.

Официальный workflow:

```text
открыть List / Kanban / Pivot / Graph
→ Actions
→ Dashboard
→ Add to my Dashboard
```

My Dashboard **не основан на Spreadsheet**.

Для операционного управления это основной dashboard layer Community.

---

# 9. Spreadsheet / Spreadsheet Dashboard — класс C для custom build

В public Community действительно есть:

```text
spreadsheet
spreadsheet_dashboard
```

и соответствующие model/rendering components.

Но повторная проверка показала:

- standard `spreadsheet.dashboard` configuration list имеет `create="false"`;
- public dashboard JS ожидает server method `action_edit_dashboard`;
- реализация полного edit/create workflow не подтверждена в public Community model layer;
- официальная документация для построения dashboard с нуля требует `Dashboards / Admin` + минимум `Documents / User`;
- Spreadsheet documentation строит основной authoring workflow через Documents.

Следовательно:

> **наличие `spreadsheet*` модулей не равно гарантированному click-only конструктору custom Spreadsheet Dashboards в чистой Community.**

Baseline аналитики:

```text
Shared Views
→ Task Analysis
→ My Dashboard
→ API / внешний BI по необходимости
```

Spreadsheet Dashboard возвращается только после runtime/edition-проверки.

---

# 10. Task Templates — класс A

Public Community содержит полноценные Task Templates внутри Project.

Подтверждены:

- `is_template`;
- dropdown templates;
- создание Task из template;
- копирование child tasks;
- исключение templates из обычного open backlog;
- копирование templates вместе с Project.

Использование:

```text
типовая разовая работа по событию
→ Task Template
```

Не путать с Recurring Task и Project Template.

---

# 11. Recurring Tasks — класс A, но subtasks требуют runtime test

Подтверждены units:

```text
Days
Weeks
Months
Years
```

Hours нет.

Следующий occurrence создаётся после закрытия предыдущей Task.

### Расхождение

Официальная документация говорит, что Subtasks не копируются.

Текущий public source 19.0 рекурсивно создаёт `child_ids`.

До runtime-теста не считать ни одно из этих поведений гарантированным для критического процесса.

---

# 12. Project Templates / Roles — класс A

Использовать для повторяемой структуры самостоятельной инициативы.

Подтверждены:

- Project Template;
- Project Roles;
- назначение assignees по roles;
- перенос структуры Project/Tasks/Subtasks.

## Availability-based scheduling — не baseline

Общая документация описывает task scheduling с учётом workload/availability/time off/working schedules.

Соответствующий planned-date task model layer не подтверждён в public Community `project.task`.

Поэтому Project Template **не считается заменой Planning**.

---

# 13. Dependencies между Projects — класс A

`Blocked by` не ограничен тем же Project.

Следовательно, можно связать:

```text
Task операционного Project
↔ dependency ↔
Task отдельной инициативы
```

Это позволяет отделять самостоятельные инициативы без потери dependency-механизма.

---

# 14. Rotting / time in Stage — класс A с аналитической границей

Odoo умеет:

- `Days to rot` на Stage;
- `Rotting` filter;
- duration tracking по `stage_id`;
- показывать длительность Stages конкретной Task.

Но готовый агрегированный SLA/time-in-stage report для всего потока не считается подтверждённым стандартным Pivot/Graph.

---

# 15. Website Form → Task — класс A по потребности

Public module:

```text
website_project
```

создаёт Tasks из формы Website.

Это связка:

```text
Website + Project
```

а не функция голого Project.

Не устанавливать Website только ради гипотетического будущего входящего канала.

---

# 16. Automation Rules / Webhooks — класс B

Public LGPL module:

```text
base_automation
```

подтверждает UI Automation Rules и triggers, включая:

- create;
- create/write;
- change;
- time;
- state;
- priority;
- stage;
- tag;
- webhook.

Automation Rules — административный/технический слой.

Не строить процесс сразу из automation.

Порядок:

```text
сущность
→ workflow
→ activity/template/recurrence
→ стабильная ручная работа
→ только затем automation
```

---

# 17. JSON-2 / API Documentation — класс B

Public Community 19 подтверждает endpoint:

```text
POST /json/2/<model>/<method>
auth = bearer
```

Hidden auto-install module:

```text
api_doc
```

предоставляет `/doc` по фактической базе:

- models;
- fields;
- methods;
- examples;
- playground.

Для новых интеграций использовать ORM/API path раньше прямого SQL write.

---

# 18. Import / Export — класс A/B

Стандартный Import подходит для master data и повторных обновлений.

Ключевое правило:

```text
External ID
→ сохранить
→ использовать для последующих обновлений и relations
```

Обычные relational fields поддерживаются import-механизмом.

Импорт **в dynamic Properties** считать отдельным runtime test, а не автоматически равным обычному Many2one field.

---

# 19. Working Schedules — класс A; rotating 2/2 planner — D/C

Resource Calendar умеет:

- обычный weekly schedule;
- two-week calendar;
- First / Second week.

Чистый цикл `2/2` — четырёхдневный, поэтому не равен бесконечно повторяющемуся two-week pattern.

Resource Calendar не выдавать за Planning/roster engine.

---

# 20. Предметные Community-приложения

По потребности подтверждены public Community-контуры:

- Employees;
- Contacts;
- Fleet;
- Maintenance;
- Inventory;
- Calendar;
- Timesheets;
- Attendances;
- Time Off;
- Work Entries;
- Remote Work;
- Recruitment;
- Expenses;
- Skills / Certifications;
- Surveys;
- eLearning;
- Purchase;
- Purchase Agreements;
- Analytic Accounting;
- Live Chat / Discuss;
- Data Recycle.

Специализированный workflow используется **только для своего предмета**.

Например Purchase не заменяет универсальное согласование.

---

# 21. Штатные bridge modules

Перед custom relation обязательно проверять bridges.

Подтверждены, среди прочего:

```text
Employees + Fleet        → hr_fleet
Employees + Maintenance  → hr_maintenance
Inventory + Maintenance  → stock_maintenance
Skills + Surveys         → hr_skills_survey
Skills + eLearning       → hr_skills_slides
Employees + Calendar     → hr_calendar

Project + Accounting     → project_account
Project + Purchase       → project_purchase
Project + Inventory      → project_stock
Project + Stock Account  → project_stock_account
Project + Expenses       → project_hr_expense
```

Bridge = связь моделей, а не обещание полного cross-app отчёта. HR bridges учитываются только если HR когда-либо будет включён по отдельной потребности; в текущий baseline они не входят.

---

# 22. Contacts data quality

Для Contacts подтверждены штатные Merge/Deduplicate механизмы.

Community также имеет `data_recycle`.

Не переносить на Community весь Enterprise Data Cleaning/Data Merge toolkit.

---

# 23. Attachments

Класс A:

- attachments в Chatter;
- attachment records;
- базовый поиск по metadata/name.

Public Community также содержит техническую indexation части содержимого файлов.

Но полноценный Documents/full-text document UX не считается подтверждённым Community baseline.

---

# 24. Security / authentication

Подтверждены public Community:

- standard ACL;
- record rules;
- Internal / Portal users;
- Project visibility/collaboration;
- LDAP;
- OAuth2;
- TOTP 2FA;
- Passkeys.

Domain relational Property — не security.

Права target model определяются ACL/record rules.

---

# 25. Что не является Community baseline

Не строить методику на:

```text
Studio
Helpdesk
Approvals
Documents
Knowledge
Planning
Project Gantt
Sign
Appointments
полном Quality app
Enterprise Data Cleaning/Data Merge
полном Inventory Barcode UI
Budget Management без отдельной редакционной проверки
availability/capacity task scheduling
произвольном rotating roster
custom Spreadsheet Dashboard без runtime/edition test
```

---

# 26. Остаточные gaps определяются только пилотом

Нормальные кандидаты:

- history/audit изменения relational Property;
- aggregated time-in-stage / SLA reporting;
- rotating 2/2 / capacity planning;
- BPM transition matrix по ролям;
- Property не выдерживает объём/API/BI/import;
- специализированная предметная бизнес-логика;
- cross-model analytics, которой нет в штатных reports/API.

Ненормальный кандидат:

```text
«в project.task нет vehicle_id, поэтому сразу пишем модуль»
```

---

# 27. Приоритетный click-only стек методики

```text
master data apps
→ relational Properties
→ bridge modules
→ Project Task workflow
→ Activities / Dependencies / Templates / Recurrence
→ Shared Views
→ Task Analysis
→ My Dashboard
→ Import / API / Automation по потребности
→ runtime pilot
→ residual gaps
→ минимальный custom module только на подтверждённые gaps
```

---

# 28. Что обязательно проверить на стенде

1. Fleet на реальном составе ТС;
2. Many2one Property `ТС` на реальном объёме;
3. Property `Сотрудник → res.partner` на реальном справочнике людей;
4. Property filter/group performance;
5. Property history в Chatter;
6. Task Import + Properties;
7. Properties через JSON-2;
8. ACL Fleet/Contacts/Maintenance под обычным Project User;
9. Task Template + Properties;
10. Recurring Task + Properties;
11. Recurring Task + Subtasks;
12. две daily recurrence chains для handover;
13. My Dashboard с реальными shared views;
14. Website Form → Inbox triage при необходимости;
15. email alias;
16. Spreadsheet Dashboard create/edit в чистой CE — только как редакционная проверка, не prerequisite;
17. русская локализация фактических menus/fields.

---

[← Главная](../README.md) · [15 — Перепроверка →](15-capability-recheck.md) · [17 — Master data: люди →](17-master-data-people.md)
