---
title: Внешние API Azure для проверки подлинности
description: Сведения о том, как использовать внешние API на C# для проверки подлинности.
author: florianborn71
ms.author: flborn
ms.date: 02/12/2010
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 7149d7ac2625eb60a1d0d22253b93b68a99475de
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99592098"
---
# <a name="use-the-azure-frontend-apis-for-authentication"></a>Использование внешних API Azure для проверки подлинности

В этом разделе мы рассмотрим, как использовать API для проверки подлинности и управления сеансами.

> [!CAUTION]
> Функции, описанные в этой главе, выдают внутренним функциям вызовы RESTFUL на сервере. Как и для всех вызовов RESTFUL, отправка этих команд слишком часто приведет к тому, что сервер будет регулировать и возвращать ошибку в конечном итоге. `SessionGeneralContext.HttpResponseCode`В данном случае значение элемента равно 429 ("слишком много запросов"). Поэтому очень важно, чтобы между последовательными вызовами была задержка в **5–10 секунд**.


## <a name="sessionconfiguration"></a>сессионконфигуратион

Сессионконфигуратион используется для настройки данных проверки подлинности для ```RemoteRenderingClient``` экземпляра в пакете SDK.

Самые важные поля в нем:

```cs
public class SessionConfiguration
{
    // Domain that will be used for account authentication for the Azure Remote Rendering service, in the form [region].mixedreality.azure.com.
    // [region] should be set to the domain of the Azure Remote Rendering account.
    public string AccountDomain;
    // Domain that will be used to generate sessions for the Azure Remote Rendering service, in the form [region].mixedreality.azure.com.
    // [region] should be selected based on the region closest to the user. For example, westus2.mixedreality.azure.com or westeurope.mixedreality.azure.com.
    public string RemoteRenderingDomain;

    // Can use one of:
    // 1) ID and Key.
    // 2) ID and AuthenticationToken.
    // 3) ID and AccessToken.
    public string AccountId = Guid.Empty.ToString();
    public string AccountKey = string.Empty;
    public string AuthenticationToken = string.Empty;
    public string AccessToken = string.Empty;
}
```

Аналогичный код на C++ выглядят следующим образом:

```cpp
struct SessionConfiguration
{
    std::string AccountDomain{};
    std::string RemoteRenderingDomain{};
    std::string AccountId{};
    std::string AccountKey{};
    std::string AuthenticationToken{};
    std::string AccessToken{};
};
```

Для сегмента _region_ в домене укажите [ближайший к вам регион](../reference/regions.md).

Сведения об учетной записи можно получить на портале, как описано в абзаце [о получении сведений об учетной записи](create-an-account.md#retrieve-the-account-information).

## <a name="azure-frontend"></a>Внешний интерфейс Azure

Применяются классы ```RemoteRenderingClient``` и ```RenderingSession```. ```RemoteRenderingClient``` используется для управления учетными записями и функциями на уровне учетной записи, к которым относятся преобразование ресурсов и создание сеанса отрисовки. ```RenderingSession``` используется для функций на уровне сеанса, к которым относятся обновление сеанса, запросы, возобновление и списание.

Каждый открытый или созданный класс ```RenderingSession``` сохраняет ссылку на интерфейс, который его создал. Чтобы корректно завершить работу, необходимо освободить все сеансы, а затем внешний интерфейс.

Отмена выделения сеанса не приведет к прерыванию сервера в Azure, `RenderingSession.StopAsync` его необходимо вызвать явным образом.

Когда сеанс будет создан и его состояние получит значение "Готов", станет возможным подключение к удаленной среде выполнения отрисовки с помощью `RenderingSession.ConnectAsync`.

### <a name="threading"></a>Потоки

Все асинхронные вызовы Рендерингсессион и Ремотерендерингклиент выполняются в фоновом потоке, а не в главном потоке приложения.

### <a name="conversion-apis"></a>Интерфейсы API преобразования

Дополнительные сведения о службе преобразования см. в статье [Использование REST API преобразования модели](conversion/conversion-rest-api.md).

#### <a name="start-asset-conversion"></a>Запуск преобразования ресурса

```cs
async void StartAssetConversion(RemoteRenderingClient client, string storageContainer, string blobinputpath, string bloboutpath, string modelName, string outputName)
{
    var result = await client.StartAssetConversionAsync(
        new AssetConversionInputOptions(storageContainer, blobinputpath, "", modelName),
        new AssetConversionOutputOptions(storageContainer, bloboutpath, "", outputName)
        );
}
```

```cpp
void StartAssetConversion(ApiHandle<RemoteRenderingClient> client, std::string storageContainer, std::string blobinputpath, std::string bloboutpath, std::string modelName, std::string outputName)
{
    AssetConversionInputOptions input;
    input.BlobContainerInformation.BlobContainerName = blobinputpath;
    input.BlobContainerInformation.StorageAccountName = storageContainer;
    input.BlobContainerInformation.FolderPath = "";
    input.InputAssetPath = modelName;

    AssetConversionOutputOptions output;
    output.BlobContainerInformation.BlobContainerName = blobinputpath;
    output.BlobContainerInformation.StorageAccountName = storageContainer;
    output.BlobContainerInformation.FolderPath = "";
    output.OutputAssetPath = outputName;

    client->StartAssetConversionAsync(input, output, [](Status status, ApiHandle<AssetConversionResult> result) {
        if (status == Status::OK)
        {
            //use result
        }
        else
        {
            printf("Failed to start asset conversion!");
        }
    });
}
```

#### <a name="get-conversion-status"></a>Получение состояния преобразования

```cs
async void GetConversionStatus(RemoteRenderingClient client, string assetId)
{
    AssetConversionStatusResult status = await client.GetAssetConversionStatusAsync(assetId);
    // do something with status (e.g. check current status etc.)
}
```

```cpp
void GetConversionStatus(ApiHandle<RemoteRenderingClient> client, std::string assetId)
{
    client->GetAssetConversionStatusAsync(assetId, [](Status status, ApiHandle<AssetConversionStatusResult> result) {
        if (status == Status::OK)
        {
            // do something with result (e.g. check current status etc.)
        }
        else
        {
            printf("Failed to get status of asset conversion!");
        }
    });
}

```


### <a name="rendering-apis"></a>Интерфейсы API отрисовки

Подробные сведения об управлении сеансами см. в статье [Использование REST API управления сеансами](session-rest-api.md).

Сеанс отрисовки может быть создан либо динамически в службе, либо уже существующий идентификатор сеанса можно открыть в объекте Рендерингсессион.

#### <a name="create-rendering-session"></a>Создание сеанса отрисовки

```cs
async void CreateRenderingSession(RemoteRenderingClient client, RenderingSessionVmSize vmSize, int maxLeaseInMinutes)
{
    CreateRenderingSessionResult result = await client.CreateNewRenderingSessionAsync(
        new RenderingSessionCreationOptions(vmSize, maxLeaseInMinutes / 60, maxLeaseInMinutes % 60));

    // if the call was successful, result.Session holds a valid session reference, otherwise check result.Context for error information
}
```

```cpp
void CreateRenderingSession(ApiHandle<RemoteRenderingClient> client, RenderingSessionVmSize vmSize, int maxLeaseInMinutes)
{
    RenderingSessionCreationOptions params;
    params.MaxLeaseInMinutes = maxLeaseInMinutes;
    params.Size = vmSize;
    client->CreateNewRenderingSessionAsync(params, [](Status status, ApiHandle<CreateRenderingSessionResult> result) {
        if (status == Status::OK && result->GetErrorCode() == Result::Success)
        {
            result->GetSession();
            //use res->Result
        }
        else
        {
            printf("Failed to create session!");
        }
    });
}
```

#### <a name="open-an-existing-rendering-session"></a>Открытие существующего сеанса отрисовки

Открытие существующего сеанса выполняется как синхронный вызов.

```cs
async void CreateRenderingSession(RemoteRenderingClient client, string sessionId)
{
    CreateRenderingSessionResult result = await client.OpenRenderingSessionAsync(sessionId);
    if (result.ErrorCode == Result.Success)
    {
        RenderingSession session = result.Session;
        // Query session status, etc.
    }
}
```

```cpp
void CreateRenderingSession(ApiHandle<RemoteRenderingClient> client, std::string sessionId)
{
    client->OpenRenderingSessionAsync(sessionId, [](Status status, ApiHandle<CreateRenderingSessionResult> result) {
        if (status == Status::OK && result->GetErrorCode()==Result::Success)
        {
            ApiHandle<RenderingSession> session = result->GetSession();
            // Query session status, etc.
        }
    });
}
```


#### <a name="get-current-rendering-sessions"></a>Получение текущих сеансов отрисовки

```cs
async void GetCurrentRenderingSessions(RemoteRenderingClient client)
{
    RenderingSessionPropertiesArrayResult result = await client.GetCurrentRenderingSessionsAsync();
    if (result.ErrorCode == Result.Success)
    {
        RenderingSessionProperties[] properties = result.SessionProperties;
        // Query session status, etc.
    }
}
```

```cpp
void GetCurrentRenderingSessions(ApiHandle<RemoteRenderingClient> client)
{
    client->GetCurrentRenderingSessionsAsync([](Status status, ApiHandle<RenderingSessionPropertiesArrayResult> result) {
        if (status == Status::OK && result->GetErrorCode() == Result::Success)
        {
            std::vector<RenderingSessionProperties> properties;
            result->GetSessionProperties(properties);
        }
        else
        {
            printf("Failed to get current rendering sessions!");
        }
    });
}
```

### <a name="session-apis"></a>Интерфейсы API сеансов

#### <a name="get-rendering-session-properties"></a>Получение свойств сеанса отрисовки

```cs
async void GetRenderingSessionProperties(RenderingSession session)
{
    RenderingSessionPropertiesResult result = await session.GetPropertiesAsync();
    if (result.ErrorCode == Result.Success)
    {
        RenderingSessionProperties properties = result.SessionProperties;
    }
    else
    {
        Console.WriteLine("Failed to get properties of session!");
    }
}
```

```cpp
void GetRenderingSessionProperties(ApiHandle<RenderingSession> session)
{
    session->GetPropertiesAsync([](Status status, ApiHandle<RenderingSessionPropertiesResult> result) {
        if (status == Status::OK && result->GetErrorCode() == Result::Success)
        {
            RenderingSessionProperties properties = result->GetSessionProperties();
        }
        else
        {
            printf("Failed to get properties of session!");
        }
    });
}
```

#### <a name="update-rendering-session"></a>Обновление сеанса отрисовки

```cs
async void UpdateRenderingSession(RenderingSession session, int updatedLeaseInMinutes)
{
    SessionContextResult result = await session.RenewAsync(
        new RenderingSessionUpdateOptions(updatedLeaseInMinutes / 60, updatedLeaseInMinutes % 60));
    if (result.ErrorCode == Result.Success)
    {
        Console.WriteLine("Rendering session renewed succeeded!");
    }
    else
    {
        Console.WriteLine("Failed to renew rendering session!");
    }
}
```

```cpp
void UpdateRenderingSession(ApiHandle<RenderingSession> session, int updatedLeaseInMinutes)
{
    RenderingSessionUpdateOptions params;
    params.MaxLeaseInMinutes = updatedLeaseInMinutes;
    session->RenewAsync(params, [](Status status, ApiHandle<SessionContextResult> result) {
        if (status == Status::OK && result->GetErrorCode() == Result::Success)
        {
            printf("Rendering session renewed succeeded!");
        }
        else
        {
            printf("Failed to renew rendering session!");
        }
    });
}
```

#### <a name="stop-rendering-session"></a>Остановка сеанса отрисовки

```cs
async void StopRenderingSession(RenderingSession session)
{
    SessionContextResult result = await session.StopAsync();
    if (result.ErrorCode == Result.Success)
    {
        Console.WriteLine("Rendering session stopped successfully!");
    }
    else
    {
        Console.WriteLine("Failed to stop rendering session!");
    }
}
```

```cpp
void StopRenderingSession(ApiHandle<RenderingSession> session)
{
    session->StopAsync([](Status status, ApiHandle<SessionContextResult> result) {
        if (status == Status::OK && result->GetErrorCode() == Result::Success)
        {
            printf("Rendering session stopped successfully!");
        }
        else
        {
            printf("Failed to stop rendering session!");
        }
    });
}
```

#### <a name="connect-to-arr-inspector"></a>Подключение к инспектору ARR

```cs
async void ConnectToArrInspector(RenderingSession session)
{
    string htmlPath = await session.ConnectToArrInspectorAsync();
#if WINDOWS_UWP
    UnityEngine.WSA.Application.InvokeOnUIThread(async () =>
    {
        var file = await Windows.Storage.StorageFile.GetFileFromPathAsync(htmlPath);
        await Windows.System.Launcher.LaunchFileAsync(file);
    }, true);
#else
    InvokeOnAppThreadAsync(() =>
        {
            System.Diagnostics.Process.Start("file:///" + htmlPath);
        });
#endif
}
```

```cpp
void ConnectToArrInspector(ApiHandle<RenderingSession> session)
{
    session->ConnectToArrInspectorAsync([](Status status, std::string result) {
        if (status == Status::OK)
        {
            // Launch the html file with default browser
            std::string htmlPath = "file:///" + result;
            ShellExecuteA(NULL, "open", htmlPath.c_str(), NULL, NULL, SW_SHOWDEFAULT);
        }
        else
        {
            printf("Failed to connect to ARR inspector!");
        }
    });
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Создание учетной записи](create-an-account.md)
* [Примеры скриптов PowerShell](../samples/powershell-example-scripts.md)
