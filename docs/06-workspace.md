# Настройка Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [03 Аналитика](03-control.md) · **06 Настройка** · [17 Люди](17-master-data-people.md)

---

Это последовательный **click-only пилот**. Его задача — проверить, можно ли закрыть реальные процессы штатными средствами Odoo 19 Community без критических компромиссов.

Пилот не должен заранее навязывать конкретные роли, Stages, смены, правила назначения или схему приёмки результата.

## 1. Установить минимальное ядро

Начать с тех приложений, без которых нельзя проверить саму модель работы.

Базовый минимум:

```text
Project
Contacts
```

Для текущего контура сотрудники компании ведутся в Contacts / `res.partner`. Employees / HR не устанавливается только ради справочника людей. Он возвращается на рассмотрение только при реальной потребности в HR-функциональности.

Остальные приложения устанавливаются только по реальной предметной потребности.

## 2. Подключать предметные приложения по смыслу

Примеры:

```text
Fleet       → ТС
Maintenance → Equipment / maintenance
Inventory   → serial / lot / location / movement
Calendar    → встречи
Purchase    → закупки
Expenses    → расходы
Timesheets  → фактические трудозатраты, если нужны
```

Не устанавливать приложение только потому, что оно существует.

## 3. Определить источники истины

До настройки Tasks составить карту предметных сущностей:

```text
сущность
→ штатная model / Property / отдельная model
→ источник загрузки
→ кто поддерживает данные
```

Для людей уже зафиксировано:

```text
человек / сотрудник компании → res.partner
доступ в Odoo                → res.users, связанный с тем же res.partner
```

Подробно: [17 — Master data: люди](17-master-data-people.md).

Если штатная model подходит, использовать её.

Если объект нужен только как атрибут работы, достаточно Property/Tag.

Если объекту нужен самостоятельный реестр, lifecycle, уникальность, права и связи — это кандидат на отдельную model.

## 4. Загрузить master data

Для CSV/XLSX использовать стандартный Import.

При повторных обновлениях сохранять External ID.

Перед массовой загрузкой проверить:

- уникальность;
- активность records;
- права;
- пригодность модели для реальных данных.

Для справочника людей один человек должен соответствовать одной записи `res.partner`; выдача доступа через `res.users` не должна создавать вторую карточку человека.

Не дублировать один и тот же master data текстом в Tasks.

## 5. Проверить bridge modules

После установки связанных приложений проверить auto-install bridges до проектирования собственных relations.

Примеры доступных технических bridges:

```text
Employees + Fleet        → hr_fleet
Employees + Maintenance  → hr_maintenance
Inventory + Maintenance  → stock_maintenance
Project + Purchase       → project_purchase
Project + Inventory      → project_stock
Project + Accounting     → project_account
Project + Expenses       → project_hr_expense
```

HR-bridges учитываются только если Employees / HR позже будет осознанно включён. В текущий baseline они не входят.

## 6. Определить границы Projects

Не использовать универсальное правило `один процесс = один Project` или `один отдел = один Project`.

Для каждого Project зафиксировать причину его существования:

- visibility;
- отдельный lifecycle;
- отдельные milestones / project-level reporting;
- существенно другой workflow;
- самостоятельная инициатива.

Если различие нужно только для поиска и аналитики, использовать Properties/Tags/Views.

## 7. Настроить Stages по фактическому workflow

Создать только те Stages, которые реально нужны процессу.

Для каждого Stage определить:

```text
что означает вход
что означает нахождение в Stage
что означает выход
```

Не создавать Stage только ради копирования старой методики.

Не дублировать штатные `Done`, `Canceled` или dependency-driven `Waiting` без отдельного смысла.

## 8. Определить использование Assignees

Процесс должен ответить:

- нужен ли Assignee сразу;
- допустима ли неназначенная Task;
- используется один или несколько Assignees;
- кто может назначать/переназначать.

После этого проверить фактический UX Odoo.

Assignees остаются штатными `res.users`. Не создавать fake users для очередей, организационных групп или людей, которым вход в Odoo не нужен.

## 9. Настроить Properties только по потребности

Добавлять Property, если оно реально нужно для:

- выполнения;
- фильтрации;
- группировки;
- шаблонов;
- аналитики.

Примеры классификационных Properties:

```text
Вид работы
Процесс
Источник
```

Это только примеры, а не обязательный набор.

## 10. Проверить relational Properties

Если Task должна ссылаться на предметный record:

```text
Property
→ Many2one / Many2many
→ Model
→ Domain при необходимости
```

Проверить на реальных моделях:

- autocomplete;
- выбор record;
- открытие связанной записи;
- права target model;
- Search / Group By;
- List/Kanban;
- производительность.

Примеры текущего контура:

```text
ТС           → fleet.vehicle
Сотрудник    → res.partner
Оборудование → maintenance.equipment
```

## 11. Domain не заменяет security

Domain ограничивает варианты выбора в UI.

Права задаются отдельно:

```text
Groups
Access Rights
Record Rules
Project visibility
```

## 12. Настроить права после определения реальных полномочий

Не начинать с искусственных ролей `исполнитель / старший / руководитель`, если они не определены процессом.

Сначала описать действия:

```text
может читать
может создавать
может изменять
может удалять
может видеть все records или только часть
может менять master data
```

Затем настроить Groups / ACL / Record Rules.

Проверять права минимум под обычным рабочим пользователем и администратором. Дополнительные профили создаются только если они реально нужны.

## 13. Deadline

Использовать Deadline только там, где есть срок результата.

Проверить:

- отображение в List/Kanban/Calendar;
- overdue filters;
- правила изменения срока конкретного процесса.

Activity Due Date использовать отдельно для следующего действия.

## 14. Priority

Odoo поддерживает Low / Medium / High / Urgent.

Не задавать локальную шкалу заранее.

Если Priority нужен, определить:

```text
что означает каждый используемый уровень
какое действие он меняет
```

## 15. Activities

Использовать Activities для follow-up и следующего действия.

Настраивать Activity Types/Plans только когда конкретный процесс этого требует.

Не превращать каждое Activity в отдельную Task.

## 16. Dependencies

Включить Dependencies, если есть реальные внутренние блокировки.

```text
Blocked by
→ Waiting
```

Проверить cross-project dependency, если процессы распределены по нескольким Projects.

## 17. Subtasks

Использовать Subtasks только для частей с самостоятельным управлением.

Не строить дерево из каждого технического шага.

## 18. Task Templates

Использовать для типовой разовой работы по событию.

Проверить:

- создание Task из template;
- Description;
- Properties;
- Subtasks;
- права пользователей.

## 19. Recurring Tasks

Использовать для календарно повторяемой работы.

Public source поддерживает Days / Weeks / Months / Years.

Если процесс зависит от копирования Properties/Subtasks, выполнить runtime test на установленной версии.

Не моделировать через recurrence организационные графики, если сам процесс этого не требует.

## 20. Project Templates + Roles

Использовать только при повторяемой структуре целого Project.

Не считать Project Template полноценным capacity/Planning engine.

## 21. Входящие каналы

После того как ручная модель работает, по потребности проверить:

- Email Alias;
- Website Form → Task;
- API;
- webhook / Automation.

Автоматический intake не обязан сразу определять Stage, Assignee, Priority и Deadline, если правило неоднозначно.

## 22. List и Kanban

Проверить оба режима на реальной работе.

### List

Проверить:

- сортировку;
- mass edit;
- Properties;
- Deadline / Priority;
- Activities;
- фильтры;
- Group By.

### Kanban

Проверить:

- читаемость workflow;
- работу со Stages;
- карточки и Properties;
- фактическую удобность для исполнителей.

Не назначать один вид основным до пользовательского теста.

## 23. Shared Views

Создавать только для устойчивых вопросов.

Примеры:

```text
просроченные
неназначенные
Waiting / Blocking
Urgent
Late Activities
Tasks по предметному Property
```

Это примеры, а не обязательный список.

## 24. Task Analysis / Pivot / Graph

Сначала проверить, отвечает ли штатная аналитика на реальные вопросы процесса.

Не превращать все доступные measures в KPI.

## 25. My Dashboard

После стабилизации Views и аналитических вопросов проверить `board` / My Dashboard.

Добавлять только те элементы, которые уже доказали полезность в рабочих Views/Pivot/Graph.

Spreadsheet Dashboard не является обязательным Community baseline до отдельной runtime/edition-проверки.

## 26. Automation Rules

Использовать после стабилизации модели.

Подходящие случаи:

- однозначная реакция на field change;
- time-based механическое действие;
- webhook integration.

Не строить сложный BPM из Automation Rules, если процесс требует большой матрицы переходов и исключений.

## 27. JSON-2 / API

Для интеграций проверить:

```text
POST /json/2/<model>/<method>
Authorization: Bearer ...
/doc
```

Использовать отдельного integration user с минимально необходимыми правами.

Перед custom API-module проверить штатный JSON-2 и фактические models/fields/methods базы.

## 28. Runtime checklist

До решения «штатного Odoo недостаточно» прогнать реальный сценарий.

### Предметные данные

- import реальных справочников;
- External IDs;
- target-model permissions;
- качество поиска;
- объём данных;
- один человек = один `res.partner`;
- выдача `res.users` без дублирования карточки человека.

### Tasks

- реалистичный набор Tasks;
- Stages конкретного процесса;
- Assignees;
- Deadline;
- Activities;
- Dependencies;
- Subtasks;
- Templates;
- Recurrence.

### Properties

- `Сотрудник → res.partner`;
- Many2one/Many2many;
- Filter / Group By;
- List/Kanban;
- performance;
- Import;
- JSON-2;
- история изменений, если она критична.

### Security

- обычные пользователи;
- дополнительные группы, если они реально нужны;
- read/write/create/delete;
- record visibility;
- master-data access.

### Analytics

- Shared Views;
- Task Analysis;
- Pivot / Graph;
- My Dashboard;
- drill-down;
- права на данные.

### Intake / integration

Только если нужны процессу:

- Email Alias;
- Website Form;
- API;
- Automation / Webhook.

## 29. Как принимать решение о доработке

Для каждого ограничения пройти последовательность:

```text
1. Верно ли выбрана сущность?
2. Есть штатная model?
3. Есть Property / relation?
4. Есть bridge module?
5. Решается Stage / State / Activity / Dependency / Template / Recurrence?
6. Решается View / Analysis / Dashboard?
7. Решается Groups / ACL / Record Rules?
8. Решается Import / API / Automation без скрытого BPM?
9. Подтверждена ли проблема на реальном пилоте?
```

Если ответ остаётся `нет`, формулируется конкретный gap и минимальное техническое решение.

Не считать custom module поражением. Плохая доработка — это дублирование уже работающего Odoo. Маленькая доработка на доказанный gap — нормальная архитектура.

---

[← 05 — Процессы](05-processes.md) · [17 — Master data: люди](17-master-data-people.md) · [Главная](../README.md)
