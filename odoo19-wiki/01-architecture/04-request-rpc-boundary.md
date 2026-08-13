# ARCH-04. Request / RPC execution boundary

> Lesson ID: `ARCH-04`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ARCH-01`  
> Canonical owner: request / RPC execution boundary  
> Aspect owner: —  
> Preview: public methods; transactions; controller routes; frontend ORM/RPC services  
> Отложено: HTTP routing; controller API; authentication details; transaction internals; frontend services; external API product semantics  
> Edition scope: platform client/server semantics; не entitlement claim конкретного feature  
> Sources: `S1`–`S6`

## Цель

Понять одну раннюю архитектурную границу:

```text
client / caller
      │
      ▼
request / RPC boundary
      │
      ▼
server-side Odoo execution
```

Этот урок нужен до Security, Transactions и Onchange, но не пытается заранее преподавать controllers или frontend framework.

---

## 1. Odoo — client/server system

**[ODOO][S1]** Odoo architecture разделяет presentation tier и Python logic tier.

**[ODOO][S2]** Frontend documentation предоставляет RPC mechanism для запросов web client к server; для работы с models frontend рекомендует ORM service, а low-level `rpc` service — прежде всего для controller routes.

Минимально:

```text
Web client / other caller
        │
        ▼
request / RPC
        │
        ▼
Odoo server execution
```

**[ВЫВОД]** Business method invocation в Odoo нельзя мыслить только как локальный Python function call: значимая часть user interaction пересекает client/server boundary.

---

## 2. RPC здесь — архитектурный термин, не название одного external API

В Odoo 19 существуют разные RPC/request contexts.

Например:

- web client вызывает server-side behavior;
- frontend `orm`/`rpc` services работают через network boundary;
- controllers имеют routes;
- external integration APIs имеют отдельную version/product semantics.

**[ODOO][S6]** External XML-RPC/JSON-RPC endpoints в Odoo 19 имеют собственную deprecation/version story и не должны автоматически смешиваться с controller JSON-RPC or generic client/server request semantics.

**[ВЫВОД]** В этом курсе `request/RPC boundary` — общий execution concept. Конкретный external API изучается отдельно, если понадобится.

---

## 3. Public server methods являются security boundary

Security details принадлежат `SEC-01`, но архитектурный факт нужен заранее.

**[ODOO][S3]** Security Reference предупреждает: public methods могут быть вызваны через RPC с caller-controlled parameters; methods с именем, начинающимся `_`, не вызываются таким же образом из action button/external API.

**[ВЫВОД]** Server method, доступный через request/RPC boundary, должен рассматривать caller input как недоверенный. Полные ACL/record-rule semantics будут позже.

---

## 4. Framework-managed transaction начинается вокруг execution boundary

Transaction semantics принадлежат `ORM-06`.

**[ODOO][S4]** Coding Guidelines описывает framework-provided transactional context для RPC calls: cursor создаётся для call, при успешном завершении framework делает commit, при exception — rollback.

Здесь это только preview:

```text
RPC/request call
      │
      ▼
server execution inside framework transaction
      │
   success / exception
      │
      └── exact commit/rollback discipline → ORM-06
```

**[ВЫВОД]** Transactions в Odoo нельзя изучать как независимый PostgreSQL topic без request execution context.

---

## 5. Почему Onchange зависит от этой границы

Onchange owner — `UI-03`.

**[ODOO][S5]** ORM Reference описывает `@api.onchange` как form-view mechanism на pseudo-record; assignments возвращаются client.

**[ВЫВОД]** Onchange — не просто server-side helper method. Он участвует в client/server interaction и поэтому требует сначала понимать request boundary и form views.

---

## 6. Что остаётся на потом

Этот урок не определяет:

- route decorators;
- HTTP authentication modes;
- JSON payload formats;
- frontend service lifecycle;
- external JSON-2 API;
- exact transaction/savepoint semantics;
- ACL/record rules;
- onchange pseudo-record details.

Owners:

```text
Transactions       → ORM-06
Security           → SEC-01
Onchange           → UI-03
HTTP/controllers   → RUN-02
Frontend services  → RUN-03
```

---

## Минимальная модель

```text
USER / CLIENT / CALLER
        │
        ▼
REQUEST / RPC BOUNDARY
        │
        ▼
SERVER-SIDE EXECUTION
        │
        ├── security aspect → SEC-01
        ├── transaction aspect → ORM-06
        ├── controllers → RUN-02
        └── frontend interaction → RUN-03 / UI-03
```

## Что нельзя заключать

- RPC = только legacy external XML-RPC — нет;
- public method parameters можно автоматически доверять — нет;
- после ARCH-04 уже изучены ACL/record rules — нет;
- после ARCH-04 уже изучена transaction implementation — нет;
- onchange является обычным database-record CRUD method — нет.

## Контрольные вопросы

1. Что означает request/RPC boundary в архитектуре курса?
2. Почему это шире, чем legacy external RPC API?
3. Почему public methods создают security concern?
4. Почему Transactions зависят от execution boundary?
5. Почему Onchange нельзя полноценно изучить как изолированный Python decorator?

## Официальные источники

- `S1` — Architecture Overview  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- `S2` — Frontend Services / RPC service  
  https://www.odoo.com/documentation/19.0/developer/reference/frontend/services.html
- `S3` — Security Reference  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html
- `S4` — Coding Guidelines  
  https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html
- `S5` — ORM API (`@api.onchange`)  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- `S6` — External RPC API  
  https://www.odoo.com/documentation/19.0/developer/reference/external_rpc_api.html