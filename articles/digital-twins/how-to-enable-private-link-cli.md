---
title: Включение частного доступа с помощью закрытой ссылки (Предварительная версия) — CLI
titleSuffix: Azure Digital Twins
description: Сведения о том, как включить частный доступ для решений Digital двойников для Azure с помощью частного канала, см. на Azure CLI.
author: baanders
ms.author: baanders
ms.date: 02/09/2021
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 5bd7ffda508980a9a56d86037887fc53a0fed640
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102202949"
---
# <a name="enable-private-access-with-private-link-preview-azure-cli"></a>Включить частный доступ с помощью частной ссылки (Предварительная версия): Azure CLI

[!INCLUDE [digital-twins-private-link-selector.md](../../includes/digital-twins-private-link-selector.md)]

В этой статье описываются различные способы [включения закрытой ссылки с закрытой конечной точкой для экземпляра Digital двойников Azure](concepts-security.md#private-network-access-with-azure-private-link-preview) (сейчас в предварительной версии). Настройка частной конечной точки для вашего экземпляра Azure Digital двойников позволяет защитить экземпляр Azure Digital двойников и исключить открытую раскрытие, а также исключить утечка данных из [виртуальной сети Azure (VNet)](../virtual-network/virtual-networks-overview.md).

В этой статье описывается процесс, использующий [**Azure CLI**](/cli/azure/what-is-azure-cli).

Ниже приведены действия, описанные в этой статье. 
1. Включите параметр частная связь и настройте частную конечную точку для экземпляра цифрового двойников Azure.
1. Отключите или включите флаги доступа к общедоступной сети, чтобы ограничить доступ через API только к частным каналам соединения.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем можно будет настроить частную конечную точку, вам потребуется [**Виртуальная сеть Azure**](../virtual-network/virtual-networks-overview.md) , в которой можно развернуть конечную точку. Если у вас еще нет виртуальной сети, для ее настройки можно воспользоваться одним из [кратких](../virtual-network/quick-create-portal.md) руководств по виртуальной сети Azure.

## <a name="manage-private-endpoints-for-an-azure-digital-twins-instance"></a>Управление частными конечными точками для экземпляра цифрового двойников Azure 

При использовании [Azure CLI](/cli/azure/what-is-azure-cli)можно настроить частные конечные точки с помощью закрытой ссылки на экземпляре двойников Azure Digital, который уже существует (он не может быть добавлен в процессе создания экземпляра). Затем можно продолжить просмотр и управление ими с помощью дополнительных команд интерфейса командной строки. 

>[!TIP]
> Вы также можете настроить конечную точку частной связи через службу Private Link, а не через экземпляр Digital двойников для Azure. Это также дает те же параметры конфигурации и тот же конечный результат.
>
> Дополнительные сведения о настройке ресурсов частной связи см. в документации по частным ссылкам для [портал Azure](../private-link/create-private-endpoint-portal.md), [Azure CLI](../private-link/create-private-endpoint-cli.md), [шаблонов ARM](../private-link/create-private-endpoint-template.md)или [PowerShell](../private-link/create-private-endpoint-powershell.md).

### <a name="add-a-private-endpoint-to-an-existing-instance"></a>Добавление частной конечной точки в существующий экземпляр

Чтобы создать частную конечную точку и связать ее с экземпляром Digital двойников Azure, используйте команду [**AZ Network Private-Endpoint Create**](/cli/azure/network/private-endpoint#az_network_private_endpoint_create) . Идентификация экземпляра Azure Digital двойников с помощью его полного идентификатора в `--private-connection-resource-id` параметре.

Ниже приведен пример использования команды для создания частной конечной точки с только необходимыми параметрами.

```azurecli-interactive
az network private-endpoint create --connection-name {private_link_service_connection} -n {name_for_private_endpoint} -g {resource_group} --subnet {subnet_ID} --private-connection-resource-id "/subscriptions/{subscription_ID}/resourceGroups/{resource_group}/providers/Microsoft.DigitalTwins/digitalTwinsInstances/{Azure_Digital_Twins_instance_name}" 
```

Полный список обязательных и необязательных параметров, а также дополнительные примеры создания частных конечных точек см. в [справочной документации по команде **AZ Network Private-Endpoint Create**](/cli/azure/network/private-endpoint#az_network_private_endpoint_create).

### <a name="manage-private-endpoint-connections-on-the-instance"></a>Управление подключениями к частным конечным точкам в экземпляре

После создания частной конечной точки для своего экземпляра Azure Digital двойников можно использовать команды [**AZ DT Network Private-Endpoint Connection**](/cli/azure/ext/azure-iot/dt/network/private-endpoint/connection) , чтобы продолжить управление **подключениями** к частным конечным точкам по отношению к экземпляру. К этим операциям относятся:
* Отображение частного подключения к конечной точке
* Задание состояния подключения к закрытой конечной точке
* Удаление подключения к закрытой конечной точке
* Вывод списка всех подключений к закрытой конечной точке для экземпляра

Дополнительные сведения и примеры см. в [справочной документации по **AZ DT для сети с частной конечной точкой**](/cli/azure/ext/azure-iot/dt/network/private-endpoint).

### <a name="manage-other-private-link-information-on-an-azure-digital-twins-instance"></a>Управление другими сведениями о личных ссылках в экземпляре цифрового двойников Azure

Дополнительные сведения о состоянии закрытой связи для экземпляра можно получить с помощью команд [**частной связи "az DT Network**](/cli/azure/ext/azure-iot/dt/network/private-link) ". К этим операциям относятся:
* Список частных ссылок, связанных с экземпляром цифрового двойников Azure
* Отображение частной ссылки, связанной с экземпляром

Дополнительные сведения и примеры см. в [справочной документации по команде **AZ DT Network Link**](/cli/azure/ext/azure-iot/dt/network/private-link).

## <a name="disable--enable-public-network-access-flags"></a>Отключить или включить флаги доступа к общедоступной сети

Вы можете настроить экземпляр Azure Digital двойников, чтобы запретить все открытые подключения и разрешить подключения только через частные конечные точки, чтобы повысить безопасность сети. Это действие выполняется с помощью **общего флага доступа к сети**. 

Эта политика позволяет ограничить доступ через API только к частным каналам соединения. Если для флага доступа к общедоступной сети задано значение *отключено*, все REST APIные вызовы к плоскости данных экземпляра Azure Digital двойников из общедоступного облака будут возвращать `403, Unauthorized` . Кроме того, если для политики задано значение *отключено* и запрос выполняется через закрытую конечную точку, вызов API будет выполнен.

В этой статье показано, как обновить значение сетевого флага с помощью [Azure CLI](/cli/azure/) или [средства командной строки ARMClient](https://github.com/projectkudu/ARMClient). Инструкции по выполнению с портал Azure см. в [версии портала](how-to-enable-private-link-portal.md) этой статьи.

### <a name="use-the-azure-cli"></a>Использование Azure CLI

В Azure CLI можно отключить или включить доступ к общедоступной сети, добавив `--public-network-access` параметр в `az dt create` команду. Хотя эта команда также может использоваться для создания нового экземпляра, ее можно использовать для изменения свойств существующего экземпляра, предоставив ему имя уже существующего экземпляра. (Дополнительные сведения об этой команде см. в [справочной документации](/cli/azure/ext/azure-iot/dt#ext_azure_iot_az_dt_create) или [общих инструкциях по настройке экземпляра Azure Digital двойников](how-to-set-up-instance-cli.md#create-the-azure-digital-twins-instance)).

Чтобы **Отключить** доступ к общедоступной сети для экземпляра Digital двойников Azure, используйте `--public-network-access` параметр следующим образом:

```azurecli-interactive
az dt create -n {name_of_existing_instance} -g {resource_group} --public-network-access Disabled
```

Чтобы **включить** доступ к общедоступной сети в экземпляре, в котором он сейчас отключен, используйте следующую команду:

```azurecli-interactive
az dt create -n {name_of_existing_instance} -g {resource_group} --public-network-access Enabled
```

### <a name="usethe-armclientcommand-tool"></a>Использование средства командной строки ARMClient 

С помощью [средства командной строки ARMClient](https://github.com/projectkudu/ARMClient)доступ к общедоступной сети включается или отключается с использованием приведенных ниже команд. 

Чтобы **Отключить** доступ к общедоступной сети, выполните следующие действия.
  
```cmd/sh
armclient login 

armclient PATCH /subscriptions/<your-Azure-subscription-ID>/resourceGroups/<your-resource-group>/providers/Microsoft.DigitalTwins/digitalTwinsInstances/<your-Azure-Digital-Twins-instance>?api-version=2020-12-01 "{ 'properties': { 'publicNetworkAccess': 'disabled' } }"  
```

Чтобы **включить** доступ к общедоступной сети, выполните следующие действия.  
  
```cmd/sh
armclient login 

armclient PATCH /subscriptions/<your-Azure-subscription-ID>/resourceGroups/<your-resource-group>/providers/Microsoft.DigitalTwins/digitalTwinsInstances/<your-Azure-Digital-Twins-instance>?api-version=2020-12-01 "{ 'properties': { 'publicNetworkAccess': 'enabled' } }"  
``` 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о закрытой ссылке для Azure: 
* [*Что такое служба частной связи Azure?*](../private-link/private-link-service-overview.md)