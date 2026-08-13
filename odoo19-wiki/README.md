# Odoo 19 Community: архитектура и идеология

Этот каталог — последовательный учебный курс по **Odoo 19.0 Community**.

Цель — не научиться нажимать кнопки в отдельных приложениях, а понять Odoo настолько глубоко, чтобы видеть:

- устройство платформы;
- модульную композицию;
- ORM и model semantics;
- границы data/UI/security layers;
- способы штатного расширения;
- нативную ERP-логику предметных моделей и сквозных процессов.

## Принцип курса

Мы изучаем Odoo **снизу вверх** и не начинаем с Sales, Purchase, Project или Inventory.

Пока не понятны platform, modules, ORM и extension semantics, изучение приложений даёт знание интерфейса, но не архитектуры.

Единый метод и порядок курса зафиксированы в [00-method.md](00-method.md). README только отображает эту структуру.

## Источники

Используются только официальные материалы Odoo 19.0:

- Odoo Documentation;
- Developer tutorials;
- Developer reference;
- Administration documentation.

Наличие функции в общей документации Odoo не считается само по себе доказательством её доступности в Community.

## Маркировка утверждений

- **[ODOO]** — непосредственно следует из официальной документации Odoo 19.0;
- **[ВЫВОД]** — логическое следствие документированных механизмов;
- **[ERP-КЛАССИФИКАЦИЯ]** — внешний ERP-термин для анализа бизнес-смысла, а не внутренний тип Odoo.

## Терминология

Основные понятия нормализованы в [глоссарии](glossary.md).

В обычном русском тексте используем:

- модель (`model`);
- запись (`record`);
- набор записей (`recordset`);
- поле (`field`);
- модуль (`module`, addon);
- приложение (`App`);
- окружение (`Environment`).

Технические API names не переводятся.

## Карта курса

### 00. Метод

- [Метод и правила курса](00-method.md)
- [Глоссарий](glossary.md)

### 01. Платформа

- [Урок 1. Что такое Odoo: архитектурный фундамент](01-platform/01-architecture-foundations.md)
- [Урок 2. Модульная система: addons path, manifest, dependencies и lifecycle](01-platform/02-module-system.md)
- [Урок 3. Загрузка модулей: Python package, imports и per-database model construction](01-platform/03-module-loading.md)
- [Урок 4. ORM Core: Model, recordset, Environment, search и CRUD](01-platform/04-orm-core.md)

### Следующие согласованные уровни

```text
05  Fields
06  Relations
07  Computed / related / onchange / constraints / recomputation
08  Module data / external IDs / noupdate
09  Security
10  Actions / menus / views
11  Inheritance / extension / mixins
12  Multi-company
13  Advanced ORM
14  HTTP / frontend
15  Testing / upgrades / deployment
16+ Shared business models / domain applications / end-to-end ERP
```

Следующие уроки добавляются только после проверки предыдущего уровня на полноту, достоверность, последовательность и отсутствие скрытых зависимостей.

## Главный вопрос курса

После изучения системы на любой новый business question мы должны уметь рассуждать не от интерфейса, а от архитектуры:

```text
Какой бизнес-смысл у объекта?
        ↓
Есть ли штатная model Odoo?
        ↓
Какой module создаёт или расширяет её?
        ↓
Какие relations и constraints заложены в модели?
        ↓
Какая часть поведения относится к model/data,
а какая — к view/action/security/mixin/context?
        ↓
Как эта model участвует в сквозном процессе Odoo?
```

## Официальные отправные точки

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- Server Framework 101: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- Data Files: https://www.odoo.com/documentation/19.0/developer/reference/backend/data.html
- Security: https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html
- Actions: https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html
