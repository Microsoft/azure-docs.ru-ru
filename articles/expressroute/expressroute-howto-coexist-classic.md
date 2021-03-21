---
title: 'Настройка параллельных подключений ExpressRoute и S2S VPN: классическая модель'
description: В этой статье описывается настройка параллельных подключений ExpressRoute и VPN типа "сеть-сеть" для классической модели развертывания.
documentationcenter: na
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: duau
ms.openlocfilehash: 70407d726b697dd6c4dfb6777cf6c445092d3b73
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92206331"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-classic"></a>Настройка параллельных подключений ExpressRoute и "сайт — сайт" (классическая версия)
> [!div class="op_single_selector"]
> * [PowerShell — Resource Manager](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell — классическая модель](expressroute-howto-coexist-classic.md)
> 
> 

Сведения в этой статье помогут настроить параллельные соединения ExpressRoute и соединения VPN типа "сеть — сеть". Возможность настройки VPN типа "сеть-сеть" и ExpressRoute дает целый ряд преимуществ. Вы можете настроить VPN-подключение "сеть — сеть" как защищенный путь отработки отказа для ExressRoute или использовать эту сеть VPN для подключения к сайтам, не подключенным через ExpressRoute. В этой статье мы рассмотрим порядок действия в каждом из этих вариантов. Эта статья относится к модели классического развертывания. Эта конфигурация недоступна на портале.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**О моделях развертывания Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

> [!IMPORTANT]
> Перед выполнением приведенных ниже инструкций необходимо настроить каналы ExpressRoute. Прежде чем следовать дальше, выполните инструкции по [созданию канала ExpressRoute](expressroute-howto-circuit-classic.md) и [настройке маршрутизации](expressroute-howto-routing-classic.md).
> 
> 

## <a name="limits-and-limitations"></a>Квоты и ограничения
* **Транзитная маршрутизация не поддерживается.** Нельзя настроить маршрутизацию (через Azure) между локальной сетью, подключенной к VPN типа "сеть — сеть", и локальной сетью, подключенной к ExpressRoute.
* **Соединение типа "точка — сеть" не поддерживается.** Невозможно включить подключения VPN "точка — сеть" к одной виртуальной сети, подключенной к ExpressRoute. VPN типа "точка-сеть" и ExpressRoute не могут существовать в одной и той же виртуальной сети.
* **Принудительное туннелирование нельзя включить в VPN-шлюзе типа "сеть — сеть".** Весь интернет-трафик можно направить в локальную сеть через ExpressRoute только "принудительно".
* **Номера SKU класса "Базовый" для шлюза не поддерживаются.** Используйте для [ExpressRoute](expressroute-about-virtual-network-gateways.md) и [VPN-шлюза](../vpn-gateway/vpn-gateway-about-vpngateways.md) номера SKU другого класса.
* **Поддерживается только VPN-шлюз на основе маршрутов.** Необходимо использовать [VPN-шлюз](../vpn-gateway/vpn-gateway-about-vpngateways.md) на основе маршрутов.
* **Для VPN-шлюза необходимо настроить статический маршрут.** Если локальная сеть подключена и к ExpressRoute, и к VPN типа "сеть — сеть", необходимо использовать статический маршрут, настроенный в локальной сети для маршрутизации VPN-подключения типа "сеть — сеть" к Интернету.
* **Шлюз ExpressRoute необходимо настроить в первую очередь.** Сначала необходимо создать шлюз ExpressRoute, а затем — добавить VPN-шлюз типа "сеть — сеть".

## <a name="configuration-designs"></a>Схемы конфигурации
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>Настройка VPN типа "сеть-сеть" как пути отработки отказа для ExpressRoute
VPN-подключение типа "сеть-сеть" можно настроить как службу архивации для ExpressRoute. Это относится только к виртуальным сетям, привязанным к пути пиринга Azure Private. Для служб, доступ к которым осуществляется через пиринг Azure Public или Microsoft, решения для отработки отказа на основе VPN не существует. Канал ExpressRoute всегда является основной ссылкой. Данные передаются по пути VPN "сеть — сеть", только если канал ExpressRoute не справляется с этой задачей. 

> [!NOTE]
> Хотя канал ExpressRoute лучше использовать для VPN типа "сеть — сеть", когда оба маршрута одинаковы, Azure будет выбирать маршрут для назначения пакета по совпадению самого длинного префикса.
> 
> 

![Схема, на которой показано VPN-подключение типа "сеть — сеть" в качестве резервной копии ExpressRoute.](media/expressroute-howto-coexist-classic/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>Настройка VPN типа "сеть-сеть" для подключения к сайтам, не подключенным через ExpressRoute
Сети можно настроить таким образом, чтобы одни из них подключались непосредственно к Azure по VPN типа "сеть-сеть", а другие — через ExpressRoute. 

![Существуют одновременно](media/expressroute-howto-coexist-classic/scenario2.jpg)

> [!NOTE]
> Настроить виртуальную сеть как транзитный маршрутизатор нельзя.
> 
> 

## <a name="selecting-the-steps-to-use"></a>Выбор действий для использования
Существует два различных набора процедур для настройки сосуществующих подключений. Выбор процедуры настройки зависит от того, существует ли уже виртуальная сеть, к которой необходимо подключиться, или требуется создать новую виртуальную сеть.

* У меня нет виртуальной сети, и мне нужно ее создать.
  
    Если у вас еще нет виртуальной сети, то, выполнив эту процедуру, вы сможете создать новую виртуальную сеть с помощью классической модели развертывания и новые подключения ExpressRoute и VPN типа "сеть-сеть". Для настройки выполните действия из раздела [Создание новой виртуальной сети и параллельных подключений](#new).
* У меня уже есть виртуальная сеть с классической моделью развертывания.
  
    Возможно, у вас уже имеется виртуальная сеть, а также подключения к VPN типа "сеть-сеть" или к ExpressRoute. В разделе [Настройка параллельных подключений для существующей виртуальной сети](#add) этой статьи описано, как удалить шлюз, а затем создать новое подключение ExpressRoute и VPN-подключение "сеть — сеть". Обратите внимание, что при создании новых подключений, необходимо выполнять действия в строго определенном порядке. При создании шлюзов и подключений не пользуйтесь инструкциями, взятыми из других статей.
  
    В этой процедуре для создания одновременно существующих подключений потребуется удалить имеющийся шлюз, а затем настроить новые шлюзы. Это означает, что у вас будет перерыв в работе подключений между организациями, на время удаления и повторного создания шлюза и подключений, но вам не потребуется выполнять миграцию всех виртуальных машин и служб в новую виртуальную сеть. Виртуальные машины и службы смогут по-прежнему осуществлять связь через балансировщик нагрузки во время настройки шлюза, если они настроены соответствующим образом.

## <a name="install-powershell-cmdlets"></a>Установка командлетов PowerShell

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="to-create-a-new-virtual-network-and-coexisting-connections"></a><a name="new"></a>Создание новой виртуальной сети и параллельных подключений
В этой процедуре подробно рассматривается создание виртуальной сети, а также параллельных подключений ExpressRoute и "сеть-сеть".

1. Вам потребуется установить последнюю версию командлетов Azure PowerShell. Дополнительные сведения об установке командлетов PowerShell см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/). Обратите внимание, что командлеты, которые будут использоваться для этой конфигурации, могут немного отличаться от знакомых вам. Обязательно используйте командлеты, указанные в инструкциях. 
2. Создайте схему для виртуальной сети. Дополнительные сведения о схеме конфигурации см. в статье [Azure Virtual Network configuration schema](/previous-versions/azure/reference/jj157100(v=azure.100)) (Схема конфигурации виртуальной сети Azure).
   
    При создании схемы, убедитесь, что используются следующие значения.
   
   * Подсеть шлюза для виртуальной сети должна иметь значение /27 или более короткий префикс (например, /26 или /25).
   * Тип подключения шлюза — "выделенное".
     
    ```xml
    <VirtualNetworkSite name="MyAzureVNET" Location="Central US">
      <AddressSpace>
        <AddressPrefix>10.17.159.192/26</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="Subnet-1">
          <AddressPrefix>10.17.159.192/27</AddressPrefix>
        </Subnet>
        <Subnet name="GatewaySubnet">
          <AddressPrefix>10.17.159.224/27</AddressPrefix>
          /Subnet>
      </Subnets>
      <Gateway>
        <ConnectionsToLocalNetwork>
          <LocalNetworkSiteRef name="MyLocalNetwork">
            <Connection type="Dedicated" />
          </LocalNetworkSiteRef>
        </ConnectionsToLocalNetwork>
      </Gateway>
    </VirtualNetworkSite>
    ```
3. После создания и настройки XML-файла схемы отправьте этот файл. Это будет созданием виртуальной сети.
   
    Чтобы отправить файл с измененными значениями, используйте следующий командлет.
   
    ```azurepowershell
    Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'
    ```
4. <a name="gw"></a>Создайте шлюз ExpressRoute. Не забудьте указать для номера SKU шлюза значение *Standard*, *HighPerformance* или *UltraPerformance*, а для типа шлюза — значение *DynamicRouting*.
   
    Используйте следующий пример, подставив собственные значения.

    ```azurepowershell
    New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance
    ```
5. Свяжите шлюз ExpressRoute с каналом ExpressRoute. После выполнения этого действия подключение локальной сети к Azure через ExpressRoute будет установлено.
   
    ```azurepowershell
    New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET
    ```
6. <a name="vpngw"></a>Далее создайте VPN-шлюз типа "сеть — сеть". Номер SKU шлюза должен иметь значение *Standard*, *HighPerformance* или *UltraPerformance*, а тип шлюза должен быть *DynamicRouting*.
   
    ```azurepowershell
    New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance
    ```
   
    Чтобы получить параметры шлюза виртуальной сети, включая идентификатор шлюза и общедоступный IP-адрес, используйте командлет `Get-AzureVirtualNetworkGateway`.
   
    ```azurepowershell
    Get-AzureVirtualNetworkGateway
   
    GatewayId            : 348ae011-ffa9-4add-b530-7cb30010565e
    GatewayName          : S2SVPN
    LastEventData        :
    GatewayType          : DynamicRouting
    LastEventTimeStamp   : 5/29/2015 4:41:41 PM
    LastEventMessage     : Successfully created a gateway for the following virtual network: GNSDesMoines
    LastEventID          : 23002
    State                : Provisioned
    VIPAddress           : 104.43.x.y
    DefaultSite          :
    GatewaySKU           : HighPerformance
    Location             :
    VnetId               : 979aabcf-e47f-4136-ab9b-b4780c1e1bd5
    SubnetId             :
    EnableBgp            : False
    OperationDescription : Get-AzureVirtualNetworkGateway
    OperationId          : 42773656-85e1-a6b6-8705-35473f1e6f6a
    OperationStatus      : Succeeded
    ```
7. Создайте сущность VPN-шлюза локального сайта. Эта команда не настраивает локальный VPN-шлюз. Она только позволяет указать параметры локального шлюза, такие как общедоступный IP-адрес и локальное адресное пространство, чтобы VPN-шлюз Azure мог подключиться к нему.
   
   > [!IMPORTANT]
   > Локальный сайт для подключения VPN типа "сеть-сеть" не определяется в netcfg. Этот командлет можно использовать только для указания параметров локального сайта. Его нельзя определить с помощью портала или файла netcfg.
   > 
   > 
   
    Используйте следующий пример, подставив собственные значения.
   
    ```azurepowershell
    New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <MyLocalGatewayIp> -AddressSpace <MyLocalNetworkAddress>
    ```
   
   > [!NOTE]
   > Если локальная сеть имеет несколько маршрутов, их все можно передать как массив.  $MyLocalNetworkAddress = @("10.1.2.0/24","10.1.3.0/24","10.2.1.0/24")  
   > 
   > 

    Чтобы получить параметры шлюза виртуальной сети, включая идентификатор шлюза и общедоступный IP-адрес, используйте командлет `Get-AzureVirtualNetworkGateway`. См. следующий пример.

    ```azurepowershell
    Get-AzureLocalNetworkGateway

    GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
    GatewayName          : MyLocalNetwork
    IpAddress            : 23.39.x.y
    AddressSpace         : {10.1.2.0/24}
    OperationDescription : Get-AzureLocalNetworkGateway
    OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
    OperationStatus      : Succeeded
    ```


1. Настройте локальное VPN-устройство для подключения к новому шлюзу. При настройке VPN-устройства используйте сведения, полученные на этапе 6. Дополнительные сведения о настройке VPN-устройства см. в статье [О VPN-устройствах для подключений VPN-шлюзов типа "сеть — сеть"](../vpn-gateway/vpn-gateway-about-vpn-devices.md).
2. Свяжите VPN-шлюз к сети типа "сеть-сеть" в Azure с локальным шлюзом.
   
    В этом примере connectedEntityId — идентификатор локального шлюза, который можно узнать, выполнив `Get-AzureLocalNetworkGateway`. virtualNetworkGatewayId можно узнать с помощью командлета `Get-AzureVirtualNetworkGateway` . После выполнения этого действия подключение локальной сети к Azure через сеть VPN типа "сеть-сеть" будет установлено.

    ```azurepowershell
    New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>
    ```

## <a name="to-configure-coexisting-connections-for-an-already-existing-vnet"></a><a name="add"></a>Настройка параллельных подключений для существующей виртуальной сети
При наличии существующей виртуальной сети проверьте размер подсети шлюза. Если размер подсети шлюза — /28 или /29, необходимо сначала удалить шлюз виртуальной сети и увеличить размер подсети шлюза. В этом разделе показано, как это сделать.

Если размер подсети шлюза — /27 или больше, а виртуальная сеть подключается с помощью ExpressRoute, можно пропустить описанные ниже шаги и перейти к шагу 6 ( [Создание VPN-шлюза подключения типа "сеть — сеть"](#vpngw) ) в предыдущем разделе.

> [!NOTE]
> При удалении существующего шлюза локальные пользователи потеряют подключение к виртуальной сети на время выполнения этой настройки.
> 
> 

1. Вам потребуется установить последнюю версию командлетов PowerShell диспетчера ресурсов Azure. Дополнительные сведения об установке командлетов PowerShell см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/). Обратите внимание, что командлеты, которые будут использоваться для этой конфигурации, могут немного отличаться от знакомых вам. Обязательно используйте командлеты, указанные в инструкциях. 
2. Удалите для сети типа "сеть — сеть" существующий VPN-шлюз или шлюз ExpressRoute. Используйте командлет, приведенный ниже, подставив свои собственные значения.
   
    ```azurepowershell
    Remove-AzureVNetGateway –VnetName MyAzureVNET
    ```
3. Экспортируйте схему виртуальной сети. Используйте командлет PowerShell, приведенный ниже, подставив свои собственные значения.
   
    ```azurepowershell
    Get-AzureVNetConfig –ExportToFile "C:\NetworkConfig.xml"
    ```
4. Измените схему файла конфигурации сети так, чтобы подсеть шлюза имела значение /27 или более короткий префикс (например, /26 или /25). См. следующий пример. 
   
   > [!NOTE]
   > Если в виртуальной сети недостаточно свободных IP-адресов для увеличения размера подсети шлюза, необходимо добавить дополнительные пространства IP-адресов. Дополнительные сведения о схеме конфигурации см. в статье [Azure Virtual Network configuration schema](/previous-versions/azure/reference/jj157100(v=azure.100)) (Схема конфигурации виртуальной сети Azure).
   > 
   > 
   
    ```xml
    <Subnet name="GatewaySubnet">
      <AddressPrefix>10.17.159.224/27</AddressPrefix>
    </Subnet>
    ```
5. Если ранее шлюз был к сети VPN типа «сеть-сеть», необходимо также изменить тип подключения на **выделенный**.
   
    ```xml
    <Gateway>
      <ConnectionsToLocalNetwork>
        <LocalNetworkSiteRef name="MyLocalNetwork">
          <Connection type="Dedicated" />
        </LocalNetworkSiteRef>
      </ConnectionsToLocalNetwork>
    </Gateway>
    ```
6. На этом этапе вы будете иметь виртуальную сеть без шлюзов. Чтобы создать новые шлюзы и завершить подключение, перейдите к шагу 4 ( [Создайте шлюз ExpressRoute](#gw)) из предыдущего раздела.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об ExpressRoute см. в статье [вопросы и ответы по expressroute](expressroute-faqs.md) .