# Перепроверка возможностей Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [06 Настройка](06-workspace.md) · **15 Перепроверка**

---

Этот документ фиксирует повторный аудит спорных возможностей после первого широкого прохода по Odoo 19.

Цель — не найти как можно больше функций, а убрать из методики всё, что было:

- перенесено из общей документации без проверки редакции;
- выведено только из наличия технического модуля;
- названо click-only возможностью без подтверждённого UI;
- перепутано с Enterprise/Documents/Planning/Studio;
- подтверждено документацией, но противоречит public source `odoo/odoo:19.0`.

## 1. Шкала доверия

В методике теперь используются четыре класса.

### A — подтверждено для Community и UI

Есть public Community source и подтверждённый пользовательский путь.

Можно включать в click-only baseline.

### B — подтверждено в Community source, но административное/техническое

Функция есть, но её настройка относится к Administrator / Developer Mode / integration layer.

Не выдавать её за обычный рабочий интерфейс исполнителя.

### C — infrastructure/source есть, но полный click-only workflow не подтверждён

Не строить методику на функции до runtime-проверки чистой Community-базы.

### D — не подтверждено для Community

Не использовать как baseline.

---

## 2. Relational Properties — подтверждено

Статус: **A**.

Odoo 19 Properties поддерживают:

- Many2one;
- Many2many;
- выбор Model;
- Domain;
- Default Value.

Поэтому стандартная Task может ссылаться на master data кликами:

```text
ТС           → Many2one(fleet.vehicle)
Сотрудник    → Many2one(hr.employee)
Оборудование → Many2one(maintenance.equipment)
Контрагент   → Many2one(res.partner)
```

В Project Task public source есть `task_properties`, а стандартный Task search поддерживает поиск и `Group By → Properties`.

### Что перепроверка не изменила

Relational Property остаётся первым решением перед custom Many2one.

### Реальная граница

`project.task.task_properties` не объявлен как обычное `tracking=True` поле.

Поэтому доказуемую историю изменения предметной ссылки нельзя считать гарантированной только из факта использования Property.

Если такая история критична — это отдельный runtime-тест и возможный residual gap.

---

## 3. Task Templates — подтверждено

Статус: **A**.

Несмотря на отсутствие отдельной основной страницы user documentation, public Community 19 содержит полноценный механизм Task Templates.

Подтверждены:

- `project.task.is_template`;
- получение template tasks конкретного Project;
- dropdown template tasks в стандартном UI;
- `action_create_from_template`;
- создание обычной Task из template;
- копирование child tasks;
- template tasks не входят в обычный open task count;
- копирование task templates при копировании Project.

Следовательно, модель повторяемости остаётся:

```text
разовая типовая работа → Task Template
календарная периодика → Recurring Task
цепочка действий → Activity Plan / chaining
типовая инициатива → Project Template
```

---

## 4. Recurring Tasks — функция подтверждена, subtasks спорны

Статус базовой recurrence: **A**.

Подтверждены:

- повтор Days / Weeks / Months / Years;
- следующий occurrence создаётся после закрытия предыдущего;
- Deadline переносится;
- recurrence не имеет штатной единицы Hours.

### Расхождение documentation/source

Официальная документация Odoo 19 говорит, что `Subtasks` не копируются.

Текущий public source `project_task_recurrence.py` при создании следующего occurrence рекурсивно формирует `child_ids`.

Методическое решение:

> Поведение subtasks в Recurring Task не объявляется гарантированным до runtime-теста конкретной сборки Odoo 19 Community.

Не проектировать критический процесс на этом расхождении.

---

## 5. My Dashboard — подтверждено

Статус: **A**.

Public Community содержит модуль `board` с назначением `Build your own dashboards`.

Он зависит от `spreadsheet_dashboard`, но сам **My Dashboard не является Spreadsheet dashboard**.

Официальный пользовательский workflow:

```text
открыть нужный List / Kanban / Pivot / Graph
→ Actions
→ Dashboard
→ Add to my Dashboard
```

My Dashboard подходит для оперативного руководительского экрана:

- overdue;
- unassigned;
- external waiting;
- Urgent;
- Pivot / Graph;
- другие сохранённые рабочие views.

Для нашей методики это основной click-only dashboard layer.

---

## 6. Spreadsheet Dashboard — прежний вывод был слишком сильным

Статус: **C**, не baseline.

### Что точно есть в public Community

Есть public LGPL-модули:

```text
spreadsheet
spreadsheet_dashboard
```

Есть:

- модель `spreadsheet.dashboard`;
- spreadsheet storage/mixin;
- dashboard renderer;
- groups/access fields;
- favorite dashboards;
- read/display infrastructure;
- list/pivot/chart spreadsheet plugins.

### Что НЕ подтверждено как чистый click-only CE workflow

Официальная документация для построения dashboard с нуля требует:

```text
Dashboards / Admin
+
Documents / User
```

а Spreadsheet documentation прямо связывает создание spreadsheets с Documents.

При этом public `spreadsheet.dashboard` configuration view имеет `create="false"`.

Public dashboard JS вызывает:

```text
spreadsheet.dashboard.action_edit_dashboard
```

но реализация этого server method в public `spreadsheet_dashboard` model layer не подтверждена.

Это сильный признак того, что полноценный create/edit workflow дополняется другим редакционным слоем.

### Новое методическое решение

Не писать:

> после настройки KPI создаём свой Spreadsheet Dashboard в Community

как гарантированную инструкцию.

Правильно:

```text
Shared Views
→ Task Analysis
→ My Dashboard
→ внешний BI / API при необходимости
```

Spreadsheet Dashboard допускается только после runtime-проверки конкретной чистой Community-инсталляции и доступного набора модулей.

Наличие Python/JS infrastructure само по себе больше не считается доказательством полного click-only workflow.

---

## 7. Project Top Bar `Save View → Shared` — подтверждено

Статус: **A**.

Официальная Project documentation 19.0 прямо описывает custom top bar buttons:

```text
открыть существующий top bar view
→ настроить search/filter/grouping
→ sliders
→ Save View
→ Shared
```

Следовательно, Shared Views остаются фундаментом рабочего пространства.

Это предпочтительнее создания большого количества Projects только ради разных представлений одних Tasks.

---

## 8. Website Form → Task — подтверждено, но это связка Website + Project

Статус: **A**, по потребности.

Public module:

```text
website_project
```

имеет назначение `Online Task Submission` и создаёт Project Tasks из формы сайта.

Он зависит от:

```text
website
project
```

Следовательно, Website Form → Task — реальная Community-возможность, но не часть голого Project.

Для внутреннего пилота без внешнего входящего канала Website устанавливать не нужно.

---

## 9. Automation Rules — подтверждено Community source

Статус: **B**.

Public module:

```text
base_automation
```

LGPL и содержит стандартный UI Automation Rules.

Подтверждены triggers, включая:

- create;
- create/write;
- UI change;
- time;
- state;
- priority;
- stage;
- tag;
- webhook.

Это реальная Community-возможность, несмотря на то что общая пользовательская документация часто размещает Automation/Webhook в разделе Studio.

### Методическое ограничение

Automation Rules не являются базовым способом описания процесса.

Порядок:

```text
правильная модель
→ Stage/State/Activity/Template/Recurrence
→ стабильный ручной процесс
→ Automation Rule только на повторяющийся механический шаг
```

Настройка Automation относится к административному/техническому уровню.

---

## 10. JSON-2 и динамическая API documentation — подтверждено

Статус: **B**.

Public Community 19 содержит:

```text
POST /json/2/<model>/<method>
auth='bearer'
```

и auto-install hidden module:

```text
api_doc
```

который предоставляет динамическую `/doc` по фактической базе:

- models;
- fields;
- methods;
- playground;
- примеры HTTP-вызовов.

Для новых интеграций это предпочтительный технический путь перед legacy RPC и прямым SQL write.

---

## 11. Project Templates / Roles — подтверждены, availability scheduling не включаем

Статус template/roles: **A**.

Подтверждены:

- Project Templates;
- Project Roles;
- перенос структуры проекта;
- назначение assignees по roles.

### Но

Общая документация также описывает автоматическое планирование task dates с учётом:

- allocated time;
- dependencies;
- assignee availability;
- working schedules;
- time off;
- public holidays;
- workload.

Соответствующий planned-date task model layer не подтверждён в public Community `project.task`.

Поэтому availability/capacity scheduling **не входит** в CE baseline.

Project Templates используем для структуры и ролей, а не как скрытую замену Planning.

---

## 12. Working Schedules — подтверждены, arbitrary 2/2 roster нет

Статус Working Schedule: **A**.

Resource Calendar поддерживает:

- обычный недельный calendar;
- two-week mode;
- First / Second week.

Но это не универсальный cyclic roster engine.

Чистый график `2/2` имеет четырёхдневный цикл и не равен повторяющемуся двухнедельному шаблону.

Следовательно:

- Resource Calendar можно использовать как HR working schedule;
- нельзя объявлять его штатным полноценным диспетчером смен `2/2`.

---

## 13. Dependencies между Projects — подтверждены source

Статус: **A**.

`Blocked by` в public `project.task` не ограничивает predecessor тем же Project.

Значит Task из операционного Project может зависеть от Task самостоятельной инициативы и наоборот.

Это важно для архитектуры:

> отдельный Project не уничтожает межпроектные зависимости.

Но создавать отдельный Project только ради классификации процесса всё равно не нужно.

---

## 14. Priority 0–3 — подтверждено

Статус: **A**.

Public model и quick-create parser подтверждают:

```text
0 → Low
1 → Medium
2 → High
3 → Urgent

!   → Medium
!!  → High
!!! → Urgent
```

Методика не обязана использовать все уровни. Четыре технических значения не означают четыре обязательных управленческих класса.

---

## 15. Project Dashboard / Updates / Burndown — подтверждены public Project

Статус: **A**, прежде всего для инициатив.

Public Project содержит:

- Project Updates;
- Project Dashboard action;
- Burndown action;
- Milestones;
- Project-level Activities;
- Project Stages.

Это отдельный управленческий слой для самостоятельных проектов/инициатив.

Не использовать его как ежедневный экран общего операционного backlog, если My Dashboard/List решают вопрос проще.

---

## 16. Gantt Project — не baseline Community

Статус: **D**.

Стандартный Community action Tasks имеет view modes:

```text
kanban
list
form
calendar
pivot
graph
activity
```

Gantt в этом action не подтверждён.

Общая документация может показывать Gantt в связанных сценариях, но методика CE от него не зависит.

---

## 17. Attachment indexation — infrastructure, не Documents UX

Статус: **C** для пользовательского document search.

Public Community умеет технически индексировать текст части вложений.

Но стандартный Attachment search не подтверждён как полноценный удобный full-text Documents UX по `index_content`.

Следовательно:

- Attachments/Chatter — подтверждены;
- техническая indexation — подтверждена;
- полноценный Documents-like поиск/управление документами — не заявляется.

---

## 18. Bridge modules — правило остаётся

Повторная проверка не отменила принцип:

> нельзя объявлять две штатные модели несвязанными до проверки bridge modules.

Уже подтверждены, среди прочего:

```text
hr_fleet
hr_maintenance
stock_maintenance
hr_skills_survey
hr_skills_slides
hr_calendar
project_account
project_purchase
project_stock
project_stock_account
project_hr_expense
```

Но наличие bridge не означает автоматически наличие нужного отчёта или полного workflow.

---

## 19. Что теперь считаем надёжным click-only ядром

```text
Project / Tasks
Task Stages + States
Assignees
Deadline
Priority
Tags
Relational Properties
Chatter / Followers / Attachments
Activities / Activity Plans / chaining
Subtasks
Dependencies
Recurring Tasks
Task Templates
Project Templates / Roles
Milestones
Project Updates / Burndown
Project Stages
Rotting
List / Kanban / Calendar / Activity / Pivot / Graph
Task Analysis
Shared Views / Project Top Bar
My Dashboard
Email Alias
Website Form → Task [если установлен Website]
```

Административный технический слой:

```text
Automation Rules
Webhooks
Import / Export
JSON-2
API Documentation /doc
```

---

## 20. Что не включаем как гарантированный click-only CE baseline

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
полный Quality app
Enterprise Data Cleaning/Data Merge
полный Inventory Barcode UI
произвольный rotating roster / capacity planner
availability-based Project Task scheduling
свой Spreadsheet Dashboard без runtime/edition проверки
```

---

## 21. Оставшиеся runtime-тесты

После повторного source/doc-аудита наиболее важны не новые поиски по каталогу, а проверка чистой Community-базы:

1. relational Property на реальном объёме Fleet/Employees/Equipment;
2. filter/group speed по Properties;
3. bulk import Tasks с dynamic Properties;
4. Properties через JSON-2;
5. история изменения Properties в Chatter;
6. ACL target model под обычным Project User;
7. пригодность Fleet для реального состава специальной техники;
8. Recurring Task + Properties;
9. Recurring Task + Subtasks;
10. Task Template + Properties;
11. two daily recurrence chains для передачи дневной/ночной смены;
12. email alias на реальной почтовой инфраструктуре;
13. Website Form → Inbox triage;
14. My Dashboard с реальными shared views;
15. наличие/отсутствие полного Spreadsheet Dashboard create/edit workflow в чистой CE;
16. attachment indexed-search UX;
17. русский интерфейс и фактические названия пунктов меню.

---

## 22. Итог повторной проверки

Главная поправка этого прохода:

> **Spreadsheet infrastructure в public Community нельзя автоматически приравнивать к гарантированному click-only конструктору Spreadsheet Dashboards.**

Остальные ключевые изменения предыдущего аудита — relational Properties, Task Templates, Shared Project Views, My Dashboard, Website Task Submission, Automation Rules и JSON-2 — повторную проверку выдержали.

Дальнейшая методика должна опираться сначала на **Project List/Kanban + Shared Views + Task Analysis + My Dashboard**, а Spreadsheet/BI выбирать только после конкретного аналитического требования.

---

[← 14 — Высокопоточный триаж](14-high-volume-triage.md) · [Главная](../README.md)
