---
title: Источники данных и типы содержимого — QnA Maker
description: Узнайте, как импортировать пары вопросов и ответов из источников данных и поддерживаемых типов содержимого, включая многие стандартные структурированные документы, такие как PDF, DOCX и TXT-QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 10/13/2020
ms.openlocfilehash: 0d4d32aba34a97c6a060c999694f66d79933d011
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99556040"
---
# <a name="importing-from-data-sources"></a>Импорт из источников данных

База знаний состоит из пар вопросов и ответов, предоставленных общедоступными URL-адресами и файлами.

## <a name="data-source-locations"></a>Расположение источника данных

Содержимое помещается в базу знаний из источника данных. Расположения источников данных — это **общедоступные URL-адреса или файлы**, которые не нуждаются в проверке подлинности.

Исключением являются [файлы SharePoint](../how-to/add-sharepoint-datasources.md), защищенные с помощью проверки подлинности. Ресурсы SharePoint должны быть файлами, а не веб-страницами. Если URL-адрес заканчивается веб-расширением, например .ASPX, импорт в QnA Maker из SharePoint не будет выполнен.

## <a name="chit-chat-content"></a>ХИ2ТЕСТ — содержимое чата

Набор содержимого ХИ2ТЕСТ-Chat предлагается как полный источник данных содержимого на нескольких языках и в стилях общения. Это может стать основой личности бота и сэкономить время и затраты на ее написание с нуля. Узнайте, [как добавить содержимое ХИ2ТЕСТ-Chat](../how-to/chit-chat-knowledge-base.md) в базу знаний.

## <a name="structured-data-format-through-import"></a>Загрузка данных в структурированном формате через импорт

При импорте базы знаний содержимое существующей базы знаний полностью заменяется. Для импорта требуется структурированный `.tsv` файл, который содержит вопросы и ответы. Эти сведения помогают QnA Maker сгруппировать пары "вопрос — ответ" и сопоставить их с конкретным источником данных.

| Вопрос  | Ответ  | Источник| Метаданные (1 ключ: 1 значение) |
|-----------|---------|----|---------------------|
| Question1 | Answer1 | Url1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| Question2 | Answer2 | Редактирование|    `Key:Value`       |

## <a name="structured-multi-turn-format-through-import"></a>Структурированное многострочное форматирование с помощью импорта

Можно создавать множественные беседы в `.tsv` формате файла. Этот формат позволяет создавать множественные беседы путем анализа предыдущих журналов чатов (с другими процессами, не использующими QnA Maker), а затем создавать `.tsv` файл с помощью автоматизации. Импортируйте файл, чтобы заменить существующую базу знаний.

> [!div class="mx-imgBorder"]
> ![Концептуальная модель с тремя уровнями многофакторного вопроса](../media/qnamaker-concepts-knowledgebase/nested-multi-turn.png)

Столбец для множественной переворачивания `.tsv` , предназначенный для множественного включения, — это **запросы**. Пример `.tsv` , показанный в Excel, показывает сведения, которые необходимо включить, чтобы определить множественные дочерние элементы:

```JSON
[
    {"displayOrder":0,"qnaId":2,"displayText":"Level 2 Question A"},
    {"displayOrder":0,"qnaId":3,"displayText":"Level 2 - Question B"}
]
```

**DISPLAYORDER** является числовым, а **DisplayText** — текстом, который не должен включать Markdown.

> [!div class="mx-imgBorder"]
> ![Пример многофакторного вопроса, как показано в Excel](../media/qnamaker-concepts-knowledgebase/multi-turn-tsv-columns-excel-example.png)

## <a name="export-as-example"></a>Экспортировать как пример

Если вы не знаете, как представлять пару QnA в `.tsv` файле:
* Используйте этот [загружаемый пример из GitHub](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Structured-multi-turn-format.xlsx?raw=true)
* Или создайте пару на портале QnA Maker, сохраните, а затем экспортируйте базу знаний, чтобы получить пример представления пары.

## <a name="content-types-of-documents-you-can-add-to-a-knowledge-base"></a>Типы содержимого документов, которые можно добавить в базу знаний
Типы содержимого включают множество стандартных структурированных документов, таких как PDF, DOC и TXT.

### <a name="file-and-url-data-types"></a>Типы данных File и URL

В приведенной ниже таблице представлены типы содержимого и форматы файлов, поддерживаемые QnA Maker.

|Тип источника|Тип содержимого| Примеры|
|--|--|--|
|URL-адрес|Часто задаваемые вопросы<br> (неструктурированные, с разделами или темами на домашней странице)<br>Страницы поддержки <br> (одностраничные руководства, устранение неполадок и т. д.)|[Простая страница вопросов и ответов](../troubleshooting.md) <br>[Часто задаваемые вопросы о со ссылками](https://www.microsoft.com/en-us/software-download/faq)<br> [Вопросы и ответы с домашней страницей разделов](https://www.microsoft.com/Licensing/servicecenter/Help/Faq.aspx)<br>[Статья технической поддержки](./best-practices.md)|
|PDF/DOC|Часто задаваемые вопросы и ответы<br> Руководство по продукту<br> Брошюры<br> Бумажные документы<br> Политика в отношении флаеров<br> Руководство по поддержке<br> Файл структурированных вопросов и ответов (QnA)<br> и т. д.|**Без множественной переворачивания**<br>[Структурированные QnA.docx](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/structured.docx),<br> [Пример руководства по продукту.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/product-manual.pdf)<br> [Пример semi-structured.docx](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/semi-structured.docx),<br> [Пример белого paper.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/white-paper.pdf),<br><br>**Множественная перевращение**:<br>[Surface Pro (DOCX)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx)<br>[Преимущества Contoso (DOCX)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.docx)<br>[Преимущества Contoso (PDF)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.pdf)|
|* Excel|Файл структурированных вопросов и ответов (QnA)<br> (включая поддержку RTF, HTML)|**Без множественного включения**:<br>[Пример часто задаваемых вопросов QnA.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/QnA%20Maker%20Sample%20FAQ.xlsx)<br><br>**Множественная перевращение**:<br>[Структурированный простой FAQ.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Structured-multi-turn-format.xlsx)<br>[FAQ.xlsSurface ноутбука ](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-Surface-Pro.xlsx)|
|* TXT/TSV|Файл структурированных вопросов и ответов (QnA)|[Пример беседы.tsv](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Scenario_Responses_Friendly.tsv)|

Если для источника данных требуется проверка подлинности, рассмотрите следующие методы получения содержимого в QnA Maker:

* Загрузите файл вручную и импортируйте его в QnA Maker
* Добавить файл из расположения, прошедшего проверку подлинности [SharePoint](../how-to/add-sharepoint-datasources.md)

### <a name="url-content"></a>Содержимое URL-адреса

С помощью **URL-адреса** в QnA Maker можно импортировать два типа документов:

* URL-адреса вопросов и ответов
* URL-адреса для поддержки

Каждый тип указывает ожидаемый формат.

### <a name="file-based-content"></a>Содержимое на основе файлов

Вы можете добавить файлы в базу знаний из общедоступного источника или локальной файловой системы на [портале QnA Maker](https://www.qnamaker.ai).

### <a name="content-format-guidelines"></a>Рекомендации по формату содержимого

Узнайте больше о [рекомендациях по формату](../reference-document-format-guidelines.md) для различных файлов.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [изменить QnA](../how-to/edit-knowledge-base.md).
