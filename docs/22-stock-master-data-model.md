# Текущая нативная модель справочников и субъектов Odoo 19 Community

[Главная](../README.md) · [16 Реестр модулей](16-community-modules-context.md) · [20 Аудит досье](20-subject-dossier-stock-audit.md) · [21 Штатный baseline](21-stock-only-baseline.md)

---

Этот документ фиксирует **текущее принятое решение проекта** после повторной проверки по официальной документации Odoo 19 и public source `odoo/odoo:19.0`.

Граница:

> используем только официальные штатные механизмы Odoo 19 Community; custom models, custom fields, OCA и собственный связующий модуль не используются.

Это решение действует до отдельного согласованного изменения.

## 1. Что перепроверено

Повторно проверены:

- `fleet.vehicle` и его штатные history records;
- `hr.employee` и HR Work Location;
- `maintenance.equipment` и `maintenance.request`;
- Inventory Locations;
- lot/serial tracking и traceability;
- Inventory Internal Transfers;
- официальный bridge `stock_maintenance`;
- официальный bridge `hr_maintenance`;
- relational Properties `Many2one` / `Many2many`;
- Packages, Repairs и BoM как возможные кандидаты для состава терминала.

По итогам предыдущая модель `участок = hr.work.location` и `серийная комплектующая = maintenance.equipment` **заменена более нативной штатной схемой**.

## 2. Текущая базовая схема

```text
ТС
→ Fleet / fleet.vehicle

Сотрудник
→ Employees / hr.employee

Операционный участок / площадка
→ Inventory / stock.location (internal)

Терминал Nobilis
→ Maintenance / maintenance.equipment
+ Inventory / product.product + unique stock.lot Serial

Рамка измерения объёма
→ Maintenance / maintenance.equipment
+ Inventory / product.product + unique stock.lot Serial

Серийная комплектующая терминала
→ Inventory / product.product + unique stock.lot Serial

Project Task
→ relational Properties → нужные штатные records
```

Отдельная универсальная сущность `Субъект` не создаётся.

## 3. Приложения текущего решения

Используем:

```text
Project      project
Employees    hr
Fleet        fleet
Maintenance  maintenance
Inventory    stock
Contacts     contacts
Discuss      mail
```

Официальные bridge-модули:

```text
hr_maintenance
stock_maintenance
```

устанавливаются автоматически при наличии зависимостей.

### 3.1. Что даёт `hr_maintenance`

Штатно связывает Equipment с Employee / Department и добавляет Equipment navigation в Employees.

### 3.2. Что даёт `stock_maintenance`

Штатно:

```text
maintenance.equipment.location_id
→ stock.location
```

добавляет переход из Equipment к Inventory serial с совпадающим серийным номером и обратный просмотр Equipment из Stock Location.

Важно:

> `maintenance.equipment.location_id` не является автоматически вычисляемым отражением stock moves.

Фактическую историю движения ведёт Inventory. Поле Equipment Location используется как штатный эксплуатационный атрибут, а не как замена Inventory traceability.

## 4. Идентичность и импорт

Для массовой загрузки используем штатный Odoo `External ID` как интеграционный ключ.

Пример:

```text
vehicle_<source_id>
employee_<source_id>
site_<source_id>
product_<source_id>
serial_<source_id>
equipment_<source_id>
```

Правила:

- не использовать ФИО как уникальный ключ;
- не использовать госномер как единственный вечный ID ТС;
- serial является физическим идентификатором экземпляра, но импортный `External ID` всё равно сохраняем;
- повторные импорты выполняются по устойчивым ключам.

## 5. ТС

### 5.1. Master record

```text
fleet.vehicle
```

Fleet является единственной карточкой ТС.

Используем штатные поля по фактической необходимости:

```text
Model
License Plate
VIN/SN
State
Tags
Location
Properties
```

`Model` штатно обязателен.

### 5.2. Штатное досье ТС

Fleet уже хранит:

```text
Drivers History
Services
Contracts
Odometer
Chatter
Activities
```

Не создаём Project Task для дублирования этих штатных Fleet events.

### 5.3. Операционный участок ТС

Штатное поле Fleet:

```text
fleet.vehicle.location → Char
```

Строгой relation `fleet.vehicle → stock.location` в базовом Fleet нет.

Поэтому текущее принятое решение:

- `Fleet.Location` можно использовать как пользовательский текст текущего места, если это удобно;
- **не считать его master relation участка**;
- в каждой работе, где участок важен для анализа, Task хранит отдельную relation `Участок → stock.location`;
- структурированная текущая привязка самого Vehicle к `stock.location` остаётся штатным gap.

### 5.4. Связь Task → ТС

```text
Property: ТС
Type: Many2one
Model: fleet.vehicle
```

`Many2many` используется только для реального процесса, где одна работа относится сразу к нескольким ТС.

## 6. Сотрудники

### 6.1. Master record

```text
hr.employee
```

Штатная карточка Employees остаётся источником идентичности человека в Odoo.

### 6.2. HR Work Location и операционный участок — разные понятия

Odoo штатно имеет:

```text
hr.employee / hr.version.work_location_id
→ hr.work.location
```

Это **HR Work Location**: место работы сотрудника.

Наш операционный участок одновременно используется для:

- Inventory movement;
- терминалов;
- рамок;
- серийных комплектующих;
- Tasks;
- аналитики.

Поэтому единым master record операционного участка принят `stock.location`, а `hr.work.location` не переименовывается методически в производственный участок.

Если в Employee нужно хранить именно текущий операционный участок:

```text
Employee Property: Операционный участок
Type: Many2one
Model: stock.location
```

HR Work Location при этом можно использовать независимо по его штатному назначению.

### 6.3. Связь Task → сотрудник

```text
Property: Сотрудник
Type: Many2one
Model: hr.employee
```

Это субъект работы, а не Assignee.

## 7. Участки / площадки

### 7.1. Master record

Текущее решение:

```text
stock.location
usage = internal
```

Каждый операционный участок создаётся отдельной Internal Location.

### 7.2. Почему выбран Inventory Location

Это наиболее нативная модель для нашего фактического сценария:

```text
участок A
→ физический объект
→ internal transfer
→ участок B
```

Inventory штатно хранит:

```text
Date
From
To
Product
Quantity
Lot / Serial
Reference
```

и позволяет открыть traceability конкретного serial.

### 7.3. Связь участка с Equipment

Официальный `stock_maintenance` добавляет:

```text
maintenance.equipment.location_id
→ stock.location
```

и smart button Equipment на Stock Location.

Это делает `stock.location` нативнее для эксплуатационного оборудования, чем `hr.work.location`.

### 7.4. Связь Task → участок

```text
Property: Участок
Type: Many2one
Model: stock.location
Domain: internal locations текущего контура
```

При необходимости процесса перемещения:

```text
Task.Property Откуда → stock.location
Task.Property Куда   → stock.location
```

Но сам факт физического движения фиксируется Inventory Transfer, а не Task.

## 8. Терминалы Nobilis

Терминал — один физический объект, но Odoo штатно рассматривает две стороны его жизни разными моделями.

### 8.1. Эксплуатационная карточка

```text
maintenance.equipment
Category = Терминал Nobilis
Serial Number = фактический serial терминала
```

Equipment используется для:

```text
эксплуатационной карточки
Maintenance Requests
ремонтов / профилактики
Chatter
Activities
maintenance metrics
```

### 8.2. Физическая единица Inventory

Создаётся тип товара терминала:

```text
product.product
Track Inventory = By Unique Serial Number
```

Каждый физический терминал:

```text
stock.lot / Serial Number
```

Inventory используется для:

```text
текущего физического местонахождения
Internal Transfers
истории движений
Traceability
```

### 8.3. Связь двух представлений

Используется один фактический serial number.

Официальный `stock_maintenance` умеет из Equipment открыть совпадающий Inventory Serial.

Не создаём третью собственную карточку терминала.

### 8.4. Перемещение терминала

Физическое перемещение:

```text
Inventory Internal Transfer
From: участок A
To:   участок B
Serial: конкретный терминал
```

Project Task создаётся только если существует самостоятельная управленческая работа вокруг перемещения.

### 8.5. Ремонт терминала

```text
maintenance.request
→ equipment_id = терминал
```

Не дублируем ремонт обычной Project Task без самостоятельной причины.

## 9. Рамки измерения объёма

Используется та же двухслойная штатная модель:

```text
maintenance.equipment
+ product.product / unique stock.lot Serial
```

Разделение ролей:

```text
Equipment
→ эксплуатация и ремонты

Inventory Serial
→ текущее физическое место и движения
```

Ремонт:

```text
maintenance.request
```

Перемещение:

```text
Inventory Internal Transfer
```

## 10. Серийные комплектующие терминалов

### 10.1. Базовая модель

Если комплектующая имеет собственный серийный номер и может сниматься, устанавливаться, храниться или перемещаться:

```text
тип комплектующей
→ product.product

конкретный экземпляр
→ stock.lot / unique Serial
```

Это более нативно, чем создавать каждую такую деталь как `maintenance.equipment`.

### 10.2. Когда комплектующая также становится Equipment

Только если сам компонент имеет самостоятельный эксплуатационный lifecycle:

```text
его отдельно обслуживают
его отдельно ремонтируют
по нему нужны Maintenance Requests
```

Тогда допускается дополнительный `maintenance.equipment` с тем же фактическим serial, аналогично терминалу.

## 11. Замены комплектующих

Сам физический serial-компонент остаётся Inventory entity.

Если замена выполняется в рамках ремонта, штатный кандидат — процесс, который фиксирует движение/расход частей через Inventory; Maintenance Request остаётся записью ремонта Equipment.

Project Task создаётся только если замена сама является отдельной контролируемой работой.

Для Task могут использоваться relational Properties:

```text
Терминал
→ maintenance.equipment

Снятая комплектующая
→ stock.lot

Установленная комплектующая
→ stock.lot

Участок
→ stock.location
```

## 12. Текущий состав терминала — подтверждённый open gap

Требование:

```text
открыть конкретный терминал
→ увидеть структурированный список конкретных serial-комплектующих,
  установленных в нём сейчас
```

повторно проверено отдельно.

### 12.1. Почему не объявляем Packages решением

Inventory Package штатно предназначен для физического контейнера, группировки товаров, хранения и перемещения содержимого.

Технически туда можно поместить serial-компоненты, но терминал как инженерное изделие не принимается автоматически за warehouse package только ради обхода отсутствующей relation.

### 12.2. Почему не объявляем BoM решением

BoM описывает нормативный/производственный состав продукта, а не обязательно фактический текущий набор установленных serial-экземпляров после эксплуатационных замен.

### 12.3. Почему Repairs пока не master состава

Repair Order штатно умеет:

```text
Product to Repair
Lot / Serial
Parts: Add / Remove / Recycle
Product Moves
```

Это сильный кандидат для конкретного сценария ремонта/замены, но он описывает **операцию ремонта**, а не самостоятельную постоянно актуальную relation «терминал → текущие serial-компоненты».

Поэтому текущим решением является:

- вести все серийные компоненты в Inventory;
- сохранять их движения и события замены штатными операциями;
- не создавать ложный «текущий состав» через неподходящую модель;
- отдельно runtime-проверить, достаточно ли traceability / repair history для ежедневной работы.

Это **принятый gap текущего штатного baseline**, а не черновая гипотеза.

## 13. Project Tasks и предметные связи

Project остаётся управленческим слоем, а не реестром физических движений.

Базовые relational Properties:

```text
ТС                   → fleet.vehicle
Сотрудник            → hr.employee
Участок              → stock.location
Оборудование         → maintenance.equipment
Серийный объект      → stock.lot
Снятая комплектующая → stock.lot
Новая комплектующая  → stock.lot
```

Создаём только те Properties, которые реально нужны конкретному Project/process.

## 14. Как выбираем запись события

```text
Исправление ПЛ / сверка / запрос / аналитическая работа
→ Project Task

Следующее действие по существующей записи
→ Activity

Ремонт / профилактика терминала или рамки
→ Maintenance Request

Физическое перемещение терминала / рамки / серийной детали
→ Inventory Internal Transfer

История конкретного серийного экземпляра
→ Inventory Traceability

Штатный Fleet service / odometer / driver event
→ Fleet record
```

Главное правило:

> если профильное приложение Odoo уже имеет естественную запись события, Project Task не создаётся только ради копии этого события.

## 15. Что считаем «все чихи по субъекту» на штатном baseline

### ТС

```text
Fleet card
→ Drivers / Services / Contracts / Odometer
+ Project filter по ТС
```

### Сотрудник

```text
Employee card
→ HR data / Activities / assigned Equipment
+ Project filter по сотруднику
```

### Терминал / рамка

```text
Equipment card
→ Maintenance Requests
→ matched Serial
→ Inventory Traceability
+ Project filter по Equipment
```

### Участок

```text
Stock Location
→ Equipment
→ Inventory movements
+ Project filter по участку
```

### Серийная комплектующая

```text
Inventory Serial
→ current location
→ Traceability
+ Project filter по serial при необходимости
```

Отсутствие одной универсальной кнопки `Все события` принимается как штатный UX-компромисс текущего этапа.

## 16. Что сейчас НЕ является решением

```text
custom department.site
custom equipment.movement
custom component assignment
OCA
самописные smart buttons
прямая БД
Task вместо Inventory Transfer
Task вместо Maintenance Request
hr.work.location как универсальный операционный участок
maintenance.equipment для каждой серийной детали без maintenance lifecycle
Packages как искусственный реестр состава терминала
```

## 17. Runtime-проверки перед процессами

На стенде обязательно проверить:

1. создать 2–3 `stock.location` участка;
2. импортировать тестовые ТС и сотрудников;
3. создать terminal product с unique serial tracking;
4. создать terminal Equipment с тем же serial;
5. проверить matched Serial через `stock_maintenance`;
6. выполнить Internal Transfer терминала между участками;
7. проверить Traceability serial;
8. открыть Stock Location и Equipment list;
9. создать Maintenance Request по терминалу/рамке;
10. создать serialized component и выполнить его движение;
11. настроить Task relational Properties на Fleet / Employee / Location / Equipment / Serial;
12. проверить search/filter/grouping по каждому субъекту;
13. отдельно проверить реальную замену компонента и понять, достаточно ли штатной traceability без структурированного current composition.

## 18. Статус решения

На текущем этапе принято:

```text
Fleet         → ТС
Employees     → люди
Inventory     → участки, serials, физические движения
Maintenance   → эксплуатационное оборудование и ремонты
Project       → управляемая работа
Properties    → связи Project/Employees с предметными records там, где нет штатного schema field
```

Это **текущее решение**, а не кандидат.

Изменяется оно только после фактического runtime-результата или отдельного согласованного решения.

---

[← 21 — Штатный baseline](21-stock-only-baseline.md)