---
title: Общие сведения о пакете SDK среды выполнения
description: Ознакомьтесь с пакетом SDK среды выполнения привязок объектов.
author: craigktreasure
manager: vriveras
services: azure-object-anchors
ms.author: crtreasu
ms.date: 03/02/2021
ms.topic: conceptual
ms.service: azure-object-anchors
ms.openlocfilehash: 74663f05c5ff995a090c7cd35e4edf46a754da17
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102034614"
---
# <a name="runtime-sdk-overview"></a>Общие сведения о пакете SDK среды выполнения

В этом разделе представлен общий обзор пакета SDK среды выполнения привязок объектов, который используется для обнаружения объектов с помощью модели привязок объектов. Вы получите представление о том, как представляется объект и какие используются различные компоненты.

Все типы, описанные ниже, можно найти в пространстве имен **Microsoft. микседреалити. обжектанчорс** .

## <a name="types"></a>Типы

### <a name="objectmodel"></a>ObjectModel

[ObjectModel](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectmodel) представляет геометрию физического объекта и кодирует необходимые параметры для оценки обнаружения и выстановки. Его необходимо создать с помощью [службы "привязки объектов](../quickstarts/get-started-model-conversion.md)". Затем приложение может загрузить созданный файл модели с помощью API привязок объектов и запросить сетку, внедренную в эту модель для визуализации.

### <a name="objectsearcharea"></a>обжектсеарчареа

[Обжектсеарчареа](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectsearcharea) указывает место для поиска одного или нескольких объектов. Он определяется ИДЕНТИФИКАТОРом узла пространственного графа и пространственными границами в системе координат, представленной ИДЕНТИФИКАТОРом узла пространственного графа. Пакет SDK для привязки объектов поддерживает четыре типа границ, а именно, **поле представления**, **ограничивающий прямоугольник**, **сферу** и **Расположение**.

### <a name="objectquery"></a>ObjectQuery

[ObjectQuery](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectquery) сообщает **наблюдателю объекта** о том, как найти объекты данной модели. Он предоставляет следующие настраиваемые параметры, значения по умолчанию которых можно получить из объектной модели.

#### <a name="minsurfacecoverage"></a>минсурфацековераже

Свойство [минсурфацековераже](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectquery.minsurfacecoverage) указывает значение, которое будет рассматриваться как обнаруженный экземпляр.

Для каждого кандидата объекта **наблюдатель** рассчитывает отношение перекрывающихся поверхностей между преобразованной объектной моделью и сценой, а затем сообщает об этом кандидату приложению, только если коэффициент покрытия превышает заданное пороговое значение.

#### <a name="isexpectedtobestandingongroundplane"></a>исекспектедтобестандингонграундплане

Свойство [исекспектедтобестандингонграундплане](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectquery.isexpectedtobestandingongroundplane) указывает, ожидается ли целевой объект на плоскости заземления.

Плоскость основания — это наименьшее горизонтальное основание в области поиска. Он предоставляет хорошее ограничение для возможных объектов. Включение этого флага поможет **наблюдателю** оценить наличие в ограниченном пространстве и улучшить точность. Этот параметр будет пропущен, если модель не должна находиться на плоскости заземления.

#### <a name="expectedmaxverticalorientationindegrees"></a>експектедмаксвертикалориентатиониндегрис

Свойство [експектедмаксвертикалориентатиониндегрис](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectquery.expectedmaxverticalorientationindegrees) указывает ожидаемый максимальный угол в градусах между направлением экземпляра объекта и сила притяжения.

Этот параметр предоставляет другое ограничение в направлении сверху от предполагаемого объекта. Например, если объект находится в правой части, этот параметр может иметь значение 0. Привязки объектов не должны обнаруживать объекты, которые отличаются от модели. Если модель работает правильно, то она не будет обнаруживать раскрывающийся список экземпляров. Для раскрывающегося макета будет использоваться новая модель. Для артикулатион применяется то же правило.

#### <a name="maxscalechange"></a>максскалечанже

Свойство [максскалечанже](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectquery.maxscalechange) указывает максимальное изменение масштаба объекта (в пределах 0 ~ 1) относительно пространственного сопоставления. Оценочная шкала применяется к преобразованным вершинам объектов, центрированным по источнику и выровненной по осям. Предполагаемое масштабирование не может быть фактическим масштабом между моделью САПР и ее физическим представлением, но некоторые значения, позволяющие приложению визуализировать объектную модель, близко к пространственному сопоставлению физического объекта.

#### <a name="searchareas"></a>сеарчареас

Свойство [сеарчареас](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectquery.searchareas) указывает массив пространственных границ, где можно найти объекты.

**Наблюдатель** будет искать объекты в пространстве Union всех областей поиска, указанных в запросе. В этом выпуске мы будем возвращать не более одного объекта с наивысшей достоверностью, чтобы сократить задержку.

### <a name="objectinstance"></a>ObjectInstance

[ObjectInstance](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectinstance) представляет гипотетическую позицию, в которой экземпляр данной модели может находиться в системе координат HoloLens. Каждый экземпляр поставляется со `SurfaceCoverage` свойством, указывающим, насколько хороша предполагаемая цель.

Экземпляр создается путем вызова `ObjectObserver.DetectAsync` метода, который затем обновляется автоматически в фоновом режиме, когда активно. Приложение может прослушивать событие изменения состояния в определенном экземпляре или изменить режим отслеживания, чтобы приостановить или возобновить обновление. При потере отслеживания экземпляр будет автоматически удален из **наблюдателя** .

### <a name="objectobserver"></a>обжектобсервер

[Обжектобсервер](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.objectobserver) загружает объектные модели, обнаруживает их экземпляры и сообщает 6 — ДОФ экземпляров каждого экземпляра в системе координат HoloLens.

Хотя любая объектная модель или экземпляр создается из **наблюдателя**, их времена жизни независимы. Приложение может ликвидировать наблюдателя и продолжать использовать объектную модель или экземпляр.

### <a name="objectdiagnosticssession"></a>обжектдиагностикссессион

[Обжектдиагностиксессион](https://docs.microsoft.com/dotnet/api/microsoft.azure.objectanchors.diagnostics.objectdiagnosticssession) записывает сведения о диагностике и записывает данные в архив.

Архив диагностики включает облако точек сцены Cloud, состояние наблюдателя и сведения о моделях. Эти сведения полезны для обнаружения возможных проблем во время выполнения. Дополнительные сведения см. в разделе [Часто задаваемые вопросы](../faq.md).

## <a name="runtime-sdk-usage-and-details"></a>Использование и сведения о пакете SDK среды выполнения

В этом разделе приводятся основные сведения об использовании пакета SDK среды выполнения. Он должен предоставить достаточный контекст для просмотра примеров приложений, чтобы увидеть, как используются привязки объектов в глобальном порядке.

### <a name="initialization"></a>Инициализация

Приложения должны вызывать API, `ObjectObserver.IsSupported()` чтобы определить, поддерживаются ли привязки объектов на устройстве перед его использованием. Если `ObjectObserver.IsSupported()` API возвращает `false` , убедитесь, что приложение включило функцию **спатиалперцептион** АНД\ОР Upgrade до последней ОС HoloLens.

```cs
if (!ObjectObserver.IsSupported())
{
    // Handle the error
}

// This call should grant the access we need.
var status = await ObjectObserver.RequestAccessAsync();
if(status != ObjectObserverStatus.Allowed)
{
    // Handle the error
}
```

Затем приложение создает наблюдателя объекта и загружает необходимые модели, созданные [службой преобразования модели привязок объектов](../quickstarts/get-started-model-conversion.md).

```cs
var observer = new ObjectObserver();

byte[] modelAsBytes; // Load a model into a byte array. Model could be a file, an embedded resource, or a network stream.
var model = await observer.LoadObjectModelAsync(modelAsBytes);

// Note that after a model is loaded, its vertices and normals are transformed into a centered coordinate system for the 
// ease of computing the object pose. The rigid transform can be retrieved through the `OriginToCenterTransform` property.
```

Приложение создает запрос для обнаружения экземпляров этой модели в пространстве.

```cs
var coordinateSystem = default(SpatialGraphCoordinateSystem);
var searchAreaAsBox = ObjectSearchArea.FromOrientedBox(coordinateSystem, default(SpatialOrientedBox));

// Optionally change the parameters, otherwise use the default values embedded in the model.
var query = new ObjectQuery(model);
query.MinSurfaceCoverage = 0.2f;
query.ExpectedMaxVerticalOrientationInDegrees = 1.5f;
query.MaxScaleChange = 0.1f;
query.SearchAreas.Add(searchAreaAsBox);

// Detection could take long, run in a background thread.
var detectedObjects = await observer.DetectAsync(query);
```

По умолчанию каждый обнаруженный экземпляр будет автоматически отслеживаться **наблюдателем**. При необходимости можно манипулировать этими экземплярами, изменив режим отслеживания или прослушивая событие изменения состояния.

```cs
foreach (var instance in detectedObjects)
{
    // Supported modes:
    // "LowLatencyCoarsePosition" - consumes less CPU cycles thus fast to update the state.
    // "HighLatencyAccuratePosition" - (not yet implemented) consumes more CPU cycles thus potentially taking longer time to update the state.
    // "Paused" - stops to update the state until mode changed to low or high.
    instance.Mode = ObjectInstanceTrackingMode.LowLatencyCoarsePosition;

    // Listen to state changed event on this instance.
    instance.Changed += InstanceChangedHandler;

    // Optionally dispose an instance if not interested in it.
    //instance.Dispose();
}
```

В событии изменения состояния можно запросить Последнее состояние или ликвидировать экземпляр, если он потерял отслеживание.

```cs
var InstanceChangedHandler = new Windows.Foundation.TypedEventHandler<ObjectInstance, ObjectInstanceChangedEventArgs>((sender, args) =>
{
    // Try to query the current instance state.
    var state = sender.TryGetCurrentState();

    if (state.HasValue)
    {
        // Process latest state via state.Value.
        // An object pose includes scale, rotation and translation, applied in the same order
        // to the object model in the centered coordinate system.
    }
    else
    {
        // This object instance is lost for tracking, and will never be recovered.
        // The caller can detach the Changed event handler from this instance and dispose it.
        sender.Dispose();
    }
});
```

Кроме того, приложение может дополнительно записать один или несколько сеансов диагностики для автономной отладки.

```cs
string diagnosticsFolderPath = Windows.Storage.ApplicationData.Current.TemporaryFolder.Path;
const uint maxSessionSizeInMegaBytes = uint.MaxValue;

// Recording starts on the creation of a diagnostics session.
var diagnostics = new ObjectDiagnosticsSession(observer, maxSessionSizeInMegaBytes);

// Wait for the observer to do a job.

// Application can report some **pseudo ground-truth** pose for an instance acquired from other means.
diagnostics.ReportActualInstanceLocation(instance, coordinateSystem, Vector3.Zero, Quaternion.Identity);

// Close a session and write the diagnostics into an archive at specified location.
await diagnostics.CloseAsync(System.IO.Path.Combine(diagnosticsFolderPath, "diagnostics.zip"));
```

Наконец, мы выпускаем ресурсы путем удаления всех объектов.

```cs
foreach(var instance in activeInstances)
{
    instance.Changed -= InstanceChangedHandler;
    instance.Dispose();
}

model.Dispose();
observer.Dispose();
```
