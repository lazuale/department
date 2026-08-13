# Governance обучающей wiki Odoo 19

Этот каталог содержит правила, которые управляют всей wiki. Они не являются учебными уроками и не должны дублировать содержание owner-уроков.

## Что здесь хранится

- [Метод курса](method.md) — scope, stable IDs, dependency order и правила качества;
- [Политика источников](source-policy.md) — official sources, приоритет и traceability;
- [Владение понятиями](concept-ownership.md) — canonical owner и aspect owners;
- [Coverage map](coverage-map.md) — official documentation surface → lesson owner → status;
- [Реестр редакций](edition-ledger.md) — отдельно platform edition facts и feature/module availability;
- [Глоссарий](glossary.md) — короткий индекс терминов.

## Базовая модель

```text
Source policy
      ↓
Coverage map ─────► не даёт потерять тему
      ↓
Concept ownership ─► не даёт определить её дважды
      ↓
Stable-ID DAG ─────► не даёт использовать concept раньше prerequisite
      ↓
Lesson
      ↓
Edition ledger ────► не даёт угадать Community/Enterprise
```

Governance не должен превращаться во вторую энциклопедию Odoo.