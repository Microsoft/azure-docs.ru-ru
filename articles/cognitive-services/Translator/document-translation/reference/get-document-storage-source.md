---
title: Метод перевода документа получить источник хранилища документов
titleSuffix: Azure Cognitive Services
description: Метод получения источника хранилища документов возвращает список поддерживаемых источников хранилища.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/25/2021
ms.author: v-jansk
ms.openlocfilehash: d6d8b1d08bbef1c37cbf0583022428037808580d
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105613147"
---
# <a name="document-translation-get-document-storage-source"></a>Преобразование документа: получение источника хранилища документов

Метод получения источника хранилища документов возвращает список источников или параметров хранилища, поддерживаемых службой преобразования документов.

## <a name="request-url"></a>URL-адрес запроса

Отправьте запрос `GET` на следующий адрес.
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/storagesources
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
|200|Все в порядке. Успешный запрос и возврат списка источников хранилища.|
|500|Внутренняя ошибка сервера.|
|Другие коды состояния|<ul><li>Слишком много запросов</li><li>Сервер временно недоступен</li></ul>|

## <a name="get-document-storage-source-response"></a>Получение ответа источника хранилища документов

### <a name="successful-get-document-storage-source-response"></a>Ответ источника хранилища документов успешно получен
Базовый тип для List возвращается в API получения источника хранилища документов.

|Имя|Тип|Описание|
|--- |--- |--- |
|value|String []|Список объектов.|


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
    "AzureBlob"
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