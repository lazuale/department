# Настройка Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [02 Сценарии](02-scripts.md) · [03 Аналитика](03-control.md) · [04 Шаблоны](04-templates.md) · [05 Процессы](05-processes.md) · **06 Настройка**

---

Этот документ — **click-only пилот** Odoo 19 Community для одного отдела.

Цель: сначала доказать, что штатная модель работает, и только затем обсуждать код.

## 1. Устанавливать приложения по предмету, а не пачкой

### База

- Project;
- Employees;
- Contacts;
- Dashboards — после настройки рабочих views.

### По реальной потребности

- Fleet;
- Maintenance;
- Inventory;
- Calendar;
- Timesheets;
- Website;
- Surveys;
- eLearning;
- Purchase;
- Expenses.

Не ставить приложение только потому, что оно есть в Community.

## 2. Сначала master data

До задач определить источники истины:

```text
сотрудники      → Employees
контакты        → Contacts
ТС              → Fleet
оборудование    → Maintenance
serial/location → Inventory
```

Если модуль не нужен предметно, его не включать.

## 3. Пользователи ≠ Employees

Создать Internal Users только для тех, кто реально работает в Odoo.

Employee может существовать без User.

Assignee Task выбирается из Users.

Property `Сотрудник` при необходимости может ссылаться на **любого доступного Employee**, даже если он не является исполнителем задачи.

## 4. Массовая загрузка справочников

Использовать стандартный Import:

```text
List
→ Actions
→ Import records
→ файл CSV/XLSX
→ сопоставление полей
→ Test
→ Import
```

Для повторных обновлений сохранять External ID.

Не вводить тысячи записей вручную.

## 5. Проверить штатные bridge modules

После установки сочетаний приложений проверить, какие bridges установились автоматически.

Ключевые:

```text
Employees + Fleet        → hr_fleet
Employees + Maintenance  → hr_maintenance
Inventory + Maintenance  → stock_maintenance
Skills + Surveys         → hr_skills_survey
Skills + eLearning       → hr_skills_slides
Project + Purchase       → project_purchase
Project + Inventory      → project_stock
Project + Accounting     → project_account
Project + Expenses       → project_hr_expense
```

До custom code проверить именно эти связи.

## 6. Создать один основной операционный Project

Например:

```text
Операционная работа
```

Не создавать отдельные Projects `Топливо`, `Транспорт`, `Табели`, `Запросы` только ради классификации.

## 7. Visibility

Для пилота выбрать минимальную достаточную Project visibility для внутренних пользователей.

Portal/collaborators не включать без конкретного внешнего сценария.

Права проверять под обычным пользователем, а не только под Administrator.

## 8. Task Stages

Базово:

```text
Входящие
Очередь
В работе
Ожидание внешнего
На проверке   [только если реально нужно]
```

Не делать отдельный Stage `Готово`, если закрытие нормально решается State `Done`.

Не делать Stage `Waiting`: внутреннее Waiting создаётся Dependencies.

## 9. State

Обычные State:

```text
In Progress
Changes Requested
Approved
Done
Canceled
```

Computed dependency state:

```text
Waiting
```

## 10. Создать классификационные Properties

Открыть Task основного Project:

```text
Actions
→ Edit Properties
```

### `Вид работы`

```text
Type: Selection
Values:
- Работа
- Запрос
- Улучшение
```

### `Процесс`

Создавать только если список процессов уже понятен и этот разрез будет использоваться.

## 11. Создать relational Property `ТС`

Только если процессы действительно работают с ТС.

```text
Actions
→ Edit Properties
→ Add a property

Label: ТС
Field Type: Many2one
Model: Fleet Vehicle / fleet.vehicle
Domain: при необходимости
Default: обычно пусто
```

### Что проверить

1. поиск Vehicle по имени/display name;
2. выбор существующей записи;
3. переход в карточку Vehicle;
4. отсутствие доступа к запрещённым Vehicles;
5. фильтрацию Tasks по Property;
6. `Group By → Properties`;
7. отображение Property в карточках, если включён `Show in cards`.

## 12. Property `Сотрудник`

Если Task касается сотрудника, но этот человек не является Assignee:

```text
Label: Сотрудник
Type: Many2one
Model: Employee / hr.employee
```

Пример:

```text
Assignee = аналитик Иванов
Сотрудник = Петров, по данным которого выполняется проверка
```

Не подменять эти роли друг другом.

## 13. Property `Оборудование`

Если Task относится к Equipment:

```text
Label: Оборудование
Type: Many2one
Model: Maintenance Equipment / maintenance.equipment
```

Если событие является обычной Maintenance Request, параллельную Task создавать не нужно.

## 14. Many2many Property

Использовать только если одна Task действительно относится к нескольким объектам.

Пример:

```text
Затронутые ТС → Many2many(fleet.vehicle)
```

Но прежде проверить, не лучше ли разделить работу на несколько Tasks с отдельным контролем.

## 15. Domain для relational Property

В Edit Properties для Many2one/Many2many можно настроить Domain.

Использовать для удобства выбора, например:

```text
только активные records
только конкретный тип
только записи нужной компании
```

Domain **не является защитой данных**.

Пользовательские access rights целевой модели всё равно должны быть правильными.

## 16. Важная проверка create rights

Relational Property использует стандартный Many2X autocomplete, который технически может предлагать создание record, если у пользователя есть create permission.

Для master data определить:

```text
кто может создавать Employee
кто может создавать Vehicle
кто может создавать Equipment
кто только выбирает существующее
```

Не давать всем Project Users право создавать master data только ради удобства Property.

## 17. Assignee и общая очередь

Неназначенная готовая Task:

```text
Stage = Очередь
Assignees = empty
```

Это штатная общая очередь.

Не создавать fake-user `Отдел` или `Общая очередь`.

## 18. Deadline

Deadline = конечный срок результата.

Для напоминания использовать Activity.

## 19. Priority

В UI доступны четыре уровня.

На старте достаточно:

```text
обычная → default
критичная → Urgent
```

Не заставлять сотрудников выбирать Medium/High без критериев.

## 20. Activities

Минимальный полезный набор Activity Types:

```text
Проверить
Напомнить
Позвонить
Встреча
```

При `Ожидание внешнего` Activity обязательна как следующая контрольная точка.

## 21. Activity chaining

До Automation Rule проверить:

- Suggest Next Activity;
- Trigger Next Activity;
- schedule relative to deadline/completion.

Если этого хватает, Automation не нужна.

## 22. Activity Plans

Использовать для повторяемого набора follow-up вокруг одной записи.

Не использовать как замену нескольким самостоятельным Tasks.

## 23. Dependencies

Включить Task Dependencies только если существуют реальные блокировки.

В successor Task:

```text
Blocked by = predecessor Task
```

Odoo сам вычисляет Waiting.

## 24. Recurring Tasks

Для календарной периодики одного результата.

Пример:

```text
Провести ежемесячную сверку
```

Не создавать 12 копий вручную.

## 25. Task Templates

Для одинаковых разовых задач, возникающих по событию.

Примеры:

```text
Разобрать расхождение
Проверить корректировку
Подготовить типовой ответ
```

В шаблоне держать устойчивую структуру, а не конкретные данные текущего случая.

Проверить копирование subtasks и Properties на тестовой базе.

## 26. Subtasks

Использовать только для частей с самостоятельным результатом/владельцем/сроком.

Не превращать Task в дерево из технических шагов.

## 27. Rotting

После наблюдения реальной длительности Stages настроить `Days to rot` только там, где сигнал полезен.

Не ставить одинаковый threshold на все Stages.

## 28. Shared Favorites

Создать минимум:

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
```

Если используются предметные Properties — добавить только реально нужные views.

Например:

```text
Открытые по ТС
Открытые по оборудованию
```

## 29. Project Top Bar

Наиболее частые views сохранить:

```text
Save View
→ Shared
```

Не выносить туда каждый возможный фильтр.

## 30. List и Kanban

### Kanban

Для движения по Stages.

### List

Для больших очередей, сортировки и сравнительного контроля полей.

Для руководителя большой очереди List часто полезнее Kanban.

## 31. Проверить поиск и группировку relational Properties

Минимальный тест:

```text
создать 10 Tasks
назначить им 3 разных Vehicles через Property
→ Filter по конкретному ТС
→ Group By → Properties → ТС
```

То же для Employee/Equipment, если используется.

Это обязательная проверка **до** решения писать custom field.

## 32. Task Analysis

Сначала проверить стандартные показатели:

- created;
- closed;
- open backlog;
- Stage;
- Assignee;
- Priority;
- working time to assign;
- working time to close.

Properties использовать как дополнительные разрезы там, где standard UI их поддерживает.

## 33. Time in Stage

Открыть Task и проверить statusbar: Odoo показывает длительность в каждом Stage.

Использовать для разбора конкретной зависшей Task.

Не обещать агрегированный SLA-report без отдельного теста/доработки.

## 34. My Dashboard

После стабилизации Shared Views добавить в My Dashboard:

- Просрочено;
- Неназначенная очередь;
- Ожидание внешнего;
- Urgent;
- Pivot по Stage/Assignee;
- Graph закрытия.

При необходимости добавить view, сгруппированный по relational Property.

## 35. Spreadsheet Dashboard

Только когда несколько периодов подряд используются одинаковые KPI.

Не начинать внедрение с красивого dashboard.

## 36. Project Stages для инициатив

Если появляются отдельные Project-инициативы, включить Project Stages.

Не использовать для единственного постоянного операционного Project без причины.

## 37. Project Template / Roles / Milestones / Updates

Использовать для самостоятельных инициатив.

Не тащить их в обычную ежедневную Task-очередь.

## 38. Inventory + Maintenance

Если нужен серийный учёт оборудования:

1. создать serial/lot в Inventory;
2. создать Equipment с соответствующим serial;
3. проверить `stock_maintenance`;
4. проверить location;
5. проверить employee allocation через `hr_maintenance`.

Только потом решать, нужна ли собственная asset-модель.

## 39. Fleet + Employees

Если Fleet используется:

1. создать Employee;
2. Vehicle/driver;
3. проверить `hr_fleet` / Fleet History;
4. затем проверить Task Property `ТС`.

## 40. Contacts dedupe

Перед импортом большого справочника проверить:

- правила External ID;
- Merge;
- Deduplicate other Contacts.

Не включать Data Recycle до определения retention policy.

## 41. Email Alias

Включать только после нормального ручного триажа.

Новые email-created Tasks должны попадать во `Входящие`.

## 42. Website Form

Использовать только как простой intake.

Если форма требует сложной бизнес-логики, это уже отдельная интеграционная задача.

## 43. API `/doc`

На тестовом self-hosted Odoo 19 проверить:

```text
/doc
```

Под системным администратором должны быть доступны динамические модели/поля/методы конкретной базы.

Это основной источник для будущей JSON-2 интеграции.

## 44. JSON-2

Для будущего BI/n8n:

```text
отдельный technical user
→ минимальные права
→ API key
→ /json/2/<model>/<method>
```

Не использовать admin API key как общий интеграционный пароль.

## 45. Webhooks

После стабилизации процесса можно тестировать:

- inbound `On webhook` Automation Rule;
- outbound `Send Webhook Notification`.

Только на тестовой базе и с ограниченным payload.

## 46. Automation Rules

До Automation проверить:

```text
Property
Task Template
Activity chaining
Activity Plan
Recurring Task
Shared View
bridge module
```

Если простой механизм решает проблему — Automation не нужна.

## 47. Безопасность production

Вместе с IT проверить:

- LDAP/OAuth при наличии корпоративного IdP;
- TOTP 2FA;
- Passkeys — опционально;
- application access rights;
- Project visibility;
- права на master data models;
- права на records через bridge modules.

## 48. Пилотные сценарии

До признания модели рабочей прогнать:

1. manual Task → очередь → работа → Done;
2. unassigned queue → self-assign;
3. external waiting + Activity;
4. dependency → Waiting → unblock;
5. recurring task;
6. Task Template → новая Task;
7. Subtask;
8. relational Property `ТС` → выбор Fleet Vehicle;
9. relational Property `ТС` → filter/group;
10. relational Property `Сотрудник`;
11. relational Property `Оборудование`;
12. Shared Favorite другим пользователем;
13. Shared Top Bar view;
14. Rotting;
15. Task stage duration;
16. Task Analysis;
17. My Dashboard;
18. Import + External ID + повторный update;
19. master data create-right restriction;
20. bridge modules нужных приложений.

Опционально после стабилизации:

21. email alias;
22. Website Form;
23. JSON-2;
24. inbound webhook;
25. outbound webhook;
26. Timesheets;
27. Automation Rule.

## 49. Когда фиксировать gap

Gap фиксируется только после теста вида:

```text
Требование
→ штатная предметная модель
→ relational Property
→ bridge module
→ standard report/view
→ реальный объём
→ что именно не работает
```

Пример корректного gap:

```text
Property ТС работает функционально,
но на X задачах Group By занимает неприемлемое время
или API/BI не может устойчиво использовать эту связь.
```

Тогда можно обсуждать schema field.

Пример некорректного gap:

```text
в project.task нет vehicle_id в Python-модели
```

Само по себе это больше не проблема: Many2one Property может закрывать пользовательскую связь click-only.

## 50. Порядок внедрения

```text
1. Users / Employees / Contacts
2. нужные master data apps
3. bridge modules
4. основной Project
5. Stages / State rules / Assignee / Deadline
6. классификационные Properties
7. relational Properties
8. Activities
9. Dependencies / Recurrence / Task Templates
10. Shared Views
11. Task Analysis
12. Rotting / stage duration
13. My Dashboard
14. Import/External ID procedure
15. API/webhooks по потребности
16. Automation по потребности
17. зафиксированные gaps
18. минимальный custom module только на доказанные gaps
```

---

[← 05 — Процессы](05-processes.md) · [Главная](../README.md)
