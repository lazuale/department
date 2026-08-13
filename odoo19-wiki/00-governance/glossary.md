# Глоссарий

Глоссарий — **индекс**, а не второй учебник. Полное canonical definition находится у owner из [concept-ownership.md](concept-ownership.md).

| Термин | Коротко | Canonical owner |
|---|---|---|
| Odoo deployment / runtime | Запущенная серверная среда Odoo; не приравнивается к одному process | `ARCH-01` |
| database / база данных | PostgreSQL database в Odoo runtime context | `ARCH-01` |
| module / addon / модуль | Техническая единица расширения Odoo | `ARCH-02` |
| server-wide module | Module, загружаемый через server-wide `--load`; отличается от большинства database-bound installed addons | `ARCH-02`, runtime aspect `RUN-06` |
| App / приложение | User-facing module, помеченный как application | `ARCH-02` |
| `addons_path` | Каталоги поиска addons | `ARCH-02` |
| manifest | `__manifest__.py`, metadata/dependencies/data declaration module | `ARCH-02` |
| Python import chain | Импорты addon package через `__init__.py` | `ARCH-03` |
| request / RPC execution boundary | Граница client/server invocation; deeper controller/frontend semantics позже | `ARCH-04` |
| model / модель | ORM-модель Odoo | `ORM-01` |
| record / запись | Конкретная запись модели; представляется singleton recordset | `ORM-01` |
| recordset / набор записей | Коллекция records одной model; основной рабочий объект ORM | `ORM-01` |
| Environment / окружение | Runtime context ORM operation | `ORM-01` |
| backend model registry | Backend per-database model mapping/context | `ORM-02` |
| field / поле | ORM field; не UI widget | `ORM-03` |
| relational field | `Many2one`, `One2many`, `Many2many` | `ORM-04` |
| domain | Декларативное ORM search/filter expression | `ORM-01`; relational/UI aspects — другие owners |
| transaction | Framework-managed database transaction context | `ORM-06` |
| Odoo module master data | Official Odoo term для data, устанавливаемых с module; не ERP classification | `DATA-01` |
| ERP master data | Будущая business classification курса | future business-model owner |
| external ID / XML ID | Устойчивый identifier module-data record | `DATA-01` |
| user as security principal | Security identity/permissions aspect пользователя | `SEC-01` |
| `res.users` | Odoo data model пользователя; business/system semantics изучаются отдельно | future business-model owner |
| ACL / access right | Model-level access control | `SEC-01` |
| record rule | Record-level access restriction | `SEC-01` |
| action | Odoo action mechanism | `UI-01` |
| view | Представление records для UI | `UI-02` |
| onchange | Form-view/client mechanism на pseudo-record | `UI-03` |
| company context / multi-company | Company-scoped runtime/data semantics | `EXT-04` |
| `res.company` | Odoo data model company; не универсальный parent всех records | future business-model owner |

## Языковое правило

При первом содержательном вводе:

> модель (`model`)

Далее используем русский термин, если речь не идёт о точном API/class/key/technical name.