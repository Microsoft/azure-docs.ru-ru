---
title: Метод форматирования словаря для перевода документов
titleSuffix: Azure Cognitive Services
description: Метод Get глоссария formats возвращает список поддерживаемых форматов глоссария.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/25/2021
ms.author: v-jansk
ms.openlocfilehash: fa4f89fb312e76d72447552b156d35bf3d0404bc
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105613140"
---
# <a name="document-translation-get-glossary-formats"></a>Преобразование документов: получение форматов глоссария

Метод "получить форматы глоссария" возвращает список поддерживаемых форматов глоссария, поддерживаемых службой перевода документов. Список содержит общее используемое расширение файла.

## <a name="request-url"></a>URL-адрес запроса

Отправьте запрос `GET` на следующий адрес.
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/glossaries/formats
```

Узнайте, как найти собственное [доменное имя](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Для всех запросов API к службе перевода документов требуется Пользовательская конечная точка домена**.
> * Вы не можете использовать конечную точку, найденную в портал Azure _ключах ресурсов и конечной_ точке глобального преобразователя, — `api.cognitive.microsofttranslator.com` для выполнения HTTP-запросов к преобразованию документов.

## <a name="request-headers"></a>Заголовки запросов

Заголовки запроса.

|Заголовки|Описание|
|--- |--- |
|Ocp-Apim-Subscription-Key|Обязательный заголовок запроса|

## <a name="response-status-codes"></a>Коды состояния ответа

Ниже приведены возможные коды состояния HTTP, которые возвращает запрос.

|Код состояния|Описание|
|--- |--- |
|200|Все в порядке. Возвращает список поддерживаемых форматов файлов глоссария.|
|500|Внутренняя ошибка сервера.|
|Другие коды состояния|<ul><li>Слишком много запросов</li><li>Сервер временно недоступен</li></ul>|


## <a name="get-glossary-formats-response"></a>Получение ответа на форматы глоссария

Базовый тип для List return в API получения формата глоссария.

### <a name="successful-get-glossary-formats-response"></a>Ответ на получение формата глоссария успешно выполнен

Базовый тип для List return в API получения формата глоссария.

|Код состояния|Описание|
|--- |--- |
|200|Все в порядке. Возвращает список поддерживаемых форматов файлов глоссария.|
|500|Внутренняя ошибка сервера.|
|Другие коды состояния|Слишком много временных временно недоступных Рекуестссервер|

### <a name="error-response"></a>Сообщение об ошибке

|Имя|Тип|Описание|
|--- |--- |--- |
|code|строка|Перечисления, содержащие коды ошибок высокого уровня. Возможные значения:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>рекуестратетухигх</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>Не авторизовано</li></ul>|
|message|строка|Возвращает сообщение об ошибке высокого уровня.|
|innerError|InnerErrorV2|Новый формат внутренней ошибки, который соответствует рекомендациям Cognitive Services API. Он содержит обязательные свойства ErrorCode, Message и необязательные свойства Target, Details (пара значений ключа), внутреннюю ошибку (может быть вложенной).|
|Иннереррор. Code|строка|Возвращает строку ошибки кода.|
|Иннереррор. Message|строка|Возвращает сообщение об ошибке высокого уровня.|

## <a name="examples"></a>Примеры

### <a name="example-successful-response"></a>Пример успешного ответа

Ниже приведен пример успешного ответа.

```JSON
{
    "value": [
        {
            "format": "XLIFF",
            "fileExtensions": [
                ".xlf"
            ],
            "contentTypes": [
                "application/xliff+xml"
            ],
            "versions": [
                "1.0",
                "1.1",
                "1.2"
            ]
        },
        {
            "format": "TSV",
            "fileExtensions": [
                ".tsv",
                ".tab"
            ],
            "contentTypes": [
                "text/tab-separated-values"
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
