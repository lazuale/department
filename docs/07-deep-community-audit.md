# Углублённый аудит Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [06 Настройка](06-workspace.md) · **07 Аудит** · [08 Интеграции Project](08-project-integrations.md) · [09 Project и безопасность](09-project-productivity-security.md) · [10 API](10-external-integrations.md) · [11 Properties](11-relational-properties.md)

---

Этот документ фиксирует дополнительные Community-возможности, найденные после первого прохода. Актуальная архитектурная граница находится в [00](00-odoo19-community.md).

## 1. Главная поправка позднего прохода

Первоначальная гипотеза, что Project Task нельзя click-only связать с Fleet/Employee/Equipment, оказалась неверной.

Odoo 19 Properties поддерживают Many2one/Many2many к выбранной модели.

Поэтому baseline:

```text
Task Property[ТС]           → fleet.vehicle
Task Property[Сотрудник]    → hr.employee
Task Property[Оборудование] → maintenance.equipment
```

Подробно: [11 — Relational Properties](11-relational-properties.md).

Все ранние тезисы этого PR о необходимости custom field только из-за отсутствия `project.task.vehicle_id` считаются отменёнными.

## 2. Task Templates

В Community Project есть отдельные **Task Templates**.

Это не Recurring Task и не Project Template.

Подтверждено public source/tests:

- template Task создаёт обычную Task;
- subtasks шаблона могут копироваться;
- template Tasks не входят в обычный open backlog;
- при копировании Project шаблоны Tasks сохраняются.

Источники:

- [`project_task_template_dropdown.js`](https://github.com/odoo/odoo/blob/19.0/addons/project/static/src/views/components/project_task_template_dropdown.js);
- [`test_task_templates.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/tests/test_task_templates.py).

Методический выбор:

```text
разовая типовая Task по событию → Task Template
Task по календарю → Recurring Task
набор follow-up → Activity Plan
повторяемая структура проекта → Project Template
```

## 3. Время в Task Stage

`project.task` наследует `mail.tracking.duration.mixin`, а standard statusbar показывает длительность нахождения Task в каждом Stage.

Это полезно для диагностики конкретного кейса.

Но готовый агрегированный SLA-report по каждому Stage в стандартном Pivot/Graph не подтверждён.

## 4. Project Calendar: Tasks to Plan

Community Calendar позволяет брать Tasks без Deadline из `Tasks to Plan` и назначать им дату через Calendar.

Это работает с `date_deadline`.

Не путать с capacity Planning или сменным расписанием.

## 5. Project Templates: структура да, capacity scheduling — не подтверждено

В Community подтверждены:

- Project Templates;
- Project Roles;
- копирование структуры/Tasks/Subtasks/configuration.

Общая документация также описывает availability-based planned-date scheduling, но соответствующий planned-date model layer не подтверждён в public Community `project.task`.

Поэтому эту часть не включаем в гарантированную CE-методику.

## 6. Resource Working Schedules

Public `resource` поддерживает:

- fixed working hours;
- flexible schedule;
- timezone;
- two-week calendar;
- leaves.

Это рабочий календарь HR/resource layer, а не Enterprise Planning.

Для произвольного скользящего графика сначала нужен отдельный пилот.

## 7. Разные измерения времени сотрудника

Не смешивать:

| Механизм | Смысл |
|---|---|
| Attendances | check-in/check-out, присутствие |
| Timesheets | заявленные трудозатраты |
| Work Entries | рабочие интервалы HR |
| Allocated Time | ожидаемая трудоёмкость Task |
| Deadline | срок результата |

Presence Control также не является производительностью.

## 8. Skills Management

Community `hr_skills` содержит:

- skills;
- skill types;
- levels;
- employee resume;
- skill history;
- reports;
- certifications layer.

Навыки должны жить в HR-модели, а не в Project Tags.

## 9. Certifications + Surveys

Bridge `hr_skills_survey` связывает Skills и Surveys.

Возможный штатный контур:

```text
Survey / assessment
→ certification
→ Employee resume
→ validity / expiration
```

Полезно для контроля обязательных компетенций без собственного регистра.

## 10. eLearning + Skills

Community eLearning — `website_slides`.

Bridge `hr_skills_slides` добавляет завершённые курсы в resume сотрудника.

Это отдельный контур обучения, не операционный backlog.

## 11. HR Org Chart

`hr_org_chart` добавляет штатную иерархию Employees.

Не выводить из оргструктуры автоматически ответственность за Tasks.

## 12. Employees ↔ Fleet

Auto-install `hr_fleet` хранит историю автомобилей, которыми управлял Employee.

Это штатная связь HR ↔ Fleet.

Для связи Task ↔ Vehicle использовать relational Property, если она нужна процессу.

## 13. Employees ↔ Maintenance

`hr_maintenance` связывает Employee и Equipment и поддерживает allocation tracking.

Перед собственным реестром «оборудование сотрудника» проверить этот контур.

## 14. Inventory ↔ Maintenance

`stock_maintenance` добавляет Equipment связь с stock location и smart action к совпавшему serial/lot.

Поэтому для серийного оборудования сначала пилотировать:

```text
Inventory
+ Maintenance
+ stock_maintenance
+ hr_maintenance
```

а не писать asset registry с нуля.

## 15. Inventory и Maintenance — разные вопросы

### Inventory

- serial/lot;
- quantity;
- location;
- movement;
- traceability.

### Maintenance

- Equipment;
- owner/allocation;
- preventive/corrective maintenance;
- maintenance request;
- team/responsible;
- service history.

Один физический объект может участвовать в обоих контурах.

## 16. Inventory Barcode

Полный warehouse Barcode UI из общей документации не подтверждён отдельным public `stock_barcode` модулем в `odoo/odoo:19.0`.

Технический `barcodes` существует, но этого недостаточно, чтобы обещать Enterprise Barcode app в CE.

## 17. Contacts Merge / Deduplicate

Для Contacts штатно подтверждены:

- manual Merge;
- Deduplicate other Contacts;
- критерии поиска дублей;
- manual/automatic merge.

Для Contacts не нужен отдельный data-cleaning модуль только ради дублей.

## 18. Data Recycle vs Data Cleaning

Официальная документация разделяет:

```text
data_recycle → Community
data_cleaning → Enterprise
data_merge → Enterprise
```

Data Recycle не включать до определения retention policy.

## 19. Canned Responses

Штатные canned responses доступны в Discuss/Chatter/Live Chat.

Использовать для повторяемых человеческих ответов, а не вместо Stage/State/Activity.

## 20. Live Chat

`im_livechat` есть в Community и содержит visitor chat/operators/chatbot/tags/ratings/reports.

Для методики это входящий коммуникационный канал, не Helpdesk и не Task.

## 21. Calendar sync

Public source содержит Google и Microsoft Calendar integrations.

Подключать, если нужен единый календарь встреч.

Не считать синхронизацией Project Deadlines.

## 22. Analytic Accounting

Community `analytic` подтверждает:

- Analytic Accounts;
- Analytic Plans;
- Distribution Models;
- Pivot/Graph.

Использовать только при реальном финансово-стоимостном вопросе.

## 23. Budget Management

Общая документация описывает Budget Management, а public `account` содержит модульный переключатель, но сам `account_budget` не подтверждён в публичной ветке Community.

Поэтому Budget не входит в гарантированный CE baseline.

## 24. Purchase Agreements

Community `purchase_requisition` поддерживает calls for tenders / blanket orders.

Использовать только для реальной закупочной модели, не как универсальное согласование.

## 25. Specialized workflows

Штатные workflow должны оставаться предметными:

| Предмет | Приложение |
|---|---|
| найм | Recruitment |
| расходы | Expenses |
| отсутствие | Time Off |
| закупка | Purchase |
| обслуживание | Maintenance |
| repair stock product | Repairs |
| опрос/тест | Surveys |

Project Task создаётся поверх них только при отдельном управленческом результате.

## 26. Presence Control

`hr_presence` технически определяет presence по системным сигналам, включая session/email/IP.

Не использовать как KPI производительности или качества.

## 27. Gamification

Community Gamification поддерживает goals/challenges/badges.

Не использовать для рейтинга операционной производительности по Task counts/hours.

## 28. Remote Work / HR Calendar

Community имеет `hr_homeworking` и `hr_calendar`.

Это HR-контекст, а не замена Project workflow.

## 29. Ключевое правило bridge modules

Перед custom code проверять:

```text
предметная модель A
+ предметная модель B
→ auto-install bridges
→ relations / smart buttons / reports
→ relational Property для Task при необходимости
→ только затем residual gap
```

## 30. Что действительно осталось проверять эмпирически

После широкого source-аудита главные неизвестные уже не «какой ещё модуль существует», а:

- пригодность Fleet для реального парка спецтехники;
- производительность relational Properties на реальном объёме Tasks/records;
- Import Tasks с dynamic Properties;
- JSON-2/BI работа с Properties;
- ACL target models через relational Properties;
- русская локализация конкретных экранов;
- сменный график 2/2 и рабочие календари;
- cross-app dashboard usability;
- email alias на реальной почтовой инфраструктуре;
- необходимость агрегированного time-in-stage reporting.

Это уже задачи тестового стенда, а не чтения каталога модулей.

---

[← 06 — Настройка](06-workspace.md) · [08 — Интеграции Project →](08-project-integrations.md)
