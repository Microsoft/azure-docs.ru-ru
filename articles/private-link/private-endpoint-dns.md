---
title: Конфигурация DNS частной конечной точки Azure
description: Сведения о конфигурации DNS частной конечной точки Azure
services: private-link
author: allensu
ms.service: private-link
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: allensu
ms.openlocfilehash: edb6748009ef9660c020c77e411ce55f60b45791
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102124607"
---
# <a name="azure-private-endpoint-dns-configuration"></a>Конфигурация DNS частной конечной точки Azure

Важно правильно настроить параметры DNS для разрешения IP-адреса частной конечной точки в полное доменное имя (FQDN) строки подключения.

Существующие службы Microsoft Azure могут уже иметь конфигурацию DNS для общедоступной конечной точки. Эту конфигурацию необходимо переопределить для подключения с помощью частной конечной точки. 
 
Сетевой интерфейс, связанный с частной конечной точкой, содержит сведения для настройки DNS. Сведения о сетевом интерфейсе включают FQDN и частные IP-адреса для ресурса частной ссылки. 
 
Чтобы настраивать параметры DNS для частных конечных точек, можно использовать следующие варианты. 
- **Использовать файл узла (рекомендуется только для тестирования).** Для переопределения DNS можно использовать файл узла на виртуальной машине.  
- **Использовать частную зону DNS**. Для переопределения разрешения DNS для частной конечной точки можно использовать [частные зоны DNS](../dns/private-dns-privatednszone.md) . Частная зона DNS может быть связана с вашей виртуальной сетью для разрешения конкретных доменов.
- **Использовать DNS-сервер пересылки (необязательно).** Вы можете использовать сервер пересылки DNS для переопределения разрешения DNS для ресурса частной ссылки. Создайте правило пересылки DNS для использования частной зоны DNS на [DNS-сервере](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server) , размещенном в виртуальной сети.

> [!IMPORTANT]
> Не рекомендуется переопределять зону, которая активно используется для разрешения открытых конечных точек. Подключения к ресурсам не смогут правильно разрешаться без переадресации DNS в общедоступную службу DNS. Чтобы избежать проблем, создайте другое доменное имя или следуйте предлагаемому имени для каждой службы ниже. 

## <a name="azure-services-dns-zone-configuration"></a>Настройка зоны DNS служб Azure
Azure создает каноническое имя DNS-записи (CNAME) на общедоступном DNS-сервере. Запись CNAME перенаправляет разрешение на имя частного домена. Разрешение можно переопределить с помощью частного IP-адреса частных конечных точек. 
 
Приложениям не нужно изменять URL-адрес подключения. При разрешении в общедоступную службу DNS DNS-сервер будет разрешаться в частные конечные точки. Этот процесс не влияет на существующие приложения. 

> [!IMPORTANT]
> Частные сети уже используют частную зону DNS для заданного типа. может подключаться только к общедоступным ресурсам, если у них нет подключений к частным конечным точкам. в противном случае для завершения последовательности разрешения DNS требуется соответствующая конфигурация DNS в частной зоне DNS. 

Для служб Azure используйте рекомендуемые имена зон, описанные в следующей таблице.

| Тип ресурса частной ссылки/Подресурс |Имя Частной зоны DNS | Общедоступные серверы переадресации зоны DNS |
|---|---|---|
| Служба автоматизации Azure/(Microsoft. Automation/automationAccounts)/веб-перехватчик, Дскандхибридворкер | privatelink.azure-automation.net | azure-automation.net |
| База данных SQL Azure (Microsoft.SQL/Servers)/SQL Server | privatelink.database.windows.net | database.windows.net |
| Azure Synapse Analytics (Microsoft.SQL/servers)/SQL Server  | privatelink.database.windows.net | database.windows.net |
| Учетная запись хранения (Microsoft.Storage/storageAccounts)/Большой двоичный объект (blob, blob_secondary) | privatelink.blob.core.windows.net | blob.core.windows.net |
| Учетная запись хранения (Microsoft.Storage/storageAccounts)/Таблица (table, table_secondary) | privatelink.table.core.windows.net | table.core.windows.net |
| Учетная запись хранения (Microsoft.Storage/storageAccounts)/Очередь (queue, queue_secondary) | privatelink.queue.core.windows.net | queue.core.windows.net |
| Учетная запись хранения (Microsoft.Storage/storageAccounts)/Файл (file, file_secondary) | privatelink.file.core.windows.net | file.core.windows.net |
| Учетная запись хранения (Microsoft.Storage/storageAccounts)/Сеть (web, web_secondary) | privatelink.web.core.windows.net | web.core.windows.net |
| Файловая система Azure Data Lake 2-го поколения (Microsoft.Storage/storageAccounts)/Файловая система Data Lake 2-го поколения (dfs, dfs_secondary) | privatelink.dfs.core.windows.net | dfs.core.windows.net |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.com | documents.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.com | mongo.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.com | cassandra.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.com | gremlin.cosmos.azure.com |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Таблица | privatelink.table.cosmos.azure.com | table.cosmos.azure.com |
| База данных Azure для PostgreSQL — одиночный сервер (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.azure.com | postgres.database.azure.com |
| База данных Azure для MySQL (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.azure.com | mysql.database.azure.com |
| База данных Azure для MariaDB (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.azure.com | mariadb.database.azure.com |
| Azure Key Vault (Microsoft.KeyVault/vaults) / vault | privatelink.vaultcore.azure.net | vault.azure.net <br> vaultcore.azure.net |
| Azure Kubernetes Service — API Kubernetes (Microsoft. ContainerService/Манажедклустерс)/Управление | privatelink.{region}.azmk8s.io | {region}.azmk8s.io |
| Поиск Azure (Microsoft.Search/searchServices) / searchService | privatelink.search.windows.net | search.windows.net |
| Реестр контейнеров Azure (Microsoft.ContainerRegistry/registries) / реестр | privatelink.azurecr.io | azurecr.io |
| Конфигурация приложений Azure (Microsoft.AppConfiguration/configurationStores) / configurationStore | privatelink.azconfig.io | azconfig.io |
| Azure Backup (Microsoft.RecoveryServices/vaults) / хранилище | privatelink.{region}.backup.windowsazure.com | {region}.backup.windowsazure.com |
| Azure Site Recovery (Microsoft. RecoveryServices/Vault)/хранилище | {Region}. привателинк. siterecovery. WindowsAzure. com | {Region}. хиперврековериманажер. WindowsAzure. com |
| Центры событий Azure (Microsoft.EventHub/namespaces)/Пространство имен | privatelink.servicebus.windows.net | servicebus.windows.net. |
| Служебная шина Azure (Microsoft.ServiceBus/namespaces) / пространство имен | privatelink.servicebus.windows.net | servicebus.windows.net. |
| Центр Интернета вещей Azure (Microsoft. Devices/IotHubs)/iotHub | privatelink.azure-devices.net<br/>privatelink.servicebus.windows.net<sup>1</sup> | azure-devices.net<br/>servicebus.windows.net. |
| Azure Relay (Microsoft.Relay/namespaces) / пространство имен | privatelink.servicebus.windows.net | servicebus.windows.net. |
| Сетка событий Azure (Microsoft.EventGrid/topics) / тема | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Сетка событий Azure (Microsoft.EventGrid/domains) / домен | privatelink.eventgrid.azure.net | eventgrid.azure.net |
| Веб-приложения Azure (Microsoft. Web/Sites) и сайты | privatelink.azurewebsites.net | azurewebsites.net; |
| Машинное обучение Azure (Microsoft. Мачинелеарнингсервицес/workspaces)/амлворкспаце | privatelink.api.azureml.ms<br/>privatelink.notebooks.azure.net | api.azureml.ms<br/>notebooks.azure.net<br/>instances.azureml.ms<br/>aznbcontent.net |
| SignalR (Microsoft. Сигналрсервице/SignalR)/SignalR | privatelink.service.signalr.net | service.signalr.net |
| Azure Monitor (Microsoft. Insights/Привателинкскопес)/азуремонитор | privatelink.monitor.azure.com<br/> privatelink.oms.opinsights.azure.com <br/> privatelink.ods.opinsights.azure.com <br/> privatelink.agentsvc.azure-automation.net | monitor.azure.com<br/> oms.opinsights.azure.com<br/> ods.opinsights.azure.com<br/> agentsvc.azure-automation.net |
| Cognitive Services (Microsoft. CognitiveServices/Accounts)/Account | privatelink.cognitiveservices.azure.com  | cognitiveservices.azure.com  |
| Синхронизация файлов Azure (Microsoft. StorageSync/Сторажесинксервицес)/AFS |  privatelink.afs.azure.net  |  afs.azure.net  |
| Фабрика данных Azure (Microsoft. данные и фабрики)/факт |  privatelink.datafactory.azure.net  |  datafactory.azure.net  |
| Фабрика данных Azure (Microsoft. данные и фабрики)/портал |  privatelink.azure.com  |  azure.com  |
| Кэш Azure для Redis (Microsoft. Cache/Redis)/redisCache | privatelink.redis.cache.windows.net | redis.cache.windows.net |

<sup>1</sup> Для использования с встроенной конечной точкой центра Интернета вещей, совместимой с концентратором событий. Дополнительные сведения см. в разделе [Поддержка частных ссылок для встроенной конечной точки центра Интернета вещей](../iot-hub/virtual-network-support.md#built-in-event-hub-compatible-endpoint) .

### <a name="china"></a>Китай

| Тип ресурса частной ссылки/Подресурс |Имя Частной зоны DNS | Общедоступные серверы переадресации зоны DNS |
|---|---|---|
| База данных SQL Azure (Microsoft.SQL/Servers)/SQL Server | privatelink.database.chinacloudapi.cn | database.chinacloudapi.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / SQL | privatelink.documents.azure.cn | documents.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / MongoDB | privatelink.mongo.cosmos.azure.cn | mongo.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Cassandra | privatelink.cassandra.cosmos.azure.cn | cassandra.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Gremlin | privatelink.gremlin.cosmos.azure.cn | gremlin.cosmos.azure.cn |
| Azure Cosmos DB (Microsoft.AzureCosmosDB/databaseAccounts) / Таблица | privatelink.table.cosmos.azure.cn | table.cosmos.azure.cn |
| База данных Azure для PostgreSQL — одиночный сервер (Microsoft.DBforPostgreSQL/servers) / postgresqlServer | privatelink.postgres.database.chinacloudapi.cn | postgres.database.chinacloudapi.cn |
| База данных Azure для MySQL (Microsoft.DBforMySQL/servers) / mysqlServer | privatelink.mysql.database.chinacloudapi.cn  | mysql.database.chinacloudapi.cn  |
| База данных Azure для MariaDB (Microsoft.DBforMariaDB/servers) / mariadbServer | privatelink.mariadb.database.chinacloudapi.cn | mariadb.database.chinacloudapi.cn |


## <a name="dns-configuration-scenarios"></a>Сценарии настройки DNS

Полное доменное имя служб автоматически разрешается в общедоступный IP-адрес. Чтобы разрешить доступ к частному IP-адресу частной конечной точки, измените конфигурацию DNS.

DNS является критически важным компонентом, обеспечивающим правильную работу приложения за счет правильного разрешения IP-адреса частной конечной точки.

Доступны следующие встроенные сценарии, связанные с разрешением DNS:

- [Рабочие нагрузки виртуальных сетей без пользовательского DNS-сервера](#virtual-network-workloads-without-custom-dns-server)
- [Локальные рабочие нагрузки с использованием DNS-сервера переадресации](#on-premises-workloads-using-a-dns-forwarder)
- [Рабочие нагрузки виртуальных сетей и локальные рабочие нагрузки с использованием DNS-сервера пересылки](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder)

> [!NOTE]
> [DNS-прокси-сервер брандмауэра Azure](../firewall/dns-settings.md#dns-proxy) можно использовать в качестве DNS-сервера пересылки для [локальных рабочих нагрузок](#on-premises-workloads-using-a-dns-forwarder) и [рабочих нагрузок виртуальной сети с помощью сервера пересылки DNS](#virtual-network-and-on-premises-workloads-using-a-dns-forwarder).

## <a name="virtual-network-workloads-without-custom-dns-server"></a>Рабочие нагрузки виртуальных сетей без пользовательского DNS-сервера

Эта конфигурация подходит для рабочих нагрузок виртуальных сетей без пользовательского DNS-сервера. В этом сценарии клиент запрашивает IP-адрес частной конечной точки для предоставленной Azure службы DNS [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md). Azure DNS будет отвечать за разрешение DNS для частных зон DNS.

> [!NOTE]
> В этом сценарии используется частная зона DNS, рекомендуемая Базой данных SQL Azure. Для других служб можно настроить эту модель, используя следующий ресурс: [Настройка зоны DNS служб Azure](#azure-services-dns-zone-configuration).

Для правильной настройки необходимы следующие ресурсы:

- виртуальная сеть клиента;

- Частная зона DNS [privatelink.database.windows.net](../dns/private-dns-privatednszone.md) с [записью типа A](../dns/dns-zones-records.md#record-types);

- сведения о частной конечной точке (имя записи FQDN и частный IP-адрес).

На следующем снимке экрана показана последовательность разрешения DNS из рабочих нагрузок виртуальной сети с использованием частной зоны DNS.

:::image type="content" source="media/private-endpoint-dns/single-vnet-azure-dns.png" alt-text="Одна виртуальная сеть и DNS, предоставляемые Azure":::

Эту модель можно расширить на одноранговые виртуальные сети, связанные с той же частной конечной точкой. [Добавьте новые ссылки на виртуальную сеть](../dns/private-dns-virtual-network-links.md) в зону частных DNS для всех одноранговых виртуальных сетей.

> [!IMPORTANT]
> Для этой конфигурации требуется одна частная зона DNS. При создании нескольких зон с одинаковым именем для разных виртуальных сетей потребуется выполнить операции объединения записей DNS вручную.

> [!IMPORTANT]
> Если вы используете частную конечную точку в модели звезды из другой подписки, повторно используйте ту же частную зону DNS в концентраторе.

В этом сценарии есть топология сети [HUB и периферийная](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) сеть. Периферийные сети совместно используют закрытую конечную точку. Периферийные виртуальные сети связаны с той же частной зоной DNS. 

:::image type="content" source="media/private-endpoint-dns/hub-and-spoke-azure-dns.png" alt-text="Модель звезда с DNS, предоставляемыми Azure":::

## <a name="on-premises-workloads-using-a-dns-forwarder"></a>Локальные рабочие нагрузки с использованием DNS-сервера переадресации

Для локальной рабочей нагрузки, чтобы разрешить полное доменное имя частной конечной точки, используйте DNS-сервер пересылки для разрешения [зоны общедоступной службы DNS](#azure-services-dns-zone-configuration) Azure в Azure.

Следующий сценарий предназначен для локальной сети, в которой есть сервер пересылки DNS в Azure. Этот сервер пересылки разрешает запросы DNS, используя сервер пересылки на уровне сервера, к предоставленному Azure DNS [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md). 

> [!NOTE]
> В этом сценарии используется частная зона DNS, рекомендуемая Базой данных SQL Azure. Для других служб можно настроить эту модель, используя следующий ресурс: [Настройка зоны DNS служб Azure](#azure-services-dns-zone-configuration).

Для правильной настройки необходимы следующие ресурсы:

- Локальная сеть
- Виртуальная сеть [, подключенная к локальной среде](/azure/architecture/reference-architectures/hybrid-networking/)
- DNS-сервер переадресации, развернутый в Azure; 
- Частная зона DNS зоны [privatelink.Database.Windows.NET](../dns/private-dns-privatednszone.md) с [типом записи A](../dns/dns-zones-records.md#record-types)
- сведения о частной конечной точке (имя записи FQDN и частный IP-адрес).

На следующей схеме показана последовательность разрешения DNS из локальной сети. В конфигурации используется сервер пересылки DNS, развернутый в Azure. Разрешение осуществляется с помощью частной зоны DNS, [связанной с виртуальной сетью](../dns/private-dns-virtual-network-links.md).

:::image type="content" source="media/private-endpoint-dns/on-premises-using-azure-dns.png" alt-text="Локальная среда с использованием Azure DNS":::

Эту конфигурацию можно расширить для локальной сети, в которой уже есть решение DNS. Локальное решение DNS настроено на перенаправление трафика DNS для Azure DNS через [сервер условной пересылки](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server). Условный сервер пересылки ссылается на сервер пересылки DNS, развернутый в Azure.

> [!NOTE]
> В этом сценарии используется частная зона DNS, рекомендуемая Базой данных SQL Azure. Для других служб можно настроить модель, используя следующую ссылку: [Конфигурация зоны DNS служб Azure](#azure-services-dns-zone-configuration) .

Для правильной настройки необходимы следующие ресурсы:

- локальная сеть с настраиваемым решением DNS; 
- Виртуальная сеть [, подключенная к локальной среде](/azure/architecture/reference-architectures/hybrid-networking/)
- DNS-сервер переадресации, развернутый в Azure;
- Частная зона DNS зоны [privatelink.Database.Windows.NET](../dns/private-dns-privatednszone.md)  с [типом записи A](../dns/dns-zones-records.md#record-types)
- сведения о частной конечной точке (имя записи FQDN и частный IP-адрес).

На следующей схеме показано разрешение DNS из локальной сети. Разрешение DNS условно перенаправляется в Azure. Разрешение осуществляется с помощью частной зоны DNS, [связанной с виртуальной сетью](../dns/private-dns-virtual-network-links.md).

> [!IMPORTANT]
> Условную переадресацию следует выполнять для рекомендуемого [сервера пересылки общедоступной зоны DNS](#azure-services-dns-zone-configuration). Пример: `database.windows.net` вместо **привателинк**. Database.Windows.NET.

:::image type="content" source="media/private-endpoint-dns/on-premises-forwarding-to-azure.png" alt-text="Локальная переадресация в Azure DNS":::

## <a name="virtual-network-and-on-premises-workloads-using-a-dns-forwarder"></a>Рабочие нагрузки виртуальных сетей и локальные рабочие нагрузки с использованием DNS-сервера пересылки

Для рабочих нагрузок, обращающихся к частной конечной точке из виртуальных и локальных сетей, используйте DNS-сервер пересылки для разрешения [общедоступной зоны DNS](#azure-services-dns-zone-configuration) службы Azure, развернутой в Azure.

Следующий сценарий предназначен для локальной сети с виртуальными сетями в Azure. Обе сети обращаются к частной конечной точке, расположенной в общей сети концентратора.

Этот DNS-сервер пересылки отвечает за разрешение всех запросов DNS с помощью серверной пересылки в предоставленную Azure службу DNS [168.63.129.16](../virtual-network/what-is-ip-address-168-63-129-16.md).

> [!IMPORTANT]
> Для этой конфигурации требуется одна частная зона DNS. Все клиентские подключения из локальных и [пиринговых виртуальных сетей](../virtual-network/virtual-network-peering-overview.md) также должны использовать одну частную зону DNS.

> [!NOTE]
> В этом сценарии используется частная зона DNS, рекомендуемая Базой данных SQL Azure. Для других служб можно настроить эту модель, используя следующий ресурс: [Настройка зоны DNS служб Azure](#azure-services-dns-zone-configuration).

Для правильной настройки необходимы следующие ресурсы:

- Локальная сеть
- Виртуальная сеть [, подключенная к локальной среде](/azure/architecture/reference-architectures/hybrid-networking/)
- [пиринговая виртуальная сеть;](../virtual-network/virtual-network-peering-overview.md) 
- DNS-сервер переадресации, развернутый в Azure;
- Частная зона DNS зоны [privatelink.Database.Windows.NET](../dns/private-dns-privatednszone.md)  с [типом записи A](../dns/dns-zones-records.md#record-types)
- сведения о частной конечной точке (имя записи FQDN и частный IP-адрес).

На следующей схеме показано разрешение DNS для сетей, локальных и виртуальных сетей. Разрешение использует сервер пересылки DNS. Разрешение осуществляется с помощью частной зоны DNS, [связанной с виртуальной сетью](../dns/private-dns-virtual-network-links.md).

:::image type="content" source="media/private-endpoint-dns/hybrid-scenario.png" alt-text="Гибридный сценарий":::

## <a name="next-steps"></a>Дальнейшие действия
- [Узнайте больше о частных конечных точках](private-endpoint-overview.md)
