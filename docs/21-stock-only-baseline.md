# Штатный baseline Odoo 19 Community

[Главная](../README.md) · [16 Реестр модулей](16-community-modules-context.md) · [20 Аудит досье субъекта](20-subject-dossier-stock-audit.md) · [22 Текущая штатная модель](22-stock-master-data-model.md)

---

Этот документ фиксирует **текущее принятое ограничение проекта**:

> На данном этапе используем только штатные возможности официального Odoo 19 Community.

Цель этапа — собрать максимально рабочий контур на стандартных моделях, официальных Community-модулях и штатной конфигурации, а фактические ограничения принимать как gaps, а не закрывать самодельным кодом.

## 1. Приоритет этого решения

Это решение имеет приоритет над более ранними гипотезами о custom-моделях и собственном связующем модуле.

В частности, идеи вида:

```text
department.site
equipment.movement
equipment.component.assignment
custom relations / smart buttons
```

**не являются решениями текущего этапа**.

Если штатного механизма недостаточно, фиксируется gap. Решение о выходе за штатный baseline принимается отдельно.

## 2. Что считается штатным вариантом

Допускаются:

- официальные модули публичной ветки `odoo/odoo:19.0`;
- стандартные Odoo models и штатные relations;
- стандартные Properties, включая `Many2one` / `Many2many`;
- стандартные Tags;
- штатные поля и настройки приложений;
- стандартные List / Kanban / Form / Search / Pivot / Graph / Activity views;
- фильтры, группировки, Favorites / Shared Views, если доступны штатно;
- Chatter и Activities;
- штатные механизмы Project;
- стандартный CSV/XLSX import/export;
- штатные Groups / Access Rights / Record Rules / visibility;
- официальные Community bridge-модули;
- штатные Inventory operations, locations, lots/serials и traceability;
- Automation Rules только после отдельной проверки конкретного сценария.

## 3. Что сейчас не принимается

На текущем этапе не используем как решение:

- собственные Odoo-модули;
- собственные Python models;
- собственные ORM fields;
- собственные XML views / smart buttons;
- OCA и иные сторонние модули;
- прямую доработку core Odoo;
- прямую запись в PostgreSQL;
- Server Action / Automation с произвольным Python-кодом как способ достроить предметную модель;
- попытку имитировать отсутствующую модель скрытым кодом.

## 4. Текущий набор приложений для предметного контура

Для текущего решения используются:

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

устанавливаются автоматически при наличии соответствующих зависимостей.

`stock_maintenance` является частью текущего решения: он добавляет Equipment штатную relation на `stock.location`, переход к совпадающему Inventory serial и обратный просмотр Equipment из Stock Location.

Важно: это **не означает автоматическую синхронизацию фактического местонахождения Equipment со stock moves**. Поле `maintenance.equipment.location_id` остаётся отдельным штатным полем. Источником истории физических перемещений является Inventory.

## 5. Текущий baseline по субъектам

### ТС

```text
fleet.vehicle
```

Fleet остаётся master record ТС.

Штатные Fleet history: водители, сервисы, контракты, одометр, Chatter/Activities.

Операционная связь Task → ТС выполняется relational Property на `fleet.vehicle`.

`fleet.vehicle.location` — текстовое поле, поэтому структурированная текущая relation ТС → операционный участок штатно не принимается как source of truth.

### Сотрудник

```text
hr.employee
```

Employees остаётся master record человека.

Операционная связь Task → сотрудник выполняется relational Property на `hr.employee`.

Если нужен текущий **операционный участок** сотрудника как единый сквозной справочник, используется Employee Property → `stock.location`. HR Work Location остаётся штатным HR-понятием и не назначается master record операционного участка.

### Участок / площадка

Текущий единый операционный справочник:

```text
stock.location
usage = internal
```

Выбор основан на фактической предметной роли участка:

- на участке физически находятся терминалы / рамки / комплектующие;
- между участками происходят реальные перемещения;
- Inventory штатно хранит From / To / Date / Product / Lot/Serial;
- `stock_maintenance` штатно связывает Equipment с Stock Location.

Task связывается с участком relational Property → `stock.location`.

`hr.work.location` не удаляется из Odoo и может использоваться по своему штатному HR-смыслу, но **не является текущим master справочником операционных участков**.

### Терминал Nobilis

Терминал имеет две штатные проекции одного физического объекта:

```text
Maintenance Equipment
→ эксплуатация / обслуживание / Maintenance Requests

Inventory Product + unique Serial
→ физический экземпляр / местонахождение / перемещения / traceability
```

Для связи используется один и тот же фактический serial number. Официальный `stock_maintenance` умеет открыть совпадающий Inventory serial из Equipment.

### Рамка измерения объёма

Та же схема:

```text
Maintenance Equipment
+ Inventory unique Serial
```

Ремонт → `maintenance.request`.

Перемещение → штатный Inventory Internal Transfer.

### Серийные комплектующие терминала

Базовая штатная модель:

```text
product.product
+ stock.lot / unique Serial
```

Комплектующая не создаётся как Maintenance Equipment только потому, что у неё есть serial.

Отдельный Equipment record нужен только если сама комплектующая является самостоятельным обслуживаемым/ремонтируемым объектом.

## 6. Событие выбирается по его предметному смыслу

```text
управляемая работа / запрос / сверка / исправление
→ Project Task

ремонт / профилактика Equipment
→ Maintenance Request

физическое перемещение серийного объекта
→ Inventory Internal Transfer / stock moves

история конкретного serial
→ Inventory Traceability
```

Не создаём Task только ради дублирования уже существующей штатной Inventory или Maintenance записи.

Task может существовать **вокруг** такого события, если есть самостоятельный контролируемый результат: организовать перемещение, получить согласование, устранить расхождение и т. п.

## 7. Требование «все чихи по субъекту»

На текущем этапе оно трактуется так:

```text
можно ли штатными средствами воспроизводимо найти профильную историю и связанные работы субъекта?
```

Принимаемый путь:

```text
ТС
→ Fleet history
→ Project search/filter по ТС

сотрудник
→ Employees
→ Equipment при hr_maintenance
→ Project search/filter по сотруднику

терминал / рамка
→ Maintenance Equipment / Maintenance Requests
→ matched Inventory Serial / Traceability
→ Inventory movements
→ Project search/filter по Equipment

участок
→ Stock Location / Equipment
→ Inventory moves
→ Project search/filter по участку
```

Отсутствие одной универсальной smart button `Все события` считается допустимым компромиссом штатного baseline, пока поиск остаётся однозначным и воспроизводимым.

## 8. Подтверждённый открытый gap: текущий состав терминала

Штатно подтверждены:

- индивидуальные serial-комплектующие;
- их перемещения;
- ремонт терминала;
- ремонтные/операционные события.

Но пока **не принят** механизм, который семантически чисто хранит relation:

```text
конкретный терминал
→ конкретный набор установленных сейчас serial-комплектующих
```

Packages предназначены прежде всего для физических контейнеров/группировки stock; BoM — для состава изделия по типам компонентов; Repair Orders — для процесса ремонта и add/remove/recycle частей. Ни один из них пока не объявляется текущим master реестром установленного состава.

Поэтому этот вопрос считается **штатным gap для отдельного runtime-теста**, а не поводом заранее строить custom model.

## 9. Как фиксируем gaps

```text
Требование
→ какой штатный путь проверен
→ где именно возникает ограничение
→ сколько действий требуется пользователю
→ что невозможно получить
→ насколько это влияет на работу / контроль / аналитику
```

Не писать `нужен custom module`, пока отдельно не принято решение выйти за пределы штатного baseline.

## 10. Следующий порядок работы

```text
загрузить штатные справочники
→ настроить Stock Locations и serial tracking
→ связать Maintenance + Inventory
→ проверить реальные движения и ремонты
→ настроить relational Properties в Project
→ прогнать реальные процессы
→ собрать только фактические gaps
```

До отдельного решения об изменении baseline действует принцип:

> **Максимально используем штатную предметную модель Odoo 19 Community и не подменяем профильные Inventory / Maintenance события Project Tasks.**

---

[← 20 — Аудит штатного досье](20-subject-dossier-stock-audit.md) · [22 — Текущая штатная модель →](22-stock-master-data-model.md)