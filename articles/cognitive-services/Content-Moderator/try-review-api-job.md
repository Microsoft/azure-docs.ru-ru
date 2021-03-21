---
title: Использование заданий по сопоставлению с консолью REST API Content Moderator
titleSuffix: Azure Cognitive Services
description: Используйте операции заданий API проверки для запуска заданий комплексной модерации контента изображений или текста в Azure Content Moderator.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: pafarley
ms.openlocfilehash: 924c21037a464770fac13c9b45ddcf261ff5a058
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96905184"
---
# <a name="define-and-use-moderation-jobs-api-console"></a>Определение и использование заданий по созадачам (консоль API)

Задание по созадаче служит разновидностью оболочки для функций контроля содержимого, рабочих процессов и проверок. В этом руководство показано, как использовать интерфейсы API для работы с другими заданиями для запуска и проверки заданий по добавлению содержимого. Когда вы понимаете структуру интерфейсов API, вы можете легко перенести эти вызовы на любую платформу, совместимую с остальными частями.

## <a name="prerequisites"></a>Предварительные условия

- Войдите или создайте учетную запись на сайте [средства проверки](https://contentmoderator.cognitive.microsoft.com/) Content Moderator.
- Используемых [Определите пользовательский рабочий процесс](./Review-Tool-User-Guide/Workflows.md) , который будет использоваться в задании. можно также использовать рабочий процесс по умолчанию.

## <a name="create-a-job"></a>Создание задания

Чтобы создать задание по созадаче, перейдите на страницу " [Задание" — "Создание](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c5) справочника по API" и нажмите кнопку для региона подписки. Вы можете найти свой регион в URL-адресе конечной точки на странице **учетные данные** [средства проверки](https://contentmoderator.cognitive.microsoft.com/). Запустится консоль API, где можно легко создавать и выполнять вызовы REST API.

![Выбор региона на странице "Job - Create" (Задание — создание)](images/test-drive-job-1.png)

### <a name="enter-rest-call-parameters"></a>Введите параметры вызова RESTFUL

Введите следующие значения для создания вызова RESTFUL:

- **теамнаме**: идентификатор команды, созданный при настройке учетной записи [средства проверки](https://contentmoderator.cognitive.microsoft.com/) (находится в поле **идентификатор** на экране учетных данных средства проверки).
- **ContentType**: это может быть "Image", "Text" или "Video".
- Идентификатор **ContentId**: строка настраиваемого идентификатора. эта строка передается в API и возвращается с помощью обратного вызова. Это полезно для связывания внутренних идентификаторов или метаданных с результатами задания по сопоставлению.
- **Workflowname**: имя рабочего процесса, созданного ранее (или "по умолчанию" для рабочего процесса по умолчанию).
- **Каллбаккендпоинт**: (необязательный) URL-адрес для получения сведений о обратном вызове по завершении проверки.
- **OCP-Apim-Subscription-Key**— ключ Content Moderator. Этот ключ можно найти на вкладке **Параметры** [средства проверки](https://contentmoderator.cognitive.microsoft.com).

### <a name="fill-in-the-request-body"></a>Заполните текст запроса

Текст вызова функции RESTFUL содержит одно поле, **контентвалуе**. Вставьте содержимое необработанного текста, если вы разрабатываете текст, или введите URL-адрес изображения или видео, если вы используете изображение или видео. Можно использовать следующий образец URL-адреса изображения: [https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg](https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg)

![Параметры запроса, заголовки и поле "Request body" (Текст запроса) в консоли "Job - Create" (Задание — создание)](images/job-api-console-inputs.PNG)

### <a name="submit-your-request"></a>Отправка запроса

Нажмите кнопку **Отправить**. Если операция выполнена, **состояние ответа** равно `200 OK` , а в поле **ответное содержимое** отображается идентификатор задания. Скопируйте этот идентификатор, чтобы использовать на следующих шагах.

![Идентификатор проверки в поле "Response content" (Содержимое ответа) в консоли "Review - Create" (Проверка — создание)](images/test-drive-job-3.PNG)

## <a name="get-job-status"></a>Получение состояния задания

Чтобы получить состояние и подробные сведения о выполняемом или завершенном задании, перейдите на страницу " [Задание"-получение](https://westus2.dev.cognitive.microsoft.com/docs/services/580519463f9b070e5c591178/operations/580519483f9b0709fc47f9c3) справочника по API и нажмите кнопку для вашего региона (регион, в котором находится ваш ключ).

![Задание — получение выбора региона](images/test-drive-region.png)

Введите параметры вызова RESTFUL, как показано в приведенном выше разделе. Для этого шага **JOBID** — это уникальная строка идентификатора, полученная при создании задания. Нажмите кнопку **Отправить**. Если операция выполнена, **состояние отклика** — `200 OK` , а в поле **ответное содержимое** отображается задание в формате JSON, как показано ниже:

```json
{  
  "Id":"2018014caceddebfe9446fab29056fd8d31ffe",
  "TeamName":"some team name",
  "Status":"Complete",
  "WorkflowId":"OCR",
  "Type":"Image",
  "CallBackEndpoint":"",
  "ReviewId":"201801i28fc0f7cbf424447846e509af853ea54",
  "ResultMetaData":[  
    {  
      "Key":"hasText",
      "Value":"True"
    },
    {  
      "Key":"ocrText",
      "Value":"IF WE DID \r\nALL \r\nTHE THINGS \r\nWE ARE \r\nCAPABLE \r\nOF DOING, \r\nWE WOULD \r\nLITERALLY \r\nASTOUND \r\nOURSELVE \r\n"
    }
  ],
  "JobExecutionReport":[  
    {  
      "Ts":"2018-01-07T00:38:29.3238715",
      "Msg":"Posted results to the Callbackendpoint: https://requestb.in/vxke1mvx"
    },
    {  
      "Ts":"2018-01-07T00:38:29.2928416",
      "Msg":"Job marked completed and job content has been removed"
    },
    {  
      "Ts":"2018-01-07T00:38:29.0856472",
      "Msg":"Execution Complete"
    },
    {  
      "Ts":"2018-01-07T00:38:26.7714671",
      "Msg":"Successfully got hasText response from Moderator"
    },
    {  
      "Ts":"2018-01-07T00:38:26.4181346",
      "Msg":"Getting hasText from Moderator"
    },
    {  
      "Ts":"2018-01-07T00:38:25.5122828",
      "Msg":"Starting Execution - Try 1"
    }
  ]
}
```

![Задание — получение ответа вызова RESTFUL](images/test-drive-job-5.png)

### <a name="examine-the-new-reviews"></a>Проверка новых проверок

Если задание содержимого привело к созданию проверки, его можно просмотреть в [средстве проверки](https://contentmoderator.cognitive.microsoft.com). Выберите **Проверка**  >   / **текст** изображения / **видео** (в зависимости от используемого содержимого). Должно отобразиться содержимое, готовое для просмотра человеком. После того как человеческий Модератор проверит назначенные им теги и данные прогноза и отправит окончательное решение об управлении, API заданий отправляет все эти сведения в указанную конечную точку обратного вызова.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как создавать и запрашивать задания по добавлению содержимого с помощью REST API. Затем интегрируйте задания в сквозной сценарий, например в учебник по внедрению [электронной коммерции](./ecommerce-retail-catalog-moderation.md) .