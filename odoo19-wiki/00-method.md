# 00. Метод изучения Odoo 19

## Цель

Изучить **Odoo 19.0 Community** как систему: платформенную архитектуру, модульную модель, ORM, представление данных, безопасность, способы расширения и нативную ERP-логику.

Курс не является инструкцией по настройке конкретной организации. Сначала устанавливается, **как Odoo устроена и почему**, и только затем — как её штатные модели применяются к предметным процессам.

## 1. Источниковая дисциплина

Для фактических утверждений используются только официальные материалы Odoo 19.0:

- Odoo 19 Documentation;
- Developer tutorials;
- Developer reference;
- Administration documentation.

Не используются как основание курса:

- блоги и форумы;
- Stack Overflow;
- статьи интеграторов;
- сторонние модули;
- документация прошлых версий, если утверждение должно описывать Odoo 19.0.

Если официальный материал прямо помечен Odoo как устаревший tutorial, он не используется как основной нормативный источник, если то же понятие описано в актуальном Server Framework 101 или Reference.

## 2. Community и Enterprise не смешиваются

Общая документация Odoo описывает продукт в целом. Само наличие страницы в документации не доказывает доступность функции в Community.

Для спорных приложений и функций используются три статуса:

- **Community подтверждено**;
- **Enterprise подтверждено**;
- **редакция не установлена официальной документацией**.

Если редакция не установлена, курс не делает предположение.

## 3. Разделяем уровни архитектуры

Нельзя смешивать понятия разных уровней.

Примеры:

- `Model`, `TransientModel`, `AbstractModel` — формальные ORM-классы Odoo;
- module/addon — техническая единица расширения платформы;
- App — user-facing module, помеченный как полноценное приложение;
- view/action/menu — механизмы пользовательского слоя;
- master data, transaction, reference data — ERP-классификация, а не типы ORM.

## 4. Три типа утверждений

### [ODOO]

Утверждение непосредственно следует из официальной документации Odoo 19.0.

### [ВЫВОД]

Логическое следствие нескольких документированных фактов, но не буквальная формулировка Odoo.

### [ERP-КЛАССИФИКАЦИЯ]

Общеархитектурный термин для анализа бизнес-смысла, не являющийся внутренним типом Odoo.

Маркировка нужна там, где интерпретацию легко спутать с канонической терминологией продукта.

## 5. Единая последовательность курса

README отражает именно эту последовательность и не имеет отдельной логики курса.

```text
00  Метод и глоссарий

01  Архитектурный фундамент
    process / databases / three tiers / module-app-model-record

02  Модульная система
    addons_path / manifest / dependencies / lifecycle

03  Загрузка модулей
    Python package / __init__ / imports / dependency order /
    per-database construction of available models

04  ORM Core
    Model / Recordset / Environment / browse / search /
    domain / create / read / write / unlink

05  Fields
    basic field semantics / metadata / storage

06  Relations
    Many2one / One2many / Many2many / relational commands

07  Derived behavior
    computed / related / dependencies / inverse /
    onchange / constraints / recomputation

08  Module data
    XML / CSV / external IDs / noupdate / load order

09  Security
    users / groups / ACL / record rules / field access

10  User interface plumbing
    actions / menus / views / domains / context

11  Inheritance and extension
    _inherit / _inherits / view inheritance / mixins /
    cross-module extension

12  Multi-company
    allowed companies / company-bound data /
    company-dependent values / consistency

13  Advanced ORM
    cache / prefetch / performance / transactions /
    flush / raw SQL / invalidation / recomputation consistency

14  HTTP and frontend
    controllers / RPC / web client / Owl / assets

15  Testing, upgrades and deployment

16+ Shared business models, domain applications and end-to-end ERP flows
```

Переход к следующему уровню делается только после того, как предыдущий не требует необъяснённых предпосылок.

## 6. Что значит «полнота урока»

Каждый урок должен **покрывать** следующие вопросы, но не обязан использовать их как буквальные заголовки:

- что это в терминах Odoo;
- зачем механизм существует;
- где он находится в архитектуре;
- с чем связан;
- что пользователь видит;
- какие выводы делать нельзя;
- какая минимальная модель должна остаться в голове;
- какие официальные источники подтверждают материал.

Это правило содержания, а не жёсткий шаблон оформления.

## 7. Один владелец понятия

Каждая фундаментальная тема имеет основной урок-владелец.

Ранний урок может упомянуть будущий механизм только настолько, насколько это необходимо для текущего объяснения, и должен явно отметить, что подробная семантика будет разобрана позже.

Например:

- ORM Core может упомянуть наличие fields, но не разбирать computed fields;
- Environment можно определить до Security, но `sudo()` подробно разбирается только после Security;
- raw SQL не разбирается до cache, transactions и recomputation.

Это предотвращает дублирование и скрытые зависимости.

## 8. Не учим Odoo через меню

Структура интерфейса не является надёжным описанием внутренней архитектуры.

Курс не делает вывод:

```text
есть меню → существует отдельная независимая бизнес-система
```

Вместо этого устанавливается:

```text
какие modules доступны и установлены
        ↓
какие models определены или расширены
        ↓
какие records и configuration data существуют
        ↓
какие actions/views/menus публикуют эту модель
        ↓
что в итоге видит пользователь
```

## 9. Не подгоняем Odoo под заранее выбранный процесс

Сначала устанавливается:

1. что механизм означает в Odoo;
2. как он устроен;
3. с чем связан;
4. какие ограничения заложены в платформе или модели;
5. где проходит граница Community/Enterprise;
6. только затем — где механизм естественно применять.

## 10. Терминология

Основные термины нормализуются в [глоссарии](glossary.md).

При первом вводе используется форма:

> модель (`model`)

Далее в русском тексте используется русский термин, если речь не идёт о конкретном API, имени класса, manifest key или техническом идентификаторе.

## 11. Официальные отправные точки

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- Models and Basic Fields: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- Data Files: https://www.odoo.com/documentation/19.0/developer/reference/backend/data.html
- Security: https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html
- Actions: https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html
- Server Framework 101: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html
