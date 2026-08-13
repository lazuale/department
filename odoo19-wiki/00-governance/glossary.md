# Глоссарий

Глоссарий — **индекс**, а не второй учебник. Полное определение всегда находится в owner-уроке из [concept-ownership.md](concept-ownership.md).

| Термин | Коротко | Owner |
|---|---|---|
| Odoo deployment / runtime | Запущенная серверная среда Odoo; не приравнивается автоматически к одному OS/Python process | Architecture foundation |
| database / база данных | Отдельная PostgreSQL database, обслуживаемая Odoo | Architecture foundation |
| module / addon / модуль | Техническая единица расширения Odoo | Module system |
| App / приложение | User-facing module, помеченный как application | Module system |
| `addons_path` | Каталоги поиска addons | Module system |
| manifest | `__manifest__.py`, декларация metadata/dependencies/data module | Module system |
| model / модель | ORM-модель Odoo | ORM Core |
| record / запись | Конкретная запись модели; представляется singleton recordset | ORM Core |
| recordset / набор записей | Коллекция records одной model; основной рабочий объект ORM | ORM Core |
| Environment / окружение | Runtime context ORM operation | ORM Core |
| field / поле | ORM field; не UI widget | Fields |
| relational field | `Many2one`, `One2many`, `Many2many` | Relations |
| domain | Декларативное условие ORM search/filtering | ORM Core; relation traversal — Relations |
| backend model registry | Backend mapping/context доступных моделей конкретной database; не frontend registry | Module loading |
| transaction | Framework-managed database transaction context | Transactions |
| Odoo module master data | Официальный термин Odoo для data, устанавливаемых с module и нужных для его работы; включает technical data вроде views/actions | Module data |
| ERP master data | Наша будущая бизнес-классификация; **не то же самое**, что Odoo module master data | Business model |
| external ID / XML ID | Устойчивый identifier module data record | Module data |
| ACL / access right | Model-level access control | Security |
| record rule | Record-level access restriction | Security |
| action | Odoo action mechanism | Actions and menus |
| view | Представление records для user interface | Views |
| onchange | Form-view/client mechanism на pseudo-record | Onchange |
| company / multi-company | Организационная и company-context semantics | Multi-company |

## Языковое правило

При первом содержательном вводе:

> модель (`model`)

Далее используем русский термин, если речь не идёт о точном API/class/key/technical name.