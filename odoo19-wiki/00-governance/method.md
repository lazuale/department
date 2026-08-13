# Метод изучения Odoo 19

## Scope

Курс изучает **Odoo 19.0 Community в self-hosted контексте** как платформу и ERP-систему: архитектуру, ORM, данные, security, UI, расширение, runtime и затем штатные business models и end-to-end процессы.

В baseline не включаются автоматически:

- Enterprise-only functionality;
- Odoo Online-specific behavior;
- Odoo.sh-specific behavior;
- SaaS branches `saas-19.x`, если утверждение должно описывать именно Odoo 19.0.

Official custom-module examples допустимы как учебный инструмент для понимания платформы. Они **не означают**, что custom addons входят в канонический Community ERP baseline.

## Три типа утверждений

- **[ODOO]** — непосредственно следует из official Odoo 19.0 documentation;
- **[ВЫВОД]** — логическое следствие документированных механизмов, но не буквальная формулировка Odoo;
- **[ERP-КЛАССИФИКАЦИЯ]** — внешний термин для анализа business semantics, а не внутренний type/model/class Odoo.

Если термин используется самой Odoo в другом значении, это значение всегда квалифицируется. Например, **Odoo module master data** и **ERP master data** — разные контексты.

## Stable lesson IDs

Каждый учебный материал получает стабильный ID, не зависящий от имени файла:

```text
ARCH-01
ORM-01
DATA-01
SEC-01
UI-01
EXT-01
RUN-01
BUS-01
```

`GOV` — зарезервированный **root pseudo-node**. Это не lesson и не учебная тема: он означает, что global governance уже действует. Первый урок может иметь `Prerequisites: GOV`; остальные prerequisites должны ссылаться на stable lesson IDs.

Это позволяет проверять dependency graph и менять файловую структуру без разрушения логики курса.

## Владение понятиями: canonical owner и aspect owners

Каждое фундаментальное понятие имеет **одного canonical owner** — место первого полного определения.

Другие уроки могут владеть только явно названным **aspect** этого понятия.

Пример:

```text
Environment basic semantics → ORM-01
Environment security aspect → SEC-01
Environment company aspect  → EXT-04
Environment cache aspect    → RUN-01
```

Разрешены три режима использования:

- **preview** — краткое обозначение до canonical owner;
- **use** — использование после canonical owner;
- **aspect** — подробное развитие только назначенного аспекта.

Запрещено давать второе независимое canonical definition. Карта хранится в [concept-ownership.md](concept-ownership.md).

## Полнота и coverage

Concept ownership защищает от дублей, но не от пропусков. Поэтому официальный documentation surface сопоставляется с курсом в [coverage-map.md](coverage-map.md).

Для каждого крупного official documentation section используется статус:

- `covered`;
- `in progress`;
- `planned`;
- `intentionally deferred`;
- `out of scope`.

Раздел курса нельзя считать завершённым, пока соответствующая часть coverage map не сверена с актуальным индексом Odoo 19.0 Documentation.

## Metadata урока

В начале каждого урока используется блок:

```text
Lesson ID: ARCH-01
Версия: Odoo 19.0
Проверено: YYYY-MM-DD
Prerequisites: GOV
Canonical owner: ...
Aspect owner: ...
Preview: ...
Отложено: ...
Edition scope: ...
Sources: S1, S2, ...
```

`Canonical owner` и `Aspect owner` могут быть пустыми, если урок только использует уже определённые concepts.

## Source traceability

Claims `[ODOO]` по возможности маркируются source ID урока:

```text
[ODOO][S1]
```

Внизу урока:

```text
S1 — ORM API — <official URL>
S2 — Server Framework 101 — <official URL>
```

Это не заменяет смысловую проверку источника, но делает реверификацию claims возможной без археологии по длинному списку ссылок.

Полная policy — в [source-policy.md](source-policy.md).

## Community / Enterprise

Наличие страницы в общей Documentation **не доказывает** Community availability. Edition evidence хранится отдельно в [edition-ledger.md](edition-ledger.md).

### Ограничение docs-only

При текущем governance официальная документация является единственным источником фактов.

Это позволяет глубоко и последовательно изучать documented platform semantics, но **не гарантирует исчерпывающий технический whitelist всех Community addons**, потому что общая документация не является полной CE/EE entitlement matrix.

Если когда-либо потребуется доказать полный module inventory Community, допуск official Odoo source/manifests должен быть отдельным изменением governance.

## Последовательность курса

```text
00  Governance

01  Architecture
    ARCH-01  Architectural foundation
    ARCH-02  Module system
    ARCH-03  Python package / import chain
    ARCH-04  Request / RPC execution boundary

02  ORM
    ORM-01   ORM Core
    ORM-02   Per-database model registry / model composition
    ORM-03   Fields
    ORM-04   Relations
    ORM-05   Computed / related / inverse / constraints
    ORM-06   Transactions

03  Data, security and UI
    DATA-01  Module data / external IDs / noupdate
    SEC-01   Security
    UI-01    Actions and menus
    UI-02    Views
    UI-03    Onchange

04  Extension
    EXT-01   Model inheritance
    EXT-02   View inheritance
    EXT-03   Mixins
    EXT-04   Multi-company

05  Runtime
    RUN-01   Advanced ORM: cache / prefetch / performance /
             flush / raw SQL / invalidation
    RUN-02   HTTP / controllers / deeper RPC
    RUN-03   Web client / Owl / frontend registries / assets
    RUN-04   Testing
    RUN-05   Upgrades / migrations
    RUN-06   Deployment / workers / server-wide modules / operations

10  Shared business model
11  Domain applications
12  End-to-end ERP flows
```

### Почему ARCH-03 больше не владеет ORM registry

Python import chain можно понять до ORM Core. Но `model instance`, effective model class и per-database model mapping требуют уже определённых `Model`/recordset concepts. Поэтому registry/model-composition semantics перенесены в `ORM-02`.

### Почему ARCH-04 ранний

Security public methods, framework-managed transactions и onchange используют request/RPC execution boundary. Поэтому базовая client/server invocation model вводится до этих тем, а подробные HTTP/controllers/frontend details остаются в RUN-02/RUN-03.

### Почему onchange после Views

`@api.onchange` — form-view/client mechanism на pseudo-record. Он не должен изучаться до формы и request boundary.

### Почему Transactions до Advanced ORM

Raw SQL, flush, invalidation и часть runtime behavior требуют понимания framework-managed transactional context.

## Data-driven принцип

Архитектурный фундамент сохраняет официальный принцип: Odoo в значительной степени data-driven. Views, actions, menus, security и другие части системы во многих случаях представлены records/data. Их полная semantics принадлежит отдельным owners.

## Не учим Odoo через меню

Курс не делает вывод:

```text
есть menu → существует независимая предметная система
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

До предметной части сначала устанавливается native semantics платформы и штатных models. Только после этого оценивается применимость к business processes.

## Правило остановки

Новый урок не добавляется, если предыдущий baseline:

- использует prerequisite, отсутствующий в stable-ID graph;
- имеет два canonical owners одного понятия;
- содержит неподтверждённый edition claim;
- смешивает Odoo terminology с ERP classification;
- выдаёт implementation caveat за архитектурный фундамент;
- оставляет relevant official documentation section без статуса в coverage map.