---
title: Подключение к API служб мультимедиа Azure v3 — .NET
description: В этой статье показано, как подключиться к API служб мультимедиа v3 с помощью .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/17/2020
ms.author: inhenkel
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: 8946f6e94dd26db45622bc7609fb2375d59bb57e
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102455388"
---
# <a name="connect-to-media-services-v3-api---net"></a>Подключение к API служб мультимедиа v3 — .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье показано, как подключиться к пакету SDK .NET для служб мультимедиа Azure v3 с помощью метода входа субъекта-службы.

## <a name="prerequisites"></a>Предварительные требования

- [Создание учетной записи Служб мультимедиа](./create-account-howto.md). Обязательно запомните имя группы ресурсов и имя учетной записи служб мультимедиа.
- Установите инструмент, который вы хотите использовать для разработки .NET. Действия, описанные в этой статье, показывают, как использовать [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Вы можете использовать Visual Studio Code, см. раздел [Работа с C#](https://code.visualstudio.com/docs/languages/csharp). Или можно использовать другой редактор кода.

> [!IMPORTANT]
> Проверьте [соглашения об именовании](media-services-apis-overview.md#naming-conventions).

## <a name="create-a-console-application"></a>Создание консольного приложение

1. Запустите среду Visual Studio. 
1. В меню **файл** выберите пункт **создать**  >  **проект**. 
1. Создайте консольное приложение **.NET Core** .

Пример приложения в этом разделе: targets `netcoreapp2.0` . В коде используется "Async Main", который доступен начиная с C# 7,1. Дополнительные сведения см. в этом [блоге](/archive/blogs/benwilli/async-main-is-available-but-hidden) .

## <a name="add-required-nuget-packagesassemblies"></a>Добавление необходимых пакетов или сборок NuGet

1. В Visual Studio выберите **инструменты**  >  **Диспетчер пакетов NuGet**  >  **консоль диспетчера NuGet**.
2. В окне **консоли диспетчера пакетов** используйте команду, `Install-Package` чтобы добавить следующие пакеты NuGet. Например, `Install-Package Microsoft.Azure.Management.Media`.

|Пакет|Описание|
|---|---|
|`Microsoft.Azure.Management.Media`|Пакет SDK служб мультимедиа Azure. <br/>Чтобы убедиться, что вы используете последний пакет служб мультимедиа Azure, проверьте [Microsoft. Azure. Management. Media](https://www.nuget.org/packages/Microsoft.Azure.Management.Media).|

### <a name="other-required-assemblies"></a>Другие необходимые сборки

- Azure. Storage. BLOB
- Microsoft.Extensions.Configuration
- Microsoft.Extensions.Configфигурации. EnvironmentVariables
- Microsoft.Extensions.Configuration.Json
- Microsoft.Rest.ClientRuntime.Azure.Authentication

## <a name="create-and-configure-the-app-settings-file"></a>Создание и Настройка файла параметров приложения

### <a name="create-appsettingsjson"></a>Создание appsettings.js

1. Go Go **General**  >  **Text File**.
1. Присвойте ему имя "appsettings.jsв".
1. Установите свойство "Копировать в выходной каталог" JSON-файла в значение "Копировать при более поздней версии" (чтобы приложение могло получить доступ к нему при публикации).

### <a name="set-values-in-appsettingsjson"></a>Установка значений в appsettings.jsв

Выполните `az ams account sp create` команду, как описано в разделе [API доступа](./access-api-howto.md). Команда возвращает JSON, который необходимо скопировать в "appsettings.js".
 
## <a name="add-configuration-file"></a>Добавление файла конфигурации

Для удобства добавьте файл конфигурации, который отвечает за чтение значений из "appsettings.jsна".

1. Добавьте в проект новый класс CS. Присвойте обработчику события имя `ConfigWrapper`. 
1. Вставьте в этот файл следующий код (в этом примере предполагается, что у вас есть пространство имен `ConsoleApp1` ).

```csharp
using System;

using Microsoft.Extensions.Configuration;

namespace ConsoleApp1
{
    public class ConfigWrapper
    {
        private readonly IConfiguration _config;

        public ConfigWrapper(IConfiguration config)
        {
            _config = config;
        }

        public string SubscriptionId
        {
            get { return _config["SubscriptionId"]; }
        }

        public string ResourceGroup
        {
            get { return _config["ResourceGroup"]; }
        }

        public string AccountName
        {
            get { return _config["AccountName"]; }
        }

        public string AadTenantId
        {
            get { return _config["AadTenantId"]; }
        }

        public string AadClientId
        {
            get { return _config["AadClientId"]; }
        }

        public string AadSecret
        {
            get { return _config["AadSecret"]; }
        }

        public Uri ArmAadAudience
        {
            get { return new Uri(_config["ArmAadAudience"]); }
        }

        public Uri AadEndpoint
        {
            get { return new Uri(_config["AadEndpoint"]); }
        }

        public Uri ArmEndpoint
        {
            get { return new Uri(_config["ArmEndpoint"]); }
        }

        public string Location
        {
            get { return _config["Location"]; }
        }
    }
}
```

## <a name="connect-to-the-net-client"></a>Подключение к клиенту .NET

Чтобы начать использование API Служб мультимедиа с .NET, создайте объект **AzureMediaServicesClient**. Чтобы создать объект, введите учетные данные, необходимые клиенту для подключения к Azure с помощью Azure AD. В приведенном ниже коде функция Жеткредентиалсасинк создает объект Сервицеклиенткредентиалс на основе учетных данных, указанных в локальном файле конфигурации.

1. Откройте `Program.cs`.
1. Вставьте следующий код:

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;

using Microsoft.Azure.Management.Media;
using Microsoft.Azure.Management.Media.Models;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;

namespace ConsoleApp1
{
    class Program
    {
        public static async Task Main(string[] args)
        {
            
            ConfigWrapper config = new ConfigWrapper(new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddEnvironmentVariables()
                .Build());

            try
            {
                IAzureMediaServicesClient client = await CreateMediaServicesClientAsync(config);
                Console.WriteLine("connected");
            }
            catch (Exception exception)
            {
                if (exception.Source.Contains("ActiveDirectory"))
                {
                    Console.Error.WriteLine("TIP: Make sure that you have filled out the appsettings.json file before running this sample.");
                }

                Console.Error.WriteLine($"{exception.Message}");

                ApiErrorException apiException = exception.GetBaseException() as ApiErrorException;
                if (apiException != null)
                {
                    Console.Error.WriteLine(
                        $"ERROR: API call failed with error code '{apiException.Body.Error.Code}' and message '{apiException.Body.Error.Message}'.");
                }
            }

            Console.WriteLine("Press Enter to continue.");
            Console.ReadLine();
        }
 
        private static async Task<ServiceClientCredentials> GetCredentialsAsync(ConfigWrapper config)
        {
            // Use ApplicationTokenProvider.LoginSilentWithCertificateAsync or UserTokenProvider.LoginSilentAsync to get a token using service principal with certificate
            //// ClientAssertionCertificate
            //// ApplicationTokenProvider.LoginSilentWithCertificateAsync

            // Use ApplicationTokenProvider.LoginSilentAsync to get a token using a service principal with symmetric key
            ClientCredential clientCredential = new ClientCredential(config.AadClientId, config.AadSecret);
            return await ApplicationTokenProvider.LoginSilentAsync(config.AadTenantId, clientCredential, ActiveDirectoryServiceSettings.Azure);
        }

        private static async Task<IAzureMediaServicesClient> CreateMediaServicesClientAsync(ConfigWrapper config)
        {
            var credentials = await GetCredentialsAsync(config);

            return new AzureMediaServicesClient(config.ArmEndpoint, credentials)
            {
                SubscriptionId = config.SubscriptionId,
            };
        }

    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Учебник. Отправка, кодирование и потоковая передача видео — .NET](stream-files-tutorial-with-api.md) 
- [Руководство. потоковая передача в реальном времени с помощью служб мультимедиа v3 — .NET](stream-live-tutorial-with-api.md)
- [Учебник. Анализ видео с помощью служб мультимедиа v3 — .NET](analyze-videos-tutorial-with-api.md)
- [Создание входных данных задания из локального файла с помощью .NET](job-input-from-local-file-how-to.md)
- [Создание входных данных задания из URL-адреса HTTPS с помощью .NET](job-input-from-http-how-to.md)
- [Кодирование с помощью пользовательского преобразования с помощью .NET](customize-encoder-presets-how-to.md)
- [Использование динамического шифрования AES-128 и службы доставки ключей с помощью .NET](protect-with-aes128.md)
- [Использование динамического шифрования DRM и службы доставки лицензий с помощью .NET](protect-with-drm.md)
- [Получение ключа подписи из имеющейся политики с помощью .NET](get-content-key-policy-dotnet-howto.md)
- [Создание фильтров с помощью Служб мультимедиа для .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Дополнительные примеры видео по запросу с использованием Функций Azure версии 2 и Cлужб мультимедиа версии 3](https://aka.ms/ams3functions)

## <a name="see-also"></a>См. также раздел

* [Справочник по .NET](/dotnet/api/overview/azure/mediaservices/management)
* Дополнительные примеры кода см. в репозитории [примеров пакета SDK для .NET](https://github.com/Azure-Samples/media-services-v3-dotnet) .
