# Метод изучения Odoo 19

## Scope

Курс изучает **Odoo 19.0 Community** как платформу и ERP-систему: архитектуру, ORM, данные, security, UI, расширение, runtime и затем штатные бизнес-модели и сквозные процессы.

Официальные custom-module examples допустимы как учебный инструмент для понимания платформы. Они **не означают**, что custom addons входят в канонический набор Odoo Community, который будет разбираться в предметной части курса.

## Три типа утверждений

- **[ODOO]** — утверждение непосредственно следует из официальной документации Odoo 19.0;
- **[ВЫВОД]** — логическое следствие документированных механизмов, но не буквальная формулировка Odoo;
- **[ERP-КЛАССИФИКАЦИЯ]** — внешний термин для анализа бизнес-смысла, а не внутренний тип ORM или platform object Odoo.

Если термин используется самой Odoo в другом значении, это значение обязательно квалифицируется. Например, **Odoo module master data** и **ERP master data** — разные контексты одного выражения.

## Один владелец понятия

Каждое фундаментальное понятие имеет ровно один основной owner-урок. Другие материалы могут:

- **preview** — кратко обозначить понятие до owner-урока;
- **use** — использовать уже определённое понятие;
- но не давать второе независимое определение.

Карта владельцев хранится в [concept-ownership.md](concept-ownership.md).

## Полнота урока

Каждый урок должен покрывать:

- что это в терминах Odoo;
- зачем механизм существует;
- где находится в архитектуре;
- с чем связан;
- что пользователь или разработчик видит;
- что из этого нельзя заключать;
- минимальную mental model;
- official sources;
- prerequisites;
- owned concepts;
- previewed concepts;
- consciously deferred topics;
- edition relevance/status;
- дату последней проверки.

## Metadata урока

В начале каждого урока используется блок:

```text
Версия: Odoo 19.0
Проверено: YYYY-MM-DD
Prerequisites: ...
Владеет понятиями: ...
Preview: ...
Отложено: ...
Edition scope: ...
```

Это не бюрократия: metadata позволяет сразу видеть скрытые зависимости и не допускать повторного определения понятий.

## Источниковая дисциплина

Используются только официальные материалы Odoo 19.0. Приоритет источников и правила разрешения расхождений описаны в [source-policy.md](source-policy.md).

Наличие feature в общей документации Odoo **не доказывает** её доступность в Community. Edition status фиксируется отдельно в [edition-ledger.md](edition-ledger.md).

## Последовательность курса

```text
00  Governance

01  Architecture
    01 Architectural foundation
    02 Module system
    03 Module loading / backend model registry context

02  ORM
    01 ORM Core
    02 Fields
    03 Relations
    04 Computed / related / inverse / constraints
    05 Transactions

03  Data, security and UI
    01 Module data / external IDs / noupdate
    02 Security
    03 Actions and menus
    04 Views
    05 Onchange

04  Extension
    01 Model inheritance
    02 View inheritance
    03 Mixins
    04 Multi-company

05  Advanced runtime
    01 Advanced ORM: cache / prefetch / performance /
       flush / raw SQL / invalidation
    02 HTTP / RPC
    03 Web client / Owl / assets
    04 Testing
    05 Upgrades / migrations
    06 Deployment / workers / operations

10  Shared business model
11  Domain applications
12  End-to-end ERP flows
```

### Почему `onchange` после Views

`@api.onchange` — form-view/client mechanism: Odoo вызывает onchange в form view на pseudo-record. Поэтому его нельзя полноценно изучать до понимания form views.

### Почему Transactions до Advanced ORM

Raw SQL, flush, invalidation и часть runtime behavior требуют предварительного понимания framework-managed transactional context. Поэтому transactions не прячутся в конце Advanced ORM.

## Data-driven принцип

Архитектурный фундамент курса должен сохранить официальный принцип: Odoo в значительной степени data-driven. Views, actions, menus, security и другие части системы во многих случаях представлены records/data. Подробности принадлежат соответствующим owner-урокам, но сам принцип вводится в Architecture.

## Не учим Odoo через меню

Курс не делает вывод:

```text
есть меню → существует независимая предметная система
```

Правильное направление анализа:

```text
module graph
    ↓
models / records / data
    ↓
security + actions + views
    ↓
user-facing interface
```

## Не подгоняем Odoo под заранее выбранный процесс

До предметной части курса сначала устанавливается нативная семантика платформы и штатных моделей. Только после этого оценивается применимость к конкретным бизнес-процессам.

## Правило остановки

Новый урок не добавляется, если предыдущий уровень:

- использует необъяснённые prerequisites;
- имеет два владельца одного понятия;
- содержит неподтверждённый edition claim;
- смешивает Odoo terminology с нашей ERP-классификацией;
- выдаёт implementation caveat за архитектурный фундамент.