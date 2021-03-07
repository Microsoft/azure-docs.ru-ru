---
title: Руководство по аварийному восстановлению для распознавателя форм Azure
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать API копирования модели для резервного копирования ресурсов распознавателя форм.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 05/27/2020
ms.author: pafarley
ms.openlocfilehash: 0343402d92498bff56250027086cbf2ceb258f0f
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102427170"
---
# <a name="back-up-and-recover-your-form-recognizer-models"></a>Резервное копирование и восстановление моделей распознавателя форм

При создании ресурса распознавателя форм в портал Azure указывается регион. После этого ваш ресурс и все его операции остаются связанными с этим конкретным регионом сервера Azure. Это случается редко, но не возможно, чтобы столкнуться с сетевой проблемой, которая достигает всего региона. Если решение всегда должно быть доступно, необходимо разработать его для отработки отказа в другой регион или разделить рабочую нагрузку между двумя или более регионами. Для обоих подходов требуется по крайней мере два ресурса распознавателя форм в разных регионах и возможность синхронизации пользовательских моделей в регионах.

API копирования включает этот сценарий, позволяя копировать пользовательские модели из одной учетной записи распознавателя форм или в другие, которые могут существовать в любом поддерживаемом географическом регионе. В этом руководство показано, как использовать копирование REST API с помощью фигурного копирования. Можно также использовать службу HTTP-запросов, например POST, чтобы выдать запросы.

## <a name="business-scenarios"></a>Бизнес-сценарии

Если приложение или бизнес зависит от использования настраиваемой модели распознавателя форм, рекомендуется скопировать модель в другую учетную запись распознавателя форм в другом регионе. Если происходит региональный сбой, можно получить доступ к модели в регионе, в котором она была скопирована.

##  <a name="prerequisites"></a>Предварительные требования

1. Ресурсы Azure для распознавания двух форм в разных регионах Azure. Если у вас их нет, перейдите к портал Azure и <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer" title=" Создайте новый ресурс распознавателя форм, чтобы " target="_blank"> создать новый ресурс распознавателя форм </a> .
1. Ключ подписки, URL-адрес конечной точки и идентификатор подписки ресурса распознавателя форм. Эти значения можно найти на вкладке **Обзор** ресурса на портал Azure.


## <a name="copy-api-overview"></a>Обзор API копирования

Процесс копирования пользовательской модели состоит из следующих этапов.

1. Сначала вы выдаете запрос на копирование авторизации в целевой ресурс, то &mdash; есть ресурс, который получит скопированную модель. Вы получаете URL-адрес вновь созданной целевой модели, которая получит скопированные данные.
1. Затем отправьте запрос на копирование в исходный ресурс, который &mdash; содержит ресурс, содержащий копируемую модель. Вы получите URL-адрес, который можно запросить для отслеживания хода выполнения операции.
1. Учетные данные исходного ресурса будут использоваться для запроса URL-адреса хода выполнения до тех пор, пока операция не завершится успешно. Можно также запросить новый идентификатор модели в целевом ресурсе, чтобы получить состояние новой модели.

> [!CAUTION]
> В настоящее время API копирования не поддерживает идентификаторы моделей для [составных пользовательских моделей](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/Compose). Создание модели — это предварительная версия функции в версии 2.1 – Preview. 2. 

## <a name="generate-copy-authorization-request"></a>Создать запрос на авторизацию копирования

Следующий HTTP-запрос получает разрешение Copy на копирование из целевого ресурса. Необходимо ввести конечную точку и ключ целевого ресурса в качестве заголовков.

```
POST https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization HTTP/1.1
Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Вы получите `201\Created` ответ со `modelId` значением в тексте. Эта строка представляет собой идентификатор вновь созданной (пустой) модели. Объект `accessToken` необходим для того, чтобы API скопировал данные в этот ресурс, а `expirationDateTimeTicks` значение — истечение срока действия маркера. Сохраните все три значения в безопасном месте.

```
HTTP/1.1 201 Created
Location: https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/33f4d42c-cd2f-4e74-b990-a1aeafab5a5d
{"modelId":"<your model ID>","accessToken":"<your access token>","expirationDateTimeTicks":637233481531659440}
```

## <a name="start-copy-operation"></a>Запуск операции копирования

Следующий HTTP-запрос запускает операцию копирования в исходном ресурсе. Необходимо ввести конечную точку и ключ исходного ресурса в качестве заголовков. Обратите внимание, что URL-адрес запроса содержит идентификатор модели исходной модели, которую необходимо скопировать.

```
POST https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/<your model ID>/copy HTTP/1.1
Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Текст запроса должен иметь следующий формат. Необходимо ввести идентификатор ресурса и имя региона целевого ресурса. Идентификатор ресурса можно найти на вкладке **Свойства** ресурса в портал Azure, а имя региона можно найти на вкладке **ключи и конечная точка** . Вам также потребуется идентификатор модели, маркер доступа и значение срока действия, скопированные на предыдущем шаге.

```json
{
   "targetResourceId": "{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_ID}",  
   "targetResourceRegion": "{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_REGION_NAME}",
   "copyAuthorization": {"modelId":"<your model ID>","accessToken":"<your access token>","expirationDateTimeTicks":637233481531659440}
}
```

> [!NOTE]
> API копирования прозрачно поддерживает функцию [АЕК/CMK](https://msazure.visualstudio.com/Cognitive%20Services/_wiki/wikis/Cognitive%20Services.wiki/52146/Customer-Managed-Keys) . Это не требует специальной обработки, но обратите внимание, что при копировании между незашифрованным ресурсом в зашифрованный ресурс необходимо включить заголовок запроса `x-ms-forms-copy-degrade: true` . Если этот заголовок не включен, операция копирования завершится ошибкой и вернет значение `DataProtectionTransformServiceError` .

Вы получите `202\Accepted` ответ с заголовком Operation-Location. Это значение представляет собой URL-адрес, который будет использоваться для отслеживания хода выполнения операции. Скопируйте его во временную папку для следующего шага.

```
HTTP/1.1 202 Accepted
Operation-Location: https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/eccc3f13-8289-4020-ba16-9f1d1374e96f/copyresults/02989ba8-1296-499f-aaf4-55cfff41b8f1
```

### <a name="common-errors"></a>Распространенные ошибки

|Ошибка|Решение|
|:--|:--|
| 400/недопустимый запрос с `"code:" "1002"` | Указывает на ошибку проверки или некорректный запрос на копирование. Распространенные проблемы включают в себя: a) недопустимые или измененные `copyAuthorization` полезные данные. б) значение срока действия `expirationDateTimeTicks` токена ( `copyAuhtorization` полезные данные действительны в течение 24 часов). в) недопустимо или не поддерживается `targetResourceRegion` . d) Недопустимая или неправильная `targetResourceId` строка.
|

## <a name="track-copy-progress"></a>Отслеживать ход копирования

Отследите ход выполнения, запросив API **получения результатов модели копирования** для конечной точки исходного ресурса.

```
GET https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/eccc3f13-8289-4020-ba16-9f1d1374e96f/copyresults/02989ba8-1296-499f-aaf4-55cfff41b8f1 HTTP/1.1
Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Ваш ответ будет зависеть от состояния операции. Найдите `"status"` поле в тексте JSON. Если вы автоматизируем этот вызов API в скрипте, мы рекомендуем запросить операцию каждую секунду.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{"status":"succeeded","createdDateTime":"2020-04-23T18:18:01.0275043Z","lastUpdatedDateTime":"2020-04-23T18:18:01.0275048Z","copyResult":{}}
```

### <a name="common-errors"></a>Распространенные ошибки

|Ошибка|Решение|
|:--|:--|
|"Errors": [{"Code": "AuthorizationError",<br>"сообщение": "сбой авторизации из-за <br>отсутствуют или недопустимые утверждения авторизации. "}]   | Происходит при `copyAuthorization` изменении полезной нагрузки или содержимого с того, что было возвращено `copyAuthorization` API. Убедитесь, что полезные данные представляют собой точное содержимое, возвращенное из предыдущего `copyAuthorization` вызова.|
|"Errors": [{"Code": "AuthorizationError",<br>"сообщение": "не удалось получить авторизацию <br>метаданных. Если проблема не исчезнет, используйте другую <br>Целевая модель для копирования. "}] | Указывает, что `copyAuthorization` полезные данные повторно используются с запросом на копирование. Успешный запрос на копирование не позволит выполнять последующие запросы, использующие одни и те же `copyAuthorization` полезные данные. Если вы создаете отдельную ошибку (как показано ниже), а затем повторяете попытку копирования с теми же полезными данными авторизации, возникает эта ошибка. Решение — создать новые `copyAuthorization` полезные данные, а затем повторно выполнить запрос на копирование.|
|"Errors": [{"Code": "Датапротектионтрансформсервицееррор",<br>"Message": запрос на пересылку данных не разрешен <br>по мере понижения до менее надежной схемы защиты данных. Обратитесь к документации или обратитесь к администратору службы. <br>Дополнительные сведения. "}]    | Происходит при копировании между `AEK` разрешенным ресурсом и `AEK` невключенным ресурсом. Чтобы разрешить копирование зашифрованной модели в целевой объект в виде незашифрованного `x-ms-forms-copy-degrade: true` заголовка с запросом Copy.|
|"Errors": [{"Code": "Ресаурцересолвереррор",<br>"сообщение": "не удалось получить сведения для переданного ресурса с идентификатором"... ". Убедитесь, что ресурс является допустимым и существует в указанном регионе "westus2".. "}] | Указывает, что ресурс Azure, указанный параметром, не `targetResourceId` является допустимым ресурсом для пересчета или не существует. Чтобы устранить эту проблему, проверьте и повторно выдайте запрос на копирование.|


### <a name="optional-track-the-target-model-id"></a>Используемых Отслеживание идентификатора целевой модели 

Можно также использовать API **получения настраиваемой модели** для наблюдения за состоянием операции путем запроса к целевой модели. Вызовите этот API, используя идентификатор целевой модели, скопированный на первом шаге.

```
GET https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/33f4d42c-cd2f-4e74-b990-a1aeafab5a5d HTTP/1.1
Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

В тексте ответа вы увидите сведения о модели. Проверьте `"status"` состояние модели в поле.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{"modelInfo":{"modelId":"33f4d42c-cd2f-4e74-b990-a1aeafab5a5d","status":"ready","createdDateTime":"2020-02-26T16:59:28Z","lastUpdatedDateTime":"2020-02-26T16:59:34Z"},"trainResult":{"trainingDocuments":[{"documentName":"0.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"1.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"2.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"3.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"4.pdf","pages":1,"errors":[],"status":"succeeded"}],"errors":[]}}
```

## <a name="curl-sample-code"></a>Фигурный образец кода

В следующих фрагментах кода используется фигурная скобка для выполнения вызовов API, описанных в приведенных выше шагах. Вам по-прежнему придется заполнять идентификаторы моделей и сведения о подписке, относящиеся к собственным ресурсам.

### <a name="generate-copy-authorization-request"></a>Создать запрос на авторизацию копирования

```bash
curl -i -X POST "https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization" -H "Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}" 
```

### <a name="start-copy-operation"></a>Запуск операции копирования

```bash
curl -i -X POST "https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}" --data-ascii "{ \"targetResourceId\": \"{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_ID}\",   \"targetResourceRegion\": \"{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_REGION_NAME}\", \"copyAuthorization\": "{\"modelId\":\"33f4d42c-cd2f-4e74-b990-a1aeafab5a5d\",\"accessToken\":\"1855fe23-5ffc-427b-aab2-e5196641502f\",\"expirationDateTimeTicks\":637233481531659440}"}"
```

### <a name="track-copy-progress"></a>Отслеживать ход копирования

```bash
curl -i GET "https://<SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT>/formrecognizer/v2.0/custom/models/{SOURCE_MODELID}/copyResults/{RESULT_ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}"
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как использовать API копирования для резервного копирования пользовательских моделей в ресурс распознаватель вторичных форм. Далее изучите справочную документацию по API, чтобы узнать, что еще можно сделать с помощью распознавателя форм.
* [Справочная документация по REST API](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)
