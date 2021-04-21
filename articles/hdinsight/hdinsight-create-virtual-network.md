---
title: Создание виртуальных сетей для кластеров Azure HDInsight
description: Узнайте, как создать виртуальную сеть Azure для подключения HDInsight к другим облачным ресурсам или ресурсам в центре обработки данных.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 04/16/2020
ms.openlocfilehash: 43d57eac94cabb5c648183911e0c0bf72889946d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98946067"
---
# <a name="create-virtual-networks-for-azure-hdinsight-clusters"></a>Создание виртуальных сетей для кластеров Azure HDInsight

В этой статье приводятся рекомендации и примеры кода для создания и настройки [виртуальных сетей Azure](../virtual-network/virtual-networks-overview.md), которые можно использовать с кластерами Azure HDInsight. Ниже представлены подробные примеры создания групп безопасности сети (NSG) и настройки службы доменных имен (DNS).

Дополнительные сведения об использовании виртуальных сетей с HDInsight см. в статье [о планировании виртуальной сети для Azure HDInsight](hdinsight-plan-virtual-network-deployment.md).

## <a name="prerequisites-for-code-samples-and-examples"></a>Необходимые условия для работы с примерами кода

Перед тем как выполнить любой из примеров кода из этой статьи, изучите общую информацию о сетях TCP/IP. Если вы не знакомы с ними, перед внесением изменений в рабочие сети обратитесь к специалистам по сетями TCP/IP.

Ниже приведены другие условия для работы с примерами из этой статьи.

* Если вы используете PowerShell, установите [модуль AZ](/powershell/azure/).
* Если вы хотите использовать Azure CLI и еще не установили его, ознакомьтесь с разделом [Установка Azure CLI](/cli/azure/install-azure-cli).

> [!IMPORTANT]  
> Пошаговые инструкции по подключению HDInsight к локальной сети с помощью виртуальной сети Azure см. в статье [Подключение HDInsight к локальной сети](connect-on-premises-network.md).

## <a name="example-network-security-groups-with-hdinsight"></a><a id="hdinsight-nsg"></a>Пример: группы безопасности сети с HDInsight

В примерах этого раздела показано, как создавать правила для групп безопасности сети. Эти правила позволяют HDInsight обмениваться данными со службами управления Azure. Прежде чем использовать эти примеры, настройте IP-адреса в соответствии с теми, которые применяются для вашего региона Azure. Эти сведения можно найти в списке [IP-адресов управления HDInsight](hdinsight-management-ip-addresses.md).

### <a name="azure-resource-management-template"></a>Шаблон управления ресурсами Azure

Следующий шаблон управления ресурсами создает виртуальную сеть, которая ограничивает входящий трафик, но разрешает трафик с IP-адресов, требуемых для HDInsight. Этот шаблон также создает кластер HDInsight в виртуальной сети.

* [Развертывание защищенной виртуальной сети Azure и кластера HDInsight Hadoop](https://azure.microsoft.com/resources/templates/101-hdinsight-secure-vnet/)

### <a name="azure-powershell"></a>Azure PowerShell

Используйте следующий сценарий PowerShell для создания виртуальной сети, которая ограничивает входящий трафик и разрешает трафик с IP-адресов для региона Северная Европа.

> [!IMPORTANT]  
> Измените IP-адреса для `hdirule1` и `hdirule2` в этом примере в соответствии с теми, которые применяются для вашего региона Azure. Эти сведения можно найти в списке [IP-адресов управления HDInsight](hdinsight-management-ip-addresses.md).

```azurepowershell
$vnetName = "Replace with your virtual network name"
$resourceGroupName = "Replace with the resource group the virtual network is in"
$subnetName = "Replace with the name of the subnet that you plan to use for HDInsight"

# Get the Virtual Network object
$vnet = Get-AzVirtualNetwork `
    -Name $vnetName `
    -ResourceGroupName $resourceGroupName

# Get the region the Virtual network is in.
$location = $vnet.Location

# Get the subnet object
$subnet = $vnet.Subnets | Where-Object Name -eq $subnetName

# Create a Network Security Group.
# And add exemptions for the HDInsight health and management services.
$nsg = New-AzNetworkSecurityGroup `
    -Name "hdisecure" `
    -ResourceGroupName $resourceGroupName `
    -Location $location `
    | Add-AzNetworkSecurityRuleConfig `
        -name "hdirule1" `
        -Description "HDI health and management address 52.164.210.96" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "52.164.210.96" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 300 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule2" `
        -Description "HDI health and management 13.74.153.132" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "13.74.153.132" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 301 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule3" `
        -Description "HDI health and management 168.61.49.99" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.49.99" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 302 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule4" `
        -Description "HDI health and management 23.99.5.239" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "23.99.5.239" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 303 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule5" `
        -Description "HDI health and management 168.61.48.131" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "168.61.48.131" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 304 `
        -Direction Inbound `
    | Add-AzNetworkSecurityRuleConfig `
        -Name "hdirule6" `
        -Description "HDI health and management 138.91.141.162" `
        -Protocol "*" `
        -SourcePortRange "*" `
        -DestinationPortRange "443" `
        -SourceAddressPrefix "138.91.141.162" `
        -DestinationAddressPrefix "VirtualNetwork" `
        -Access Allow `
        -Priority 305 `
        -Direction Inbound `

# Set the changes to the security group
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# Apply the NSG to the subnet
Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name $subnetName `
    -AddressPrefix $subnet.AddressPrefix `
    -NetworkSecurityGroup $nsg
$vnet | Set-AzVirtualNetwork
```

В этом примере показано, как добавить правила, чтобы разрешить входящий трафик на требуемые IP-адреса. В нем не содержится правило, ограничивающее входящий доступ из других источников. В следующем примере кода показано, как включить доступ по SSH через Интернет:

```azurepowershell
Get-AzNetworkSecurityGroup -Name hdisecure -ResourceGroupName RESOURCEGROUP |
Add-AzNetworkSecurityRuleConfig -Name "SSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 306 -Direction Inbound
```

### <a name="azure-cli"></a>Azure CLI

Ниже приведен порядок действий по созданию виртуальной сети, которая ограничивает входящий трафик, но разрешает трафик с IP-адресов, требуемых для HDInsight.

1. Используйте следующую команду, чтобы создать новую группу безопасности сети с именем `hdisecure`. Замените `RESOURCEGROUP` на имя группы ресурсов, которая содержит виртуальную сеть Azure. Замените `LOCATION` на название расположения (региона), в котором была создана группа.

    ```azurecli
    az network nsg create -g RESOURCEGROUP -n hdisecure -l LOCATION
    ```

    После создания группы вы получите сведения о новой группе.

2. Используйте следующую команду для добавления правил в новую группу безопасности сети. Эти правила разрешают входящий трафик через порт 443 от службы работоспособности и управления Azure HDInsight. Замените `RESOURCEGROUP` на имя группы ресурсов, которая содержит виртуальную сеть Azure.

    > [!IMPORTANT]  
    > Измените IP-адреса для `hdirule1` и `hdirule2` в этом примере в соответствии с теми, которые применяются для вашего региона Azure. Эти сведения можно найти в списке [IP-адресов управления HDInsight](hdinsight-management-ip-addresses.md).

    ```azurecli
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule1 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "52.164.210.96" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 300 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule2 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "13.74.153.132" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 301 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule3 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.49.99" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 302 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule4 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "23.99.5.239" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 303 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule5 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "168.61.48.131" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 304 --direction "Inbound"
    az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n hdirule6 --protocol "*" --source-port-range "*" --destination-port-range "443" --source-address-prefix "138.91.141.162" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 305 --direction "Inbound"
    ```

3. Используйте следующую команду, чтобы получить уникальный идентификатор для этой группы безопасности сети:

    ```azurecli
    az network nsg show -g RESOURCEGROUP -n hdisecure --query "id"
    ```

    Эта команда возвращает значение следующего вида:

    ```output
    "/subscriptions/SUBSCRIPTIONID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

4. Используйте следующую команду, чтобы применить группу безопасности сети к подсети. Замените `GUID` и `RESOURCEGROUP` на значения, возвращенные на предыдущем шаге. Замените `VNETNAME` и `SUBNETNAME` на имена виртуальной сети и подсети, которые необходимо создать.

    ```azurecli
    az network vnet subnet update -g RESOURCEGROUP --vnet-name VNETNAME --name SUBNETNAME --set networkSecurityGroup.id="/subscriptions/GUID/resourceGroups/RESOURCEGROUP/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    ```

    После выполнения этой команды можно установить HDInsight в виртуальную сеть.

Данные действия только открывают доступ к службе работоспособности и управления HDInsight в облаке Azure. Весь остальной доступ к кластеру HDInsight из-за пределов виртуальной сети блокируется. Чтобы разрешить доступ из-за пределов виртуальной сети, необходимо добавить дополнительные правила группы безопасности сети.

В следующем примере кода показано, как включить доступ по SSH через Интернет:

```azurecli
az network nsg rule create -g RESOURCEGROUP --nsg-name hdisecure -n ssh --protocol "*" --source-port-range "*" --destination-port-range "22" --source-address-prefix "*" --destination-address-prefix "VirtualNetwork" --access "Allow" --priority 306 --direction "Inbound"
```

## <a name="example-dns-configuration"></a><a id="example-dns"></a> Пример: конфигурация DNS

### <a name="name-resolution-between-a-virtual-network-and-a-connected-on-premises-network"></a>Разрешение DNS-имен между виртуальной сетью и подключенной локальной сетью

Данный пример исходит из следующих предположений:

* Имеется виртуальная сеть Azure, подключенная к локальной сети посредством VPN-шлюза.

* Пользовательский DNS-сервер в виртуальной сети работает под управлением операционной системы Unix или Linux.

* На пользовательском DNS-сервере установлен [Bind](https://www.isc.org/downloads/bind/).

На пользовательском DNS-сервере в виртуальной сети:

1. Воспользуйтесь Azure PowerShell или Azure CLI для поиска DNS-суффикса виртуальной сети:

    Замените `RESOURCEGROUP` на имя группы ресурсов, которая содержит виртуальную сеть Azure, а затем введите указанную ниже команду.

    ```azurepowershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

1. На пользовательском DNS-сервере для виртуальной сети добавьте в файл `/etc/bind/named.conf.local` следующее содержимое:

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {168.63.129.16;}; # Azure recursive resolver
    };
    ```

    Замените значение `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` DNS-суффиксом виртуальной сети.

    Такая конфигурация обеспечивает перенаправление запросов DNS-суффиксов виртуальной сети на рекурсивный сопоставитель Azure.

1. На пользовательском DNS-сервере для виртуальной сети добавьте в файл `/etc/bind/named.conf.options` следующее содержимое:

    ```
    // Clients to accept requests from
    // TODO: Add the IP range of the joined network to this list
    acl goodclients {
        10.0.0.0/16; # IP address range of the virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            # All other requests are sent to the following
            forwarders {
                192.168.0.1; # Replace with the IP address of your on-premises DNS server
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```
    
    * Замените значение `10.0.0.0/16` диапазоном IP-адресов виртуальной сети. Эта запись разрешает запросы на разрешение имен в этом диапазоне.

    * Добавьте диапазон IP-адресов локальной сети в раздел `acl goodclients { ... }`.  Эта запись разрешает запросы на разрешения имен с ресурсов в локальной сети.
    
    * Замените значение `192.168.0.1` IP-адресом локального DNS-сервера. Эта запись направляет все остальные DNS-запросы на локальный DNS-сервер.

1. Чтобы использовать конфигурацию, перезапустите Bind. Например, `sudo service bind9 restart`.

1. Добавьте условный сервер пересылки на локальный DNS-сервер. Настройте условный сервер пересылки на отправку запросов DNS-суффикса из шага 1 на пользовательский DNS-сервер.

    > [!NOTE]  
    > Обратитесь к документации по используемому программному обеспечению DNS, чтобы ознакомиться со сведениями о добавлении условного сервера пересылки.

После выполнения этих действий можно подключаться к ресурсам в любой из этих сетей, используя полные доменные имена (FQDN). Теперь можно установить HDInsight в виртуальной сети.

### <a name="name-resolution-between-two-connected-virtual-networks"></a>Разрешение имен между двумя подключенными виртуальными сетями

Данный пример исходит из следующих предположений:

* Имеется две виртуальных сети Azure, подключенные друг к другу посредством VPN-шлюза или пиринга.

* Пользовательский DNS-сервер в обеих виртуальных сетях работает под управлением операционной системы Unix или Linux.

* На пользовательских DNS-серверах установлен [Bind](https://www.isc.org/downloads/bind/).

1. Воспользуйтесь Azure PowerShell или Azure CLI для поиска DNS-суффикса в обеих виртуальных сетях:

    Замените `RESOURCEGROUP` на имя группы ресурсов, которая содержит виртуальную сеть Azure, а затем введите указанную ниже команду.

    ```azurepowershell
    $NICs = Get-AzNetworkInterface -ResourceGroupName "RESOURCEGROUP"
    $NICs[0].DnsSettings.InternalDomainNameSuffix
    ```

    ```azurecli
    az network nic list --resource-group RESOURCEGROUP --query "[0].dnsSettings.internalDomainNameSuffix"
    ```

2. На пользовательском DNS-сервере добавьте в файл `/etc/bind/named.config.local` следующее содержимое. Внесите это изменение на пользовательские DNS-серверы в обеих виртуальных сетях.

    ```
    // Forward requests for the virtual network suffix to Azure recursive resolver
    zone "0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net" {
        type forward;
        forwarders {10.0.0.4;}; # The IP address of the DNS server in the other virtual network
    };
    ```

    Замените значение `0owcbllr5hze3hxdja3mqlrhhe.ex.internal.cloudapp.net` DNS-суффиксом __другой__ виртуальной сети. Эта запись направляет запросы DNS-суффикса удаленной сети на пользовательский DNS-сервер в этой сети.

3. На пользовательских DNS-серверах в обеих виртуальных сетях добавьте в файл `/etc/bind/named.conf.options` следующее содержимое:

    ```
    // Clients to accept requests from
    acl goodclients {
        10.1.0.0/16; # The IP address range of one virtual network
        10.0.0.0/16; # The IP address range of the other virtual network
        localhost;
        localnets;
    };

    options {
            directory "/var/cache/bind";

            recursion yes;

            allow-query { goodclients; };

            forwarders {
            168.63.129.16;   # Azure recursive resolver
            };

            dnssec-validation auto;

            auth-nxdomain no;    # conform to RFC1035
            listen-on { any; };
    };
    ```

   Замените значения `10.0.0.0/16` и `10.1.0.0/16` диапазонами IP-адресов виртуальных сетей. Эта запись позволяет ресурсам в каждой сети выполнять запросы к DNS-серверам.

    Запросы, которые не предназначены для получения DNS-суффиксов виртуальных сетей (например, microsoft.com), обрабатываются рекурсивным сопоставителем Azure.

4. Чтобы использовать конфигурацию, перезапустите Bind. Например, `sudo service bind9 restart` на обоих DNS-серверах.

После выполнения этих действий можно подключаться к ресурсам в виртуальной сети, используя полные доменные имена (FQDN). Теперь можно установить HDInsight в виртуальной сети.

## <a name="next-steps"></a>Дальнейшие действия

* Полный пример настройки HDInsight для подключения к локальной сети см. в статье [Подключение HDInsight к локальной сети](./connect-on-premises-network.md).
* Инструкции по настройке кластеров Apache HBase в виртуальных сетях Azure см. в статье [Создание кластеров Apache HBase в HDInsight в виртуальной сети Azure](hbase/apache-hbase-provision-vnet.md).
* См. инструкции по [настройке георепликации кластера Apache HBase в виртуальных сетях Azure](hbase/apache-hbase-replication.md).
* Дополнительные сведения о виртуальных сетях Azure см. в статье [Виртуальная сеть Azure](../virtual-network/virtual-networks-overview.md).

* Дополнительные сведения о группах безопасности сети см. в статье [Фильтрация сетевого трафика с помощью групп безопасности сети](../virtual-network/network-security-groups-overview.md).

* Дополнительные сведения о пользовательских маршрутах см. в статье [User-defined routes and IP forwarding](../virtual-network/virtual-networks-udr-overview.md) (Определяемые пользователем маршруты и IP-пересылка).
