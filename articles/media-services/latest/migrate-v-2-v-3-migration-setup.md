---
title: Установка миграции служб мультимедиа версии 2 – V3
description: Эта статья поможет вам настроить среду для миграции из служб мультимедиа Azure версии 2 в v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
editor: ''
tags: ''
keywords: службы мультимедиа Azure, миграция, Потоковая трансляция, динамическая, пакет SDK
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 1/14/2021
ms.author: inhenkel
ms.openlocfilehash: 792f4f81d84b61f0f78c53e62168921221b4f330
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102503170"
---
# <a name="step-3---set-up-to-migrate-to-the-v3-rest-api-or-client-sdk"></a>Шаг 3. Настройка миграции на версии 3 REST API или клиентский пакет SDK

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![шаги миграции 2](./media/migration-guide/steps-3.svg)

Ниже описаны действия, которые необходимо выполнить, чтобы настроить среду для использования API-интерфейса служб мультимедиа v3.

## <a name="sdk-model"></a>Модель SDK

В API v2 существовало два разных клиентских пакета SDK: один для API управления, который позволял программное создание учетных записей и один для управления ресурсами.

Ранее разработчики работали с ИДЕНТИФИКАТОРом клиента субъекта-службы Azure и секретом клиента, а также с определенной конечной точкой v2 REST API для своей учетной записи AMS.

API V3 — на основе Azure Resource Management (ARM). Он использует идентификаторы и ключи субъекта-службы Azure Active Directory (Azure AD) для подключения к API. Для подключения к API разработчики должны будут создавать субъекты-службы или управляемые удостоверения. В API V3 API использует стандартные конечные точки ARM, использует аналогичные и последовательные модели для всех остальных служб Azure.

Клиенты, которые ранее использовали версию 2015-10-01 API управления ARM для управления учетными записями версии 2, должны использовать версию 2020-05-01 API управления ARM, поддерживаемую для доступа к API V3.

## <a name="create-a-new-media-services-account-for-testing"></a>Создать новую учетную запись служб мультимедиа для тестирования

Следуйте инструкциям краткого руководства по [настройке среды](how-to-set-azure-subscription.md?tabs=portal) с помощью портал Azure. Выберите доступ к API и проверка подлинности субъекта-службы, чтобы создать новый идентификатор приложения Azure AD и секреты для использования с этой тестовой учетной записью.

[Создайте учетную запись служб мультимедиа](create-account-howto.md?tabs=portal).
[Получите учетные данные для доступа к API служб мультимедиа](access-api-howto.md?tabs=portal).

## <a name="download-client-sdk-of-your-choice-and-set-up-your-environment"></a>Скачайте клиентский пакет SDK и настройте среду.

- Пакеты SDK доступны [для .NET](/dotnet/api/overview/azure/mediaservices/management), .NET Core, [Node.js](/javascript/api/overview/azure/mediaservices/management), [Python](/python/api/overview/azure/mediaservices/management), [Java](/java/api/overview/azure/mediaservices/management), [Go](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/mediaservices/mgmt/2018-07-01/media)и [Ruby](https://github.com/Azure/azure-sdk-for-ruby/blob/master/README.md).
- [Azure CLI](/cli/azure/ams)   интеграция для простой поддержки сценариев.

> [!NOTE]
> Пакет SDK для PHP сообщества больше не доступен для служб мультимедиа Azure на v3. Если вы используете PHP в версии 2, необходимо выполнить миграцию в REST API непосредственно в коде.

## <a name="open-api-specifications"></a>Спецификации Open API

- Версия 3 основана на унифицированной поверхности API, которая предоставляет функциональные возможности управления и операций, основанные на Azure Resource Manager. Шаблоны Azure Resource Manager можно использовать для создания и развертывания преобразований, конечных точек потоковой передачи, динамических событий и многого другого.

- В [спецификации OpenAPI](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2020-05-01) (прежнее название — Swagger) объясняется схема всех компонентов службы.

- Все клиентские пакеты SDK являются производными и формируются из спецификации Open API, опубликованной на GitHub. На момент публикации этой статьи последние открытые спецификации API хранятся в GitHub. Версия 2020-05-01 является [последним стабильным выпуском](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2020-05-01).

## <a name="rest"></a>[REST](#tab/rest)

Используйте [POST](./media-rest-apis-with-postman.md) для вызовов служб мультимедиа v3 REST API.
Ознакомьтесь со [страницами со ссылками на REST API](/rest/api/media/).

Следует использовать строку версии 2020-05-01 в коллекции POST.

## <a name="net"></a>[.NET](#tab/net)

Дополнительные сведения о настройке среды см. в статье [Подключение к API служб мультимедиа v3 с помощью .NET](configure-connect-dotnet-howto.md) .

Если вы просто хотите установить последнюю версию пакета SDK с помощью PackageManager, используйте следующую команду:

```Install-Package Microsoft.Azure.Management.Media```

Чтобы установить последнюю версию пакета SDK с помощью .NET CLI, используйте следующую команду:

```dotnet add package Microsoft.Azure.Management.Media```

Кроме того, полные примеры .NET доступны в [Azure-Samples/Media-Services-v3-DotNet](https://github.com/Azure-Samples/media-services-v3-dotnet) для различных сценариев. Проекты в этом репозитории показывают, как реализовать различные сценарии служб мультимедиа Azure с помощью версии v3.

### <a name="get-started-adjusting-your-code"></a>Приступая к настройке кода

Найдите в базе кода экземпляры `CloudMediaContext` использования, чтобы начать процесс обновления до версии API V3.

В следующем коде показано, как ранее был получен доступ к API v2 с помощью пакета SDK для .NET версии 2. Разработчики начинают с создания `CloudMediaContext` и создания экземпляра с `AzureAdTokenCredentials` объектом.

```dotnet

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
        try
        {
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

```

## <a name="java"></a>[Java](#tab/java)

Прочтите статью [Подключение к API служб мультимедиа v3 с помощью Java](configure-connect-java-howto.md) , чтобы настроить среду.

## <a name="python"></a>[Python](#tab/python)

Чтобы настроить среду, ознакомьтесь со статьей [Подключение к службам мультимедиа Azure v3 API-Python](configure-connect-python-howto.md) .

## <a name="nodejs"></a>[Node.js](#tab/nodejs)

Дополнительные сведения о настройке среды см. в статье [Подключение к API-интерфейсу служб мультимедиа Azure v3 Node.js](configure-connect-nodejs-howto.md) .

## <a name="ruby"></a>[Ruby](#tab/ruby)

- Получите пакет [Ruby](https://github.com/Azure/azure-sdk-for-ruby/blob/master/README.md) SDK на GitHub.

## <a name="go"></a>[GO](#tab/go)

Скачайте пакет SDK для [Go](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/mediaservices/mgmt/2018-07-01/media) .

---

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [migration guide next steps](./includes/migration-guide-next-steps.md)]