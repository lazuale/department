# Контроль и аналитика

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [02 Сценарии](02-scripts.md) · **03 Аналитика** · [04 Шаблоны](04-templates.md) · [05 Процессы](05-processes.md) · [06 Настройка](06-workspace.md) · [15 Перепроверка](15-capability-recheck.md)

---

Контроль строится по тем же данным, в которых ведётся работа. Отдельный ручной отчёт не нужен, если управленческий вопрос уже решается штатным view Odoo.

## 1. Четыре уровня контроля

| Уровень | Инструмент | Вопрос |
|---|---|---|
| личный | My Tasks, Activities, To-Do | что делать пользователю сейчас |
| оперативный | Shared Views, List/Kanban | где требуется вмешательство сегодня |
| периодический | Task Analysis, Pivot/Graph, My Dashboard | где меняется поток и копится хвост |
| предметный | Fleet / Maintenance / Inventory / HR reports | что происходит с объектами, а не с Tasks |

## 2. Общие views публикуются централизованно

Минимум:

- `Входящие`;
- `Очередь`;
- `Неназначенная очередь`;
- `В работе`;
- `Просрочено`;
- `Ожидание внешнего`;
- `Waiting / Blocking`;
- `Urgent`;
- `Rotting`;
- `Просроченные Activities`.

Использовать Shared Favorites и Project Top Bar `Save View → Shared`.

## 3. Входящие

Вопрос:

> Что зарегистрировано, но ещё не разобрано?

Смотреть:

- количество;
- возраст;
- дубли;
- записи, которые вообще не должны быть Task.

Если входящие копятся, проблема в triage, а не в исполнении.

## 4. Очередь

Разделять:

```text
неназначенная очередь
Stage = Очередь + Assignees empty
```

и

```text
назначенная очередь
Stage = Очередь + Assignee set
```

Неназначенность сама по себе не ошибка. Ошибка — работа, которая должна была стартовать, но не стартовала.

## 5. В работе

Смотреть:

- чрезмерный WIP;
- overdue;
- Urgent;
- Task без движения;
- Task, которая фактически ждёт, но ошибочно оставлена `В работе`.

Большой WIP чаще показывает плохую дисциплину Stage, а не высокую производительность.

## 6. Просрочено

Просрочка = open Task с Deadline в прошлом.

Разбирать минимум по:

- Stage;
- Assignee;
- Priority;
- `Вид работы`;
- `Процесс`, если используется;
- предметному relational Property, если он нужен процессу.

## 7. Ожидание внешнего

Красные сигналы:

```text
Stage = Ожидание внешнего
+ Activity отсутствует
```

или

```text
Activity overdue
```

Это отдельный управленческий риск от внутреннего Waiting.

## 8. Внутренние зависимости

Использовать `Blocked by` / computed `Waiting`.

Смотреть:

- какие Tasks Waiting;
- какая Task блокирует несколько последующих;
- просрочена ли блокирующая Task.

`Blocked by` может связывать Tasks разных Projects, поэтому самостоятельные инициативы не становятся изолированными островами.

## 9. Rotting

Rotting показывает длительное отсутствие смены Stage.

Не смешивать:

```text
Deadline = срок результата
Activity = срок следующего действия
Rotting = отсутствие смены Stage
```

Threshold настраивать по реальной нормальной длительности конкретного Stage.

## 10. Время в Stage

Odoo 19 Community уже показывает в statusbar конкретной Task длительность нахождения в каждом Stage.

Это полезно для разбора одного зависшего случая.

Но стандартный Pivot/Graph не считаем готовым агрегированным SLA-отчётом по каждому Stage.

Если понадобится системная метрика `среднее время в Stage`, это отдельный аналитический gap, который сначала проверяется на тестовом стенде/API.

## 11. Priority

Community поддерживает Low / Medium / High / Urgent.

Основные исключения:

- Urgent open;
- Urgent overdue;
- Urgent unassigned;
- Urgent Waiting.

Если почти всё High/Urgent, Priority перестал быть сигналом.

## 12. Relational Properties — штатные аналитические разрезы Tasks

Если процесс реально требует связи с предметным объектом, Task получает Many2one/Many2many Property.

Примеры:

```text
ТС           → fleet.vehicle
Сотрудник    → hr.employee
Оборудование → maintenance.equipment
```

Стандартный Project Task Search View поддерживает поиск по `task_properties` и `Group By → Properties`.

Поэтому штатно можно отвечать на вопросы:

- какие open Tasks относятся к конкретному ТС;
- какой backlog по ТС;
- какие Tasks относятся к конкретному сотруднику как объекту;
- какие Tasks относятся к Equipment.

Это **не означает**, что предметные показатели объекта надо считать из Tasks.

## 13. Task analytics и предметная analytics не дублируются

### Fleet отвечает на вопросы

- Vehicle;
- driver;
- odometer;
- services;
- costs.

### Project отвечает на вопрос

> Какая работа по этому ТС сейчас открыта/просрочена/Waiting?

Связь между вопросами обеспечивается Property `ТС` Many2one → Fleet Vehicle.

Аналогично:

- Maintenance → состояние/обслуживание Equipment;
- Project → отдельные обязательства по Equipment;
- Employees → карточка сотрудника;
- Project → обязательства, относящиеся к сотруднику как объекту.

## 14. Когда использовать Many2many Property

Если одна Task действительно относится к нескольким объектам, Many2many допустим.

Но аналитически он сложнее.

Перед использованием проверить, не лучше ли разбить результат на несколько самостоятельных Tasks.

## 15. Task Analysis

Штатно использовать для базовых показателей:

- created;
- closed;
- current backlog;
- Stage;
- State;
- Assignee;
- Priority;
- working time to assign;
- working time to close;
- task count.

Не придумывать KPI, которых нет в достоверной модели.

## 16. Минимальный периодический набор

### Поток

- создано;
- закрыто;
- open backlog;
- изменение backlog.

### Риски

- overdue;
- Urgent;
- Waiting;
- external waiting;
- overdue Activities;
- Rotting.

### Распределение

- по Stage;
- по Assignee;
- по Process/Work Type при необходимости;
- по relational Properties там, где это управленческий вопрос.

### Время

- working time to assign;
- working time to close;
- time-in-stage конкретной Task при расследовании.

## 17. Что не является KPI сотрудника

Не использовать напрямую для рейтинга:

- число закрытых Tasks;
- количество комментариев;
- число Activities;
- Timesheet hours без контекста;
- Presence Control;
- Customer Rating;
- WIP count.

Разные Tasks имеют разную сложность и зависимости.

## 18. Allocated Time и Timesheets

### Allocated Time

Ожидаемая трудоёмкость.

Включать в методику только если по нему реально принимается решение о нагрузке.

### Timesheets

Фактически заявленное время.

Включать, если нужен факт трудоёмкости/стоимости/план-факта.

Не использовать как контроль присутствия.

## 19. My Dashboard — основной click-only dashboard layer

После стабилизации Shared Views собрать, например:

1. Просрочено — List;
2. Неназначенная очередь — List;
3. Ожидание внешнего — List;
4. Urgent — List;
5. открытые по Stage/Assignee — Pivot;
6. закрытие по периодам — Graph.

При необходимости добавить view, сгруппированный по `ТС` / `Сотрудник` / `Оборудование` Property.

My Dashboard основан на Odoo views, а не на Spreadsheet, и подтверждён public Community-модулем `board`.

## 20. Spreadsheet Dashboard — не считать Community baseline

Public `spreadsheet` и `spreadsheet_dashboard` действительно существуют в `odoo/odoo:19.0`, но повторная проверка не подтверждает полный click-only create/edit workflow собственного Spreadsheet Dashboard в чистой Community.

Официальный workflow построения dashboard с нуля требует `Dashboards / Admin` и минимум `Documents / User`, а Documents не входит в Community baseline этой методики.

Поэтому текущий порядок аналитики:

```text
Shared Views
→ Task Analysis
→ My Dashboard
→ API / внешний BI при необходимости
```

Spreadsheet Dashboard можно вернуть в архитектуру только после runtime-проверки конкретной Community-инсталляции и фактически доступного набора модулей.

Не считать наличие технических `spreadsheet*` модулей доказательством полной пользовательской функции.

## 21. Project initiatives — отдельный режим контроля

Для самостоятельных инициатив использовать:

- Project Stages;
- Project Manager;
- Project Updates;
- Milestones;
- Project Dashboard/Burndown;
- Project-level Activities;
- при необходимости analytic/purchase/stock/expense integrations.

Не смешивать с ежедневной операционной очередью.

## 22. Качество master data

Перед выводами по analytics проверять:

- дубли Contacts;
- актуальность Employees;
- корректность Fleet/Equipment;
- External IDs при импорте;
- отсутствие параллельных текстовых копий справочника;
- права пользователей на целевые relational models.

## 23. Производительность relational Properties — проверять, а не предполагать

Relational Property — baseline click-only.

До custom field провести тест на реальном масштабе:

```text
объём Tasks
объём target records
filter speed
Group By speed
Kanban/List rendering
API/BI extraction
```

Только измеренная проблема является основанием менять модель.

## 24. Управленческий принцип

Любой показатель должен заканчиваться решением:

> Если X изменилось, мы делаем Y.

Если действия Y нет — показатель не нужен.

---

[← 02 — Сценарии](02-scripts.md) · [04 — Шаблоны →](04-templates.md)
