---
title: Краткое руководство по использованию клиентской библиотеки Content Moderator для .NET
titleSuffix: Azure Cognitive Services
description: В этом кратком руководстве показано, как начать работу с клиентской библиотекой Content Moderator для .NET. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 09/15/2020
ms.author: pafarley
ms.custom: devx-track-dotnet, cog-serv-seo-aug-2020
ms.openlocfilehash: bdf49f16b003c25da9fe8972a3fb7e3837d62819
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105498"
---
Начало работы с клиентской библиотекой Azure Content Moderator для .NET. Выполните приведенные здесь действия, чтобы установить пакет NuGet и протестировать пример кода для выполнения базовых задач. 

Content Moderator — это служба ИИ, позволяющая управлять содержимым, которое может носить оскорбительный характер или представлять опасность, либо нежелательным содержимым иного рода. Используйте службу модерации содержимого на основе ИИ, чтобы проверить текст, изображения и видео, а также автоматически применить флаги содержимого. Затем интегрируйте свое приложение со средством проверки — средой модерации для команды рецензентов для работы в подключенном режиме. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.

Клиентскую библиотеку Content Moderator для .NET можно использовать для такой задачи:

* Модерация текста
* Модерация изображений
* Создание проверки

[Справочная документация](/dotnet/api/overview/azure/cognitiveservices/client/contentmoderator) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.ContentModerator) | [Пакет (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) | [Примеры](../../samples-dotnet.md)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [IDE Visual Studio](https://visualstudio.microsoft.com/vs/) или текущая версия [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Получив подписку Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Создание ресурса Content Moderator"  target="_blank">создайте ресурс Content Moderator </a> на портале Azure, чтобы получить ключ и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Для подключения приложения к Content Moderator потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

#### <a name="visual-studio-ide"></a>[Интегрированная среда разработки Visual Studio](#tab/visual-studio)

С помощью Visual Studio создайте приложение .NET Core. 

### <a name="install-the-client-library"></a>Установка клиентской библиотеки 

После создания проекта установите клиентскую библиотеку, щелкнув правой кнопкой мыши решение проекта в **Обозревателе решений** и выбрав пункт **Управление пакетами NuGet**. В открывшемся диспетчере пакетов выберите **Просмотр**, установите флажок **Включить предварительные версии** и выполните поиск по запросу `Microsoft.Azure.CognitiveServices.ContentModerator`. Выберите версию `2.0.0`, а затем **Установить**. 

#### <a name="cli"></a>[CLI](#tab/cli)

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `content-moderator-quickstart`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: *Program.cs*.

```console
dotnet new console -n content-moderator-quickstart
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

В каталоге приложения установите клиентскую библиотеку Content Moderator для .NET с помощью следующей команды:

```console
dotnet add package Microsoft.Azure.CognitiveServices.ContentModerator --version 2.0.0
```

---

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/ContentModerator/Program.cs), где размещены примеры кода для этого краткого руководства.

В каталоге проекта откройте файл *Program.cs* в предпочитаемом редакторе или интегрированной среде разработки. Добавьте следующие операторы `using` :

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_using)]

В классе **Program** создайте переменные для ключа и конечной точки вашего ресурса.

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Content Moderator, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_creds)]


В методе `main()` приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут созданы позже.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_client)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_textmod_call)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_call)]

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_call)]


## <a name="object-model"></a>Объектная модель

Следующие классы обрабатывают некоторые основные функции клиентской библиотеки Content Moderator для .NET.

|Имя|Описание|
|---|---|
|[ContentModeratorClient](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.contentmoderatorclient)|Этот класс требуется для всех функций Content Moderator. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов.|
|[ImageModeration](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation)|Этот класс предоставляет функции для анализа изображений на наличие содержимого для взрослых, личной информации или лиц людей.|
|[TextModeration](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.textmoderation)|Этот класс предоставляет функции для анализа текста с целью определения языка, наличия ненормативной лексики, ошибок и личной информации.|
|[Проверки](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.reviews)|Этот класс предоставляет функциональные возможности API-интерфейсов проверки, в том числе методы создания заданий, настраиваемых рабочих процессов и пользовательских проверок.|

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Content Moderator для .NET:

* [аутентификация клиента](#authenticate-the-client);
* [модерация текста](#moderate-text);
* [модерация изображений](#moderate-images);
* [создание проверки](#create-a-review).

## <a name="authenticate-the-client"></a>Аутентификация клиента

В новом методе создайте клиентские объекты с использованием конечной точки и ключа.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_auth)]

## <a name="moderate-text"></a>Модерация текста

В следующем коде используется клиент Content Moderator для анализа текста и вывода результатов в консоль. В корне класса **Program** определите входные и выходные файлы:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_text_vars)]

Затем в корне проекта добавьте файл *TextFile.txt*. Добавьте в этот файл свой текст или используйте текст из этого примера:

```
Is this a grabage email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
Crap is the profanity here. Is this information PII? phone 4255550111
```


Затем в любом месте в классе **Program** определите метод модерации текста:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_textmod)]

## <a name="moderate-images"></a>Модерация изображений

В следующем коде используется клиент Content Moderator и объект [ImageModeration](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation) для анализа удаленных изображений на наличие содержимого для взрослых и содержимого непристойного характера.

> [!NOTE]
> Можно также проанализировать содержимое локального изображения. Методы и операции, работающие с локальными изображениями, см. в [справочной документации](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.imagemoderation.evaluatefileinputwithhttpmessagesasync#Microsoft_Azure_CognitiveServices_ContentModerator_ImageModeration_EvaluateFileInputWithHttpMessagesAsync_System_IO_Stream_System_Nullable_System_Boolean__System_Collections_Generic_Dictionary_System_String_System_Collections_Generic_List_System_String___System_Threading_CancellationToken_).

### <a name="get-sample-images"></a>Получить образцы изображений

В корне класса **Program** определите входные и выходные файлы:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_image_vars)]

Затем в корне проекта создайте входной файл *ImageFiles.txt*. В этом файле вы добавите URL-адреса изображений для анализа (по одному URL-адресу в каждой строке). Вы можете использовать следующий пример изображений:

```
https://moderatorsampleimages.blob.core.windows.net/samples/sample2.jpg
https://moderatorsampleimages.blob.core.windows.net/samples/sample5.png
```

### <a name="define-helper-class"></a>Определение вспомогательного класса

Добавьте следующее определение класса в класс **Program**. Этот внутренний класс обработает результаты модерации изображений.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_dataclass)]

### <a name="define-the-image-moderation-method"></a>Определение метода модерации изображений

Следующий метод выполняет перебор изображений по их URL-адресам в текстовом файле, создает экземпляр **EvaluationData** и анализирует изображение на наличие содержимого для взрослых и непристойного содержимого, текста и человеческих лиц. Затем он добавляет в список окончательный экземпляр **EvaluationData** и записывает в консоль полный список возвращенных данных.

#### <a name="iterate-through-images"></a>Перебор изображений

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_iterate)]

#### <a name="analyze-content"></a>Анализ содержимого

Дополнительные сведения об атрибутах изображения, доступных в Content Moderator, см. в разделе [Основные принципы модерации изображений](../../image-moderation-api.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_analyze)]

#### <a name="write-moderation-results-to-file"></a>Запись результатов модерации в файл

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_imagemod_save)]

## <a name="create-a-review"></a>Создание проверки

С помощью клиентской библиотеки Content Moderator для .NET можно передать содержимое в [средство проверки](https://contentmoderator.cognitive.microsoft.com) для его проверки человеком. Чтобы узнать больше о средстве проверки, ознакомьтесь с разделом [Средство проверки Content Moderator](../../review-tool-user-guide/human-in-the-loop.md).

В этом разделе метод использует класс [Reviews](/dotnet/api/microsoft.azure.cognitiveservices.contentmoderator.reviews) для создания проверки, получения ее идентификатора и уточнения сведений о ней после выполнения действий пользователем-модератором на веб-портале средства проверки. Все эти сведения записываются в выходной текстовый файл. 

### <a name="get-sample-images"></a>Получить образцы изображений

Объявите следующий массив в корне класса **Program**. Эта переменная ссылается на образец изображения, используемый для создания проверки.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_urls)]

### <a name="get-review-credentials"></a>Получение учетных данных проверки

Войдите в [средство проверки](https://contentmoderator.cognitive.microsoft.com), чтобы узнать имя своей группы проверки. Затем назначьте его соответствующей переменной в классе **Program**. Вы можете также настроить конечную точку обратного вызова для получения сведений о действиях проверки.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_vars)]

### <a name="define-helper-class"></a>Определение вспомогательного класса

Добавьте следующее определение класса в свой класс **Program**. Этот класс будет использоваться для представления отдельного экземпляра проверки, отправляемого в средство проверки.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_review_item)]

### <a name="define-helper-method"></a>Определение вспомогательного метода

Добавьте следующий метод в класс **Program**. Этот метод будет записывать результаты запросов проверки в выходной текстовый файл.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_writeline)]

### <a name="define-the-review-creation-method"></a>Определение метода создания проверки

Теперь можно приступать к определению метода, который будет создавать проверку и выполнять ее запрос. Добавьте новый метод **CreateReviews** и определите следующие локальные переменные.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_fields)]

#### <a name="post-reviews-to-the-review-tool"></a>Действия после проверки в средстве проверки

Затем добавьте следующий код для перебора заданных образцов изображений, добавления метаданных и последующей их передачи в средство проверки одним пакетом. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_create)]

Объект, возвращенный из вызова API, будет содержать уникальные значения идентификатора для каждого переданного изображения. Следующий код анализирует эти идентификаторы, а затем использует их, чтобы запросить Content Moderator о состоянии каждого изображения в пакете.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_ids)]

### <a name="get-review-details"></a>Получение сведений о проверке

Следующий код вынуждает программу ожидать ввода данных пользователем. Когда вы приступите к этому шагу во время выполнения, можно перейти к [средству проверки](https://contentmoderator.cognitive.microsoft.com), убедиться, что образец изображения был отправлен, и начать взаимодействовать с ним. Сведения о том, как взаимодействовать с проверкой, см. в статье [Руководство по проверкам](../../review-tool-user-guide/review-moderated-images.md). По завершении можно нажать любую клавишу, чтобы продолжить выполнение программы и получить результаты процесса проверки.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/ContentModerator/Program.cs?name=snippet_createreview_results)]

Если в этом сценарии использовалась конечная точка обратного вызова, в ней должно быть получено событие в таком формате:

```console
{'callback_endpoint': 'https://requestb.in/qmsakwqm',
 'content': '',
 'content_id': '3ebe16cb-31ed-4292-8b71-1dfe9b0e821f',
 'created_by': 'cspythonsdk',
 'metadata': [{'key': 'sc', 'value': 'True'}],
 'review_id': '201901i14682e2afe624fee95ebb248643139e7',
 'reviewer_result_tags': [{'key': 'a', 'value': 'True'},
                          {'key': 'r', 'value': 'True'}],
 'status': 'Complete',
 'sub_team': 'public',
 'type': 'Image'}
```

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

Из этого краткого руководстве вы узнали, как модерировать содержимое с помощью библиотеки .NET для Content Moderator. Теперь узнайте больше о модерации изображений или другого мультимедиа, прочитав концептуальное руководство.

> [!div class="nextstepaction"]
> [Принципы модерации изображений](../../image-moderation-api.md)