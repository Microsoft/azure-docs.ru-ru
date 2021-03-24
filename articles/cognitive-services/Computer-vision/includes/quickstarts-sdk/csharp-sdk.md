---
title: Краткое руководство. Клиентская библиотека службы "Компьютерное зрение" для .NET
description: В этом кратком руководстве описано, как приступить работы с клиентской библиотекой API "Компьютерное зрение" для .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 0509ba61e21fa38daf1747124000c8d1270cc4db
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103622276"
---
<a name="HOLTop"></a>

С помощью клиентской библиотеки службы "Компьютерное зрение" можно выполнять следующие операции:

* анализ изображений на наличие тегов, текстового описания, лиц, содержимого для взрослых и многого другого.
* считывание печатного и рукописного текста с помощью API службы чтения.

[Справочная документация](/dotnet/api/overview/azure/cognitiveservices/client/computervision) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.ComputerVision) | [Пакет (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/) | [Примеры](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [IDE Visual Studio](https://visualstudio.microsoft.com/vs/) или текущая версия [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="создайте ресурс Компьютерного зрения"  target="_blank">create a Computer Vision resource </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

#### <a name="visual-studio-ide"></a>[Интегрированная среда разработки Visual Studio](#tab/visual-studio)

С помощью Visual Studio создайте приложение .NET Core. 

### <a name="install-the-client-library"></a>Установка клиентской библиотеки 

После создания проекта установите клиентскую библиотеку, щелкнув правой кнопкой мыши решение проекта в **Обозревателе решений** и выбрав пункт **Управление пакетами NuGet**. В открывшемся диспетчере пакетов выберите **Просмотр**, установите флажок **Включить предварительные версии** и выполните поиск по запросу `Microsoft.Azure.CognitiveServices.Vision.ComputerVision`. Выберите версию `6.0.0-preview.1`, а затем **Установить**. 

#### <a name="cli"></a>[CLI](#tab/cli)

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `computer-vision-quickstart`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: *Program.cs*.

```console
dotnet new console -n computer-vision-quickstart
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

В каталоге приложения установите клиентскую библиотеку службы "Компьютерное зрение" для .NET с помощью следующей команды:

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

---

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs), где размещены примеры кода для этого краткого руководства.

В каталоге проекта откройте файл *Program.cs* в предпочитаемом редакторе или интегрированной среде разработки.

### <a name="find-the-subscription-key-and-endpoint"></a>Поиск ключа подписки и конечной точки

Перейдите на портал Azure. Если ресурс Компьютерного зрения, созданный с учетом **предварительных требований**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ подписки и конечную точку можно найти на странице **Ключи и конечная точка** ресурса в разделе **Управление ресурсами**. 

В классе приложения **Program** создайте переменные для ключа и конечной точки Компьютерного зрения. Вставьте ключ подписки и конечную точку в следующий код, где указано. Конечная точка Компьютерного зрения имеет такой формат: `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_using_and_vars)]

> [!IMPORTANT]
> Не забудьте удалить ключ подписки из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

В методе `Main` приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут созданы позже.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_client)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_analyzeinmain)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_extracttextinmain)]

> [!div class="nextstepaction"]
> [Мной настроен клиент](?success=set-up-client#object-model) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Csharp&Section=set-up-client)

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывают некоторые основные функции пакета SDK .NET для службы "Компьютерное зрение".

|Имя|Описание|
|---|---|
| [ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient) | Этот класс требуется для всех функций Компьютерного зрения. Вы создаете его экземпляр с информацией о подписке и используете его для выполнения большинства операций с образами.|
|[ComputerVisionClientExtensions](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclientextensions)| Этот класс содержит дополнительные методы для **ComputerVisionClient**.|
|[VisualFeatureTypes](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.models.visualfeaturetypes)| Это перечисление определяет различные типы анализа изображений, которые можно выполнить в стандартной операции анализа. Набор значений VisualFeatureTypes указывается в зависимости от потребностей. |

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки службы "Компьютерное зрение" для .NET.

* [аутентификация клиента](#authenticate-the-client);
* [Анализ изображения](#analyze-an-image)
* [Чтение печатного и рукописного текста](#read-printed-and-handwritten-text).

## <a name="authenticate-the-client"></a>Аутентификация клиента

В новом методе в классе **Program** создайте экземпляр клиента с использованием конечной точки и ключа подписки. Создайте объект **[ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.apikeyserviceclientcredentials)** с помощью ключа подписки и используйте его со своей конечной точкой, чтобы создать объект **[ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_auth)]

> [!div class="nextstepaction"]
> [Мной аутентифицирован клиент](?success=authenticate-client#analyze-an-image) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Csharp&Section=authenticate-client)

## <a name="analyze-an-image"></a>Анализ изображения

В следующем коде определен метод `AnalyzeImageUrl`, который использует клиентский объект для анализа удаленного образа и печати результатов. Метод возвращает текстовое описание, категоризацию, список тегов, обнаруженные лица, флажки содержимого для взрослых, основные цвета и тип изображения.

> [!TIP]
> Можно также проанализировать локальный образ. Изучите информацию о методах класса [ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient), например об **AnalyzeImageInStreamAsync**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs) для сценариев, включающих использование локальных изображений.

### <a name="set-up-test-image"></a>Настройка тестового изображения

В классе **Program** сохраните ссылку на URL-адрес изображения, который вы хотите проанализировать.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_analyze_url)]

### <a name="specify-visual-features"></a>Указание визуальных компонентов

Определите новый метод для анализа образов. Добавьте приведенный ниже код, в котором задаются визуальные компоненты, которые вы хотите извлечь при анализе. Полный список компонентов см. в описании перечисления **[VisualFeatureTypes](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.models.visualfeaturetypes)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_visualfeatures)]

Вставьте любой из приведенных ниже блоков кода в метод **AnalyzeImageUrl**, чтобы реализовать их функции. Не забудьте добавить закрывающую скобку в конце.

```csharp
}
```

### <a name="analyze"></a>Анализ

Метод **AnalyzeImageAsync** возвращает объект **ImageAnalysis**, содержащий всю извлеченную информацию.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_analyze_call)]

В следующих разделах подробно показано, как анализировать эти сведения.

### <a name="get-image-description"></a>Получение описания изображения

Следующий код получает список созданных заголовков для изображения. Дополнительные сведения см. в статье [Описание изображений на понятном для пользователя языке](../../concept-describing-images.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_describe)]

### <a name="get-image-category"></a>Получение категории изображения

Следующий код получает обнаруженную категорию изображения. Дополнительные сведения см. в статье [Categorize images by subject matter](../../concept-categorizing-images.md) (Классификация изображений по темам).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_categorize)]

### <a name="get-image-tags"></a>Получение тегов изображения

Следующий код получает набор обнаруженных тегов изображения. Дополнительные сведения см. в статье о [тегах содержимого](../../concept-tagging-images.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_tags)]

### <a name="detect-objects"></a>Обнаружение объектов

Следующий код обнаруживает общие объекты в образе и выводит их на консоль. Дополнительные сведения см. в статье [Обнаружение объектов](../../concept-object-detection.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_objects)]

### <a name="detect-brands"></a>Обнаружение торговых марок

Следующий код обнаруживает фирменные торговые марки и логотипы в образе и выводит их на консоль. Дополнительные сведения см. в статье [Обнаружение торговых марок](../../concept-brand-detection.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_brands)]

### <a name="detect-faces"></a>Распознавание лиц

Следующий код возвращает обнаруженные лица на изображении с их координатами прямоугольника и атрибутами выбора лиц. Дополнительные сведения см. в статье [Определение лиц с помощью компьютерного зрения](../../concept-detecting-faces.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_faces)]

### <a name="detect-adult-racy-or-gory-content"></a>Обнаружение содержимого для взрослых, непристойного или жуткого содержимого

Следующий код выводит сведения об обнаружении содержимого для взрослых на изображении. Дополнительные сведения см. в статье [Detect adult content](../../concept-detecting-adult-content.md) (Обнаружение содержимого для взрослых).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_adult)]

### <a name="get-image-color-scheme"></a>Получение цветовой схемы изображения

Следующий код выводит обнаруженные атрибуты цвета на изображении, например доминирующие и акцентные цвета. Дополнительные сведения см. в статье [Обнаружение цветовых схем на изображениях](../../concept-detecting-color-schemes.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_color)]

### <a name="get-domain-specific-content"></a>Получение содержимого, связанного с определенной предметной областью

API "Компьютерное зрение" может использовать специализированные модели для дальнейшего анализа изображений. Дополнительные сведения см. в статье [Обнаружение содержимого, связанного с определенными предметными областями](../../concept-detecting-domain-content.md). 

Следующий код анализирует данные об обнаруженных известных лицах на изображении.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_celebs)]

Следующий код анализирует данные об обнаруженных достопримечательностях на изображении.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_landmarks)]

### <a name="get-the-image-type"></a>Получение сведений о типе изображения

Следующий код выводит информацию о типе изображения, независимо от того, является ли это картинкой или рисунком.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_type)]

> [!div class="nextstepaction"]
> [Мной проанализировано изображение](?success=analyze-image#read-printed-and-handwritten-text) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Csharp&Section=analyze-image)

## <a name="read-printed-and-handwritten-text"></a>Чтение печатного и рукописного текста

Компьютерное зрение может считывать видимый текст в образе и преобразовывать его в поток символов. Дополнительные сведения о распознавании текста см. в основном документе [Оптическое распознавание текста (OCR)](../../concept-recognizing-text.md#read-api). Здесь в коде используется [пакет SDK Компьютерного зрения для чтения последней версии 3.0](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision/6.0.0-preview.1) и определяется метод `BatchReadFileUrl`, который использует клиентский объект для обнаружения и извлечения текста на изображении.

> [!TIP]
> Кроме того, можно извлечь текст из локального образа. Изучите информацию о методах класса [ComputerVisionClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.computervision.computervisionclient), например о **ReadInStreamAsync**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs) для сценариев, включающих использование локальных изображений.

### <a name="set-up-test-image"></a>Настройка тестового изображения

В классе **Program** сохраните ссылку на URL-адрес изображения, из которого вы хотите извлечь текст. Этот фрагмент кода включает в себя примеры изображений печатного и рукописного текста.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readtext_url)]

### <a name="call-the-read-api"></a>Вызов API чтения

Определите новый метод для чтения текста. Добавьте приведенный ниже код, который вызывает метод **ReadAsync** для заданного изображения. Он возвращает идентификатор операции и запускает асинхронный процесс чтения содержимого образа.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readfileurl_1)]

### <a name="get-read-results"></a>Получение результатов чтения

Затем получите идентификатор операции, возвращенный из вызова **ReadAsync**, и получите от службы результаты операции по этому идентификатору. Следующий код проверяет операцию, пока не будут возвращены результаты. После этого извлеченные текстовые данные выводятся на консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readfileurl_2)]

### <a name="display-read-results"></a>Отображение результатов чтения

Добавьте следующий код для анализа и отображения полученных текстовых данных и завершения определения метода.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ComputerVision/ComputerVisionQuickstart.cs?name=snippet_readfileurl_3)]

## <a name="run-the-application"></a>Выполнение приложения

#### <a name="visual-studio-ide"></a>[Интегрированная среда разработки Visual Studio](#tab/visual-studio)

Запустите приложение, нажав кнопку **Отладка** в верхней части окна интегрированной среды разработки.

#### <a name="cli"></a>[CLI](#tab/cli)

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```dotnet
dotnet run
```

---

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Справочник по API службы "Компьютерное зрение" (.NET)](/dotnet/api/overview/azure/cognitiveservices/client/computervision)

* [Что собой представляет Компьютерное зрение](../../overview.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ComputerVision/ComputerVisionQuickstart.cs).
