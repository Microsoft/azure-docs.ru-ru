---
title: Управление кэшем Azure для Redis с помощью классической инфраструктуры интерфейса командной строки Azure
description: Узнайте, как установить классический Azure CLI на любой платформе, как использовать его для подключения к учетной записи Azure и как с его помощью создать кэш Azure для Redis и управлять им.
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 01/23/2017
ms.author: yegu
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7643f882d5ac330046c169e0a3f2fa4920331d4e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92537700"
---
# <a name="how-to-create-and-manage-azure-cache-for-redis-using-the-azure-classic-cli"></a>Создание кэша Azure для Redis и управление им с помощью классического Azure CLI
> [!div class="op_single_selector"]
> * [PowerShell](cache-how-to-manage-redis-cache-powershell.md)
> * [Классический интерфейс командной строки Azure](cache-manage-cli.md)
>

Классический Azure CLI позволяет управлять инфраструктурой Azure с любой платформы. В этой статье показано, как создавать экземпляры кэша Azure для Redis и управлять ими с помощью классического Azure CLI.

[!INCLUDE [outdated-cli-content](../../includes/contains-classic-cli-content.md)]
> [!NOTE]
> Последние примеры сценариев Azure CLI для кэша Azure для Redis см. в [этой статье](cli-samples.md).

## <a name="prerequisites"></a>Предварительные условия
Для создания экземпляров кэша Azure для Redis и управления ими с помощью классического Azure CLI необходимо выполнить следующие действия.

* Необходимо иметь учетную запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за пару минут.
* [Установите классический интерфейс командной строки Azure](/cli/azure/install-classic-cli).
* Подключите установленный Azure CLI к личной либо рабочей или учебной учетной записи Azure, а затем выполните вход из классического Azure CLI с помощью команды `azure login`.
* Перед выполнением любой из указанных ниже команд переключите классический Azure CLI в режим диспетчера ресурсов, выполнив команду `azure config mode arm`. Дополнительные сведения см. в разделе [Управление ресурсами и группами ресурсов Azure с помощью классического Azure CLI](../azure-resource-manager/management/manage-resources-cli.md).

## <a name="azure-cache-for-redis-properties"></a>Свойства кэша Azure для Redis
При создании и обновлении экземпляров кэша Azure для Redis используются следующие свойства.

| Свойство. | Параметр | Описание |
| --- | --- | --- |
| name |-n, --name |Имя кэша Azure для Redis. |
| resource group |-g, --resource-group |Имя группы ресурсов. |
| location |-l, --location |Расположение для создания кэша. |
| size |-z, --size |Размер кэша Azure для Redis. Допустимые значения: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4] |
| sku |-x, --sku |Номер SKU Redis. Должен иметь одно из этих значений: [Basic, Standard, Premium] |
| EnableNonSslPort |-e, --enable-non-ssl-port |Свойство EnableNonSslPort кэша Azure для Redis. Добавьте этот флаг, если вы хотите включить для кэша порт, не поддерживающий протокол TLS/SSL. |
| Конфигурация Redis |-c, --redis-configuration |Конфигурация Redis. Введите строку ключей и значений конфигурации в формате JSON. Формат:"{"":"","":""}" |
| Конфигурация Redis |-f, --redis-configuration-file |Конфигурация Redis. Введите путь к файлу, содержащему ключи и значения. Формат записи в файле: {"":"","":""} |
| Число сегментов |-r, --shard-count |Число сегментов, которые будут созданы при создании кэша уровня "Премиум" с включенной кластеризацией. |
| Виртуальная сеть |-v, --virtual-network |При размещении кэша в виртуальной сети определяет точный идентификатор ресурса ARM виртуальной сети, в которой будет развернут кэш Azure для Redis. Пример формата: /subscriptions/{идентификатор_подписки}/resourceGroups/{имя_группы_ресурсов}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| key type |-t, --key-type |Тип обновляемого ключа. Допустимые значения: [Primary, Secondary]. |
| StaticIP |-p,--Static-IP \<static-ip\> |При размещении кэша в виртуальной сети определяет уникальный IP-адрес подсети для кэша. Если IP-адрес не указан, он автоматически выбирается из подсети. |
| Подсеть |t, --subnet \<subnet\> |При размещении кэша в виртуальной сети определяет имя подсети, в которой будет развернут кэш. |
| Виртуальная сеть |-v,--виртуальная сеть \<virtual-network\> |При размещении кэша в виртуальной сети определяет точный идентификатор ресурса ARM виртуальной сети, в которой будет развернут кэш Azure для Redis. Пример формата: /subscriptions/{идентификатор_подписки}/resourceGroups/{имя_группы_ресурсов}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| Подписка |-s, --subscription |Идентификатор подписки. |

## <a name="see-all-azure-cache-for-redis-commands"></a>Просмотр всех команд кэша Azure для Redis
Для просмотра всех команд кэша Azure для Redis и их параметров используйте команду `azure rediscache -h`.

```azurecli
C:\>azure rediscache -h
help:    Commands to manage your Azure Cache for Redis(s)
help:
help:    Create an Azure Cache for Redis
help:      rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
help:
help:    Delete an existing Azure Cache for Redis
help:      rediscache delete [--name <name> --resource-group <resource-group> ]
help:
help:    List all Azure Cache for Redis within your Subscription or Resource Group
help:      rediscache list [options]
help:
help:    Show properties of an existing Azure Cache for Redis
help:      rediscache show [--name <name> --resource-group <resource-group>]
help:
help:    Change settings of an existing Azure Cache for Redis
help:      rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
help:
help:    Renew the authentication key for an existing Azure Cache for Redis
help:      rediscache renew-key [--name <name> --resource-group <resource-group> ]
help:
help:    Lists Primary and Secondary key of an existing Azure Cache for Redis
help:      rediscache list-keys [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help  output usage information
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="create-an-azure-cache-for-redis"></a>Создание экземпляра кэша Azure для Redis
Чтобы создать кэш Azure для Redis, используйте следующую команду:

```azurecli
    azure rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
```

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache create -h`.

```azurecli
C:\>azure rediscache create -h
help:    Create an Azure Cache for Redis
help:
help:    Usage: rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
help:
help:    Options:
help:      -h, --help                                               output usage information
help:      -v, --verbose                                            use verbose output
help:      -vv                                                      more verbose with debug output
help:      --json                                                   use json output
help:      -n, --name <name>                                        Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>                    Name of the Resource Group
help:      -l, --location <location>                                Location to create cache.
help:      -z, --size <size>                                        Size of the Azure Cache for Redis. Valid values: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4]
help:      -x, --sku <sku>                                          Redis SKU. Should be one of : [Basic, Standard, Premium]
help:      -e, --enable-non-ssl-port                                EnableNonSslPort property of the Azure Cache for Redis. Add this flag if you want to enable the non-TLS/SSL Port for your cache
help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here. Format:"{"<key1>":"<value1>","<key2>":"<value2>"}"
help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here. Format for the file entry: {"<key1>":"<value1>","<key2>":"<value2>"}
help:      -r, --shard-count <shard-count>                          Number of Shards to create on a Premium Cluster Cache
help:      -v, --virtual-network <virtual-network>                  The exact ARM resource ID of the virtual network to deploy the Azure Cache for Redis in. Example format: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1
help:      -t, --subnet <subnet>                                    Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network
help:      -p, --static-ip <static-ip>                              Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network
help:      -s, --subscription <id>                                  the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="delete-an-existing-azure-cache-for-redis"></a>Удаление имеющегося кэша Azure для Redis
Чтобы удалить кэш Azure для Redis, используйте следующую команду:

```azurecli
    azure rediscache delete [--name <name> --resource-group <resource-group> ]
```

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache delete -h`.

```azurecli
C:\>azure rediscache delete -h
help:    Delete an existing Azure Cache for Redis
help:
help:    Usage: rediscache delete [--name <name> --resource-group <resource-group> ]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which the cache exists
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="list-all-azure-cache-for-redis-within-your-subscription-or-resource-group"></a>Вывод списка всех кэшей Azure для Redis в подписке или группе ресурсов
Чтобы вывести список всех кэшей Azure для Redis в подписке или группе ресурсов, выполните следующую команду.

```azurecli
    azure rediscache list [options]
```

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache list -h`.

```azurecli
C:\>azure rediscache list -h
help:    List all Azure Cache for Redis within your Subscription or Resource Group
help:
help:    Usage: rediscache list [options]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -g, --resource-group <resource-group>  Name of the Resource Group
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="show-properties-of-an-existing-azure-cache-for-redis"></a>Отображение свойств имеющегося кэша Azure для Redis
Чтобы отобразить свойства имеющегося кэша Azure для Redis, используйте следующую команду:

```azurecli
    azure rediscache show [--name <name> --resource-group <resource-group>]
```

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache show -h`.

```azurecli
C:\>azure rediscache show -h
help:    Show properties of an existing Azure Cache for Redis
help:
help:    Usage: rediscache show [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

<a name="scale"></a>

## <a name="change-settings-of-an-existing-azure-cache-for-redis"></a>Изменение параметров имеющегося кэша Azure для Redis
Чтобы изменить параметры имеющегося кэша Azure для Redis, используйте следующую команду:

```azurecli
    azure rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
```

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache set -h`.

```azurecli
C:\>azure rediscache set -h
help:    Change settings of an existing Azure Cache for Redis
help:
help:    Usage: rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
help:
help:    Options:
help:      -h, --help                                               output usage information
help:      -v, --verbose                                            use verbose output
help:      -vv                                                      more verbose with debug output
help:      --json                                                   use json output
help:      -n, --name <name>                                        Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>                    Name of the Resource Group
help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here.
help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here.
help:      -s, --subscription <subscription>                        the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="renew-the-authentication-key-for-an-existing-azure-cache-for-redis"></a>Обновление ключа аутентификации для имеющегося кэша Azure для Redis
Чтобы обновить ключ аутентификации для имеющегося кэша Azure для Redis, используйте следующую команду:

```azurecli
    azure rediscache renew-key [--name <name> --resource-group <resource-group> --key-type <key-type>]
```

Укажите `Primary` или `Secondary` для `key-type`.

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache renew-key -h`.

```azurecli
C:\>azure rediscache renew-key -h
help:    Renew the authentication key for an existing Azure Cache for Redis
help:
help:    Usage: rediscache renew-key [--name <name> --resource-group <resource-group> ]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which cache exists
help:      -t, --key-type <key-type>              type of key to renew. Valid values are: 'Primary', 'Secondary'.
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="list-primary-and-secondary-keys-of-an-existing-azure-cache-for-redis"></a>Вывод первичного и вторичного ключей имеющегося кэша Azure для Redis
Чтобы вывести первичный и вторичный ключи имеющегося кэша Azure для Redis, используйте следующую команду:

```azurecli
    azure rediscache list-keys [--name <name> --resource-group <resource-group>]
```

Чтобы получить дополнительные сведения об этой команде, выполните команду `azure rediscache list-keys -h`.

```azurecli
C:\>azure rediscache list-keys -h
help:    Lists Primary and Secondary key of an existing Azure Cache for Redis
help:
help:    Usage: rediscache list-keys [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which Cache exists
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```