# Высокопоточный триаж и массовая работа с Tasks

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [03 Аналитика](03-control.md) · [06 Настройка](06-workspace.md) · **14 Высокопоточный триаж**

---

Этот документ проверяет стандартный Project не как красивую Kanban-доску, а как рабочий инструмент для большого ежедневного потока входящих Tasks.

## 1. Task List в Community имеет Mass Editing

Стандартный Project Task List View в public `odoo/odoo:19.0` содержит:

```xml
<attribute name="multi_edit">1</attribute>
```

Официальная документация view architecture определяет `multi_edit="1"` как возможность установить одно значение поля сразу нескольким выбранным records.

Источники:

- [`project_task_views.xml`](https://github.com/odoo/odoo/blob/19.0/addons/project/views/project_task_views.xml);
- [View architectures — multi_edit](https://www.odoo.com/documentation/19.0/developer/reference/user_interface/view_architectures.html).

### Методический вывод

Для большой очереди List — не только просмотр.

Он должен использоваться для пакетного triage там, где несколько Tasks действительно получают одно и то же значение.

## 2. Что не нужно делать вручную по одной карточке

Если выбранной группе Tasks действительно одинаково назначается:

- Assignee;
- Stage;
- Deadline;
- Priority;
- другой доступный массовому редактированию field,

использовать mass edit вместо открытия каждой формы.

Перед массовым изменением всегда фильтровать/выбирать records осознанно.

Не использовать mass edit как замену разбору неоднозначных входящих.

## 3. Properties уже отображаются в стандартном List

Public Task list содержит:

```xml
<field name="task_properties"/>
```

Properties также доступны в Form/Kanban/Calendar и Search.

Это означает, что предметные ссылки `ТС`, `Сотрудник`, `Оборудование` можно держать видимыми в операционном списке без отдельной формы/модуля.

### Проверить на пилоте

Насколько удобно именно **массовое изменение dynamic Property** из List. Наличие `multi_edit` у List и наличие Property-колонки не следует автоматически трактовать как гарантированно одинаковый UX для каждого типа Property.

## 4. Next Activity можно видеть прямо в List

Standard Project Task List добавляет:

```text
Next Activity
My Deadline
```

через `activity_ids` и `my_activity_date_deadline`.

Это полезно для внешних ожиданий и follow-up без открытия каждой Task.

## 5. Standard Search уже содержит важные operational filters

Подтверждены:

- Unassigned;
- Blocking;
- Creation Date;
- Open;
- Closed;
- Deadline / Future / This Week / Today / Overdue;
- Rotting;
- Unread Messages;
- My Activities;
- Late Activities;
- Today Activities;
- Future Activities;
- Private Tasks;
- Properties search.

То есть большую часть рабочего места не нужно реализовывать custom domain-логикой.

## 6. Standard Group By уже покрывает основу

Подтверждены grouping по:

- Assignees;
- Stage;
- Project;
- Priority;
- Tags;
- Customer;
- Company;
- Creation Date;
- Deadline;
- **Properties**.

### Практический вывод

Например:

```text
Filter: Open + Overdue
Group By: Assignee
```

или:

```text
Filter: Open
Group By: Properties → ТС
```

может сразу дать операционную картину без отдельного отчёта.

## 7. Default order Task List уже управленчески осмыслен

Public List задаёт порядок:

```text
priority desc
sequence
state
date_deadline asc
id desc
```

Это означает, что Odoo уже пытается поднимать более приоритетные Tasks и учитывать Deadline.

Не нужно обязательно писать собственный алгоритм сортировки очереди до проверки стандартного поведения.

## 8. Kanban и List отвечают на разные вопросы

### Kanban

Лучше отвечает:

> Где работа находится в потоке?

Полезен для:

- движения по Stages;
- визуального WIP;
- quick create;
- небольших/средних наборов.

### List

Лучше отвечает:

> Какие именно records сейчас требуют решения?

Полезен для:

- большого backlog;
- нескольких полей одновременно;
- сортировки;
- массовой правки;
- Deadline;
- Activities;
- relational Properties.

### Методическое правило

Для руководителя/старшего **List должен быть основным диспетчерским представлением**, а Kanban — представлением потока.

Не заставлять большой операционный поток жить только на Kanban.

## 9. Быстрый Task input

Официальная документация Project Task creation подтверждает shortcuts в title input:

```text
30h   → Allocated Time
#tag  → Tag
@user → Assignee
!     → Priority
```

Это может ускорять ручную регистрацию.

Но shortcuts — UX-ускоритель, а не обязательный синтаксис методики.

## 10. Email Alias для массового входа не отменяет triage

Email-created Task автоматически получает:

- sender → Customer;
- subject → Task title;
- body → Description;
- полный email → Chatter;
- внутренние recipients → Followers.

Для 50–70 писем в день это может сильно снизить ручную регистрацию.

Но автоматически созданная Task всё равно должна пройти:

- проверку на дубль;
- определение результата;
- нормализацию Title;
- Deadline;
- Properties;
- решение: Task вообще нужна или нет.

## 11. У автоматического intake должен быть один входной Stage

Все Tasks из email/form/API, которые ещё не прошли детерминированный triage, направлять во:

```text
Входящие
```

Не распределять их автоматически по десяткам Stages только по теме письма без доказанной точности правил.

## 12. Shared Views — часть высокопоточного UX

Для старшего/руководителя подготовить общие кнопки/filters, например:

```text
Новые входящие
Входящие старше X
Неназначенные
Просроченные
Urgent
Late Activities
Waiting
Ожидание внешнего
Rotting
```

Чтобы пользователь не конструировал каждый фильтр заново.

## 13. Unread Messages — отдельный полезный сигнал

Standard Task Search имеет `Unread Messages`.

Это может быть полезно, если новые ответы приходят в Chatter/email thread Task.

Но unread не заменяет Activity/Deadline:

```text
Unread = появилась новая коммуникация
Activity = есть обязательное следующее действие
```

Не превращать Inbox/Unread в единственный способ контролировать follow-up.

## 14. Activity View полезен как отдельная очередь действий

Project Task action включает `activity` view.

Она отвечает не на вопрос статуса результата, а на вопрос:

> Какие следующие действия кому и когда нужно выполнить?

Для процессов с большим числом внешних ожиданий Activity View может оказаться полезнее отдельного custom списка напоминаний.

## 15. Массовое назначение не должно разрушать принцип одного владельца

Mass Editing делает технически легко назначить пользователей пачке Tasks.

Но методика всё равно требует:

- понятного владельца результата;
- разумного размера активного WIP;
- фактического перевода в `В работе` только при старте.

Нельзя превратить массовое назначение в «раздать сотруднику 100 Tasks и считать их активными».

## 16. Массовая классификация Properties требует осторожности

Удобный кейс:

```text
10 Tasks из одного набора входящих
→ один Process
→ один объект/контур
```

Плохой кейс:

```text
100 неоднородных входящих
→ массово поставить один Process/ТС без просмотра
```

Mass edit ускоряет правильное решение, но не делает неправильное решение правильным.

## 17. Spreadsheet как промежуточный аналитический слой

Public Community содержит LGPL-модули:

- `spreadsheet`;
- `spreadsheet_dashboard`.

Официальная документация Odoo 19 описывает вставку отфильтрованных List records в Spreadsheet и динамические списки/пивоты.

Public `spreadsheet` source действительно содержит Odoo list/pivot plugins.

Источники:

- [`spreadsheet/__manifest__.py`](https://github.com/odoo/odoo/blob/19.0/addons/spreadsheet/__manifest__.py);
- [`spreadsheet_dashboard/__manifest__.py`](https://github.com/odoo/odoo/blob/19.0/addons/spreadsheet_dashboard/__manifest__.py);
- [Insert a list](https://www.odoo.com/documentation/19.0/applications/productivity/spreadsheet/insert/insert_list.html).

### Но редакционную доступность конкретного `Actions → Spreadsheet` пути проверить на стенде

Механизм Spreadsheet и plugins публично есть в Community source, но общая документация может описывать комбинацию модулей/приложений шире конкретной CE-установки.

Поэтому до включения в baseline выполнить runtime-тест:

```text
Project Task List
→ фильтр
→ Actions
→ проверить Spreadsheet actions
```

## 18. Если runtime-тест подтверждает Spreadsheet actions

Разовые аналитические задачи можно решать по лестнице:

```text
List/Pivot/Graph
→ My Dashboard
→ Spreadsheet для расчётного/презентационного свода
→ Spreadsheet Dashboard для устойчивого KPI
→ внешний BI только при реальной потребности
```

Это уменьшает преждевременную зависимость от внешнего Excel/BI.

## 19. Минимальный runtime-тест high-volume режима

Загрузить/создать тестово хотя бы 200–500 Tasks и проверить:

1. List responsiveness;
2. filter `Входящие`;
3. sorting by Deadline/Priority;
4. mass edit обычного поля;
5. mass assign;
6. Properties column;
7. filter/group by relational Property;
8. Activities columns;
9. Late Activities filter;
10. Unread Messages;
11. Shared Views;
12. Spreadsheet action — если доступен;
13. usability на реальном экране пользователей.

Только после этого оценивать, нужен ли отдельный custom triage UI.

---

[← 13 — Смены и регламентные циклы](13-shift-routines.md) · [Главная](../README.md)
