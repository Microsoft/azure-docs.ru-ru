---
title: Настройка VPN на Azure Stack пограничном устройстве R с помощью Azure PowerShell
description: Описание настройки VPN на Azure Stack пограничном устройстве R с помощью скрипта Azure PowerShell для создания ресурсов Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 11/17/2020
ms.author: alkohli
ms.openlocfilehash: 9fa4c678a04342b47601f81ede7c49ab841f42ba
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102630968"
---
# <a name="configure-vpn-on-your-azure-stack-edge-mini-r-device-via-azure-powershell"></a>Настройка VPN на Azure Stack пограничном устройстве R с помощью Azure PowerShell

<!--[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]-->

VPN-подключение обеспечивает второй уровень шифрования для передачи данных по *протоколу TLS* из Azure Stack пограничных устройств с мини-r или Azure Stack ребра Pro r в Azure. VPN можно настроить на Azure Stack пограничном устройстве R с помощью портал Azure или с помощью Azure PowerShell. 

В этой статье описываются шаги, необходимые для настройки VPN-подключения типа "точка — сеть" (P2S) на Azure Stack пограничном устройстве R с помощью скрипта Azure PowerShell для создания конфигурации в облаке. Конфигурация на Azure Stack пограничном устройстве выполняется через локальный пользовательский интерфейс.

## <a name="about-vpn-setup"></a>Сведения о настройке VPN

Подключение к VPN-шлюзу P2S позволяет создать безопасное подключение к виртуальной сети с отдельного клиентского компьютера или с мини-устройства на Azure Stack пограничном устройстве R. Вы запускаете подключение P2S с клиентского компьютера или устройства. В этом случае в подключении P2S используется VPN-решение IKEv2, основанное на стандартах IPsec.

Стандартный рабочий процесс включает следующие этапы.

1. Настройте необходимые компоненты.
2. Настройте необходимые ресурсы в Azure.
    1. Создайте и настройте виртуальную сеть и необходимые подсети. 
    2. Создание и настройка VPN-шлюза Azure (шлюза виртуальной сети).
    3. Настройка брандмауэра Azure и добавление правил сети и приложений.
    4. Создание таблиц маршрутизации Azure и добавление маршрутов.
    5. Включите подключение типа "точка — сеть" в VPN-шлюз.
        1. Добавьте пулы адресов клиента.
        2. Настройте тип туннеля.
        3. Настройте тип проверки подлинности.
        4. Создать сертификат.
        5. Отправьте сертификат.
    6. Скачайте телефонную книгу.
3. Настройте VPN в локальном веб-ИНТЕРФЕЙСе устройства. 
    1. Укажите телефонную книгу.
    2. Укажите файл тегов служб (JSON).


Подробные инструкции приведены в следующих разделах.

## <a name="configure-prerequisites"></a>Настройка необходимых компонентов

- У вас должен быть доступ к устройству мини R Azure Stack пограничных устройств, которое установлено в соответствии с инструкциями в [статье Установка устройства мини r с Azure Stack](azure-stack-edge-mini-r-deploy-install.md). Это устройство будет устанавливать подключение P2S к Azure. 

- У вас должен быть доступ к допустимой подписке Azure, которая включена для службы Azure Stack ребра в Azure. Используйте эту подписку, чтобы создать соответствующий ресурс в Azure для управления устройством с мини-R Azure Stack.  

- У вас есть доступ к клиенту Windows, который будет использоваться для доступа к мини-устройству с Azure Stack пограничным устройством R. Этот клиент будет использоваться для программного создания конфигурации в облаке.

    1. Чтобы установить требуемую версию PowerShell на клиенте Windows, выполните следующие команды:

        ```azurepowershell
        Install-Module -Name Az -AllowClobber -Scope CurrentUser 
        Import-Module Az.Accounts
        ```
    2. Чтобы подключиться к учетной записи Azure и подписке, выполните следующие команды:

        ```azurepowershell
        Connect-AzAccount 
        Set-AzContext -Subscription "<Your subscription name>"
        ```
        Укажите имя подписки Azure, которое вы используете с помощью мини-устройства R Azure Stack погранично, чтобы настроить VPN.

    3. [Скачайте скрипт,](https://aka.ms/ase-vpn-deployment) необходимый для создания конфигурации в облаке. Он выполняет следующие действия:
        
        - Создайте виртуальную сеть Azure и следующие подсети: *GatewaySubnet* и *азурефиреваллсубнет*.
        - Создание и настройка VPN-шлюза Azure.
        - Создание и Настройка шлюза локальной сети Azure.
        - Создание и настройка VPN-подключения Azure между VPN-шлюзом Azure и шлюзом локальной сети.
        - Создайте брандмауэр Azure и добавьте правила сети, правила приложений.
        - Создайте таблицу маршрутизации Azure и добавьте в нее маршруты.

    4. Создайте группу ресурсов в портал Azure, в которой необходимо создать ресурсы Azure. Перейдите к списку служб в портал Azure, выберите **Группа ресурсов** , а затем щелкните **+ Добавить**. Укажите сведения о подписке и имя группы ресурсов, а затем щелкните **создать**. Если вы перейдете в эту группу ресурсов, в настоящее время в ней не должно быть ресурсов.

        ![Группа ресурсов Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-resource-group-1.png)
    
    5. Вам потребуется сертификат с основанием Base 64 в `.cer` формате для мини-устройства с Azure Stackным пограничным устройством R. Этот сертификат должен быть отправлен на устройство Azure Stack пограничной, как `pfx` и закрытый ключ. Этот сертификат также необходимо установить в доверенном корневом каталоге хранилища на клиенте, который пытается установить подключение P2S.

## <a name="use-the-script"></a>Использование скрипта

Сначала измените файл, `parameters-p2s.json` чтобы ввести параметры. Затем запустите скрипт, используя измененный файл JSON.

Каждое из этих действий описано в следующих разделах.

### <a name="download-service-tags-file"></a>Скачать файл тегов службы

Возможно, у вас уже есть `ServiceTags.json` файл в папке, в которую был скачан скрипт. В противном случае можно скачать файл тегов службы.

[!INCLUDE [azure-stack-edge-gateway-download-service-tags](../../includes/azure-stack-edge-gateway-download-service-tags.md)]

### <a name="modify-parameters-file"></a>Изменение файла параметров

Первым шагом является изменение `parameters-p2s.json` файла и сохранение изменений. 

Для создаваемых ресурсов Azure можно указать следующие имена:

|Имя параметра  |Описание  |
|---------|---------|
|virtualNetworks_vnet_name    | Имя виртуальной сети Azure        |
|azureFirewalls_firewall_name     | Имя брандмауэра Azure        |
|routeTables_routetable_name     | Имя таблицы маршрутов Azure        |
|publicIPAddresses_VNGW_public_ip_name     | Имя общедоступного IP-адреса для шлюза виртуальной сети       |
|virtualNetworkGateways_VNGW_name    | Имя VPN-шлюза Azure (шлюз виртуальной сети)        |
|publicIPAddresses_firewall_public_ip_name     | Имя общедоступного IP-адреса для брандмауэра Azure         |
|location     |Это регион, в котором вы хотите создать виртуальную сеть. Выберите тот же регион, который соответствует устройству.         |
|RouteTables_routetable_onprem_name| Это имя дополнительной таблицы маршрутов, которая помогает брандмауэру перенаправлять пакеты обратно на Azure Stack пограничное устройство. Скрипт создает два дополнительных маршрута и связывает *по умолчанию* и *фиреваллсубнет* с этой таблицей маршрутов.|

Укажите следующие IP-адреса и адресные пространства для ресурсов Azure, которые создаются, включая виртуальную сеть и связанные подсети (*по умолчанию*, *брандмауэр*, *GatewaySubnet*).

|Имя параметра  |Описание  |
|---------|---------|
|VnetIPv4AddressSpace    | Это адресное пространство, связанное с виртуальной сетью. Укажите диапазон IP-адресов виртуальной сети в качестве диапазона частных IP-адресов ( https://en.wikipedia.org/wiki/Private_network#Private_IPv4_addresses) .     |
|DefaultSubnetIPv4AddressSpace    |Это адресное пространство, связанное с `Default` подсетью виртуальной сети.         |
|FirewallSubnetIPv4AddressSpace    |Это адресное пространство, связанное с `Firewall` подсетью виртуальной сети.          |
|GatewaySubnetIPv4AddressSpace    |Это адресное пространство, связанное с `GatewaySubnet` виртуальной сетью.          |
|GatewaySubnetIPv4bgpPeeringAddress    | Это IP-адрес, зарезервированный для связи BGP и основанный на адресном пространстве, связанном с `GatewaySubnet` виртуальной сетью.          |
|клиентаддресспул    | Этот IP-адрес используется для пула адресов в конфигурации P2S в портал Azure.         |
|публикцертдата     | Данные общедоступного сертификата используются VPN-шлюзом для проверки подлинности P2S клиентов, подключающихся к нему. Чтобы получить данные сертификата, установите корневой сертификат. Убедитесь, что сертификат является Base-64 в кодировке с расширением CER. Откройте этот сертификат и скопируйте в него текст в поле сертификат между = = начальный сертификат = = и = = конечный сертификат = = в одной непрерывной строке.     |



### <a name="run-the-script"></a>Выполнение скрипта

Выполните следующие действия, чтобы использовать измененный `parameters-p2s.json` и запускаемый скрипт для создания ресурсов Azure.

1. Запустите PowerShell. Перейдите в каталог, в котором находится скрипт.

3. Выполните скрипт.

    `.\AzDeployVpn.ps1 -Location <Location> -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "<Service tag json file>"  -ResourceGroupName "<Resource group name>" -AzureDeploymentName "<Deployment name>" -NetworkRuleCollectionName "<Name for collection of network rules>" -Priority 115 -AppRuleCollectionName "<Name for collection of app rules>"`

    > [!NOTE]
    > В этом выпуске сценарий работает только в регионе "Восточная часть США".

    При выполнении скрипта необходимо ввести следующие сведения:

    
    |Параметр  |Описание  |
    |---------|---------|
    |Расположение     |Это регион, в котором должны быть созданы ресурсы Azure.         |
    |азуреаппрулефилепас     | Это путь к файлу для `appRule.json` .       |
    |азуреипранжесфилепас     |Это JSON-файл тега службы, скачанный на предыдущем шаге.         |
    |ResourceGroupName     | Это имя группы ресурсов, в которой создаются все ресурсы Azure.        |
    |азуредеплойментнаме    |Это имя для развертывания Azure.         |
    |нетворкрулеколлектионнаме            | Это имя для коллекции всех правил сети, которые создаются и добавляются в брандмауэр Azure.             |
    |Приоритет            | Это приоритет, назначенный всем создаваемым правилам сети и приложений.              |
    |аппрулеколлектионнаме            |Это имя для коллекции всех правил приложения, создаваемых и добавленных в брандмауэр Azure.                |


    Результат выполнения команды показан ниже.
    
    ```powershell
    PS C:\Offline docs\AzureVpnConfigurationScripts> .\AzDeployVpn.ps1 -Location eastus -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath ".\ServiceTags_Public_20200203.json" -ResourceGroupName "mytmarg3" -AzureDeploymentName "tmap2stestdeploy1" -NetworkRuleCollectionName "testnrc1" -Priority 115 -AppRuleCollectionName "testarc2"
        validating vpn deployment parameters
        Starting vpn deployment
        C:\Offline docs\AzureVpnConfigurationScripts\parameters-p2s.json
        C:\Offline docs\AzureVpnConfigurationScripts\template-p2s.json
        vpn deployment: tmap2stestdeploy1 started and status: Running
        Waiting for vpn deployment completion....
        ==== CUT ==================== CUT ==============
        Adding route 191.236.0.0/18 for AzureCloud.eastus
        Adding route 191.237.0.0/17 for AzureCloud.eastus
        Adding route 191.238.0.0/18 for AzureCloud.eastus
        Total Routes:294, Existing Routes: 74, New Routes Added: 220
        Additional routes getting added
    ```    

    > [!IMPORTANT]
    > - Выполнение сценария занимает примерно 90 минут. Перед запуском сценария обязательно выполните вход в сеть.
    > - Если по какой-либо причине произошел сбой сеанса с скриптом, удалите группу ресурсов, чтобы удалить все созданные в ней ресурсы.
  
    
    После завершения скрипта создается журнал развертывания в той же папке, в которой находится скрипт.


## <a name="verify-the-azure-resources"></a>Проверка ресурсов Azure

После успешного выполнения скрипта убедитесь, что все ресурсы были созданы в Azure. Перейдите к созданной группе ресурсов. Вы должны увидеть следующие ресурсы:

![Ресурсы Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/script-resources.png)


<!--## Enable point-to-site in VPN gateway

1. In the Azure portal, go to the resource group and then select the virtual network gateway that you created in the earlier step.

    ![Azure virtual network gateway](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-virtual-network-gateway.png)

2. Go to **Settings > Point-to-site configuration**. Select **Configure now**.

    ![Enable P2S configuration](media/azure-stack-edge-mini-r-configure-vpn-powershell/enable-p2s.png)


3.  On the **Point-to-site configuration** blade:

    1. You'll add the client address pool. This pool is a range of private IP addresses that you specify. The clients that connect over a P2S VPN dynamically receive an IP address from this range. Use a private IP address range that does not overlap with the on-premises location that you connect from, or the VNet that you want to connect to. 
    2. You can select the tunnel type. For the VPN tunnel, you will use the IKEv2 protocol. 
    3. You will now define the type of authentication. Before Azure accepts a P2S VPN connection, the user has to be authenticated first. In this case, you authenticate using the native Azure certificate authentication. Set **Authentication type** to **Azure certificate**.
    4. You will now create and upload the certificates to Azure and your client. You will need to install a root certificate on your VPN gateway. The VPN gateway validates the client certificates when the P2S connection is being established by the client. The root certificate is required for the validation and must be uploaded to Azure.
    
        The clients trying to establish the P2S connection will have a client certificate that authenticates the connecting user.  To create these certificates, follow the steps in [Generate and export certificates for Point-to-Site using PowerShell](../vpn-gateway/vpn-gateway-certificates-point-to-site.md). 

        To install the root certificate, make sure the certificate is Base-64 encoded with a .cer extension. Open this certificate and copy the text in the certificate between ==BEGIN CERTIFICATE== and ==END CERTIFICATE== in one continuous line in the public certificate data under Root certificates.

        To upload the root certificates, follow the detailed steps in [Upload the root certificate public certificate data](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile).
    
    5. Save the configuration.

        ![Save P2S configuration](media/azure-stack-edge-mini-r-configure-vpn-powershell/enable-p2s-config.png)-->

## <a name="download-phone-book-for-vpn-profile"></a>Скачать телефонную книгу для профиля VPN

На этом шаге вы загрузит профиль VPN для своего устройства.

1. В портал Azure перейдите в группу ресурсов, а затем выберите шлюз виртуальной сети, созданный на предыдущем шаге.

    ![Шлюз виртуальной сети Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/azure-virtual-network-gateway.png)

2. Перейдите в раздел **параметры > конфигурация "точка — сеть**". Выберите **скачать VPN-клиент**.

    ![Включить конфигурацию P2S 1](media/azure-stack-edge-mini-r-configure-vpn-powershell/download-vpn-client.png)

2. Сохраните сжатый профиль и извлеките его на клиенте Windows.

    ![Включить конфигурацию P2S 2](media/azure-stack-edge-mini-r-configure-vpn-powershell/save-extract-profile.png)

3. Перейдите в папку *WindowsAmd64* и извлеките `.exe` : *VpnClientSetupAmd64.exe*.

    ![Включить конфигурацию P2S 3](media/azure-stack-edge-mini-r-configure-vpn-powershell/extract-exe.png)

3. Создайте временный путь. Пример.

    `C:\NewTemp\vnet\tmp`

4. Запустите PowerShell и перейдите в каталог, в котором находится `.exe` . Чтобы выполнить `.exe` , введите:

    `.\VpnClientSetupAmd64.exe /Q /C /T:"C:\NewTemp\vnet\tmp"`

5. Временный путь будет содержать новые файлы. Пример выходных данных:

    ```powershell
    
    PS C:\windows\system32> cd "C:\Users\Ase\Downloads\vngw5\WindowsAmd64"
    PS C:\Users\Ase\Downloads\vngw5\WindowsAmd64> .\VpnClientSetupAmd64.exe /Q /C /T:"C:\NewTemp\vnet\tmp"
    PS C:\Users\Ase\Downloads\vngw5\WindowsAmd64> cd "C:\NewTemp\vnet"
    PS C:\NewTemp\vnet> ls .\tmp
    
        Directory: C:\NewTemp\vnet\tmp
    
    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    -a----         2/6/2020   6:18 PM            947 8c670077-470b-421a-8dd8-8cedb4f2f08a.cer
    -a----         2/6/2020   6:18 PM            155 8c670077-470b-421a-8dd8-8cedb4f2f08a.cmp
    -a----         2/6/2020   6:18 PM           3564 8c670077-470b-421a-8dd8-8cedb4f2f08a.cms
    -a----         2/6/2020   6:18 PM          11535 8c670077-470b-421a-8dd8-8cedb4f2f08a.inf
    -a----         2/6/2020   6:18 PM           2285 8c670077-470b-421a-8dd8-8cedb4f2f08a.pbk
    -a----         2/6/2020   6:18 PM           5430 azurebox16.ico
    -a----         2/6/2020   6:18 PM           4286 azurebox32.ico
    -a----         2/6/2020   6:18 PM         138934 azurevpnbanner.bmp
    -a----         2/6/2020   6:18 PM          46064 cmroute.dll
    -a----         2/6/2020   6:18 PM            196 routes.txt
    
    PS C:\NewTemp\vnet>
    ```

6. *PBK* -файл — это телефонная книга для профиля VPN. Он будет использоваться в локальном пользовательском интерфейсе.


## <a name="vpn-configuration-on-the-device"></a>Конфигурация VPN на устройстве

Выполните следующие действия в локальном пользовательском интерфейсе устройства Azure Stack пограничной Организации.

1. В локальном пользовательском интерфейсе перейдите на страницу **VPN** . В разделе Состояние VPN выберите **настроить**.

    ![Настройка VPN 1](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-1.png)

2. В колонке **Настройка VPN** выполните следующие действия:
    
    1. В файле отправить телефонную книгу укажите файл. pbk, созданный на предыдущем шаге.
    2. В файле конфигурации списка отправки общедоступных IP-адресов укажите в качестве входных данных JSON-файл центра обработки данных Azure. Этот файл был скачан на предыдущем шаге с: [https://www.microsoft.com/download/details.aspx?id=56519](https://www.microsoft.com/download/details.aspx?id=56519) .
    3. Выберите **eastus** в качестве региона и нажмите кнопку **Применить**.

    ![Настройка VPN 2](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-2.png)

3. В разделе **диапазоны IP-адресов, к которым будет осуществляться доступ с использованием только VPN** , введите диапазон IPv4 виртуальной сети, который был выбран для вашей виртуальной сетью Azure.

    ![Настройка VPN 3](media/azure-stack-edge-mini-r-configure-vpn-powershell/configure-vpn-3.png)

## <a name="verify-client-connection"></a>Проверка подключения клиента

1. В портал Azure перейдите к VPN-шлюзу.
2. Перейдите в раздел **параметры > конфигурация "точка — сеть**". В разделе **выделенные IP-адреса** ОТОБРАЗИТСЯ IP-адрес устройства Azure Stack пограничных устройств.

## <a name="validate-data-transfer-through-vpn"></a>Проверка передаваемых данных через VPN

Чтобы убедиться, что VPN работает, скопируйте данные в общую папку SMB. Выполните действия, описанные в разделе [Добавление общей папки](azure-stack-edge-gpu-manage-shares.md#add-a-share) на Azure Stack пограничной устройство. 

1. Скопируйте файл, например \data\pictures\waterfall.jpg, в общую папку SMB, подключенную к клиентской системе. 
2. Чтобы проверить, что данные проходят через VPN, во время копирования данных:

    1. Перейдите к VPN-шлюзу в портал Azure. 

    2. Перейдите к разделу **мониторинг > метрики**.

    3. На правой панели выберите **область** в качестве VPN-шлюза, **метрику** в качестве шлюза P2S пропускную способность и **агрегирование** как AVG.

    4. При копировании данных вы увидите увеличение уровня использования пропускной способности, а также после завершения копирования данных использование пропускной способности будет удалено.

        ![Метрики VPN Azure](media/azure-stack-edge-mini-r-configure-vpn-powershell/vpn-metrics-1.png)

3. Убедитесь, что этот файл отображается в вашей учетной записи хранения в облаке.
 
## <a name="debug-issues"></a>Отладка проблем

Для отладки любых проблем используйте следующие команды:

```
Get-AzResourceGroupDeployment -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
```

Ниже приведен пример выходных данных:


```azurepowershell
PS C:\Projects\TZL\VPN\Azure-VpnDeployment> Get-AzResourceGroupDeployment -DeploymentName "tznvpnrg14_deployment" -ResourceGroupName "tznvpnrg14"


DeploymentName          : tznvpnrg14_deployment
ResourceGroupName       : tznvpnrg14
ProvisioningState       : Succeeded
Timestamp               : 1/21/2020 6:23:13 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                                         Type                       Value
                          ===========================================  =========================  ==========
                          virtualNetworks_vnet_name                    String                     tznvpnrg14_vnet
                          azureFirewalls_firewall_name                 String                     tznvpnrg14_firewall
                          routeTables_routetable_name                  String                     tznvpnrg14_routetable
                          publicIPAddresses_VNGW_public_ip_name        String                     tznvpnrg14_vngwpublicip
                          virtualNetworkGateways_VNGW_name             String                     tznvpnrg14_vngw
                          publicIPAddresses_firewall_public_ip_name    String                     tznvpnrg14_fwpip
                          localNetworkGateways_LNGW_name               String                     tznvpnrg14_lngw
                          connections_vngw_lngw_name                   String                     tznvpnrg14_connection
                          location                                     String                     East US
                          vnetIPv4AddressSpace                         String                     172.24.0.0/16
                          defaultSubnetIPv4AddressSpace                String                     172.24.0.0/24
                          firewallSubnetIPv4AddressSpace               String                     172.24.1.0/24
                          gatewaySubnetIPv4AddressSpace                String                     172.24.2.0/24
                          gatewaySubnetIPv4bgpPeeringAddress           String                     172.24.2.254
                          customerNetworkAddressSpace                  String                     10.0.0.0/18
                          customerPublicNetworkAddressSpace            String                     207.68.128.0/24
                          dbeIOTNetworkAddressSpace                    String                     10.139.218.0/24
                          azureVPNsharedKey                            String                     1234567890
                          dbE-Gateway-ipaddress                        String                     207.68.128.113

Outputs                 :
                          Name                     Type                       Value
                          =======================  =========================  ==========
                          virtualNetwork           Object                     {
                            "provisioningState": "Succeeded",
                            "resourceGuid": "dcf673d3-5c73-4764-b077-77125eda1303",
                            "addressSpace": {
                              "addressPrefixes": [
                                "172.24.0.0/16"
                              ]
================= CUT ============================= CUT ===========================
```


```
Get-AzResourceGroupDeploymentOperation -ResourceGroupName $ResourceGroupName -DeploymentName $AzureDeploymentName
```

## <a name="next-steps"></a>Дальнейшие действия

[Настройте VPN через локальный пользовательский интерфейс на устройстве Azure Stack](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md#configure-vpn).