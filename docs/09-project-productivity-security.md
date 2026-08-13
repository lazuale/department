# Скрытые настройки Project и базовая безопасность Odoo 19 Community

[Главная](../README.md) · [07 Углублённый аудит](07-deep-community-audit.md) · [08 Интеграции Project](08-project-integrations.md) · **09 Project и безопасность**

---

Этот проход проверяет две группы возможностей, которые легко недооценить:

1. настройки и UX стандартного `Project`, неочевидные из основного руководства;
2. штатные механизмы аутентификации Community, важные для реального рабочего стенда.

## 1. Project Stages — отдельный workflow на уровне проектов

Task Stages и Project Stages — разные сущности.

Публичный Community `project` содержит отдельную модель:

```text
project.project.stage
```

а в Settings есть включаемая группа:

```python
group_project_stages = fields.Boolean(
    "Project Stages",
    implied_group="project.group_project_stages",
)
```

Источники:

- [`res_config_settings.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/res_config_settings.py);
- [`project_project_stage.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_project_stage.py).

### Что умеет Project Stage

Подтверждены:

- name;
- sequence;
- company;
- color;
- email template;
- Folded;
- project grouping/filtering by Stage;
- tracking времени нахождения Project в Stage через `statusbar_duration`.

Если Project Stage помечен Folded, проекты в нём считаются закрытыми.

### Методическое решение

Для **одного постоянного операционного Project** Project Stages почти ничего не дают.

Для нескольких самостоятельных инициатив они полезны как портфельный pipeline, например:

```text
Идея / подготовка
→ Запланирован
→ В реализации
→ На паузе
→ Завершён
```

Но эта схема вводится только если реально ведётся портфель проектов.

Не смешивать:

```text
Project Stage = состояние целого проекта
Task Stage    = состояние отдельной задачи
Project Update Status = управленческая оценка On Track / At Risk / Off Track / On Hold / Done
```

Это три разных измерения.

## 2. Project также хранит собственные Start / End dates

В базовой Community-модели Project есть:

```text
date_start = Start Date
date       = Expiration Date
```

В стандартной форме они отображаются как `Planned Date` диапазоном.

Источник: [`project_project.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_project.py), [`project_project_views.xml`](https://github.com/odoo/odoo/blob/19.0/addons/project/views/project_project_views.xml).

### Методическое решение

Для самостоятельной инициативы использовать project timeframe.

Для вечного операционного контура отдела искусственную дату завершения не задавать.

## 3. Название `Tasks` можно менять на уровне проекта

Community Project содержит поле:

```python
label_tasks = fields.Char(
    string='Use Tasks as',
    default='Tasks',
    help='Name used to refer to the tasks of your project e.g. tasks, tickets, sprints, etc...'
)
```

В форме Project оно доступно как:

```text
Name of the Tasks
```

Источник: [`project_project.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_project.py), [`project_project_views.xml`](https://github.com/odoo/odoo/blob/19.0/addons/project/views/project_project_views.xml).

### Методическое решение

Это можно использовать для понятной терминологии конкретного Project, если она действительно улучшает UX.

Например:

```text
Tasks → Работы
```

Но не использовать переименование, чтобы скрыть неправильный выбор сущности.

Например, переименовать Tasks в `Автомобили` не превращает Project Task в Fleet Vehicle.

## 4. Project Manager и Favorites — отдельные признаки проекта

Базовый Project содержит:

- `user_id` — Project Manager;
- `favorite_user_ids` / `is_favorite` — personal favorite project;
- фильтры `My Projects`, `My Favorites`, `Unassigned Projects`.

Для портфеля инициатив это позволяет отделить:

```text
кто управляет проектом
от
кто является assignee конкретной Task
```

Не назначать руководителя Assignee всех Tasks только потому, что он Project Manager.

## 5. Project-level Activities

`project.project` наследует `mail.activity.mixin`.

То есть Activity можно ставить не только на Task, но и на сам Project.

### Где полезно

Для действий уровня инициативы:

```text
провести статус-встречу
обновить Project Update
проверить готовность milestone
эскалировать риск проекта
```

### Где не нужно

Обычное рабочее действие по конкретной Task должно оставаться Activity этой Task.

Не использовать Project Activity как общую корзину follow-up всего отдела.

## 6. Время в Project Stage также отслеживается

`project.project` наследует `mail.tracking.duration.mixin` и задаёт:

```python
_track_duration_field = 'stage_id'
```

В форме Project `stage_id` использует:

```text
widget="statusbar_duration"
```

Следовательно, при включённых Project Stages Odoo умеет показывать длительность нахождения самого Project в стадиях.

Это полезно для диагностики портфеля инициатив.

Не смешивать с time-in-stage Task.

## 7. Priority: модель и UI действительно поддерживают четыре уровня

В `project.task` Community подтверждены:

```text
0 Low priority
1 Medium priority
2 High priority
3 Urgent
```

Стандартная форма использует `priority_switch`.

`PriorityField` строит звёзды по всем Selection options выше нулевого значения, а `PrioritySwitchField` также создаёт команды для выбора каждого значения.

Источники:

- [`project_task.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task.py);
- [`project_task_priority_switch_field.js`](https://github.com/odoo/odoo/blob/19.0/addons/project/static/src/components/project_task_priority_switch_field/project_task_priority_switch_field.js);
- [`priority_field.xml`](https://github.com/odoo/odoo/blob/19.0/addons/web/static/src/views/fields/priority/priority_field.xml).

### Почему документация вводит в заблуждение

Пользовательская документация Task creation описывает простое действие со звездой как high priority, но публичная модель и widget 19.0 поддерживают четыре значения.

Для Community-границы здесь source является более точным описанием фактической реализации.

### Методическое решение сохраняется

Не обязаны использовать все четыре уровня.

Начальный вариант:

```text
обычная → Low/default
критичная → Urgent
```

Medium/High вводить только после появления однозначных критериев.

## 8. Быстрый ввод Tasks уже умеет часть структурирования

Официальная документация Odoo 19 для manual task creation подтверждает inline shortcuts в названии Task:

```text
30h       → Allocated Time
#tag      → Tags
@user     → Assignee
!         → Priority
```

Пример из логики документации:

```text
Подготовить отчёт 5h #сверка @Иван !
```

Источник: [Task creation](https://www.odoo.com/documentation/19.0/applications/services/project/tasks/task_creation.html).

### Методическое решение

После стабилизации Tags и привычек пользователей это может ускорить регистрацию работы.

Но не делать shortcuts обязательным языком работы: они ускорение UI, а не бизнес-правило.

## 9. Community не подтверждает Sign как штатное приложение

Общая документация Odoo 19 содержит `Sign`, но `addons/sign` отсутствует в публичной ветке `odoo/odoo:19.0`.

### Методический вывод

Не закладывать электронное подписание документов в Community baseline.

Если в будущем появится требование электронной подписи, его нужно рассматривать отдельно от текущей click-only методики.

## 10. Community не подтверждает Appointments как штатное приложение

Общая документация Odoo содержит Appointments, но публичный `addons/appointment` в `odoo/odoo:19.0` не подтверждён.

Для встреч Community baseline остаётся:

```text
Calendar Event
Activities
```

Не обещать публичную booking page Appointments без проверки редакции.

## 11. Полноценный Quality app также не подтверждён public CE

В публичной ветке Odoo 19 не подтверждён отдельный стандартный `quality` app из Enterprise-контура качества.

### Методический вывод

Не использовать Quality Checks / Quality Control как скрытую зависимость методики Community.

Если потребуется устойчивый процесс контрольных проверок, сначала определить:

- решается ли он Project review Stage/Activity;
- является ли это Maintenance/Inventory процессом;
- требуется ли отдельная доработка.

## 12. LDAP Authentication есть в Community

Публичный модуль:

```text
auth_ldap
```

Официальная документация Odoo 19 подтверждает настройку через:

```text
Settings → Integrations → LDAP Authentication
```

и параметры:

- LDAP server/port;
- TLS;
- bind DN/password;
- LDAP base/filter;
- automatic user creation;
- User template.

Источники:

- [LDAP authentication](https://www.odoo.com/documentation/19.0/applications/general/users/ldap.html);
- [`auth_ldap`](https://github.com/odoo/odoo/blob/19.0/addons/auth_ldap/__manifest__.py).

### Методический/эксплуатационный вывод

Если в организации есть LDAP/Active Directory и IT готово его использовать, штатная Community-аутентификация может убрать отдельное управление паролями Odoo.

Это инфраструктурное решение, не часть workflow задач.

## 13. TOTP 2FA есть в Community и может быть обязательной

Публичный `auth_totp` автоматически устанавливается с Web и реализует TOTP 2FA.

Официальная документация Odoo 19 подтверждает:

- пользовательскую настройку 2FA;
- QR/TOTP authenticator;
- принудительное включение для Employees или All Users.

Источники:

- [Two-factor authentication](https://www.odoo.com/documentation/19.0/applications/general/users/2fa.html);
- [`auth_totp`](https://github.com/odoo/odoo/blob/19.0/addons/auth_totp/__manifest__.py).

### Решение

Для production-контуров рассматривать 2FA как штатный механизм безопасности, а не стороннюю доработку.

## 14. Passkeys есть в публичном Community source

Модуль [`auth_passkey`](https://github.com/odoo/odoo/blob/19.0/addons/auth_passkey/__manifest__.py):

- WebAuthn;
- passkey login;
- auto-install;
- security views пользователя.

Официальная документация пользовательского портала также содержит управление Passkeys в Connection & Security.

### Решение

Passkeys — дополнительный штатный вариант аутентификации.

Не делать его обязательным условием методики управления; это решение IT/security.

## 15. OAuth2 Authentication есть в Community

Публичный [`auth_oauth`](https://github.com/odoo/odoo/blob/19.0/addons/auth_oauth/__manifest__.py) позволяет login через OAuth2 provider.

Общая документация Odoo 19 также имеет отдельные настройки Google Sign-In и Microsoft Azure sign-in.

### Решение

При корпоративном Identity Provider сначала проверить штатный OAuth/LDAP путь, прежде чем писать кастомную SSO-интеграцию.

## 16. Иерархия безопасности для пилота

Для рабочего стенда методика не должна задавать самодельную security-архитектуру, но должна проверять её в правильном порядке:

```text
аутентификация
→ Internal / Portal user type
→ application access rights
→ Project visibility
→ bridge-module visibility
→ только затем custom groups / record rules
```

### Отдельно проверить

- обычный исполнитель;
- старший/руководитель;
- администратор;
- при необходимости portal collaborator.

Не использовать Superuser для ежедневной работы и не тестировать права только под администратором.

## 17. Что добавляется в пилот после этого прохода

Не новые приложения пачкой, а новые **проверки**:

```text
Project Stages     → проверить только на проектных инициативах
Name of the Tasks  → решить, улучшает ли русскую терминологию
Task quick syntax  → проверить удобство пользователей
4-level Priority   → подтвердить визуальное поведение локализованного UI
LDAP/OAuth         → только вместе с IT, если есть корпоративный IdP
2FA                → проверить production security policy
Passkeys           → опционально
```

## 18. Итог

После этого прохода становится видно, что даже внутри базового Community Project есть два уровня управления:

```text
Portfolio / Project
→ Project Stage
→ Project Manager
→ Project Update Status
→ Milestones
→ Project Activities

Execution / Task
→ Task Stage
→ Assignee
→ Task State
→ Deadline
→ Activities
→ Dependencies
```

Для постоянной операционной работы нам в основном нужен второй уровень.

Для самостоятельных инициатив можно использовать оба — без собственной системы портфельного управления.

---

[← 08 — Интеграции Project](08-project-integrations.md) · [Главная](../README.md)
