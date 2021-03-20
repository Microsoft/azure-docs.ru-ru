---
title: Свойства метаданных контента
titleSuffix: Azure Cognitive Search
description: Свойства метаданных документов могут предоставлять содержимое для полей в индексе поиска или сведения, уведомляющие о поведении индексирования во время выполнения. В этой статье перечислены свойства метаданных, поддерживаемые Когнитивный поиск Azure.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/22/2021
ms.openlocfilehash: cbb35f596a1d32816d1a73b462bf590d9dde0d52
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101668424"
---
# <a name="content-metadata-properties-used-in-azure-cognitive-search"></a>Свойства метаданных содержимого, используемые в Когнитивный поиск Azure

SharePoint Online и хранилище BLOB-объектов Azure могут содержать различные данные, а многие из этих типов содержимого имеют свойства метаданных, которые могут быть полезны для индексирования. Точно так же, как можно создавать поля поиска для стандартных свойств больших двоичных объектов, таких как **`metadata_storage_name`** , можно создавать поля для свойств метаданных, относящихся к формату документа.

## <a name="supported-document-formats"></a>Поддерживаемые форматы документов

Когнитивный поиск поддерживает индексирование больших двоичных объектов и индексирование документов SharePoint Online для следующих форматов документов:

[!INCLUDE [search-blob-data-sources](../../includes/search-blob-data-sources.md)]

## <a name="properties-by-document-format"></a>Свойства по формату документа

В следующей таблице приводится сводка по обработке для каждого формата документа и описываются свойства метаданных, извлеченные индексатором больших двоичных объектов и индексатором SharePoint Online.

| Формат документа или тип содержимого | Извлеченные метаданные | Сведения об обработке |
| --- | --- | --- |
| HTML (text/html) |`metadata_content_encoding`<br/>`metadata_content_type`<br/>`metadata_language`<br/>`metadata_description`<br/>`metadata_keywords`<br/>`metadata_title` |Удаление разметки HTML и извлечение текста |
| PDF (приложение/PDF) |`metadata_content_type`<br/>`metadata_language`<br/>`metadata_author`<br/>`metadata_title` |Извлечение текста, включая внедренные документы (кроме изображений) |
| DOCX (application/vnd.openxmlformats-officedocument.wordprocessingml.document) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Извлечение текста, включая внедренные документы |
| DOC (application/msword) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Извлечение текста, включая внедренные документы |
| DOCM (Application/vnd.ms-word.docумент. макроенаблед. 12) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Извлечение текста, включая внедренные документы |
| WORD XML (application/vnd. MS-word2006ml) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Удаление разметки XML и извлечение текста |
| WORD 2003 XML (application/vnd. МС-вордмл) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date` |Удаление разметки XML и извлечение текста |
| XLSX (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Извлечение текста, включая внедренные документы |
| XLS (application/vnd.ms-excel) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Извлечение текста, включая внедренные документы |
| XLSM (приложение/vnd. MS-Excel а. sheet. макроенаблед. 12) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Извлечение текста, включая внедренные документы |
| PPTX (application/vnd.openxmlformats-officedocument.presentationml.presentation) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |Извлечение текста, включая внедренные документы |
| PPT (application/vnd.ms-powerpoint) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |Извлечение текста, включая внедренные документы |
| PPTM (приложение/vnd. МС-поверпоинт. Presentation. макроенаблед. 12) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_slide_count`<br/>`metadata_title` |Извлечение текста, включая внедренные документы |
| MSG (application/vnd.ms-outlook) |`metadata_content_type`<br/>`metadata_message_from`<br/>`metadata_message_from_email`<br/>`metadata_message_to`<br/>`metadata_message_to_email`<br/>`metadata_message_cc`<br/>`metadata_message_cc_email`<br/>`metadata_message_bcc`<br/>`metadata_message_bcc_email`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_subject` |Извлеките текст, включая текст, извлеченный из вложений. `metadata_message_to_email``metadata_message_cc_email`и `metadata_message_bcc_email` являются строковыми коллекциями, остальные поля являются строками.|
| ODT (application/vnd. Oasis. OpenDocument. Text) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`metadata_page_count`<br/>`metadata_word_count` |Извлечение текста, включая внедренные документы |
| ODS (application/vnd. Oasis. OpenDocument. электронных таблиц) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified` |Извлечение текста, включая внедренные документы |
| ODP (приложение/vnd. Oasis. OpenDocument. Presentation) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_last_modified`<br/>`title` |Извлечение текста, включая внедренные документы |
| ZIP (application/zip) |`metadata_content_type` |Извлечение текста из всех документов в архиве |
| GZ (приложение/gzip) |`metadata_content_type` |Извлечение текста из всех документов в архиве |
| ЕПУБ (Application/епуб + ZIP) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_creation_date`<br/>`metadata_title`<br/>`metadata_description`<br/>`metadata_language`<br/>`metadata_keywords`<br/>`metadata_identifier`<br/>`metadata_publisher` |Извлечение текста из всех документов в архиве |
| XML (application/xml) |`metadata_content_type`<br/>`metadata_content_encoding`<br/> |Удаление разметки XML и извлечение текста |
| JSON (application/json) |`metadata_content_type`<br/>`metadata_content_encoding` |Извлечение текста<br/>Примечание. Инструкции по извлечению нескольких полей документа из большого двоичного объекта JSON см. в статье [Индексирование BLOB-объектов JSON с помощью индексатора BLOB-объектов службы поиска Azure](search-howto-index-json-blobs.md). |
| EML (message/rfc822) |`metadata_content_type`<br/>`metadata_message_from`<br/>`metadata_message_to`<br/>`metadata_message_cc`<br/>`metadata_creation_date`<br/>`metadata_subject` |Извлечение текста, включая вложения |
| RTF (приложение или RTF) |`metadata_content_type`<br/>`metadata_author`<br/>`metadata_character_count`<br/>`metadata_creation_date`<br/>`metadata_page_count`<br/>`metadata_word_count`<br/> | Извлечение текста|
| Обычный текст (text/plain) |`metadata_content_type`<br/>`metadata_content_encoding`<br/> | Извлечение текста|

## <a name="see-also"></a>См. также раздел

* [Indexers in Azure Cognitive Search](search-indexer-overview.md) (Индексаторы в службе "Когнитивный поиск Azure")
* [Общие сведения о больших двоичных объектах с помощью AI](search-blob-ai-integration.md)
* [Общие сведения об индексировании BLOB-объектов](search-blob-storage-integration.md)
* [Индексирование SharePoint Online](search-howto-index-sharepoint-online.md)