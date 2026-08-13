# Глоссарий

Этот файл фиксирует терминологию курса. Цель — не перевести API Odoo на русский, а не смешивать обычный русский текст с техническими именами без необходимости.

## Платформа

### Odoo server / сервер Odoo

Серверная часть Odoo, выполняющая Python-логику и работающая с одной или несколькими базами PostgreSQL.

### database / база данных

Отдельная база PostgreSQL, обслуживаемая Odoo. Набор установленных модулей и, следовательно, доступных ORM-моделей может отличаться между базами одного Odoo instance.

### module / модуль

Техническая единица расширения Odoo. В документации также используется термин **addon**.

Модуль объявляется через `__manifest__.py` и может содержать Python-код, data files, views, security, web controllers, static assets или только часть этих элементов.

### addon

Синоним Odoo module в контексте серверной модульной системы.

### App / приложение

User-facing Odoo module, который помечен как полноценное приложение. Большинство технических Odoo modules не являются Apps.

В курсе **App не используется как синоним функциональной области**.

### functional area / функциональная область

Термин курса, а не формальный тип Odoo. Означает предметный контур, который может складываться из нескольких modules.

### `addons_path`

Список каталогов, в которых сервер Odoo ищет modules/addons.

### manifest / манифест

Файл `__manifest__.py`, объявляющий Python package как Odoo module и задающий metadata, dependencies, data files и другие свойства.

## ORM

### model / модель

ORM-модель Odoo. Описывает структуру и поведение набора однотипных records.

Не следует автоматически приравнивать модель к одной таблице PostgreSQL: модель является объектом ORM и может включать fields, methods, relations, constraints и behavior, собранное из нескольких module extensions.

### `Model`

Базовый ORM-класс для обычных database-persisted models.

### `TransientModel`

Базовый ORM-класс для временных records, которые сохраняются в БД, но периодически автоматически очищаются.

### `AbstractModel`

Базовый ORM-класс для общего поведения, предназначенного для переиспользования наследующими models.

### record / запись

Конкретная запись ORM-модели. В Odoo record не имеет отдельного объектного представления: одна запись представляется singleton recordset.

### recordset / набор записей

Упорядоченная коллекция records одной модели. Основной рабочий объект ORM API.

### singleton recordset

Recordset, содержащий ровно один record.

### field / поле

Атрибут ORM-модели, описанный классом `fields.*`. Field может представлять обычное значение, relation, computed value и другую ORM-семантику.

Не следует путать field модели с визуальным полем формы.

### relational field / реляционное поле

Field, связывающее records: `Many2one`, `One2many`, `Many2many`.

### Environment / окружение

Runtime-контекст ORM. Содержит database cursor, текущего user id, context, superuser state и предоставляет доступ к model registry/cache/recomputation machinery.

До уроков Security и Multi-company курс не раскрывает полную семантику изменения Environment.

### domain / домен поиска

Декларативное условие выбора records в ORM Odoo.

### CRUD

Условное обозначение операций создания, чтения, изменения и удаления. В Odoo они выполняются через model/recordset API и не сводятся к прямым SQL-командам.

## Данные и UI

### data file / файл данных модуля

XML или CSV, загружаемый Odoo при установке/обновлении module, если он объявлен в manifest.

### external ID / XML ID

Устойчивый идентификатор record в module data. Будет отдельно разобран в уроке по data files.

### action / действие

Системная запись/описание действия, которое клиент Odoo выполняет в ответ на пользовательское или программное событие. Window actions, например, открывают views моделей.

### view / представление

Описание пользовательского представления данных модели. Не является самой ORM-моделью.

### menu / меню

Элемент навигации web client. Наличие menu не доказывает наличие отдельной предметной модели.

## Security и organization

### user / пользователь

Пользователь Odoo. Подробная модель `res.users`, groups и доступ будет разобрана позже.

### access right / ACL

Права на операции с model. Подробная семантика — в отдельном уроке Security.

### record rule

Правило ограничения доступа к конкретным records. Подробная семантика — в уроке Security.

### company / компания

Организационный объект Odoo. Не рассматривается как универсальный родитель всех records. Multi-company semantics будет отдельным уроком.

## ERP-классификация курса

Следующие термины полезны для бизнес-анализа, но **не являются формальными ORM-классами Odoo**:

- master data;
- reference data;
- transactional record/document;
- configuration data;
- organizational data.

При их использовании применяется маркировка **[ERP-КЛАССИФИКАЦИЯ]**.

## Официальные источники

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- Actions: https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html
