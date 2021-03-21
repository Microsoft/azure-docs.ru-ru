---
title: Перенос данных о лицах между подписками — лицом
titleSuffix: Azure Cognitive Services
description: В этом руководство показано, как перенести сохраненные данные о лицах из одной подписки в другую.
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: nitinme
ms.custom: devx-track-csharp
ms.openlocfilehash: c8d3c5b10c670e7aa4f1fd00f47ef47e772416cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101706866"
---
# <a name="migrate-your-face-data-to-a-different-face-subscription"></a>Перенос данных распознавания лиц в другую подписку API распознавания лиц

В этом руководство показано, как переместить данные о лицах, например сохраненный объект Персонграуп с лицами, в другую подписку Azure Cognitive Services Face. Для перемещения данных используется функция моментального снимка. Это позволит избежать многократного создания и обучения объекта Персонграуп или Фацелист при перемещении или расширении операций. Например, предположим, что вы создали объект Персонграуп с бесплатной подпиской и теперь хотите перенести ее в платную подписку. Или может потребоваться синхронизация данных о лицах между подписками в разных регионах для крупных корпоративных операций.

Эта же стратегия переноса применяется к объектам LargePersonGroup и LargeFaceList. Если вы не знакомы с понятиями, изложенными в этом пошаговом окне, см. их определения в разделе " [Основные понятия распознавания лиц](../concepts/face-recognition.md) ". В этом руководство используется клиентская библиотека .NET для языка C#.

## <a name="prerequisites"></a>Предварительные условия

Вам потребуются следующие элементы:

- Ключи подписки двух лиц, одна с существующими данными, а другая — для миграции. Чтобы подписаться на службу лиц и получить ключ, следуйте инструкциям в разделе [Создание учетной записи Cognitive Services](../../cognitive-services-apis-create-account.md).
- Строка идентификатора подписки лица, соответствующая целевой подписке. Чтобы найти его, выберите **Обзор** в портал Azure. 
- Любой выпуск [Visual Studio 2015 или 2017](https://www.visualstudio.com/downloads/).

## <a name="create-the-visual-studio-project"></a>Создание проекта Visual Studio

В этом руководством для выполнения переноса данных о лицах используется простое консольное приложение. Полную реализацию см. в [образце моментальных снимков для лиц](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) на GitHub.

1. В Visual Studio создайте новое консольное приложение платформа .NET Framework проекте. Назовите его **фацеаписнапшотсампле**.
1. Получите необходимые пакеты NuGet. Щелкните правой кнопкой мыши проект в обозреватель решений и выберите **Управление пакетами NuGet**. Перейдите на вкладку **Обзор** и выберите **включить предварительные выпуски**. Найдите и установите следующий пакет:
    - [Microsoft.Azure.CognitiveServices.Vision.Face 2.3.0-preview](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.2.0-preview).

## <a name="create-face-clients"></a>Создание клиентов для распознавания лиц

В методе **Main** в файле *Program.cs* создайте два экземпляра [FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient) для исходной и целевой подписок. В этом примере используется подписка на лицо в регионе Восточная Азия в качестве источника и подписке "Западная часть США" в качестве цели. В этом примере показано, как перенести данные из одного региона Azure в другой. 

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

```csharp
var FaceClientEastAsia = new FaceClient(new ApiKeyServiceClientCredentials("<East Asia Subscription Key>"))
    {
        Endpoint = "https://southeastasia.api.cognitive.microsoft.com/>"
    };

var FaceClientWestUS = new FaceClient(new ApiKeyServiceClientCredentials("<West US Subscription Key>"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com/"
    };
```

Укажите значения ключей подписки и URL-адреса конечных точек для исходной и целевой подписок.


## <a name="prepare-a-persongroup-for-migration"></a>Подготовка PersonGroup для переноса

Вам потребуется идентификатор объекта PersonGroup в вашей исходной подписке, чтобы перенести его в целевую подписку. Используйте метод [персонграупоператионсекстенсионс. ListAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperationsextensions.listasync) для получения списка объектов персонграуп. Затем получите свойство [персонграуп. персонграупид](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.persongroup.persongroupid#Microsoft_Azure_CognitiveServices_Vision_Face_Models_PersonGroup_PersonGroupId) . Этот процесс выглядит по-разному в зависимости от того, какие объекты Персонграуп у вас есть. В этом руководством идентификатор источника Персонграуп хранится в `personGroupId` .

> [!NOTE]
> [Пример кода](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) создает и обучает новый персонграуп для миграции. В большинстве случаев у вас уже должен быть Персонграуп для использования.

## <a name="take-a-snapshot-of-a-persongroup"></a>Создание моментального снимка Персонграуп

Моментальный снимок является временным удаленным хранилищем для определенных типов данных лиц. Он функционирует как своего рода буфер обмена для копирования данных из одной подписки в другую. Сначала необходимо создать моментальный снимок данных в исходной подписке. Затем вы примените его к новому объекту данных в целевой подписке.

Используйте экземпляр Фацеклиент исходной подписки, чтобы создать моментальный снимок Персонграуп. Используйте [такеасинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperationsextensions.takeasync) с идентификатором ПЕРСОНГРАУП и идентификатором целевой подписки. Если у вас есть несколько целевых подписок, добавьте их в качестве записей массива в третий параметр.

```csharp
var takeSnapshotResult = await FaceClientEastAsia.Snapshot.TakeAsync(
    SnapshotObjectType.PersonGroup,
    personGroupId,
    new[] { "<Azure West US Subscription ID>" /* Put other IDs here, if multiple target subscriptions wanted */ });
```

> [!NOTE]
> Процесс создания и применения моментальных снимков не нарушает обычные вызовы исходного или целевого Персонграупс или Фацелистс. Не делайте одновременных вызовов, изменяющих исходный объект, например [вызовы управления фацелист](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations) или вызов [персонграуп](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations) Train. Операция моментального снимка может выполняться до или после этих операций или может столкнуться с ошибками.

## <a name="retrieve-the-snapshot-id"></a>Получение идентификатора моментального снимка

Метод, используемый для создания моментальных снимков, является асинхронным, поэтому необходимо дождаться его завершения. Операции с моментальными снимками не могут быть отменены. В этом коде `WaitForOperation` метод отслеживает асинхронный вызов. Он проверяет состояние каждые 100 мс. После завершения операции получите идентификатор операции, анализируя `OperationLocation` поле. 

```csharp
var takeOperationId = Guid.Parse(takeSnapshotResult.OperationLocation.Split('/')[2]);
var operationStatus = await WaitForOperation(FaceClientEastAsia, takeOperationId);
```

Типичное `OperationLocation` значение выглядит следующим образом:

```csharp
"/operations/a63a3bdd-a1db-4d05-87b8-dbad6850062a"
```

Вспомогательный метод `WaitForOperation` находится здесь:

```csharp
/// <summary>
/// Waits for the take/apply operation to complete and returns the final operation status.
/// </summary>
/// <returns>The final operation status.</returns>
private static async Task<OperationStatus> WaitForOperation(IFaceClient client, Guid operationId)
{
    OperationStatus operationStatus = null;
    do
    {
        if (operationStatus != null)
        {
            Thread.Sleep(TimeSpan.FromMilliseconds(100));
        }

        // Get the status of the operation.
        operationStatus = await client.Snapshot.GetOperationStatusAsync(operationId);

        Console.WriteLine($"Operation Status: {operationStatus.Status}");
    }
    while (operationStatus.Status != OperationStatusType.Succeeded
            && operationStatus.Status != OperationStatusType.Failed);

    return operationStatus;
}
```

После отображения состояния операции `Succeeded` Получите идентификатор моментального снимка, анализируя `ResourceLocation` поле возвращенного экземпляра значение operationstatus.

```csharp
var snapshotId = Guid.Parse(operationStatus.ResourceLocation.Split('/')[2]);
```

Типичное `resourceLocation` значение выглядит следующим образом:

```csharp
"/snapshots/e58b3f08-1e8b-4165-81df-aa9858f233dc"
```

## <a name="apply-a-snapshot-to-a-target-subscription"></a>Применение моментального снимка к целевой подписке

Затем создайте новый Персонграуп в целевой подписке, используя сгенерированный случайным образом идентификатор. Затем используйте экземпляр Фацеклиент целевой подписки для применения моментального снимка к этому Персонграуп. Передайте идентификатор моментального снимка и новый идентификатор Персонграуп.

```csharp
var newPersonGroupId = Guid.NewGuid().ToString();
var applySnapshotResult = await FaceClientWestUS.Snapshot.ApplyAsync(snapshotId, newPersonGroupId);
```


> [!NOTE]
> Объект моментального снимка действителен только в 48 часа. Создавайте моментальные снимки только в том случае, если планируется использовать его для переноса данных вскоре после.

Запрос на применение моментального снимка возвращает другой идентификатор операции. Чтобы получить этот идентификатор, проанализируйте `OperationLocation` поле возвращенного экземпляра апплиснапшотресулт. 

```csharp
var applyOperationId = Guid.Parse(applySnapshotResult.OperationLocation.Split('/')[2]);
```

Процесс приложения моментального снимка также является асинхронным, поэтому снова используйте `WaitForOperation` для ожидания его завершения.

```csharp
operationStatus = await WaitForOperation(FaceClientWestUS, applyOperationId);
```

## <a name="test-the-data-migration"></a>Проверка переноса данных

После применения моментального снимка новый Персонграуп в целевой подписке заполняется данными исходного лица. По умолчанию результаты обучения также копируются. Новая Персонграуп готова к идентификационным звонкам лиц без необходимости повторного обучения.

Чтобы протестировать перенос данных, выполните следующие операции и сравните результаты, которые они печатают на консоли:

```csharp
await DisplayPersonGroup(FaceClientEastAsia, personGroupId);
await IdentifyInPersonGroup(FaceClientEastAsia, personGroupId);

await DisplayPersonGroup(FaceClientWestUS, newPersonGroupId);
// No need to retrain the person group before identification,
// training results are copied by snapshot as well.
await IdentifyInPersonGroup(FaceClientWestUS, newPersonGroupId);
```

Используйте следующие вспомогательные методы:

```csharp
private static async Task DisplayPersonGroup(IFaceClient client, string personGroupId)
{
    var personGroup = await client.PersonGroup.GetAsync(personGroupId);
    Console.WriteLine("Person Group:");
    Console.WriteLine(JsonConvert.SerializeObject(personGroup));

    // List persons.
    var persons = await client.PersonGroupPerson.ListAsync(personGroupId);

    foreach (var person in persons)
    {
        Console.WriteLine(JsonConvert.SerializeObject(person));
    }

    Console.WriteLine();
}
```

```csharp
private static async Task IdentifyInPersonGroup(IFaceClient client, string personGroupId)
{
    using (var fileStream = new FileStream("data\\PersonGroup\\Daughter\\Daughter1.jpg", FileMode.Open, FileAccess.Read))
    {
        var detectedFaces = await client.Face.DetectWithStreamAsync(fileStream);

        var result = await client.Face.IdentifyAsync(detectedFaces.Select(face => face.FaceId.Value).ToList(), personGroupId);
        Console.WriteLine("Test identify against PersonGroup");
        Console.WriteLine(JsonConvert.SerializeObject(result));
        Console.WriteLine();
    }
}
```

Теперь вы можете использовать новый Персонграуп в целевой подписке. 

Чтобы снова обновить целевой Персонграуп в будущем, создайте новый Персонграуп для получения моментального снимка. Для этого выполните действия, описанные в этом руководстве. К одному объекту Персонграуп можно применить моментальный снимок только один раз.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения переноса данных о лицах вручную удалите объект моментального снимка.

```csharp
await FaceClientEastAsia.Snapshot.DeleteAsync(snapshotId);
```

## <a name="next-steps"></a>Дальнейшие действия

Затем ознакомьтесь с соответствующей справочной документацией по API, изучите пример приложения, использующего функцию создания моментальных снимков, или следуйте инструкциям по началу работы с другими операциями API, упомянутыми здесь:

- [Справочная документация по моментальным снимкам (SDK для .NET)](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperations)
- [Пример снимка грани](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample)
- [Добавление лиц](how-to-add-faces.md)
- [Определение лиц на изображении](HowtoDetectFacesinImage.md)
