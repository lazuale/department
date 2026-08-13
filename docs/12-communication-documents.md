# Коммуникации, sharing и вложения в Odoo 19 Community

[Главная](../README.md) · [00 Возможности](00-odoo19-community.md) · [09 Project и безопасность](09-project-productivity-security.md) · [10 API](10-external-integrations.md) · [11 Properties](11-relational-properties.md) · **12 Коммуникации и вложения**

---

Этот документ фиксирует штатные механизмы вокруг Project/Chatter, которые могут упростить реальные рабочие сценарии, но не должны превращаться в отдельный параллельный workflow.

## 1. Chatter — журнал коммуникации вокруг записи

Официальная документация Odoo 19 подтверждает для Chatter:

- сообщения и email;
- internal notes;
- followers;
- `@mentions`;
- Activities;
- attachments;
- поиск по сообщениям/заметкам внутри chatter.

Источник: [Chatter](https://www.odoo.com/documentation/19.0/applications/productivity/discuss/chatter.html).

### Методическое решение

Chatter хранит:

- контекст;
- переписку;
- основание изменения;
- комментарии участников;
- вложения.

Не использовать Chatter вместо:

- Stage;
- State;
- Deadline;
- Assignee;
- Activity;
- структурированного Property.

## 2. Followers — подписка, не ответственность

Follower получает коммуникацию по записи в зависимости от notification settings и subtypes.

Follower **не является владельцем результата**.

Поэтому:

```text
Assignee = отвечает за Task
Follower = должен видеть/получать коммуникацию
```

Не назначать несколько Assignees только для того, чтобы люди получали обновления.

## 3. Task можно шарить отдельно

Public Community source содержит:

```text
task.share.wizard
```

который наследует `portal.share`.

При отправке share-приглашения для `project.task` выбранные partners также подписываются на Task как followers.

Источники:

- [`project_task_share_wizard.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/wizard/project_task_share_wizard.py);
- [`portal_share.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/wizard/portal_share.py).

### Методический вывод

Если внешнему участнику нужно дать доступ к **конкретной Task**, не обязательно создавать для него внутреннего пользователя Odoo.

Но внешний sharing включается только при реальном сценарии и после проверки portal permissions.

## 4. Project sharing и Task sharing — разные масштабы

### Project collaboration

Для внешних collaborators Project поддерживает режимы:

- Read;
- Edit with limited access;
- Edit.

### Task sharing

Отдельный share wizard позволяет работать с конкретной Task.

### Правило

Выбирать минимальную достаточную границу доступа:

```text
нужен весь рабочий Project → Project sharing
нужна одна конкретная Task → Task sharing
нужна только переписка → email/Chatter без portal edit
```

Не давать человеку доступ ко всему Project только ради одного результата.

## 5. Task Stages могут быть общими для нескольких Projects

Public `project.task.type` содержит `project_ids` и прямо объясняет, что один Stage можно использовать в нескольких Projects с одинаковым workflow для consolidated information.

Источник: [`project_task_type.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task_type.py).

### Методический вывод

Если позже появятся несколько однотипных Projects, не обязательно создавать независимые копии одинаковых Task Stages.

Но это не причина заранее дробить один операционный контур на множество Projects.

## 6. Personal Stages существуют отдельно от общих Task Stages

Odoo имеет user-specific personal stages. Public source явно запрещает связывать personal stage с Project, потому что он виден только соответствующему пользователю.

To-Do documentation также показывает `+ Personal Stage`.

Источник: [To-do](https://www.odoo.com/documentation/19.0/applications/productivity/to_do.html), [`project_task_type.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task_type.py).

### Методическое решение

Personal Stages можно использовать как личный способ организации работы.

Они **не должны** становиться источником общего управленческого статуса.

Руководитель опирается на общие Task Stages/State/Deadline, а не на персональную доску пользователя.

## 7. To-Do технически можно расшарить

Официальная документация Odoo 19 уточняет:

> добавление Assignees к To-Do делится этой To-Do с выбранными пользователями.

То есть To-Do не абсолютно private.

### Методическое решение

Несмотря на технический sharing, To-Do остаётся **личным/малой совместной буферной сущностью**, а не вторым departmental backlog.

Если результат должен контролироваться отделом:

```text
To-Do → Convert to Task
```

## 8. Email Template на Task Stage

Public `project.task.type` содержит:

```text
mail_template_id = Email Template
```

Если он задан, Odoo автоматически отправляет email customer при переходе Task в этот Stage.

Источник: [`project_task_type.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task_type.py).

### Где полезно

Для устойчивого внешнего сервисного процесса, например:

```text
Stage = Принято
→ автоматически отправить подтверждение
```

или:

```text
Stage = Готово к получению
→ отправить уведомление
```

### Где не нужно

Во внутренней операционной очереди не назначать email template на каждый Stage только потому, что функция существует.

Иначе каждое перетаскивание Kanban превратится в внешний side effect.

## 9. Customer Rating на Stage значительно мощнее простой оценки

Public Task Stage содержит:

- rating email template;
- отправку rating при входе в Stage;
- periodic rating;
- периодичность daily/weekly/twice a month/monthly/quarterly/yearly;
- `Automatic Kanban Status`.

При включённой auto validation:

```text
good feedback
→ Approved

neutral / bad feedback
→ Changes Requested
```

Источник: [`project_task_type.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task_type.py).

### Методическое решение

Это полезно только для customer-facing процесса, где внешняя оценка действительно означает принятие/возврат результата.

**Не использовать customer rating как рейтинг сотрудников.**

Для внутренней работы отдела не включать automatic status changes от rating без отдельного обоснования.

## 10. Scheduled email/message подтверждён в стандартном Mail composer

Public `mail.compose.message` view содержит:

- поле `scheduled_date`;
- кнопку `Schedule`;
- действие `action_schedule_message`.

Источник: [`mail_compose_message_views.xml`](https://github.com/odoo/odoo/blob/19.0/addons/mail/wizard/mail_compose_message_views.xml).

### Где полезно

Например, если сообщение точно должно уйти в определённое время независимо от ручного follow-up.

### Где не использовать

Scheduled message не заменяет Activity.

Различие:

```text
Activity
= человеку нужно проверить ситуацию и решить, что делать

Scheduled message
= заранее известно, что именно и когда нужно отправить
```

Если ответ внешней стороны может изменить действие, лучше Activity, а не заранее запланированная серия писем.

## 11. Mail Templates и Canned Responses — разные вещи

### Mail Template

Структурированный повторяемый email, может использовать dynamic placeholders и применяться программно/из workflow.

### Canned Response

Короткая заготовка текста для человека в Discuss/Chatter/Live Chat.

### Правило выбора

```text
человек выбирает стандартную фразу → Canned Response
структурированное письмо по шаблону → Mail Template
автоматическое письмо при Stage → Stage Email Template
```

Не создавать Automation Rule только для вставки повторяемого текста.

## 12. Attachments — штатная часть Chatter и records

Официальная документация подтверждает:

- upload через paperclip;
- drag & drop;
- Files section в Chatter;
- download пользователями, имеющими доступ к thread.

Источник: [Chatter — Attach files](https://www.odoo.com/documentation/19.0/applications/productivity/discuss/chatter.html).

### Методическое решение

Файлы-основания Task прикладывать к самой записи/переписке, если они нужны для выполнения и истории.

Не заводить отдельный Excel-реестр только для пути к тому же файлу.

## 13. Community содержит Attachment Indexation

Публичный LGPL-модуль:

```text
attachment_indexation
```

Описание:

```text
Attachments List and Document Indexation
```

Public source индексирует текст из:

- DOCX;
- PPTX;
- XLSX;
- OpenDocument;
- PDF.

Для PDF требуется Python-библиотека `pdfminer.six`.

Источники:

- [`attachment_indexation/__manifest__.py`](https://github.com/odoo/odoo/blob/19.0/addons/attachment_indexation/__manifest__.py);
- [`attachment_indexation/models/ir_attachment.py`](https://github.com/odoo/odoo/blob/19.0/addons/attachment_indexation/models/ir_attachment.py).

## 14. Но Attachment Indexation не равен Documents

Base `ir.attachment` действительно хранит:

```text
index_content
```

Однако стандартный Attachment Search View подтверждённо ищет прежде всего по:

- attachment name;
- date;
- creator;
- type;
- metadata/grouping.

Поле `Indexed Content` в стандартной Attachment form доступно как техническое поле developer mode.

Источники:

- [`ir_attachment.py`](https://github.com/odoo/odoo/blob/19.0/odoo/addons/base/models/ir_attachment.py);
- [`ir_attachment_views.xml`](https://github.com/odoo/odoo/blob/19.0/odoo/addons/base/views/ir_attachment_views.xml).

### Поэтому нельзя заявлять

> Community имеет полноценный пользовательский full-text document search как Documents app.

Корректный вывод:

> Community имеет техническое индексирование содержимого ряда attachment formats, но удобство и полноту пользовательского поиска по этому индексу нужно отдельно проверять; это не замена Documents.

## 15. История Description Task

Public `project.task` наследует:

```text
html.field.history.mixin
```

и указывает `description` как versioned field.

Источник: [`project_task.py`](https://github.com/odoo/odoo/blob/19.0/addons/project/models/project_task.py).

### Что подтверждено

Описание Task участвует в штатном механизме history/versioning HTML field.

### Что пока не фиксируем как рабочее правило

До проверки конкретного UI не считаем этот механизм полноценной пользовательской системой версий документа.

Chatter всё равно остаётся местом для оснований значимых изменений, если их должен понимать другой человек.

## 16. Коммуникационные side effects должны быть минимальны

Перед автоматическим email/rating/webhook по Stage проверить:

1. Stage действительно означает однозначное событие?
2. Получатель всегда одинаков по смыслу?
3. Ошибочный drag-and-drop не отправит ложное сообщение?
4. Повторный вход в Stage не создаст нежелательную коммуникацию?
5. Есть ли тестовая база?

Если нет — оставить действие ручным или Activity-driven.

## 17. Рекомендуемый порядок для внешней коммуникации

```text
ручное сообщение в Chatter
→ Canned Response
→ Mail Template
→ Scheduled message
→ Stage Email Template
→ Automation Rule
→ webhook/API
```

Двигаться вправо только когда правило стало однозначным и устойчивым.

## 18. Для внутреннего отдела baseline остаётся простым

Базово достаточно:

- Chatter;
- Followers;
- Attachments;
- Activities;
- при необходимости Canned Responses.

Не включать автоматически:

- customer rating;
- automatic state validation from rating;
- Stage email templates;
- scheduled outbound messages;
- portal Task sharing.

Они используются только в конкретном внешнем сценарии.

---

[← 11 — Relational Properties](11-relational-properties.md) · [Главная](../README.md)
