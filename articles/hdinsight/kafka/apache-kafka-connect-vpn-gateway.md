---
title: Подключение к Kafka с помощью виртуальных сетей в Azure HDInsight
description: Узнайте, как подключиться напрямую к Kafka в HDInsight с помощью виртуальной сети Azure. Узнайте, как подключиться к Kafka из клиентов разработки с помощью шлюза VPN или из клиентов в локальной сети, используя устройство шлюза VPN.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-python
ms.date: 03/04/2020
ms.openlocfilehash: ad802b2bdf08a8e43179beece5f52d869513aff3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98933042"
---
# <a name="connect-to-apache-kafka-on-hdinsight-through-an-azure-virtual-network"></a>Подключение к Apache Kafka в HDInsight с помощью виртуальной сети Azure

Узнайте, как подключиться напрямую к Apache Kafka в HDInsight с помощью виртуальной сети Azure. В этой статье приведены сведения о подключении к Kafka с использованием приведенных ниже конфигураций.

* Из ресурсов в локальной сети. Это подключение устанавливается с помощью VPN-устройства (программного или аппаратного) в локальной сети.
* Из среды разработки с помощью программного VPN-клиента.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="architecture-and-planning"></a>Архитектура и планирование

HDInsight не разрешает прямое подключение к Kafka через общедоступный Интернет. Вместо этого клиенты Kafka (производители и потребители) должны использовать один из следующих методов подключения.

* Запустите клиент в той же виртуальной сети, что и Kafka в HDInsight. Сведения об использовании этой конфигурации см. в статье [Приступая к работе с Apache Kafka в HDInsight](apache-kafka-get-started.md). Клиент работает непосредственно на узлах кластера HDInsight или на другой виртуальной машине в той же сети.

* Подключите частную сеть, например локальную, к виртуальной сети. Эта конфигурация разрешает клиентам в локальной сети напрямую работать с Kafka. Чтобы включить эту конфигурацию, сделайте следующее:

  1. Создайте виртуальную сеть.
  2. Создайте VPN-шлюз, использующий конфигурацию типа "сеть — сеть". Конфигурация, используемая в этой статье, подключается к устройству шлюза VPN в локальной сети.
  3. Создайте DNS-сервер в виртуальной сети.
  4. Настройте переадресацию между DNS-серверами в каждой сети.
  5. Создайте Kafka в кластере HDInsight в виртуальной сети.

     Дополнительные сведения см. в разделе [Подключение к Apache Kafka из локальной сети](#on-premises).

* Подключите отдельные виртуальные машины к виртуальной сети с помощью VPN-шлюза и VPN-клиента. Чтобы включить эту конфигурацию, сделайте следующее:

  1. Создайте виртуальную сеть.
  2. Создайте VPN-шлюз, использующий конфигурацию типа "точка — сеть". Эту конфигурацию можно использовать с клиентами Windows и MacOS.
  3. Создайте Kafka в кластере HDInsight в виртуальной сети.
  4. Настройте Kafka для объявления IP-адресов. Такая конфигурация позволяет клиенту подключаться с помощью IP-адресов брокера вместо использования доменных имен.
  5. Скачайте и используйте VPN-клиент в системе разработки.

     Дополнительные сведения см. в разделе [Подключение к Apache Kafka с помощью VPN-клиента](#vpnclient).

     > [!WARNING]  
     > Эта конфигурация рекомендуется только для целей разработки из-за следующих ограничений:
     >
     > * Каждый клиент должен подключиться с помощью программного VPN-клиента.
     > * Клиент VPN не передает запросы на разрешения имен в виртуальную сеть, поэтому для обмена данными с Kafka необходимо использовать IP-адрес. Для взаимодействия с IP-адресом требуется дополнительная настройка кластера Kafka.

Дополнительные сведения об использовании HDInsight в виртуальной сети см. в статье [планирование виртуальной сети для кластеров Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md).

## <a name="connect-to-apache-kafka-from-an-on-premises-network"></a><a id="on-premises"></a> Подключение к Apache Kafka из локальной сети

Чтобы создать кластер Kafka, который взаимодействует с локальной сетью, выполните действия, описанные в статье [Подключение HDInsight к локальной сети](./../connect-on-premises-network.md).

> [!IMPORTANT]  
> При создании кластера HDInsight выберите тип кластера __Kafka__.

Выполнив эти действия, вы создадите следующую конфигурацию:

* Виртуальная сеть Azure
* VPN-шлюз типа "сеть — сеть";
* учетная запись хранения Azure (используемая HDInsight);
* Kafka HDInsight

Чтобы убедиться, что клиент Kafka может подключиться к кластеру из локальной сети, ознакомьтесь с разделом [Пример: клиент Python](#python-client).

## <a name="connect-to-apache-kafka-with-a-vpn-client"></a><a id="vpnclient"></a> Подключение к Apache Kafka с помощью VPN-клиента

В этом разделе описаны действия по созданию следующей конфигурации:

* Виртуальная сеть Azure
* VPN-шлюз типа "точка — сеть"
* Учетная запись хранения Azure (используется HDInsight)
* Kafka HDInsight

1. Дополнительные сведения см. в статье [Создание и экспорт сертификатов для подключений типа "точка — сеть" с помощью PowerShell в Windows 10](../../vpn-gateway/vpn-gateway-certificates-point-to-site.md). Там приведены действия по созданию сертификатов, необходимых для шлюза.

2. Откройте командную строку PowerShell и используйте следующий код для входа в подписку Azure:

    ```powershell
    Connect-AzAccount
    # If you have multiple subscriptions, uncomment to set the subscription
    #Select-AzSubscription -SubscriptionName "name of your subscription"
    ```

3. Используйте следующий код, чтобы создать переменные, которые содержат сведения о конфигурации:

    ```powershell
    # Prompt for generic information
    $resourceGroupName = Read-Host "What is the resource group name?"
    $baseName = Read-Host "What is the base name? It is used to create names for resources, such as 'net-basename' and 'kafka-basename':"
    $location = Read-Host "What Azure Region do you want to create the resources in?"
    $rootCert = Read-Host "What is the file path to the root certificate? It is used to secure the VPN gateway."

    # Prompt for HDInsight credentials
    $adminCreds = Get-Credential -Message "Enter the HTTPS user name and password for the HDInsight cluster" -UserName "admin"
    $sshCreds = Get-Credential -Message "Enter the SSH user name and password for the HDInsight cluster" -UserName "sshuser"

    # Names for Azure resources
    $networkName = "net-$baseName"
    $clusterName = "kafka-$baseName"
    $storageName = "store$baseName" # Can't use dashes in storage names
    $defaultContainerName = $clusterName
    $defaultSubnetName = "default"
    $gatewaySubnetName = "GatewaySubnet"
    $gatewayPublicIpName = "GatewayIp"
    $gatewayIpConfigName = "GatewayConfig"
    $vpnRootCertName = "rootcert"
    $vpnName = "VPNGateway"

    # Network settings
    $networkAddressPrefix = "10.0.0.0/16"
    $defaultSubnetPrefix = "10.0.0.0/24"
    $gatewaySubnetPrefix = "10.0.1.0/24"
    $vpnClientAddressPool = "172.16.201.0/24"

    # HDInsight settings
    $HdiWorkerNodes = 4
    $hdiVersion = "3.6"
    $hdiType = "Kafka"
    ```

4. Используйте следующий код, чтобы создать группу ресурсов и виртуальную сеть Azure:

    ```powershell
    # Create the resource group that contains everything
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create the subnet configuration
    $defaultSubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $defaultSubnetName `
        -AddressPrefix $defaultSubnetPrefix
    $gatewaySubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $gatewaySubnetName `
        -AddressPrefix $gatewaySubnetPrefix

    # Create the subnet
    New-AzVirtualNetwork -Name $networkName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AddressPrefix $networkAddressPrefix `
        -Subnet $defaultSubnetConfig, $gatewaySubnetConfig

    # Get the network & subnet that were created
    $network = Get-AzVirtualNetwork -Name $networkName `
        -ResourceGroupName $resourceGroupName
    $gatewaySubnet = Get-AzVirtualNetworkSubnetConfig -Name $gatewaySubnetName `
        -VirtualNetwork $network
    $defaultSubnet = Get-AzVirtualNetworkSubnetConfig -Name $defaultSubnetName `
        -VirtualNetwork $network

    # Set a dynamic public IP address for the gateway subnet
    $gatewayPublicIp = New-AzPublicIpAddress -Name $gatewayPublicIpName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AllocationMethod Dynamic
    $gatewayIpConfig = New-AzVirtualNetworkGatewayIpConfig -Name $gatewayIpConfigName `
        -Subnet $gatewaySubnet `
        -PublicIpAddress $gatewayPublicIp

    # Get the certificate info
    # Get the full path in case a relative path was passed
    $rootCertFile = Get-ChildItem $rootCert
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($rootCertFile)
    $certBase64 = [System.Convert]::ToBase64String($cert.RawData)
    $p2sRootCert = New-AzVpnClientRootCertificate -Name $vpnRootCertName `
        -PublicCertData $certBase64

    # Create the VPN gateway
    New-AzVirtualNetworkGateway -Name $vpnName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -IpConfigurations $gatewayIpConfig `
        -GatewayType Vpn `
        -VpnType RouteBased `
        -EnableBgp $false `
        -GatewaySku Standard `
        -VpnClientAddressPool $vpnClientAddressPool `
        -VpnClientRootCertificates $p2sRootCert
    ```

    > [!WARNING]  
    > Этот процесс может занять несколько минут.

5. Используйте следующий код, чтобы создать учетную запись хранения Azure и контейнер BLOB-объектов:

    ```powershell
    # Create the storage account
    New-AzStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $storageName `
        -SkuName Standard_GRS `
        -Location $location `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly 1

    # Get the storage account keys and create a context
    $defaultStorageKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName `
        -Name $storageName)[0].Value
    $storageContext = New-AzStorageContext -StorageAccountName $storageName `
        -StorageAccountKey $defaultStorageKey

    # Create the default storage container
    New-AzStorageContainer -Name $defaultContainerName `
        -Context $storageContext
    ```

6. Используйте следующий код, чтобы создать кластер HDInsight:

    ```powershell
    # Create the HDInsight cluster
    New-AzHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $hdiWorkerNodes `
        -ClusterType $hdiType `
        -OSType Linux `
        -Version $hdiVersion `
        -HttpCredential $adminCreds `
        -SshCredential $sshCreds `
        -DefaultStorageAccountName "$storageName.blob.core.windows.net" `
        -DefaultStorageAccountKey $defaultStorageKey `
        -DefaultStorageContainer $defaultContainerName `
        -DisksPerWorkerNode 2 `
        -VirtualNetworkId $network.Id `
        -SubnetName $defaultSubnet.Id
    ```

   > [!WARNING]  
   > Эта процедура занимает около 15 минут.

### <a name="configure-kafka-for-ip-advertising"></a>Настройка Kafka для объявления IP-адресов

По умолчанию Apache Zookeeper возвращает клиентам доменное имя брокеров Kafka. Эта конфигурация не работает для программного VPN-клиента, так как он не может использовать разрешение имен для сущностей в виртуальной сети. Для этой конфигурации выполните следующие действия, чтобы настроить Kafka для объявления IP-адресов вместо доменных имен:

1. Откройте веб-браузер и перейдите по адресу `https://CLUSTERNAME.azurehdinsight.net`. Замените `CLUSTERNAME` именем Kafka в кластере HDInsight.

    При появлении запроса введите имя пользователя и пароль HTTPS для кластера. Отобразится веб-интерфейс Ambari для кластера.

2. Чтобы просмотреть сведения о Kafka, из списка слева выберите __Kafka__.

    ![Список служб с выделенной службой Kafka](./media/apache-kafka-connect-vpn-gateway/select-kafka-service.png)

3. Чтобы просмотреть конфигурацию Kafka, выберите пункт __Configs__ (Конфигурации) в верхней части окна.

    ![Настройка служб Apache Ambari](./media/apache-kafka-connect-vpn-gateway/select-kafka-config1.png)

4. Чтобы найти конфигурацию __kafka-env__, введите `kafka-env` в поле __фильтра__ в правом верхнем углу.

    ![Конфигурация Kafka, kafka-env](./media/apache-kafka-connect-vpn-gateway/search-for-kafka-env.png)

5. Чтобы настроить Kafka для объявления IP-адресов, добавьте следующий текст в нижнюю часть поля __kafka-env template__ (шаблон kafka-env):

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

6. Чтобы настроить интерфейс, через который Kafka ожидает передачи данных, введите `listeners` в поле __фильтра__ в правом верхнем углу.

7. Чтобы настроить Kafka для ожидания передачи данных через все сетевые интерфейсы, измените значение в поле __listeners__ на `PLAINTEXT://0.0.0.0:9092`.

8. Нажмите кнопку __Save__ (Сохранить), чтобы сохранить изменения в конфигурации. Введите текст, описывающий изменения. После сохранения изменений нажмите кнопку __ОК__.

    ![Настройка сохранения Apache Ambari](./media/apache-kafka-connect-vpn-gateway/save-configuration-button.png)

9. Для предотвращения ошибок при перезапуске Kafka нажмите кнопку __Service Actions__ (Действия со службой) и выберите __Turn On Maintenance Mode__ (Включить режим обслуживания). Чтобы завершить эту операцию, нажмите кнопку "ОК".

    ![Кнопка "Service Actions" (Действия со службой) с выделенной командой "Turn On Maintenance Mode" (Включить режим обслуживания)](./media/apache-kafka-connect-vpn-gateway/turn-on-maintenance-mode.png)

10. Чтобы перезапустить Kafka, нажмите кнопку __Restart__ (Перезапустить) и выберите __Restart All Affected__ (Перезапустить все затронутые). Подтвердите перезапуск, а после завершения операции нажмите кнопку __ОК__.

    ![Кнопка "Restart" (Перезапустить) с выделенной командой "Restart All Affected" (Перезапустить все затронутые)](./media/apache-kafka-connect-vpn-gateway/restart-required-button.png)

11. Чтобы отключить режим обслуживания нажмите кнопку __Service Actions__ (Действия со службой) и выберите __Turn Off Maintenance Mode__ (Отключить режим обслуживания). Чтобы завершить эту операцию, нажмите кнопку **ОК**.

### <a name="connect-to-the-vpn-gateway"></a>Подключение к VPN-шлюзу

Для подключения к VPN-шлюзу используйте раздел __Подключение к Azure__ статьи [Настройка подключения типа "точка — сеть"](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md#connect).

## <a name="example-python-client"></a><a id="python-client"></a> Пример: клиент Python

Чтобы проверить подключение к Kafka, выполните следующие действия для создания и запуска производителя и потребителя Python:

1. Чтобы получить полное доменное имя (FQDN) и IP-адреса узлов кластера Kafka, используйте один из следующих методов.

    ```powershell
    $resourceGroupName = "The resource group that contains the virtual network used with HDInsight"

    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName $resourceGroupName | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group <resourcegroupname> --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    В этом сценарии предполагается, что `$resourceGroupName` — это имя группы ресурсов Azure, содержащей виртуальную сеть.

    Сохраните полученные данные для использования на последующих шагах.

2. Чтобы установить клиент [kafka-python](https://kafka-python.readthedocs.io/), используйте следующую команду:

    ```bash
    pip install kafka-python
    ```

3. Чтобы отправить данные в Kafka, используйте следующий код Python:

   ```python
   from kafka import KafkaProducer
   # Replace the `ip_address` entries with the IP address of your worker nodes
   # NOTE: you don't need the full list of worker nodes, just one or two.
   producer = KafkaProducer(bootstrap_servers=['kafka_broker_1','kafka_broker_2'])
   for _ in range(50):
      producer.send('testtopic', b'test message')
   ```

    Замените записи `'kafka_broker'` адресами, полученными на шаге 1 этого раздела.

   * При использовании __программного VPN-клиента__ замените записи `kafka_broker` IP-адресом рабочих узлов.

   * Если вы включили __разрешение имен через пользовательский DNS-сервер__, замените записи `kafka_broker` полным доменным именем рабочих узлов.

     > [!NOTE]
     > Этот код отправляет строку `test message` в раздел `testtopic`. По умолчанию Kafka HDInsight создает раздел, если он не существует.

4. Для получения сообщений от Kafka используйте следующий код Python:

   ```python
   from kafka import KafkaConsumer
   # Replace the `ip_address` entries with the IP address of your worker nodes
   # Again, you only need one or two, not the full list.
   # Note: auto_offset_reset='earliest' resets the starting offset to the beginning
   #       of the topic
   consumer = KafkaConsumer(bootstrap_servers=['kafka_broker_1','kafka_broker_2'],auto_offset_reset='earliest')
   consumer.subscribe(['testtopic'])
   for msg in consumer:
     print (msg)
   ```

    Замените записи `'kafka_broker'` адресами, полученными на шаге 1 этого раздела.

    * При использовании __программного VPN-клиента__ замените записи `kafka_broker` IP-адресом рабочих узлов.

    * Если вы включили __разрешение имен через пользовательский DNS-сервер__, замените записи `kafka_broker` полным доменным именем рабочих узлов.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании HDInsight с виртуальной сетью см. в документе [Планирование развертывания виртуальной сети для кластеров Azure HDInsight](../hdinsight-plan-virtual-network-deployment.md) .

Дополнительные сведения о создании виртуальной сети Azure с VPN-шлюзом типа "точка — сеть" см. в следующих документах:

* [Настройка подключения типа "точка — сеть" к виртуальной сети с помощью портала Azure](../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)

* [Настройка подключения типа "точка — сеть" к виртуальной сети с помощью PowerShell](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

Дополнительные сведения о работе с Apache Kafka HDInsight см. в следующих документах:

* [Get started with Apache Kafka on HDInsight (preview)](apache-kafka-get-started.md) (Приступая к работе с Apache Kafka в HDInsight (предварительная версия))
* [Использование репликации разделов Apache Kafka с помощью Kafka в HDInsight и MirrorMaker](apache-kafka-mirroring.md)
