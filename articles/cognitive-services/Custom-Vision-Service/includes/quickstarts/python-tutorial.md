---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 10/25/2020
ms.openlocfilehash: c60c0326018e615a0c84d56c98faee58560f1d87
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113471"
---
Начало работы с клиентской библиотекой службы "Пользовательское визуальное распознавание" для Python. Выполните следующие действия, чтобы установить пакет и протестировать пример кода для построения модели по классификации изображений. Здесь объясняется, как создать проект, добавить теги, обучить проект и использовать URL-адрес конечной точки прогнозирования проекта для тестирования программными средствами. Этот пример можно использовать как шаблон при создании собственного приложения для распознавания изображений.

> [!NOTE]
> Если вы хотите создать и обучить модель классификации _без_ написания кода, см. руководство по [работе со средствами на основе браузера](../../getting-started-build-a-classifier.md).

Используйте клиентскую библиотеку службы "Пользовательское визуальное распознавание" для Python, чтобы выполнять такие действия:

* Создание проекта службы "Пользовательское визуальное распознавание"
* Добавление тегов в проект
* Отправка и снабжение тегами изображений
* Обучение проекта
* Публикация текущей итерации
* Тестирование конечной точки прогнозирования

[Справочная документация](/python/api/overview/azure/cognitiveservices/customvision) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-customvision/azure/cognitiveservices/vision/customvision) | [Пакет (PyPI)](https://pypi.org/project/azure-cognitiveservices-vision-customvision/) | [Примеры](/samples/browse/?languages=python&products=azure&term=vision&terms=vision)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [Python 3.x](https://www.python.org/)
  * Установка Python должна включать [pip](https://pip.pypa.io/en/stable/). Чтобы проверить, установлен ли pip, выполните команду `pip --version` в командной строке. Чтобы использовать pip, установите последнюю версию Python.
* Если у вас есть подписка Azure, <a href="https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision"  title="создайте ресурс "Пользовательское визуальное распознавание""  target="_blank">Создание ресурса "Пользовательское визуальное распознавание"</a> на портале Azure, чтобы создать ресурс для обучения и прогнозирования, а также получить ключи и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Вам понадобится ключ и конечная точка из ресурсов, которые вы создаете, чтобы подключить ваше приложение к службе "Пользовательское визуальное распознавание". Эти ключи и конечную точку вы позже вставите в код, приведенный ниже в этом кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Чтобы написать приложение для анализа изображений с помощью службы "Пользовательское визуальное распознавание" для Python, вам потребуется клиентская библиотека этой службы. После установки Python выполните следующую команду в PowerShell или в окне консоли:

```powershell
pip install azure-cognitiveservices-vision-customvision
```

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте файл Python и импортируйте следующие библиотеки.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/CustomVision/ImageClassification/CustomVisionQuickstart.py), где размещены примеры кода для этого краткого руководства.

Создайте переменные для конечной точки ресурса Azure и ключей подписки.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_creds)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурсы Пользовательского визуального распознавания, созданные в соответствии с указаниями в разделе **Предварительные требования**, успешно развернуты, нажмите кнопку **Go to Resource** (Перейти к ресурсу) в разделе **Дальнейшие действия**. Ключи и конечную точку можно найти на страницах **ключа и конечной точки** ресурсов. Вам будет нужно получить ключи для учебных ресурсов и ресурсов прогнозирования, а также конечную точку API для учебного ресурса.
>
> Значение идентификатора для ресурса прогнозирования будет указано в поле **Идентификатор подписки** на вкладке **Свойства**.
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

## <a name="object-model"></a>Объектная модель

|Имя|Описание|
|---|---|
|[CustomVisionTrainingClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient) | Этот класс обрабатывает создание, обучение и публикацию ваших моделей. |
|[CustomVisionPredictionClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient)| Этот класс обрабатывает запросы ваших моделей для прогноза классификаций изображений.|
|[ImagePrediction](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.models.imageprediction)| Этот класс определяет прогнозирование одного объекта на одном изображении. Он содержит свойства для идентификатора объекта и его имени, расположение ограничивающего прямоугольника объекта и оценку достоверности.|

## <a name="code-examples"></a>Примеры кода

В этих фрагментах кода показано, как выполнить следующие действия с помощью клиентской библиотеки службы "Пользовательское визуальное распознавание" для Python:

* [аутентификация клиента](#authenticate-the-client);
* [Создание проекта службы "Пользовательское визуальное распознавание"](#create-a-new-custom-vision-project)
* [Добавление тегов в проект](#add-tags-to-the-project)
* [Отправка и снабжение тегами изображений](#upload-and-tag-images)
* [Обучение проекта](#train-the-project)
* [Публикация текущей итерации](#publish-the-current-iteration)
* [Тестирование конечной точки прогнозирования](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр клиента для обучения и прогноза, используя ключи и конечную точку. Создайте объекты **ApiKeyServiceClientCredentials** со своими ключами и используйте их с конечной точкой для создания объекта [CustomVisionTrainingClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient) и [CustomVisionPredictionClient](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient).

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_auth)]

## <a name="create-a-new-custom-vision-project"></a>Создание проекта службы "Пользовательское визуальное распознавание"

Добавьте в скрипт следующий код, чтобы создать проект Пользовательской службы визуального распознавания. 

Ознакомьтесь с методом [create_project](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.operations.customvisiontrainingclientoperationsmixin#create-project-name--description-none--domain-id-none--classification-type-none--target-export-platforms-none--custom-headers-none--raw-false----operation-config-), чтобы указать другие параметры при создании проекта (см. пояснения в руководстве по [созданию классификатора с помощью веб-портала](../../getting-started-build-a-classifier.md)).  

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_create)]

## <a name="add-tags-to-the-project"></a>Добавление тегов в проект

Чтобы добавить теги классификации в проект, добавьте следующий код:

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_tags)]


## <a name="upload-and-tag-images"></a>Отправка и снабжение тегами изображений

Сначала загрузите примеры изображений для этого проекта. Сохраните содержимое папки [примеров изображений](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ImageClassification/Images) на локальном устройстве.

> [!NOTE]
> Вам нужен более широкий набор изображений для выполнения обучения? Trove, проект Microsoft Garage, позволяет создавать и покупать наборы изображений для обучения. После сбора изображений их можно скачать, а затем импортировать в проект Пользовательского визуального распознавания обычным способом. Чтобы узнать больше, посетите [страницу Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3).

Чтобы добавить примеры изображений в проект, вставьте следующий код после создания тегов. Этот код загружает каждое изображение с соответствующим тегом. В одном пакете можно передать до 64 изображений.

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_upload)]

> [!NOTE]
> Вам также понадобится изменить путь к изображениям в зависимости от того, куда вы скачали репозиторий с примерами для пакета SDK Cognitive Services для Python.

## <a name="train-the-project"></a>Обучение проекта

Этот код создает первую итерацию модели прогнозирования. 

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_train)]

> [!TIP]
> Обучение с использованием выбранных тегов
>
> При необходимости вы можете выполнить обучение с использованием только некоторых из примененных тегов. Это может потребоваться, если вы еще не применили достаточное количество определенных тегов, но у вас достаточно других. В вызове **[train_project](/python/api/azure-cognitiveservices-vision-customvision/azure.cognitiveservices.vision.customvision.training.operations.customvisiontrainingclientoperationsmixin#train-project-project-id--training-type-none--reserved-budget-in-hours-0--force-train-false--notification-email-address-none--selected-tags-none--custom-headers-none--raw-false----operation-config-&preserve-view=true)** задайте для необязательного параметра *selected_tags* список строк с идентификаторами тегов, которые хотите использовать. Модель будет обучаться с распознаванием только тегов в этом списке.

## <a name="publish-the-current-iteration"></a>Публикация текущей итерации

Итерация недоступна в конечной точке прогнозирования, пока она не будет опубликована. С помощью следующего кода текущая итерация модели становится доступной для запросов. 

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_publish)]


## <a name="test-the-prediction-endpoint"></a>Тестирование конечной точки прогнозирования

Чтобы отправить изображение в конечную точку прогнозирования и извлечь прогнозирование, добавьте в конец файла следующий код:

[!code-python[](~/cognitive-services-quickstart-code/python/CustomVision/ImageClassification/CustomVisionQuickstart.py?name=snippet_test)]

## <a name="run-the-application"></a>Выполнение приложения

Запустите *CustomVisionQuickstart.py*.

```powershell
python CustomVisionQuickstart.py
```

Выходные данные приложения должны выглядеть приблизительно так:

```console
Creating project...
Adding images...
Training...
Training status: Training
Training status: Completed
Done!
        Hemlock: 93.53%
        Japanese Cherry: 0.01%
```

Вы можете убедиться, что тестовое изображение (в **<base_image_location>/images/Test/** ) отмечено соответствующим образом. Вы также можете вернуться к [веб-сайту Пользовательской службы визуального распознавания](https://customvision.ai) и просмотреть текущее состояние созданного проекта.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как выполнять в коде каждый этап классификации изображений. В этом примере выполняется одна итерация обучения, но часто нужно несколько раз обучать и тестировать модель, чтобы сделать ее более точной.

> [!div class="nextstepaction"]
> [Тестирование и переобучение модели с помощью Пользовательской службы визуального распознавания](../../test-your-model.md)

* Что собой представляет Пользовательское визуальное распознавание
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/CustomVision/ImageClassification/CustomVisionQuickstart.py)
* [Справочная документация по пакету SDK](/python/api/overview/azure/cognitiveservices/customvision)
