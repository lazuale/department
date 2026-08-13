# Governance обучающей wiki Odoo 19

Этот каталог содержит правила, которые управляют всей wiki. Они не являются учебными уроками и не должны дублировать содержание owner-уроков.

## Что здесь хранится

- [Метод курса](method.md) — scope, stable IDs и правила качества;
- [Канонический prerequisite DAG](course-dag.md) — единственный нормативный graph зависимостей lessons;
- [Политика источников](source-policy.md) — official sources, приоритет и строгая `[ODOO][S#]` traceability;
- [Владение понятиями](concept-ownership.md) — canonical owner и aspect owners;
- [Coverage map](coverage-map.md) — official documentation surface → owner/scope → status;
- [Реестр редакций](edition-ledger.md) — отдельно platform edition facts и feature/module availability;
- [Глоссарий](glossary.md) — короткий индекс терминов;
- [Baseline B1](baseline.md) — замороженный фундамент и правила его изменения.

## Базовая модель

```text
Source policy
      ↓
Coverage map ───────► не даёт потерять тему
      ↓
Concept ownership ──► не даёт определить concept дважды
      ↓
Course DAG ─────────► не даёт использовать concept раньше prerequisite
      ↓
Lesson
      ↓
Edition ledger ─────► не даёт угадать Community/Enterprise
```

## Baseline status

```text
Baseline: B1
Date:     2026-08-13
Status:   frozen foundation
```

После B1 baseline не перестраивается из-за stylistic preference или появления очередного lesson. Основания для structural change перечислены только в `baseline.md`.

Governance не должен превращаться во вторую энциклопедию Odoo.