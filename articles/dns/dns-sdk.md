---
title: Создание зон и наборов записей DNS с помощью пакета SDK для .NET
titleSuffix: Azure DNS
description: В этой схеме обучения приступайте к созданию зон DNS и наборов записей в Azure DNS с помощью пакета SDK для .NET.
services: dns
documentationcenter: na
author: rohinkoul
manager: kumudD
ms.assetid: eed99b87-f4d4-4fbf-a926-263f7e30b884
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2016
ms.author: rohink
ms.custom: devx-track-csharp
ms.openlocfilehash: 8e116096afbd01af4914be49d5675881724d5069
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96015066"
---
# <a name="create-dns-zones-and-record-sets-using-the-net-sdk"></a>Создание зон и наборов записей DNS с помощью пакета SDK для .NET

Вы можете автоматизировать операции создания, удаления или обновления зон, наборов записей и записей DNS, используя пакет SDK для DNS и библиотеку управления DNS для .NET. Полный проект Visual Studio доступен [здесь](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True).

## <a name="create-a-service-principal-account"></a>Создание учетной записи субъекта-службы

Как правило, программный доступ к ресурсам Azure предоставляется через выделенную учетную запись, а не через учетные данные пользователя. Эти выделенные учетные записи называются учетными записями субъектов-служб. Чтобы использовать пример проекта SDK для Azure DNS, сначала необходимо создать учетную запись субъекта-службы и назначить ей соответствующие разрешения.

1. Создайте учетную запись субъекта-службы, следуя [этим инструкциям](../active-directory/develop/howto-authenticate-service-principal-powershell.md) (в примере проекта SDK для Azure DNS предполагается использование проверки подлинности на основе пароля).
2. Создайте группу ресурсов (см. сведения [здесь](../azure-resource-manager/templates/deploy-portal.md)).
3. Предоставьте разрешения участника зоны DNS учетной записи субъекта-службы для группы ресурсов с помощью Azure RBAC (см. сведения [здесь](../role-based-access-control/role-assignments-portal.md)).
4. При использовании примера проекта SDK для Azure DNS измените файл program.cs следующим образом:

   * Задайте правильные значения для параметров `tenantId`, `clientId` (также известный как идентификатор учетной записи), `secret` (пароль учетной записи субъекта-службы) и `subscriptionId`, как описано на шаге 1.
   * Введите имя группы ресурсов, выбранное на шаге 2.
   * Введите имя зоны DNS по своему выбору.

## <a name="nuget-packages-and-namespace-declarations"></a>Пакеты NuGet и объявления пространств имен

Чтобы использовать пакет SDK .NET для Azure DNS, необходимо установить пакет NuGet **библиотеки управления DNS для Azure** и другие необходимые пакеты Azure.

1. В **Visual Studio**, откройте существующий или новый проект.
2. Выберите **инструменты** **>** **Диспетчер пакетов NuGet** **>** **Управление пакетами NuGet для решения.**..
3. Щелкните **Обзор**, установите флажок **Включить предварительные выпуски** и введите в поле поиска **Microsoft.Azure.Management.Dns**.
4. Выберите пакет и щелкните **Установить** , чтобы добавить его в проект Visual Studio.
5. Повторите всю процедуру, чтобы установить пакет **Microsoft.Rest.ClientRuntime.Azure.Authentication** и **Microsoft.Azure.Management.ResourceManager**.

## <a name="add-namespace-declarations"></a>Добавление объявлений пространств имен

Добавьте следующие объявления пространств имен:

```cs
using Microsoft.Rest.Azure.Authentication;
using Microsoft.Azure.Management.Dns;
using Microsoft.Azure.Management.Dns.Models;
```

## <a name="initialize-the-dns-management-client"></a>Инициализация DNS-клиента управления

`DnsManagementClient` содержит методы и свойства, необходимые для управления зонами и наборами записей DNS.  Следующий код входит в учетную запись субъекта-службы и создает объект `DnsManagementClient`.

```cs
// Build the service credentials and DNS management client
var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, secret);
var dnsClient = new DnsManagementClient(serviceCreds);
dnsClient.SubscriptionId = subscriptionId;
```

## <a name="create-or-update-a-dns-zone"></a>Создание или обновление зоны DNS

Чтобы создать зону DNS, сначала создается объект Zone, который будет содержать параметры зоны DNS. Так как зоны DNS не связаны с конкретным регионом, для расположения устанавливается значение global. В этом примере в зону также добавляется [тег Azure Resource Manager](https://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/) .

Для создания или обновления зоны в Azure DNS объект, содержащий параметры зоны, передается в метод `DnsManagementClient.Zones.CreateOrUpdateAsyc`.

> [!NOTE]
> DnsManagementClient поддерживает три режима работы: синхронный (CreateOrUpdate), асинхронный (CreateOrUpdateAsync) или асинхронный с доступом к HTTP-ответу (CreateOrUpdateWithHttpMessagesAsync).  Вы можете выбрать любой из этих режимов в зависимости от потребностей приложения.

Служба Azure DNS поддерживает оптимистичный параллелизм с помощью [тегов Etag](./dns-getstarted-powershell.md). В этом примере значение * для заголовка If-None-Match сообщает Azure DNS о необходимости создать зону DNS, если она еще не существует.  Вызов завершится сбоем, если зона с указанным именем уже существует в заданной группе ресурсов.

```cs
// Create zone parameters
var dnsZoneParams = new Zone("global"); // All DNS zones must have location = "global"

// Create an Azure Resource Manager 'tag'.  This is optional.  You can add multiple tags
dnsZoneParams.Tags = new Dictionary<string, string>();
dnsZoneParams.Tags.Add("dept", "finance");

// Create the actual zone.
// Note: Uses 'If-None-Match *' ETAG check, so will fail if the zone exists already.
// Note: For non-async usage, call dnsClient.Zones.CreateOrUpdate(resourceGroupName, zoneName, dnsZoneParams, null, "*")
// Note: For getting the http response, call dnsClient.Zones.CreateOrUpdateWithHttpMessagesAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*")
var dnsZone = await dnsClient.Zones.CreateOrUpdateAsync(resourceGroupName, zoneName, dnsZoneParams, null, "*");
```

## <a name="create-dns-record-sets-and-records"></a>Создание записей и наборов записей DNS

Записи DNS управляются как набор записей. Набор записей — это набор записей с одним и тем же именем и типом в пределах зоны.  Имя набора записей указывается относительно имени зоны, а не полного имени DNS.

Для создания или обновления набора записей создается объект параметров RecordSet, который передается в `DnsManagementClient.RecordSets.CreateOrUpdateAsync`. Здесь также доступно три режима работы: синхронный (CreateOrUpdate), асинхронный (CreateOrUpdateAsync) или асинхронный с доступом к HTTP-ответу (CreateOrUpdateWithHttpMessagesAsync).

Как и в случае с зонами DNS, операции с наборами записей поддерживают оптимистичный параллелизм.  Так как в этом примере не указан ни If-Match, ни If-None-Match, набор записей создается всегда.  В зоне DNS этот вызов перезаписывает любой имеющийся набор записей с таким же именем и типом записи.

```cs
// Create record set parameters
var recordSetParams = new RecordSet();
recordSetParams.TTL = 3600;

// Add records to the record set parameter object.  In this case, we'll add a record of type 'A'
recordSetParams.ARecords = new List<ARecord>();
recordSetParams.ARecords.Add(new ARecord("1.2.3.4"));

// Add metadata to the record set.  Similar to Azure Resource Manager tags, this is optional and you can add multiple metadata name/value pairs
recordSetParams.Metadata = new Dictionary<string, string>();
recordSetParams.Metadata.Add("user", "Mary");

// Create the actual record set in Azure DNS
// Note: no ETAG checks specified, will overwrite existing record set if one exists
var recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSetParams);
```

## <a name="get-zones-and-record-sets"></a>Получение зон и наборов записей

Методы `DnsManagementClient.Zones.Get` и `DnsManagementClient.RecordSets.Get` извлекают отдельные зоны и множество записей соответственно. Коллекции RecordSets идентифицируются по типу, имени и зоне (и группе ресурсов), в которой они существуют. Зоны идентифицируются имени и группе ресурсов, в которой они существуют.

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);
```

## <a name="update-an-existing-record-set"></a>Обновление имеющегося набора записей

Для обновления имеющегося набора записей DNS сначала извлеките его, а затем обновите его содержимое и отправьте изменения.  В этом примере мы указываем Etag из извлеченного набора записей в параметре If-Match. Вызов завершится сбоем, если параллельная операция изменила набор записей.

```cs
var recordSet = dnsClient.RecordSets.Get(resourceGroupName, zoneName, recordSetName, RecordType.A);

// Add a new record to the local object.  Note that records in a record set must be unique/distinct
recordSet.ARecords.Add(new ARecord("5.6.7.8"));

// Update the record set in Azure DNS
// Note: ETAG check specified, update will be rejected if the record set has changed in the meantime
recordSet = await dnsClient.RecordSets.CreateOrUpdateAsync(resourceGroupName, zoneName, recordSetName, RecordType.A, recordSet, recordSet.Etag);
```

## <a name="list-zones-and-record-sets"></a>Вывод списка зон и наборов записей

Чтобы вывести список зон, используйте методы *DnsManagementClient. Zones. List...* , которые поддерживают перечисление всех зон в заданной группе ресурсов или всех зон в заданной подписке Azure (в группах ресурсов). Чтобы вывести список наборов записей, используйте методы *DnsManagementClient. recordsets. List...* , которые поддерживают либо перечисление всех наборов записей в данной зоне, либо только наборы записей определенного типа.

Обратите внимание, что при выводе списка зон и наборов записей может быть применена разбивка на страницы.  В приведенном ниже примере показано, как выполнить итерацию по страницам результатов. (Для принудительного разбиения на страницы используется небольшой размер страницы "2". На практике этот параметр следует пропустить и использовать размер страницы по умолчанию.)

```cs
// Note: in this demo, we'll use a very small page size (2 record sets) to demonstrate paging
// In practice, to improve performance you would use a large page size or just use the system default
int recordSets = 0;
var page = await dnsClient.RecordSets.ListAllInResourceGroupAsync(resourceGroupName, zoneName, "2");
recordSets += page.Count();

while (page.NextPageLink != null)
{
    page = await dnsClient.RecordSets.ListAllInResourceGroupNextAsync(page.NextPageLink);
    recordSets += page.Count();
}
```

## <a name="next-steps"></a>Дальнейшие действия

Загрузите [пример проекта SDK .NET для Azure DNS](https://www.microsoft.com/en-us/download/details.aspx?id=47268&WT.mc_id=DX_MVP4025064&e6b34bbe-475b-1abd-2c51-b5034bcdd6d2=True), который содержит дополнительные примеры использования пакета SDK .NET для Azure DNS, а также примеры других типов записей DNS.