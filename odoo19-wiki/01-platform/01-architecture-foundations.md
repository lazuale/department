# Урок 1. Что такое Odoo: архитектурный фундамент

## Цель урока

Понять Odoo до знакомства с предметными приложениями. После этого урока должны быть различимы уровни `database`, `module`, `model`, `record`, `recordset`, `view`, `action` и пользовательское приложение.

---

## 1. Odoo — многоуровневое приложение

**[ODOO]** Odoo использует многоуровневую архитектуру и в официальном Architecture Overview описывается как three-tier application:

- presentation tier — HTML5, JavaScript и CSS;
- logic tier — Python;
- data tier — PostgreSQL.

Минимальная схема:

```text
ПОЛЬЗОВАТЕЛЬ
     │
     ▼
Presentation tier
HTML / JavaScript / CSS
     │
     ▼
Logic tier
Python / Odoo framework
     │
     ▼
Data tier
PostgreSQL
```

**[ВЫВОД]** Экран Odoo нельзя считать самой бизнес-моделью. Пользовательский интерфейс — представление и управление логикой, а сохраняемые данные и бизнес-поведение живут на других уровнях.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

---

## 2. База данных — контекст установленной системы

**[ODOO]** ORM автоматически инстанцирует модели для каждой базы данных. Набор доступных моделей зависит от модулей, установленных в этой базе.

Следовательно, полезная начальная модель:

```text
Odoo installation
      │
      └── Database
             │
             └── installed modules
                    │
                    └── available models
```

**[ВЫВОД]** Нельзя говорить о полном наборе моделей «Odoo вообще» без учёта установленных модулей конкретной базы. Установка модуля может изменить доступный model registry.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 3. Module — техническая единица расширения Odoo

**[ODOO]** Odoo-модуль объявляется Python-пакетом с файлом `__manifest__.py`. Manifest содержит метаданные модуля и, среди прочего, его зависимости и файлы данных.

Упрощённо:

```text
module/
├── __manifest__.py
├── models/
├── views/
├── data/
├── security/
└── ...
```

Manifest может задавать зависимости:

```python
'depends': ['base']
```

**[ODOO]** Зависимые модули должны быть загружены до модуля, который от них зависит.

**[ВЫВОД]** Модуль — не то же самое, что отдельная бизнес-сущность. Один модуль может:

- создать модели;
- расширить модели другого модуля;
- загрузить records;
- добавить views;
- добавить actions;
- определить security;
- вообще существовать преимущественно ради данных.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

---

## 4. Model — основной объект ORM

**[ODOO]** Odoo называет ORM ключевым компонентом платформы. Бизнес-объекты объявляются Python-классами, наследующими модели ORM.

Основные типы:

```text
BaseModel
├── Model
├── TransientModel
└── AbstractModel
```

### Model

**[ODOO]** Основной суперкласс обычных моделей, сохраняемых в базе данных.

### TransientModel

**[ODOO]** Модель для временных данных. Данные хранятся в базе, но периодически автоматически очищаются.

### AbstractModel

**[ODOO]** Абстрактная модель для общего поведения, предназначенного для использования наследующими моделями.

Минимальный пример обычной модели из принципа ORM:

```python
from odoo import models

class TestModel(models.Model):
    _name = 'test.model'
```

**[ВЫВОД]** `master data`, `transaction`, `document`, `reference data` — не альтернативные формальные типы ORM. Это возможная бизнес-классификация конкретных моделей и records.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html
- https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 5. Record и recordset

**[ODOO]** Экземпляр модели в ORM представлен через recordset. Recordset — упорядоченная коллекция records одной модели.

**[ODOO]** Даже одна запись представляется recordset из одного элемента.

Модель в голове:

```text
MODEL
  │
  ├── record 1
  ├── record 2
  └── record 3

recordset = выбранный набор records этой модели
```

Например концептуально:

```text
model: res.partner

recordset:
[partner 7, partner 21, partner 45]
```

**[ВЫВОД]** Нужно различать определение объекта и конкретные данные:

```text
Model
= структура + поведение

Record
= конкретный экземпляр данных

Recordset
= набор records одной model
```

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 6. Установленные модули формируют итоговую модель

**[ODOO]** ORM поддерживает несколько механизмов наследования и расширения моделей. В частности, модель можно расширить in-place через `_inherit` без создания новой `_name`.

Концептуально:

```text
Module A
   │
   └── creates Model X
           ├── field A
           └── method A

Module B
   │
   └── extends Model X
           ├── + field B
           └── + method/behavior B
```

**[ВЫВОД]** Итоговая модель, которой пользуется человек в работающей базе, может быть результатом вклада нескольких установленных модулей.

Это один из важнейших принципов Odoo: расширение существующей модели является штатным механизмом архитектуры, а не исключением.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

---

## 7. Odoo в значительной степени data-driven

**[ODOO]** Официальные developer tutorials характеризуют Odoo как highly data driven. Модули загружают records через data files; такие файлы могут быть XML или CSV и объявляются в manifest.

**[ODOO]** Actions и menus также являются обычными records базы данных, обычно объявляемыми в data files.

Пример логики:

```text
XML / CSV data
      │
      ▼
Database records
      │
      ├── business/configuration data
      ├── security records
      ├── actions
      └── menus
```

**[ВЫВОД]** В Odoo граница между «данными приложения» и «описанием части поведения интерфейса» не совпадает с интуитивным делением на таблицы и программный код. Значительная часть конфигурации самой системы представлена records.

Официальные источники:

- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/04_securityintro.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

---

## 8. Action связывает пользовательское действие с поведением клиента

**[ODOO]** Actions определяют поведение системы в ответ на действия пользователя. Action может храниться в базе или возвращаться как словарь.

**[ODOO]** Window actions используются для открытия представлений модели в web client.

Концептуально:

```text
Menu / button
      │
      ▼
Action
      │
      ▼
Model + requested view modes
      │
      ▼
Web client renders UI
```

**[ВЫВОД]** Наличие отдельного пункта меню не доказывает наличие отдельной независимой предметной модели. Menu может лишь запускать action, работающий с уже существующей моделью.

Официальный источник:
https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html

---

## 9. App, module и model — не синонимы

После предыдущих разделов можно зафиксировать первое важное различие.

```text
APP
= пользовательская функциональная область

MODULE
= техническая единица поставки и расширения

MODEL
= зарегистрированная ORM-модель

RECORD
= конкретные данные модели

RECORDSET
= набор records одной модели

ACTION / VIEW / MENU
= механизмы представления и взаимодействия
```

**[ВЫВОД]** Ошибка «одно приложение = одна модель = одна таблица = один экран» несовместима с модульной архитектурой Odoo.

---

## 10. Минимальная архитектурная модель Odoo после первого урока

```text
                        ODOO
                          │
                          ▼
                       DATABASE
                          │
                          ▼
                  INSTALLED MODULES
                          │
            ┌─────────────┴─────────────┐
            │                           │
            ▼                           ▼
      define / extend              load records
          MODELS                        │
            │                           │
            ▼                           │
       MODEL REGISTRY                   │
            │                           │
            ▼                           │
     RECORDS / RECORDSETS ◄─────────────┘
            │
            ├────────► security/configuration
            │
            └────────► actions / views / menus
                              │
                              ▼
                         WEB CLIENT
                              │
                              ▼
                            USER
```

Это не полная архитектура Odoo. Здесь сознательно пока не разобраны:

- fields и relational fields;
- Environment;
- domains;
- computed fields;
- constraints;
- onchange;
- ACL и record rules;
- views подробно;
- inheritance подробно;
- mixins;
- multi-company;
- frontend framework;
- HTTP/controllers;
- конкретные бизнес-модели.

Они должны изучаться следующими уроками, а не сваливаться в один обзор.

---

## 11. Что после этого урока считать ошибкой

### Ошибка 1

> «Odoo — это набор независимых приложений».

Слишком грубо: приложения работают поверх общей модульной платформы и общей ORM.

### Ошибка 2

> «Экран показывает внутреннюю архитектуру».

Нет. Экран формируется через views/actions и может показывать модель, расширенную несколькими модулями.

### Ошибка 3

> «У каждого бизнес-понятия должна быть отдельная модель».

Из архитектуры Odoo это не следует. Сначала надо проверить существующие модели и механизмы их расширения.

### Ошибка 4

> «Module = Model».

Нет. Модуль может определять и расширять несколько моделей и загружать другие типы records.

### Ошибка 5

> «Record — отдельный Python-объект вне recordset».

В ORM Odoo одиночная запись представляется singleton recordset.

---

## 12. Контрольные вопросы

Перед следующим уроком нужно уметь без интерфейса объяснить:

1. Какие три уровня выделяет официальная архитектура Odoo?
2. Чем module отличается от model?
3. От чего зависит набор зарегистрированных моделей базы?
4. Чем record отличается от model?
5. Что такое recordset?
6. Какие три базовых типа моделей описывает ORM?
7. Почему один модуль может изменить модель другого модуля?
8. Почему menu нельзя считать бизнес-моделью?
9. Почему пользовательское приложение нельзя автоматически приравнять к техническому модулю?
10. Почему нельзя начинать архитектурный анализ Odoo с одного только интерфейса?

---

## Официальные источники урока

1. Architecture Overview  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html

2. Models and Basic Fields  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/03_basicmodel.html

3. ORM API  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html

4. Building a Module  
   https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html

5. Define module data  
   https://www.odoo.com/documentation/19.0/developer/tutorials/define_module_data.html

6. Actions  
   https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html

7. Security — A Brief Introduction  
   https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/04_securityintro.html
