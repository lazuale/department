# Управление работой в Odoo 19 Community

Практическая методика использования **Odoo 19 Community** для управления обязательствами, предметными данными, контролем и аналитикой.

Методика адаптируется под штатную модель Odoo, а не пытается воспроизвести заранее заданную организационную схему.

**[Граница возможностей Community →](docs/00-odoo19-community.md)**  
**[Модель управления →](docs/01-methodology.md)**  
**[Рабочие сценарии →](docs/02-scripts.md)**  
**[Контроль и аналитика →](docs/03-control.md)**  
**[Click-only пилот →](docs/06-workspace.md)**  
**[Реестр master data →](docs/19-master-data.md)**  
**[Master data: люди →](docs/24-master-data-people.md)**  
**[Master data: техника и оборудование →](docs/25-master-data-assets.md)**  
**[Связь Task с master data →](docs/26-task-master-data-relations.md)**

## Три слоя

### 1. Технические возможности Odoo

Что реально умеет Odoo 19 Community:

- Projects / Tasks;
- Stages / States;
- Assignees;
- Deadline / Priority;
- Activities;
- Dependencies;
- Subtasks;
- Task Templates;
- Recurring Tasks;
- Chatter;
- Properties, включая Many2one/Many2many;
- List / Kanban / Calendar / Activity / Pivot / Graph;
- Shared Views;
- Task Analysis;
- My Dashboard;
- Groups / Access Rights / Record Rules;
- Import / Export;
- Automation / Webhooks;
- JSON-2 API.

### 2. Нейтральная методика

Из возможностей Odoo выводятся только общие правила:

> **Task — отдельно контролируемый результат, а не универсальная карточка объекта.**

```text
предметная сущность
→ штатная Odoo-модель, если она подходит
→ связь с Task при необходимости
→ выполнение и контроль
```

Согласованные предметные модели текущего контура:

```text
Люди / сотрудники
→ res.partner

Доступ в Odoo
→ res.users, связанный с тем же res.partner

Самосвалы + строительная техника + малая механизация
→ fleet.vehicle

Сканирующие рамки объёма
→ maintenance.equipment

Терминал Nobilis как комплект
→ stock.package

Тип комплектующей Nobilis
→ product.product

Конкретная комплектующая / Serial Number
→ stock.lot

Местонахождение и движение Nobilis
→ stock.location / stock.quant / stock.move / stock.move.line
```

Employees / HR не входит в текущий baseline только ради справочника сотрудников.

Подробные решения ведутся в [реестре master data](docs/19-master-data.md). Технические исследования не считаются принятыми решениями автоматически.

Task хранит работу:

```text
результат
Assignees
Deadline
Stage / State
Priority
Activities
Dependencies
Properties
Chatter
```

### 3. Локальные правила процесса

Методика **не задаёт заранее**:

- кто назначает работу;
- кто имеет право закрывать или отменять;
- нужен ли отдельный контроль результата;
- один или несколько Assignees используются;
- какие именно Stages нужны;
- какая схема Priority применяется;
- используется ли общая очередь;
- есть ли смены и какой у них график;
- кто и когда меняет Deadline.

Эти решения определяются конкретным процессом и затем настраиваются средствами Odoo.

## Связь Task с предметными объектами

Источник истины остаётся в предметной модели. Task хранит только связь с объектом, когда она нужна конкретной работе.

Штатные исполнители:

```text
project.task.user_ids
→ res.users
```

Предметные связи нативного click-only пилота:

```text
Property[Заявитель]       → Many2one(res.partner)
Property[Связанные люди]  → Many2many(res.partner)
Property[Техника]         → Many2one / Many2many(fleet.vehicle)
Property[Рамка]           → Many2one(maintenance.equipment)
Property[Терминал]        → Many2one(stock.package)
Property[Комплектующие]   → Many2many(stock.lot)
```

`project.task.partner_id` не переиспользуется как универсальный «Заявитель»: его штатная семантика в Project — Customer.

Task не копирует ФИО, госномер, VIN, инвентарный номер, серийники или текущий состав Package. Для Nobilis источником истины по составу, местонахождению и движениям остаётся Stock.

Relational Properties определяются на уровне Project. Поэтому они приняты как базовый механизм пилота, но до окончательного production-решения проверяются поиск, аналитика, импорт, API, работа при нескольких Projects и права на target records.

Обычные ORM-поля в `project.task` добавляются только при доказанном gap. Само отсутствие schema field не является основанием для custom module.

Подробно: [26 — Связь Task с master data](docs/26-task-master-data-relations.md).

## Как выбирать сущность

```text
Есть предметная сущность
        ↓
Есть подходящая штатная модель Odoo?
        │
     да ───→ использовать её
        │
       нет
        ↓
Нужна ли сущности самостоятельная жизнь?
        │
        ├─ нет → Property / Tag / данные Task / Chatter
        │
        └─ да  → кандидат на минимальную собственную model
```

Собственная model оправдана, если объекту нужен самостоятельный реестр, уникальность, lifecycle, отдельные права, связи с множеством records или собственная аналитика.

## Project не является классификатором

Количество Projects определяется реальными границами:

- visibility;
- lifecycle;
- отдельный управленческий контур;
- milestones / project-level reporting;
- существенно различающийся workflow.

Разные виды работ, ТС, сотрудники или категории сами по себе не требуют отдельных Projects: для этого существуют Properties, Tags и Views.

## Stage, State, Deadline и Activity

```text
Stage     → положение Task в настроенном процессе
State     → системное состояние Task
Deadline  → срок результата
Activity  → следующее действие / follow-up
Blocked by → внутренняя зависимость от другой Task
```

Конкретный набор Stages задаётся процессом. Не нужно дублировать системные `Done`, `Canceled` или dependency-driven `Waiting` отдельными Stages без причины.

## Повторяемость

```text
типовая разовая Task по событию
→ Task Template

календарно повторяемая Task
→ Recurring Task

повторяемый набор follow-up
→ Activity Plan / chaining

повторяемая структура инициативы
→ Project Template + Roles
```

Используется только тот механизм, который соответствует реальной потребности.

## Контроль

Базовый путь без собственной разработки:

```text
List / Kanban / Activities
→ Shared Views
→ Task Analysis
→ Pivot / Graph
→ My Dashboard
→ API / внешний BI при необходимости
```

Для большого потока List удобен как рабочее место для поиска, сортировки, массовых действий и сравнения полей; Kanban — как визуализация движения по Stages.

## Права

Организационные полномочия не зашиваются в методику заранее.

После определения фактических ролей используются штатные механизмы Odoo:

```text
Groups
→ Access Rights
→ Record Rules
→ Project visibility
```

Domain relational Property помогает выбирать records, но не заменяет безопасность target model.

## Где допустима адаптация, а где начинается критический компромисс

Методику можно менять под Odoo, если меняется только привычный способ организации работы.

Нельзя считать адаптацию приемлемой, если из-за неё теряется возможность:

- однозначно определить предмет работы;
- зафиксировать контролируемый результат;
- определить ответственных;
- контролировать срок;
- сохранить необходимую историю;
- обеспечить требуемое разграничение доступа;
- получить достоверные данные для контроля.

Если штатная конфигурация не закрывает один из этих пунктов, фиксируется конкретный gap и рассматривается минимальная доработка.

## Порядок развития

```text
штатные Odoo models
→ Properties / штатные relations
→ bridge modules
→ Project workflow
→ Views / Analytics
→ security configuration
→ Import / API / Automation по потребности
→ реальный пилот
→ доказанные gaps
→ минимальная доработка только на gaps
```

## Структура репозитория

### Нормативный слой

- [00 — Возможности Community](docs/00-odoo19-community.md)
- [01 — Модель управления](docs/01-methodology.md)
- [02 — Рабочие сценарии](docs/02-scripts.md)
- [03 — Контроль и аналитика](docs/03-control.md)
- [04 — Шаблоны](docs/04-templates.md)
- [05 — Описание процессов](docs/05-processes.md)
- [06 — Настройка пилота](docs/06-workspace.md)
- [19 — Реестр master data](docs/19-master-data.md)
- [24 — Master data: люди](docs/24-master-data-people.md)
- [25 — Master data: техника и оборудование](docs/25-master-data-assets.md)
- [26 — Связь Task с master data](docs/26-task-master-data-relations.md)

### Технические аудиты

Документы `07–16` и `20–23` фиксируют результаты углублённой проверки возможностей, ограничений, модулей и runtime-гипотез. Они являются техническим приложением и **не задают организационные правила работы по умолчанию**, если нормативный слой уже зафиксировал конкретное решение.

## Где хранится что

- **Odoo** — предметные records, фактическая работа, история, ответственность, сроки и аналитика.
- **Репозиторий** — нейтральная методика, описание конкретных процессов и технические результаты аудита.

Не требуется вести параллельный ручной реестр тех же обязательств, если они уже корректно учитываются в Odoo.

---

Работа ведётся в Draft PR. Merge в `main` — только по отдельной явной команде.

## Лицензия

[CC BY 4.0](LICENSE). Материалы можно использовать и адаптировать при указании источника.
