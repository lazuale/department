# Владение понятиями

Этот файл предотвращает повторные canonical definitions и фиксирует, где разрешено развивать отдельные aspects понятия.

## Режимы

- **canonical owner** — первое полное определение concept;
- **aspect owner** — подробное определение только явно названного аспекта;
- **preview** — краткое обозначение до canonical owner;
- **use** — использование после canonical owner.

## Карта

| Понятие | Canonical owner | Aspect owners |
|---|---|---|
| three-tier architecture | `ARCH-01` | — |
| deployment / runtime / process distinction | `ARCH-01` | workers/multiprocessing/server operations → `RUN-06` |
| multi-database property | `ARCH-01` | per-database model mapping → `ORM-02`; company semantics → `EXT-04` |
| data-driven architecture | `ARCH-01` | module data → `DATA-01`; security data → `SEC-01`; UI records → `UI-01`/`UI-02` |
| module / addon | `ARCH-02` | server-wide modules → `RUN-06` |
| App | `ARCH-02` | domain-app semantics → future `11-domain-apps/` |
| `addons_path` | `ARCH-02` | deployment configuration → `RUN-06` |
| manifest / `__manifest__.py` | `ARCH-02` | data entries → `DATA-01`; assets → `RUN-03` |
| module dependencies / `auto_install` | `ARCH-02` | inheritance use → `EXT-01`; integration patterns in domain apps → future business layer |
| module lifecycle | `ARCH-02` | upgrades/migrations → `RUN-05`; operational CLI → `RUN-06` |
| server-wide modules / `--load` | `ARCH-02` | runtime/deployment details → `RUN-06` |
| Python package / `__init__.py` import chain | `ARCH-03` | — |
| request / RPC execution boundary | `ARCH-04` | transactions → `ORM-06`; public-method security → `SEC-01`; controllers → `RUN-02`; web client services → `RUN-03` |
| Model / TransientModel / AbstractModel | `ORM-01` | model inheritance/composition → `ORM-02`/`EXT-01` |
| record / recordset / singleton | `ORM-01` | cache/prefetch/performance → `RUN-01` |
| Environment basic semantics | `ORM-01` | security → `SEC-01`; company → `EXT-04`; cache/recompute → `RUN-01` |
| browse / exists / ensure_one | `ORM-01` | — |
| search / basic domain semantics | `ORM-01` | relation traversal → `ORM-04`; UI domains → `UI-01`/`UI-02` |
| CRUD | `ORM-01` | security checks → `SEC-01`; transactions → `ORM-06`; performance → `RUN-01` |
| backend model registry context | `ORM-02` | exact runtime/deployment lifecycle only if documented later → `RUN-06` |
| per-database model construction / effective model class | `ORM-02` | inheritance mechanics → `EXT-01` |
| field taxonomy / metadata / storage | `ORM-03` | field access security → `SEC-01`; company-dependent fields → `EXT-04` |
| Many2one / One2many / Many2many / commands | `ORM-04` | relational domain traversal → `ORM-04` |
| computed / related / inverse / depends / constraints | `ORM-05` | recomputation/cache consistency → `RUN-01` |
| transaction / commit / rollback discipline | `ORM-06` | raw SQL/flush/savepoints/performance → `RUN-01` |
| Odoo module master data / demo data | `DATA-01` | views/actions/security as particular records → their owners |
| external ID / XML ID / `noupdate` | `DATA-01` | upgrades → `RUN-05` |
| user as security principal / groups / ACL / record rule / field access | `SEC-01` | `res.users` model semantics → future shared business/system model layer |
| action / menu | `UI-01` | action-specific domain/context usage → `UI-01` |
| view / form/list/search view | `UI-02` | view inheritance → `EXT-02` |
| onchange / pseudo-record | `UI-03` | request/client interaction uses `ARCH-04`/`RUN-03` |
| model inheritance / `_inherit` / `_inherits` | `EXT-01` | per-database effective class uses `ORM-02` |
| view inheritance | `EXT-02` | — |
| mixins | `EXT-03` | domain-specific mixins later |
| company-context / multi-company semantics | `EXT-04` | `res.company` model semantics → future shared business/system model layer |
| cache / prefetch / flush / raw SQL / invalidation | `RUN-01` | — |
| HTTP / controllers / deeper RPC | `RUN-02` | request boundary canonical definition → `ARCH-04` |
| web client / Owl / frontend registries / assets | `RUN-03` | — |
| testing | `RUN-04` | transactional test context uses `ORM-06` |
| upgrades / migrations | `RUN-05` | module lifecycle uses `ARCH-02`; external IDs uses `DATA-01` |
| workers / multiprocessing / deployment operations | `RUN-06` | process distinction canonical definition → `ARCH-01` |
| `res.users` as data model | future shared business/system model owner | security-principal aspect → `SEC-01` |
| `res.company` as data model | future shared business/system model owner | company-context aspect → `EXT-04` |
| ERP master data / transaction / reference-data classification | future `10-business-model/` intro owner | concrete domain classification in later business lessons |

## Правило конфликтов

Если новый урок затрагивает существующий concept:

1. проверить canonical owner;
2. определить, это `use` или новый aspect;
3. если aspect уже имеет owner — ссылаться на него;
4. если нового aspect owner ещё нет — сначала изменить эту карту;
5. не копировать canonical definition.

Если canonical owner выбран неправильно, сначала меняются governance/DAG, затем тексты.