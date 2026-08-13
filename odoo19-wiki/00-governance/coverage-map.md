# Карта покрытия Odoo 19.0

Цель этого файла — защищать курс не от дублей, а от **пропусков**.

Он сопоставляет крупные разделы официальной документации с владельцами уроков и статусом покрытия.

## Допустимые статусы

Используются только пять машинно-стабильных значений:

- `covered` — урок-владелец существует и проверен;
- `in progress` — владелец существует, материал ещё развивается;
- `planned` — владелец назначен, урок ещё не создан;
- `intentionally deferred` — тема сознательно отложена до нужных предпосылок или следующей фазы;
- `out of scope` — тема явно исключена из текущей области курса.

Нельзя создавать произвольные статусы вроде `covered at architecture level`. Частичное покрытие или отдельный аспект описывается в колонке «Владелец / область».

> Карта покрытия не заменяет индекс официальной документации. Перед завершением каждого крупного блока она сверяется с актуальной документацией Odoo 19.0.

## Правила курса / архитектура

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| Server Framework 101 — Architecture Overview | `ARCH-01` | covered |
| Server Framework 101 — A New Application / базовая структура addon | `ARCH-02` | covered |
| Module manifests | `ARCH-02` | covered |
| CLI — `addons_path`, жизненный цикл установки в базе, различие `--load` | архитектурный аспект `ARCH-02` | covered |
| CLI — workers, multiprocessing, подробная работа `--load` | runtime-аспект `RUN-06` | planned |
| Python-пакет / цепочка импортов | `ARCH-03` | covered |
| граница клиент/сервер и RPC | `ARCH-04` | covered |

## Backend / ORM Reference

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| `Model` / `TransientModel` / `AbstractModel` | `ORM-01` | covered |
| записи / `recordset` / singleton | `ORM-01` | covered |
| `Environment` — базовая семантика | `ORM-01` | covered |
| `browse()` / `exists()` / `ensure_one()` / `search()` / основы domain | `ORM-01` | covered |
| `create()` / `read()` / `write()` / `unlink()` | `ORM-01` | covered |
| серверный реестр моделей / композиция итоговой модели конкретной базы | `ORM-02` | covered |
| технические метаданные модели (`_name`, `_description`, `_auto`, `_table`, `_register`, `_log_access`, `_rec_name`, `_order`, иерархические/UI-связанные параметры) | `ORM-03` | covered |
| объявления SQL-схемы: `Constraint`, `Index`, `UniqueIndex` и связанные правила хранения | `ORM-03` | covered |
| атрибуты наследования (`_inherit`, `_inherits`) | `EXT-01` | planned |
| атрибут multi-company (`_check_company_auto`) | `EXT-04` | planned |
| `Field`: общие параметры, несвязные типы, автоматические/зарезервированные поля, правила хранения | `ORM-04` | covered |
| безопасность поля (`groups`) | `SEC-01` | planned |
| `company_dependent` | `EXT-04` | planned |
| настройка индексов/производительности на уровне поля | `RUN-01` | planned |
| `Many2one` / `One2many` / `Many2many` / `Command` | `ORM-05` | planned |
| вычисляемые / связанные / `inverse` / `depends` / Python-ограничения | `ORM-06` | planned |
| транзакция RPC / `commit` / `rollback` | `ORM-07`, предпосылка `ARCH-04` | planned |
| изменения `Environment` в аспектах безопасности и компаний | аспекты `SEC-01`, `EXT-04` | planned |
| кэш / prefetch / `flush` / raw SQL / invalidation | `RUN-01` | planned |
| рекомендации по производительности | `RUN-01` | planned |
| продвинутые особенности реализации | `RUN-01` или отдельные справочные заметки | intentionally deferred |

## Данные модуля / действия / безопасность

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| файлы данных XML / CSV | `DATA-01` | planned |
| внешние ID / XML ID | `DATA-01` | planned |
| `noupdate` / семантика обновления данных | `DATA-01` | planned |
| master data / demo data модуля Odoo | `DATA-01` | planned |
| серверные действия | `UI-01` | planned |
| меню | `UI-01` | planned |
| права доступа / группы / record rules / доступ к полям | `SEC-01` | planned |
| небезопасные публичные методы / RPC-аспект безопасности | `SEC-01`, предпосылка `ARCH-04` | planned |

## Пользовательский интерфейс / frontend

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| записи представлений | `UI-02` | planned |
| архитектура представлений form/list/search и общая семантика | `UI-02` | planned |
| `@api.onchange` / псевдозапись | `UI-03`, предпосылки `ARCH-04`, `UI-02`, `ORM-04` | planned |
| QWeb-отчёты / действия отчётов | `UI-04` | planned |
| общий/frontend QWeb | аспект `RUN-03` | planned |
| HTTP-контроллеры / маршруты | `RUN-02` | planned |
| frontend RPC / ORM services | `RUN-03`, каноническая граница → `ARCH-04` | planned |
| компоненты Owl | `RUN-03` | planned |
| frontend registries | `RUN-03` | planned |
| frontend services / hooks / patching | `RUN-03` | planned |
| assets | `RUN-03` | planned |

## Механизмы расширения

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| наследование моделей / `_inherit` / `_inherits` | `EXT-01` | planned |
| наследование представлений | `EXT-02` | planned |
| mixins | `EXT-03` | planned |
| multi-company: значения по компаниям, согласованность, рекомендации | `EXT-04` | planned |

## Runtime / инженерная часть

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| Coding Guidelines — дисциплина транзакций RPC | `ORM-07`, предпосылка `ARCH-04` | planned |
| общий транзакционный аспект HTTP-запросов/контроллеров | `RUN-02`, только если прямо документирован | planned |
| Performance Reference | `RUN-01` | planned |
| HTTP Reference | `RUN-02` | planned |
| Testing framework | `RUN-04` | planned |
| upgrade scripts / upgrade utils | `RUN-05` | planned |
| CLI / workers / multiprocessing / server-wide runtime | `RUN-06` | planned |
| установка из source/package | `RUN-06` | planned |
| специфическая эксплуатация Odoo Online | отдельный будущий контекст | out of scope |
| специфическая эксплуатация Odoo.sh | отдельный будущий контекст | out of scope |

## Отчёты / другие разделы backend reference

| Раздел документации / понятие | Владелец / область | Статус |
|---|---|---|
| QWeb-отчёты / действия отчётов | `UI-04` | planned |
| developer reference штатных модулей | предметная/бизнес-фаза | intentionally deferred |
| внешний JSON-2 API | интеграционная/runtime-фаза при необходимости | intentionally deferred |
| устаревающие внешние XML-RPC / JSON-RPC API | историческая/версионная заметка при необходимости | intentionally deferred |

## Бизнес-слой

| Раздел | Владелец / область | Статус |
|---|---|---|
| общие/системные модели (`res.partner`, `res.users`, `res.company`, продукты и т. п.) | будущие владельцы `10-business-model/` | planned |
| пользовательская документация предметных приложений | будущие владельцы `11-domain-apps/` | planned |
| сквозные процессы между приложениями | будущие владельцы `12-end-to-end/` | planned |
| исчерпывающий технический список Community addons | требует изменения политики источников, если понадобятся официальные manifest-файлы | intentionally deferred |

## Правило завершения блока

Перед тем как считать крупный блок завершённым:

1. открыть актуальный индекс официальной документации Odoo 19.0;
2. проверить релевантные разделы;
3. каждой релевантной теме назначить владельца/область и один допустимый статус;
4. не оставлять неизвестные пробелы без строки;
5. новых владельцев сначала внести в `concept-ownership.md` и `course-dag.md`;
6. после Baseline B1 структурные изменения проверять по `baseline.md`.