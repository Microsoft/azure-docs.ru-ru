---
title: Получение метода состояния документа при переводе документа
titleSuffix: Azure Cognitive Services
description: Метод получения сведений о состоянии документа возвращает состояние конкретного документа.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/25/2021
ms.author: v-jansk
ms.openlocfilehash: 79bc3d076c1a7e164cab9c3231b29be84370e04a
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105613152"
---
# <a name="document-translation-get-document-status"></a>Перевод документа: получение состояния документа

Метод получения сведений о состоянии документа возвращает состояние конкретного документа. Метод возвращает состояние перевода для конкретного документа на основе идентификатора запроса и идентификатора документа.

## <a name="request-url"></a>URL-адрес запроса

Отправьте запрос `GET` на следующий адрес.
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/batches/{id}/documents/{documentId}
```

Узнайте, как найти собственное [доменное имя](../get-started-with-document-translation.md#find-your-custom-domain-name).

> [!IMPORTANT]
>
> * **Для всех запросов API к службе перевода документов требуется Пользовательская конечная точка домена**.
> * Вы не можете использовать конечную точку, найденную в портал Azure _ключах ресурсов и конечной_ точке глобального преобразователя, — `api.cognitive.microsofttranslator.com` для выполнения HTTP-запросов к преобразованию документов.

## <a name="request-parameters"></a>Параметры запроса

В таблице ниже приведены параметры, которые передаются в строке запроса.

|Параметр запроса|Обязательно|Описание|
|--- |--- |--- |
|documentId|True|Идентификатор документа.|
|идентификатор|True|Идентификатор пакета.|
## <a name="request-headers"></a>Заголовки запросов

Заголовки запроса.

|Заголовки|Описание|
|--- |--- |
|Ocp-Apim-Subscription-Key|Обязательный заголовок запроса|

## <a name="response-status-codes"></a>Коды состояния ответа

Ниже приведены возможные коды состояния HTTP, которые возвращает запрос.

|Код состояния|Описание|
|--- |--- |
|200|Все в порядке. Успешный запрос, и он принимается службой. Возвращаются сведения об операции. Хеадерсретри-After: Интежеретаг: строка|
|401|Не авторизовано. Проверьте учетные данные.|
|404|Не найден. Ресурс не найден.|
|500|Внутренняя ошибка сервера.|
|Другие коды состояния|<ul><li>Слишком много запросов</li><li>Сервер временно недоступен</li></ul>|

## <a name="get-document-status-response"></a>Получение ответа о состоянии документа

### <a name="successful-get-document-status-response"></a>Ответ об успешном получении состояния документа

|Имя|Тип|Описание|
|--- |--- |--- |
|path|строка|Расположение документа или папки.|
|креатеддатетимеутк|строка|Дата и время создания операции.|
|ластактиондатетимеутк|строка|Дата и время обновления состояния операции.|
|status|Строка|Список возможных состояний для задания или документа: <ul><li>Отменено</li><li>Cancelling</li><li>Сбой</li><li>NotStarted</li><li>Запущен</li><li>Выполнено</li><li>ValidationFailed,</li></ul>|
|значение|строка|Код языка с двумя буквами для языка. См. список языков.|
|progress|number|Ход выполнения перевода, если он доступен|
|идентификатор|строка|Идентификатор документа.|
|чарактерчаржед|Целое число|Символы, оплаченные API.|

### <a name="error-response"></a>Сообщение об ошибке

|Имя|Тип|Описание|
|--- |--- |--- |
|code|строка|Перечисления, содержащие коды ошибок высокого уровня. Возможные значения:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>рекуестратетухигх</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>Не авторизовано</li></ul>|
|message|строка|Возвращает сообщение об ошибке высокого уровня.|
|innerError|InnerErrorV2|Новый формат внутренней ошибки, который соответствует рекомендациям Cognitive Services API. Он содержит обязательные свойства ErrorCode, Message и необязательные свойства Target, Details (пара значений ключа), внутреннюю ошибку (может быть вложенным).|
|Иннереррор. Code|строка|Возвращает строку ошибки кода.|
|Иннереррор. Message|строка|Возвращает сообщение об ошибке высокого уровня.|

## <a name="examples"></a>Примеры

### <a name="example-successful-response"></a>Пример успешного ответа
Следующий объект JSON является примером успешного ответа.

```JSON
{
  "path": "https://myblob.blob.core.windows.net/destinationContainer/fr/mydoc.txt",
  "createdDateTimeUtc": "2020-03-26T00:00:00Z",
  "lastActionDateTimeUtc": "2020-03-26T01:00:00Z",
  "status": "Running",
  "to": "fr",
  "progress": 0.1,
  "id": "273622bd-835c-4946-9798-fd8f19f6bbf2",
  "characterCharged": 0
}
```

### <a name="example-error-response"></a>Пример ответа на ошибку

Следующий объект JSON является примером ответа на ошибку. Схема для других кодов ошибок одинакова.

Код состояния: 401

```JSON
{
  "error": {
    "code": "Unauthorized",
    "message": "User is not authorized",
    "target": "Document",
    "innerError": {
      "code": "Unauthorized",
      "message": "Operation is not authorized"
    }
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше об использовании перевода документов и клиентской библиотеки, следуйте нашим инструкциям в этом кратком руководстве.

> [!div class="nextstepaction"]
> [Приступая к преобразованию документов](../get-started-with-document-translation.md)
