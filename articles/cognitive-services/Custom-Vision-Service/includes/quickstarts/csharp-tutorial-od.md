---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 09/15/2020
ms.openlocfilehash: 2b14c6e679a25ad5f37a9cb10bce78616126990b
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113036"
---
Начало работы с клиентской библиотекой "Пользовательское визуальное распознавание" для .NET. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для сборки модели обнаружения объектов. Здесь объясняется, как создать проект, добавить теги, обучить проект в примерах изображений и использовать URL-адрес конечной точки прогнозирования проекта для тестирования программными средствами. Этот пример можно использовать как шаблон при создании собственного приложения для распознавания изображений.

> [!NOTE]
> Если вы хотите создать и обучить модель обнаружения объекта _без_ написания кода, см. руководство по [работе со средствами на основе браузера](../../get-started-build-detector.md).

Используйте клиентскую библиотеку "Пользовательское визуальное распознавание" для .NET, чтобы выполнять такие действия:

* Создание проекта службы "Пользовательское визуальное распознавание"
* Добавление тегов в проект
* Отправка и снабжение тегами изображений
* Обучение проекта
* Публикация текущей итерации
* Тестирование конечной точки прогнозирования

[Справочная документация](/dotnet/api/overview/azure/cognitiveservices/client/customvision) | Исходный код библиотеки [(обучение)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.CustomVision.Training) [(прогноз)](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.CustomVision.Prediction) | Пакет (NuGet) [(обучение)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/) [(прогноз)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/) | [Примеры](/samples/browse/?products=azure&term=vision&terms=vision)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [IDE Visual Studio](https://visualstudio.microsoft.com/vs/) или текущая версия [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Если у вас есть подписка Azure, <a href="https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision"  title=""  target="_blank">создайте ресурс "Пользовательское визуальное распознавание"</a> на портале Azure, чтобы создать ресурс для обучения и прогнозирования, а также получить ключи и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Вам понадобится ключ и конечная точка из ресурсов, которые вы создаете, чтобы подключить ваше приложение к службе "Пользовательское визуальное распознавание". Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

#### <a name="visual-studio-ide"></a>[Интегрированная среда разработки Visual Studio](#tab/visual-studio)

С помощью Visual Studio создайте приложение .NET Core. 

### <a name="install-the-client-library"></a>Установка клиентской библиотеки 

После создания проекта установите клиентскую библиотеку, щелкнув правой кнопкой мыши решение проекта в **Обозревателе решений** и выбрав пункт **Управление пакетами NuGet**. В открывшемся диспетчере пакетов выберите **Просмотр**, установите флажок **Включить предварительные версии** и выполните поиск по запросу `Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training` и `Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction`. Выберите последнюю версию и затем нажмите **Установка**. 

#### <a name="cli"></a>[CLI](#tab/cli)

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `custom-vision-quickstart`. Эта команда создает простой проект "Hello World" на языке C# с одним файлом исходного кода: *program.cs*. 

```console
dotnet new console -n custom-vision-quickstart
```

Измените каталог на созданную папку приложения. Чтобы создать приложение, выполните следующую команду:

```console
dotnet build
```

Выходные данные сборки не должны содержать предупреждений или ошибок. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки 

В каталоге приложения установите клиентскую библиотеку "Пользовательское визуальное распознавание" для .NET с помощью следующей команды:

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

---

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/CustomVision/ObjectDetection/Program.cs), где размещены примеры кода для этого краткого руководства.

В каталоге проекта откройте файл *program.cs* и с помощью `using` добавьте следующие директивы:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_imports)]

В методе **Main** приложения создайте переменные для ключа и конечной точки вашего ресурса.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_creds)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурсы Пользовательского визуального распознавания, созданные в соответствии с указаниями в разделе **Предварительные требования**, успешно развернуты, нажмите кнопку **Go to Resource** (Перейти к ресурсу) в разделе **Дальнейшие действия**. Ключи и конечную точку можно найти на страницах ресурсов **ключа и конечной точки** в разделе **управления ресурсами**. Вам будет нужно получить ключи для учебных ресурсов и ресурсов прогнозирования, а также конечную точку API для учебного ресурса.
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

В методе **Main** приложения добавьте вызовы методов, используемых в этом кратком руководстве. Вы реализуете их позже.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_maincalls)]

## <a name="object-model"></a>Объектная модель

|Имя|Описание|
|---|---|
|[CustomVisionTrainingClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient) | Этот класс обрабатывает создание, обучение и публикацию ваших моделей. |
|[CustomVisionPredictionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient)| Этот класс обрабатывает запросы ваших моделей для прогнозов обнаружения объектов.|
|[PredictionModel](/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.prediction.models.predictionmodel)| Этот класс определяет прогнозирование одного объекта на одном изображении. Он содержит свойства для идентификатора объекта и его имени, расположение ограничивающего прямоугольника объекта и оценку достоверности.|

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки "Пользовательское визуальное распознавание" для .NET:

* [аутентификация клиента](#authenticate-the-client);
* [Создание проекта службы "Пользовательское визуальное распознавание"](#create-a-new-custom-vision-project)
* [Добавление тегов в проект](#add-tags-to-the-project)
* [Отправка и снабжение тегами изображений](#upload-and-tag-images)
* [Обучение проекта](#train-the-project)
* [Публикация текущей итерации](#publish-the-current-iteration)
* [Тестирование конечной точки прогнозирования](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте в новом методе экземпляры клиентов для обучения и прогноза с помощью ключей и конечной точки.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_auth)]

## <a name="create-a-new-custom-vision-project"></a>Создание проекта службы "Пользовательское визуальное распознавание"

Следующий метод создает проект обнаружения объектов. Созданный проект будет отображаться на [веб-сайте Пользовательской службы визуального распознавания](https://customvision.ai/). Ознакомьтесь с методом [CreateProject](/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.training.customvisiontrainingclientextensions.createproject#Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_CustomVisionTrainingClientExtensions_CreateProject_Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_ICustomVisionTrainingClient_System_String_System_String_System_Nullable_System_Guid__System_String_System_Collections_Generic_IList_System_String__&preserve-view=true), чтобы указать другие параметры при создании проекта (см. пояснения в руководстве по [созданию средства обнаружения объектов с помощью веб-портала](../../get-started-build-detector.md)).  

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_create)]

## <a name="add-tags-to-the-project"></a>Добавление тегов в проект

Этот метод определяет теги, по которым вы будете обучать модель.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_tags)]

## <a name="upload-and-tag-images"></a>Отправка и снабжение тегами изображений

Сначала загрузите примеры изображений для этого проекта. Сохраните содержимое папки [примеров изображений](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ObjectDetection/Images) на локальном устройстве.

> [!NOTE]
> Вам нужен более широкий набор изображений для выполнения обучения? Trove, проект Microsoft Garage, позволяет создавать и покупать наборы изображений для обучения. После сбора изображений их можно скачать, а затем импортировать в проект Пользовательского визуального распознавания обычным способом. Чтобы узнать больше, посетите [страницу Trove](https://www.microsoft.com/ai/trove?activetab=pivot1:primaryr3).

При добавлении тегов к изображениям в проектах обнаружения объектов вам нужно указать регион каждого помеченного объекта, используя нормализированные координаты. Следующий код связывает все примеры изображений с отмеченным регионом.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_upload_regions)]

> [!NOTE]
> Если у вас нет служебной программы для щелчка и перетаскивания для вашего проекта, пометьте координаты регионов с помощью пользовательского веб-интерфейса на веб-сайте [Пользовательского визуального распознавания](https://www.customvision.ai/). В этом примере координаты уже предоставлены.

Затем это сопоставление ассоциаций используется для отправки каждого примера изображения с координатами региона. В одном пакете можно передать до 64 изображений. Возможно вам потребуется изменить значение `imagePath`, чтобы оно указывало на правильные расположения папок.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_upload)]

На этом этапе вы передали все примеры изображений и пометили каждый из них (**вилка** или **ножницы**) со связанным пиксельным прямоугольником.

## <a name="train-the-project"></a>Обучение проекта

Этот метод создает первую обучающую итерацию в проекте. Он отправляет запрос в службу до завершения обучения.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_train)]

> [!TIP]
> Обучение с использованием выбранных тегов
>
> При необходимости вы можете выполнить обучение с использованием только некоторых из примененных тегов. Это может потребоваться, если вы еще не применили достаточное количество определенных тегов, но у вас достаточно других. В вызове [TrainProject](/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.training.customvisiontrainingclientextensions.trainproject#Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_CustomVisionTrainingClientExtensions_TrainProject_Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_ICustomVisionTrainingClient_System_Guid_System_String_System_Nullable_System_Int32__System_Nullable_System_Boolean__System_String_Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_Models_TrainingParameters_&preserve-view=true) используйте параметр *trainingParameters*. Создайте [TrainingParameters](/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.training.models.trainingparameters) и задайте для свойства **SelectedTags** список идентификаторов тегов, которые хотите использовать. Модель будет обучаться с распознаванием только тегов в этом списке.

## <a name="publish-the-current-iteration"></a>Публикация текущей итерации

С помощью этого метода текущая итерация модели становится доступной для запросов. Вы можете использовать имя модели в качестве ссылки для отправки запросов на прогнозирование. Введите собственное значение для `predictionResourceId`. Идентификатор ресурса прогнозирования можно найти на вкладке **Обзор** ресурса на портале Azure, указанный как **Идентификатор подписки**.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_publish)]


## <a name="test-the-prediction-endpoint"></a>Тестирование конечной точки прогнозирования

Этот метод загружает тестовое изображение, запрашивает конечную точку модели и выводит данные прогнозирования на консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/CustomVision/ObjectDetection/Program.cs?name=snippet_prediction)]

## <a name="run-the-application"></a>Выполнение приложения

#### <a name="visual-studio-ide"></a>[Интегрированная среда разработки Visual Studio](#tab/visual-studio)

Запустите приложение, нажав кнопку **Отладка** в верхней части окна интегрированной среды разработки.

#### <a name="cli"></a>[CLI](#tab/cli)

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```dotnet
dotnet run
```

---

По мере запуска приложения он должен открыть консольное окно и записать следующие данные вывода:

```console
Creating new project:
        Training
Done!

Making a prediction:
        fork: 98.2% [ 0.111609578, 0.184719115, 0.6607002, 0.6637112 ]
        scissors: 1.2% [ 0.112389535, 0.119195729, 0.658031344, 0.7023591 ]
```

Вы можете убедиться, что тестовое изображение (в **Images/Test/**) помечено соответствующим образом и что область обнаружения верна. На этом этапе нажмите любую клавишу для выхода из приложения.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы выполнили в коде каждый шаг процесса обнаружения объектов. В этом примере выполняется одна итерация обучения, но часто нужно несколько раз обучать и тестировать модель, чтобы сделать ее более точной. В следующем руководстве объясняется классификация изображений, но ее принципы сходны с поиском объекта.

> [!div class="nextstepaction"]
> [Тестирование и переобучение модели с помощью Пользовательской службы визуального распознавания](../../test-your-model.md)

* Что собой представляет Пользовательское визуальное распознавание
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/CustomVision/ObjectDetection/Program.cs)
* [Справочная документация по пакету SDK](/dotnet/api/overview/azure/cognitiveservices/client/customvision)
