# Coverage map Odoo 19.0

Цель этого файла — защищать курс не от дублей, а от **пропусков**.

Он сопоставляет major official documentation surfaces с lesson owners и статусом покрытия.

## Допустимые статусы

Используются только пять значений:

- `covered` — owner-урок существует и проверен;
- `in progress` — owner существует, material ещё развивается;
- `planned` — owner назначен, lesson ещё не создан;
- `intentionally deferred` — тема сознательно отложена до prerequisites/следующей фазы;
- `out of scope` — явно исключено baseline scope.

Нельзя создавать произвольные статусы вроде `covered at architecture level`. Частичное или aspect-specific coverage описывается в колонке `Owner / scope`.

> Coverage map не заменяет official documentation index. Перед завершением каждого крупного блока он сверяется с актуальным Odoo 19.0 Documentation.

## Governance / architecture

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| Server Framework 101 — Architecture Overview | `ARCH-01` | covered |
| Server Framework 101 — A New Application / addon skeleton | `ARCH-02` | covered |
| Module manifests | `ARCH-02` | covered |
| CLI — addons path / database-bound lifecycle / `--load` distinction | architecture aspect `ARCH-02` | covered |
| CLI — workers / multiprocessing / detailed runtime `--load` operation | runtime aspect `RUN-06` | planned |
| Python package / import chain | `ARCH-03` | covered |
| Client/server request and RPC execution boundary | `ARCH-04` | covered |

## Backend / ORM Reference

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| Model / TransientModel / AbstractModel | `ORM-01` | covered |
| records / recordsets / singleton | `ORM-01` | covered |
| Environment — basic semantics | `ORM-01` | covered |
| browse / exists / ensure_one / search / domain basics | `ORM-01` | covered |
| create / read / write / unlink | `ORM-01` | covered |
| backend model registry / per-database effective model composition | `ORM-02` | covered |
| model technical metadata (`_name`, `_description`, `_auto`, `_table`, `_register`, `_log_access`, `_rec_name`, `_order`, hierarchy/UI-related options) | `ORM-03` | covered |
| SQL/schema declarations: `Constraint`, `Index`, `UniqueIndex` and related model-storage semantics | `ORM-03` | covered |
| inheritance-specific model attributes (`_inherit`, `_inherits`) | `EXT-01` | planned |
| multi-company model attribute (`_check_company_auto`) | `EXT-04` | planned |
| field classes / attributes / automatic fields / field storage semantics | `ORM-04` | planned |
| Many2one / One2many / Many2many / commands | `ORM-05` | planned |
| computed / related / inverse / depends / Python constraints | `ORM-06` | planned |
| RPC transaction / commit / rollback discipline | `ORM-07`, prerequisite `ARCH-04` | planned |
| Environment security/company transformations | `SEC-01`, `EXT-04` aspects | planned |
| cache / prefetch / flush / raw SQL / invalidation | `RUN-01` | planned |
| performance guidance | `RUN-01` | planned |
| advanced/implementation caveats | `RUN-01` or dedicated reference notes | intentionally deferred |

## Module data / actions / security

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| Data files — XML / CSV | `DATA-01` | planned |
| external IDs / XML IDs | `DATA-01` | planned |
| `noupdate` / data update semantics | `DATA-01` | planned |
| Odoo module master data / demo data | `DATA-01` | planned |
| backend actions | `UI-01` | planned |
| menus | `UI-01` | planned |
| access rights / groups / record rules / field access | `SEC-01` | planned |
| unsafe public methods / RPC security aspect | `SEC-01`, prerequisite `ARCH-04` | planned |

## User interface / frontend

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| view records | `UI-02` | planned |
| view architectures: form/list/search and common semantics | `UI-02` | planned |
| `@api.onchange` / pseudo-record | `UI-03`, prerequisites `ARCH-04`, `UI-02`, `ORM-04` | planned |
| QWeb reports / report actions | `UI-04` | planned |
| generic/frontend QWeb templates | `RUN-03` aspect | planned |
| HTTP controllers / routes | `RUN-02` | planned |
| frontend RPC / ORM services | `RUN-03`, boundary canonical owner `ARCH-04` | planned |
| Owl components | `RUN-03` | planned |
| frontend registries | `RUN-03` | planned |
| frontend services / hooks / patching | `RUN-03` | planned |
| assets | `RUN-03` | planned |

## Extension mechanisms

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| model inheritance / `_inherit` / `_inherits` | `EXT-01` | planned |
| view inheritance | `EXT-02` | planned |
| mixins | `EXT-03` | planned |
| multi-company guidelines / company-dependent values / consistency | `EXT-04` | planned |

## Runtime / engineering

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| Coding Guidelines — RPC transactional discipline | `ORM-07`, prerequisite `ARCH-04` | planned |
| generic HTTP request/controller transaction aspect | `RUN-02`, only when directly documented | planned |
| Performance reference | `RUN-01` | planned |
| HTTP reference | `RUN-02` | planned |
| Testing framework | `RUN-04` | planned |
| Upgrade scripts / upgrade utils | `RUN-05` | planned |
| CLI / workers / multiprocessing / server-wide runtime details | `RUN-06` | planned |
| source/package installation | `RUN-06` | planned |
| Odoo Online-specific operations | separate future context | out of scope |
| Odoo.sh-specific operations | separate future context | out of scope |

## Reports / other backend reference surfaces

| Official surface / concept | Owner / scope | Status |
|---|---|---|
| QWeb reports / report actions | `UI-04` | planned |
| standard-module developer reference | business/domain phase | intentionally deferred |
| external JSON-2 API | integration/runtime phase if needed | intentionally deferred |
| legacy external XML-RPC / JSON-RPC APIs | historical/version note only unless needed | intentionally deferred |

## Business layer

| Official surface | Owner / scope | Status |
|---|---|---|
| shared/system business models (`res.partner`, `res.users`, `res.company`, products, etc.) | future `10-business-model/` owners | planned |
| user documentation by domain application | future `11-domain-apps/` owners | planned |
| cross-application end-to-end processes | future `12-end-to-end/` owners | planned |
| exhaustive Community addon inventory | requires source-policy change if official source manifests become necessary | intentionally deferred |

## Правило завершения блока

Перед тем как считать крупный блок завершённым:

1. открыть актуальный official Odoo 19.0 documentation index;
2. проверить relevant sections;
3. каждой relevant теме назначить owner/scope и один допустимый status;
4. неизвестные пробелы не оставлять без строки;
5. новые owners сначала внести в `concept-ownership.md` и `course-dag.md`;
6. после Baseline B1 структурные изменения проверять по `baseline.md`.