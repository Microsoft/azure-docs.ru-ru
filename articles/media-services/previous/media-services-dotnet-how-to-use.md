---
title: Как настроить компьютер для разработки служб мультимедиа с помощью .NET
description: Сведения о требованиях для разработки служб мультимедиа с помощью пакета SDK служб мультимедиа для .NET. Сведения о создании приложений Visual Studio.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ec2804c7-c656-4fbf-b3e4-3f0f78599a7f
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: a2a9ecbcafa89b4981fe403d116ac0288e60b6b6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103013896"
---
# <a name="media-services-development-with-net"></a>Разработка служб мультимедиа с помощью .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

В этой статье описано, как приступить к разработке приложений Служб мультимедиа с помощью .NET.

Библиотека **пакета SDK служб мультимедиа Azure для .NET** позволяет программировать для служб мультимедиа с помощью .NET. Чтобы упростить разработку с помощью .NET, предоставляется библиотека **расширений пакета SDK служб мультимедиа Azure для .NET** . Эта библиотека содержит набор методов расширения и вспомогательные функции, упрощающие код .NET. Обе библиотеки доступны в **NuGet** и на **GitHub**.

## <a name="prerequisites"></a>Предварительные требования
* Учетная запись служб мультимедиа в новой или существующей подписке Azure. См. дополнительные сведения о [создании учетной записи Служб мультимедиа Azure](media-services-portal-create-account.md).
* Операционные системы: Windows 10, Windows 7, Windows 2008 R2 или Windows 8.
* .NET Framework 4.5 или более поздней версии.
* приведенному.

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio
В этом разделе показано, как создать проект в Visual Studio и настроить его для разработки с использованием служб мультимедиа.  Хотя в нашем примере проект — это консольное приложение Windows на C#, те же действия для установки применяются для других типов проектов, которые можно создать для приложений служб мультимедиа (например, приложения Windows Forms или веб-приложения ASP.NET).

В этом разделе показано, как с помощью **NuGet** добавить расширения пакета SDK служб мультимедиа для .NET или другие зависимые библиотеки.

Кроме того, вы можете получить на GitHub актуальный код для пакета SDK служб мультимедиа для .NET ([github.com/Azure/azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services) и [github.com/Azure/azure-sdk-for-media-services-extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions)), создать решение и добавить ссылки на клиентский проект. Все необходимые зависимости скачиваются и извлекаются автоматически.

1. Создайте в Visual Studio консольное приложение C#. Введите значения в поля **Имя**, **Расположение** и **Имя решения**, а затем нажмите кнопку "ОК".
2. Создайте решение.
3. Используйте пакет **NuGet**, чтобы установить и добавить **расширения пакета SDK служб мультимедиа Azure для .NET** (**windowsazure.mediaservices.extensions**). При установке этого пакета также устанавливается **пакет SDK служб мультимедиа для .NET** и добавляются все остальные необходимые зависимости.
   
    Убедитесь, что у вас установлена новейшая версия NuGet. Дополнительную информацию и инструкции по установке см. на сайте [NuGet](https://nuget.codeplex.com/).

    1. В обозревателе решений щелкните правой кнопкой мыши имя проекта и выберите **Управление пакетами NuGet**.

    2. Откроется диалоговое окно Управление пакетами NuGet.

    3. В интерактивной коллекции найдите расширения служб мультимедиа Azure, выберите **расширения пакета SDK служб мультимедиа Azure для .NET** (**windowsazure.mediaservices.extensions**) и нажмите кнопку **Установить**.
   
    4. Проект будет изменен, и в него будут добавлены ссылки на расширения пакета SDK служб мультимедиа для .NET, пакет SDK служб мультимедиа для .NET и другие зависимые сборки.
4. Чтобы сделать среду разработки чище, рекомендуем включить восстановление пакета NuGet. Дополнительную информацию см. в статье [Восстановление пакета NuGet](https://docs.nuget.org/consume/package-restore).
5. Добавьте ссылку на сборку **System.Configuration** . Эта сборка содержит класс **System.Configuration.ConfigurationManager** , используемый для доступа к файлам конфигурации (например, App.config).
   
    1. Чтобы добавить ссылки в диалоговом окне управления ссылками, в обозревателе решений щелкните имя проекта правой кнопкой мыши. Щелкните **Добавить**, а затем — **Ссылка...**.
   
    2. Откроется диалоговое окно "Управление ссылками".
    3. В списке сборок платформы .NET найдите и выберите сборку System.Configuration, а затем нажмите кнопку **ОК**.
6. Откройте файл App.config и добавьте в него раздел **appSettings**. Укажите необходимые значения для подключения к API служб мультимедиа. Дополнительные сведения см. в статье [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

    Набор значений, которые требуются для подключения с использованием аутентификации с помощью **субъекта-службы**.

    ```xml
    <configuration>
    ...
        <appSettings>
            <add key="AMSAADTenantDomain" value="tenant"/>
            <add key="AMSRESTAPIEndpoint" value="endpoint"/>
            <add key="AMSClientId" value="id"/>
            <add key="AMSClientSecret" value="secret"/>
        </appSettings>
    </configuration>
    ```

7. Добавьте в проект ссылку на сборку **System.Configuration**.
8. Замените существующие инструкции **using** в начале файла Program.cs следующим кодом:

    ```csharp      
    using System;
    using System.Configuration;
    using System.IO;
    using Microsoft.WindowsAzure.MediaServices.Client;
    using System.Threading;
    using System.Collections.Generic;
    using System.Linq;
    ```

    На этом этапе вы готовы начать разработку приложения служб мультимедиа.    

## <a name="example"></a>Пример

Ниже приведен пример небольшого приложения, которое подключается к API AMS и выводит все доступные обработчики мультимедиа.

```csharp
class Program
{
    // Read values from the App.config file.

    private static readonly string _AADTenantDomain =
        ConfigurationManager.AppSettings["AMSAADTenantDomain"];
    private static readonly string _RESTAPIEndpoint =
        ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
    private static readonly string _AMSClientId =
        ConfigurationManager.AppSettings["AMSClientId"];
    private static readonly string _AMSClientSecret =
        ConfigurationManager.AppSettings["AMSClientSecret"];
        
    private static CloudMediaContext _context = null;
    static void Main(string[] args)
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
        
        // List all available Media Processors
        foreach (var mp in _context.MediaProcessors)
        {
            Console.WriteLine(mp.Name);
        }
        
    }
 ```

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы готовы [подключиться к API AMS](media-services-use-aad-auth-to-access-ams-api.md) и [начать разработку](media-services-dotnet-get-started.md).


## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
