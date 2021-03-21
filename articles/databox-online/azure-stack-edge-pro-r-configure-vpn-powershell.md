---
title: Настройка VPN на устройстве Azure Stack ребра Pro R с помощью Azure PowerShell
description: В этой статье описано, как настроить VPN на устройстве с Azure Stack пограничным Pro R с помощью скрипта Azure PowerShell для создания ресурсов Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 10/23/2020
ms.author: alkohli
ms.openlocfilehash: 66edd4cad5b2f38696ef1df2030687bf4c7d9956
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102634181"
---
# <a name="configure-vpn-on-your-azure-stack-edge-pro-r-device-via-azure-powershell"></a>Настройка VPN на устройстве Azure Stack ребра Pro R с помощью Azure PowerShell

<!--[!INCLUDE [applies-to-r-skus](../../includes/azure-stack-edge-applies-to-r-sku.md)]-->

VPN-подключение обеспечивает второй уровень шифрования для передачи данных по *протоколу TLS* с устройства Azure Stack пограничной Pro R в Azure. VPN можно настроить на устройстве Azure Stack пограничной Pro R через портал Azure или через Azure PowerShell.

В этой статье описываются шаги, необходимые для настройки VPN на устройстве Azure Stack пограничной Pro R с помощью Azure PowerShell для создания конфигурации в облаке.

## <a name="about-vpn-setup"></a>Сведения о настройке VPN

VPN-подключение между локальными сетями состоит из VPN-шлюза Azure, локального VPN-устройства и VPN-туннеля IPsec S2S, соединяющего шлюз и устройство. Стандартный рабочий процесс включает следующие этапы.

- Настройте необходимые компоненты.
- Настройте необходимые ресурсы в Azure.
    - Создание и настройка VPN-шлюза Azure (шлюза виртуальной сети).
    - Создайте и настройте шлюз локальной сети Azure, который представляет локальную сеть и VPN-устройство.
    - Создание и настройка VPN-подключения Azure между VPN-шлюзом Azure и шлюзом локальной сети.
    - Настройка брандмауэра Azure и добавление правил сети и приложений.
    - Создайте таблицу маршрутизации Azure и добавьте маршруты.
- Настройте VPN в локальном веб-ИНТЕРФЕЙСе устройства. Настройте локальное VPN-устройство, представленное локальным сетевым шлюзом, чтобы установить фактический VPN-туннель S2S с VPN-шлюзом Azure.

Подробные инструкции приведены в следующих разделах.

## <a name="configure-prerequisites"></a>Настройка необходимых компонентов

1. У вас должен быть доступ к устройству с Azure Stack пограничной Pro R, установленному в соответствии с инструкциями в [статье Установка устройства с правами на установку Azure Stack для Pro r](azure-stack-edge-pro-r-deploy-install.md). Это устройство будет использоваться как локальное VPN-устройство для создания VPN-подключения к Azure. 

2. VPN-устройство должно иметь статический общедоступный IP-адрес (внешний). Этот адрес не должен быть NAT.

3. У вас должен быть доступ к допустимой подписке Azure, которая включена для службы Azure Stack ребра в Azure. Используйте эту подписку, чтобы создать соответствующий ресурс в Azure для управления устройством с Azure Stack погранично Pro R.  

4. У вас есть доступ к клиенту Windows, который вы будете использовать для доступа к устройству R Azure Stack ребра Pro. Этот клиент будет использоваться для программного создания конфигурации в облаке.

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
        Укажите имя подписки Azure, которое вы используете для устройства R Azure Stack ребра Pro для настройки VPN.

    3. [Скачайте скрипт,](https://aka.ms/ase-vpn-deployment) необходимый для создания конфигурации в облаке. Он выполняет следующие действия:
        
        - Создайте виртуальную сеть Azure и следующие подсети: *GatewaySubnet* и *азурефиреваллсубнет*.
        - Создание и настройка VPN-шлюза Azure.
        - Создание и Настройка шлюза локальной сети Azure.
        - Создание и настройка VPN-подключения Azure между VPN-шлюзом Azure и шлюзом локальной сети.
        - Создайте брандмауэр Azure и добавьте правила сети, правила приложений.
        - Создайте таблицу маршрутизации Azure и добавьте в нее маршруты.


## <a name="use-the-script"></a>Использование скрипта

Сначала измените файл, `parameters.json` чтобы ввести параметры. Затем запустите скрипт, используя измененный файл JSON.

Каждое из этих действий описано в следующих разделах.

### <a name="download-service-tags-file"></a>Скачать файл тегов службы

Возможно, у вас уже есть `ServiceTags.json` файл в папке, в которую был скачан скрипт. В противном случае можно скачать файл тегов службы.

[!INCLUDE [azure-stack-edge-gateway-download-service-tags](../../includes/azure-stack-edge-gateway-download-service-tags.md)]

### <a name="modify-parameters-file"></a>Изменение файла параметров

Первым шагом является изменение `parameters.json` файла и сохранение изменений. 


Для создаваемых ресурсов Azure можно указать следующие имена:

|Имя параметра  |Описание  |
|---------|---------|
|virtualNetworks_vnet_name    | Имя виртуальной сети Azure        |
|azureFirewalls_firewall_name     | Имя брандмауэра Azure        |
|routeTables_routetable_name     | Имя таблицы маршрутов Azure        |
|publicIPAddresses_VNGW_public_ip_name     | Имя общедоступного IP-адреса для шлюза виртуальной сети       |
|virtualNetworkGateways_VNGW_name    | Имя VPN-шлюза Azure (шлюз виртуальной сети)        |
|publicIPAddresses_firewall_public_ip_name     | Имя общедоступного IP-адреса для брандмауэра Azure         |
|localNetworkGateways_LNGW_name    |Имя шлюза локальной сети Azure          |
|connections_vngw_lngw_name    | Имя VPN-подключения Azure. Это подключение между шлюзом виртуальной сети и локальным сетевым шлюзом.       |
|location     |Это регион, в котором вы хотите создать виртуальную сеть. Выберите тот же регион, который соответствует устройству.         |

Следующие IP-адреса и адресные пространства относятся к ресурсам Azure, которые создаются, включая виртуальную сеть и связанные подсети (по умолчанию, брандмауэр, GatewaySubnet).

|Имя параметра  |Описание  |
|---------|---------|
|VnetIPv4AddressSpace    | Это адресное пространство, связанное с виртуальной сетью.       |
|DefaultSubnetIPv4AddressSpace    |Это адресное пространство, связанное с `Default` подсетью виртуальной сети.         |
|FirewallSubnetIPv4AddressSpace    |Это адресное пространство, связанное с `Firewall` подсетью виртуальной сети.          |
|GatewaySubnetIPv4AddressSpace    |Это адресное пространство, связанное с `GatewaySubnet` виртуальной сетью.          |
|GatewaySubnetIPv4bgpPeeringAddress    | Это IP-адрес, зарезервированный для связи BGP и основанный на адресном пространстве, связанном с `GatewaySubnet` виртуальной сетью.          |

Следующие IP-адреса и адресные пространства относятся к локальной сети (где развернуто устройство R Azure Stack ребра Pro).

|Имя параметра  |Описание  |
|---------|---------|
|кустомернетворкаддрессспаце    |  Это адресное пространство для частного IP-адреса.       |
|кустомерпубликнетворкаддрессспаце    |  Это адресное пространство для общедоступного IP-адреса.       |
|дбеиотнетворкаддрессспаце    |Этот IP-адрес зарезервирован службой IoT. Не изменяйте этот параметр.        |
|азуревпншаредкэй    |Этот общий ключ используется во время создания ресурса VPN-подключения Azure. Этот ключ также предоставляется в качестве общего секрета VPN во время настройки локальной виртуальной сети веб-интерфейса.         |
|DBE-Gateway-IPAddress   | Общедоступный IP-адрес для устройства R Azure Stack ребра Pro. Это может быть неизвестным, и вы можете запустить сценарий с заполнительным IP-адресом. Измените шлюз локальной сети позже, указав фактический IP-адрес.     |

#### <a name="caveats-to-keep-in-mind"></a>Помните, что следует помнить:

- У вас нет IP-адреса устройства R для Azure Stack ребра Pro. Вы можете использовать заполнительный IP-адрес для создания ресурса и позже изменить шлюз локальной сети Azure, чтобы назначить фактический IP-адрес устройства и адресное пространство локальной сети для устройства.
- В зависимости от того, откуда предназначается IETF в Интернете (IANA), используйте любую подсеть из 172.16. x. y в 172.24. з.а. Мы резервируем диапазоны IPv4-адресов 172,24 для сети Azure.

### <a name="run-the-script"></a>Выполнение скрипта

Выполните следующие действия, чтобы использовать измененный `parameters.json` и запускаемый скрипт для создания ресурсов Azure.

1. Откройте сеанс PowerShell от имени администратора.

2. Перейдите в каталог, в котором находится скрипт.

3. Выполните скрипт.

    `.\AzDeployVpn.ps1 -Location <Location> -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "<Service tag json file>"  -ResourceGroupName "<Resource group name>" -AzureDeploymentName "<Deployment name>" -NetworkRuleCollectionName "<Name for collection of network rules>" -Priority 115 -AppRuleCollectionName "<Name for collection of app rules>"`

    Результат выполнения команды показан ниже.

    `.\AzDeployVpn.ps1 -Location eastus -AzureAppRuleFilePath "appRule.json" -AzureIPRangesFilePath "ServiceTags_Public_20191216.json"  -ResourceGroupName "devtestrg4" -AzureDeploymentName "dbetestdeployment20" -NetworkRuleCollectionName "testnrc20" -Priority 115 -AppRuleCollectionName "testarc20"`

    Выполнение сценария занимает примерно 90 минут. После завершения скрипта создается журнал развертывания в той же папке, в которой находится скрипт.


## <a name="verify-the-azure-resources"></a>Проверка ресурсов Azure

После успешного выполнения скрипта убедитесь, что все ресурсы были созданы в Azure.

Далее вы настроите VPN в локальном веб-ИНТЕРФЕЙСе вашего устройства.


## <a name="vpn-configuration-on-the-device"></a>Конфигурация VPN на устройстве

[!INCLUDE [azure-stack-edge-gateway-configure-vpn-local-ui](../../includes/azure-stack-edge-gateway-configure-vpn-local-ui.md)]


## <a name="verify-vpn"></a>Проверка VPN

[!INCLUDE [azure-stack-edge-gateway-verify-vpn](../../includes/azure-stack-edge-gateway-verify-vpn.md)]

## <a name="validate-data-transfer-through-vpn"></a>Проверка передаваемых данных через VPN

Чтобы убедиться, что VPN работает, скопируйте данные в общую папку SMB. Выполните действия, описанные в разделе [Добавление общей папки](azure-stack-edge-gpu-manage-shares.md#add-a-share) на устройство Azure Stack пограничной Pro R. 

1. Скопируйте файл, например \data\pictures\waterfall.jpg, в общую папку SMB, подключенную к клиентской системе. 
2. Убедитесь, что этот файл отображается в вашей учетной записи хранения в облаке.

Чтобы проверить, что данные проходят через VPN:

1. Откройте ресурс подключения, имеющийся в группе ресурсов. 

2. Проверьте данные в значениях и исходящих данных.
 
3. Откройте шлюз виртуальной сети в группе ресурсов. Просмотрите диаграммы для **общих** **входящих** и исходящих туннелей.
 
## <a name="debug-issues"></a>Отладка проблем

Для отладки любых проблем используйте следующие команды:

```
Get-AzResourceGroupDeployment -DeploymentName $deploymentName -ResourceGroupName $ResourceGroupName
```

Ниже приведен пример выходных данных:


```azurepowershell
PS C:\Projects\VPN\Azure-VpnDeployment> Get-AzResourceGroupDeployment -DeploymentName "aseprorvpnrg14_deployment" -ResourceGroupName "aseprorvpnrg14"


DeploymentName          : aseprorvpnrg14_deployment
ResourceGroupName       : aseprorvpnrg14
ProvisioningState       : Succeeded
Timestamp               : 10/21/2020 6:23:13 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name                                         Type                       Value
                          ===========================================  =========================  ==========
                          virtualNetworks_vnet_name                    String                     aseprorvpnrg14_vnet
                          azureFirewalls_firewall_name                 String                     aseprorvpnrg14_firewall
                          routeTables_routetable_name                  String                     aseprorvpnrg14_routetable
                          publicIPAddresses_VNGW_public_ip_name        String                     aseprorvpnrg14_vngwpublicip
                          virtualNetworkGateways_VNGW_name             String                     aseprorvpnrg14_vngw
                          publicIPAddresses_firewall_public_ip_name    String                     aseprorvpnrg14_fwpip
                          localNetworkGateways_LNGW_name               String                     aseprorvpnrg14_lngw
                          connections_vngw_lngw_name                   String                     aseprorvpnrg14_connection
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

## <a name="next-steps"></a>Следующие шаги

[Настройка VPN с помощью локального интерфейса пользователя на устройстве с Azure Stack ребра Pro R](azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption.md#configure-vpn)
