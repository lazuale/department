# Канонический prerequisite DAG

Этот файл — **единственный нормативный источник зависимостей между уроками**.

README, оглавления и текстовые схемы курса могут визуализировать порядок, но не создают собственные prerequisites.

`GOV` — root pseudo-node: governance действует до начала учебных уроков.

## Правила

- prerequisite означает, что concept из указанного lesson должен быть понят до текущего lesson;
- перечисляются минимальные прямые prerequisites, а не все транзитивные ancestors;
- новый lesson ID сначала добавляется сюда и в `concept-ownership.md`, затем создаётся content file;
- изменение уже опубликованной зависимости после freeze требует основания из `baseline.md`.

## Architecture / ORM

| Lesson ID | Прямые prerequisites | Назначение |
|---|---|---|
| `ARCH-01` | `GOV` | architectural foundation |
| `ARCH-02` | `ARCH-01` | module system |
| `ARCH-03` | `ARCH-02` | Python package / import chain |
| `ARCH-04` | `ARCH-01` | request / RPC execution boundary |
| `ORM-01` | `ARCH-03` | ORM Core |
| `ORM-02` | `ORM-01` | per-database model registry / composition |
| `ORM-03` | `ORM-02` | model metadata / SQL storage / schema declarations |
| `ORM-04` | `ORM-03` | fields |
| `ORM-05` | `ORM-04` | relations |
| `ORM-06` | `ORM-05` | computed / related / inverse / Python constraints |
| `ORM-07` | `ORM-01`, `ARCH-04` | transaction discipline |

## Data / Security / UI

| Lesson ID | Прямые prerequisites | Назначение |
|---|---|---|
| `DATA-01` | `ARCH-02`, `ORM-01` | module data / external IDs / noupdate |
| `SEC-01` | `ORM-01`, `ARCH-04` | users/groups/access rights/record rules/field access |
| `UI-01` | `DATA-01`, `ORM-01` | actions and menus |
| `UI-02` | `UI-01`, `ORM-04` | views |
| `UI-03` | `UI-02`, `ARCH-04`, `ORM-04` | onchange / pseudo-record |
| `UI-04` | `UI-01`, `UI-02` | QWeb reports / report actions |

## Extension

| Lesson ID | Прямые prerequisites | Назначение |
|---|---|---|
| `EXT-01` | `ORM-02`, `ARCH-02` | model inheritance |
| `EXT-02` | `UI-02` | view inheritance |
| `EXT-03` | `EXT-01` | mixins |
| `EXT-04` | `SEC-01`, `ORM-04` | multi-company / company context |

## Runtime / engineering

| Lesson ID | Прямые prerequisites | Назначение |
|---|---|---|
| `RUN-01` | `ORM-07`, `ORM-06` | advanced ORM / cache / prefetch / raw SQL / invalidation |
| `RUN-02` | `ARCH-04` | HTTP / controllers / deeper RPC |
| `RUN-03` | `ARCH-04`, `UI-02` | web client / Owl / frontend registries / assets / frontend QWeb |
| `RUN-04` | `ORM-07`, `ARCH-02` | testing |
| `RUN-05` | `ARCH-02`, `DATA-01`, `ORM-03` | upgrades / migrations |
| `RUN-06` | `ARCH-01`, `ARCH-02` | deployment / workers / `--load` runtime mechanics / operations |

## Текущий базовый граф

```text
GOV
 │
 ▼
ARCH-01
 ├──────────────► ARCH-04
 │                  │
 ▼                  ├──────────────► SEC-01
ARCH-02             └──────────────► ORM-07
 │
 ▼
ARCH-03
 │
 ▼
ORM-01
 │
 ▼
ORM-02
 │
 ▼
ORM-03
 │
 ▼
ORM-04
 │
 ▼
ORM-05
 │
 ▼
ORM-06
```

Ветви UI, extension и runtime используют таблицы выше; ASCII-схема не заменяет их.

## Проверка перед новым уроком

Перед созданием lesson:

1. его stable ID существует здесь;
2. каждый prerequisite уже существует или имеет согласованный planned owner;
3. нет cycle;
4. concept ownership соответствует prerequisite direction;
5. README не содержит альтернативного graph.