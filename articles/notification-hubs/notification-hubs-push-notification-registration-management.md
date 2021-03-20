---
title: Управление регистрацией
description: В этом разделе описывается регистрация устройств в центрах уведомлений для получения push-уведомлений.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: fd0ee230-132c-4143-b4f9-65cef7f463a1
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 07/07/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 04/08/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 7807d28da459656938acb399eb8c621e4c292372
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89001575"
---
# <a name="registration-management"></a>Управление регистрацией

В этом разделе описывается регистрация устройств в центрах уведомлений для получения push-уведомлений. Сначала приводится общее описание, затем демонстрируются два основных типа регистрации устройств: регистрация с устройства непосредственно в центре уведомлений и регистрация с помощью серверной части приложения.

## <a name="what-is-device-registration"></a>Регистрация устройств

Регистрация устройств в центре уведомлений осуществляется через **регистрацию** или **установку**.

### <a name="registrations"></a>Регистрация

Регистрация связывает маркер службы отправки уведомлений платформы (PNS) для устройства с тегами и, возможно, шаблоном. Маркером PNS может быть ChannelURI, маркер устройства или идентификатор регистрации FCM. Теги используются для направления уведомлений правильному набору маркеров устройств. Дополнительные сведения см. в разделе [Маршрутизация и выражения тегов](notification-hubs-tags-segment-push-message.md). Шаблоны используются для преобразований в рамках регистрации. Дополнительные сведения см. в статье [Шаблоны](notification-hubs-templates-cross-platform-push-messages.md).

> [!NOTE]
> Центры уведомлений Azure поддерживают не более 60 тегов для каждого устройства.

### <a name="installations"></a>Установки

Установка — это расширенная регистрация, включающая набор свойств, связанных с push-уведомлениями. Это новый и рекомендуемый способ регистрации устройств. Однако в настоящее время он не поддерживается пакетом SDK .NET на стороне клиента ([пакетом SDK центра уведомлений для серверных операций](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)).  Это означает, что при регистрации с самого клиентского устройства вам потребуется использовать [REST API концентраторов уведомлений](/rest/api/notificationhubs/create-overwrite-installation) для поддержки установки. При использовании внутренней службы должна иметься возможность применения [пакета SDK концентратора уведомлений для серверных операций](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

Ниже приведены некоторые ключевые преимущества регистрации через установку.

- Создание или обновление установки является полностью идемпотентной операцией. Поэтому его можно повторять, не заботясь о дублирующихся регистрациях.
- Модель установки поддерживает специальный формат тегов ( `$InstallationId:{INSTALLATION_ID}` ), который позволяет отправлять уведомления непосредственно на конкретное устройство. Например, если код приложения задает идентификатор установки `joe93developer` для этого устройства, разработчик может ориентироваться на это устройство при отправке уведомления в `$InstallationId:{joe93developer}` тег. Это позволяет ориентироваться на конкретное устройство, не требуя дополнительного программирования.
- Установка также позволяет частично обновлять регистрацию. Частичное обновление установки запрашивается с помощью метода PATCH из [стандарта JSON-Patch](https://tools.ietf.org/html/rfc6902). Это удобно в тех случаях, когда нужно обновить теги в регистрации. Нет необходимости удалять всю регистрацию и заново отправлять все прошлые теги.

Установка может содержать следующие свойства. Полный список свойств установки см. в статье об [установке или перезаписи установки с помощью REST API](/rest/api/notificationhubs/create-overwrite-installation) или в статье [Класс Installation](/dotnet/api/microsoft.azure.notificationhubs.installation).

```json
// Example installation format to show some supported properties
{
    installationId: "",
    expirationTime: "",
    tags: [],
    platform: "",
    pushChannel: "",
    ………
    templates: {
        "templateName1" : {
            body: "",
            tags: [] },
        "templateName2" : {
            body: "",
            // Headers are for Windows Store only
            headers: {
                "X-WNS-Type": "wns/tile" }
            tags: [] }
    },
    secondaryTiles: {
        "tileId1": {
            pushChannel: "",
            tags: [],
            templates: {
                "otherTemplate": {
                    bodyTemplate: "",
                    headers: {
                        ... }
                    tags: [] }
            }
        }
    }
}
```

> [!NOTE]
> По умолчанию срок регистрации и установки не истекает.

Регистрации и установки должны содержать действительный дескриптор PNS для каждого устройства или канала. Поскольку маркеры PNS можно получить только в клиентском приложении на устройстве, одним из типовых способов регистрации является регистрация непосредственно на этом устройстве с клиентским приложением. С другой стороны, режим безопасности и бизнес-логика, связанная с тегами, могут требовать, чтобы управление регистрацией устройств осуществлялось в серверной части приложения.

> [!NOTE]
> API установки не поддерживает службу Baidu (хотя API регистраций). 

### <a name="templates"></a>Шаблоны

Если вы хотите использовать [шаблоны](notification-hubs-templates-cross-platform-push-messages.md), то установка устройства содержит все шаблоны, связанные с этим устройством, в формате JSON (см. пример выше). Названия шаблонов помогают связывать разные шаблоны с одним устройством.

Имя каждого шаблона сопоставляется с текстом шаблона и необязательным набором тегов. Кроме того, каждая платформа может иметь дополнительные свойства шаблонов. Для Магазина Windows (при использовании WNS) и Windows Phone 8 (при использовании MPNS) в состав шаблона может входить дополнительный набор заголовков. Для имени точки доступа (APN) в свойстве окончания срока действия можно задать фиксированное значение или связать его с выражением шаблона. Полный список свойств установки см. в статье о [создании или замене установки с помощью REST](/rest/api/notificationhubs/create-overwrite-installation).

### <a name="secondary-tiles-for-windows-store-apps"></a>Вспомогательные плитки для приложений из Магазина Windows

Для клиентских приложений из Магазина Windows отправка уведомлений во вспомогательные плитки выполняется аналогично отправке уведомлений в основные плитки. Это также поддерживается в установках. У вспомогательных элементов другое значение ChannelUri, которое автоматически обрабатывается пакетом SDK в клиентском приложении.

В словаре SecondaryTiles используется тот же идентификатор TileId, с помощью которого создается объект SecondaryTiles в приложении Магазина Windows. Как и в случае с основным ChannelUri, идентификаторы ChannelUri вспомогательных плиток можно изменить в любой момент. Для поддержки актуальности установок в центре уведомлений устройство должно обновлять их с использованием текущих ChannelUri вспомогательных плиток.

## <a name="registration-management-from-the-device"></a>Управление регистрацией с устройства

При управлении регистрацией устройства из клиентских приложений серверная часть отвечает только за отправку уведомлений. Клиентские приложения поддерживают актуальность дескрипторов PNS и регистрируют теги. На следующем рисунке показана схема работы.

![Регистрация с устройства](./media/notification-hubs-registration-management/notification-hubs-registering-on-device.png)

Сначала устройство извлекает маркер PNS из службы PNS, а затем напрямую регистрируется в центре уведомлений. После успешной регистрации серверная часть приложения может отправить уведомление для этой регистрации. Дополнительные сведения об отправке уведомлений см. в статье [Маршрутизация и выражения тегов](notification-hubs-tags-segment-push-message.md).

В этом случае для доступа к своим центрам уведомлений с устройства следует использовать только права прослушивания. Дополнительные сведения см. в статье [Безопасность](notification-hubs-push-notification-security.md).

Регистрация с устройства — это самый простой метод, но у него есть свои недостатки.

- Клиентское приложение может обновлять свои теги, только когда оно активно. Например, если у пользователя есть два устройства, которые регистрируют теги, связанные со спортивными командами, то когда первое устройство регистрирует дополнительный тег (например Seahawks), второе устройство получит уведомление о теге Seahawks только при следующем запуске приложения на втором устройстве. В целом, если на теги влияют сразу несколько устройств, рекомендуется управлять тегами из серверной части.
- Так как приложения могут быть взломаны, защита регистрации в конкретных тегах требует дополнительной осторожности, как описано в статье [Безопасность](notification-hubs-push-notification-security.md).

### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-an-installation"></a>Пример кода регистрации в центре уведомлений с устройства с помощью установки

В настоящее время эта процедура поддерживается только с помощью [API REST центра уведомлений](/rest/api/notificationhubs/create-overwrite-installation).

Для обновления установки также можно использовать метод PATCH из [стандарта JSON-Patch](https://tools.ietf.org/html/rfc6902) .

```csharp
class DeviceInstallation
{
    public string installationId { get; set; }
    public string platform { get; set; }
    public string pushChannel { get; set; }
    public string[] tags { get; set; }

    private async Task<HttpStatusCode> CreateOrUpdateInstallationAsync(DeviceInstallation deviceInstallation,
        string hubName, string listenConnectionString)
    {
        if (deviceInstallation.installationId == null)
            return HttpStatusCode.BadRequest;

        // Parse connection string (https://msdn.microsoft.com/library/azure/dn495627.aspx)
        ConnectionStringUtility connectionSaSUtil = new ConnectionStringUtility(listenConnectionString);
        string hubResource = "installations/" + deviceInstallation.installationId + "?";
        string apiVersion = "api-version=2015-04";

        // Determine the targetUri that we will sign
        string uri = connectionSaSUtil.Endpoint + hubName + "/" + hubResource + apiVersion;

        //=== Generate SaS Security Token for Authorization header ===
        // See https://msdn.microsoft.com/library/azure/dn495627.aspx
        string SasToken = connectionSaSUtil.getSaSToken(uri, 60);

        using (var httpClient = new HttpClient())
        {
            string json = JsonConvert.SerializeObject(deviceInstallation);

            httpClient.DefaultRequestHeaders.Add("Authorization", SasToken);

            var response = await httpClient.PutAsync(uri, new StringContent(json, System.Text.Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

    string installationId = null;
    var settings = ApplicationData.Current.LocalSettings.Values;

    // If we have not stored an installation ID in application data, create and store as application data.
    if (!settings.ContainsKey("__NHInstallationId"))
    {
        installationId = Guid.NewGuid().ToString();
        settings.Add("__NHInstallationId", installationId);
    }

    installationId = (string)settings["__NHInstallationId"];

    var deviceInstallation = new DeviceInstallation
    {
        installationId = installationId,
        platform = "wns",
        pushChannel = channel.Uri,
        //tags = tags.ToArray<string>()
    };

    var statusCode = await CreateOrUpdateInstallationAsync(deviceInstallation, 
                    "<HUBNAME>", "<SHARED LISTEN CONNECTION STRING>");

    if (statusCode != HttpStatusCode.Accepted)
    {
        var dialog = new MessageDialog(statusCode.ToString(), "Registration failed. Installation Id : " + installationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    else
    {
        var dialog = new MessageDialog("Registration successful using installation Id : " + installationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
}
```

### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-a-registration"></a>Пример кода регистрации в центре уведомлений с устройства с помощью регистрации

Эти методы создают или обновляют регистрацию устройства, на котором они вызываются. Это означает, что для обновления маркера или тегов необходимо перезаписать всю регистрацию. Помните, что регистрация является временным объектом, поэтому необходимо всегда иметь надежное хранилище с текущими тегами, нужными для конкретного устройства.

```csharp
// Initialize the Notification Hub
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(listenConnString, hubName);

// The Device ID from the PNS
var pushChannel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

// If you are registering from the client itself, then store this registration ID in device
// storage. Then when the app starts, you can check if a registration ID already exists or not before
// creating.
var settings = ApplicationData.Current.LocalSettings.Values;

// If we have not stored a registration ID in application data, store in application data.
if (!settings.ContainsKey("__NHRegistrationId"))
{
    // make sure there are no existing registrations for this push handle (used for iOS and Android)    
    string newRegistrationId = null;
    var registrations = await hub.GetRegistrationsByChannelAsync(pushChannel.Uri, 100);
    foreach (RegistrationDescription registration in registrations)
    {
        if (newRegistrationId == null)
        {
            newRegistrationId = registration.RegistrationId;
        }
        else
        {
            await hub.DeleteRegistrationAsync(registration);
        }
    }

    newRegistrationId = await hub.CreateRegistrationIdAsync();

    settings.Add("__NHRegistrationId", newRegistrationId);
}

string regId = (string)settings["__NHRegistrationId"];

RegistrationDescription registration = new WindowsRegistrationDescription(pushChannel.Uri);
registration.RegistrationId = regId;
registration.Tags = new HashSet<string>(YourTags);

try
{
    await hub.CreateOrUpdateRegistrationAsync(registration);
}
catch (Microsoft.WindowsAzure.Messaging.RegistrationGoneException e)
{
    settings.Remove("__NHRegistrationId");
}
```

## <a name="registration-management-from-a-backend"></a>Управление регистрацией из серверной части

Для управления регистрацией из серверной части требуется дополнительный код. Приложение с устройства должно предоставлять обновленный маркер PNS серверной части при каждом запуске приложения (а также теги и шаблоны), а серверная часть должна обновлять этот маркер в центре уведомлений. На следующем рисунке показана схема работы.

![Управление регистрацией](./media/notification-hubs-registration-management/notification-hubs-registering-on-backend.png)

Преимуществом управления регистрациями из серверной части является возможность изменения тегов для регистраций даже в том случае, когда соответствующее приложение на устройстве неактивно, а также возможность проверять подлинность клиентского приложения перед добавлением тега в его регистрацию.

### <a name="example-code-to-register-with-a-notification-hub-from-a-backend-using-an-installation"></a>Пример кода регистрации в центре уведомлений из серверной части с помощью установки

Клиентское устройство по-прежнему получает обработчик PNS и соответствующие свойства установки, как и прежде, и вызывает настраиваемый API на серверной части, который может выполнять регистрацию и авторизацию тегов и т. д. Серверная часть может использовать [пакет SDK концентратора уведомлений для внутренних операций](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

Для обновления установки также можно использовать метод PATCH из [стандарта JSON-Patch](https://tools.ietf.org/html/rfc6902) .

```csharp
// Initialize the Notification Hub
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(listenConnString, hubName);

// Custom API on the backend
public async Task<HttpResponseMessage> Put(DeviceInstallation deviceUpdate)
{

    Installation installation = new Installation();
    installation.InstallationId = deviceUpdate.InstallationId;
    installation.PushChannel = deviceUpdate.Handle;
    installation.Tags = deviceUpdate.Tags;

    switch (deviceUpdate.Platform)
    {
        case "mpns":
            installation.Platform = NotificationPlatform.Mpns;
            break;
        case "wns":
            installation.Platform = NotificationPlatform.Wns;
            break;
        case "apns":
            installation.Platform = NotificationPlatform.Apns;
            break;
        case "fcm":
            installation.Platform = NotificationPlatform.Fcm;
            break;
        default:
            throw new HttpResponseException(HttpStatusCode.BadRequest);
    }


    // In the backend we can control if a user is allowed to add tags
    //installation.Tags = new List<string>(deviceUpdate.Tags);
    //installation.Tags.Add("username:" + username);

    await hub.CreateOrUpdateInstallationAsync(installation);

    return Request.CreateResponse(HttpStatusCode.OK);
}
```

### <a name="example-code-to-register-with-a-notification-hub-from-a-device-using-a-registration-id"></a>Пример кода регистрации в центре уведомлений с устройства, использующего идентификатор регистрации

Из серверной части приложения с регистрациями можно выполнять основные операции CRUDS. Пример:

```csharp
var hub = NotificationHubClient.CreateClientFromConnectionString("{connectionString}", "hubName");

// create a registration description object of the correct type, e.g.
var reg = new WindowsRegistrationDescription(channelUri, tags);

// Create
await hub.CreateRegistrationAsync(reg);

// Get by ID
var r = await hub.GetRegistrationAsync<RegistrationDescription>("id");

// update
r.Tags.Add("myTag");

// update on hub
await hub.UpdateRegistrationAsync(r);

// delete
await hub.DeleteRegistrationAsync(r);
```

Серверная часть должна обеспечивать параллельность обновлений регистраций. Служебная шина предоставляет управление оптимистичным параллелизмом для управления регистрациями. На уровне HTTP это реализуется посредством использования ETag в операциях управления регистрацией. Эта функция автоматически используется в пакетах Microsoft SDK, которые выдают исключение, если обновление отклоняется по причинам параллелизма. Серверная часть приложения отвечает за обработку этих исключений и перезапуск обновления в случае необходимости.
