---
title: Использование аутентификации Azure AD для доступа к API служб мультимедиа Azure с помощью .NET | Документация Майкрософт
description: В этом разделе показано, как использовать аутентификацию Azure Active Directory (Azure AD) для доступа к API служб мультимедиа Azure (AMS) с помощью .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: fd19bce05d18eb8f9b8fd3806082caeebe98bdad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103013998"
---
# <a name="use-azure-ad-authentication-to-access-azure-media-services-api-with-net"></a>Использование аутентификации Azure AD для доступа к API служб мультимедиа Azure с помощью .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Начиная с windowsazure.mediaservices 4.0.0.4, службы мультимедиа Azure поддерживают аутентификацию на основе Azure Active Directory (Azure AD). В этом разделе показано, как использовать аутентификацию Azure AD для доступа к API служб мультимедиа Azure (AMS) с помощью Microsoft .NET.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure. Дополнительные сведения см. на странице с [бесплатной пробной версией Azure](https://azure.microsoft.com/pricing/free-trial/).
- Учетная запись служб мультимедиа. Дополнительные сведения см. в статье [Создание учетной записи служб мультимедиа Azure с помощью портала Azure](media-services-portal-create-account.md).
- Последняя версия пакета [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices).
- Изучение статьи [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Процесс аутентификации Azure AD в службах мультимедиа Azure можно выполнить двумя способами.

- **Аутентификация пользователя** позволяет проверить подлинность пользователя, который использует приложение для взаимодействия с ресурсами служб мультимедиа Azure. Интерактивное приложение должно сначала запросить у пользователя учетные данные. Примером может послужить приложение консоли управления, которое используется авторизованными пользователями для мониторинга заданий кодирования или потоковой трансляции.
- **Аутентификация субъекта-службы** позволяет проверить подлинность службы. Этот метод аутентификации обычно используют приложения, которые выполняют службы управляющей программы, службы среднего уровня или запланированные задания, например веб-приложения, приложения-функции, приложения логики, интерфейсы API или микрослужбы.

>[!IMPORTANT]
>Службы мультимедиа Azure в настоящее время поддерживают модель аутентификации с помощью службы контроля доступа Azure. Тем не менее авторизация с помощью службы контроля доступа будет считаться нерекомендуемой с 22 июня 2018 г. Мы рекомендуем как можно быстрее перейти на использование модели аутентификации Azure Active Directory.

## <a name="get-an-azure-ad-access-token"></a>Получение токена доступа Azure AD

Чтобы подключиться к API служб мультимедиа Azure, используя аутентификацию Azure AD, клиентскому приложению необходимо запросить маркер доступа Azure AD. При использовании клиентского пакета SDK служб мультимедиа для .NET многие сведения о том, как получить маркер доступа Azure AD, упакованы в упрощенном виде в классы [AzureAdTokenProvider](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenProvider.cs) и [AzureAdTokenCredentials](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenCredentials.cs).

Например, не требуется указывать центр Azure AD, универсальный код ресурса (URI) для ресурса служб мультимедиа или сведения о собственном приложении Azure AD. Это хорошо известные значения, которые уже настроены с помощью класса поставщика маркеров доступа Azure AD.

Если вы не используете пакет SDK служб мультимедиа Azure для .NET, мы рекомендуем использовать [библиотеки аутентификации Azure AD](../../active-directory/azuread-dev/active-directory-authentication-libraries.md). Чтобы получить значения параметров, необходимых для использования библиотеки аутентификации Azure AD, ознакомьтесь с разделом [Приступая к работе с аутентификацией Azure AD с помощью портала Azure](media-services-portal-get-started-with-aad.md).

Кроме того, вы можете заменить реализацию по умолчанию **AzureAdTokenProvider** собственной реализацией.

## <a name="install-and-configure-azure-media-services-net-sdk"></a>Установка и настройка пакета SDK служб мультимедиа для .NET.

>[!NOTE]
>Для использования аутентификации Azure AD с помощью пакета SDK служб мультимедиа для .NET необходима последняя версия пакета [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices). Кроме того, добавьте ссылку на сборку **Microsoft.IdentityModel.Clients.ActiveDirectory**. При использовании существующего приложения следует добавить сборку **Microsoft.WindowsAzure.MediaServices.Client.Common.Authentication.dll**.

1. Создайте в Visual Studio новое консольное приложение C#.
2. Используйте пакет Nuget [windowsazure.mediaservices](https://www.nuget.org/packages/windowsazure.mediaservices), чтобы установить **пакет SDK служб мультимедиа Azure для .NET**.

    Чтобы добавить ссылки с помощью пакета NuGet, сделайте следующее: в **обозревателе решений** щелкните правой кнопкой мыши имя проекта и выберите **Управление пакетами NuGet**. Затем найдите **windowsazure.mediaservices** и щелкните **Установить**.

    -или-

    В **консоли диспетчера пакетов** Visual Studio выполните приведенную ниже команду.

    ```console
    Install-Package windowsazure.mediaservices -Version 4.0.0.4
    ```

3. Добавьте инструкцию **using** в исходный код.

    ```csharp
    using Microsoft.WindowsAzure.MediaServices.Client;
    ```

## <a name="use-user-authentication"></a>Использование аутентификации пользователя

Для подключения к API служб мультимедиа Azure с помощью аутентификации пользователя клиентскому приложению требуется запросить маркер Azure AD со следующими параметрами:

- Конечная точка клиента Azure AD. Информацию о клиенте можно получить на портале Azure. Наведите указатель мыши на пользователя, выполнившего вход, в правом верхнем углу.
- Универсальный код ресурса (URI) для ресурса служб мультимедиа.
- Идентификатор клиента (собственного) приложения служб мультимедиа.
- Универсальный код ресурса (URI) перенаправления (собственного) приложения служб мультимедиа.

Значения этих параметров можно найти в **AzureEnvironments.AzureCloudEnvironment**. Константа **AzureEnvironments.AzureCloudEnvironment** является константой вспомогательного класса в пакете SDK для .NET, которая используется для получения правильных параметров переменных среды для общедоступного центра обработки данных Azure.

Она содержит предопределенные параметры среды для доступа к службам мультимедиа только в общедоступных центрах обработки данных. Для независимых облачных регионов или облачных регионов для государственных организаций можно использовать **AzureChinaCloudEnvironment**, **AzureUsGovernmentEnvironment** или **AzureGermanCloudEnvironment** соответственно.

Приведенный ниже пример кода создает маркер.

```csharp
var tokenCredentials = new AzureAdTokenCredentials("microsoft.onmicrosoft.com", AzureEnvironments.AzureCloudEnvironment);
var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
```

Чтобы начать программирование для служб мультимедиа, необходимо создать экземпляр **CloudMediaContext**, представляющий контекст сервера. **CloudMediaContext** содержит ссылки на важные коллекции, в том числе на задания, ресурсы, файлы, политики доступа и указатели.

Необходимо также передать **универсальный код ресурса (URI) для ресурса REST служб мультимедиа** в конструктор **CloudMediaContext**. Чтобы получить универсальный код ресурса (URI) для ресурса REST служб мультимедиа, войдите на портал Azure, выберите учетную запись служб мультимедиа Azure, выберите **Доступ к API**, а затем щелкните **Подключение к API служб мультимедиа Azure с помощью проверки подлинности пользователя**.

Следующий пример кода создает экземпляр **CloudMediaContext**.

```csharp
CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
```

В следующем примере показано, как создать маркер Azure AD и контекст.

```csharp
namespace AzureADAuthSample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Specify your Azure AD tenant domain, for example "microsoft.onmicrosoft.com".
            var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}", AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
            CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

            var assets = context.Assets;
            foreach (var a in assets)
            {
                Console.WriteLine(a.Name);
            }
        }
    }
}
```

>[!NOTE]
>Если возникнет исключение "Удаленный сервер возвратил ошибку: 401 - Не санкционировано", ознакомьтесь с разделом [Управление доступом](media-services-use-aad-auth-to-access-ams-api.md#access-control) обзора доступа к API служб мультимедиа Azure с помощью аутентификации Azure AD.

## <a name="use-service-principal-authentication"></a>Использование аутентификации субъекта-службы

Для подключения к API служб мультимедиа Azure в режиме аутентификации субъекта-службы приложению среднего уровня (веб-API или веб-приложению) требуется запросить маркер Azure AD со следующими параметрами:

- Конечная точка клиента Azure AD. Информацию о клиенте можно получить на портале Azure. Наведите указатель мыши на пользователя, выполнившего вход, в правом верхнем углу.
- Универсальный код ресурса (URI) для ресурса служб мультимедиа.
- Значения приложения Azure AD: **идентификатор клиента** и **секрет клиента**.

Значения **идентификатора клиента** и **секрета клиента** можно найти на портале Azure. Дополнительные сведения см. в разделе [Приступая к работе с аутентификацией Azure AD с помощью портала Azure](media-services-portal-get-started-with-aad.md).

Следующий пример кода создает маркер с помощью конструктора **AzureAdTokenCredentials**, принимающего **AzureAdClientSymmetricKey** как параметр.

```csharp
var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                        new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"),
                        AzureEnvironments.AzureCloudEnvironment);

var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
```

Можно также указать конструктор **AzureAdTokenCredentials**, принимающий **AzureAdClientCertificate** как параметр.

Инструкции по созданию и настройке сертификата в форме, которая может использоваться в Azure AD, см. в разделе [Authenticating to Azure AD in daemon apps with certificates - manual configuration steps](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) (Ручная настройка аутентификации приложений управляющей программы в Azure AD с помощью сертификатов).

```csharp
var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                        new AzureAdClientCertificate("{YOUR CLIENT ID HERE}", "{YOUR CLIENT CERTIFICATE THUMBPRINT}"),
                        AzureEnvironments.AzureCloudEnvironment);
```

Чтобы начать программирование для служб мультимедиа, необходимо создать экземпляр **CloudMediaContext**, представляющий контекст сервера. Необходимо также передать **универсальный код ресурса (URI) для ресурса REST служб мультимедиа** в конструктор **CloudMediaContext**. Значение **универсального кода ресурса (URI) для ресурса REST служб мультимедиа** можно получить на портале Azure.

Следующий пример кода создает экземпляр **CloudMediaContext**.

```csharp
CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
```

В следующем примере показано, как создать маркер Azure AD и контекст.

```csharp
namespace AzureADAuthSample
{
    class Program
    {
        static void Main(string[] args)
        {
            var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                                        new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"),
                                        AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
            CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

            var assets = context.Assets;
            foreach (var a in assets)
            {
                Console.WriteLine(a.Name);
            }

            Console.ReadLine();
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Приступите к [передаче файлов в учетную запись](media-services-dotnet-upload-files.md).
