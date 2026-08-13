# Coverage map Odoo 19.0

Цель этого файла — защищать курс не от дублей, а от **пропусков**.

Он сопоставляет major official documentation surfaces с lesson owners и статусом покрытия.

Статусы:

- `covered` — owner-урок существует и прошёл аудит;
- `in progress` — owner существует, материал ещё развивается;
- `planned` — owner назначен, урок ещё не создан;
- `intentionally deferred` — тема сознательно отложена до prerequisites;
- `out of scope` — явно исключено baseline scope.

> Coverage map не заменяет official documentation index. Перед завершением каждого крупного блока он сверяется с актуальным Odoo 19.0 Documentation.

## Governance / architecture

| Official surface / concept | Lesson | Status |
|---|---|---|
| Server Framework 101 — Architecture Overview | `ARCH-01` | covered |
| Server Framework 101 — A New Application / addon skeleton | `ARCH-02` | covered |
| Module manifests | `ARCH-02` | covered |
| CLI — addons path / module lifecycle / server-wide modules | `ARCH-02`, aspect `RUN-06` | in progress |
| Python package / import chain | `ARCH-03` | in progress |
| Client/server request and RPC execution boundary | `ARCH-04` | planned |

## Backend / ORM Reference

| Official surface / concept | Lesson | Status |
|---|---|---|
| Model / TransientModel / AbstractModel | `ORM-01` | covered |
| records / recordsets / singleton | `ORM-01` | covered |
| Environment — basic semantics | `ORM-01` | covered |
| browse / exists / ensure_one / search / domain basics | `ORM-01` | covered |
| create / read / write / unlink | `ORM-01` | covered |
| backend model registry / per-database effective model composition | `ORM-02` | planned |
| field classes / attributes / storage / automatic fields | `ORM-03` | planned |
| Many2one / One2many / Many2many / commands | `ORM-04` | planned |
| computed / related / inverse / depends / constraints | `ORM-05` | planned |
| transactions / commit / rollback discipline | `ORM-06` | planned |
| Environment security/company transformations | `SEC-01`, `EXT-04` | planned |
| cache / prefetch / flush / raw SQL / invalidation | `RUN-01` | planned |
| performance guidance | `RUN-01` | planned |
| advanced/implementation caveats | `RUN-01` or reference notes | intentionally deferred |

## Module data / actions / security

| Official surface / concept | Lesson | Status |
|---|---|---|
| Data files — XML / CSV | `DATA-01` | planned |
| external IDs / XML IDs | `DATA-01` | planned |
| `noupdate` / data update semantics | `DATA-01` | planned |
| Odoo module master data / demo data | `DATA-01` | planned |
| backend actions | `UI-01` | planned |
| menus | `UI-01` | planned |
| access rights / groups / record rules / field access | `SEC-01` | planned |
| unsafe public methods / RPC security boundary | `SEC-01`, prerequisite `ARCH-04` | planned |

## User interface / frontend

| Official surface / concept | Lesson | Status |
|---|---|---|
| view records | `UI-02` | planned |
| view architectures: form/list/search and common semantics | `UI-02` | planned |
| `@api.onchange` / pseudo-record | `UI-03` | planned |
| HTTP controllers / routes | `RUN-02` | planned |
| frontend RPC / ORM services | `RUN-03` | planned |
| Owl components | `RUN-03` | planned |
| frontend registries | `RUN-03` | planned |
| frontend services / hooks / patching | `RUN-03` | planned |
| assets | `RUN-03` | planned |
| QWeb / templates | `RUN-03` or dedicated aspect if scope expands | planned |

## Extension mechanisms

| Official surface / concept | Lesson | Status |
|---|---|---|
| model inheritance / `_inherit` / `_inherits` | `EXT-01` | planned |
| view inheritance | `EXT-02` | planned |
| mixins | `EXT-03` | planned |
| multi-company guidelines / company-dependent values / consistency | `EXT-04` | planned |

## Runtime / engineering

| Official surface / concept | Lesson | Status |
|---|---|---|
| Coding Guidelines — transactional discipline | `ORM-06` | planned |
| Performance reference | `RUN-01` | planned |
| HTTP reference | `RUN-02` | planned |
| Testing framework | `RUN-04` | planned |
| Upgrade scripts / upgrade utils | `RUN-05` | planned |
| CLI / workers / multiprocessing | `RUN-06` | planned |
| source/package installation | `RUN-06` | planned |
| Odoo Online-specific operations | separate future context | out of scope |
| Odoo.sh-specific operations | separate future context | out of scope |

## Reports / other backend reference surfaces

| Official surface / concept | Lesson | Status |
|---|---|---|
| QWeb reports / report actions | owner to be assigned before UI/report block closes | planned |
| standard-module developer reference | business/domain phase | intentionally deferred |
| external JSON-2 API | integration/runtime phase if needed | intentionally deferred |
| legacy external XML-RPC / JSON-RPC APIs | historical/version note only unless needed | intentionally deferred |

## Business layer

| Official surface | Lesson | Status |
|---|---|---|
| shared/system business models (`res.partner`, `res.users`, `res.company`, products, etc.) | `10-business-model/` owners | planned |
| user documentation by domain application | `11-domain-apps/` | planned |
| cross-application end-to-end processes | `12-end-to-end/` | planned |
| exhaustive Community addon inventory | requires governance source-policy change if source manifests become necessary | intentionally deferred |

## Правило завершения блока

Перед тем как считать крупный блок завершённым:

1. открыть актуальный official Odoo 19.0 documentation index;
2. проверить relevant sections;
3. каждой relevant теме назначить status;
4. неизвестные пробелы не оставлять без строки;
5. новые owners сначала внести в concept ownership и course DAG.