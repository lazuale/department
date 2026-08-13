# Odoo 19 Community: архитектура и идеология

Этот каталог — последовательный учебный курс по **Odoo 19 Community**. Цель — не научиться нажимать кнопки в отдельных приложениях, а понять устройство Odoo настолько глубоко, чтобы видеть архитектуру системы, назначение моделей, границы модулей и нативную логику проектирования.

## Принцип курса

Мы изучаем Odoo **снизу вверх**:

1. платформенная архитектура;
2. модульная система;
3. ORM, модели, records и recordsets;
4. данные, views, actions и menus;
5. security и multi-company;
6. наследование и расширение моделей;
7. общие бизнес-модели;
8. транзакционные модели;
9. предметные приложения;
10. сквозные ERP-процессы;
11. граница штатной конфигурации и разработки;
12. эксплуатационная архитектура.

Мы **не начинаем с Sales, Purchase, Project или Inventory**. Пока не понятна платформа, изучение приложений даёт знание интерфейса, но не архитектуры.

## Источники

Используются только официальные материалы Odoo 19:

- Odoo 19 Documentation;
- Developer tutorials;
- Developer reference;
- Administration documentation.

Если официальная документация описывает функцию, но не позволяет доказать, относится ли она к Community или Enterprise, редакция считается **не установленной**. Наличие страницы в общей документации Odoo не считается само по себе доказательством доступности функции в Community.

## Маркировка утверждений

В курсе используются три уровня:

- **[ODOO]** — утверждение непосредственно следует из официальной документации Odoo 19;
- **[ВЫВОД]** — логическое следствие документированной архитектуры, но не буквальная формулировка Odoo;
- **[ERP-КЛАССИФИКАЦИЯ]** — общеархитектурный термин, полезный для понимания ERP, но не являющийся внутренним типом или термином ORM Odoo.

Это нужно, чтобы не выдавать удобную интерпретацию за внутреннюю архитектуру продукта.

## Карта курса

### 00. Метод изучения

- [Метод и правила курса](00-method.md)

### 01. Платформа

- [Урок 1. Что такое Odoo: архитектурный фундамент](01-platform/01-architecture-foundations.md)
- [Урок 2. Модульная система: addons path, manifest, зависимости и загрузка](01-platform/02-module-system.md)

Следующие уроки добавляются последовательно после проверки предыдущих. Курс намеренно не пытается заранее зафиксировать всю предметную модель Odoo.

## Главный вопрос курса

После изучения системы на любой новый бизнес-вопрос мы должны уметь рассуждать не от интерфейса, а от архитектуры:

```text
Какой бизнес-смысл у объекта?
        ↓
Есть ли штатная модель Odoo?
        ↓
Какой модуль определяет или расширяет её?
        ↓
Какие отношения и ограничения заложены в модели?
        ↓
Какая часть поведения относится к данным,
а какая — к view/action/security/mixin/configuration?
        ↓
Как эта модель участвует в сквозном процессе Odoo?
```

## Официальные отправные точки

- Architecture Overview: https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/01_architecture.html
- Module Manifests: https://www.odoo.com/documentation/19.0/developer/reference/backend/module.html
- Source install: https://www.odoo.com/documentation/19.0/administration/on_premise/source.html
- ORM API: https://www.odoo.com/documentation/19.0/developer/reference/backend/orm.html
- Building a Module: https://www.odoo.com/documentation/19.0/developer/tutorials/backend.html
- Actions: https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html
- Security: https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html
