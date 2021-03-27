---
title: Метод перевода документа для получения формата документа
titleSuffix: Azure Cognitive Services
description: Метод Get Document Formats возвращает список поддерживаемых форматов документов.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/25/2021
ms.author: v-jansk
ms.openlocfilehash: 3cd74f70416d2049a0fb969371507c292dd47fdf
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105613159"
---
# <a name="document-translation-get-document-formats"></a>Преобразование документов: получение форматов документов

Метод Get Document Formats возвращает список форматов документов, поддерживаемых службой преобразования документов. Список содержит общее расширение файла и тип содержимого при использовании API передачи.

## <a name="request-url"></a>URL-адрес запроса

Отправьте запрос `GET` на следующий адрес.
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/documents/formats
```

Узнайте, как найти собственное [доменное имя](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Для всех запросов API к службе перевода документов требуется Пользовательская конечная точка домена**.
> * Вы не можете использовать конечную точку, найденную в портал Azure _ключах ресурсов и конечной_ точке глобального преобразователя, — `api.cognitive.microsofttranslator.com` для выполнения HTTP-запросов к преобразованию документов.

## <a name="request-headers"></a>Заголовки запросов

Заголовки запроса.

|Заголовки|Описание|
|-----|-----|
|Ocp-Apim-Subscription-Key|Обязательный заголовок запроса|

## <a name="response-status-codes"></a>Коды состояния ответа

Ниже приведены возможные коды состояния HTTP, которые возвращает запрос.

|Код состояния|Описание|
|-----|-----|
|200|Все в порядке. Возвращает список поддерживаемых форматов файлов документов.|
|500|Внутренняя ошибка сервера.|
|Другие коды состояния|<ul><li>Слишком много запросов</li><li>Сервер временно недоступен</li></ul>|

## <a name="file-format-response"></a>Ответ формата файла

### <a name="successful-fileformatlistresult-response"></a>Успешный ответ Филеформатлистресулт

При успешном отклике возвращается следующая информация.

|Имя|Тип|Описание|
|--- |--- |--- |
|value|FileFormat []|FileFormat [] содержит приведенные ниже сведения.|
|значение. формат|string[]|Поддерживаемые типы содержимого для этого формата.|
|value. fileExtensions|string[]|Поддерживаемое расширение файла для этого формата.|
|значения. contentType|string[]|Имя формата.|
|value. Versions|String[]|Поддерживаемая версия.|

### <a name="error-response"></a>Сообщение об ошибке

|Имя|Тип|Описание|
|--- |--- |--- |
 |code|строка|Перечисления, содержащие коды ошибок высокого уровня. Возможные значения:<ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>рекуестратетухигх</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>Не авторизовано</li></ul>|
|message|строка|Возвращает сообщение об ошибке высокого уровня.|
|innerError|InnerErrorV2|Новый формат внутренней ошибки, который соответствует рекомендациям Cognitive Services API. Он содержит обязательные свойства ErrorCode, Message и необязательные свойства Target, Details (пара значений ключа), внутреннюю ошибку (может быть вложенной).|
|Иннереррор. Code|строка|Возвращает строку ошибки кода.|
|Иннереррор. Message|строка|Возвращает сообщение об ошибке высокого уровня.|

## <a name="examples"></a>Примеры

### <a name="example-successful-response"></a>Пример успешного ответа
Ниже приведен пример успешного ответа.

Код состояния: 200.

```JSON
{
  "value": [
    {
      "format": "PlainText",
      "fileExtensions": [
        ".txt"
      ],
      "contentTypes": [
        "text/plain"
      ],
      "versions": []
    },
    {
      "format": "PortableDocumentFormat",
      "fileExtensions": [
        ".pdf"
      ],
      "contentTypes": [
        "application/pdf"
      ],
      "versions": []
    },
    {
      "format": "OpenXmlPresentation",
      "fileExtensions": [
        ".pptx"
      ],
      "contentTypes": [
        "application/vnd.openxmlformats-officedocument.presentationml.presentation"
      ],
      "versions": []
    },
    {
      "format": "OpenXmlSpreadsheet",
      "fileExtensions": [
        ".xlsx"
      ],
      "contentTypes": [
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
      ],
      "versions": []
    },
    {
      "format": "OutlookMailMessage",
      "fileExtensions": [
        ".msg"
      ],
      "contentTypes": [
        "application/vnd.ms-outlook"
      ],
      "versions": []
    },
    {
      "format": "HtmlFile",
      "fileExtensions": [
        ".html"
      ],
      "contentTypes": [
        "text/html"
      ],
      "versions": []
    },
    {
      "format": "OpenXmlWord",
      "fileExtensions": [
        ".docx"
      ],
      "contentTypes": [
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
      ],
      "versions": []
    }
  ]
}
```

### <a name="example-error-response"></a>Пример ответа на ошибку

Ниже приведен пример ответа на ошибку. Схема для других кодов ошибок одинакова.

Код состояния: 500

```JSON
{
  "error": {
    "code": "InternalServerError",
    "message": "Internal Server Error",
    "innerError": {
      "code": "InternalServerError",
      "message": "Unexpected internal server error has occurred"
    }
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше об использовании перевода документов и клиентской библиотеки, следуйте нашим инструкциям в этом кратком руководстве.

> [!div class="nextstepaction"]
> [Приступая к преобразованию документов](../get-started-with-document-translation.md)