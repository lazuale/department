# Владение понятиями

Этот файл предотвращает повторное определение фундаментальных понятий в разных уроках.

Статусы использования:

- **owner** — единственное место полного определения;
- **preview** — короткое обозначение до owner-урока;
- **use** — использование после определения.

## Карта

| Понятие | Owner |
|---|---|
| three-tier architecture | `01-architecture/01-architecture-foundations.md` |
| Odoo deployment / server runtime / process distinction | `01-architecture/01-architecture-foundations.md` |
| multi-database property | `01-architecture/01-architecture-foundations.md` |
| data-driven architecture | `01-architecture/01-architecture-foundations.md` |
| module / addon | `01-architecture/02-module-system.md` |
| App | `01-architecture/02-module-system.md` |
| `addons_path` | `01-architecture/02-module-system.md` |
| manifest / `__manifest__.py` | `01-architecture/02-module-system.md` |
| module dependencies / `auto_install` | `01-architecture/02-module-system.md` |
| module lifecycle | `01-architecture/02-module-system.md` |
| Python package / `__init__.py` import chain | `01-architecture/03-module-loading.md` |
| backend model registry context | `01-architecture/03-module-loading.md` |
| per-database model construction | `01-architecture/03-module-loading.md` |
| Model / TransientModel / AbstractModel | `02-orm/01-orm-core.md` |
| record / recordset / singleton | `02-orm/01-orm-core.md` |
| Environment basic semantics | `02-orm/01-orm-core.md` |
| browse / exists / ensure_one | `02-orm/01-orm-core.md` |
| search / basic domain semantics | `02-orm/01-orm-core.md` |
| CRUD | `02-orm/01-orm-core.md` |
| field taxonomy / metadata / storage | `02-orm/02-fields.md` |
| Many2one / One2many / Many2many / commands | `02-orm/03-relations.md` |
| computed / related / inverse / depends / constraints | `02-orm/04-derived-fields-constraints.md` |
| transaction / commit / rollback discipline | `02-orm/05-transactions.md` |
| Odoo module master data / demo data | `03-data-security-ui/01-module-data.md` |
| external ID / XML ID / `noupdate` | `03-data-security-ui/01-module-data.md` |
| user / group / ACL / record rule / field access | `03-data-security-ui/02-security.md` |
| action / menu | `03-data-security-ui/03-actions-menus.md` |
| view / form/list/search view | `03-data-security-ui/04-views.md` |
| onchange / pseudo-record | `03-data-security-ui/05-onchange.md` |
| model inheritance / `_inherit` / `_inherits` | `04-extension/01-model-inheritance.md` |
| view inheritance | `04-extension/02-view-inheritance.md` |
| mixins | `04-extension/03-mixins.md` |
| multi-company semantics | `04-extension/04-multi-company.md` |
| cache / prefetch / flush / raw SQL / invalidation | `05-runtime/01-advanced-orm.md` |
| HTTP / RPC | `05-runtime/02-http-rpc.md` |
| web client / Owl / frontend registries / assets | `05-runtime/03-web-client.md` |
| testing | `05-runtime/04-testing.md` |
| upgrades / migrations | `05-runtime/05-upgrades-migrations.md` |
| workers / multiprocessing / deployment operations | `05-runtime/06-deployment-workers.md` |
| ERP master data / transactions / reference data classification | `10-business-model/` intro owner, когда предметный слой будет создан |

## Правило конфликтов

Если новый урок хочет объяснить понятие, уже имеющее owner:

1. не копировать определение;
2. сослаться на owner;
3. описывать только новый аспект, принадлежащий текущему уроку.

Если выясняется, что текущий owner выбран неправильно, сначала меняется эта карта и структура курса, затем тексты.