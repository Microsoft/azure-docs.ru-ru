---
author: PatrickFarley
ms.custom: devx-track-java
ms.author: pafarley
ms.service: cognitive-services
ms.date: 10/13/2020
ms.openlocfilehash: a5e2089682b0f9b886c16369bdd7565024690188
ms.sourcegitcommit: 740698a63c485390ebdd5e58bc41929ec0e4ed2d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/03/2021
ms.locfileid: "99500190"
---
Сведения о том, как начать работу с клиентской библиотекой Пользовательского визуального распознавания для Java и создать модель обнаружения объектов. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач. Этот пример можно использовать как шаблон при создании своего приложения для распознавания изображений.

> [!NOTE]
> Если вы хотите создать и обучить модель обнаружения объекта _без_ написания кода, см. руководство по [работе со средствами на основе браузера](../../get-started-build-detector.md).

Используйте клиентскую библиотеку Пользовательского визуального распознавания для Java, чтобы выполнять такие задачи:

* Создание проекта службы "Пользовательское визуальное распознавание"
* Добавление тегов в проект
* Отправка и снабжение тегами изображений
* Обучение проекта
* Публикация текущей итерации
* Тестирование конечной точки прогнозирования

[Справочная документация](/java/api/overview/azure/cognitiveservices/client/customvision) | Исходный код библиотеки [(обучение)](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-customvision-training) [(прогноз)](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-customvision-prediction) | Артефакт (Maven) [(обучение)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-training/1.1.0-preview.2/jar) [(прогноз)](https://search.maven.org/artifact/com.azure/azure-cognitiveservices-customvision-prediction/1.1.0-preview.2/jar) | 
[Примеры](/samples/browse/?products=azure&terms=custom%20vision)


## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [пакета средств разработки Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* [Средство сборки Gradle](https://gradle.org/install/) или другой диспетчер зависимостей.
* Если у вас есть подписка Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesCustomVision"  title=""  target="_blank">создайте ресурс "Пользовательское визуальное распознавание"<span class="docon docon-navigate-external x-hidden-focus"></span></a> на портале Azure, чтобы создать ресурс для обучения и прогнозирования, а также получить ключи и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Вам понадобится ключ и конечная точка из ресурсов, которые вы создаете, чтобы подключить ваше приложение к службе "Пользовательское визуальное распознавание". Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-gradle-project"></a>Создание проекта Gradle

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `gradle init` из рабочей папки. Эта команда создает необходимые файлы сборки для Gradle, включая *build.gradle.kts*, который используется во время выполнения для создания и настройки приложения.

```console
gradle init --type basic
```

Когда появится запрос на выбор **предметно-ориентированного языка**, выберите **Kotlin**.

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Найдите файл *build.gradle.kts* и откройте его в предпочитаемой интегрированной среде разработки или текстовом редакторе. Затем скопируйте и вставьте в файл приведенную ниже конфигурацию сборки. Эта конфигурация определяет проект как приложение Java, точкой входа которого является класс **CustomVisionQuickstart**. Он импортирует библиотеки Пользовательского визуального распознавания.

```kotlin
plugins {
    java
    application
}
application { 
    mainClassName = "CustomVisionQuickstart"
}
repositories {
    mavenCentral()
}
dependencies {
    compile(group = "com.azure", name = "azure-cognitiveservices-customvision-training", version = "1.1.0-preview.2")
    compile(group = "com.azure", name = "azure-cognitiveservices-customvision-prediction", version = "1.1.0-preview.2")
}
```

### <a name="create-a-java-file"></a>Создание файла Java


В рабочей папке выполните следующую команду, чтобы создать исходную папку проекта.

```console
mkdir -p src/main/java
```

Перейдите в новую папку и создайте файл с именем *CustomVisionQuickstart.java*. Откройте его в предпочитаемом редакторе или интегрированной среде разработки и добавьте следующие операторы `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java), где размещены примеры кода для этого краткого руководства.


В классе приложения **CustomVisionQuickstart** создайте переменные для ключа и конечной точки вашего ресурса.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_creds)]


> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурсы Пользовательского визуального распознавания, созданные в соответствии с указаниями в разделе **Предварительные требования**, успешно развернуты, нажмите кнопку **Go to Resource** (Перейти к ресурсу) в разделе **Дальнейшие действия**. Ключи и конечную точку можно найти на страницах ресурсов **ключа и конечной точки** в разделе **управления ресурсами**. Вам потребуется получить ключи обучения и прогнозирования, а также конечную точку учебных материалов.
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

В методе **main** приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут определены позже.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_maincalls_od)]

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы воплощают некоторые основные функции клиентской библиотеки Пользовательского визуального распознавания для Java.

|Имя|Описание|
|---|---|
|[CustomVisionTrainingClient](/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.training.customvisiontrainingclient) | Этот класс обрабатывает создание, обучение и публикацию ваших моделей. |
|[CustomVisionPredictionClient](/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.prediction.customvisionpredictionclient)| Этот класс обрабатывает запросы ваших моделей для прогнозов обнаружения объектов.|
|[ImagePrediction](/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.prediction.models.imageprediction)| Этот класс определяет прогнозирование одного объекта на одном изображении. Он содержит свойства для идентификатора объекта и его имени, расположение ограничивающего прямоугольника объекта и оценку достоверности.|

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки службы "Пользовательское визуальное распознавание" для Java:

* [аутентификация клиента](#authenticate-the-client);
* [Создание проекта службы "Пользовательское визуальное распознавание"](#create-a-new-custom-vision-project)
* [Добавление тегов в проект](#add-tags-to-the-project)
* [Отправка и снабжение тегами изображений](#upload-and-tag-images)
* [Обучение проекта](#train-the-project)
* [Публикация текущей итерации](#publish-the-current-iteration)
* [Тестирование конечной точки прогнозирования](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте в своем методе **main** экземпляры клиентов для обучения и прогноза с помощью ключей и конечной точки.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_auth)]

## <a name="create-a-new-custom-vision-project"></a>Создание проекта службы "Пользовательское визуальное распознавание"

Следующий метод создает проект обнаружения объектов. Созданный проект будет отображаться на [веб-сайте Пользовательской службы визуального распознавания](https://customvision.ai/), который вы посещали ранее. Ознакомьтесь с перегрузками метода [CreateProject](/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.training.trainings.createproject#com_microsoft_azure_cognitiveservices_vision_customvision_training_Trainings_createProject_String_CreateProjectOptionalParameter_&preserve-view=true), чтобы указать другие параметры при создании проекта (см. пояснения в руководстве по [созданию средства обнаружения объектов с помощью веб-портала](../../get-started-build-detector.md)).

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_create_od)]

## <a name="add-tags-to-your-project"></a>Добавление тегов в проект

Этот метод определяет теги, по которым вы будете обучать модель.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_tags_od)]

## <a name="upload-and-tag-images"></a>Отправка и снабжение тегами изображений

Сначала загрузите примеры изображений для этого проекта. Сохраните содержимое папки [примеров изображений](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ObjectDetection/Images) на локальном устройстве.

> [!NOTE]
> Вам нужен более широкий набор изображений для выполнения обучения? Trove, проект Microsoft Garage, позволяет создавать и покупать наборы изображений для обучения. После сбора изображений их можно скачать, а затем импортировать в проект Пользовательского визуального распознавания обычным способом. Чтобы узнать больше, посетите [страницу Trove](https://www.microsoft.com/en-us/ai/trove?activetab=pivot1:primaryr3).

При добавлении тегов к изображениям в проектах обнаружения объектов вам нужно указать регион каждого помеченного объекта, используя нормализированные координаты. Следующий код связывает все примеры изображений с отмеченным регионом.

> [!NOTE]
> Если у вас нет служебной программы для щелчка и перетаскивания, пометьте координаты регионов с помощью пользовательского веб-интерфейса в [Customvision.ai](https://www.customvision.ai/). В этом примере координаты уже предоставлены.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_od_mapping)]

Следующий блок кода добавляет изображения в проект. Вам необходимо изменить аргументы вызовов `GetImage`, чтобы они указывали на расположения скачанных вами папок **fork** и **scissors**.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_upload_od)]

Предыдущий фрагмент кода применяет две вспомогательные функции, которые извлекают изображения как потоки данных и загружают их в службу (можно передать до 64 изображений в одном пакете). Определите такие методы. 

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_helpers)]

## <a name="train-the-project"></a>Обучение проекта

Этот метод создает первую обучающую итерацию в проекте. Он отправляет запрос в службу до завершения обучения.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_train_od)]


## <a name="publish-the-current-iteration"></a>Публикация текущей итерации

С помощью этого метода текущая итерация модели становится доступной для запросов. Вы можете использовать имя модели в качестве ссылки для отправки запросов на прогнозирование. Введите собственное значение для `predictionResourceId`. Идентификатор ресурса прогнозирования можно найти на вкладке **Обзор** ресурса на портале Azure, указанный как **Идентификатор подписки**.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_publishOD)]

## <a name="test-the-prediction-endpoint"></a>Тестирование конечной точки прогнозирования

Этот метод загружает тестовое изображение, запрашивает конечную точку модели и выводит данные прогнозирования на консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_predict_od)]

## <a name="run-the-application"></a>Выполнение приложения

Чтобы создать приложение, выполните следующую команду:

```console
gradle build
```

Запустите приложение, выполнив команду `gradle run`:

```console
gradle run
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы выполнили в коде каждый шаг процесса обнаружения объектов. В этом примере выполняется одна итерация обучения, но часто нужно несколько раз обучать и тестировать модель, чтобы сделать ее более точной. В следующем руководстве объясняется классификация изображений, но ее принципы сходны с поиском объекта.

> [!div class="nextstepaction"]
> [Тестирование и переобучение модели с помощью Пользовательской службы визуального распознавания](../../test-your-model.md)

* Что собой представляет Пользовательское визуальное распознавание
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java)
