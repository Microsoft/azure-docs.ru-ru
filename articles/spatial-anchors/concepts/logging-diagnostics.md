---
title: Ведение журнала и диагностика
description: Подробное описание способов создания и получения журналов и диагностики в пространственных привязках Azure.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: da8ffd7ff0b8473ce558943bb420b36f26c3fc32
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95494647"
---
# <a name="logging-and-diagnostics-in-azure-spatial-anchors"></a>Ведение журналов и диагностика в пространственных привязках Azure

Пространственные привязки Azure предоставляют стандартный механизм ведения журнала, который удобен для разработки приложений. Режим ведения журнала диагностики пространственных привязок полезен, если требуется дополнительная информация для отладки. В журнале диагностики хранятся образы среды.

## <a name="standard-logging"></a>Стандартное ведение журнала
В API-интерфейсе пространственных привязок можно подписываться на механизм ведения журнала, чтобы получить полезные журналы для разработки и отладки приложений. Стандартные API для ведения журнала не хранят изображения среды на диске устройства. Пакет SDK предоставляет эти журналы в качестве обратных вызовов событий. Вы можете интегрировать эти журналы в механизм ведения журнала приложения.

### <a name="configuration-of-log-messages"></a>Настройка сообщений журнала
Существует два обратных вызова, представляющих интерес для пользователя. В следующем примере показано, как настроить сеанс.

```csharp
    cloudSpatialAnchorSession = new CloudSpatialAnchorSession();
    . . .
    // set up the log level for the runtime session
    cloudSpatialAnchorSession.LogLevel = SessionLogLevel.Information;

    // configure the callback for the debug log
    cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;

    // configure the callback for the error log
    cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;
```

### <a name="events-and-properties"></a>События и свойства

Эти обратные вызовы событий предоставляются для обработки журналов и ошибок из сеанса:

- [LogLevel](/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.loglevel): задает уровень детализации для событий, получаемых из среды выполнения.
- [Онлогдебуг](/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.onlogdebug): предоставляет стандартные события журнала отладки.
- [Ошибка](/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.error): предоставляет события журнала, которые среда выполнения считает ошибками.

## <a name="diagnostics-logging"></a>ведет журнал диагностики.

В дополнение к стандартному режиму работы для ведения журнала, пространственные привязки также имеют режим диагностики. Режим диагностики фиксирует образы среды и записывает их на диск. Этот режим можно использовать для отладки определенных типов проблем, например при сбое предсказуемого определения местоположения привязки. Включите ведение журнала диагностики только для воспроизведения определенной проблемы. Затем отключите его. Не включайте диагностику, если вы используете приложения в обычном режиме.

При взаимодействии с корпорацией Майкрософт специалист Майкрософт может попросить вас отправить диагностический пакет для дальнейшего изучения. В этом случае можно включить диагностику и воспроизвести ошибку, чтобы можно было отправить диагностический пакет.

Если вы отправите в корпорацию Майкрософт журнал диагностики без предварительного подтверждения от представителя корпорации Майкрософт, отправка будет неответной.

В следующих разделах показано, как включить режим диагностики, а также как отправлять журналы диагностики в корпорацию Майкрософт.

### <a name="enable-diagnostics-logging"></a>Включение ведения журналов диагностики

При включении сеанса для ведения журнала диагностики все операции в сеансе имеют соответствующие журналы диагностики в локальной файловой системе. Во время ведения журнала образы среды сохраняются на диске.

```csharp
private void ConfigureSession()
{
    cloudSpatialAnchorSession = new CloudSpatialAnchorSession();
    . . .

    // set up the log level for the runtime session
    cloudSpatialAnchorSession.LogLevel = SessionLogLevel.Information;

    // configure the callbacks for logging and errors
    cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;
    cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;

    // opt in to diagnostics logging of environment images
    // if this is enabled, the diagnostics bundle includes images of the environment captured by the session
    cloudSpatialAnchorSession.Diagnostics.ImagesEnabled = true;

    // set the level of detail to be collected in the diagnostics log by the session
    cloudSpatialAnchorSession.Diagnostics.LogLevel = SessionLogLevel.All;

    // set the max bundle size to capture the bug information
    cloudSpatialAnchorSession.Diagnostics.MaxDiskSizeInMB = 200;
    . . .
}
```

### <a name="submit-the-diagnostics-bundle"></a>Отправка пакета диагностики

В следующем фрагменте кода показано, как отправить диагностический пакет в корпорацию Майкрософт. Этот пакет будет включать образы среды, захваченные сеансом, после включения диагностики.

```csharp
// method to handle the diagnostics bundle submission
private async Task CreateAndSubmitBundle()
{
    // create the diagnostics bundle manifest to collect the session information
    string path = await cloudSpatialAnchorSession
                              .Diagnostics
                              .CreateManifestAsync("Description of the issue");

    // submit the manifest and data to send feedback to Microsoft
    await cloudSpatialAnchorSession.Diagnostics.SubmitManifestAsync(path);
}
```

### <a name="parts-of-a-diagnostics-bundle"></a>Части пакета диагностики
Диагностический пакет может содержать следующие сведения:

- **Изображения опорных кадров**: изображения окружения, захваченные во время сеанса, при включенной диагностике.
- **Журналы**: регистрировать события, регистрируемые средой выполнения.
- **Метаданные сеанса**: метаданные, идентифицирующие сеанс.