# Настройка Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [03 Аналитика](03-control.md) · **06 Настройка** · [15 Перепроверка](15-capability-recheck.md)

---

Это последовательный **click-only пилот** для одного отдела.

Цель не «включить все функции Odoo», а доказать минимальную рабочую модель:

```text
правильные master data
→ Task как обязательство
→ relational Properties
→ понятный workflow
→ контроль через Views / Analysis / My Dashboard
→ только затем automation / integration / code
```

Если функция не подтверждена для Community — она не является prerequisite пилота.

---

# 1. Установить только базовые приложения

Для первого стенда:

```text
Project
Employees
Contacts
```

После стабилизации рабочих views:

```text
Dashboards / My Dashboard (`board`)
```

Не начинать с десятка приложений одновременно.

---

# 2. Предметные приложения ставить только по реальному процессу

По потребности:

```text
Fleet
Maintenance
Inventory
Calendar
Timesheets
Website
Surveys
eLearning
Purchase
Expenses
Attendances
Time Off
```

Пример:

- если нужна карточка ТС → проверяем Fleet;
- если нужен serial/location → Inventory;
- если нужен ремонт Equipment → Maintenance;
- если нужно поручение по ТС → Project Task со ссылкой на Vehicle.

Не использовать Project Task как замену предметной модели.

---

# 3. Пользователь и Employee — разные сущности

Создать Internal User только тем, кто реально работает в Odoo.

```text
res.users   = пользователь системы / Assignee
hr.employee = сотрудник как HR-сущность
```

Employee может существовать без User.

Не создавать пользователей всем сотрудникам только ради справочника.

---

# 4. Сначала загрузить master data

Перед массовым созданием Tasks определить источники истины:

```text
сотрудники      → Employees
контакты        → Contacts
ТС              → Fleet
оборудование    → Maintenance
serial/location → Inventory
```

Для CSV/XLSX использовать стандартный Import:

```text
List
→ Actions
→ Import records
→ сопоставить fields
→ Test
→ Import
```

Для повторных обновлений сохранять **External ID**.

Не вести тот же master data параллельно текстом в Project.

---

# 5. Проверить auto-install bridge modules

После установки сочетаний приложений проверить уже существующие bridges.

Ключевые:

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

Не проектировать custom relation, пока штатный bridge не проверен.

---

# 6. Создать один основной операционный Project

Например:

```text
Операционная работа
```

Не создавать Projects:

```text
Транспорт
Топливо
Табели
Запросы
Сверки
```

только ради классификации.

Процесс/вид работы — аналитический признак Task, а не автоматически отдельный Project.

Отдельный Project нужен при настоящей границе:

- самостоятельная инициатива;
- свой lifecycle;
- свой manager;
- milestones;
- отдельная visibility;
- отдельный project-level контроль.

---

# 7. Настроить Visibility

Для внутреннего пилота выбрать минимально достаточную Project visibility.

Права проверять минимум под тремя профилями:

```text
обычный исполнитель
руководитель / senior
administrator
```

Не тестировать систему только под Administrator.

Portal/collaborators включать только при конкретном внешнем сценарии.

---

# 8. Настроить Task Stages

Базово:

```text
Входящие
Очередь
В работе
Ожидание внешнего
На проверке   [только если реально нужно]
```

Не создавать Stage `Готово`, если закрытие решается State `Done`.

Не создавать Stage `Waiting` для внутренних зависимостей: dependency сама даёт computed Waiting.

---

# 9. Не путать Stage и State

Обычные States:

```text
In Progress
Changes Requested
Approved
Done
Canceled
```

Dependency-driven:

```text
Waiting
```

Stage отвечает на вопрос:

> где работа в процессе?

State отвечает на системное состояние/закрытие/approval.

---

# 10. Создать минимальные классификационные Properties

Открыть Task основного Project:

```text
Actions
→ Edit Properties
```

Минимально полезный кандидат:

```text
Вид работы
Type: Selection

Работа
Запрос
Улучшение
```

`Процесс` добавлять только когда устойчивый список процессов уже понятен и этот разрез реально используется в фильтрах/аналитике.

Не создавать десятки Properties заранее.

---

# 11. Создать relational Property `ТС`

Только если Fleet реально нужен процессам.

```text
Actions
→ Edit Properties
→ Add property

Label: ТС
Type: Many2one
Model: fleet.vehicle
Domain: при необходимости
Default: пусто
```

Проверить:

1. autocomplete;
2. выбор существующего Vehicle;
3. открытие связанной записи;
4. права доступа;
5. поиск Tasks по Property;
6. `Group By → Properties`;
7. отображение в List/Kanban;
8. скорость на реальном объёме.

---

# 12. Property `Сотрудник`

Если Employee является **объектом работы**, а не исполнителем:

```text
Label: Сотрудник
Type: Many2one
Model: hr.employee
```

Пример:

```text
Assignee = аналитик Иванов
Сотрудник = Петров, чьи данные проверяются
```

Не смешивать Employee-object и Assignee.

---

# 13. Property `Оборудование`

Если Task касается Equipment:

```text
Label: Оборудование
Type: Many2one
Model: maintenance.equipment
```

Если событие само по себе является обычной Maintenance Request, не создавать параллельную Project Task без отдельного контролируемого результата.

---

# 14. Many2many использовать осторожно

Пример:

```text
Затронутые ТС
→ Many2many(fleet.vehicle)
```

Использовать только если один результат действительно относится к нескольким records.

Если по каждому объекту отдельный владелец/срок/результат — лучше отдельные Tasks.

---

# 15. Domain — не security

Domain в relational Property ограничивает варианты выбора в UI.

Он не заменяет:

```text
ACL
record rules
company rules
```

Если пользователь должен только выбирать Vehicle, но не создавать новые Vehicles, это решается правами Fleet.

---

# 16. Проверить governance master data

Определить:

```text
кто создаёт Employees
кто создаёт Vehicles
кто создаёт Equipment
кто может исправлять master data
кто только выбирает записи в Task
```

Не давать всем исполнителям create/write master-data rights только ради удобства Project.

---

# 17. Общая очередь = неназначенные Tasks

Готовая к разбору общая Task:

```text
Stage = Очередь
Assignees = empty
```

Это нормальный штатный shared backlog.

Не создавать fake user:

```text
Отдел
Очередь
Все
```

---

# 18. Одна управленческая ответственность

Odoo технически позволяет несколько Assignees.

Методически для обычной операционной Task должен быть понятен один основной владелец результата.

Несколько Assignees использовать только когда коллективное исполнение действительно осмысленно.

---

# 19. Deadline и Activity — разные даты

```text
Deadline
= когда должен быть готов результат

Activity Due Date
= когда выполнить следующее действие
```

При `Ожидание внешнего` ставить Activity на дату следующего контроля.

Не двигать Deadline каждый раз только потому, что нужно кому-то напомнить.

---

# 20. Priority

UI поддерживает:

```text
Low
Medium
High
Urgent
```

Quick-create:

```text
!   Medium
!!  High
!!! Urgent
```

На старте методики достаточно:

```text
обычная работа → default
критичная      → Urgent
```

Medium/High вводить после появления однозначных критериев.

---

# 21. Activities

Минимальный полезный набор типов:

```text
Проверить
Напомнить
Позвонить
Встреча
```

Activity — follow-up, а не отдельная полноценная Task.

---

# 22. Activity chaining

Перед Automation Rule проверить штатные:

- Suggest Next Activity;
- Trigger Next Activity;
- относительные сроки.

Если цепочка закрывается Activities, не создавать дополнительные Tasks и Automation Rules.

---

# 23. Activity Plans

Использовать для повторяемого набора follow-up на одной записи.

Например:

```text
запросить информацию
→ проверить ответ
→ напомнить при отсутствии
```

Не использовать Activity Plan как замену набору независимых обязательств.

---

# 24. Dependencies

Включить Task Dependencies только если есть реальные блокировки.

```text
Blocked by = predecessor Task
```

Odoo сам вычисляет `Waiting`.

Важно: predecessor может находиться **в другом Project**.

Поэтому отдельная инициатива может блокировать операционную Task без искусственного объединения Projects.

---

# 25. Recurring Tasks

Использовать для одной календарно повторяемой работы.

Пример:

```text
ежемесячная сверка
```

Units public source:

```text
Days
Weeks
Months
Years
```

Hours нет.

Не создавать будущий горизонт из десятков копий вручную.

### Обязательный runtime test

Официальная документация говорит, что Subtasks не копируются, а current public source 19.0 рекурсивно создаёт `child_ids`.

Если recurrence использует subtasks — проверить поведение на конкретной сборке.

---

# 26. Task Templates

Использовать для типовой разовой работы по событию.

Например:

```text
Разобрать расхождение
Проверить корректировку
Подготовить типовой ответ
```

В template хранить устойчивую структуру:

- description;
- tags;
- типовые Properties;
- типовые subtasks при необходимости.

Не зашивать конкретный Vehicle/Employee, если объект каждый раз меняется.

Проверить Task Template + Properties на стенде.

---

# 27. Subtasks

Создавать subtask, если у части есть самостоятельный:

- результат;
- owner;
- deadline;
- dependency;
- контроль.

Не превращать Task в чек-лист технических действий из десятков child tasks.

---

# 28. Rotting

После наблюдения фактической длительности Stages настроить `Days to rot` там, где этот сигнал реально полезен.

Не ставить один threshold на все Stages.

```text
Deadline = срок результата
Activity = срок follow-up
Rotting = Task слишком долго не меняет Stage
```

---

# 29. List — диспетчерское рабочее место

Для большого входящего потока открыть **List**.

Standard Project Task List поддерживает mass edit.

Использовать List для:

- сортировки;
- сравнения полей;
- массового triage;
- assignment;
- deadline/priority review;
- Activities;
- Properties.

Kanban использовать прежде всего для визуального flow по Stages.

---

# 30. Создать Shared Views

Минимум:

```text
Входящие
Очередь
Неназначенная очередь
В работе
Просрочено
Ожидание внешнего
Waiting / Blocking
Urgent
Rotting
Просроченные Activities
```

При необходимости:

```text
Открытые по ТС
Открытые по Equipment
```

Не сохранять десятки почти одинаковых views.

---

# 31. Project Top Bar

Официальный Project workflow:

```text
открыть view
→ настроить filter / group / search
→ sliders
→ Save View
→ Shared
```

Использовать для действительно постоянных общих рабочих срезов.

Это сильнее, чем плодить отдельные Projects для разных представлений одного backlog.

---

# 32. Task Analysis

Сначала использовать штатные показатели:

- created;
- closed;
- backlog;
- Stage;
- State;
- Assignee;
- Priority;
- working time to assign;
- working time to close;
- task count.

Не создавать KPI только потому, что поле можно вывести в Pivot.

---

# 33. Time in Stage

Открыть конкретную Task и проверить duration statusbar.

Использовать для расследования зависшего случая.

Не обещать готовый aggregated SLA by Stage report: это отдельная аналитическая потребность.

---

# 34. My Dashboard — основной click-only dashboard

После стабилизации Views установить/проверить модуль Dashboards (`board`).

Официальный workflow:

```text
открыть нужный List / Kanban / Pivot / Graph
→ Actions
→ Dashboard
→ Add to my Dashboard
```

Рекомендуемый состав:

```text
Просрочено
Неназначенная очередь
Ожидание внешнего
Urgent
Pivot Stage × Assignee
Graph закрытия по периодам
```

My Dashboard основан на Odoo views и не требует строить отдельную spreadsheet-модель показателей.

---

# 35. Spreadsheet Dashboard — не включать в baseline

Public Community содержит `spreadsheet` и `spreadsheet_dashboard`, но наличие этих модулей не подтверждает полный click-only authoring workflow собственного Spreadsheet Dashboard в чистой CE.

Официальное создание dashboard с нуля требует:

```text
Dashboards / Admin
+
Documents / User
```

Documents не входит в Community baseline этой методики.

Поэтому **не настраивать Spreadsheet Dashboard как обязательный этап пилота**.

Если позже появится конкретная потребность:

1. проверить чистую Community-инсталляцию;
2. проверить доступный create/edit workflow;
3. проверить, какие модули реально требуются;
4. только после этого решать между Spreadsheet Dashboard и внешним BI.

---

# 36. Отдельная инициатива

Если работа имеет самостоятельный lifecycle, создать отдельный Project.

Использовать по необходимости:

- Project Manager;
- Project Start/End;
- Project Stages;
- Milestones;
- Project Updates;
- Project Dashboard/Burndown;
- Project-level Activities.

Не использовать этот слой для ежедневного операционного backlog без причины.

---

# 37. Project Stages

Включать для портфеля самостоятельных Projects.

Пример:

```text
Подготовка
Запланирован
В реализации
На паузе
Завершён
```

Не смешивать:

```text
Project Stage
Project Update Status
Task Stage
Task State
```

---

# 38. Project Templates + Roles

Использовать для повторяемой структуры целого Project.

Roles позволяют назначить людей на роль при создании Project из template и раздать соответствующие Tasks.

### Не обещать Planning

Общая документация описывает availability/workload scheduling, но этот planned-date layer не подтверждён public Community Project model.

В CE baseline Project Template = **структура + роли**, а не полноценный capacity planner.

---

# 39. Website Form → Task

Только если нужен реальный входящий web-канал.

Установленная связка:

```text
Website + Project
→ website_project
```

может создавать Project Tasks из website form.

Все такие Tasks направлять во `Входящие`, а не сразу `В работе`.

До внешней публикации проверить:

- spam/duplicate handling;
- обязательные поля;
- attachment policy;
- security;
- triage ownership.

---

# 40. Email Alias

Включать после того, как ручной triage уже работает.

Новые email-created Tasks должны попадать во `Входящие`.

Проверить:

- sender → Customer/Contact;
- subject → Task name;
- body/attachments;
- replies;
- duplicate threads;
- mail routing;
- внешние адреса.

Не запускать email ingestion как первый шаг пилота.

---

# 41. Automation Rules

Community source подтверждает `base_automation`.

Настройка относится к административному/technical layer.

После стабилизации процесса:

```text
Settings
→ Developer Mode
→ Technical
→ Automation
→ Automation Rules
```

Использовать только для механических правил, например:

- реакция на конкретный field change;
- time-based action;
- webhook integration.

Не пытаться построить в Automation Rules полноценный BPM со скрытыми переходами.

---

# 42. JSON-2 / API

Для внешней интеграции сначала проверить public Odoo 19 JSON-2:

```text
POST /json/2/<model>/<method>
Authorization: Bearer ...
```

Auto-install `api_doc` предоставляет:

```text
/doc
```

с фактическими models/fields/methods базы.

Перед интеграцией:

1. проверить access rights отдельного integration user;
2. открыть `/doc`;
3. проверить read/search/write нужной модели;
4. проверить формат Properties;
5. не писать напрямую в PostgreSQL.

---

# 43. Fleet + Employees

Если Fleet используется:

1. загрузить Vehicles;
2. связать drivers/Employees по штатной модели;
3. проверить `hr_fleet` / Fleet History;
4. проверить Task Property `ТС`;
5. проверить права Project Users на Fleet.

Только после этого оценивать необходимость custom vehicle registry.

---

# 44. Inventory + Maintenance

Для serialised equipment:

1. проверить serial/lot в Inventory;
2. Equipment в Maintenance;
3. `stock_maintenance`;
4. current location;
5. employee allocation через `hr_maintenance`, если нужен;
6. Task Property `Оборудование` при отдельном обязательстве.

Не создавать собственную asset-model раньше этой проверки.

---

# 45. Contacts quality

При большом импорте:

- использовать External IDs;
- проверить Merge/Deduplicate;
- определить владельца справочника;
- не включать автоматический Data Recycle без retention policy.

Не обещать весь Enterprise Data Cleaning toolkit.

---

# 46. Сменный график 2/2

Resource Calendar умеет weekly и two-week schedule.

Это **не** arbitrary four-day rotating roster.

Не пытаться насильно моделировать `2/2` как универсальный repeating two-week calendar.

Для handover задачи можно испытать две независимые daily recurrence chains:

```text
дневная передача
ночная передача
```

но recurrence не является заменой планированию состава смен.

---

# 47. Что не делать в пилоте

Не делать:

```text
отдельный Project на каждый процесс
fake assignee «Отдел»
Stage «Waiting» вместо Dependencies
Stage «Готово» вместо Done без причины
десятки Properties заранее
копию Fleet/Employees в Selection
обязательные Timesheets без управленческого вопроса
employee leaderboard по количеству Tasks
Automation до стабилизации workflow
Spreadsheet Dashboard как обязательный CE-шаг
прямой SQL write в Odoo
custom module до измеренного gap
```

---

# 48. Обязательный runtime checklist

До решения «штатный Odoo не хватает» прогнать:

### Master data

- Employees import;
- Fleet import на реальном объёме;
- Equipment/serial при необходимости;
- External IDs;
- права обычного пользователя.

### Project

- 100+ реалистичных Tasks;
- List mass edit;
- Kanban;
- Shared Views;
- Activities;
- Dependencies;
- Rotting;
- Task Templates;
- Recurring Tasks.

### Properties

- Many2one Vehicle;
- Many2one Employee;
- Many2one Equipment;
- filter;
- Group By;
- performance;
- Import;
- JSON-2;
- history/audit.

### Analytics

- Task Analysis;
- Pivot;
- Graph;
- My Dashboard;
- drill-down;
- права на данные.

### Routines

- recurrence + Properties;
- recurrence + Subtasks;
- daily handover chains;
- Activity Plans.

### Integration

- email alias;
- Website Form, если нужен;
- `/doc`;
- JSON-2;
- Automation/webhook только после стабилизации.

---

# 49. Решение после пилота

Для каждого неудобства задать вопросы в порядке:

```text
1. Неправильно выбрана сущность?
2. Есть штатное поле/Property/View?
3. Есть отдельное Community-приложение?
4. Есть bridge module?
5. Есть Task Template/Activity/Recurrence?
6. Решает Shared View / My Dashboard?
7. Решает Import/API/Automation?
8. Проблема доказана на реальном объёме?
```

Только после ответа `нет` на предыдущие уровни появляется нормальное техническое задание на минимальный custom module.

---

[← 05 — Процессы](05-processes.md) · [15 — Перепроверка возможностей →](15-capability-recheck.md)
