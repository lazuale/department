# ARCH-03. Python package и import chain addon

> Lesson ID: `ARCH-03`  
> Версия: Odoo 19.0  
> Проверено: 2026-08-13  
> Prerequisites: `ARCH-02`  
> Canonical owner: Python package / `__init__.py` import chain  
> Aspect owner: —  
> Preview: Python model declarations  
> Отложено: ORM Model semantics; backend model registry; per-database model construction; inheritance; metaclasses; schema synchronization  
> Edition scope: platform/module mechanics  
> Sources: `S1`–`S3`

## Цель

Понять только Python-level часть addon loading и не использовать ORM concepts раньше их owner.

Главное различие:

```text
addon package существует
        ≠
Python file импортирован и declarations executed
```

Per-database model registry и effective model composition **не принадлежат этому уроку**; их owner — `ORM-02` после ORM Core.

---

## 1. Addon shell и Python package

**[ODOO][S1]** Current Server Framework 101 начинает addon с `__init__.py` и `__manifest__.py`.

```text
module/
├── __init__.py
└── __manifest__.py
```

`__init__.py` может оставаться empty до появления Python modules/files.

**[ODOO][S2]** Architecture Overview показывает, что при наличии Python business objects они организуются в package/subpackages с `__init__.py` import instructions.

---

## 2. Manifest и imports — разные механизмы

Из `ARCH-02`:

```text
__manifest__.py
= Odoo metadata / dependencies / declared data

__init__.py
= Python import chain
```

**[ВЫВОД]** Manifest не заменяет Python imports, а Python import не заменяет manifest dependency.

---

## 3. Типичный import chain

Когда появляются Python model files, tutorial использует цепочку вида:

```python
# module/__init__.py
from . import models
```

```python
# module/models/__init__.py
from . import example
```

```text
module/__init__.py
        │
        ▼
models/__init__.py
        │
        ▼
models/example.py
        │
        ▼
Python declarations execute
```

**[ODOO][S2][S3]** `__init__.py` содержит import instructions для Python files/subpackages addon.

---

## 4. `.py` file сам по себе не исполняется только из-за расположения

```text
models/example.py exists
        │
        ▼
import chain reaches file?
     ┌──────┴──────┐
    no             yes
    │               │
not executed     declarations execute
through that     in Python runtime
package chain
```

**[ВЫВОД]** Filesystem organization и executed Python declaration — разные states.

---

## 5. Intra-module import и inter-module dependency

```text
INTRA-MODULE
__init__.py → Python imports

INTER-MODULE
__manifest__.py `depends` → Odoo dependency graph
```

**[ODOO][S3]** Manifest dependencies должны быть loaded before dependent module и используются для module-level dependencies.

**[ВЫВОД]** Нельзя объяснять `depends` обычным Python import и нельзя считать прямой import соседнего addon заменой declared dependency.

---

## 6. Где заканчивается этот урок

После ARCH-03 мы знаем:

```text
addon skeleton
    │
    ├── manifest → Odoo module declaration
    └── __init__ → Python import path
                       │
                       ▼
                Python declarations
```

Но мы **ещё не определили**:

- что такое ORM `Model`;
- что такое recordset;
- что означает model instance;
- как выглядит per-database model mapping;
- как несколько Python classes составляют effective ORM model.

Это сознательно оставлено для `ORM-01` и `ORM-02`.

---

## Что нельзя заключать

- каждый `.py` file addon automatically executes — нет;
- manifest импортирует Python files — нет;
- Python import = Odoo module dependency — нет;
- после ARCH-03 уже понятен backend model registry — нет;
- после ARCH-03 уже понятна inheritance composition ORM models — нет.

## Контрольные вопросы

1. Чем `__manifest__.py` отличается от `__init__.py`?
2. Почему empty `__init__.py` нужен уже в minimal addon shell current tutorial?
3. Что такое intra-module import chain?
4. Почему file presence не равно executed declaration?
5. Чем Python import chain отличается от manifest `depends`?

## Официальные источники

- `S1` — A New Application  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/02_newapp.html
- `S2` — Architecture Overview  
  https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- `S3` — Module Manifests  
  https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html