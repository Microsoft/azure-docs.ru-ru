---
title: Управление кэшем Azure для Redis с помощью Azure PowerShell | Документация Майкрософт
description: Сведения об администрировании кэша Azure для Redis с помощью Azure PowerShell.
services: cache
documentationcenter: ''
author: yegu-ms
manager: jhubbard
editor: ''
ms.assetid: 1136efe5-1e33-4d91-bb49-c8e2a6dca475
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.author: yegu
ms.openlocfilehash: f7f4f9ae6a80052e06b2cafa68cb5c11dfa1333a
ms.sourcegitcommit: d4dfbc34a1f03488e1b7bc5e711a11b72c717ada
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/13/2019
ms.locfileid: "62097935"
---
# <a name="manage-azure-cache-for-redis-with-azure-powershell"></a>Управление кэшем Azure для Redis с использованием Azure PowerShell
> [!div class="op_single_selector"]
> * [PowerShell](cache-howto-manage-redis-cache-powershell.md)
> * [Интерфейс командной строки Azure](cache-manage-cli.md)
> 
> 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

В этом разделе демонстрируется выполнение таких обычных задач, как создание, обновление и масштабирование экземпляров кэша Azure для Redis, повторное создание ключей доступа и просмотр сведений о кэшах. Ознакомьтесь с полным списком [командлетов PowerShell для работы с кэшем Azure для Redis](https://docs.microsoft.com/powershell/module/az.rediscache).

[!INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]

Дополнительные сведения о классической модели развертывания см. в статье [Развертывание с помощью Azure Resource Manager и классическое развертывание: сведения о моделях развертывания и состоянии ресурсов](../azure-resource-manager/resource-manager-deployment-model.md).

## <a name="prerequisites"></a>Технические условия
Если вы уже установили Azure PowerShell, необходимо использовать Azure PowerShell 1.0.0 или более поздней версии. Установленную версию Azure PowerShell можно узнать в командной строке Azure PowerShell с помощью такой команды:

    Get-Module Az | format-table version


Сначала необходимо войти в Azure с помощью следующей команды.

    Connect-AzAccount

Укажите адрес электронной почты и пароль своей учетной записи Azure в диалоговом окне входа в Microsoft Azure.

Далее, если у вас есть несколько подписок Azure, необходимо задать лишь одну из них. Чтобы просмотреть список текущих подписок, выполните следующую команду.

    Get-AzSubscription | sort SubscriptionName | Select SubscriptionName

Чтобы указать подписку, выполните указанную ниже команду. В приведенном ниже примере имя подписки — `ContosoSubscription`.

    Select-AzSubscription -SubscriptionName ContosoSubscription

Чтобы использовать Windows PowerShell с диспетчером ресурсов Azure, необходимо следующее:

* Windows PowerShell версии 3.0 или 4.0. Чтобы найти версию Windows PowerShell, введите `$PSVersionTable` и убедитесь, что для `PSVersion` указано значение 3.0 или 4.0. Сведения об установке совместимой версии см. в разделе [Windows Management Framework 3.0](https://www.microsoft.com/download/details.aspx?id=34595) или [Windows Management Framework 4.0](https://www.microsoft.com/download/details.aspx?id=40855).

Чтобы получить подробную справку для любого командлета, встречающегося в этом учебнике, используйте командлет Get-Help.

    Get-Help <cmdlet-name> -Detailed

Например, чтобы получить справку для командлета `New-AzRedisCache` , введите:

    Get-Help New-AzRedisCache -Detailed

### <a name="how-to-connect-to-other-clouds"></a>Подключение к другим облакам
Среда Azure по умолчанию — это `AzureCloud`, представляющая экземпляр глобального облака Azure. Чтобы подключиться к другому экземпляру, используйте команду `Connect-AzAccount`, заменив параметр `-Environment` или -`EnvironmentName` на нужную вам среду или имя среды.

Чтобы просмотреть список доступных сред, выполните командлет `Get-AzEnvironment` .

### <a name="to-connect-to-the-azure-government-cloud"></a>Подключение к облаку Azure Government
Чтобы подключиться к облаку Azure Government, используйте одну из следующих команд.

    Connect-AzAccount -EnvironmentName AzureUSGovernment

или

    Connect-AzAccount -Environment (Get-AzEnvironment -Name AzureUSGovernment)

Чтобы создать кэш в облаке Azure Government, используйте одно из следующих расположений.

* Правительство штата Вирджиния
* Правительство штата Айова

Дополнительные сведения об облаке Azure для государственных организаций см. на странице [Microsoft Azure для государственных организаций](https://azure.microsoft.com/features/gov/) и в [руководстве для разработчиков Microsoft Azure для государственных организаций](../azure-government-developer-guide.md).

### <a name="to-connect-to-the-azure-china-cloud"></a>Подключение к облаку Azure China
Чтобы подключиться к облаку Azure China, используйте одну из следующих команд.

    Connect-AzAccount -EnvironmentName AzureChinaCloud

или

    Connect-AzAccount -Environment (Get-AzEnvironment -Name AzureChinaCloud)

Чтобы создать кэш в облаке Azure China, используйте одно из следующих расположений.

* Восток Китая
* Север Китая

Дополнительные сведения об облаке Azure для Китая см. на странице [AzureChinaCloud для Azure под управлением 21Vianet в Китае](http://www.windowsazure.cn/).

### <a name="to-connect-to-microsoft-azure-germany"></a>Подключение к Microsoft Azure для Германии
Чтобы подключиться к Microsoft Azure для Германии, используйте одну из следующих команд.

    Connect-AzAccount -EnvironmentName AzureGermanCloud


или

    Connect-AzAccount -Environment (Get-AzEnvironment -Name AzureGermanCloud)

Чтобы создать кэш в Microsoft Azure для Германии, используйте одно из следующих расположений.

* Центральная Германия
* Северо-восточная Германия

Дополнительные сведения о Microsoft Azure для Германии см. [здесь](https://azure.microsoft.com/overview/clouds/germany/).

### <a name="properties-used-for-azure-cache-for-redis-powershell"></a>Свойства, используемые в командлетах PowerShell кэша Azure для Redis
Приведенная ниже таблица содержит свойства и описания параметров, часто используемых при создании экземпляров кэша Azure для Redis с помощью Azure PowerShell и управлении такими экземплярами.

| Параметр | Описание | значение по умолчанию |
| --- | --- | --- |
| Name |Имя кэша | |
| Location |Расположение кэша | |
| ResourceGroupName |Имя группы ресурсов, в которой необходимо создать кэш | |
| Size |Размер кэша. Допустимые значения: P1, P2, P3, P4, C0, C1, C2, C3, C4, C5, C6, 250 МБ, 1 ГБ, 2,5 ГБ, 6 ГБ, 13 ГБ, 26 ГБ, 53 ГБ |1 ГБ |
| ShardCount |Число сегментов, которые будут созданы при создании кэша уровня Premium с включенной кластеризацией. Допустимые значения: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 | |
| SKU |Определяет SKU кэша. Допустимые значения: "Базовый", "Стандартный" и "Премиум" |Стандартная |
| RedisConfiguration |Задает параметры конфигурации кластера Redis. Подробные сведения о каждом параметре представлены в таблице [Свойства RedisConfiguration](#redisconfiguration-properties) . | |
| EnableNonSslPort |Определяет, включен ли порт без SSL. |False |
| MaxMemoryPolicy |Этот параметр устарел, вместо него используется параметр RedisConfiguration. | |
| StaticIP |При размещении кэша в виртуальной сети определяет уникальный IP-адрес подсети для кэша. Если IP-адрес не указан, он автоматически выбирается из подсети. | |
| Subnet |При размещении кэша в виртуальной сети определяет имя подсети, в которой будет развернут кэш. | |
| VirtualNetwork |При размещении кэша в виртуальной сети определяет идентификатор ресурса виртуальной сети, в которой будет развернут кэш. | |
| KeyType |Определяет, какой ключ доступа будет создаваться повторно при обновлении ключей доступа. Допустимые значения: первичный или вторичный | |

### <a name="redisconfiguration-properties"></a>Свойства RedisConfiguration
| Свойство | Описание | Ценовые категории |
| --- | --- | --- |
| rdb-backup-enabled |Указывает на то, включен ли параметр [Сохраняемость данных Redis](cache-how-to-premium-persistence.md) . |Только "Премиум" |
| rdb-storage-connection-string |Строка подключения к учетной записи хранения для параметра [Сохраняемость данных Redis](cache-how-to-premium-persistence.md) |Только "Премиум" |
| rdb-backup-frequency |Указывает частоту резервного копирования для параметра [Сохраняемость данных Redis](cache-how-to-premium-persistence.md) |Только "Премиум" |
| maxmemory-reserved |Определяет [объем памяти, зарезервированный](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) для процессов, не связанных с кэшем. |"Стандартный" и "Премиум" |
| maxmemory-policy |Определяет [политику вытеснения](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) для кэша. |Все ценовые категории |
| notify-keyspace-events |Настраивает [уведомления пространства ключей](cache-configure.md#keyspace-notifications-advanced-settings) |"Стандартный" и "Премиум" |
| hash-max-ziplist-entries |Настраивает [оптимизацию памяти](https://redis.io/topics/memory-optimization) для небольших сводных данных. |"Стандартный" и "Премиум" |
| hash-max-ziplist-value |Настраивает [оптимизацию памяти](https://redis.io/topics/memory-optimization) для небольших сводных данных. |"Стандартный" и "Премиум" |
| set-max-intset-entries |Настраивает [оптимизацию памяти](https://redis.io/topics/memory-optimization) для небольших сводных данных. |"Стандартный" и "Премиум" |
| zset-max-ziplist-entries |Настраивает [оптимизацию памяти](https://redis.io/topics/memory-optimization) для небольших сводных данных. |"Стандартный" и "Премиум" |
| zset-max-ziplist-value |Настраивает [оптимизацию памяти](https://redis.io/topics/memory-optimization) для небольших сводных данных. |"Стандартный" и "Премиум" |
| databases |Определяет количество баз данных. Это свойство можно настроить только в момент создания кэша. |"Стандартный" и "Премиум" |

## <a name="to-create-an-azure-cache-for-redis"></a>Создание экземпляра кэша Azure для Redis
Новые экземпляры кэша Azure для Redis создаются с помощью командлета [New-AzRedisCache](https://docs.microsoft.com/powershell/module/az.rediscache/new-azrediscache).

> [!IMPORTANT]
> Когда кэш Azure для Redis создается в подписке с использованием портала Azure впервые, портал регистрирует пространство имен `Microsoft.Cache` для этой подписки. Если первый кэш Azure для Redis создается в подписке с использованием PowerShell, необходимо зарегистрировать пространство имен с помощью представленной ниже команды, иначе командлеты `New-AzRedisCache` и `Get-AzRedisCache` завершатся ошибкой.
> 
> `Register-AzResourceProvider -ProviderNamespace "Microsoft.Cache"`
> 
> 

Чтобы увидеть список доступных параметров свойства `New-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help New-AzRedisCache -detailed

    NAME
        New-AzRedisCache

    SYNOPSIS
        Creates a new Azure Cache for Redis.


    SYNTAX
        New-AzRedisCache -Name <String> -ResourceGroupName <String> -Location <String> [-RedisVersion <String>]
        [-Size <String>] [-Sku <String>] [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort
        <Boolean>] [-ShardCount <Integer>] [-VirtualNetwork <String>] [-Subnet <String>] [-StaticIP <String>]
        [<CommonParameters>]


    DESCRIPTION
        The New-AzRedisCache cmdlet creates a new Azure Cache for Redis.


    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis to create.

        -ResourceGroupName <String>
            Name of resource group in which to create the Azure Cache for Redis.

        -Location <String>
            Location in which to create the Azure Cache for Redis.

        -RedisVersion <String>
            RedisVersion is deprecated and will be removed in future release.

        -Size <String>
            Size of the Azure Cache for Redis. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
            C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.

        -Sku <String>
            Sku of Azure Cache for Redis. The default value is Standard. Possible values are Basic, Standard and Premium.

        -MaxMemoryPolicy <String>
            The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
            MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}

        -RedisConfiguration <Hashtable>
            All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
            rdb-backup-frequency, maxmemory-reserved, maxmemory-policy, notify-keyspace-events, hash-max-ziplist-entries,
            hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries, zset-max-ziplist-value, databases.

        -EnableNonSslPort <Boolean>
            EnableNonSslPort is used by Azure Cache for Redis. If no value is provided, the default value is false and the
            non-SSL port will be disabled. Possible values are true and false.

        -ShardCount <Integer>
            The number of shards to create on a Premium Cluster Cache.

        -VirtualNetwork <String>
            The exact ARM resource ID of the virtual network to deploy the Azure Cache for Redis in. Example format: /subscriptions/{
            subid}/resourceGroups/{resourceGroupName}/providers/Microsoft.ClassicNetwork/VirtualNetworks/{vnetName}

        -Subnet <String>
            Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network.

        -StaticIP <String>
            Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

Чтобы создать кэш с параметрами по умолчанию, выполните следующую команду:

    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US"

`ResourceGroupName`, `Name` и `Location` — обязательные параметры; остальные указываются по желанию и имеют значения по умолчанию. При выполнении предыдущей команды создается экземпляр кэша Azure для Redis SKU "Стандартный" с заданным именем, расположением и группой ресурсов; размер экземпляра — 1 ГБ, порт без SSL отключен.

Для создания кэша уровня Premium укажите размер P1 (6-60 ГБ), P2 (13-130 ГБ), (26-260 ГБ), P3 и P4 (53-530 ГБ). Чтобы включить кластеризацию, укажите количество сегментов с помощью параметра `ShardCount` . В следующем примере создается кэш P1 уровня Premium с 3 сегментами. Размер кэша P1 уровня Premium — 6 ГБ, и, поскольку мы указали три сегмента, общий размер составляет 18 ГБ (3 x 6 ГБ).

    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -Sku Premium -Size P1 -ShardCount 3

Чтобы указать значения для параметра `RedisConfiguration`, включите их в фигурные скобки (`{}`) как пары "ключ-значение" типа `@{"maxmemory-policy" = "allkeys-random", "notify-keyspace-events" = "KEA"}`. В следующем примере создается кэш уровня Standard размером 1 ГБ с политикой максимальной памяти `allkeys-random` и уведомлениями пространства ключей, настроенными с помощью `KEA`. Дополнительные сведения см. в разделах [Уведомления пространства ключей (дополнительные параметры)](cache-configure.md#keyspace-notifications-advanced-settings) и [Политики памяти](cache-configure.md#memory-policies).

    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random", "notify-keyspace-events" = "KEA"}

<a name="databases"></a>

## <a name="to-configure-the-databases-setting-during-cache-creation"></a>Настройка параметров баз данных в процессе создания кэша
Параметр `databases` можно настроить только при создании кэша. В следующем примере создается кэш P3 (26 ГБ) уровня Premium на 48 базах данных с использованием командлета [New-AzRedisCache](https://docs.microsoft.com/powershell/module/az.rediscache/New-azRedisCache).

    New-AzRedisCache -ResourceGroupName myGroup -Name mycache -Location "North Central US" -Sku Premium -Size P3 -RedisConfiguration @{"databases" = "48"}

Дополнительные сведения о свойстве `databases` см. в разделе о [конфигурации сервера кэша Azure для Redis по умолчанию](cache-configure.md#default-redis-server-configuration). Дополнительные сведения о создании кэша с помощью командлета [New-AzRedisCache](https://docs.microsoft.com/powershell/module/az.rediscache/new-azrediscache) см. в предыдущем разделе "Создание экземпляра кэша Azure для Redis".

## <a name="to-update-an-azure-cache-for-redis"></a>Обновление экземпляра кэша Azure для Redis
Экземпляры кэша Azure для Redis обновляются с помощью командлета [Set-AzRedisCache](https://docs.microsoft.com/powershell/module/az.rediscache/Set-azRedisCache).

Чтобы увидеть список доступных параметров свойства `Set-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help Set-AzRedisCache -detailed

    NAME
        Set-AzRedisCache

    SYNOPSIS
        Set Azure Cache for Redis updatable parameters.

    SYNTAX
        Set-AzRedisCache -Name <String> -ResourceGroupName <String> [-Size <String>] [-Sku <String>]
        [-MaxMemoryPolicy <String>] [-RedisConfiguration <Hashtable>] [-EnableNonSslPort <Boolean>] [-ShardCount
        <Integer>] [<CommonParameters>]

    DESCRIPTION
        The Set-AzRedisCache cmdlet sets Azure Cache for Redis parameters.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis to update.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        -Size <String>
            Size of the Azure Cache for Redis. The default value is 1GB or C1. Possible values are P1, P2, P3, P4, C0, C1, C2, C3,
            C4, C5, C6, 250MB, 1GB, 2.5GB, 6GB, 13GB, 26GB, 53GB.

        -Sku <String>
            Sku of Azure Cache for Redis. The default value is Standard. Possible values are Basic, Standard and Premium.

        -MaxMemoryPolicy <String>
            The 'MaxMemoryPolicy' setting has been deprecated. Please use 'RedisConfiguration' setting to set
            MaxMemoryPolicy. e.g. -RedisConfiguration @{"maxmemory-policy" = "allkeys-lru"}

        -RedisConfiguration <Hashtable>
            All Redis Configuration Settings. Few possible keys: rdb-backup-enabled, rdb-storage-connection-string,
            rdb-backup-frequency, maxmemory-reserved, maxmemory-policy, notify-keyspace-events, hash-max-ziplist-entries,
            hash-max-ziplist-value, set-max-intset-entries, zset-max-ziplist-entries, zset-max-ziplist-value.

        -EnableNonSslPort <Boolean>
            EnableNonSslPort is used by Azure Cache for Redis. The default value is null and no change will be made to the
            currently configured value. Possible values are true and false.

        -ShardCount <Integer>
            The number of shards to create on a Premium Cluster Cache.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

Командлет `Set-AzRedisCache` можно использовать для обновления таких свойств, как значения `Size`, `Sku`, `EnableNonSslPort` и `RedisConfiguration`. 

Следующая команда обновляет политику максимального объема памяти (maxmemory-policy) кэша Azure для Redis с именем myCache.

    Set-AzRedisCache -ResourceGroupName "myGroup" -Name "myCache" -RedisConfiguration @{"maxmemory-policy" = "allkeys-random"}

<a name="scale"></a>

## <a name="to-scale-an-azure-cache-for-redis"></a>Масштабирование кэша Azure для Redis
`Set-AzRedisCache` можно использовать для масштабирования экземпляров кэша Azure для Redis при изменении свойств `Size`, `Sku` или `ShardCount`. 

> [!NOTE]
> Масштабирование кэша с помощью PowerShell подчиняется тем же ограничениям и правилам, что и масштабирование кэша на портале Azure. Вы можете выполнить масштабирование до другой ценовой категории со следующими ограничениями.
> 
> * Перейти с более высокой ценовой категории на более низкую нельзя.
> * Ценовую категорию кэша **Премиум** нельзя изменить на категорию **Стандартный** или **Базовый**.
> * Ценовую категорию кэша **Стандартный** нельзя изменить на категорию **Базовый**.
> * Вы можете выполнить масштабирование кэша с уровня **Базовый** до уровня **Стандартный**, но вам не удастся одновременно с этим изменить размер кэша. Если требуется изменить размер, можно выполнить последующую операцию масштабирования до нужного размера.
> * Ценовую категорию кэша **Базовый** нельзя изменить сразу на уровень **Премиум**. Необходимо сначала перейти с категории **Базовый** на категорию **Стандартный**, а затем — с категории **Стандартный** на категорию **Премиум**.
> * Вам не удастся выполнить масштабирование с большего размера до размера **C0 (250 МБ)** .
> 
> Дополнительные сведения см. в статье [How to Scale Azure Cache for Redis](cache-how-to-scale.md) (Как масштабировать кэш Azure для Redis).
> 
> 

В следующем примере демонстрируется масштабирование кэша с именем `myCache` в кэш размером 2,5 ГБ. Обратите внимание, что команда будет работать в кэше уровня "Базовый" или "Стандартный".

    Set-AzRedisCache -ResourceGroupName myGroup -Name myCache -Size 2.5GB

После запуска этой команды будет возвращено состояние кэша (аналогично вызову `Get-AzRedisCache`). Обратите внимание на то, что параметр `ProvisioningState` имеет значение `Scaling`.

    PS C:\> Set-AzRedisCache -Name myCache -ResourceGroupName myGroup -Size 2.5GB


    Name               : mycache
    Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/mygroup/providers/Mi
                         crosoft.Cache/Redis/mycache
    Location           : South Central US
    Type               : Microsoft.Cache/Redis
    HostName           : mycache.redis.cache.windows.net
    Port               : 6379
    ProvisioningState  : Scaling
    SslPort            : 6380
    RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 150], [notify-keyspace-events, KEA],
                         [maxmemory-delta, 150]...}
    EnableNonSslPort   : False
    RedisVersion       : 3.0
    Size               : 1GB
    Sku                : Standard
    ResourceGroupName  : mygroup
    PrimaryKey         : ....
    SecondaryKey       : ....
    VirtualNetwork     :
    Subnet             :
    StaticIP           :
    TenantSettings     : {}
    ShardCount         :

По завершении операции масштабирования значение параметра `ProvisioningState` изменяется на `Succeeded`. Если масштабирование необходимо повторить, например изменить уровень кэша с уровня "Базовый" на уровень "Стандартный", а затем скорректировать его размер, нужно дождаться завершения предыдущей операции. В противном случае возникнет ошибка следующего вида:

    Set-AzRedisCache : Conflict: The resource '...' is not in a stable state, and is currently unable to accept the update request.

## <a name="to-get-information-about-an-azure-cache-for-redis"></a>Получение сведений об использовании кэша Azure для Redis
Вы можете получить сведения об использовании кэша с помощью командлета [Get-AzRedisCache](https://docs.microsoft.com/powershell/module/az.rediscache/get-azrediscache).

Чтобы увидеть список доступных параметров свойства `Get-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help Get-AzRedisCache -detailed

    NAME
        Get-AzRedisCache

    SYNOPSIS
        Gets details about a single cache or all caches in the specified resource group or all caches in the current
        subscription.

    SYNTAX
        Get-AzRedisCache [-Name <String>] [-ResourceGroupName <String>] [<CommonParameters>]

    DESCRIPTION
        The Get-AzRedisCache cmdlet gets the details about a cache or caches depending on input parameters. If both
        ResourceGroupName and Name parameters are provided then Get-AzRedisCache will return details about the
        specific cache name provided.

        If only ResourceGroupName is provided than it will return details about all caches in the specified resource group.

        If no parameters are given than it will return details about all caches the current subscription.

    PARAMETERS
        -Name <String>
            The name of the cache. When this parameter is provided along with ResourceGroupName, Get-AzRedisCache
            returns the details for the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache or caches. If ResourceGroupName is provided with Name
            then Get-AzRedisCache returns the details of the cache specified by Name. If only the ResourceGroup
            parameter is provided, then details for all caches in the resource group are returned.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

Чтобы получить сведения обо всех кэшах в текущей подписке, выполните командлет `Get-AzRedisCache` без параметров.

    Get-AzRedisCache

Чтобы получить сведения обо всех кэшах в определенной группе ресурсов, выполните командлет `Get-AzRedisCache` с параметром `ResourceGroupName`.

    Get-AzRedisCache -ResourceGroupName myGroup

Чтобы получить сведения о конкретном кэше, выполните командлет `Get-AzRedisCache` с параметром `Name`, содержащим имя кэша, и параметром `ResourceGroupName` с группой ресурсов, содержащей этот кэш.

    PS C:\> Get-AzRedisCache -Name myCache -ResourceGroupName myGroup

    Name               : mycache
    Id                 : /subscriptions/12ad12bd-abdc-2231-a2ed-a2b8b246bbad4/resourceGroups/myGroup/providers/Mi
                         crosoft.Cache/Redis/mycache
    Location           : South Central US
    Type               : Microsoft.Cache/Redis
    HostName           : mycache.redis.cache.windows.net
    Port               : 6379
    ProvisioningState  : Succeeded
    SslPort            : 6380
    RedisConfiguration : {[maxmemory-policy, volatile-lru], [maxmemory-reserved, 62], [notify-keyspace-events, KEA],
                         [maxclients, 1000]...}
    EnableNonSslPort   : False
    RedisVersion       : 3.0
    Size               : 1GB
    Sku                : Standard
    ResourceGroupName  : myGroup
    VirtualNetwork     :
    Subnet             :
    StaticIP           :
    TenantSettings     : {}
    ShardCount         :

## <a name="to-retrieve-the-access-keys-for-an-azure-cache-for-redis"></a>Получение ключей доступа к кэшу Azure для Redis
С помощью командлета [Get-AzRedisCacheKey](https://docs.microsoft.com/powershell/module/az.rediscache/Get-azRedisCacheKey) получите ключи доступа к кэшу.

Чтобы увидеть список доступных параметров свойства `Get-AzRedisCacheKey`и их описания, выполните следующую команду:

    PS C:\> Get-Help Get-AzRedisCacheKey -detailed

    NAME
        Get-AzRedisCacheKey

    SYNOPSIS
        Gets the accesskeys for the specified Azure Cache for Redis.


    SYNTAX
        Get-AzRedisCacheKey -Name <String> -ResourceGroupName <String> [<CommonParameters>]

    DESCRIPTION
        The Get-AzRedisCacheKey cmdlet gets the access keys for the specified cache.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

Чтобы получить ключи к кэшу, вызовите командлет `Get-AzRedisCacheKey` и передайте имя кэша и имя группы ресурсов, содержащей этот кэш.

    PS C:\> Get-AzRedisCacheKey -Name myCache -ResourceGroupName myGroup

    PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
    SecondaryKey : ABhfB757JgjIgt785JgKH9865eifmekfnn649303JKL=

## <a name="to-regenerate-access-keys-for-your-azure-cache-for-redis"></a>Повторное создание ключей доступа к кэшу Azure для Redis
С помощью командлета [New-AzRedisCacheKey](https://docs.microsoft.com/powershell/module/az.rediscache/New-azRedisCacheKey) повторно создайте ключи доступа к кэшу.

Чтобы увидеть список доступных параметров свойства `New-AzRedisCacheKey`и их описания, выполните следующую команду:

    PS C:\> Get-Help New-AzRedisCacheKey -detailed

    NAME
        New-AzRedisCacheKey

    SYNOPSIS
        Regenerates the access key of an Azure Cache for Redis.

    SYNTAX
        New-AzRedisCacheKey -Name <String> -ResourceGroupName <String> -KeyType <String> [-Force] [<CommonParameters>]

    DESCRIPTION
        The New-AzRedisCacheKey cmdlet regenerate the access key of an Azure Cache for Redis.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis.

        -ResourceGroupName <String>
            Name of the resource group for the cache.

        -KeyType <String>
            Specifies whether to regenerate the primary or secondary access key. Possible values are Primary or Secondary.

        -Force
            When the Force parameter is provided, the specified access key is regenerated without any confirmation prompts.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

Чтобы повторно создать первичный или вторичный ключ кэша, вызовите командлет `New-AzRedisCacheKey`, передайте имя и группу ресурсов и укажите значение `Primary` или `Secondary` для параметра `KeyType`. В приведенном ниже примере создается вторичный ключ доступа для кэша.

    PS C:\> New-AzRedisCacheKey -Name myCache -ResourceGroupName myGroup -KeyType Secondary

    Confirm
    Are you sure you want to regenerate Secondary key for Azure Cache for Redis 'myCache'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


    PrimaryKey   : b2wdt43sfetlju4hfbryfnregrd9wgIcc6IA3zAO1lY=
    SecondaryKey : c53hj3kh4jhHjPJk8l0jji785JgKH9865eifmekfnn6=

## <a name="to-delete-an-azure-cache-for-redis"></a>Удаление кэша Azure для Redis
С помощью командлета [Remove-AzRedisCache](https://docs.microsoft.com/powershell/module/az.rediscache/remove-azrediscache) удалите кэш Azure для Redis.

Чтобы увидеть список доступных параметров свойства `Remove-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help Remove-AzRedisCache -detailed

    NAME
        Remove-AzRedisCache

    SYNOPSIS
        Remove Azure Cache for Redis if exists.

    SYNTAX
        Remove-AzRedisCache -Name <String> -ResourceGroupName <String> [-Force] [-PassThru] [<CommonParameters>

    DESCRIPTION
        The Remove-AzRedisCache cmdlet removes an Azure Cache for Redis if it exists.

    PARAMETERS
        -Name <String>
            Name of the Azure Cache for Redis to remove.

        -ResourceGroupName <String>
            Name of the resource group of the cache to remove.

        -Force
            When the Force parameter is provided, the cache is removed without any confirmation prompts.

        -PassThru
            By default Remove-AzRedisCache removes the cache and does not return any value. If the PassThru par
            is provided then Remove-AzRedisCache returns a boolean value indicating the success of the operatio

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).

В приведенном ниже примере удаляется кэш с именем `myCache` .

    PS C:\> Remove-AzRedisCache -Name myCache -ResourceGroupName myGroup

    Confirm
    Are you sure you want to remove Azure Cache for Redis 'myCache'?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


## <a name="to-import-an-azure-cache-for-redis"></a>Импорт экземпляра кэша Azure для Redis
Импортировать данные в кэш Azure для Redis можно с помощью командлета `Import-AzRedisCache`.

> [!IMPORTANT]
> Функция импорта и экспорта доступна только для кэшей категории [Премиум](cache-premium-tier-intro.md). Дополнительные сведения об импорте и экспорте см. в статье [Импорт и экспорт данных кэша Azure для Redis](cache-how-to-import-export-data.md).
> 
> 

Чтобы увидеть список доступных параметров свойства `Import-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help Import-AzRedisCache -detailed

    NAME
        Import-AzRedisCache

    SYNOPSIS
        Import data from blobs to Azure Cache for Redis.


    SYNTAX
        Import-AzRedisCache -Name <String> -ResourceGroupName <String> -Files <String[]> [-Format <String>] [-Force]
        [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Import-AzRedisCache cmdlet imports data from the specified blobs into Azure Cache for Redis.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -Files <String[]>
            SAS urls of blobs whose content should be imported into the cache.

        -Format <String>
            Format for the blob.  Currently "rdb" is the only supported, with other formats expected in the future.

        -Force
            When the Force parameter is provided, import will be performed without any confirmation prompts.

        -PassThru
            By default Import-AzRedisCache imports data in cache and does not return any value. If the PassThru
            parameter is provided then Import-AzRedisCache returns a boolean value indicating the success of the
            operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).


Приведенная ниже команда импортирует данные из большого двоичного объекта, указанного в универсальном коде ресурса (URI) SAS, в кэш Azure для Redis.

    PS C:\>Import-AzRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -Files @("https://mystorageaccount.blob.core.windows.net/mycontainername/blobname?sv=2015-04-05&sr=b&sig=caIwutG2uDa0NZ8mjdNJdgOY8%2F8mhwRuGNdICU%2B0pI4%3D&st=2016-05-27T00%3A00%3A00Z&se=2016-05-28T00%3A00%3A00Z&sp=rwd") -Force

## <a name="to-export-an-azure-cache-for-redis"></a>Экспорт кэша Azure для Redis
Экспортировать данные из кэша Azure для Redis можно с помощью командлета `Export-AzRedisCache`.

> [!IMPORTANT]
> Функция импорта и экспорта доступна только для кэшей категории [Премиум](cache-premium-tier-intro.md). Дополнительные сведения об импорте и экспорте см. в статье [Импорт и экспорт данных кэша Azure для Redis](cache-how-to-import-export-data.md).
> 
> 

Чтобы увидеть список доступных параметров свойства `Export-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help Export-AzRedisCache -detailed

    NAME
        Export-AzRedisCache

    SYNOPSIS
        Exports data from Azure Cache for Redis to a specified container.


    SYNTAX
        Export-AzRedisCache -Name <String> -ResourceGroupName <String> -Prefix <String> -Container <String> [-Format
        <String>] [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Export-AzRedisCache cmdlet exports data from Azure Cache for Redis to a specified container.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -Prefix <String>
            Prefix to use for blob names.

        -Container <String>
            SAS url of container where data should be exported.

        -Format <String>
            Format for the blob.  Currently "rdb" is the only supported, with other formats expected in the future.

        -PassThru
            By default Export-AzRedisCache does not return any value. If the PassThru parameter is provided
            then Export-AzRedisCache returns a boolean value indicating the success of the operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).


Приведенная ниже команда экспортирует данные из экземпляра кэша Azure для Redis в контейнер, указанный универсальным кодом ресурса SAS.

        PS C:\>Export-AzRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -Prefix "blobprefix"
        -Container "https://mystorageaccount.blob.core.windows.net/mycontainer?sv=2015-04-05&sr=c&sig=HezZtBZ3DURmEGDduauE7
        pvETY4kqlPI8JCNa8ATmaw%3D&st=2016-05-27T00%3A00%3A00Z&se=2016-05-28T00%3A00%3A00Z&sp=rwdl"

## <a name="to-reboot-an-azure-cache-for-redis"></a>Перезапуск кэша Azure для Redis
Перезапустить экземпляр кэша Azure для Redis можно с помощью командлета `Reset-AzRedisCache`.

> [!IMPORTANT]
> Функция перезагрузки доступна только для кэшей [уровня "Премиум"](cache-premium-tier-intro.md). Дополнительные сведения о перезапуске кэша см. в разделе [Reboot](cache-administration.md#reboot) статьи "Администрирование кэша Redis для Azure".
> 
> 

Чтобы увидеть список доступных параметров свойства `Reset-AzRedisCache`и их описания, выполните следующую команду:

    PS C:\> Get-Help Reset-AzRedisCache -detailed

    NAME
        Reset-AzRedisCache

    SYNOPSIS
        Reboot specified node(s) of an Azure Cache for Redis instance.


    SYNTAX
        Reset-AzRedisCache -Name <String> -ResourceGroupName <String> -RebootType <String> [-ShardId <Integer>]
        [-Force] [-PassThru] [<CommonParameters>]


    DESCRIPTION
        The Reset-AzRedisCache cmdlet reboots the specified node(s) of an Azure Cache for Redis instance.


    PARAMETERS
        -Name <String>
            The name of the cache.

        -ResourceGroupName <String>
            The name of the resource group that contains the cache.

        -RebootType <String>
            Which node to reboot. Possible values are "PrimaryNode", "SecondaryNode", "AllNodes".

        -ShardId <Integer>
            Which shard to reboot when rebooting a premium cache with clustering enabled.

        -Force
            When the Force parameter is provided, reset will be performed without any confirmation prompts.

        -PassThru
            By default Reset-AzRedisCache does not return any value. If the PassThru parameter is provided
            then Reset-AzRedisCache returns a boolean value indicating the success of the operation.

        <CommonParameters>
            This cmdlet supports the common parameters: Verbose, Debug,
            ErrorAction, ErrorVariable, WarningAction, WarningVariable,
            OutBuffer, PipelineVariable, and OutVariable. For more information, see
            about_CommonParameters (https://go.microsoft.com/fwlink/?LinkID=113216).


Перезапустить оба узла указанного кэша можно с помощью такой команды:

        PS C:\>Reset-AzRedisCache -ResourceGroupName "resourceGroupName" -Name "cacheName" -RebootType "AllNodes"
        -Force


## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об использовании Windows PowerShell с Azure см. в следующих ресурсах:

* [Документация по командлету кэша Azure для Redis на MSDN](https://docs.microsoft.com/powershell/module/az.rediscache)
* [Overview of Azure PowerShell](https://go.microsoft.com/fwlink/?LinkID=394765) (Общие сведения об Azure PowerShell) — сведения об использовании командлетов в модуле Azure Resource Manager.
* [Развертывание ресурсов с использованием шаблонов Resource Manager и портала Azure](../azure-resource-manager/resource-group-template-deploy-portal.md) — сведения о создании групп ресурсов и управлении ими на портале Azure.
* [Блог Azure](https://azure.microsoft.com/blog/) — сведения о новых возможностях в Azure.
* [Блог Windows PowerShell](https://blogs.msdn.com/powershell) — сведения о новых возможностях в Windows PowerShell.
* [Блог "Hey, Scripting Guy!"](https://blogs.technet.com/b/heyscriptingguy/) — реальные советы и рекомендации от сообщества Windows PowerShell.

