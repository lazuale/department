# Baseline B1 — Odoo 19 learning wiki

> Baseline ID: `B1`  
> Зафиксирован: 2026-08-13  
> Scope: Odoo 19.0 Community self-hosted  
> Evidence policy: official Odoo 19.0 documentation only  
> Status: **frozen foundation**

## Что зафиксировано

Baseline B1 фиксирует не содержание будущих уроков, а **архитектуру самого курса**:

- product/source scope;
- stable lesson-ID system;
- canonical prerequisite DAG;
- canonical owner + aspect-owner model;
- coverage statuses;
- edition-evidence separation;
- `[ODOO][S#]` traceability rule;
- каталог верхнеуровневых блоков;
- первые owner-lessons `ARCH-01`–`ARCH-04`, `ORM-01`–`ORM-02` как стартовую основу.

## Что freeze означает

После B1 запрещено перестраивать фундамент только потому, что новая структура кажется красивее или удобнее.

Архитектурное изменение baseline допускается только при одном из оснований:

1. новое official Odoo 19.0 evidence противоречит текущей модели;
2. обнаружена фактическая внутренняя contradiction между governance/lessons;
3. coverage audit выявил фундаментальный prerequisite или official surface, который невозможно корректно встроить существующим aspect/owner mechanism;
4. пользователь явно меняет product/source scope;
5. Odoo 19.0 documentation materially changes и повторная проверка требует изменения baseline.

## Что не требует изменения baseline

Не являются основанием для unfreeze:

- литературная редактура;
- исправление опечаток;
- добавление новых source IDs без изменения semantics;
- добавление planned lesson по уже предусмотренному stable ID;
- расширение content внутри назначенного canonical/aspect owner;
- добавление business/domain lessons в предусмотренные блоки;
- обновление edition ledger конкретными evidence entries;
- обновление coverage status `planned → in progress → covered`.

## Канонические governance-файлы B1

- `method.md` — метод и scope;
- `course-dag.md` — единственный нормативный prerequisite graph;
- `concept-ownership.md` — canonical/aspect ownership;
- `coverage-map.md` — полнота official documentation coverage;
- `source-policy.md` — evidence/traceability;
- `edition-ledger.md` — edition facts и feature entitlement;
- `glossary.md` — терминологический индекс.

## Текущие зафиксированные owner-lessons

| ID | Owner |
|---|---|
| `ARCH-01` | architectural foundation |
| `ARCH-02` | module system |
| `ARCH-03` | Python addon import chain |
| `ARCH-04` | request / RPC execution boundary |
| `ORM-01` | ORM Core |
| `ORM-02` | per-database model registry / model composition |

Следующий planned owner после freeze — `ORM-03`.

## Контроль изменения baseline

Если baseline всё же меняется, изменение должно одновременно:

1. назвать основание из списка выше;
2. обновить этот файл новым Baseline ID (`B2`, `B3`, ...);
3. обновить `course-dag.md`, `concept-ownership.md`, `coverage-map.md` и README, если они затронуты;
4. пройти отдельный consistency audit до продолжения новых уроков.

Без этого baseline считается неизменным.