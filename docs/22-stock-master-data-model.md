# Штатная модель справочников и субъектов Odoo 19 Community

[Главная](../README.md) · [16 Реестр модулей](16-community-modules-context.md) · [20 Аудит досье](20-subject-dossier-stock-audit.md) · [21 Штатный baseline](21-stock-only-baseline.md)

---

Этот документ фиксирует **принятую на текущем этапе штатную модель справочников и субъектов**.

Граница решения:

> используем только официальные возможности Odoo 19 Community; custom models, custom fields, OCA и собственный связующий модуль не используются.

Цель — получить единые предметные записи, к которым можно привязывать Tasks и по которым можно воспроизводимо искать историю работы.

## 1. Базовая схема

```text
ТС                    → Fleet / fleet.vehicle
Сотрудники             → Employees / hr.employee
Участки                → Employees / hr.work.location
Терминалы Nobilis      → Maintenance / maintenance.equipment
Комплектующие терминала→ Maintenance / maintenance.equipment
Рамки измерения объёма → Maintenance / maintenance.equipment

Task → relational Properties → соответствующие штатные records
```

Отдельная универсальная сущность `Субъект` не создаётся.

Одна Task может одновременно ссылаться на несколько предметов, например:

```text
ТС + сотрудник + участок
терминал + участок
рамка + участок
терминал + снятая комплектующая + установленная комплектующая
```

## 2. Минимальный набор приложений

Для master-data baseline активируем:

```text
Project      project
Employees    hr
Fleet        fleet
Maintenance  maintenance
Contacts     contacts
Discuss      mail
```

При одновременной установке Employees и Maintenance официальный bridge:

```text
hr_maintenance
```

устанавливается автоматически (`auto_install=True`). Он связывает Equipment с сотрудником/подразделением и добавляет Equipment smart button на карточку сотрудника.

Inventory (`stock`) **не является обязательным** для master-data baseline. Он рассматривается отдельно, только если будет принято вести полноценные складские перемещения, lot/serial traceability и stock locations.

## 3. Главное правило импорта и идентичности

Для всех загружаемых справочников используем штатный Odoo `External ID`.

Причина:

- повторный импорт обновляет существующие записи, а не плодит дубли;
- relations можно импортировать через `<Field>/External ID`;
- ключ не зависит от отображаемого имени;
- можно использовать исходный устойчивый ID из 1С / реестра / другой системы.

Рекомендуемый формат:

```text
vehicle_<source_id>
employee_<source_id>
site_<source_id>
equipment_<source_id>
```

Не использовать имя, ФИО, госномер или serial как единственный интеграционный ключ, если источник уже имеет стабильный ID.

## 4. ТС

### 4.1. Master record

```text
fleet.vehicle
```

Это единственная карточка ТС в Odoo.

Не создаём второй справочник ТС в Project Properties, Contacts или Tags.

### 4.2. Штатные данные

Минимально используем:

```text
External ID   — устойчивый импортный ключ
Model         — штатно required
License Plate — основной пользовательский поиск
VIN/SN        — если доступен
State         — только если реально используется
Location      — текущее местонахождение текстом, если требуется
Tags          — только для настоящей дополнительной классификации
```

Не превращаем Fleet в систему путевых листов.

### 4.3. Участок ТС

У Fleet есть штатное поле:

```text
location → Char
```

Поэтому без кода нет единой строгой relation:

```text
fleet.vehicle → hr.work.location
```

Это **принятый компромисс штатного baseline**.

Если текущий участок ТС нужен прямо в Fleet, используем `Location` с согласованными наименованиями участков.

Для аналитики работы участок фиксируется **отдельно в Task** через relational Property `Участок → hr.work.location`. Историческая Task не должна зависеть от текущего текста `fleet.vehicle.location`.

### 4.4. Связь с Task

В Project создаётся relational Property:

```text
ТС
Type: Many2one
Model: fleet.vehicle
```

Если конкретная работа одновременно относится к нескольким ТС, допускается `Many2many`, но baseline — `Many2one`, пока реальные процессы не докажут обратное.

### 4.5. Что видно по ТС штатно

Из карточки Fleet штатно доступны собственные истории Fleet:

```text
Drivers History
Contracts
Services
Odometer
Chatter
Activities
```

Операционные Project Tasks ищутся в Project фильтром по Property `ТС`.

Отсутствие обратной smart button `Все Tasks` на карточке ТС принято как штатное ограничение этапа.

## 5. Сотрудники

### 5.1. Master record

```text
hr.employee
```

ФИО в Task текстом не хранится как замена Employee record.

### 5.2. Минимальные данные

```text
External ID
Name
Company
Work Location
Work Email / Work Phone — только если реально нужны
Job / Department — только если реально используются в системе
Employee Properties — для дополнительных рабочих атрибутов
```

Табельный номер, если он нужен пользователю и в штатных полях нет подходящего семантического поля, допускается хранить как Employee Property `Табельный номер`.

При этом:

- `External ID` остаётся интеграционным ключом;
- Property не считается database unique constraint;
- ФИО не используется для обновления записей при импорте.

### 5.3. Участок сотрудника

Используем штатную relation:

```text
hr.version.work_location_id
→ hr.work.location
```

Это нативная связь Employees.

### 5.4. Связь с Task

```text
Сотрудник
Type: Many2one
Model: hr.employee
```

Это **предмет работы**, а не Assignee.

Не использовать `Task.Assignees` для значения «по какому человеку выполняется проверка/корректировка».

### 5.5. Что видно по сотруднику штатно

Карточка сотрудника даёт собственные Employees-данные, Chatter/Activities и при `hr_maintenance` — Equipment smart button.

Операционные Tasks по сотруднику ищутся в Project через Property `Сотрудник`.

## 6. Участки / площадки

### 6.1. Принятый штатный справочник

На текущем этапе используем:

```text
hr.work.location
```

Причина выбора:

- это самостоятельный штатный справочник Odoo;
- он имеет `name`, `active`, `company`, `address`, `location_number`;
- Employee штатно имеет настоящую Many2one relation на Work Location;
- Property `Many2one` может ссылаться на запись другой модели, поэтому Work Location можно использовать в Tasks и Equipment без своего поля.

### 6.2. Как заводим производственный участок

Для каждого реального участка создаётся отдельная Work Location.

Рекомендуемая схема:

```text
Work Location: <наименование участка>
Location Type: Other
Location Number: <устойчивый короткий код>, если используется
Work Address: соответствующий штатный res.partner address
Active: True
```

Если точный почтовый адрес для производственной площадки не важен, создаётся минимальная штатная запись Work Address, достаточная для обязательного поля Odoo. Не выдумываем фиктивные юридические реквизиты сверх того, что требует форма.

### 6.3. Где используется участок

```text
Employee.Work Location → hr.work.location       — штатное поле
Task.Property Участок → hr.work.location        — relational Property
Equipment.Property Участок → hr.work.location   — relational Property
```

Для Fleet прямой relation нет; см. раздел 4.3.

### 6.4. Связь с Task

В Project:

```text
Участок
Type: Many2one
Model: hr.work.location
```

Для перемещений дополнительно могут использоваться:

```text
Откуда → hr.work.location
Куда   → hr.work.location
```

Это штатные Properties, не custom fields.

### 6.5. Ограничение

`hr.work.location` остаётся HR-моделью и не становится полноценным общим «досье площадки».

Нам важнее, что она даёт **один контролируемый список значений** для Employees, Task Properties и Equipment Properties.

Все работы конкретного участка ищутся через Project filter/grouping по Property `Участок`.

## 7. Терминалы Nobilis

### 7.1. Master record

```text
maintenance.equipment
Category = Терминал Nobilis
```

Каждый физический терминал — отдельный Equipment record.

Минимум:

```text
External ID
Equipment Name
Equipment Category = Терминал Nobilis
Serial Number
Model — при наличии
Vendor — при необходимости
Warranty — при необходимости
Properties
Chatter / Activities
```

### 7.2. Текущий участок терминала

На Equipment Category `Терминал Nobilis` создаём Property:

```text
Участок
Type: Many2one
Model: hr.work.location
```

Это отвечает на вопрос:

> где терминал находится сейчас?

Изменение текущего участка само по себе не считается полноценным журналом перемещений.

### 7.3. Ремонт

Используем штатно:

```text
maintenance.request
→ equipment_id = терминал
```

Maintenance Request — источник ремонтной истории терминала.

Не создаём параллельную Project Task только ради дублирования ремонта.

Task нужна только если вокруг ремонта существует отдельная операционная работа, не совпадающая с самим Maintenance Request.

### 7.4. Project Tasks по терминалу

В Project:

```text
Оборудование
Type: Many2one
Model: maintenance.equipment
```

При необходимости Domain ограничивает выбор нужными Equipment Categories.

Так фиксируются операционные работы, которые не являются Maintenance Request:

```text
перемещение
проверка
замена комплектующей
сверка / разбор
другая контролируемая работа
```

## 8. Комплектующие терминалов

### 8.1. Когда комплектующая становится отдельным Equipment

Если у комплектующей есть собственный серийный номер и её нужно различать физически, она заводится как:

```text
maintenance.equipment
```

с отдельной Equipment Category, например:

```text
Nobilis — <тип компонента>
```

Одна физическая комплектующая = один Equipment record.

### 8.2. Текущий состав терминала без custom

На категории `Терминал Nobilis` можно создать relational Properties по реальным типам комплектующих:

```text
<Компонент A> → Many2one maintenance.equipment
<Компонент B> → Many2one maintenance.equipment
<Компонент C> → Many2one maintenance.equipment
```

Для каждой Property задаётся Domain по соответствующей Equipment Category.

Это штатно даёт **текущий состав терминала**.

Если компонентов одного типа может быть несколько, используется `Many2many`.

### 8.3. История замен

Properties показывают текущий состав, но baseline не считает их отдельным структурированным журналом замен.

Историю замены фиксируем как реальную Project Task, если замена является контролируемой работой:

```text
Тип работы = Замена комплектующей
Оборудование = терминал
Снятая комплектующая = maintenance.equipment
Установленная комплектующая = maintenance.equipment
Участок = hr.work.location
```

`Снятая комплектующая` и `Установленная комплектующая` — обычные relational Task Properties.

После выполнения Task текущая Property терминала обновляется вручную на фактический компонент.

Это штатный компромисс:

```text
текущий состав → Equipment Properties
история выполненных замен → Project Tasks
```

Без скрытой автоматизации и без собственного регистра.

## 9. Рамки измерения объёма

### 9.1. Master record

```text
maintenance.equipment
Category = Рамка измерения объёма
```

Минимум:

```text
External ID
Equipment Name
Serial Number / устойчивый номер
Category
Model — если нужен
Properties
Chatter / Activities
```

### 9.2. Текущий участок

На Equipment Category создаём:

```text
Участок
Type: Many2one
Model: hr.work.location
```

### 9.3. Ремонт

```text
maintenance.request
```

используется штатно для ремонта рамки.

### 9.4. Перемещение

Если перемещение является контролируемой операцией, используем Project Task:

```text
Тип работы = Перемещение оборудования
Оборудование = рамка
Откуда = hr.work.location
Куда = hr.work.location
```

После фактического перемещения обновляется текущая Equipment Property `Участок`.

Таким образом:

```text
текущее местонахождение → Equipment Property
история перемещений      → Project Tasks
ремонтная история        → Maintenance Requests
```

## 10. Набор субъектных Properties в Project

Это не отдельные models, а штатные Properties задачи.

Базовый словарь:

| Property | Type | Model / значения | Назначение |
|---|---|---|---|
| **ТС** | Many2one | `fleet.vehicle` | работа по конкретному ТС |
| **Сотрудник** | Many2one | `hr.employee` | сотрудник как предмет работы |
| **Участок** | Many2one | `hr.work.location` | контекст работы |
| **Оборудование** | Many2one | `maintenance.equipment` | терминал / рамка / компонент |
| **Откуда** | Many2one | `hr.work.location` | только для работ с перемещением |
| **Куда** | Many2one | `hr.work.location` | только для работ с перемещением |
| **Снятая комплектующая** | Many2one | `maintenance.equipment` | только для замены |
| **Установленная комплектующая** | Many2one | `maintenance.equipment` | только для замены |

Properties `Откуда`, `Куда`, `Снятая комплектующая`, `Установленная комплектующая` не обязаны присутствовать в каждом Project. Их добавляем только туда, где соответствующий процесс реально живёт.

Не создаём Property `Субъект` как текстовое универсальное поле.

## 11. Что является текущим состоянием, а что историей

Критически важно не смешивать эти два слоя.

### Текущее состояние

```text
Employee.Work Location
Fleet.Location
Equipment.Property Участок
Terminal Properties текущих комплектующих
```

### История

```text
Fleet native logs
Maintenance Requests
Project Tasks
Chatter / Activities
```

Правило:

> изменение текущего состояния не должно уничтожать смысл старой Task.

Поэтому Task, относящаяся к участку A, сохраняет Property `Участок = A`, даже если потом сотрудник, ТС или оборудование переместилось на участок B.

## 12. «Все чихи по субъекту» в штатном baseline

Принимаем не единую custom-кнопку, а воспроизводимый штатный маршрут.

### ТС

```text
Fleet record
→ native Fleet history
+
Project → filter ТС = <vehicle>
```

### Сотрудник

```text
Employee record
→ штатные Employee / Equipment данные
+
Project → filter Сотрудник = <employee>
```

### Терминал / рамка

```text
Equipment record
→ Maintenance history
→ текущий участок / текущий состав через Properties
+
Project → filter Оборудование = <equipment>
```

### Участок

```text
Work Location record
+
Project → filter/group Участок = <site>
```

Требование считается закрытым штатно, если пользователь может найти нужную историю по выбранному record без ручного сопоставления строк и без знания внутренних ID.

## 13. Порядок первичной загрузки

Рекомендуемый порядок:

```text
1. Contacts / Work Addresses для участков
2. Work Locations
3. Employees
4. Fleet brands/models
5. Vehicles
6. Maintenance Equipment Categories
7. Terminals Nobilis
8. Terminal components
9. Volume-measurement frames
10. Project Properties
11. Тестовые Tasks
```

Каждый массовый справочник сначала импортируется на тестовом наборе 5–10 строк.

После проверки выполняется полный импорт.

## 14. Правила CSV/XLSX

Для массовой загрузки:

- используем штатный Import Records;
- сначала выполняем `Test`;
- сохраняем `External ID`;
- повторные обновления выполняем с тем же `External ID`;
- relations по возможности связываем через `<Field>/External ID`;
- перед большим импортом экспортируем 1–2 тестовые записи из нужной модели, чтобы получить фактические имена импортируемых полей.

Не обновляем Odoo прямой записью в PostgreSQL.

## 15. Архивирование

Справочник не очищается удалением исторически использованных records.

Используем штатный `Active / Archived`, где он доступен.

Принцип:

```text
объект больше не используется
→ archive
→ старые relations сохраняются
```

Не переиспользовать старую карточку физического терминала/рамки/ТС под другой физический объект.

## 16. Принятые компромиссы

В рамках stock-only baseline принимаются следующие ограничения:

1. Fleet не имеет штатной relation `vehicle → hr.work.location`; текущее местонахождение ТС остаётся `Location` text, а исторический участок работы фиксируется в Task.
2. Нет единой reverse smart button `Все Project Tasks` на Fleet / Employee / Equipment.
3. `hr.work.location` используется как единый штатный справочник участков, хотя сама модель остаётся частью HR.
4. Текущий состав терминала через Equipment Properties не является отдельным журналом замен; история замен ведётся Tasks.
5. Текущее местонахождение Equipment через Property не является отдельным журналом перемещений; история перемещений ведётся Tasks.
6. Properties не заменяют строгие schema fields и database constraints.
7. Если один и тот же набор Properties нужен в нескольких Projects, его придётся поддерживать в каждом Project штатными средствами.

Эти пункты сейчас **не являются основанием для разработки**.

## 17. Что пока не включаем

Не включаем в baseline:

```text
Inventory / stock
custom module
OCA
собственные models
собственные smart buttons
собственные ORM relations
скрытую автоматизацию для синхронизации субъектов
```

Inventory возвращается на рассмотрение только если фактический процесс требует:

```text
stock quantities
warehouses
internal transfers
lot/serial traceability
складских документов
```

## 18. Что нужно проверить на стенде

До того как на этой модели строится вся методика, обязательно проверить руками:

1. CSV/XLSX import Work Locations и Employees с External ID.
2. Task Property `Many2one → fleet.vehicle`.
3. Task Property `Many2one → hr.employee` с реальными правами обычного пользователя.
4. Task Property `Many2one → hr.work.location`.
5. Task Property `Many2one → maintenance.equipment`.
6. Filter / group Tasks по каждому relational Property.
7. Equipment Property `Many2one → hr.work.location`.
8. Equipment Property `Many2one → maintenance.equipment` с Domain по категории комплектующей.
9. Maintenance smart button с историей ремонта терминала/рамки.
10. Employee Equipment smart button после auto-install `hr_maintenance`.
11. Повторный импорт справочников с теми же External IDs без дублей.
12. Архивирование субъекта и поведение старых Task relations.

Результат проверки фиксируется как:

```text
PASS
PASS с приемлемым компромиссом
FAIL / gap
```

До runtime-проверки не добавлять custom-код для улучшения UX.

## 19. Источники проверки

Официальная документация Odoo 19.0:

- `Applications → Essentials → Property fields`;
- `Applications → Essentials → Export and import data`;
- `Applications → Human Resources → Employees → New employees`;
- `Applications → Human Resources → Employees → Equipment`;
- `Applications → Inventory and MRP → Maintenance`;
- `Applications → Inventory and MRP → Maintenance → Add new equipment`;
- `Applications → Inventory and MRP → Maintenance → Maintenance requests`.

Официальный Community source `odoo/odoo:19.0`:

```text
addons/fleet/models/fleet_vehicle.py
addons/fleet/views/fleet_vehicle_views.xml
addons/hr/models/hr_work_location.py
addons/hr/models/hr_version.py
addons/maintenance/models/maintenance.py
addons/maintenance/views/maintenance_views.xml
addons/hr_maintenance/__manifest__.py
addons/hr_maintenance/models/equipment.py
addons/hr_maintenance/views/hr_views.xml
addons/project/models/project_task.py
```

Текущий source snapshot для этой итерации:

```text
2f0f8e5e00685129b5bbe954117bc9f80a568e88
```

---

**Статус:** принято как stock-only master-data baseline текущего пилота. Изменять границу только отдельным решением после фактической runtime-проверки.