---
author: areddish
ms.author: areddish
ms.service: cognitive-services
ms.date: 10/26/2020
ms.custom: devx-track-js
ms.openlocfilehash: 07aff220e211578a6a4b18e8d88ff44a4cc39a86
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/16/2020
ms.locfileid: "97582646"
---
Это руководство содержит инструкции и пример кода, которые помогут вам начать работу с клиентской библиотекой Пользовательского визуального распознавания для Node.js и создать модель классификации изображений. Здесь объясняется, как создать проект, добавить теги, обучить проект и использовать URL-адрес конечной точки прогнозирования проекта для тестирования программными средствами. Этот пример можно использовать как шаблон при создании собственного приложения для распознавания изображений.

> [!NOTE]
> Если вы хотите создать и обучить модель классификации _без_ написания кода, см. руководство по [работе со средствами на основе браузера](../../getting-started-build-a-classifier.md).

Используйте клиентскую библиотеку "Пользовательское визуальное распознавание" для .NET, чтобы выполнять такие действия:

* Создание проекта службы "Пользовательское визуальное распознавание"
* Добавление тегов в проект
* Отправка и снабжение тегами изображений
* Обучение проекта
* Публикация текущей итерации
* Тестирование конечной точки прогнозирования

Справочная документация [(обучение)](/javascript/api/@azure/cognitiveservices-customvision-training/?view=azure-node-latest) [(прогноз)](/javascript/api/@azure/cognitiveservices-customvision-prediction/?view=azure-node-latest) | Исходный код библиотеки [(обучение)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-customvision-training) [(прогноз)](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-customvision-prediction) | Пакет (npm) [(обучение)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-training) [(прогноз)](https://www.npmjs.com/package/@azure/cognitiveservices-customvision-prediction) | [Примеры](/samples/browse/?products=azure&terms=custom%20vision&languages=javascript)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Текущая версия [Node.js](https://nodejs.org/)
* Если у вас есть подписка Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesCustomVision"  title=""  target="_blank">создайте ресурс "Пользовательское визуальное распознавание"<span class="docon docon-navigate-external x-hidden-focus"></span></a> на портале Azure, чтобы создать ресурс для обучения и прогнозирования, а также получить ключи и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Вам понадобится ключ и конечная точка из ресурсов, которые вы создаете, чтобы подключить ваше приложение к службе "Пользовательское визуальное распознавание". Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `npm init`, чтобы создать приложение узла с помощью файла `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Чтобы написать приложение для анализа изображений с помощью Пользовательского визуального распознавания для Node.js, вам потребуются пакеты NPM этой службы. Чтобы установить их, выполните в PowerShell следующую команду:

```shell
npm install @azure/cognitiveservices-customvision-training
npm install @azure/cognitiveservices-customvision-prediction
```

Файл `package.json` этого приложения будет дополнен зависимостями.

Создайте файл с именем `index.js` и импортируйте следующие библиотеки:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_imports)]


> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js), где размещены примеры кода для этого краткого руководства.

Создайте переменные для конечной точки Azure и ключей ресурса. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурсы обучения Пользовательскому визуальному распознаванию, созданные в соответствии с указаниями в разделе **Предварительные требования**, успешно развернуты, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

Кроме того, добавьте поля имени проекта и параметра времени ожидания для асинхронных вызовов.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_vars)]


## <a name="object-model"></a>Объектная модель

|Имя|Описание|
|---|---|
|[TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient?view=azure-node-latest) | Этот класс обрабатывает создание, обучение и публикацию ваших моделей. |
|[PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient?view=azure-node-latest)| Этот класс обрабатывает запросы ваших моделей для прогноза классификаций изображений.|
|[Прогнозирование](/javascript/api/@azure/cognitiveservices-customvision-prediction/prediction?view=azure-node-latest)| Этот интерфейс определяет один прогноз для одного изображения. Он содержит свойства идентификатора и имени объекта, а также оценку доверительного уровня.|

## <a name="code-examples"></a>Примеры кода

В этих фрагментах кода показано, как выполнить следующие действия с помощью клиентской библиотеки службы "Пользовательское визуальное распознавание" для JavaScript:

* [аутентификация клиента](#authenticate-the-client);
* [Создание проекта службы "Пользовательское визуальное распознавание"](#create-a-new-custom-vision-project)
* [Добавление тегов в проект](#add-tags-to-the-project)
* [Отправка и снабжение тегами изображений](#upload-and-tag-images)
* [Обучение проекта](#train-the-project)
* [Публикация текущей итерации](#publish-the-current-iteration)
* [Тестирование конечной точки прогнозирования](#test-the-prediction-endpoint)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляры клиентских объектов с использованием конечной точки и ключа. Создайте объект **ApiKeyCredentials** с ключом и используйте его с конечной точкой, чтобы создать объект [TrainingAPIClient](/javascript/api/@azure/cognitiveservices-customvision-training/trainingapiclient?view=azure-node-latest) и [PredictionAPIClient](/javascript/api/@azure/cognitiveservices-customvision-prediction/predictionapiclient?view=azure-node-latest).

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_auth)]


## <a name="create-a-new-custom-vision-project"></a>Создание проекта службы "Пользовательское визуальное распознавание"

Запустите новую функцию, чтобы она содержала все вызовы функций службы "Пользовательское визуальное распознавание". Добавьте следующий код, чтобы создать проект для использования службы "Пользовательское визуальное распознавание".


[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_create)]


## <a name="add-tags-to-the-project"></a>Добавление тегов в проект

Чтобы создать теги классификации в проекте, добавьте в функцию следующий код:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_tags)]


## <a name="upload-and-tag-images"></a>Отправка и снабжение тегами изображений

Сначала загрузите примеры изображений для этого проекта. Сохраните содержимое папки [примеров изображений](https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/CustomVision/ImageClassification/Images) на локальном устройстве.

Чтобы добавить примеры изображений в проект, вставьте следующий код после создания тегов. Этот код загружает каждое изображение с соответствующим тегом.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_upload)]

> [!IMPORTANT]
> Вам также понадобится изменить путь к изображениям (`sampleDataRoot`) в зависимости от того, куда вы скачали репозиторий с примерами для пакета SDK Cognitive Services для Python.

## <a name="train-the-project"></a>Обучение проекта

Этот код создает первую итерацию модели прогнозирования. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_train)]

## <a name="publish-the-current-iteration"></a>Публикация текущей итерации

Этот код позволяет опубликовать обученную итерацию в конечной точке прогнозирования. Имя, присвоенное опубликованной итерации, можно использовать для отправки запросов на прогнозирование. Итерация недоступна в конечной точке прогнозирования, пока она не будет опубликована.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_publish)]


## <a name="test-the-prediction-endpoint"></a>Тестирование конечной точки прогнозирования

Чтобы отправить изображение в конечную точку прогнозирования и получить результат прогнозирования, добавьте в функцию указанный ниже код. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_test)]

Затем добавьте в конец функции службы "Пользовательское визуальное распознавание" закрывающую фигурную скобку и вызовите функцию.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js?name=snippet_function_close)]


## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node` для файла quickstart.

```console
node index.js
```

Выходные данные приложения должны выглядеть приблизительно так:

```console
Creating project...
Adding images...
Training...
Training started...
Training status: Training
Training status: Training
Training status: Training
Training status: Completed
Results:
         Hemlock: 94.97%
         Japanese Cherry: 0.01%
```

Вы можете убедиться, что тестовое изображение (в **<sampleDataRoot>/Test/** ) помечено соответствующим образом. Вы также можете вернуться к [веб-сайту Пользовательской службы визуального распознавания](https://customvision.ai) и просмотреть текущее состояние созданного проекта.

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы узнали, как выполнять в коде каждый шаг процесса обнаружения объектов. В этом примере выполняется одна итерация обучения, но часто нужно несколько раз обучать и тестировать модель, чтобы сделать ее более точной.

> [!div class="nextstepaction"]
> [Тестирование и переобучение модели с помощью Пользовательской службы визуального распознавания](../../test-your-model.md)

* Что собой представляет Пользовательское визуальное распознавание
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/CustomVision/ImageClassification/CustomVisionQuickstart.js)
* [Справочная документация по пакету SDK (обучение)](/javascript/api/@azure/cognitiveservices-customvision-training/?view=azure-node-latest)
* [Справочная документация по пакету SDK (прогнозирование)](/javascript/api/@azure/cognitiveservices-customvision-prediction/?view=azure-node-latest)
