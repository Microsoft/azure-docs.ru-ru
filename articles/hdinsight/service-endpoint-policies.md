---
title: Настройка политик конечной точки службы в Azure HDInsight
description: Узнайте, как настроить политики конечной точки службы для виртуальной сети с помощью Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.date: 07/15/2020
ms.openlocfilehash: 992cd994f96b5637d5afd91bccfecde8704d2886
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98940630"
---
# <a name="configure-virtual-network-service-endpoint-policies-for-azure-hdinsight"></a>Настройка политик конечной точки службы виртуальной сети для Azure HDInsight

В этой статье содержатся сведения о том, как реализовать политики конечной точки службы в виртуальных сетях с помощью Azure HDInsight.

## <a name="background"></a>Историческая справка

Azure HDInsight позволяет создавать кластеры в собственной виртуальной сети. Если необходимо разрешить исходящий трафик из виртуальной сети в другие службы Azure, такие как учетные записи хранения, можно создать [политики конечной точки службы](../virtual-network/virtual-network-service-endpoint-policies-overview.md). Однако политики конечной точки службы, созданные с помощью портал Azure, позволяют создавать только политику для одной учетной записи, всех учетных записей в подписке или всех учетных записей в группе ресурсов.

Однако в качестве управляемой службы Azure HDInsight собирает файлы данных и журналов из каждого кластера в конкретных учетных записях хранения в каждом регионе. Чтобы эти данные могли получить доступ к HDInsight из виртуальной сети, необходимо создать политики конечной точки службы, которые разрешают исходящий трафик к конкретным точкам сбора данных, управляемым Azure HDInsight.

## <a name="service-endpoint-policies-for-hdinsight"></a>Политики конечной точки службы для HDInsight

Эти политики конечной точки службы поддерживают следующие функциональные возможности.

- Коллекция журналов и данных телеметрии при создании кластера, выполнении заданий и операциях платформы, таких как масштабирование.
- Подключение виртуальных жестких дисков (VHD) к вновь созданным узлам кластера для подготовки программного обеспечения и библиотек в кластере.

Если политики конечной точки службы не созданы для включения этого потока данных, создание кластера может завершиться сбоем, и Azure HDInsight не сможет предоставить поддержку кластеров.

## <a name="create-service-endpoint-policies-for-hdinsight"></a>Создание политик конечной точки службы для HDInsight

Перед созданием новых кластеров убедитесь, что к виртуальной сети подключены правильные политики конечной точки службы. В противном случае создание кластера может завершиться ошибкой или привести к ошибке.

Используйте следующую процедуру, чтобы создать необходимые политики конечной точки службы.

1. Определите регион, в котором будет создан кластер HDInsight.
1. Найдите этот регион в [списке ресурсов политики конечной точки службы](https://github.com/Azure-Samples/hdinsight-enterprise-security/blob/main/hdinsight-service-endpoint-policy-resources.json), который предоставляет все группы ресурсов для учетных записей хранения управления HDInsight.
1. Выберите список групп ресурсов для вашего региона. Ниже приведен пример ресурсов для `Canada Central` .

    ```json
    "Canada Central":[
        "/subscriptions/235d341f-7fb9-435c-9bdc-034b7306c9b4/resourceGroups/Default-Storage-WestUS",
        "/subscriptions/da0c4c68-9283-4f88-9c35-18f7bd72fbdd/resourceGroups/GenevaWarmPathManageRG",
        "/subscriptions/6a853a41-3423-4167-8d9c-bcf37dc72818/resourceGroups/GenevaWarmPathManageRG",
        "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/Default-Storage-CanadaCentral",
        "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/cancstorage",
        "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/GenevaWarmPathManageRG"
    ],
    ```

1. Вставьте этот список групп ресурсов в сценарий установки, написанный на Azure CLI или Azure PowerShell.

    ```azurecli
    $subscriptionId = "<subscription id>"
    $rgName="<resource group name> "
    $location="<location name>"
    $vnetName="<vnet name>"
    $subnetName="<subnet name>"
    $sepName="<service endpoint policy name>"
    $sepDefName="<service endpoint policy definition name>"
    
    # Set to the right subscription ID
    az account set --subscription $subscriptionId
    
    # setup service endpoint on the virtual network subnet
    az network vnet subnet update -g $rgName --vnet-name $vnetName -n $subnetName --service-endpoints Microsoft.Storage
    
    # Create Service Endpoint Policy
    az network service-endpoint policy create -g $rgName  -n $sepName -l $location
    
    # Insert the list of HDInsight owned resources for the region your clusters will be created in.
    [String[]]$resources = @("/subscriptions/235d341f-7fb9-435c-9bdc-034b7306c9b4/resourceGroups/Default-Storage-WestUS",`
    "/subscriptions/da0c4c68-9283-4f88-9c35-18f7bd72fbdd/resourceGroups/GenevaWarmPathManageRG",`
    "/subscriptions/6a853a41-3423-4167-8d9c-bcf37dc72818/resourceGroups/GenevaWarmPathManageRG",`
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/Default-Storage-CanadaCentral",`
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/cancstorage",`
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/GenevaWarmPathManageRG")
    
    #Assign service resources to the SEP policy.
    az network service-endpoint policy-definition create -g $rgName --policy-name $sepName -n $sepDefName --service "Microsoft.Storage" --service-resources $resources
    
    # Associate a subnet to the service endpoint policy just created. If there is a delay in updating it to subnet, you can use the Azure portal to associate the policy with the subnet.
    az network vnet subnet update -g $rgName --vnet-name $vnetName -n $subnetName --service-endpoint-policy $sepName
    ```

    Если вы предпочитаете настроить политику конечной точки службы с помощью PowerShell, используйте следующий фрагмент кода.
    
    ```powershell
    #Script to assign SEP 
    $subscriptionId = "<subscription id>"
    $rgName = "<resource group name>"
    $vnetName = "<vnet name>"
    $subnetName = "<subnet Name"
    $location = "Canada Central"
    
    # Connect to your Azure Account
    Connect-AzAccount
    
    # Select the Subscription that you want to use
    Select-AzSubscription -SubscriptionId $subscriptionId
    
    # Retrieve VNet Config
    $vnet = Get-AzVirtualNetwork -ResourceGroupName $rgName -Name $vnetName
    
    # Retrieve Subnet Config
    $subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet
    
    # Insert the list of HDInsight owned resources for the region your clusters will be created in.
    [String[]]$resources = @("/subscriptions/235d341f-7fb9-435c-9bdc-034b7306c9b4/resourceGroups/Default-Storage-WestUS",
    "/subscriptions/da0c4c68-9283-4f88-9c35-18f7bd72fbdd/resourceGroups/GenevaWarmPathManageRG",
    "/subscriptions/6a853a41-3423-4167-8d9c-bcf37dc72818/resourceGroups/GenevaWarmPathManageRG",
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/Default-Storage-CanadaCentral",
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/cancstorage",
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/GenevaWarmPathManageRG")
    
    #Declare service endpoint policy definition
    $sepDef = New-AzServiceEndpointPolicyDefinition -Name "SEPHDICanadaCentral" -Description "Service Endpoint Policy Definition" -Service "Microsoft.Storage" -ServiceResource $resources
    
    # Service Endpoint Policy
    $sep= New-AzServiceEndpointPolicy -ResourceGroupName $rgName -Name "SEPHDICanadaCentral" -Location $location -ServiceEndpointPolicyDefinition $sepDef
    
    # Associate a subnet to the service endpoint policy just created. If there is a delay in updating it to subnet, you can use the Azure portal to associate the policy with the subnet.
    Set-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet -AddressPrefix $subnet.AddressPrefix -ServiceEndpointPolicy $sep
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Архитектура виртуальной сети Azure HDInsight](hdinsight-virtual-network-architecture.md)
* [Настройка сетевого виртуального устройства](./network-virtual-appliance.md)
