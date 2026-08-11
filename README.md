# Управление работой отдела в Odoo 19 Community

Практическая методика управления операционной работой, справочниками, контролем и аналитикой в **стандартном Odoo 19 Community**.

Методика строится вокруг реальных штатных возможностей: предметные данные живут в своих Odoo-моделях, а Project управляет **обязательствами и контролируемыми результатами**.

**[Граница возможностей Community →](docs/00-odoo19-community.md)**  
**[Модель управления →](docs/01-methodology.md)**  
**[Click-only настройка →](docs/06-workspace.md)**

## Как проверяются возможности

Для каждого существенного решения:

1. поведение сверяется с официальной документацией **Odoo 19.0**;
2. принадлежность Community при необходимости проверяется в публичной ветке **`odoo/odoo:19.0`**;
3. перед объявлением разрыва проверяются штатные bridge modules;
4. до custom module проверяются relational Properties, standard views/reports, Import/API/Automation и реальный пилот.

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

## Базовый операционный Project

Для постоянной работы одного отдела обычно достаточно одного Project:

```text
Входящие
→ Очередь
→ В работе
→ Ожидание внешнего
→ На проверке   [только при реальной необходимости]
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

## Рабочее место без разработки

Подтверждены и включены в методику:

- Shared Favorites;
- Project Top Bar `Save View → Shared`;
- List / Kanban / Calendar / Activity / Pivot / Graph;
- Task Analysis;
- My Dashboard;
- Rotting;
- Activities и Activity Plans;
- Dependencies;
- Recurring Tasks;
- **Task Templates**;
- Project Templates и Project Roles;
- Project Stages для портфеля инициатив;
- Milestones / Project Updates / Burndown для инициатив.

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

## API и интеграции

В публичном Community 19 подтверждены:

```text
JSON-2 /json/2/<model>/<method>
API keys
динамическая /doc
Import / Export
Email Alias
Website Form
Inbound Automation Webhook
Outbound Webhook Action
Automation Rules
```

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
- Budget Management без отдельной редакционной проверки.

## Реальные возможные gaps после углублённого аудита

Больше **не считаются gaps по умолчанию**:

```text
Task → Vehicle
Task → Employee
Task → Equipment
```

Сначала они реализуются relational Properties.

Реальными кандидатами на доработку остаются только подтверждённые пилотом проблемы, например:

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
→ API/Automation по потребности
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
| [03 — Контроль и аналитика](docs/03-control.md) | Shared Views, Task Analysis, Properties, Dashboards |
| [04 — Шаблоны](docs/04-templates.md) | Task/Activity/process templates |
| [05 — Процессы](docs/05-processes.md) | описание процессов и источников истины |
| [06 — Настройка](docs/06-workspace.md) | последовательный click-only пилот |
| [07 — Углублённый аудит](docs/07-deep-community-audit.md) | дополнительные Community-модули и bridges |
| [08 — Интеграции Project](docs/08-project-integrations.md) | Project ↔ analytic/purchase/stock/expenses |
| [09 — Project и безопасность](docs/09-project-productivity-security.md) | Project Stages, UX, authentication |
| [10 — API и интеграции](docs/10-external-integrations.md) | JSON-2, webhooks, integration patterns |
| [11 — Relational Properties](docs/11-relational-properties.md) | техническая проверка Many2one/Many2many Properties |

## Где хранится что

- **Odoo** — master data, фактическая работа, история, ответственность, сроки и analytics.
- **Репозиторий** — методика, правила процессов и результаты технического аудита.

Не нужно вести параллельный Excel-реестр тех же обязательств.

---

Работа ведётся только в Draft PR. Merge в `main` — только по отдельной явной команде.

## Лицензия

[CC BY 4.0](LICENSE). Материалы можно использовать и адаптировать при указании источника.
