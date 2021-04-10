---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 12/09/2020
ms.topic: include
ms.openlocfilehash: 269cae85c0f007c487f063b1f7ca7ea4dca15376
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105363"
---
Начало работы с REST API Пользовательского визуального распознавания Выполните описанные ниже действия, чтобы вызвать API и создать модель классификации изображений. Здесь объясняется, как создать проект, добавить теги, обучить проект и использовать URL-адрес конечной точки прогнозирования проекта для тестирования программными средствами. Этот пример можно использовать как шаблон при создании собственного приложения для распознавания изображений.

> [!NOTE]
> Пользовательское визуальное распознавание проще всего использовать с помощью пакета SDK клиентской библиотеки. Также можно воспользоваться руководством по работе со [средствами на основе браузера](../../get-started-build-detector.md).

Используйте клиентскую библиотеку "Пользовательское визуальное распознавание" для .NET, чтобы выполнять такие действия:

* Создание проекта службы "Пользовательское визуальное распознавание"
* Добавление тегов в проект
* Отправка и снабжение тегами изображений
* Обучение проекта
* Публикация текущей итерации
* Тестирование конечной точки прогнозирования

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Если у вас есть подписка Azure, <a href="https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision"  title=""  target="_blank">создайте ресурс "Пользовательское визуальное распознавание"</a> на портале Azure, чтобы создать ресурс для обучения и прогнозирования, а также получить ключи и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Вам понадобится ключ и конечная точка из ресурсов, которые вы создаете, чтобы подключить ваше приложение к службе "Пользовательское визуальное распознавание". Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* [PowerShell версии 6.0 и выше](/powershell/scripting/install/installing-powershell-core-on-windows) или аналогичное приложение командной строки.


## <a name="create-a-new-custom-vision-project"></a>Создание проекта службы "Пользовательское визуальное распознавание"

Для создания проекта классификации изображений используйте команду, которая похожа на приведенную ниже. Созданный проект будет отображаться на [веб-сайте Пользовательской службы визуального распознавания](https://customvision.ai/).


:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="createproject":::

Скопируйте команду в текстовый редактор и внесите следующие изменения:

* Замените `{subscription key}` действительным ключом подписки API распознавания лиц.
* Замените `{endpoint}` конечной точкой, соответствующей ключу подписки.
   [!INCLUDE [subdomains-note](../../../../../includes/cognitive-services-custom-subdomains-note.md)]
* Замените `{name}` именем проекта.
* При необходимости задайте другие параметры URL-адреса, чтобы указать, какой тип модели будет использоваться в проекте. Варианты см. на странице, посвященной [API CreatProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae).

Вы получите примерно такой ответ JSON. Сохраните значение `"id"` проекта во временном расположении.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "useNegativeSet": true,
    "detectionParameters": "string",
    "imageProcessingSettings": {
      "augmentationMethods": {}
    }
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

## <a name="add-tags-to-the-project"></a>Добавление тегов в проект

Используйте следующую команду, чтобы определить теги, по которым будет проводиться обучение модели.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="createtag":::

* Еще раз вставьте собственный ключ и URL-адрес конечной точки.
* Замените `{projectId}` собственным идентификатором проекта.
* Замените `{name}` именем необходимого тега.

Повторите эту процедуру для всех тегов, которые нужно использовать в проекте. Если вы используете предоставленные примеры изображений, добавьте теги `"Hemlock"` и `"Japanese Cherry"`.

Вы получите примерно такой ответ JSON. Сохраните значение `"id"` для каждого тега во временном расположении.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "type": "Regular",
  "imageCount": 0
}
```

## <a name="upload-and-tag-images"></a>Отправка и снабжение тегами изображений

Теперь скачайте примеры изображений для этого проекта. Сохраните содержимое папки [примеров изображений](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ImageClassification/Images) на локальном устройстве.

> [!NOTE]
> Вам нужен более широкий набор изображений для выполнения обучения? Trove, проект Microsoft Garage, позволяет создавать и покупать наборы изображений для обучения. После сбора изображений их можно скачать, а затем импортировать в проект Пользовательского визуального распознавания обычным способом. Чтобы узнать больше, посетите [страницу Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3).

С помощью следующей команды отправьте изображения и примените теги: один раз используйте команду для изображений с тегом Hemlock, а другой — для изображений с тегом Japanese Cherry. Дополнительные варианты см. на странице, посвященной [API CreateImagesFromData](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb5).

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="uploadimages":::

* Еще раз вставьте собственный ключ и URL-адрес конечной точки.
* Замените `{projectId}` собственным идентификатором проекта.
* Замените `{tagArray}` идентификатором тега.
* Затем укажите в тексте запроса двоичные данные изображений, которые нужно пометить тегами.

## <a name="train-the-project"></a>Обучение проекта

Этот метод обучает модель, используя отправленные изображения с тегами, и возвращает идентификатор для текущей итерации проекта.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="trainproject":::

* Еще раз вставьте собственный ключ и URL-адрес конечной точки.
* Замените `{projectId}` собственным идентификатором проекта.
* Замените `{tagArray}` идентификатором тега.
* Затем укажите в тексте запроса двоичные данные изображений, которые нужно пометить тегами.
* При необходимости можно использовать другие параметры URL-адреса. Варианты см. на странице, посвященной [API TrainProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee1).

> [!TIP]
> Обучение с использованием выбранных тегов
>
> При необходимости вы можете выполнить обучение с использованием только некоторых из примененных тегов. Это может потребоваться, если вы еще не применили достаточное количество определенных тегов, но у вас достаточно других. Добавьте необязательное содержимое JSON в текст запроса. Заполните массив `"selectedTags"` идентификаторами необходимых тегов.
> ```json
> {
>   "selectedTags": [
>     "00000000-0000-0000-0000-000000000000"
>   ]
> }
> ```

Ответ JSON содержит сведения о вашем обученном проекте, включая идентификатор итерации (`"id"`). Сохраните это значение для следующего шага.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "status": "string",
  "created": "string",
  "lastModified": "string",
  "trainedAt": "string",
  "projectId": "00000000-0000-0000-0000-000000000000",
  "exportable": true,
  "exportableTo": [
    "CoreML"
  ],
  "domainId": "00000000-0000-0000-0000-000000000000",
  "classificationType": "Multiclass",
  "trainingType": "Regular",
  "reservedBudgetInHours": 0,
  "trainingTimeInMinutes": 0,
  "publishName": "string",
  "originalPublishResourceId": "string"
}
```

## <a name="publish-the-current-iteration"></a>Публикация текущей итерации

С помощью этого метода текущая итерация модели становится доступной для запросов. Используйте полученное имя модели в качестве ссылки для отправки запросов на прогнозирование. 

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="publish":::

* Еще раз вставьте собственный ключ и URL-адрес конечной точки.
* Замените `{projectId}` собственным идентификатором проекта.
* Замените `{iterationId}` идентификатором, возвращенным на предыдущем шаге.
* Замените `{publishedName}` именем, которое вы хотите присвоить модели прогнозирования.
* Замените `{predictionId}` своим идентификатором ресурса прогнозирования. Этот идентификатор можно найти на вкладке **Обзор** ресурса прогнозирования на портале Azure, он будет указан как **Идентификатор подписки**.
* При необходимости можно использовать другие параметры URL-адреса. См. статью, посвященную [API PublishIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fdded5).

## <a name="test-the-prediction-endpoint"></a>Тестирование конечной точки прогнозирования

Теперь с помощью приведенной ниже команды протестируйте обученную модель, отправив новое изображение для классификации с использованием тегов. Можно использовать изображение из папки Test с примерами файлов, скачанных ранее.

:::code language="shell" source="~/cognitive-services-quickstart-code/curl/custom-vision/image-classifier.sh" ID="publish":::

* Еще раз вставьте собственный ключ и URL-адрес конечной точки.
* Замените `{projectId}` собственным идентификатором проекта.
* Замените `{publishedName}` значением, которое вы использовали на предыдущем шаге.
* Добавьте двоичные данные локального изображения в текст запроса.
* При необходимости можно использовать другие параметры URL-адреса. См. статью, посвященную [API ClassifyImage](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Prediction_3.1/operations/5eb37d24548b571998fde5f3).

В полученном ответе JSON будут перечислены все теги, которые модель применила к изображению, а также оценки вероятности для каждого тега. 

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "project": "00000000-0000-0000-0000-000000000000",
  "iteration": "00000000-0000-0000-0000-000000000000",
  "created": "string",
  "predictions": [
    {
      "probability": 0.0,
      "tagId": "00000000-0000-0000-0000-000000000000",
      "tagName": "string",
      "boundingBox": {
        "left": 0.0,
        "top": 0.0,
        "width": 0.0,
        "height": 0.0
      },
      "tagType": "Regular"
    }
  ]
}
```

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы выполнили все этапы процесса классификации изображений с помощью REST API. В этом примере выполняется одна итерация обучения, но часто нужно несколько раз обучать и тестировать модель, чтобы сделать ее более точной.

> [!div class="nextstepaction"]
> [Тестирование и переобучение модели с помощью Пользовательской службы визуального распознавания](../../test-your-model.md)

* Что собой представляет Пользовательское визуальное распознавание
* [Справочная документация по API (обучение)](/dotnet/api/overview/azure/cognitiveservices/client/customvision)
* [Справочная документация по API (прогнозирование)](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae)