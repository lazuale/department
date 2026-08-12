# Аудит штатного «досье субъекта» в Odoo 19 Community

[Главная](../README.md) · [16 Реестр модулей](16-community-modules-context.md) · [17 Карта процессов](17-real-process-map.md) · [19 Master data](19-master-data.md)

---

Этот документ отвечает на один практический вопрос:

> **Насколько далеко можно дойти штатным Odoo 19 Community, если требуется открыть ТС, сотрудника, терминал, рамку или участок и увидеть всё существенное, что с этим субъектом происходило?**

Здесь **не проектируется custom module** и не принимается решение о доработке. Сначала фиксируется штатное поведение и компромисс.

## 1. Snapshot проверки

Проверка выполнена: **2026-08-12**.

Текущий head официальной Community-ветки на момент проверки:

```text
repository: odoo/odoo
branch:     19.0
commit:     2f0f8e5e00685129b5bbe954117bc9f80a568e88
```

Ранее `docs/16` был привязан к `cb731ca8720cd01a5719890cf6c5e140dc551546`. Между этими snapshot ветка ушла вперёд на два commit. В проверенном compare нет изменений в `addons/fleet`, `addons/maintenance`, `addons/hr_maintenance` или `addons/project`, поэтому выводы этого аудита по рассматриваемым моделям не конфликтуют с предыдущим snapshot. Сам `docs/16` в этой итерации не изменяется.

## 2. Что считаем полноценным досье

Для каждого субъекта проверяются пять уровней.

| Уровень | Что означает |
|---|---|
| **A — прямо из карточки** | на карточке есть штатный smart button / вкладка / история, открывающая связанные записи |
| **B — штатно за 1–2 перехода** | история доступна без кода, но не собрана непосредственно на карточке |
| **C — через поиск / фильтр** | данные можно найти штатно в другом приложении, указав субъект |
| **D — только частичная история** | штатная модель покрывает только один тип событий |
| **E — штатной связи нет** | требование не закрывается без изменения модели / разработки / внешней витрины |

Главный критерий будущего решения — не наличие поля как такового, а удобство сценария:

```text
открыть субъект
→ понять текущее состояние
→ увидеть историю
→ найти открытые и закрытые работы
→ перейти к первичному событию / ремонту / задаче
```

## 3. Общий вывод до runtime-пилота

Штатный Odoo уже умеет строить **сильные предметные досье внутри каждого приложения**:

```text
ТС          → Fleet history
Сотрудник   → Employees + назначенное Equipment
Оборудование→ Maintenance history
Serial/lot  → Inventory movement history
```

Но Project Task в штатном `project` не имеет универсальных полей:

```text
vehicle_id
employee_id
maintenance_equipment_id
work_location_id
```

`project.task.user_ids` — это исполнители `res.users`, а не предмет работы. Для произвольной предметной связи без кода Odoo предлагает relational Properties (`Many2one` / `Many2many`). Properties являются pseudo-fields, определяемыми родителем, например Project для Task.

Отсюда базовый компромисс штатного решения:

```text
из Task → открыть связанный субъект      = штатно возможно через relational Property
из списка Tasks → найти работы субъекта  = проверить runtime UX фильтра/поиска Properties
из субъекта → открыть все такие Tasks    = штатного reverse smart button не найдено
```

То есть **«найти всё по субъекту» и «открыть субъект и сразу получить всё» — не одно и то же требование**.

## 4. ТС — `fleet.vehicle`

### 4.1. Что штатно есть прямо в карточке

`fleet.vehicle` является самостоятельной карточкой ТС, наследует Chatter и Activities.

В официальном form view присутствуют штатные smart buttons:

```text
Drivers History
Contracts
Services
Odometer
```

Карточка также содержит, среди прочего:

```text
license_plate
VIN/SN
model / brand
state
current driver / future driver
location (text)
Properties
Chatter
Activities
```

### 4.2. Насколько это уже «все чихи»

Для **штатных событий Fleet** — хорошо:

- история водителей — **A**;
- сервисы — **A**;
- контракты — **A**;
- показания одометра — **A**;
- сообщения / Activities самой карточки — **A**.

Для **наших Project Tasks об исправлениях, сверках, запросах и т. п.** — иначе.

В проверенном стандартном Fleet form нет smart button `Tasks`, а в public Community source не найден штатный bridge `project_fleet`.

Relational Property на Task может указывать на `fleet.vehicle`, но сама по себе Property не добавляет обратный smart button на `fleet.vehicle`.

### 4.3. Штатный компромисс

Без кода возможна модель:

```text
Task
→ Property ТС = fleet.vehicle

поиск истории работы
→ Project / Tasks
→ фильтр по ТС
```

Оценка до runtime:

```text
Fleet-native dossier      = A
все Project Tasks по ТС   = C / runtime check
единая кнопка «все работы» на ТС = не подтверждена штатно
```

## 5. Сотрудник — `hr.employee`

### 5.1. Что штатно есть в карточке

`hr.employee` является полноценной карточкой сотрудника и наследует Chatter / Activities.

В зависимости от установленных HR-модулей в Employee концентрируются кадровые и организационные данные. При установленной связке Employees + Maintenance (`hr_maintenance`) на карточке сотрудника появляется штатный smart button **Equipment**.

Он открывает оборудование, назначенное сотруднику.

В `hr_maintenance` связь реальная ORM-связь:

```text
maintenance.equipment.employee_id → hr.employee
```

### 5.2. Что это даёт для нашего досье

- собственная карточка сотрудника — **A**;
- Chatter / Activities сотрудника — **A**;
- текущее закреплённое Equipment — **A**;
- переход к карточкам оборудования и их Maintenance — **A/B**.

Но наши операционные Tasks «про сотрудника» не равны Tasks, **назначенным сотруднику как исполнителю**.

В `project.task`:

```text
user_ids → res.users
```

Это Assignees, то есть исполнители. Использовать Assignee как признак «Task относится к Иванову как к предмету проверки» методически неверно.

Для предметной связи Task → Employee штатный no-code кандидат — relational Property `Many2one/Many2many → hr.employee`.

### 5.3. Штатный компромисс

```text
Employee-native dossier            = A/B
оборудование сотрудника             = A
операционные Tasks, где он субъект  = C / runtime check
единая кнопка «все работы по человеку» = не подтверждена штатно
```

## 6. Терминал Nobilis — кандидат `maintenance.equipment`

Здесь пока проверяется только штатный вариант; решение о модели не считается окончательным.

### 6.1. Что умеет Equipment штатно

`maintenance.equipment` предназначен для индивидуальных единиц оборудования и содержит:

```text
name
category
model
serial number
vendor
warranty
Properties
Chatter
Activities
Maintenance Requests
```

В Community source `serial_no` имеет unique constraint.

Из карточки Equipment штатно открываются Maintenance Requests; Maintenance хранит corrective/preventive repair workflow, даты, стадии, ответственного и историю.

### 6.2. Связь с сотрудником

`hr_maintenance` добавляет штатные поля:

```text
Assigned Employee → hr.employee
Assigned Department → hr.department
Used By
Assigned Date
```

и Employee smart button в обратную сторону.

### 6.3. Что с перемещениями

Базовый Maintenance сам по себе не является журналом физических перемещений между операционными участками.

В документации Equipment есть `Used in location`, но это рабочее поле текущего местонахождения, а не полноценный журнал движений.

Если подключён `stock_maintenance`, Equipment получает `stock.location`, а также механизм поиска `stock.lot` с таким же serial number. Это **не создаёт** автоматически историю состава терминала и замен комплектующих.

Inventory умеет полноценную историю stock moves по product + lot/serial + From/To location. Но это уже означает моделирование объекта/компонента как складской единицы и выполнение stock operations.

### 6.4. Оценка штатного компромисса

```text
карточка терминала как Equipment      = A
ремонты / Maintenance Requests        = A
Chatter / Activities                   = A
текущий пользователь / сотрудник      = A при hr_maintenance
текущее местоположение                = A/D
структурированная история перемещений = A/B только если идти в Inventory model
история замен серийных компонентов    = частично возможна через Inventory, требует отдельного runtime-моделирования
Project Tasks по терминалу            = C / runtime check через relational Property
```

Главный вопрос пилота: **приемлем ли Inventory как естественная модель эксплуатации терминалов и компонентов, или это уже слишком большой складской компромисс**.

## 7. Рамка измерения объёма — кандидат `maintenance.equipment`

Для рамки штатный Maintenance выглядит естественнее, чем для составного терминала, потому что подтверждены два события:

```text
перемещение между участками
ремонт
```

### 7.1. Что закрывается хорошо

```text
карточка рамки / serial        → Equipment
ремонт                          → Maintenance Request
ремонтная история               → smart button Maintenance
Chatter / Activities            → Equipment / Maintenance
```

Ремонты получаются **A**.

### 7.2. Что остаётся спорным

Перемещения:

- текстовое/current location в Maintenance — текущая точка, но не структурированная история;
- `stock.location + stock moves` — структурированная история, но рамка становится ещё и stock-tracked item.

Поэтому:

```text
ремонтное досье рамки           = A
история перемещений без Inventory = D
история перемещений с Inventory  = A/B
Project Tasks по рамке           = C / runtime check
```

## 8. Участок / площадка

Это самый важный нерешённый субъект.

### 8.1. `hr.work.location`

Штатная `hr.work.location` содержит по source:

```text
name
company
location_type = Home / Office / Other
work address
location number
active
```

В Employees она описывает, **где работает сотрудник**, включая обычное место работы по дням.

Плюсы:

- штатный справочник;
- Employees уже умеет на него ссылаться;
- смысл «место работы» частично совпадает с участком.

Компромисс:

- модель HR-ориентированная;
- нет штатного досье ТС, Equipment, Project Tasks и всех событий площадки;
- сама карточка Work Location не является общей операционной точкой входа.

### 8.2. `stock.location`

Inventory Location — место хранения/движения stock внутри складской модели.

Она отлично отвечает на вопросы:

```text
что сейчас находится здесь?
какие stock moves прошли через location?
какие serial/lot здесь были?
```

Но плохо представляет универсальный производственный участок, если тот одновременно нужен для:

```text
сотрудников
ТС
Tasks
оборудования вне stock model
операционной аналитики
```

### 8.3. Текущий вывод

На проверенных штатных моделях **нет одной очевидной Community-модели участка, которая без семантического компромисса становится общим досье для Employees + Fleet + Maintenance + Project**.

Это ещё не основание делать custom model.

Нужно runtime-сравнение минимум двух штатных вариантов:

```text
A. участок = hr.work.location
B. участок = stock.location
```

и, при необходимости, проверить использование обычного `res.partner`/адреса как нейтральной записи площадки. Только после сравнения решается, приемлем ли какой-либо вариант.

## 9. Chatter не является автоматическим агрегатором всей истории

`fleet.vehicle`, `hr.employee`, `maintenance.equipment`, `maintenance.request` и `project.task` поддерживают свои Chatter/Activities там, где соответствующий mixin подключён.

Но Chatter конкретной записи хранит коммуникацию **этой записи**.

Если Task ссылается Property на ТС, сообщения Task не становятся автоматически сообщениями Chatter автомобиля. Аналогично Maintenance Request остаётся отдельной записью, хотя Equipment даёт штатную навигацию к нему.

Следовательно, требование:

> «открыть субъект и увидеть в одной ленте вообще всё из всех связанных моделей»

**не подтверждается штатным Chatter**.

## 10. Что relational Properties реально решают

Odoo 19 Properties поддерживают:

```text
Many2one → запись другой model
Many2many → несколько записей другой model
Domain → ограничение выбора
```

Для Task это позволяет штатно и без schema-разработки сделать связи:

```text
Task → ТС
Task → Сотрудник
Task → Equipment
Task → Work Location / Stock Location, если выбрана модель участка
```

Но Property остаётся pseudo-field, определённым Project.

Поэтому Properties рассматриваются как **первый no-code способ проверить предметные связи**, а не как заранее признанная полноценная замена обычной ORM relation.

На стенде нужно отдельно проверить:

- поиск Task по relational Property;
- custom filter;
- group by;
- отображение в List;
- открытие target record из Task;
- удобство работы при нескольких Projects;
- можно ли практически за приемлемое число действий получить «все Tasks по конкретному субъекту».

## 11. Матрица текущего компромисса

| Субъект | Собственная штатная карточка | Своя предметная история | Все наши Tasks прямо из карточки | Штатный вариант без кода |
|---|---|---|---|---|
| **ТС** | сильная Fleet | сильная по Fleet-событиям | **нет подтверждения** | Task Property → Fleet + поиск Tasks |
| **Сотрудник** | сильная Employees | сильная по HR/Equipment | **нет подтверждения** | Task Property → Employee + поиск Tasks |
| **Терминал** | сильная Maintenance Equipment | сильная по ремонтам; движения спорно | **нет подтверждения** | Equipment + Maintenance; Inventory проверить отдельно; Task Property |
| **Рамка** | сильная Maintenance Equipment | ремонт — сильный; движения спорно | **нет подтверждения** | Equipment + Maintenance; Inventory для movements; Task Property |
| **Участок** | подходящая универсальная карточка не подтверждена | зависит от выбранной модели | **нет** | сравнить Work Location / Stock Location / нейтральный address record |

## 12. Что проверяем на стенде до любого custom

Создать минимальный тестовый набор:

```text
2 ТС
2 сотрудника
2 участка-кандидата
1 терминал
2 серийные комплектующие
1 рамка
5–10 Tasks разных типов
2 Maintenance Requests
несколько Activities
```

### Проверка A — ТС

1. Открыть `fleet.vehicle`.
2. Проверить все стандартные smart buttons.
3. Создать Tasks с Property → ТС.
4. Закрыть часть Tasks.
5. Попытаться получить все открытые + закрытые Tasks этой ТС штатным UI.
6. Зафиксировать путь в кликах и пригодность фильтра для ежедневной работы.

### Проверка B — сотрудник

1. Открыть Employee.
2. Проверить Equipment smart button.
3. Создать Tasks, где сотрудник — **subject**, а не Assignee.
4. Найти все Tasks по relational Property.
5. Проверить разграничение доступа к Employee records.

### Проверка C — терминал

1. Создать Equipment с serial.
2. Создать Maintenance Requests.
3. Проверить, что ремонтная история читается прямо из карточки.
4. Проверить current location без Inventory.
5. Отдельно на копии данных попробовать serial/lot + stock locations + internal transfers.
6. Оценить, не превращает ли Inventory простой эксплуатационный учёт в лишний складской workflow.

### Проверка D — рамка

То же, но без состава комплектующих:

```text
Equipment
→ перемещение
→ ремонт
→ перемещение
```

Сравнить простоту Maintenance-only и Maintenance+Inventory.

### Проверка E — участок

Параллельно создать один и тот же реальный участок как:

```text
hr.work.location
stock.location
```

Проверить:

- привязку сотрудников;
- привязку/поиск Tasks;
- привязку Equipment;
- возможность увидеть, что находится на участке;
- историческую аналитику;
- количество лишних складских/HR понятий.

## 13. Как принимаем решение после пилота

Для каждого субъекта заполняется итог:

```text
Полностью штатно
или
Штатно с приемлемым компромиссом
или
Штатно, но неудобство критично
или
Требование отсутствует в Community
```

Custom рассматривается **только** для конкретного подтверждённого неудобства.

Пример корректной формулировки gap:

```text
НЕ: «нам нужен свой модуль субъектов»

А:
«при штатной relation через Property все Tasks по ТС доступны только через
ручной поиск в Project; из карточки fleet.vehicle обратной навигации нет,
а это не проходит согласованный критерий рабочего UX»
```

После этого уже можно выбирать минимальное исправление конкретного gap.

## 14. Проверенные официальные источники

Official Odoo 19 User Docs:

- Employees: `https://www.odoo.com/documentation/19.0/applications/hr/employees.html`
- Employee Equipment: `https://www.odoo.com/documentation/19.0/applications/hr/employees/equipment.html`
- New Employee / Work Locations: `https://www.odoo.com/documentation/19.0/applications/hr/employees/new_employee.html`
- Maintenance Equipment: `https://www.odoo.com/documentation/19.0/applications/inventory_and_mrp/maintenance/add_new_equipment.html`
- Inventory Locations: `https://www.odoo.com/documentation/19.0/applications/inventory_and_mrp/inventory/warehouses_storage/inventory_management/use_locations.html`
- Moves History: `https://www.odoo.com/documentation/19.0/applications/inventory_and_mrp/inventory/warehouses_storage/reporting/moves_history.html`
- Property Fields: `https://www.odoo.com/documentation/19.0/applications/essentials/property_fields.html`

Official Community source checked on branch `19.0`:

- `addons/fleet/models/fleet_vehicle.py`
- `addons/fleet/views/fleet_vehicle_views.xml`
- `addons/hr/models/hr_employee.py`
- `addons/hr/models/hr_work_location.py`
- `addons/hr_maintenance/models/equipment.py`
- `addons/hr_maintenance/views/hr_views.xml`
- `addons/maintenance/models/maintenance.py`
- `addons/maintenance/views/maintenance_views.xml`
- `addons/stock_maintenance/models/maintenance.py`
- `addons/project/models/project_task.py`

---

[← 19 — Master data](19-master-data.md) · [17 — Карта процессов](17-real-process-map.md)
