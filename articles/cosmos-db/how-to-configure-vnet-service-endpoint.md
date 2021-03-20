---
title: Настройка доступа на основе виртуальной сети для учетной записи Azure Cosmos
description: В этом документе описываются шаги, необходимые для настройки конечной точки службы для виртуальной сети для Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/13/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1d63d21f4c49e3c7aef035208477ac9fc79f2e51
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94637189"
---
# <a name="configure-access-to-azure-cosmos-db-from-virtual-networks-vnet"></a>Настройка доступа к Azure Cosmos DB из виртуальных сетей (VNet)
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Учетные записи Azure Cosmos можно настроить для предоставления доступа только из определенной подсети виртуальной сети. Если включить [конечную точку службы](../virtual-network/virtual-network-service-endpoints-overview.md) для доступа к Azure Cosmos DB в подсети в виртуальной сети, трафик из этой подсети отправляется в базу данных Azure Cosmos DB с удостоверением подсети и виртуальной сети. После включения конечной точки службы Azure Cosmos DB можно ограничить доступ к подсети, добавив ее в учетную запись Azure Cosmos.

По умолчанию доступ к учетной записи Azure Cosmos можно получить через любой источник при условии, что запрос сопровождается допустимым маркером проверки подлинности. При добавлении одной или нескольких подсетей в виртуальную сеть только запросы, полученные из этих подсетей, будут получать правильный ответ. Запросы, полученные из любого другого источника, получат ответ "403 (запрещено)". 

Учетные записи Azure Cosmos DB можно настроить для предоставления доступа только из определенной подсети виртуальной сети Azure. Для ограничения доступа к учетной записи Azure Cosmos DB с подключениями из подсети в виртуальной сети выполните следующие шаги:

1. Включите в подсети отправку удостоверения подсети и виртуальной сети в Azure Cosmos DB. Это можно сделать, включив конечную точку службы для Azure Cosmos DB в определенной подсети.

1. Добавьте правило в учетной записи Azure Cosmos DB, определяющее подсеть как источник, из которого можно получить доступ к учетной записи.

> [!NOTE]
> После того как конечная точка службы для учетной записи Azure Cosmos DB будет включена в подсети, источник трафика, подключающийся к Azure Cosmos DB, переключится с общего IP-адреса на виртуальную сеть и подсеть. Переключение трафика применяется к любой учетной записи Azure Cosmos DB, доступ к которой осуществляется из этой подсети. Если в ваших учетных записях Azure Cosmos DB используется брандмауэр на основе IP-адреса для разрешения этой подсети, запросы из подсети с поддержкой службы больше не будут соответствовать правилам брандмауэра для IP-адресов, поэтому будут отклонены.
>
> Дополнительные сведения см. в разделе [Переход от правила брандмауэра для IP-адресов к списку управления доступом виртуальной сети](#migrate-from-firewall-to-vnet) этой статьи.

В следующих разделах описано, как настроить конечную точку службы для виртуальной сети для учетной записи Azure Cosmos DB.

## <a name="configure-a-service-endpoint-by-using-the-azure-portal"></a><a id="configure-using-portal"></a>Настройка конечной точки службы с помощью портала Azure

### <a name="configure-a-service-endpoint-for-an-existing-azure-virtual-network-and-subnet"></a>Настройка конечной точки службы для существующей виртуальной сети Azure и подсети

1. В колонке **Все ресурсы** найдите учетную запись Azure Cosmos DB, которую требуется защитить.

1. В меню параметров выберите **Брандмауэры и виртуальные сети**, затем в разделе **Выбранные сети** разрешите доступ.

1. Чтобы предоставить доступ к подсети имеющейся виртуальной сети, в разделе **Виртуальные сети** выберите **Add existing Azure virtual network** (Добавление имеющейся виртуальной сети Azure).

1. Выберите **подписку**, из которой требуется добавить виртуальную сеть Azure. Выберите **виртуальную сеть** и **подсети** Azure, которым необходимо предоставить доступ к учетной записи Azure Cosmos DB. Затем выберите **Включить**, чтобы включить выбранные сети с конечными точками службы для Microsoft.AzureCosmosDB. По завершении нажмите кнопку **Добавить**.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/choose-subnet-and-vnet.png" alt-text="Выбор виртуальной сети и подсети":::

1. После настройки доступа к учетной записи Azure Cosmos DB из виртуальной сети будет разрешен трафик только из этой выбранной подсети. Добавленная виртуальная сеть и подсеть должны выглядеть так, как показано на следующем снимке экрана:

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/vnet-and-subnet-configured-successfully.png" alt-text="Успешная настройка виртуальной сети и подсети":::

> [!NOTE]
> Чтобы включить конечные точки служб для виртуальной сети, потребуются следующие права доступа к подписке:
>   * подписка с виртуальной сетью — участник сетей;
>   * подписка с учетной записью Azure Cosmos DB — участник учетной записи DocumentDB.
>   * Если ваша виртуальная сеть и учетная запись Azure Cosmos DB находятся в разных подписках, убедитесь, что в подписке с виртуальной сетью также `Microsoft.DocumentDB` зарегистрирован поставщик ресурсов. Сведения о регистрации поставщика ресурсов см. в статье [поставщики и типы ресурсов Azure](../azure-resource-manager/management/resource-providers-and-types.md) .

Ниже приведены инструкции по регистрации подписки с помощью поставщика ресурсов.

### <a name="configure-a-service-endpoint-for-a-new-azure-virtual-network-and-subnet"></a>Настройка конечной точки службы для новой виртуальной сети и подсети Azure

1. В колонке **Все ресурсы** найдите учетную запись Azure Cosmos DB, которую требуется защитить.  

1. В меню параметров выберите **Брандмауэры и виртуальные сети Azure**, затем в разделе **Выбранные сети** разрешите доступ.  

1. Чтобы предоставить доступ к новой виртуальной сети Azure, в разделе **Виртуальные сети** выберите **Добавить новую виртуальную сеть**.  

1. Укажите сведения, необходимые для создания новой виртуальной сети, а затем выберите **Создать**. Подсеть будет создана с конечной точкой службы для включенной "Microsoft.AzureCosmosDB".

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/choose-subnet-and-vnet-new-vnet.png" alt-text="Выбор виртуальной сети и подсети для новой виртуальной сети":::

Если учетная запись Azure Cosmos DB используется другими службами Azure, такими как Azure Когнитивный поиск, или доступ к ней осуществляется из Stream Analytics или Power BI, вы разрешаете доступ, выбрав **принять подключения из в глобальных центрах обработки данных Azure**.

Чтобы иметь доступ к метрикам Azure Cosmos DB с портала, необходимо включить параметр **Разрешение доступа с портала Azure**. Дополнительные сведения об этих параметрах см. в статье [Настройка брандмауэра IP-адресов](how-to-configure-firewall.md). После включения доступа выберите **Сохранить** для сохранения настроек.

## <a name="remove-a-virtual-network-or-subnet"></a><a id="remove-vnet-or-subnet"></a>Удаление виртуальной сети или подсети

1. В колонке **Все ресурсы** найдите учетную запись Azure Cosmos DB, для которой назначены конечные точки службы.  

1. Выберите **Брандмауэры и виртуальные сети** в меню параметров.  

1. Чтобы удалить правило подсети или виртуальной сети, выберите **...** рядом с виртуальной сетью или подсетью и нажмите **Удалить**.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/remove-a-vnet.png" alt-text="Удаление виртуальной сети":::

1. Выберите **Сохранить**, чтобы применить изменения.

## <a name="configure-a-service-endpoint-by-using-azure-powershell"></a><a id="configure-using-powershell"></a>Настройка конечной точки службы с помощью Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для настройки конечной точки службы для учетной записи Azure Cosmos DB с помощью Azure PowerShell выполните следующие действия:  

1. Установите [Azure PowerShell](/powershell/azure/install-Az-ps) и [выполните вход](/powershell/azure/authenticate-azureps).  

1. Включите конечную точку службы для существующей подсети виртуальной сети.  

   ```powershell
   $resourceGroupName = "<Resource group name>"
   $vnetName = "<Virtual network name>"
   $subnetName = "<Subnet name>"
   $subnetPrefix = "<Subnet address range>"
   $serviceEndpoint = "Microsoft.AzureCosmosDB"

   Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName | Set-AzVirtualNetworkSubnetConfig `
      -Name $subnetName `
      -AddressPrefix $subnetPrefix `
      -ServiceEndpoint $serviceEndpoint | Set-AzVirtualNetwork
   ```

   > [!NOTE]
   > Если вы используете PowerShell или интерфейс командной строки Azure, не забудьте указать полный список фильтров IP-адресов и ACL виртуальной сети в параметрах, а не только те, которые должны быть добавлены.

1. Получите сведения о виртуальной сети.

   ```powershell
   $vnet = Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName

   $subnetId = $vnet.Id + "/subnets/" + $subnetName
   ```

1. Подготовка правила виртуальной сети Cosmos DB

   ```powershell
   $vnetRule = New-AzCosmosDBVirtualNetworkRule `
      -Id $subnetId
   ```

1. Обновите свойства учетной записи Azure Cosmos DB с помощью новой конфигурации конечной точки виртуальной сети: 

   ```powershell
   $accountName = "<Cosmos DB account name>"

   Update-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName `
      -EnableVirtualNetwork $true `
      -VirtualNetworkRuleObject @($vnetRule)
   ```

1. Выполните следующую команду, чтобы убедиться, что ваша учетная запись Azure Cosmos DB обновлена с конечной точкой службы для виртуальной сети, настроенной на предыдущем этапе:

   ```powershell
   $account = Get-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName

   $account.IsVirtualNetworkFilterEnabled
   $account.VirtualNetworkRules
   ```

## <a name="configure-a-service-endpoint-by-using-the-azure-cli"></a><a id="configure-using-cli"></a>Настройка конечной точки службы с помощью интерфейса командной строки Azure

Учетные записи Azure Cosmos можно настроить для конечных точек службы, когда они будут созданы или обновлены позже, если для них уже настроена подсеть. Конечные точки службы также можно включить в учетной записи Cosmos, где для них еще не настроена подсеть, а затем начнет работать, когда подсеть будет настроена позже. Эта гибкость позволяет администраторам, которые не имеют доступа к ресурсам учетной записи Cosmos и виртуальной сети, делать свои конфигурации независимыми друг от друга.

### <a name="create-a-new-cosmos-account-and-connect-it-to-a-back-end-subnet-for-a-new-virtual-network"></a>Создайте новую учетную запись Cosmos и подключите ее к внутренней подсети для новой виртуальной сети.

В этом примере виртуальная сеть и подсеть создаются с помощью конечных точек службы, которые будут включены при создании.

```azurecli-interactive
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet

# Resource group and Cosmos account variables
resourceGroupName='MyResourceGroup'
location='West US 2'
accountName='mycosmosaccount'

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
   -n $vnetName \
   -g $resourceGroupName \
   --address-prefix 10.0.0.0/16 \
   --subnet-name $frontEnd \
   --subnet-prefix 10.0.1.0/24

# Create a back-end subnet with service endpoints enabled for Cosmos DB
az network vnet subnet create \
   -n $backEnd \
   -g $resourceGroupName \
   --address-prefix 10.0.2.0/24 \
   --vnet-name $vnetName \
   --service-endpoints Microsoft.AzureCosmosDB

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values and service endpoints
az cosmosdb create \
   -n $accountName \
   -g $resourceGroupName \
   --enable-virtual-network true \
   --virtual-network-rules $svcEndpoint
```

### <a name="connect-and-configure-a-cosmos-account-to-a-back-end-subnet-independently"></a>Подключение и настройка учетной записи Cosmos для внутренней подсети

В этом примере показано, как подключить учетную запись Azure Cosmos к существующей новой виртуальной сети, в которой подсеть еще не настроена для конечных точек службы. Это делается с помощью `--ignore-missing-vnet-service-endpoint` параметра. Это позволяет завершить настройку учетной записи Cosmos без ошибок до завершения настройки подсети виртуальной сети. После завершения настройки подсети учетная запись Cosmos будет доступна через настроенную подсеть.

```azurecli-interactive
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet
# that is not yet enabled for service endpoints.

# Resource group and Cosmos account variables
resourceGroupName='MyResourceGroup'
location='West US 2'
accountName='mycosmosaccount'

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
   -n $vnetName \
   -g $resourceGroupName \
   --address-prefix 10.0.0.0/16 \
   --subnet-name $frontEnd \
   --subnet-prefix 10.0.1.0/24

# Create a back-end subnet but without configuring service endpoints (--service-endpoints Microsoft.AzureCosmosDB)
az network vnet subnet create \
   -n $backEnd \
   -g $resourceGroupName \
   --address-prefix 10.0.2.0/24 \
   --vnet-name $vnetName

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values
az cosmosdb create -n $accountName -g $resourceGroupName

# Add the virtual network rule but ignore the missing service endpoint on the subnet
az cosmosdb network-rule add \
   -n $accountName \
   -g $resourceGroupName \
   --virtual-network $vnetName \
   --subnet svcEndpoint \
   --ignore-missing-vnet-service-endpoint true

read -p'Press any key to now configure the subnet for service endpoints'

az network vnet subnet update \
   -n $backEnd \
   -g $resourceGroupName \
   --vnet-name $vnetName \
   --service-endpoints Microsoft.AzureCosmosDB
```

## <a name="port-range-when-using-direct-mode"></a>Диапазон портов при использовании прямого режима

При использовании конечных точек службы с учетной записью Azure Cosmos через подключение в режиме прямого подключения необходимо убедиться, что в диапазоне TCP-портов от 10000 до 20000 открыт.

## <a name="migrating-from-an-ip-firewall-rule-to-a-virtual-network-acl"></a><a id="migrate-from-firewall-to-vnet"></a>Переход от использования правила брандмауэра для IP-адресов к использованию ACL виртуальной сети

Чтобы перенести учетную запись Azure Cosmos DB из использования правил брандмауэра IP-адресов для использования конечных точек службы виртуальной сети, выполните следующие действия.

После того как учетная запись Azure Cosmos DB настроена для конечной точки службы в подсети, запросы из этой подсети отправляются в Azure Cosmos DB с использованием сведений об источнике виртуальной сети и подсети вместо общедоступного IP-адреса источника. Эти запросы больше не будут соответствовать IP-фильтру, настроенному для учетной записи Azure Cosmos DB, поэтому необходимо выполнить следующие действия, чтобы избежать простоев.

Прежде чем продолжать, включите конечную точку службы Azure Cosmos DB в виртуальной сети и подсети, выполнив описанный выше шаг в статье Включение конечной точки службы для существующей подсети виртуальной сети.

1. Получение сведений о виртуальной сети и подсети.

   ```powershell
   $resourceGroupName = "myResourceGroup"
   $accountName = "mycosmosaccount"
   $vnetName = "myVnet"
   $subnetName = "mySubnet"

   $vnet = Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName

   $subnetId = $vnet.Id + "/subnets/" + $subnetName
   ```

1. Подготовка нового объекта правила виртуальной сети для учетной записи Azure Cosmos DB:

   ```powershell
   $vnetRule = New-AzCosmosDBVirtualNetworkRule `
      -Id $subnetId
   ```

1. Обновите учетную запись Azure Cosmos DB, чтобы включить доступ к конечной точке службы из подсети:

   ```powershell
   Update-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName `
      -EnableVirtualNetwork $true `
      -VirtualNetworkRuleObject @($vnetRule)
   ```

1. Повторите предыдущие шаги для всех Azure Cosmos DB учетных записей, доступных из подсети.

1. Удалите правило брандмауэра IP для подсети из правил брандмауэра учетной записи Azure Cosmos DB.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

Ниже приведены некоторые часто задаваемые вопросы о настройке доступа из виртуальных сетей.

### <a name="are-notebooks-and-mongocassandra-shell-currently-compatible-with-virtual-network-enabled-accounts"></a>Совместимы ли записные книжки и оболочка Mongo/Cassandra с учетными записями с поддержкой виртуальной сети?

В настоящее время интеграция [оболочки Mongo](https://devblogs.microsoft.com/cosmosdb/preview-native-mongo-shell/) и [оболочки Cassandra](https://devblogs.microsoft.com/cosmosdb/announcing-native-cassandra-shell-preview/) в обозреватель данных Cosmos DB и [Служба записных книжек Jupyter](./cosmosdb-jupyter-notebooks.md)не поддерживаются при доступе к виртуальной сети. В настоящее время эта возможность находится на стадии активной разработки.

### <a name="can-i-specify-both-virtual-network-service-endpoint-and-ip-access-control-policy-on-an-azure-cosmos-account"></a>Можно ли указать политику конечной точки службы для виртуальной сети и политику контроля доступа к IP-адресу в учетной записи Azure Cosmos? 

Вы можете включить конечную точку службы виртуальной сети и политику управления доступом на использование IP-адресов (также называемую брандмауэром) в учетной записи Azure Cosmos. Эти две возможности дополняют друг друга и совместно обеспечивают изоляцию и безопасность вашей учетной записи Azure Cosmos. Использование брандмауэра IP гарантирует, что статические IP-адреса могут получать доступ к учетной записи Azure Cosmos DB. 

### <a name="how-do-i-limit-access-to-subnet-within-a-virtual-network"></a>Как ограничить доступ к подсети в виртуальной сети? 

Существует два шага, необходимые для ограничения доступа к учетной записи Azure Cosmos из подсети. Во-первых, разрешите трафику из подсети переносить свою подсеть и идентификатор виртуальной сети в Azure Cosmos DB. Это делается путем включения конечной точки службы для Azure Cosmos DB в подсети. Далее добавляется правило в учетной записи Azure Cosmos, определяющее эту подсеть как источник, с которого можно получить доступ к учетной записи.

### <a name="will-virtual-network-acls-and-ip-firewall-reject-requests-or-connections"></a>Будут ли ACL виртуальной сети и брандмауэр IP-адресов отклонять запросы или соединения? 

Когда добавляются правила доступа брандмауэра IP-адресов или виртуальной сети, только запросы из разрешенных источников получают действительные ответы. Другие запросы отклоняются с ошибкой "403 (запрещено)". Важно отличать брандмауэр учетной записи Azure Cosmos от брандмауэра на уровне подключения. Источник данных может по-прежнему подключаться к службе, сами подключения не отклоняются.

### <a name="my-requests-started-getting-blocked-when-i-enabled-service-endpoint-to-azure-cosmos-db-on-the-subnet-what-happened"></a>Мои запросы начали блокироваться при включении конечной точки службы для Azure Cosmos DB в подсети. Что произошло?

После того как конечная точка службы для Azure Cosmos DB будет включена в подсети, источник трафика, подключающийся к учетной записи, переключится с общего IP-адреса на виртуальную сеть и подсеть. Если ваша учетная запись Azure Cosmos имеет только брандмауэр на основе IP-адреса, трафик из подсети с поддержкой службы больше не будет соответствовать правилам брандмауэра для IP-адресов и, следовательно, будет отклонен. Выполните действия, чтобы прозрачно выполнить миграцию из брандмауэра на основе IP-адресов в виртуальную сеть с управлением доступом.

### <a name="are-additional-azure-rbac-permissions-needed-for-azure-cosmos-accounts-with-vnet-service-endpoints"></a>Требуются ли дополнительные разрешения RBAC Azure, необходимые для учетных записей Azure Cosmos с конечными точками службы виртуальной сети?

После добавления конечных точек службы виртуальной сети в учетную запись Azure Cosmos для внесения изменений в параметры учетной записи вам потребуется доступ к действию `Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action` для всех виртуальных сетей, настроенных в учетной записи Azure Cosmos. Это разрешение необходимо, так как в процессе авторизации подтверждается право доступа к ресурсам (ресурсам базы данных и виртуальной сети). Только после этого выполняется оценка параметров.
 
В ходе авторизации подтверждается разрешение на выполнение действия ресурсом службы виртуальной сети, даже если пользователь не задаст списки контроля доступа к виртуальной сети, используя интерфейс командной строки Azure. В настоящее время уровень управления учетной записи Azure Cosmos поддерживает настройку полного состояния учетной записи Azure Cosmos. Одним из параметров для вызова уровня управления является `virtualNetworkRules`. Если этот параметр не задан, интерфейс командной строки Azure отправляет вызов получения базы данных, чтобы извлечь `virtualNetworkRules`, и использует это значение в вызове обновления.

### <a name="do-the-peered-virtual-networks-also-have-access-to-azure-cosmos-account"></a>У одноранговых виртуальных сетей также есть доступ к учетной записи Azure Cosmos? 
Доступ имеют только виртуальные сети и их подсети, добавленные к учетной записи Azure Cosmos. Одноранговые виртуальные сети не могут получить доступ к учетной записи, пока подсети в одноранговых виртуальных сетях не будут добавлены в учетную запись.

### <a name="what-is-the-maximum-number-of-subnets-allowed-to-access-a-single-cosmos-account"></a>Каково максимальное число подсетей, которым разрешен доступ к одной учетной записи Cosmos? 
В настоящее время для учетной записи Azure Cosmos можно разрешить не более 256 подсетей.

### <a name="can-i-enable-access-from-vpn-and-express-route"></a>Можно ли включить доступ для VPN и Express Route? 
Для доступа к учетной записи Azure Cosmos через Express Route из локальной среды необходимо включить пиринг Microsoft. После размещения правил доступа к брандмауэру IP-адресов или виртуальной сети можно добавить общедоступные IP-адреса, используемые для пиринга Майкрософт в брандмауэре учетной записи Azure Cosmos, чтобы разрешить локальным службам доступ к учетной записи. 

### <a name="do-i-need-to-update-the-network-security-groups-nsg-rules"></a>Нужно ли обновлять правила группы безопасности сети (NSG)? 
Правила группы безопасности сети позволяют ограничить возможность подключения к подсети с помощью виртуальной сети. При добавлении конечной точки службы для Azure Cosmos DB к подсети нет необходимости открывать исходящие подключения в группе безопасности сети для учетной записи Azure Cosmos. 

### <a name="are-service-endpoints-available-for-all-vnets"></a>Доступны ли конечные точки служб для всех виртуальных сетей?
Нет, только виртуальные сети Azure Resource Manager могут использовать конечную точку службы. Классические виртуальные сети не поддерживают конечные точки службы.

### <a name="can-i-accept-connections-from-within-public-azure-datacenters-when-service-endpoint-access-is-enabled-for-azure-cosmos-db"></a>Можно ли принимать подключения из общедоступных центров данных Azure, если для Azure Cosmos DB включен доступ через конечную точку службы?  
Это необходимо, только если нужен доступ к учетной записи Azure Cosmos DB из других служб Azure, таких как Фабрика данных Azure, Когнитивный поиск Azure или другой службы, развернутой в данном регионе Azure.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о настройке брандмауэра для Azure Cosmos DB см. в статье [Поддержка брандмауэра](how-to-configure-firewall.md) для Azure Cosmos DB.
