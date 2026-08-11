# Управление работой отдела в Odoo 19 Community

Практическая методика управления операционной работой, справочниками, контролем и аналитикой в **стандартном Odoo 19 Community**.

Методика строится вокруг реальных штатных возможностей: предметные данные живут в своих Odoo-моделях, а Project управляет **обязательствами и контролируемыми результатами**.

**[Граница возможностей Community →](docs/00-odoo19-community.md)**  
**[Модель управления →](docs/01-methodology.md)**  
**[Click-only настройка →](docs/06-workspace.md)**  
**[Повторная проверка спорных возможностей →](docs/15-capability-recheck.md)**

## Как проверяются возможности

Для каждого существенного решения:

1. поведение сверяется с официальной документацией **Odoo 19.0**;
2. принадлежность Community при необходимости проверяется в публичной ветке **`odoo/odoo:19.0`**;
3. перед объявлением разрыва проверяются штатные bridge modules;
4. наличие технического модуля не считается доказательством полного click-only workflow;
5. до custom module проверяются relational Properties, standard views/reports, Import/API/Automation и реальный пилот.

Наличие функции в общей документации Odoo само по себе не считается доказательством её наличия в Community.

## Главный принцип

> **Task — контролируемый результат, а не универсальная карточка объекта.**

Master data:

```text
Employees   → сотрудники
Contacts    → внешние люди/организации
Fleet       → ТС, если модель подходит реальному парку
Maintenance → оборудование и обслуживание
Inventory   → serial / lot / location / movement
```

Project Task хранит работу:

```text
результат
ответственный
Deadline
Stage / State
Priority
Activities
Dependencies
Properties
Chatter
```

## Ключевая находка углублённого аудита: relational Properties

Odoo 19 Properties штатно поддерживают **Many2one и Many2many к выбранной модели**.

Поэтому Task можно связать кликами с предметным справочником:

```text
Property[ТС]           → Many2one(fleet.vehicle)
Property[Сотрудник]    → Many2one(hr.employee)
Property[Оборудование] → Many2one(maintenance.equipment)
```

Это не копия справочника. Источник истины остаётся в Fleet/Employees/Maintenance, а Property хранит ссылку на record.

Следовательно, отсутствие обычного Python-поля `project.task.vehicle_id` **само по себе больше не является основанием для custom module**.

Сначала relational Property проверяется на реальном объёме, в filter/group, правах, Import/API и BI.

Важная граница: стандартный `task_properties` не включён в Chatter tracking как обычное `tracking=True` поле. Если доказуемая история смены предметной ссылки обязательна, это уже отдельный критерий для минимальной доработки или специального правила фиксации изменения.

## Базовый операционный Project

Для постоянной работы одного отдела обычно достаточно одного Project:

```text
Входящие
→ Очередь
→ В работе
→ Ожидание внешнего
→ На проверке   [только если реально необходимо]
```

Закрытие:

```text
Done / Canceled
```

Внутренняя блокировка:

```text
Blocked by
→ computed Waiting
```

Dependencies могут связывать Tasks разных Projects, поэтому самостоятельную инициативу можно вынести в отдельный Project без потери `Blocked by`.

## Рабочее место без разработки

Подтверждены и включены в click-only baseline:

- Shared Favorites;
- Project Top Bar `Save View → Shared`;
- List / Kanban / Calendar / Activity / Pivot / Graph;
- **mass edit в Task List**;
- Task Analysis;
- **My Dashboard**;
- Rotting;
- Activities и Activity Plans;
- Dependencies;
- Recurring Tasks;
- **Task Templates**;
- Project Templates и Project Roles;
- Project Stages для портфеля инициатив;
- Milestones / Project Updates / Burndown для инициатив.

Для большого входящего потока **List рассматривается как диспетчерское рабочее место**, а Kanban — как визуализация потока по Stages.

### Важная поправка про Spreadsheet Dashboard

Public Community содержит `spreadsheet` и `spreadsheet_dashboard`, но повторная проверка показала, что наличие этих модулей **не доказывает полноценный click-only create/edit workflow собственного Spreadsheet Dashboard в чистой Community**.

Официальный workflow построения dashboard с нуля требует как минимум прав `Documents / User`, а Documents не является нашей Community-базой.

Поэтому основной путь аналитики сейчас:

```text
Shared Views
→ Task Analysis
→ My Dashboard
→ API / внешний BI при необходимости
```

Spreadsheet Dashboard используется только после runtime-проверки конкретной Community-инсталляции и доступного редакционного слоя.

## Четыре разных механизма повторяемости

```text
типовая разовая Task по событию
→ Task Template

одинаковая Task по расписанию
→ Recurring Task

типовой набор follow-up
→ Activity Plan / Activity chaining

повторяемая структура проекта
→ Project Template + Project Roles
```

Automation Rule нужен только если более простой механизм не решает задачу.

По Recurring Tasks есть расхождение документации и текущего public source по копированию Subtasks, поэтому этот сценарий проверяется runtime и не считается гарантированным заранее.

## Штатные bridge modules

В ходе аудита подтверждены, среди прочего:

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

Поэтому две отдельные модели нельзя объявлять несвязанными, пока не проверены штатные bridges.

Наличие bridge не означает автоматически наличие нужного отчёта или полного workflow.

## Дополнительные Community-контуры, которые изучены

По потребности могут использоваться:

- Skills / Certifications;
- eLearning;
- Surveys;
- Attendances;
- Time Off;
- Work Entries;
- Remote Work;
- Recruitment;
- Expenses;
- Purchase / Purchase Agreements;
- Analytic Accounting;
- Calendar sync;
- Live Chat / Canned Responses;
- Contacts Merge/Deduplicate;
- Data Recycle.

Они не включаются автоматически: специализированный workflow используется только для своего предмета.

## Сменная работа

Community Resource Calendar умеет обычный и двухнедельный рабочий календарь, но это не произвольный roster engine. Чистый цикл `2/2` не следует автоматически считать закрытым штатным two-week schedule.

Recurring Task поддерживает Days / Weeks / Months / Years, но не часы. Поэтому recurrence `каждые 12 часов` штатно не подтверждена; дневной и ночной handover при необходимости тестируются как две отдельные daily chains.

## API и интеграции

В публичном Community 19 подтверждены:

```text
JSON-2 /json/2/<model>/<method>
Bearer authentication / API keys
динамическая /doc
Import / Export
Email Alias
Website + Project → Website Form → Task
Inbound Automation Webhook
Automation Rules
```

Automation Rules и webhooks — **административный/технический Community-слой**, а не рабочий интерфейс обычного исполнителя.

Для новой системной интеграции JSON-2 предпочтительнее legacy XML-RPC/JSON-RPC.

Прямой SQL write в PostgreSQL не является штатной интеграционной моделью Odoo.

## Безопасность

В Community подтверждены:

- application access rights;
- record rules;
- Project visibility/collaboration;
- LDAP;
- OAuth2;
- TOTP 2FA;
- Passkeys.

Relational Property не обходит права target model. Domain — удобство выбора, а не security boundary.

## Что сознательно не является базой Community-методики

Не строим обязательную архитектуру на:

- Studio;
- Helpdesk;
- Approvals;
- Documents;
- Knowledge;
- Planning;
- Gantt Project;
- Sign;
- Appointments;
- полноценном Quality app;
- Enterprise Data Cleaning/Data Merge;
- полном Inventory Barcode UI;
- Budget Management без отдельной редакционной проверки;
- availability/capacity scheduling Project Tasks;
- собственном Spreadsheet Dashboard без runtime/edition-проверки.

## Реальные возможные gaps после углублённого аудита

Больше **не считаются gaps по умолчанию**:

```text
Task → Vehicle
Task → Employee
Task → Equipment
```

Сначала они реализуются relational Properties.

Реальными кандидатами на доработку остаются только подтверждённые пилотом проблемы, например:

- обязательная история изменения Property-ссылки;
- агрегированный time-in-stage/SLA reporting;
- произвольное capacity/shift planning;
- жёсткая BPM-матрица переходов по ролям;
- Property недостаточен из-за производительности/constraint/API/BI;
- специализированная бизнес-логика предметного процесса.

## Порядок развития

```text
штатные master data models
→ relational Properties
→ bridge modules
→ Project workflow
→ Task Templates / Activities / Dependencies / Recurrence
→ Shared Views
→ Task Analysis
→ My Dashboard
→ API / внешний BI по потребности
→ реальный нагрузочный/пользовательский пилот
→ подтверждённые residual gaps
→ минимальный custom module только на gaps
```

## Разделы

| Раздел | Содержание |
|---|---|
| [00 — Возможности Community](docs/00-odoo19-community.md) | актуальная техническая граница |
| [01 — Модель управления](docs/01-methodology.md) | единица работы, Properties, ответственность, поток |
| [02 — Сценарии](docs/02-scripts.md) | рабочие ситуации |
| [03 — Контроль и аналитика](docs/03-control.md) | Shared Views, Task Analysis, Properties, My Dashboard |
| [04 — Шаблоны](docs/04-templates.md) | Task/Activity/process templates |
| [05 — Процессы](docs/05-processes.md) | описание процессов и источников истины |
| [06 — Настройка](docs/06-workspace.md) | последовательный click-only пилот |
| [07 — Углублённый аудит](docs/07-deep-community-audit.md) | дополнительные Community-модули и bridges |
| [08 — Интеграции Project](docs/08-project-integrations.md) | Project ↔ analytic/purchase/stock/expenses |
| [09 — Project и безопасность](docs/09-project-productivity-security.md) | Project Stages, UX, authentication |
| [10 — API и интеграции](docs/10-external-integrations.md) | JSON-2, webhooks, integration patterns |
| [11 — Relational Properties](docs/11-relational-properties.md) | Many2one/Many2many Properties, tracking и residual gaps |
| [12 — Коммуникации и вложения](docs/12-communication-documents.md) | sharing, Chatter, Stage email/rating, scheduled messages, attachment indexation |
| [13 — Смены и регламентные циклы](docs/13-shift-routines.md) | Resource Calendar, 2/2, handover, recurrence и runtime-границы |
| [14 — Высокопоточный триаж](docs/14-high-volume-triage.md) | Task List, mass edit, operational filters, Activities и нагрузочный пилот |
| [15 — Перепроверка возможностей](docs/15-capability-recheck.md) | повторный аудит спорных функций и исправления границы CE |

## Где хранится что

- **Odoo** — master data, фактическая работа, история, ответственность, сроки и analytics.
- **Репозиторий** — методика, правила процессов и результаты технического аудита.

Не нужно вести параллельный Excel-реестр тех же обязательств.

---

Работа ведётся только в Draft PR. Merge в `main` — только по отдельной явной команде.

## Лицензия

[CC BY 4.0](LICENSE). Материалы можно использовать и адаптировать при указании источника.
