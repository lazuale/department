# Опорный реестр модулей Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [06 Настройка](06-workspace.md) · [19 Master data](19-master-data.md) · [24 Люди](24-master-data-people.md) · [25 Техника](25-master-data-assets.md) · [26 Task ↔ master data](26-task-master-data-relations.md)

---

Этот файл — технический source of truth по официальным модулям Odoo 19 Community и их месту в текущей архитектуре.

Модуль оценивается не по принципу «может пригодиться», а от уже принятой предметной модели:

```text
реальный master record / процесс
→ штатная Odoo model
→ модуль-владелец model
→ обязательные dependencies / auto-install bridges
→ только затем дополнительные кандидаты
```

Наличие модуля в Community source не означает, что его нужно использовать методически.

## 1. Источники и состояние проверки

Полный каталог Community Apps ранее проверен на snapshot:

```text
repository: odoo/odoo
branch:     19.0
snapshot:   cb731ca8720cd01a5719890cf6c5e140dc551546
```

После согласования master data 2026-08-13 повторно проверены критичные manifests/source текущей ветки `19.0`:

```text
addons/contacts/__manifest__.py
addons/fleet/__manifest__.py
addons/maintenance/__manifest__.py
addons/stock/__manifest__.py
addons/stock_maintenance/__manifest__.py
addons/project_stock/__manifest__.py
addons/project_stock/models/stock_picking.py
addons/project_stock/models/project_project.py
addons/barcodes/__manifest__.py
addons/barcodes_gs1_nomenclature/__manifest__.py
```

Правило доверия:

1. public source `odoo/odoo:19.0` — наличие, dependencies, auto-install и модель;
2. официальная документация Odoo 19 — пользовательский смысл;
3. runtime нашего стенда — фактический UI и права;
4. решение проекта — только после сопоставления с реальным процессом.

## 2. Согласованные master data определяют обязательные Apps

Сейчас зафиксировано:

```text
люди / сотрудники              → res.partner
путевая техника                → fleet.vehicle
сканирующие рамки              → maintenance.equipment
терминал Nobilis как комплект  → stock.package
тип части Nobilis              → product.product
серийная часть Nobilis         → stock.lot
движение / положение Nobilis   → stock.*
работа                          → project.task
```

Отсюда без дополнительного выбора следуют приложения:

```text
Project      project
Contacts     contacts
Fleet        fleet
Maintenance  maintenance
Inventory    stock
Discuss      mail
```

`Discuss / mail` одновременно является инфраструктурой Chatter и Activities для Project, Fleet и Maintenance.

## 3. Статусы

| Статус | Значение |
|---|---|
| **Используем** | нужен принятой предметной модели или базовой модели работы |
| **Возможно** | нужен только при появлении конкретного процесса/требования |
| **Вероятно нет** | известные процессы его не требуют либо он создаст второй конкурирующий контур |
| **Автоматически присутствует** | приезжает dependency/auto-install вследствие уже выбранных Apps; отдельного решения на установку не требует |
| **Не Community baseline** | нельзя считать штатной возможностью CE |

## 4. Community Apps — актуальный выбор 34/34

| Группа | App | Technical name | Решение | Причина |
|---|---|---|---|---|
| Работа | **Project / Проекты** | `project` | **Используем** | `project.task` — управляемая работа, сроки, workflow, Activities, зависимости и анализ |
| Коммуникация | **Discuss / Обсуждения** | `mail` | **Используем** | Chatter, Activities и коммуникационная инфраструктура |
| HR | **Employees / Сотрудники** | `hr` | **Вероятно нет** | master людей уже принят на `res.partner`; HR нужен только при реальном HR-процессе |
| Справочник | **Contacts / Контакты** | `contacts` | **Используем** | рабочий каталог людей на `res.partner` |
| Транспорт | **Fleet / Автопарк** | `fleet` | **Используем** | master путевой техники на `fleet.vehicle` |
| Productivity | **Calendar / Календарь** | `calendar` | **Возможно** | только при реальной потребности во встречах/календарных событиях |
| Productivity | **To-Do** | `project_todo` | **Возможно методически; автоматически присутствует с Project** | технически приезжает, но не должен стать вторым backlog |
| Data | **Data Recycle** | `data_recycle` | **Возможно** | административный lifecycle устаревших records после определения правил |
| Supply Chain | **Inventory / Склад** | `stock` | **Используем** | Nobilis: package, serial/lot, location, quant, stock moves и traceability |
| Supply Chain | **Maintenance / Обслуживание** | `maintenance` | **Используем** | master сканирующих рамок и maintenance lifecycle |
| Data collection | **Surveys / Опросы** | `survey` | **Возможно** | только если появится реальный структурированный сбор данных |
| Website | **Website / Сайт** | `website` | **Возможно** | кандидат для будущего web-входа заявок |
| Supply Chain | **Purchase / Закупки** | `purchase` | **Возможно** | только если сам закупочный процесс переносится в Odoo; Stock и возвраты сами по себе Purchase не требуют |
| Accounting | **Accounting / Бухгалтерия** | `account` | **Вероятно нет** | не создаём второй бухгалтерский контур |
| Sales | **CRM** | `crm` | **Вероятно нет** | входящие обязательства не являются leads/opportunities |
| Sales | **Sales / Продажи** | `sale_management` | **Вероятно нет** | вне текущего контура |
| Sales | **Point of Sale** | `point_of_sale` | **Вероятно нет** | вне текущего контура |
| Sales | **Restaurant POS** | `pos_restaurant` | **Вероятно нет** | вне текущего контура |
| Manufacturing | **Manufacturing / Производство** | `mrp` | **Вероятно нет** | master Nobilis — учёт комплектов и частей, а не производство изделий |
| Supply Chain | **Repair / Ремонт** | `repair` | **Вероятно нет** | рамки закрываются Maintenance; Nobilis сейчас перемещается/возвращается производителю, отдельный внутренний Repair workflow не заявлен |
| HR | **Attendances / Посещаемость** | `hr_attendance` | **Вероятно нет** | не является текущей системой табеля |
| HR | **Time Off / Отпуска** | `hr_holidays` | **Вероятно нет** | вне текущего контура |
| HR | **Expenses / Расходы** | `hr_expense` | **Вероятно нет** | вне текущего контура |
| HR | **Skills / Навыки** | `hr_skills` | **Вероятно нет** | вне текущего контура |
| HR | **Recruitment / Подбор** | `hr_recruitment` | **Вероятно нет** | вне текущего контура |
| Internal services | **Lunch / Питание** | `lunch` | **Вероятно нет** | корпоративный заказ еды ≠ наши сверки |
| Marketing | **Email Marketing** | `mass_mailing` | **Вероятно нет** | маркетинговый процесс не нужен |
| Marketing | **SMS Marketing** | `mass_mailing_sms` | **Вероятно нет** | вне текущего контура |
| Marketing | **Marketing Card** | `marketing_card` | **Вероятно нет** | вне текущего контура |
| Website | **Live Chat** | `im_livechat` | **Вероятно нет** | публичный chat не нужен текущей модели |
| Website | **eCommerce** | `website_sale` | **Вероятно нет** | вне текущего контура |
| Website | **Events** | `website_event` | **Вероятно нет** | вне текущего контура |
| Website | **eLearning** | `website_slides` | **Вероятно нет** | вне текущего контура |
| Website / HR | **Online Recruitment** | `website_hr_recruitment` | **Вероятно нет** | вне текущего контура |

Контроль:

```text
Используем:      6 Apps
Возможно:        6 Apps
Вероятно нет:   22 Apps
Итого:          34 Apps
```

Главное изменение после согласования master data:

```text
Inventory   Возможно → Используем
Maintenance Возможно → Используем
```

## 5. Технические модули, которые теперь входят в baseline автоматически

### Inventory dependency chain

Manifest `stock` зависит от:

```text
product
barcodes_gs1_nomenclature
    ↓
barcodes
uom

digest
```

Поэтому после принятия Inventory для Nobilis:

| Модуль | Статус | Что это означает |
|---|---|---|
| `product` | **Автоматически присутствует / используем технически** | владелец `product.product`, который уже является master типа комплектующей |
| `barcodes_gs1_nomenclature` | **Автоматически присутствует** | dependency Inventory |
| `barcodes` | **Автоматически присутствует** | dependency GS1 nomenclature; наличие библиотеки сканирования не равно Enterprise Barcode App |
| `digest` | **Автоматически присутствует** | dependency Inventory/других Apps, не отдельный бизнес-процесс |

Полный Enterprise-style Barcode workflow не принимается как Community baseline только из-за наличия `barcodes`.

### Project technical layer

```text
project
  ↓
project_todo [auto_install]
```

`project_todo` технически присутствует, но его использование как отдельного рабочего списка остаётся методическим выбором.

### Dashboard / API

```text
board → spreadsheet_dashboard → spreadsheet
web   → api_doc [auto_install]
```

`board` / My Dashboard и `api_doc` остаются используемой технической инфраструктурой. Полный Documents-style Spreadsheet workflow не считается обязательным CE baseline.

## 6. Auto-install bridges, которые теперь неизбежны

### Inventory + Maintenance → `stock_maintenance`

```text
stock + maintenance
        ↓
stock_maintenance [auto_install=True]
```

Поскольку оба родительских Apps теперь **Используем**, bridge больше не является кандидатом — он **автоматически присутствует**.

Он, в частности, расширяет `maintenance.equipment`:

```text
location_id → stock.location
```

и даёт переход к совпадающему `stock.lot` по serial number.

Это техническая возможность. Она **не означает**, что рамки автоматически становятся Stock items и что `stock.location` уже принят универсальной моделью «Участок». Этот вопрос остаётся отдельным master-data решением.

### Project + Inventory → `project_stock`

```text
project + stock
      ↓
project_stock [auto_install=True]
```

Поскольку Project и Inventory уже **Используем**, bridge также **автоматически присутствует**.

Его фактическая связь в Odoo 19:

```text
stock.picking.project_id → project.project
```

а на Project появляются действия для связанных Stock Pickings.

Важно:

> `project_stock` связывает складскую операцию с **Project**, а не `project.task` с `stock.package` или `stock.lot`.

Поэтому он не заменяет согласованную связь Task ↔ Nobilis через relational Properties из `docs/26-task-master-data-relations.md`.

### HR bridges

```text
hr + fleet       → hr_fleet
hr + maintenance → hr_maintenance
```

Они не входят в baseline, поскольку `hr` не выбран.

## 7. Реальные кандидаты после фиксации master data

Master data больше не дают оснований держать длинный список «может пригодиться».

Остаются кандидаты только по конкретным будущим процессам:

| Приоритет | Модуль | Когда рассматривать | Что НЕ является основанием |
|---|---|---|---|
| 1 | `base_automation` | когда стабилизированы правила и нужна автоматизация рутинных действий | желание автоматизировать ещё не определённый процесс |
| 1 | `website` + `website_project` | если нужен web-вход заявок от пользователей вне внутреннего Project UI | само наличие Portal |
| 2 | `calendar` | если встречи/календарные события становятся частью рабочего процесса | Deadline Task уже не требует Calendar |
| 2 | `data_recycle` | если определены правила архивирования/очистки устаревших records | наличие большого справочника само по себе |
| 2 | `survey` | если нужен самостоятельный структурированный опрос/анкета | обычная форма Task |
| 3 | `purchase` | только если закупки, заказы поставщикам и соответствующий lifecycle реально ведём в Odoo | Nobilis, serials, перемещения или возврат производителю сами по себе |

Инфраструктура рассматривается отдельно:

```text
auth_ldap / auth_oauth / auth_passkey / auth_totp
google_gmail / microsoft_outlook
google_calendar / microsoft_calendar
```

## 8. Что master data, наоборот, исключили из ближайших кандидатов

### Employees / HR

```text
люди → res.partner
доступ → res.users
```

Поэтому `hr` не нужен ради справочника людей, Fleet drivers или ссылок из Task.

### Repair

```text
рамка → maintenance.equipment + maintenance.request
Nobilis → stock traceability / возврат производителю
```

Отдельный `repair` появится в рассмотрении только если возникнет реальный внутренний repair-order workflow, которого сейчас нет.

### Manufacturing

Комплект Nobilis моделируется как `stock.package` с серийными частями. Это не производственная спецификация и не основание включать `mrp`.

### HR time modules

`hr_timesheet`, `hr_attendance`, `hr_work_entry` не являются заменой действующим табельным данным или внешним источникам сверки.

### Accounting / Sales / CRM

Master data техники и оборудования не требуют переносить бухгалтерию, продажи или CRM в Odoo.

## 9. Базовый контур после master-data решений

### Осознанно используем

```text
Project        project
Discuss        mail
Contacts       contacts
Fleet          fleet
Maintenance    maintenance
Inventory      stock
My Dashboard   board
API docs       api_doc
```

### Технически присутствует вследствие выбранного контура

```text
product
barcodes
barcodes_gs1_nomenclature
project_todo
project_stock
stock_maintenance
spreadsheet_dashboard
spreadsheet
api_doc
и остальные прямые dependencies выбранных Apps
```

### Не входит в baseline

```text
Employees / HR
Purchase
Repair
Manufacturing
Accounting
CRM / Sales
Website
Calendar
Survey
```

Некоторые из них остаются кандидатами, но не устанавливаются до появления соответствующего процесса.

## 10. Предметные границы

### Люди

```text
человек / сотрудник → res.partner
доступ в Odoo       → res.users
```

### Путевая техника

```text
самосвалы + строительная техника + малая механизация
→ fleet.vehicle

путевые листы
→ действующая учётная система

работа по технике
→ project.task + relation на fleet.vehicle
```

### Рамки

```text
рамка                 → maintenance.equipment
обслуживание / ремонт → Maintenance
работа                → project.task + relation на Equipment
```

### Nobilis

```text
терминал / комплект  → stock.package
тип части            → product.product
серийная часть       → stock.lot
положение            → stock.location / stock.quant
движение / состав    → stock.move / stock.move.line
работа               → project.task + relation на Package/Lot при необходимости
```

Inventory является source of truth по физическому составу и движениям; Task не копирует эти данные.

## 11. Крупные функции, которые не считать Community baseline

В public Community source нельзя закладываться как на штатные Apps на:

```text
Studio
Helpdesk
Planning
Documents
Knowledge
Approvals
Sign
Appointments
Quality / Quality Control
полный Enterprise Barcode workflow
Project Gantt / Enterprise planning UX
```

Наличие похожей технической функции не означает наличие соответствующего Enterprise-приложения.

## 12. Критерий следующего включения модуля

Новый App переводится в **Используем** только если есть ответ:

1. какой реальный объект или процесс он представляет;
2. где источник истины;
3. какая штатная model нужна;
4. почему уже выбранные Apps этого не закрывают;
5. не создаёт ли App второй конкурирующий реестр;
6. нужен ли он пользователям ежедневно;
7. какую управленческую или аналитическую ценность он добавляет.

Если таких ответов нет — модуль не устанавливается «на будущее».

## 13. Когда перепроверять

Повторная проверка нужна, если:

- существенно изменился `odoo/odoo:19.0`;
- меняется major-версия;
- принимается новый master-data объект;
- появляется новый реальный процесс;
- runtime показывает отличия от ожидаемого поведения;
- рассматривается OCA/сторонний модуль или custom addon.

---

**Текущее решение:** базовый предметный контур уже определён master data. Следующий модуль выбирается не из общего каталога Odoo, а только под конкретный ещё не закрытый процесс.
