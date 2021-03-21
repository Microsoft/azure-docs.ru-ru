---
title: Устранение задержки при использовании службы распознавания лиц
titleSuffix: Azure Cognitive Services
description: Узнайте, как уменьшить задержку при использовании службы распознавания лиц.
services: cognitive-services
author: v-jaswel
manager: chrhoder
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 1/5/2021
ms.author: v-jawe
ms.openlocfilehash: 2c771509de5ac246bac0d8e006a5d0b884a410b0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101706815"
---
# <a name="how-to-mitigate-latency-when-using-the-face-service"></a>Как уменьшить задержку при использовании службы распознавания лиц

При использовании службы распознавания лиц может возникать задержка. Задержка относится к любой задержке, возникающей при обмене данными по сети. Как правило, возможны следующие причины задержки:
- Физическое расстояние, в течение которого каждый пакет должен проходить от источника к назначению.
- Проблемы с носителем передачи.
- Ошибки в маршрутизаторах или коммутаторах вдоль пути передачи.
- Время, необходимое для проверки пакетов антивирусными программами, брандмауэрами и другими механизмами безопасности.
- Неисправен в клиентских или серверных приложениях.

В этом разделе рассказывается о возможных причинах задержек, связанных с использованием Cognitive Services Azure, и о том, как можно устранить эти причины.

> [!NOTE]
> Cognitive Services Azure не предоставляет Соглашение об уровне обслуживания (SLA) в отношении задержки.

## <a name="possible-causes-of-latency"></a>Возможные причины задержки

### <a name="slow-connection-between-the-cognitive-service-and-a-remote-url"></a>Снижение скорости подключения между службой и удаленным URL-адресом

Некоторые Cognitive Services Azure предоставляют методы, которые получают данные из предоставленного удаленного URL-адреса. Например, при вызове [метода детектвисурласинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync#Microsoft_Azure_CognitiveServices_Vision_Face_FaceOperationsExtensions_DetectWithUrlAsync_Microsoft_Azure_CognitiveServices_Vision_Face_IFaceOperations_System_String_System_Nullable_System_Boolean__System_Nullable_System_Boolean__System_Collections_Generic_IList_System_Nullable_Microsoft_Azure_CognitiveServices_Vision_Face_Models_FaceAttributeType___System_String_System_Nullable_System_Boolean__System_String_System_Threading_CancellationToken_) службы Face можно указать URL-адрес изображения, в котором служба пытается обнаружить грани.

```csharp
var faces = await client.Face.DetectWithUrlAsync("https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg");
```

Затем служба распознавания лиц должна загрузить образ с удаленного сервера. Если подключение из службы "лица" к удаленному серверу работает слишком долго, то оно повлияет на время отклика метода обнаружения.

Чтобы устранить эту проблемы, рекомендуется [сохранить образ в хранилище BLOB-объектов Azure Premium](../../../storage/blobs/storage-upload-process-images.md?tabs=dotnet). Пример:

``` csharp
var faces = await client.Face.DetectWithUrlAsync("https://csdx.blob.core.windows.net/resources/Face/Images/Family1-Daughter1.jpg");
```

### <a name="large-upload-size"></a>Большой размер передачи

Некоторые Cognitive Services Azure предоставляют методы, которые получают данные из отправляемого файла. Например, при вызове [метода детектвисстреамасинк](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync#Microsoft_Azure_CognitiveServices_Vision_Face_FaceOperationsExtensions_DetectWithStreamAsync_Microsoft_Azure_CognitiveServices_Vision_Face_IFaceOperations_System_IO_Stream_System_Nullable_System_Boolean__System_Nullable_System_Boolean__System_Collections_Generic_IList_System_Nullable_Microsoft_Azure_CognitiveServices_Vision_Face_Models_FaceAttributeType___System_String_System_Nullable_System_Boolean__System_String_System_Threading_CancellationToken_) службы распознавания изображений можно передать изображение, в котором служба пытается обнаружить лиц.

```csharp
using FileStream fs = File.OpenRead(@"C:\images\face.jpg");
System.Collections.Generic.IList<DetectedFace> faces = await client.Face.DetectWithStreamAsync(fs, detectionModel: DetectionModel.Detection02);
```

Если файл для отправки имеет большой размер, он повлияет на время отклика `DetectWithStreamAsync` метода по следующим причинам:
- Отправка файла занимает больше времени.
- Она занимает больше времени для обработки файла, пропорционально размеру файла.

Устранение рисков:
- Рекомендуется [сохранить образ в хранилище BLOB-объектов Azure Premium](../../../storage/blobs/storage-upload-process-images.md?tabs=dotnet). Пример:
``` csharp
var faces = await client.Face.DetectWithUrlAsync("https://csdx.blob.core.windows.net/resources/Face/Images/Family1-Daughter1.jpg");
```
- Попробуйте отправить файл меньшего размера.
    - См. рекомендации по [входным данным для обнаружения лиц](../concepts/face-detection.md#input-data) и [входных данных для распознавания лиц](../concepts/face-recognition.md#input-data).
    - При обнаружении лиц при использовании модели обнаружения `DetectionModel.Detection01` уменьшение размера файла изображения увеличит скорость обработки. При использовании модели обнаружения `DetectionModel.Detection02` уменьшение размера файла изображения увеличивает скорость обработки, только если размер файла изображения меньше 1920 x 1080.
    - Для распознавания лиц уменьшение размера шрифта до 200x200 пикселей не влияет на точность модели распознавания.
    - Производительность `DetectWithUrlAsync` `DetectWithStreamAsync` методов и также зависит от количества лиц в изображении. Служба распознавания лиц может возвращать до 100 лиц для изображения. Стороны имеют ранжирование по размеру прямоугольника от больших к большим.
    - Если необходимо вызвать несколько методов службы, рекомендуется вызывать их параллельно, если проект приложения допускает его. Например, если необходимо обнаружить грани на двух изображениях для сравнения лиц, выполните следующие действия:
```csharp
var faces_1 = client.Face.DetectWithUrlAsync("https://www.biography.com/.image/t_share/MTQ1MzAyNzYzOTgxNTE0NTEz/john-f-kennedy---mini-biography.jpg");
var faces_2 = client.Face.DetectWithUrlAsync("https://www.biography.com/.image/t_share/MTQ1NDY3OTIxMzExNzM3NjE3/john-f-kennedy---debating-richard-nixon.jpg");
Task.WaitAll (new Task<IList<DetectedFace>>[] { faces_1, faces_2 });
IEnumerable<DetectedFace> results = faces_1.Result.Concat (faces_2.Result);
```

### <a name="slow-connection-between-your-compute-resource-and-the-face-service"></a>Снижение скорости подключения между ресурсами вычислений и службой лиц

Если компьютер имеет низкую связь со службой распознавания лиц, это повлияет на время отклика методов службы.

Устранение рисков:
- При создании подписки на приложение необходимо выбрать регион, ближайший к месту размещения приложения.
- Если необходимо вызвать несколько методов службы, рекомендуется вызывать их параллельно, если проект приложения допускает его. Пример см. в предыдущем разделе.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как уменьшить задержку при использовании службы распознавания лиц. Далее Узнайте, как масштабироваться от существующих объектов Персонграуп и Фацелист до объектов Ларжеперсонграуп и LargeFaceList соответственно.

> [!div class="nextstepaction"]
> [Пример Использование функции для увеличения масштаба](how-to-use-large-scale.md)

## <a name="related-topics"></a>Связанные темы

- [Справочная документация (ОСТАВШАЯся)](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
- [Справочная документация (пакет SDK для .NET)](/dotnet/api/overview/azure/cognitiveservices/client/faceapi)