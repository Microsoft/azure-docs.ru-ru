---
title: Установка между компьютером и виртуальной сетью подключения типа "точка — сеть" с использованием аутентификации RADIUS и PowerShell | Документация Майкрософт
description: Безопасное подключение клиентов Windows и OS X к виртуальной сети с помощью P2S и аутентификации RADIUS.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/18/2020
ms.author: cherylmc
ms.openlocfilehash: 9d962d3a4757b4c7b2d217f91aaf73d6ad4164d3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94964853"
---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-radius-authentication-powershell"></a>Настройка подключения типа "точка — сеть" к виртуальной сети с использованием аутентификации RADIUS и PowerShell

В этой статье объясняется, как создать виртуальную сеть с подключением типа "точка — сеть", при котором выполняется аутентификация RADIUS. Эта конфигурация доступна только для ресурсов, развернутых в модели Resource Manager.

VPN-шлюз типа "точка — сеть" (P2S) позволяет создать безопасное подключение к виртуальной сети с отдельного клиентского компьютера. VPN-подключения типа "точка — сеть" удобны для подключения к виртуальной сети из удаленного расположения, например, если вы дома или на конференции. Такая конфигурация также эффективна для использования вместо VPN-подключения типа "сеть — сеть" при наличии небольшого количества клиентов, которым требуется подключение к виртуальной сети.

VPN-подключение "точка — сеть" устанавливается на устройствах Windows и Mac. Клиенты могут использовать один из следующих способов аутентификации при подключении: 

* сервер RADIUS;
* собственная аутентификация VPN-шлюза с помощью сертификата.
* Проверка подлинности машинного Azure Active Directory (только Windows 10)

Из этой статьи вы узнаете, как настроить конфигурацию "точка — сеть" с аутентификацией при помощи сервера RADIUS. Если вы хотите проверить подлинность с помощью созданных сертификатов и собственной проверки подлинности сертификата VPN-шлюза, см. статью [Настройка подключения типа "точка — сеть" к виртуальной сети с помощью собственного сертификата VPN-шлюза](vpn-gateway-howto-point-to-site-rm-ps.md) или [Создание клиента Azure Active Directory для подключений по протоколу P2S опенвпн](openvpn-azure-ad-tenant.md) для проверки подлинности Azure Active Directory.

![Схема, на которой показана конфигурация P2S с проверкой подлинности с помощью сервера RADIUS.](./media/point-to-site-how-to-radius-ps/p2sradius.png)

Для подключения типа "точка — сеть" не требуется VPN-устройство или общедоступный IP-адрес. P2S создает VPN-подключение по протоколу SSTP (Secure Socket Tunneling Protocol), Опенвпн или IKEv2.

* SSTP — это туннель VPN на основе TLS, который поддерживается только на клиентских платформах Windows. Этот туннель проходит через брандмауэры и является отличным вариантом для подключения к Azure из любого расположения. На стороне сервера поддерживается SSTP версии 1.0, 1.1 и 1.2. Какую версию использовать, решает клиент. Для Windows 8.1 и более поздних версий по умолчанию используется SSTP версии 1.2.

* Протокол Опенвпн®, протокол VPN на основе SSL/TLS. VPN-решение TLS может проникнуть брандмауэры, так как большинство брандмауэров открывают порт TCP 443 для исходящего трафика, используемый TLS. Опенвпн можно использовать для подключения с устройств Android, iOS (версии 11,0 и выше), Windows, Linux и Mac (OSX версии 10,13 и выше).

* IKEv2 VPN — решение VPN на основе стандартов IPsec. IKEv2 VPN можно использовать для подключения с устройств Mac (OSX версии 10.11 и выше).

Для подключений типа "точка — сеть" требуется следующее:

* VPN-шлюз с маршрутизацией на основе маршрутов. 
* Сервер RADIUS для аутентификации пользователей. Сервер RADIUS можно развернуть локально или в виртуальной сети Azure. Можно также настроить два сервера RADIUS для обеспечения высокой доступности.
* Пакет конфигурации VPN-клиента для устройств с Windows, которые будут подключаться к виртуальной сети. Пакет конфигурации VPN-клиента содержит параметры, необходимые для подключения VPN-клиента типа "точка — сеть".

## <a name="about-active-directory-ad-domain-authentication-for-p2s-vpns"></a><a name="aboutad"></a>Сведения об аутентификации домена Active Directory (AD) для VPN-подключений типа "точка — сеть".

Если используется аутентификация домена AD, пользователи могут входить в Azure с помощью учетных данных домена организации. Для этой проверки требуется сервер RADIUS, который интегрирован с сервером AD. Организации могут также использовать существующие серверы RADIUS.
 
Сервер RADIUS можно развернуть локально или в виртуальной сети Azure. Во время аутентификации VPN-шлюз выступает в качестве транзитного и переадресовывает сообщения аутентификации между сервером RADIUS и подключаемым устройством. Очень важно, чтобы VPN-шлюз мог связаться с сервером RADIUS. Если сервер RADIUS развернут локально, требуется VPN-подключение "сеть — сеть" между Azure и локальной сетью.

Сервер RADIUS можно интегрировать не только с Active Directory, но и с другими системами внешних идентификаторов. Благодаря этой возможности для VPN-подключения типа "точка — сеть" доступно множество вариантов аутентификации, в том числе варианты многофакторной проверки подлинности. Список систем идентификаторов для интеграции см. в документации поставщика сервера RADIUS.

![Схема подключения — RADIUS](./media/point-to-site-how-to-radius-ps/radiusimage.png)

> [!IMPORTANT]
>С локальным сервером RADIUS можно установить только VPN-соединение типа "сеть — сеть". Нельзя использовать подключение ExpressRoute.
>
>

## <a name="before-beginning"></a><a name="before"></a>Подготовка

Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial).

### <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

### <a name="example-values"></a><a name="example"></a>Примеры значений

Эти примеры значений можно использовать для создания тестовой среды или анализа примеров из этой стать. Вы можете использовать эти пошаговые инструкции, используя указанные в них значения, или же изменить значения в соответствии со своей средой.

* **Имя: VNet1**
* **Адресное пространство: 192.168.0.0/16** и **10.254.0.0/16**.<br>Чтобы продемонстрировать, что эта конфигурация будет работать с несколькими адресными пространствами, в этом примере мы используем несколько адресных пространств. Однако это необязательно для этой конфигурации.
* **Имя подсети: интерфейсная часть**
  * **Диапазон адресов подсети: 192.168.1.0/24**.
* **Имя подсети: BackEnd.**
  * **Диапазон адресов подсети: 10.254.1.0/24.**
* **Имя подсети: GatewaySubnet.**<br>Имя подсети *GatewaySubnet* обязательно для работы VPN-шлюза.
  * **Диапазон адресов подсети шлюза: 192.168.200.0/24.** 
* **Пул адресов VPN-клиента: 172.16.201.0/24.**<br>VPN-клиенты, подключающиеся к виртуальной сети с помощью этого подключения типа "точка — сеть", получают IP-адреса из пула адресов VPN-клиента.
* **Подписка**. Если у вас есть несколько подписок, убедитесь, что используется правильная.
* **Группа ресурсов: TestRG**
* **Расположение: Восточная часть США**
* **DNS-сервер: IP-адрес** DNS-сервера, который нужно использовать для разрешения имен в виртуальной сети. (необязательно).
* **Имя шлюза: Vnet1GW**.
* **Имя общедоступного IP-адреса: VNet1GWPIP**.
* **Тип VPN: RouteBased.**

## <a name="1-set-the-variables"></a><a name="signin"></a>1. Задание переменных

Объявите переменные, которые вы хотите использовать. Используйте следующий пример, подставив собственные значения в соответствующих параметрах. Если необходимо закрыть сеанс PowerShell / Cloud Shell во время выполнения упражнения, просто скопируйте и вставьте значения в повторно объявленные переменные.

  ```azurepowershell-interactive
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $BESubName = "Backend"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "192.168.0.0/16"
  $VNetPrefix2 = "10.254.0.0/16"
  $FESubPrefix = "192.168.1.0/24"
  $BESubPrefix = "10.254.1.0/24"
  $GWSubPrefix = "192.168.200.0/26"
  $VPNClientAddressPool = "172.16.201.0/24"
  $RG = "TestRG"
  $Location = "East US"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWPIP"
  $GWIPconfName = "gwipconf"
  ```

## <a name="2-create-the-resource-group-vnet-and-public-ip-address"></a>2. <a name="vnet"></a> Создайте группу ресурсов, виртуальную сеть и общедоступный IP-адрес.

Выполните инструкции ниже, чтобы создать группу ресурсов и виртуальную сеть в группе ресурсов с тремя подсетями. При замене значений важно, чтобы вы назвали подсеть шлюза именем GatewaySubnet. Если вы используете другое имя, создание шлюза завершится сбоем.

1. Создайте группу ресурсов.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name "TestRG" -Location "East US"
   ```
2. Создайте конфигурации подсети для виртуальной сети, присвоив им имена *FrontEnd*, *BackEnd* и *GatewaySubnet*. Эти префиксы должны быть частью объявленного адресного пространства виртуальной сети.

   ```azurepowershell-interactive
   $fesub = New-AzVirtualNetworkSubnetConfig -Name "FrontEnd" -AddressPrefix "192.168.1.0/24"  
   $besub = New-AzVirtualNetworkSubnetConfig -Name "Backend" -AddressPrefix "10.254.1.0/24"  
   $gwsub = New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "192.168.200.0/24"
   ```
3. Создание виртуальной сети.

   В этом примере параметр сервера -DnsServer является необязательным. Если указать значение, DNS-сервер не создается. Необходимо указать IP-адрес DNS-сервера, который может разрешать имена для ресурсов, к которым вы подключаетесь из своей виртуальной сети. В этом примере мы использовали частный IP-адрес, но, скорее всего, это не IP-адрес DNS-сервера. Подставьте собственные значения. Указанное значение используется для ресурсов, развертываемых в виртуальной сети, а не для подключений типа "точка — сеть".

   ```azurepowershell-interactive
   New-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG" -Location "East US" -AddressPrefix "192.168.0.0/16","10.254.0.0/16" -Subnet $fesub, $besub, $gwsub -DnsServer 10.2.1.3
   ```
4. VPN-шлюз должен иметь общедоступный IP-адрес. Сначала запросите ресурс IP-адреса, а затем укажите его при создании шлюза виртуальной сети. IP-адрес динамически назначается ресурсу при создании VPN-шлюза. В настоящее время VPN-шлюз поддерживает выделение *динамических* общедоступных IP-адресов. Вы не можете запросить назначение статического общедоступного IP-адреса. Однако это не означает, что IP-адрес изменяется после назначения VPN-шлюзу. Общедоступный IP-адрес изменяется только после удаления и повторного создания шлюза. При изменении размера, сбросе или других внутренних операциях обслуживания или обновления IP-адрес VPN-шлюза не изменяется.

   Укажите переменные, чтобы подать запрос на динамически назначенный общедоступный IP-адрес.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "TestRG"  
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet 
   $pip = New-AzPublicIpAddress -Name "VNet1GWPIP" -ResourceGroupName "TestRG" -Location "East US" -AllocationMethod Dynamic 
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name "gwipconf" -Subnet $subnet -PublicIpAddress $pip
   ```

## <a name="3-set-up-your-radius-server"></a>3. <a name="radius"></a> Настройка сервера RADIUS

Прежде чем создавать и настраивать шлюз виртуальной сети, необходимо правильно настроить сервер RADIUS для аутентификации.

1. Если вы еще не развернули сервер RADIUS, сделайте это. Инструкции по развертыванию см. в руководстве по установке, которое предоставлено поставщиком RADIUS.  
2. Настройте VPN-шлюз как клиент RADIUS на сервере RADIUS. При добавлении клиента RADIUS укажите подсеть шлюза виртуальной сети, которую вы создали. 
3. После настройки сервера RADIUS получите его IP-адрес и общий секрет, который клиенты RADIUS будут использовать для обмена данными с сервером RADIUS. Если сервер RADIUS находится в виртуальной сети Azure, используйте IP-адрес центра сертификации виртуальной машины сервера RADIUS.

Статья [Сервер политики сети (NPS)](/windows-server/networking/technologies/nps/nps-top) содержит сведения о настройке сервера RADIUS (NPS) под управлением Windows для аутентификации домена AD.

## <a name="4-create-the-vpn-gateway"></a>4. <a name="creategw"></a> Создание VPN-шлюза

Настройте и создайте VPN-шлюз для своей виртуальной сети.

* Для параметра -GatewayType должно быть установлено значение Vpn, а для параметра -VpnTyp — RouteBased.
* Для выполнения VPN-шлюза может потребоваться до 45 минут в зависимости от выбранного [номера SKU шлюза](vpn-gateway-about-vpn-gateway-settings.md#gwsku)   .

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1
```

## <a name="5-add-the-radius-server-and-client-address-pool"></a>5. <a name="addradius"></a> Добавление RADIUS-сервера и пула адресов клиента
 
* -RadiusServer можно указать с помощью имени или IP-адреса. Если вы укажете имя и сервер находится в локальной среде, VPN-шлюз может не разрешить имя. В таком случае рекомендуем указать IP-адрес сервера. 
* Параметр -RadiusSecret должен соответствовать настроенному параметру на сервере RADIUS.
* -VpnClientAddressPool — это диапазон, из которого VPN-клиенты будут получать IP-адреса при подключении. Используйте диапазон частных IP-адресов, который не пересекается с локальным расположением, из которого будет выполняться подключение, или с виртуальной сетью, к которой вы хотите подключиться. Настройте достаточное количество адресов в пуле.  

1. Создайте безопасную строку секрета RADIUS.

   ```azurepowershell-interactive
   $Secure_Secret=Read-Host -AsSecureString -Prompt "RadiusSecret"
   ```

2. Появится запрос на ввод секрета RADIUS. Символы, которые вы вводите, не будут отображаться. Вместо них вы увидите символы "*".

   ```azurepowershell-interactive
   RadiusSecret:***
   ```
3. Добавьте пул адресов VPN-клиента и сведения о сервере RADIUS.

   Для конфигураций SSTP:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "SSTP" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Для конфигураций® Опенвпн:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientRootCertificates @()
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "OpenVPN" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```


   Для конфигураций IKEv2:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol "IKEv2" `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Для конфигураций SSTP и IKEv2:

    ```azurepowershell-interactive
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway `
    -VpnClientAddressPool "172.16.201.0/24" -VpnClientProtocol @( "SSTP", "IkeV2" ) `
    -RadiusServerAddress "10.51.0.15" -RadiusServerSecret $Secure_Secret
    ```

   Чтобы указать **два** RADIUS-сервера, используйте следующий синтаксис. При необходимости измените значение параметра **-VpnClientProtocol**

    ```azurepowershell-interactive
    $radiusServer1 = New-AzRadiusServer -RadiusServerAddress 10.1.0.15 -RadiusServerSecret $radiuspd -RadiusServerScore 30
    $radiusServer2 = New-AzRadiusServer -RadiusServerAddress 10.1.0.16 -RadiusServerSecret $radiuspd -RadiusServerScore 1

    $radiusServers = @( $radiusServer1, $radiusServer2 )

    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $actual -VpnClientAddressPool 201.169.0.0/16 -VpnClientProtocol "IkeV2" -RadiusServerList $radiusServers
    ```

## <a name="6-download-the-vpn-client-configuration-package-and-set-up-the-vpn-client"></a>6. <a name="vpnclient"></a> скачайте пакет конфигурации VPN-клиента и настройте VPN-клиент.

Конфигурация VPN-клиента позволяет устройствам устанавливать с виртуальной сетью соединение типа "точка — сеть". Чтобы создать пакет конфигурации VPN-клиента и настроить VPN-клиент, см. инструкции в статье о [создании файла конфигурации VPN-клиента для аутентификации RADIUS](point-to-site-vpn-client-configuration-radius.md).

## <a name="7-connect-to-azure"></a><a name="connect"></a>7. подключение к Azure

### <a name="to-connect-from-a-windows-vpn-client"></a>Подключение из VPN-клиента для Windows

1. Чтобы подключиться к виртуальной сети, откройте VPN-подключения на клиентском компьютере и найдите созданное VPN-подключение. Его имя совпадает с названием вашей виртуальной сети. Введите учетные данные своего домена и нажмите кнопку "Подключиться". Появится всплывающее сообщение с запросом на повышенные права. Примите запрос и введите учетные данные.

   ![Подключение VPN-клиента к Azure](./media/point-to-site-how-to-radius-ps/client.png)
2. Теперь подключение установлено.

   ![Подключение установлено](./media/point-to-site-how-to-radius-ps/connected.png)

### <a name="connect-from-a-mac-vpn-client"></a>Подключение из VPN-клиента для Mac

В диалоговом окне Network (Сеть) найдите необходимый клиентский профиль и нажмите кнопку **Connect** (Подключиться).

  ![Подключение для Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="to-verify-your-connection"></a><a name="verify"></a>Проверка подключения

1. Чтобы проверить, активно ли VPN-подключение, откройте окно командной строки от имени администратора и выполните команду *ipconfig/all*.
2. Просмотрите результаты. Обратите внимание, что полученный вами IP-адрес — это один из адресов в пуле адресов VPN-клиента подключения "точка–cеть", указанном в конфигурации. Вы должны увидеть результат, аналогичный приведенному ниже.

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

Устранение неполадок подключения типа "точка — сеть" описывается в разделе [Устранение неполадок подключения типа "точка — сеть" Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Подключение к виртуальной машине

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* Убедитесь, что пакет конфигурации VPN-клиента был создан после IP-адресов DNS-сервера, заданных для виртуальной сети. Если вы обновили IP-адреса DNS-сервера, создайте и установите новый пакет конфигурации VPN-клиента.

* Используйте ipconfig, чтобы проверить IPv4-адрес, назначенный Ethernet-адаптеру на компьютере, с которого выполняется подключение. Если IP-адрес находится в диапазоне адресов виртуальной сети, к которой выполняется подключение, или в диапазоне адресов VPNClientAddressPool, адресное пространство перекрывается. В таком случае сетевой трафик не достигает Azure и остается в локальной сети.

## <a name="faq"></a><a name="faq"></a>Часто задаваемые вопросы

Вопросы и ответы о подключении "точка — сеть", при котором выполняется аутентификация RADIUS

[!INCLUDE [Point-to-Site RADIUS FAQ](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Установив подключение, можно добавить виртуальные машины в виртуальные сети. Дополнительные сведения см. в статье [виртуальные машины](../index.yml). Дополнительные сведения о сетях и виртуальных машинах см. в статье [Azure и Linux: обзор сетей виртуальных машин](../virtual-machines/network-overview.md).
