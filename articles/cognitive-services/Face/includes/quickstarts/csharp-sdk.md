---
title: Краткое руководство по использованию клиентской библиотеки API "Распознавание лиц" (.NET)
description: Клиентская библиотека API "Распознавание лиц" для .NET позволяет обнаруживать лица, находить похожие лица (поиск лиц по изображению), идентифицировать лица (поиск с распознаванием лиц) и переносить ваши данные о лицах.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 10/26/2020
ms.author: pafarley
ms.openlocfilehash: ab99d574588989c84783e532fcf801dcaffdd54d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105958401"
---
В этом руководстве описано, как приступить к работе с клиентской библиотекой API "Распознавание лиц" для .NET. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач. В службе "Распознавание лиц" доступны передовые алгоритмы обнаружения и распознавания лиц на изображениях.

Клиентская библиотека Распознавания лиц для .NET позволяет выполнять такие задачи:

* [Определение лиц на изображении](#detect-faces-in-an-image)
* [поиск похожих лиц](#find-similar-faces);
* [Создание PersonGroup](#create-a-persongroup)
* [опознание лица](#identify-a-face);

[Справочная документация](/dotnet/api/overview/azure/cognitiveservices/client/faceapi) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Vision.Face) | [Пакет (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.6.0-preview.1) | [Примеры](/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>Предварительные требования


* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [IDE Visual Studio](https://visualstudio.microsoft.com/vs/) или текущая версия [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Получив подписку Azure, перейдите к <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Создание ресурса Распознавания лиц"  target="_blank">созданию ресурса Распознавания лиц</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Распознавания лиц потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

#### <a name="visual-studio-ide"></a>[Интегрированная среда разработки Visual Studio](#tab/visual-studio)

С помощью Visual Studio создайте приложение .NET Core. 

### <a name="install-the-client-library"></a>Установка клиентской библиотеки 

После создания проекта установите клиентскую библиотеку, щелкнув правой кнопкой мыши решение проекта в **Обозревателе решений** и выбрав пункт **Управление пакетами NuGet**. В открывшемся диспетчере пакетов выберите **Просмотр**, установите флажок **Включить предварительные версии** и выполните поиск по запросу `Microsoft.Azure.CognitiveServices.Vision.Face`. Выберите версию `2.6.0-preview.1`, а затем **Установить**. 

#### <a name="cli"></a>[CLI](#tab/cli)

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `face-quickstart`. Эта команда создает простой проект "Hello World" на языке C# с одним файлом исходного кода: *program.cs*. 

```console
dotnet new console -n face-quickstart
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

В каталоге приложения установите клиентскую библиотеку Распознавания лиц для .NET с помощью следующей команды:

```console
dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
```

---

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs), где размещены примеры кода для этого краткого руководства.


В каталоге проекта откройте файл *program.cs* и с помощью `using` добавьте следующие директивы:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_using)]

В классе приложения **Program** создайте переменные для ключа и конечной точки вашего ресурса.


> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс "Распознавание лиц", созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_creds)]

В методе **Main** приложения добавьте вызовы методов, используемых в этом кратком руководстве. Вы реализуете их позже.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_maincalls)]

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы воплощают некоторые основные функции клиентской библиотеки API "Распознавание лиц" для .NET.

|Имя|Описание|
|---|---|
|[FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient) | Этот класс реализует авторизацию для использования Распознавания лиц и требуется для реализации всех ее функций. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов. |
|[FaceOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperations)|Этот класс обрабатывает основные задачи по обнаружению и распознаванию лиц. |
|[DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface)|Этот класс представляет все данные об отдельном лице, обнаруженном на изображении. Его можно использовать для получения подробных сведений о лице.|
|[FaceListOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations)|Этот класс управляет хранимыми в облаке конструкциями **FaceList**, которые включают систематизированную коллекцию лиц. |
|[PersonGroupPersonExtensions](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongrouppersonextensions)| Этот класс управляет хранимыми в облаке конструкциями **Person**, в которых хранится коллекция лиц одного человека.|
|[PersonGroupOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations)| Этот класс управляет хранимыми в облаке конструкциями **PersonGroup**, в которых хранится систематизированная коллекция объектов **Person**. |

## <a name="code-examples"></a>Примеры кода

В приведенных ниже фрагментах кода показано, как выполнять следующие задачи с клиентской библиотекой Распознавания лиц для .NET:

* [аутентификация клиента](#authenticate-the-client);
* [Определение лиц на изображении](#detect-faces-in-an-image)
* [поиск похожих лиц](#find-similar-faces);
* [Создание PersonGroup](#create-a-persongroup)
* [опознание лица](#identify-a-face);

## <a name="authenticate-the-client"></a>Аутентификация клиента

В новом методе создайте экземпляр клиента с использованием конечной точки и ключа. Создайте объект **[ApiKeyServiceClientCredentials](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.apikeyserviceclientcredentials)** с помощью ключа и используйте его со своей конечной точкой, чтобы создать объект **[FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient)** .

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_auth)]

### <a name="declare-helper-fields"></a>Объявление вспомогательных полей

Следующие поля требуются для некоторых операций Распознавания лиц, которые будут добавлены позже. В корне класса **Program** определите следующую строку URL-адреса. Этот URL-адрес указывает на папку с примерами изображений.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_image_url)]

В методе **Main** определите строки, указывающие на разные типы моделей распознавания. Позже вы сможете указать, какую модель распознавания нужно использовать для обнаружения лиц. Сведения о доступных вариантах см. в разделе [Указание модели распознавания](../../Face-API-How-to-Topics/specify-recognition-model.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect_models)]

## <a name="detect-faces-in-an-image"></a>Определение лиц на изображении

### <a name="get-detected-face-objects"></a>Получение обнаруженных объектов лиц

Создайте новый метод для обнаружения лиц. Метод `DetectFaceExtract` обрабатывает три изображения, размещенные по указанному URL-адресу, и создает список объектов **[DetectedFace](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface)** в памяти программы. В списке значений **[FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype)** указано, какие признаки следует извлечь. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect)]

> [!TIP]
> Обнаружение лиц можно также выполнить, используя локальное изображение. Изучите информацию о методах класса [IFaceOperations](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.ifaceoperations), например о **DetectWithStreamAsync**.

### <a name="display-detected-face-data"></a>Отображение обнаруженных данных о лицах

Оставшаяся часть метода `DetectFaceExtract` анализирует и выводит данные атрибутов для каждого обнаруженного лица. Каждый атрибут должен быть указан отдельно в исходном вызове API обнаружения лиц (в списке **[FaceAttributeType](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype)** ). Следующий код обрабатывает каждый атрибут, но вам, скорее всего, потребуются лишь один-два из них.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_detect_parse)]

## <a name="find-similar-faces"></a>Поиск похожих лиц

Следующий код принимает одно обнаруженное лицо (источник) и выполняет поиск совпадений (поиск лиц по изображению) в коллекции других лиц (цель). При обнаружении совпадения в консоль выводится идентификатор лица, с которым найдено совпадение.

### <a name="detect-faces-for-comparison"></a>Обнаружение лиц для сравнения

Во-первых, определите второй метод для обнаружения лиц. Прежде чем сравнивать лица, их нужно обнаружить на изображениях, и этот метод обнаружения оптимизирован для операций сравнения. Он не извлекает подробные атрибуты лица, как в предыдущем разделе, и использует другую модель распознавания.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_face_detect_recognize)]

### <a name="find-matches"></a>Поиск совпадений

Следующий метод обнаруживает лица в наборе целевых изображений и в одном исходном изображении. Затем он сравнивает их и находит все целевые изображения, аналогичные исходному изображению.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_find_similar)]

### <a name="print-matches"></a>Вывод совпадений

Следующий код выводит в консоль сведения о совпадениях.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_find_similar_print)]

## <a name="identify-a-face"></a>опознание лиц;

Операция идентификации (Identify) принимает изображение человека или нескольких людей и пытается опознать каждое лицо на этом изображении (поиск с распознаванием лиц). Он сравнивает каждое обнаруженное лицо с **PersonGroup**, которая является базой данных объектов **Person** с известными характеристиками лиц. Чтобы выполнить операцию Identify, сначала необходимо создать и обучить **PersonGroup**.

### <a name="create-a-persongroup"></a>Создание PersonGroup

Следующий код создает группу **PersonGroup** с шестью разными объектами **Person**. Он сопоставляет каждый объект **Person** с набором тестовых изображений, а затем обучается распознавать каждого человека по характеристикам их лиц. Объекты **Person** и **PersonGroup** используются в операциях Verify, Identify и Group.

Объявите в корне класса строковую переменную, которая будет представлять идентификатор создаваемой группы **PersonGroup**.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_declare)]

Добавьте указанный ниже код в новый метод. Этот метод будет выполнять операцию Identify. Первый блок кода связывает имена людей с примерами их изображений.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_files)]

Обратите внимание, что этот код определяет переменную `sourceImageFileName`. Эта переменная соответствует исходному изображению, которое содержит пользователей для распознавания.

Теперь добавьте следующий код, чтобы создать в словаре объект **Person** для каждого человека и добавить данные о лицах из соответствующих изображений. Каждый объект **Person** связывается с одним объектом **PersonGroup** через уникальную строку идентификатора. Не забудьте передать в этот метод переменные `client`, `url` и `RECOGNITION_MODEL1`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_create)]

> [!TIP]
> Объект **PersonGroup** можно также создать, используя локальные изображения. Изучите информацию о методах класса [IPersonGroupPerson](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.ipersongroupperson), например о **AddFaceFromStreamAsync**.

### <a name="train-the-persongroup"></a>Обучение PersonGroup

После извлечения данных о лицах и сортировки их по разным объектам **Person** вам нужно обучить объект **PersonGroup** для выявления визуальных характеристик, связанных с каждым из объектов **Person** в этой группе. Следующий код вызывает асинхронный метод **train** и опрашивает результаты, а затем выводит сведения о состоянии в консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_train)]

> [!TIP]
> API службы "Распознавание лиц" выполняется на наборе предварительно созданных моделей, которые по своей природе являются статическими (производительность модели не ухудшается и не улучшается при выполнении службы). Результаты, предоставляемые моделью, могут измениться, если корпорация Майкрософт обновит внутреннюю структуру модели без перехода на полностью новую версию модели. Чтобы воспользоваться преимуществами новой версии модели, вы можете повторно обучить **PersonGroup**, указав более новую модель в качестве параметра с такими же изображениями для регистрации.

Эта группа **Person** и связанные с ней объекты **Person** теперь готовы к использованию в операциях проверки, определения или группирования.

### <a name="identify-faces"></a>Идентификация лиц

Следующий код принимает исходное изображение и создает список всех лиц, обнаруженных в этом изображении. Именно эти лица будут сопоставляться с объектами в группе **PersonGroup**.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_identify_sources)]

Следующий фрагмент кода вызывает операцию **IdentifyAsync** и выводит результаты в консоль. Служба здесь пытается сопоставить каждое лицо с исходного изображения с определенным объектом **Person** в указанной группе **PersonGroup**. Это приведет к закрытию метода Identify.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_identify)]

## <a name="run-the-application"></a>Запуск приложения

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

Если при изучении этого краткого руководства вы создали объект **PersonGroup** и хотите его удалить, выполните в программе следующий код:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_persongroup_delete)]

Определите метод удаления со следующим кодом:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/Face/FaceQuickstart.cs?name=snippet_deletepersongroup)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководство вы узнали, как с помощью клиентской библиотеки Распознавания лиц для .NET выполнять базовые задачи по распознаванию лиц. Далее ознакомьтесь со справочной документацией, чтобы узнать больше о библиотеке.

> [!div class="nextstepaction"]
> [Справочник по API распознавания лиц (.NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi)

* [Что представляет собой служба "Распознавание лиц"?](../../overview.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/dotnet/Face/FaceQuickstart.cs).
