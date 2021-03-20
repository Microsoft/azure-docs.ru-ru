---
title: Формат Markdown — QnA Maker
description: Ниже приведен список форматов Markdown, которые можно использовать в тексте ответа QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 03/19/2020
ms.openlocfilehash: f49e05578f55a38845acbd4010f928fb17c51606
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96352275"
---
# <a name="markdown-format-supported-in-qna-maker-answer-text"></a>Формат Markdown, поддерживаемый в тексте ответа QnA Maker

QnA Maker сохраняет текст ответа как Markdown. Существует множество разновидностей Markdown. Чтобы убедиться, что текст ответа возвращен и отображается правильно, используйте эту ссылку.

Используйте **[CommonMark](https://commonmark.org/help/tutorial/index.html)** для проверки Markdown. Руководство содержит функцию **Попробовать** для быстрой проверки копирования и вставки.

## <a name="when-to-use-rich-text-editing-versus-markdown"></a>Когда следует использовать редактирование RTF-текста и Markdown

Редактирование ответов с помощью [расширенного текста](How-To/edit-knowledge-base.md#add-an-editorial-qna-set) позволяет разработчику использовать панель инструментов форматирования для быстрого выбора и форматирования текста.

Markdown — это лучшее средство, когда необходимо автоматически создавать содержимое для создания баз знаний, импортируемых в рамках конвейера CI/CD или для [пакетного тестирования](./index.yml).

## <a name="supported-markdown-format"></a>Поддерживаемый формат Markdown

Ниже приведен список форматов Markdown, которые можно использовать в тексте ответа QnA Maker.

|Назначение|Формат|Пример разметки|Отрисовка<br>как отображается в чате чата|
|--|--|--|--|
Новая строка из двух предложений.|`\n\n`|`How can I create a bot with \n\n QnA Maker?`|![форматировать новую строку между двумя предложениями](./media/qnamaker-concepts-datasources/format-newline.png)|
|Заголовки от H1 до H6, количество `#` определяет, какой заголовок. 1 `#` — это H1.|`\n# text \n## text \n### text \n####text \n#####text` |`## Creating a bot \n ...text.... \n### Important news\n ...text... \n### Related Information\n ....text...`<br><br>`\n# my h1 \n## my h2\n### my h3 \n#### my h4 \n##### my h5`|![формат с заголовками Markdown](./media/qnamaker-concepts-datasources/format-headers.png)<br>![формат с заголовками Markdown от H1 до H5](./media/qnamaker-concepts-datasources/format-h1-h5.png)|
|Курсив |`*text*`|`How do I create a bot with *QnA Maker*?`|![форматировать курсивом](./media/qnamaker-concepts-datasources/format-italics.png)|
|Strong (полужирный)|`**text**`|`How do I create a bot with **QnA Maker**?`|![формат с усиленной маркировкой для полужирного шрифта](./media/qnamaker-concepts-datasources/format-strong.png)|
|URL-адрес для ссылки|`[text](https://www.my.com)`|`How do I create a bot with [QnA Maker](https://www.qnamaker.ai)?`|![формат URL-адреса (гиперссылка)](./media/qnamaker-concepts-datasources/format-url.png)|
|* URL-адрес для общедоступного образа|`![text](https://www.my.com/image.png)`|`How can I create a bot with ![QnAMaker](https://review.docs.microsoft.com/azure/cognitive-services/qnamaker/media/qnamaker-how-to-key-management/qnamaker-resource-list.png)`|![формат URL-адреса общедоступного образа ](./media/qnamaker-concepts-datasources/format-image-url.png)|
|Зачеркнутый|`~~text~~`|`some ~~questoins~~ questions need to be asked`|![формат для зачеркивания](./media/qnamaker-concepts-datasources/format-strikethrough.png)|
|Полужирный и курсив|`***text**_`|`How can I create a _*_QnA Maker_** bot?`|![формат для полужирного шрифта и курсива](./media/qnamaker-concepts-datasources/format-bold-italics.png)|
|Полужирный URL-адрес для ссылки|`[**text**](https://www.my.com)`|`How do I create a bot with [**QnA Maker**](https://www.qnamaker.ai)?`|![формат для URL-адреса полужирного шрифта](./media/qnamaker-concepts-datasources/format-bold-url.png)|
|URL-адрес курсивов для ссылки|`[*text*](https://www.my.com)`|`How do I create a bot with [*QnA Maker*](https://www.qnamaker.ai)?`|![формат URL-адреса курсивов](./media/qnamaker-concepts-datasources/format-url-italics.png)|
|Escape-символы Markdown|`\*text\*`|`How do I create a bot with \*QnA Maker\*?`|![Формат для символов escape-Markdown.](./media/qnamaker-concepts-datasources/format-escape-markdown-symbols.png)|
|Упорядоченный список|`\n 1. item1 \n 1. item2`|`This is an ordered list: \n 1. List item 1 \n 1. List item 2`<br>В предыдущем примере используется автоматическая нумерация, встроенная в Markdown.<br>`This is an ordered list: \n 1. List item 1 \n 2. List item 2`<br>В предыдущем примере используется явная нумерация.|![формат упорядоченного списка](./media/qnamaker-concepts-datasources/format-ordered-list.png)|
|Неупорядоченный список|`\n * item1 \n * item2`<br>или диспетчер конфигурации служб<br>`\n - item1 \n - item2`|`This is an unordered list: \n * List item 1 \n * List item 2`|![формат неупорядоченного списка](./media/qnamaker-concepts-datasources/format-unordered-list.png)|
|Вложенные списки|`\n * Parent1 \n\t * Child1 \n\t * Child2 \n * Parent2`<br><br>`\n * Parent1 \n\t 1. Child1 \n\t * Child2 \n 1. Parent2`<br><br>Упорядоченные и неупорядоченные списки можно вкладывать вместе. Вкладка, `\t` указывает уровень отступа дочернего элемента.|`This is an unordered list: \n * List item 1 \n\t * Child1 \n\t * Child2 \n * List item 2`<br><br>`This is an ordered nested list: \n 1. Parent1 \n\t 1. Child1 \n\t 1. Child2 \n 1. Parent2`|![формат вложенного неупорядоченного списка](./media/qnamaker-concepts-datasources/format-nested-unordered-list.png)<br>![формат вложенного упорядоченного списка](./media/qnamaker-concepts-datasources/format-nested-ordered-list.png)|

* QnA Maker не обрабатывает образ каким бы то ни было образом. Это роль клиентского приложения для отрисовки образа.

Если требуется добавить содержимое с помощью API обновления или замены базы знаний, а содержимое или файл содержит теги HTML, можно сохранить HTML в файле, убедившись, что открытие и закрытие тегов преобразуются в закодированном формате.

| Сохранить HTML  | Представление в запросе API  | Представление в КБ |
|-----------|---------|-------------------------|
| Да | \&lt; br \& gt; | &lt;br&gt; |
| Да | \&lt; H3 \& gt; заголовок \& lt;/H3 \& gt; | &lt;&gt;/H3 заголовка &lt; H3&gt; |

Кроме того, CR LF (\r\n) преобразуется в значение \n в КИЛОБАЙТах. LF (\n) сохраняется как есть. Если вы хотите экранировать любую escape-последовательность, например \t или \n, можно использовать обратную косую черту, например " \\ \\ r \\ \\ n" и " \\ \\ t".

## <a name="next-steps"></a>Дальнейшие действия

Просмотрите [форматы файлов](reference-tsv-format-batch-testing.md)пакетного тестирования.