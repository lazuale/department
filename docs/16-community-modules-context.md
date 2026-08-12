# Опорный реестр модулей Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [01 Модель](01-methodology.md) · [06 Настройка](06-workspace.md)

---

Этот файл — **технический source of truth проекта по составу официального Odoo 19 Community**, на который можно опираться при дальнейшей адаптации методики.

Он отвечает на четыре разных вопроса и **не смешивает их**:

1. модуль присутствует в публичном Community source;
2. модуль является пользовательским App или Extra/hidden-модулем;
3. модуль устанавливается явно или автоматически как dependency/bridge;
4. используем ли мы его методически в нашем рабочем контуре.

Наличие модуля в исходниках **не означает**, что его надо использовать в процессе. Наличие страницы в общей документации Odoo **не означает**, что функция входит в Community: документация охватывает разные редакции, поэтому edition-boundary проверяется по публичной ветке и manifest.

## 1. Зафиксированный snapshot проверки

Проверка выполнена: **2026-08-12**.

Источник Community-кода:

```text
repository: odoo/odoo
branch:     19.0
commit:     cb731ca8720cd01a5719890cf6c5e140dc551546
```

Источник назначения функций:

```text
Odoo 19.0 official user documentation
```

Ключевая официальная особенность каталога Odoo:

```text
Apps filter  → пользовательские приложения
Extra filter → остальные устанавливаемые модули
```

Поэтому полный контекст нельзя строить только по иконкам Apps.

### Правило доверия к источникам

При противоречии использовать порядок:

1. `odoo/odoo:19.0` manifest/source — подтверждает наличие Community-модуля и dependency/auto_install;
2. официальная документация Odoo 19 — подтверждает назначение функции;
3. runtime-проверка нашего стенда — подтверждает фактический UI, права и поведение конкретной сборки;
4. только после этого — методическое решение проекта.

## 2. Граница полноты этого файла

Реестр считается полным для дальнейшей методики, если он содержит:

1. **все Community Apps** публичной ветки, у которых `application=True`;
2. **функционально значимые Extra/hidden-модули**, которые документированы Odoo либо могут повлиять на наши процессы;
3. **dependency / auto-install / bridge-модули**, влияющие на `Используем` и `Возможно`;
4. **крупные функции общей документации, отсутствующие в Community source**, чтобы их нельзя было случайно принять за CE-возможность;
5. явные семейства модулей, которые сознательно не перечисляются поштучно.

На проверенном snapshot найдено **34 Community Apps с `application=True`**. Ниже перечислены **34/34**.

Не является целью перечислить каждую локализацию, payment provider, тестовый модуль, тему сайта или низкоуровневую библиотеку: это не помогает выбирать рабочую модель отдела.

## 3. Значение проектных статусов

| Статус | Значение |
|---|---|
| **Используем** | модуль или функция входит в базовый рабочий/технический контур |
| **Возможно** | есть реальный сценарий; решение принимается после разбора процесса или runtime-теста |
| **Вероятно нет** | для известных процессов модуль не нужен либо создаст конкурирующий предметный контур |
| **Автоматически присутствует** | модуль приезжает dependency/auto-install; это не означает обязательное методическое использование |
| **Не Community baseline** | на этот модуль/функцию нельзя опираться в CE без отдельной внешней реализации |

Отдельно:

```text
технически установлен
≠
виден пользователю как отдельное приложение
≠
используется методикой
```

## 4. Полный перечень Community Apps — 34/34

Категория в первой колонке — смысловая группа для навигации по реестру; техническое имя и `application=True` проверены по публичной ветке.

| Группа | App | Technical name | Решение | Роль / причина |
|---|---|---|---|---|
| Работа | **Project / Проекты** | `project` | **Используем** | самостоятельная управляемая работа, Tasks, workflow, сроки, зависимости, шаблоны, recurrence, анализ |
| Коммуникация | **Discuss / Обсуждения** | `mail` | **Используем** | Chatter, Activities, mail gateway, каналы и контекстная коммуникация |
| HR / справочник | **Employees / Сотрудники** | `hr` | **Используем** | нативные записи сотрудников; не равны `res.users` |
| Справочник | **Contacts / Контакты** | `contacts` | **Используем** | внешние люди, организации и контрагенты |
| Транспорт | **Fleet / Автопарк** | `fleet` | **Используем** | нативные записи ТС; не система путевых листов |
| Productivity | **Calendar / Календарь** | `calendar` | **Возможно** | встречи и календарные события, если они реально нужны процессам |
| Productivity | **To-Do** | `project_todo` | **Возможно методически; автоматически присутствует с Project** | личный лёгкий список; не должен стать вторым скрытым backlog |
| Data | **Data Recycle** | `data_recycle` | **Возможно** | архивирование/удаление устаревших записей по правилам; административная функция |
| Supply Chain | **Inventory / Склад** | `stock` | **Возможно** | serial/lot/location/movement для оборудования и материальных объектов |
| Supply Chain | **Maintenance / Обслуживание** | `maintenance` | **Возможно** | Equipment и Maintenance Requests, если этот предметный контур действительно ведём в Odoo |
| Data collection | **Surveys / Опросы** | `survey` | **Возможно** | структурированные анкеты/опросы, если подходят реальному входу данных |
| Website | **Website / Сайт** | `website` | **Возможно** | web-вход и публичные/внутренние страницы, если потребуется |
| Supply Chain | **Purchase / Закупки** | `purchase` | **Возможно** | только если закупочный workflow реально переносится в Odoo |
| Accounting | **Accounting / Бухгалтерия** | `account` | **Вероятно нет** | не создаём второй учётный контур вместо действующей системы |
| Sales | **CRM** | `crm` | **Вероятно нет** | наши входящие обязательства не являются leads/opportunities |
| Sales | **Sales / Продажи** | `sale_management` | **Вероятно нет** | не относится к текущим процессам |
| Sales | **Point of Sale** | `point_of_sale` | **Вероятно нет** | не относится к текущим процессам |
| Sales | **Restaurant POS** | `pos_restaurant` | **Вероятно нет** | не относится к текущим процессам |
| Manufacturing | **Manufacturing / Производство** | `mrp` | **Вероятно нет** | не заменяет действующие производственные информационные системы |
| Supply Chain | **Repair / Ремонт** | `repair` | **Вероятно нет** | отдельный repair workflow не заявлен; для внутренних активов сначала оценивается Maintenance |
| HR | **Attendances / Посещаемость** | `hr_attendance` | **Вероятно нет** | check-in/check-out не является нашим табелем |
| HR | **Time Off / Отпуска** | `hr_holidays` | **Вероятно нет** | HR-процесс отсутствий не входит в текущий контур |
| HR | **Expenses / Расходы** | `hr_expense` | **Вероятно нет** | не относится к известным процессам отдела |
| HR | **Skills / Навыки** | `hr_skills` | **Вероятно нет** | не влияет на текущую операционную модель |
| HR | **Recruitment / Подбор** | `hr_recruitment` | **Вероятно нет** | подбор персонала не относится к текущему контуру |
| Internal services | **Lunch / Питание** | `lunch` | **Вероятно нет** | корпоративный заказ еды, а не наша сверка питания |
| Marketing | **Email Marketing** | `mass_mailing` | **Вероятно нет** | маркетинговые рассылки не нужны для операционного управления |
| Marketing | **SMS Marketing** | `mass_mailing_sms` | **Вероятно нет** | не относится к текущим процессам |
| Marketing | **Marketing Card** | `marketing_card` | **Вероятно нет** | не относится к текущим процессам |
| Website | **Live Chat** | `im_livechat` | **Вероятно нет** | публичный web-chat не требуется; внутренний контекст закрывает Discuss |
| Website | **eCommerce** | `website_sale` | **Вероятно нет** | не относится к текущим процессам |
| Website | **Events** | `website_event` | **Вероятно нет** | мероприятия/регистрации не относятся к текущему контуру |
| Website | **eLearning** | `website_slides` | **Вероятно нет** | отдельный контур обучения, не управление текущей работой |
| Website / HR | **Online Recruitment** | `website_hr_recruitment` | **Вероятно нет** | не относится к текущему контуру |

### Контроль количества

```text
Используем:      5 Apps
Возможно:        8 Apps
Вероятно нет:   21 Apps
Итого:          34 Apps
```

`board` и `api_doc` не входят в эти 34, потому что это Extra/hidden-модули, рассмотренные отдельно ниже.

## 5. Функционально значимые Extra / hidden-модули

Эти модули могут не отображаться отдельной карточкой при стандартном Apps filter, но присутствуют в public Community source.

| Группа | Модуль | Technical name | Технический статус | Наше решение |
|---|---|---|---|---|
| Control | **Dashboards / My Dashboard** | `board` | installable, зависит от `spreadsheet_dashboard` | **Используем My Dashboard**; spreadsheet-based authoring не делаем обязательным без runtime-проверки |
| Control | **Spreadsheet Dashboard engine** | `spreadsheet_dashboard` | installable, зависит от `spreadsheet` | **Автоматически/технически рядом с `board`**; не отдельное методическое требование |
| Control | **Spreadsheet engine** | `spreadsheet` | installable Community source | **Техническая зависимость**; standalone Documents-based workflow не считать CE-baseline без runtime-проверки |
| API | **API Documentation** | `api_doc` | `auto_install=True`, зависит от `web` | **Используем как техническую инфраструктуру** для `/doc` и фактической схемы API |
| Automation | **Automation Rules** | `base_automation` | installable Community source | **Возможно** после стабилизации процесса; не строить модель процесса из автоматизаций |
| Project/Web | **Online Task Submission** | `website_project` | `auto_install=True`, зависит от `website` + `project` | **Возможно** для будущего web-входа заявок |
| Time tracking | **Timesheets / Task Logs** | `hr_timesheet` | installable Community source | **Вероятно нет**: время по Project Tasks ≠ наши табельные данные |
| HR time | **Work Entries** | `hr_work_entry` | installable Community source | **Вероятно нет**: HR work intervals ≠ система сверки табелей |
| Inventory UX | **Barcode base** | `barcodes` | installable Community source | **Возможно** только если Inventory/серийники реально требуют сканирования; не путать с полным Enterprise Barcode workflow |
| Website | **Forum** | `website_forum` | installable Community source, user-facing | **Вероятно нет** |
| Website | **Blog** | `website_blog` | installable Community source, user-facing | **Вероятно нет** |
| Events | **Event core** | `event` | installable базовый модуль, dependency `website_event` | **Вероятно нет**; не путать с App `website_event` |
| Calendar integration | **Google Calendar** | `google_calendar` | installable Community source | **Возможно**, если Calendar реально используется |
| Calendar integration | **Outlook Calendar** | `microsoft_calendar` | installable Community source | **Возможно**, если Calendar реально используется |
| Mail integration | **Google Gmail** | `google_gmail` | hidden, `auto_install=True`, зависит от `mail` | **Технически может присутствовать автоматически**; настраивать только при использовании Gmail |
| Mail integration | **Microsoft Outlook** | `microsoft_outlook` | hidden, `auto_install=True`, зависит от `mail` | **Технически может присутствовать автоматически**; настраивать только при использовании Outlook |
| Security | **LDAP authentication** | `auth_ldap` | installable, требует `python-ldap` | **Возможно** при наличии корпоративного LDAP |
| Security | **OAuth2 authentication** | `auth_oauth` | installable Community source | **Возможно** при необходимости SSO/OAuth |
| Security | **Passkeys** | `auth_passkey` | `auto_install=True` | **Возможно как security capability**; не является процессной сущностью |
| Security | **TOTP MFA** | `auth_totp` | `auto_install=True` | **Возможно как security capability**; не является процессной сущностью |

### Важное ограничение Spreadsheet / Dashboards

Public source подтверждает наличие:

```text
board
  ↓ depends
spreadsheet_dashboard
  ↓ depends
spreadsheet
```

Официальная документация описывает Spreadsheet как часть Documents и одновременно использует Spreadsheet как основу Dashboards. Поэтому для Community методика **может опираться на My Dashboard**, но не должна считать полный Documents-style Spreadsheet workflow гарантированным до runtime-проверки нашего стенда.

## 6. Dependency graph базового контура

### Project

Manifest `project` напрямую зависит от:

```text
analytic
base_setup
mail
portal
rating
resource
web
web_tour
digest
```

Это технические зависимости Project, а не отдельные методические приложения отдела.

### Project → To-Do

```text
project
  ↓
project_todo   [auto_install=True]
```

Следствие:

> при нашем обязательном Project `project_todo` нельзя трактовать как полностью независимое решение «ставить / не ставить»; решаем только, **используем ли To-Do в методике**.

### Dashboards

```text
board
  ↓
spreadsheet_dashboard
  ↓
spreadsheet
```

Следствие:

> факт технической установки Spreadsheet-компонентов не означает, что пользователи обязаны работать через Spreadsheet.

### API docs

```text
web
 ↓
api_doc   [auto_install=True]
```

### Employees + Fleet

```text
hr + fleet
    ↓
hr_fleet   [auto_install=True]
```

Для нашего baseline это штатная автоматическая связь Employees ↔ Fleet.

### Discuss / mail infrastructure

`mail` является общей инфраструктурой Chatter, Activities, aliases и mail gateway для многих приложений. Установка зависимых приложений не означает, что каждый email должен автоматически превращаться в Task.

## 7. Bridge-модули, важные для возможного расширения

| Bridge | Technical name | Связка | Auto-install / роль | Решение |
|---|---|---|---|---|
| Fleet History | `hr_fleet` | Employees ↔ Fleet | auto-install | **Используем автоматически** |
| Maintenance – HR | `hr_maintenance` | Employees ↔ Maintenance | auto-install | **Возможно автоматически** |
| Stock – Maintenance | `stock_maintenance` | Inventory ↔ Maintenance | auto-install | **Возможно автоматически** |
| Project – Purchase | `project_purchase` | Project ↔ Purchase | штатный bridge | **Возможно** |
| Project – Stock | `project_stock` | Project ↔ Stock | штатный bridge | **Возможно** |
| Project Stock Account | `project_stock_account` | Project/Stock ↔ stock accounting analytics | auto-install при зависимостях | **Вероятно нет**, пока Accounting не в нашем контуре |
| Project – Accounting | `project_account` | Project ↔ accounting/analytic layer | штатный bridge | **Вероятно нет** |
| Project – Expenses | `project_hr_expense` | Project ↔ employee expenses | штатный bridge | **Вероятно нет** |
| HR Calendar | `hr_calendar` | Employees ↔ Calendar working hours | auto-install | **Возможно**, если Calendar войдёт в процесс |
| Skills Certification | `hr_skills_survey` | Skills ↔ Survey | auto-install | **Вероятно нет** |
| Skills eLearning | `hr_skills_slides` | Skills ↔ eLearning | auto-install | **Вероятно нет** |
| Accounting/Fleet | `account_fleet` | Fleet ↔ Accounting | auto-install | **Вероятно нет** |

Правило:

> bridge подтверждает штатную техническую связь двух моделей. Он **не доказывает**, что нам нужен соответствующий предметный процесс, и не гарантирует готовый отчёт именно под наш управленческий вопрос.

## 8. Безопасность и инфраструктура: не путать с методикой

Следующие Community-модули могут быть важны для production-инсталляции, но не определяют бизнес-методику:

```text
auth_ldap
  → LDAP, если есть корпоративный каталог

auth_oauth
  → OAuth2 / SSO, если требуется

auth_passkey
  → passkeys

auth_totp
  → TOTP MFA

google_gmail / microsoft_outlook
  → транспорт почты через соответствующих провайдеров

google_calendar / microsoft_calendar
  → внешняя синхронизация Calendar
```

Их включение определяется требованиями ИБ и инфраструктуры отдельно от рабочих процессов.

## 9. Крупные функции официальной документации, на которые нельзя опираться как на Community baseline

На проверенном public `odoo/odoo:19.0` отсутствуют соответствующие top-level модули:

| Функция / приложение | Ожидаемый module family | Community baseline |
|---|---|---|
| Studio | `web_studio` | **Нет** |
| Helpdesk | `helpdesk` | **Нет** |
| Planning | `planning` | **Нет** |
| Documents | `documents` | **Нет** |
| Knowledge | `knowledge` | **Нет** |
| Approvals | `approvals` | **Нет** |
| Sign | `sign` | **Нет** |
| Appointments | `appointment` | **Нет** |
| Quality / Quality Control | `quality`, `quality_control` | **Нет** |
| Full Data Cleaning / Deduplication | `data_cleaning`, `data_merge*` | **Нет; Community имеет только `data_recycle` из этого семейства** |
| Project Gantt / Enterprise planning UX | enterprise project/planning additions | **Не считать baseline** |

Это не означает, что аналогичный процесс невозможно реализовать иначе. Это означает только:

> нельзя писать методику так, будто данное Enterprise-приложение уже является штатной возможностью Odoo 19 Community.

## 10. Семейства public add-ons, которые сознательно не перечисляются поштучно

Они существуют в Community source, но не должны раздувать рабочий контекстный реестр.

| Семейство | Как учитываем |
|---|---|
| `l10n_*` | локализации; рассматриваются только при внедрении бухгалтерского/налогового контура |
| `payment_*` | payment engine/providers; не относятся к текущим процессам |
| `website_*` технические bridges/snippets/themes | перечисляем отдельно только user-facing или архитектурно значимые (`website_project`, Forum, Blog и т.п.) |
| EDI / fiscal provider modules | вне текущего контура |
| POS-specific bridges | вне текущего контура |
| test/demo/tour helpers | техническая разработка, не бизнес-функция |
| low-level `base`, `web`, `bus`, `portal`, `resource`, `rating`, `digest`, `analytic` и т.п. | учитываются как dependencies; не рассматриваются как самостоятельные бизнес-приложения |

Такое исключение **не является пробелом**: файл остаётся полным по определённой выше функционально-архитектурной границе.

## 11. Базовый контур проекта

### Осознанно используем

```text
Project        project
Discuss        mail
Employees      hr
Contacts       contacts
Fleet          fleet
My Dashboard   board
API docs       api_doc
```

### Технически приезжает рядом

```text
project_todo
hr_fleet
spreadsheet_dashboard
spreadsheet
api_doc (auto-install через web)
и прямые инфраструктурные dependencies выбранных Apps
```

### Первые кандидаты на расширение по реальному процессу

```text
Maintenance    maintenance
Inventory      stock
Calendar       calendar
Website        website
Survey         survey
Data Recycle   data_recycle
Automation     base_automation
Purchase       purchase
```

Отдельно при соответствующей инфраструктуре:

```text
auth_ldap / auth_oauth / auth_passkey / auth_totp
google_gmail / microsoft_outlook
google_calendar / microsoft_calendar
```

## 12. Предметные границы для известных процессов

### Транспорт / путевые листы

```text
ТС                         → Fleet
исходные/учётные ПЛ        → действующая предметная система
исправление / проверка     → Project Task, если это самостоятельная работа
следующий follow-up        → Activity
```

Fleet не превращается в систему путевых листов только потому, что Task относится к ТС.

### Табели / Мстрой / 1С

```text
сотрудник                  → Employees
табельные данные           → предметная система
сверка / исправление       → Task при самостоятельном управляемом результате
```

`hr_timesheet`, `hr_attendance` и `hr_work_entry` не используются как искусственная замена действующих табельных систем.

### Сверки

```text
исходные строки            → SQL / 1С / Мстрой / портал / BI / файл
цикл сверки                → Task, если его нужно отдельно управлять
отдельное исключение       → новая Task только при самостоятельном результате / исполнителе / сроке / решении
```

Не создавать Task на каждую строку расхождения.

### Топливные карты

Транзакции, литры и расхождения остаются предметными данными. Odoo управляет работой по сверке, запросу, исправлению и контролю результата.

### Оборудование

```text
Equipment                  → Maintenance, если модель подходит
serial / lot / location    → Inventory, если нужен физический учёт
самостоятельная работа     → Project Task
```

### Входящие

```text
email / web form
      ↓
существующая работа?       → Chatter
новая управляемая работа?  → Task
```

`website_project` включается только при реальной необходимости создавать Tasks через web-вход.

## 13. Критерий выбора модуля в методику

Модуль переводится между `Используем / Возможно / Вероятно нет` только после ответов:

1. Какой реальный процесс или объект он представляет?
2. Где находится источник истины?
3. Не создаёт ли он второй конкурирующий реестр?
4. Повышает ли он удобство ежедневной работы?
5. Сохраняет ли контроль и аналитическую пригодность?
6. Нужна ли его собственная предметная модель, или достаточно существующей записи / Activity / Task / Property?
7. Не требует ли решение Enterprise-компонента, которого нет в public Community source?

Если модуль не даёт явной ценности — не устанавливать его «на будущее».

## 14. Когда этот реестр надо перепроверять

Полная повторная проверка обязательна, если:

- изменился commit публичной ветки `odoo/odoo:19.0`, и нам важны появившиеся/изменившиеся модули;
- переходим на новую major-версию Odoo;
- хотим добавить новый предметный App;
- официальная документация описывает функцию, которой нет в текущем реестре;
- runtime-пилот показывает отличие от ожидаемого поведения;
- рассматриваем OCA/сторонний модуль или собственную доработку.

При обновлении фиксировать новый source snapshot в начале файла.

## 15. Проверенные исходники, критичные для этого реестра

### Базовые Apps / функции

```text
addons/project/__manifest__.py
addons/mail/__manifest__.py
addons/hr/__manifest__.py
addons/contacts/__manifest__.py
addons/fleet/__manifest__.py
addons/calendar/__manifest__.py
addons/maintenance/__manifest__.py
addons/stock/__manifest__.py
addons/project_todo/__manifest__.py
addons/data_recycle/__manifest__.py
addons/website_event/__manifest__.py
```

### Extra / technical

```text
addons/board/__manifest__.py
addons/spreadsheet_dashboard/__manifest__.py
addons/spreadsheet/__manifest__.py
addons/api_doc/__manifest__.py
addons/base_automation/__manifest__.py
addons/hr_timesheet/__manifest__.py
addons/hr_work_entry/__manifest__.py
addons/barcodes/__manifest__.py
addons/website_project/__manifest__.py
addons/website_forum/__manifest__.py
addons/website_blog/__manifest__.py
addons/event/__manifest__.py
```

### Integrations / security

```text
addons/google_calendar/__manifest__.py
addons/microsoft_calendar/__manifest__.py
addons/google_gmail/__manifest__.py
addons/microsoft_outlook/__manifest__.py
addons/auth_ldap/__manifest__.py
addons/auth_oauth/__manifest__.py
addons/auth_passkey/__manifest__.py
addons/auth_totp/__manifest__.py
```

### Bridges

```text
addons/hr_fleet/__manifest__.py
addons/hr_maintenance/__manifest__.py
addons/stock_maintenance/__manifest__.py
addons/project_purchase/__manifest__.py
addons/project_stock/__manifest__.py
addons/project_stock_account/__manifest__.py
addons/project_account/__manifest__.py
addons/project_hr_expense/__manifest__.py
addons/hr_calendar/__manifest__.py
addons/hr_skills_survey/__manifest__.py
addons/hr_skills_slides/__manifest__.py
addons/account_fleet/__manifest__.py
```

### Официальная документация, важная для edition-boundary

```text
applications/general/apps_modules.html
applications/productivity/data_cleaning.html
applications/productivity/dashboards.html
applications/productivity/spreadsheet.html
applications/services/timesheets.html
applications.html
```

## 16. Статус документа

Этот файл можно использовать как **опорный технический контекст при дальнейшем проектировании методики** для проверенного snapshot Odoo 19 Community.

Но он не является статичным навсегда: доступность модулей — свойство конкретной версии исходников.

Нормативная методика должна ссылаться на этот реестр, а не заново перечислять edition-boundary и набор приложений в каждом документе.

Конкретное решение использовать модуль в реальном процессе переносится в нормативный слой только после отдельного согласования.