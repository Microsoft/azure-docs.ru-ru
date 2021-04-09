---
title: Настройка ресурсов для развертывания модели машинного обучения в SQL для пограничных вычислений
description: В части 1 (из 3) руководства по SQL для пограничных вычислений, посвященного прогнозированию примеси железной руды, вы установите необходимое программное обеспечение и настроите ресурсы Azure, необходимые для развертывания модели машинного обучения в SQL Azure для пограничных вычислений.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: bc2f802a902212633fdbc081d33a51407d374e36
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98696368"
---
# <a name="install-software-and-set-up-resources-for-the-tutorial"></a>Установка программного обеспечения и настройка ресурсов для учебника

В этом руководстве из трех частей вы создадите модель машинного обучения для прогнозирования примеси железной руды, а именно содержание кремнезема в процентах, а затем развернете модель в SQL Azure для пограничных вычислений. В первой части вы установите необходимое программное обеспечение и развернете ресурсы Azure.

## <a name="prerequisites"></a>Предварительные требования

1. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/).
2. Установите Visual Studio 2019 со 
      * средствами Azure IoT Edge;
      * средствами кросс-платформенной разработки .NET Core;
      * Средства разработки контейнеров
3. Установите [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio/)
4. Откройте Azure Data Studio и настройте Python для записных книжек. Подробные сведения см. в разделе [Настройка Python для Записных книжек](/sql/azure-data-studio/sql-notebooks#configure-python-for-notebooks). Это может занять несколько минут.
5. Установите [последнюю версию](https://github.com/Azure/azure-powershell/releases/tag/v3.5.0-February2020) Azure CLI. Для следующих сценариев требуется, чтобы зона доступности PowerShell была последней версией (3.5.0, февраль 2020).
6. Настройте среду для отладки, запуска и тестирования решения IoT Edge, установив [средство разработки Azure IoT EdgeHub](https://pypi.org/project/iotedgehubdev/).
7. Установить Docker.

## <a name="deploy-azure-resources-using-powershell-script"></a>Развертывание ресурсов Azure с помощью скрипта PowerShell

Разверните ресурсы Azure, необходимые для этого руководства по SQL Azure для пограничных вычислений. Их можно развернуть либо с помощью сценария PowerShell, либо с помощью портала Azure. В этом руководстве используется сценарий PowerShell.

1. Импортируйте модули, необходимые для выполнения сценария PowerShell в этом руководстве.

   ```powershell
   Import-Module Az.Accounts -RequiredVersion 1.7.3
   Import-Module -Name Az -RequiredVersion 3.5.0
   Import-Module Az.IotHub -RequiredVersion 2.1.0
   Import-Module Az.Compute -RequiredVersion 3.5.0
   az extension add --name azure-iot
   az extension add --name azure-cli-ml
   ```

   [!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]


2. Объявите переменные, необходимые для сценария PowerShell.

   ```powershell
   $ResourceGroup = "<name_of_the_resource_group>"
   $IoTHubName = "<name_of_the_IoT_hub>"
   $location = "<location_of_your_Azure_Subscription>"
   $SubscriptionName = "<your_azure_subscription>"
   $NetworkSecGroup = "<name_of_your_network_security_group>"
   $StorageAccountName = "<name_of_your_storage_account>"
   ```

3. Объявите остальные переменные.

   ```powershell
   $IoTHubSkuName = "S1"
   $IoTHubUnits = 4
   $EdgeDeviceId = "IronOrePredictionDevice"
   $publicIpName = "VMPublicIP"
   $imageOffer = "iot_edge_vm_ubuntu"
   $imagePublisher = "microsoft_iot_edge"
   $imageSku = "ubuntu_1604_edgeruntimeonly"
   $AdminAcc = "iotadmin"
   $AdminPassword = ConvertTo-SecureString "IoTAdmin@1234" -AsPlainText -Force
   $VMSize = "Standard_DS3"
   $NetworkName = "MyNet"
   $NICName = "MyNIC"
   $SubnetName = "MySubnet"
   $SubnetAddressPrefix = "10.0.0.0/24"
   $VnetAddressPrefix = "10.0.0.0/16"
   $MyWorkSpace = "SQLDatabaseEdgeDemo"
   $containerRegistryName = $ResourceGroup + "ContRegistry"
   ```

4. Чтобы начать создание ресурсов, войдите в Azure.

   ```powershell
   Login-AzAccount
   
   az login
   ```

5. Настройте идентификатор подписки Azure

   ```powershell
   Select-AzSubscription -Subscription $SubscriptionName
   az account set --subscription $SubscriptionName
   ```

6. Создайте группу ресурсов, если она еще не существует.

   ```powershell
   $rg = Get-AzResourceGroup -Name $ResourceGroup
   if($rg -eq $null)
   {
       Write-Output("Resource Group $ResourceGroup does not exist, creating Resource Gorup")
       New-AzResourceGroup -Name $ResourceGroup -Location $location
   }
   else
   {
       Write-Output ("Resource Group $ResourceGroup exists")
   }
   ```

7. Создайте учетную запись хранения и контейнер учетной записи хранения в группе ресурсов.

   ```powershell
   $sa = Get-AzStorageAccount -ResourceGroupName $ResourceGroup -Name $StorageAccountName 
   if ($sa -eq $null)
   {
       New-AzStorageAccount -ResourceGroupName $ResourceGroup -Name $StorageAccountName -SkuName Standard_LRS -Location $location -Kind Storage
       $sa = Get-AzStorageAccount -ResourceGroupName $ResourceGroup -Name $StorageAccountName 
       $storagekey = Get-AzStorageAccountKey -ResourceGroupName $ResourceGroup -Name $StorageAccountName 
       $storageContext = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $storagekey[0].Value
       New-AzStorageContainer -Name "sqldatabasedacpac" -Context $storageContext 
   }
   else
   {
      Write-Output ("Storage Account $StorageAccountName exists in Resource Group $ResourceGroup")     
   }
   ```

8. Отправьте файл базы данных `dacpac` в учетную запись хранения и создайте URL-адрес SAS для большого двоичного объекта. **Запишите URL-адрес SAS для базы данных `dacpac` большого двоичного объекта.**

   ```powershell
   $file = Read-Host "Please Enter the location to the zipped Database DacPac file:"
   Set-AzStorageBlobContent -File $file -Container "sqldatabasedacpac" -Blob "SQLDatabasedacpac.zip" -Context $sa.Context
   $DacpacFileSASURL = New-AzStorageBlobSASToken -Container "sqldatabasedacpac" -Blob "SQLDatabasedacpac.zip" -Context $sa.Context -Permission r -StartTime (Get-Date).DateTime -ExpiryTime (Get-Date).AddMonths(12) -FullUri
   ```

9. Создайте реестр контейнеров Azure в этой группе ресурсов.

   ```powershell
   $containerRegistry = Get-AzContainerRegistry -ResourceGroupName $ResourceGroup -Name $containerRegistryName 
   if ($containerRegistry -eq $null)
   {
       New-AzContainerRegistry -ResourceGroupName $ResourceGroup -Name $containerRegistryName -Sku Standard -Location $location -EnableAdminUser 
       $containerRegistry = Get-AzContainerRegistry -ResourceGroupName $ResourceGroup -Name $containerRegistryName 
   }
   else
   {
       Write-Output ("Container Registry $containerRegistryName exists in Resource Group $ResourceGroup")
   }
   ```

10. Создайте группу безопасности сети в группе ресурсов.

    ```powershell
    $nsg = Get-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroup -Name $NetworkSecGroup 
    if($nsg -eq $null)
    {
        Write-Output("Network Security Group $NetworkSecGroup does not exist in the resource group $ResourceGroup")
    
        $rule1 = New-AzNetworkSecurityRuleConfig -Name "SSH" -Description "Allow SSH" -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 22
        $rule2 = New-AzNetworkSecurityRuleConfig -Name "SQL" -Description "Allow SQL" -Access Allow -Protocol Tcp -Direction Inbound -Priority 101 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 1600
        New-AzNetworkSecurityGroup -Name $NetworkSecGroup -ResourceGroupName $ResourceGroup -Location $location -SecurityRules $rule1, $rule2
    
        $nsg = Get-AzNetworkSecurityGroup -ResourceGroupName $ResourceGroup -Name $NetworkSecGroup
    }
    else
    {
        Write-Output ("Network Security Group $NetworkSecGroup exists in the resource group $ResourceGroup")
    }
    ```

11. Создайте виртуальную машину Azure с поддержкой SQL Azure для пограничных вычислений. Эта виртуальная машина будет работать в качестве пограничного устройства.

    ```powershell
    $AzVM = Get-AzVM -ResourceGroupName $ResourceGroup -Name $EdgeDeviceId
    If($AzVM -eq $null)
    {
        Write-Output("The Azure VM with Name- $EdgeVMName is not present in the Resource Group- $ResourceGroup ")
    
        $SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
        $Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroup -Location $location -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
        $publicIp = New-AzPublicIpAddress -Name $publicIpName -ResourceGroupName $ResourceGroup -AllocationMethod Static -Location $location  
        $NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroup -Location $location -SubnetId $Vnet.Subnets[0].Id -NetworkSecurityGroupId $nsg.Id -PublicIpAddressId $publicIp.Id
         
        ##Set-AzNetworkInterfaceIpConfig -Name "ipconfig1"  -NetworkInterface $NIC -PublicIpAddress $publicIp
    
        $Credential = New-Object System.Management.Automation.PSCredential ($AdminAcc, $AdminPassword);
    
        $VirtualMachine = New-AzVMConfig -VMName $EdgeDeviceId -VMSize $VMSize
        $VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Linux -ComputerName $EdgeDeviceId -Credential $Credential
        $VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id  
        $VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName $imagePublisher -Offer $imageOffer -Skus $imageSku -Version latest 
        $VirtualMachine = Set-AzVMPlan -VM $VirtualMachine -Name $imageSku -Publisher $imagePublisher -Product $imageOffer
    
        $AzVM = New-AzVM -ResourceGroupName $ResourceGroup -Location $location -VM $VirtualMachine -Verbose
        $AzVM = Get-AzVM -ResourceGroupName $ResourceGroup -Name $EdgeDeviceId
          
    }
    else
    {
        Write-Output ("The Azure VM with Name- $EdgeDeviceId is present in the Resource Group- $ResourceGroup ")
    }
    ```

12. Создайте центр Интернета вещей в группе ресурсов.

    ```powershell
    $iotHub = Get-AzIotHub -ResourceGroupName $ResourceGroup -Name $IoTHubName
    If($iotHub -eq $null)
    {
        Write-Output("IoTHub $IoTHubName does not exists, creating The IoTHub in the resource group $ResourceGroup")
        New-AzIotHub -ResourceGroupName $ResourceGroup -Name $IoTHubName -SkuName $IoTHubSkuName -Units $IoTHubUnits -Location $location -Verbose
    }
    else
    {
        Write-Output ("IoTHub $IoTHubName present in the resource group $ResourceGroup") 
    }
    ```

13. Добавьте пограничное устройство в центр Интернета вещей. На этом шаге создается только цифровое удостоверение устройства.

    ```powershell
    $deviceIdentity = Get-AzIotHubDevice -ResourceGroupName $ResourceGroup -IotHubName $IoTHubName -DeviceId $EdgeDeviceId
    If($deviceIdentity -eq $null)
    {
        Write-Output("The Edge Device with DeviceId- $EdgeDeviceId is not registered to the IoTHub- $IoTHubName ")
        Add-AzIotHubDevice -ResourceGroupName $ResourceGroup -IotHubName $IoTHubName -DeviceId $EdgeDeviceId -EdgeEnabled  
    }
    else
    {
        Write-Output ("The Edge Device with DeviceId- $EdgeDeviceId is registered to the IoTHub- $IoTHubName")
    }
    $deviceIdentity = Get-AzIotHubDevice -ResourceGroupName $ResourceGroup -IotHubName $IoTHubName -DeviceId $EdgeDeviceId
    ```

14. Получите основную строку подключения устройства. Это потребуется позже для виртуальной машины. Следующая команда для развертываний использует Azure CLI.

    ```powershell
    $deviceConnectionString = az iot hub device-identity show-connection-string --device-id $EdgeDeviceId --hub-name $IoTHubName --resource-group $ResourceGroup --subscription $SubscriptionName
    $connString = $deviceConnectionString[1].Substring(23,$deviceConnectionString[1].Length-24)
    $connString
    ```

15. Обновите строку подключения в файле конфигурации IoT Edge на пограничном устройстве. Следующие команды используют Azure CLI для развертываний.

    ```powershell
    $script = "/etc/iotedge/configedge.sh '" + $connString + "'"
    az vm run-command invoke -g $ResourceGroup -n $EdgeDeviceId  --command-id RunShellScript --script $script
    ```

16. Создайте рабочую область "Машинное обучение Azure" в группе ресурсов.

    ```powershell
    az ml workspace create -w $MyWorkSpace -g $ResourceGroup
    ```


## <a name="next-steps"></a>Next Steps

* [Настройка модулей и подключений IoT Edge](tutorial-set-up-iot-edge-modules.md)
