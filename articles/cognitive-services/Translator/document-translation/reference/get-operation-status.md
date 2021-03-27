---
title: Состояние операции получения перевода документа
titleSuffix: Azure Cognitive Services
description: Метод Get operation Status возвращает состояние запроса перевода документа.
services: cognitive-services
author: jann-skotdal
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 03/25/2021
ms.author: v-jansk
ms.openlocfilehash: e8aa9b93dc089cc05dd5157a7ac84cceb5f6fcd5
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105613086"
---
# <a name="document-translation-get-operation-status"></a>Перевод документа: получение состояния операции

Метод Get операции Documents Status возвращает состояние запроса перевода документа. Состояние включает общее состояние запроса и состояние документов, которые транслируются в рамках этого запроса.

## <a name="request-url"></a>URL-адрес запроса

Отправьте запрос `GET` на следующий адрес.
```HTTP
GET https://<NAME-OF-YOUR-RESOURCE>.cognitiveservices.azure.com/translator/text/batch/v1.0-preview.1/batches/{id}
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
|идентификатор|True|ИДЕНТИФИКАТОР операции.|

## <a name="request-headers"></a>Заголовки запросов

Заголовки запроса.

|Заголовки|Описание|
|--- |--- |
|Ocp-Apim-Subscription-Key|Обязательный заголовок запроса|

## <a name="response-status-codes"></a>Коды состояния ответа

Ниже приведены возможные коды состояния HTTP, которые возвращает запрос.

|Код состояния|Описание|
|--- |--- |
|200|Все в порядке. Успешный запрос и возврат состояния операции преобразования пакета. Хеадерсретри-After: Интежеретаг: строка|
|401|Не авторизовано. Проверьте учетные данные.|
|404|Ресурс не найден.|
|500|Внутренняя ошибка сервера.|
|Другие коды состояния|<ul><li>Слишком много запросов</li><li>Сервер временно недоступен</li></ul>|

## <a name="get-operation-status-response"></a>Получение ответа состояния операции

### <a name="successful-get-operation-status-response"></a>Ответ об успешном получении состояния операции

При успешном отклике возвращается следующая информация.

|Имя|Тип|Описание|
|--- |--- |--- |
|идентификатор|строка|Идентификатор операции.|
|креатеддатетимеутк|строка|Дата и время создания операции.|
|ластактиондатетимеутк|строка|Дата и время обновления состояния операции.|
|status|Строка|Список возможных состояний для задания или документа: <ul><li>Отменено</li><li>Cancelling</li><li>Сбой</li><li>NotStarted</li><li>Запущен</li><li>Выполнено</li><li>ValidationFailed,</li></ul>|
|Итоги|статуссуммари|Сводка, содержащая приведенные ниже сведения.|
|Сводка. Итого|Целое число|Общее количество.|
|Сводка. сбой|Целое число|Число сбоев.|
|Сводка. успешное завершение|Целое число|Количество успешных выполнений.|
|Сводка. выполняется|Целое число|Число выполняющихся.|
|Summary. Нотетстартед|Целое число|Число еще не запущенных.|
|Сводка. отменено|Целое число|Число отмененных.|
|Summary. Тоталчарактерчаржед|Целое число|Всего символов, оплаченных API.|

###<a name="error-response"></a>Сообщение об ошибке

|Имя|Тип|Описание|
|--- |--- |--- |
|code|строка|Перечисления, содержащие коды ошибок высокого уровня. Возможные значения:<br/><ul><li>InternalServerError</li><li>InvalidArgument</li><li>InvalidRequest</li><li>рекуестратетухигх</li><li>ResourceNotFound</li><li>ServiceUnavailable</li><li>Не авторизовано</li></ul>|
|message|строка|Возвращает сообщение об ошибке высокого уровня.|
|target|строка|Возвращает источник ошибки. Например, это будет "Documents" или "Document ID" для недопустимого документа.|
|innerError|InnerErrorV2|Новый формат внутренней ошибки, который соответствует рекомендациям Cognitive Services API. Он содержит обязательные свойства ErrorCode, Message и необязательные свойства Target, Details (пара значений ключа), внутреннюю ошибку (может быть вложенным).|
|Иннереррор. Code|строка|Возвращает строку ошибки кода.|
|Иннереррор. Message|строка|Возвращает сообщение об ошибке высокого уровня.|

## <a name="examples"></a>Примеры

### <a name="example-successful-response"></a>Пример успешного ответа

Следующий объект JSON является примером успешного ответа.

```JSON
{
  "id": "727bf148-f327-47a0-9481-abae6362f11e",
  "createdDateTimeUtc": "2020-03-26T00:00:00Z",
  "lastActionDateTimeUtc": "2020-03-26T01:00:00Z",
  "status": "Succeeded",
  "summary": {
    "total": 10,
    "failed": 1,
    "success": 9,
    "inProgress": 0,
    "notYetStarted": 0,
    "cancelled": 0,
    "totalCharacterCharged": 0
  }
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
