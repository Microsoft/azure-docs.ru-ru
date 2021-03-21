---
title: 'Подключение к виртуальной сети из компьютера — P2S VPN и собственная аутентификация Azure Certificate: PowerShell'
description: Безопасное подключение клиентов Windows и macOS к виртуальной сети Azure с помощью P2S, а также самозаверяющих сертификатов или выданных ЦС. В этой статье используется PowerShell.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/29/2020
ms.author: cherylmc
ms.openlocfilehash: ed0a60c88c33af70b7d780d6c4735c5f8e65b35b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94660412"
---
# <a name="configure-a-point-to-site-vpn-connection-to-a-vnet-using-native-azure-certificate-authentication-powershell"></a>Настройка VPN-подключения типа "точка — сеть" к виртуальной сети с помощью собственной аутентификации Azure на основе сертификата: PowerShell

Эта статья поможет вам безопасно подключать отдельные клиенты под управлением Windows, Linux или macOS к виртуальной сети Azure. VPN-подключения типа "точка — сеть" удобно использовать, если требуется подключиться к виртуальной сети из удаленного расположения, например при удаленном подключении из дома или конференции. Вы также можете использовать подключение "точка — сеть" вместо VPN-подключения "сеть — сеть" при наличии небольшого количества клиентов, которым требуется подключение к виртуальной сети. Для подключений типа "точка — сеть" не требуется VPN-устройство или общедоступный IP-адрес. Подключение "точка — сеть" — это VPN-подключение по протоколу SSTP (Secure Socket Tunneling Protocol) или IKEv2.

:::image type="content" source="./media/vpn-gateway-how-to-point-to-site-rm-ps/point-to-site-diagram.png" alt-text="Подключение с компьютера к схеме подключения &quot;точка — сеть&quot; в Azure":::

Дополнительные сведения о VPN-подключениях типа "точка — сеть" см. [в статье о VPN-подключениях типа "точка](point-to-site-about.md)— сеть". Чтобы создать эту конфигурацию с помощью портал Azure, см. раздел [Настройка VPN-подключения типа "точка — сеть" с помощью портал Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

[!INCLUDE [P2S basic architecture](../../includes/vpn-gateway-p2s-architecture.md)]

## <a name="prerequisites"></a>Предварительные условия

Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial).

### <a name="azure-powershell"></a>Azure PowerShell

>[!IMPORTANT]
> Многие действия, описанные в этой статье, могут использовать Azure Cloud Shell. Однако для создания сертификатов нельзя использовать Cloud Shell. Кроме того, чтобы передать открытый ключ корневого сертификата, необходимо либо использовать Azure PowerShell локально, либо портал Azure.
>

[!INCLUDE [PowerShell](../../includes/vpn-gateway-cloud-shell-powershell-about.md)]

## <a name="1-sign-in"></a><a name="signin"></a>1. Вход

[!INCLUDE [sign in](../../includes/vpn-gateway-cloud-shell-ps-login.md)]

## <a name="2-declare-variables"></a><a name="declare"></a>2. Объявление переменных

Мы используем переменные для этой статьи, чтобы вы могли легко изменять значения для применения в собственной среде, не изменяя сами примеры. Объявите переменные, которые вы хотите использовать. Вы можете использовать следующий пример, заменяя собственные значения при необходимости. Если необходимо закрыть сеанс PowerShell / Cloud Shell во время выполнения упражнения, просто скопируйте и вставьте значения в повторно объявленные переменные.

```azurepowershell-interactive
$VNetName  = "VNet1"
$FESubName = "FrontEnd"
$GWSubName = "GatewaySubnet"
$VNetPrefix = "10.1.0.0/16"
$FESubPrefix = "10.1.0.0/24"
$GWSubPrefix = "10.1.255.0/27"
$VPNClientAddressPool = "172.16.201.0/24"
$RG = "TestRG1"
$Location = "EastUS"
$GWName = "VNet1GW"
$GWIPName = "VNet1GWpip"
$GWIPconfName = "gwipconf"
$DNS = "10.2.1.4"
```

## <a name="3-configure-a-vnet"></a><a name="ConfigureVNet"></a>3. Настройка виртуальной сети

1. Создайте группу ресурсов.

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $RG -Location $Location
   ```

1. Создайте конфигурации подсети для виртуальной сети, присвоив им имя *внешнего* интерфейса и *GatewaySubnet*. Эти префиксы должны быть частью объявленного адресного пространства виртуальной сети.

   ```azurepowershell-interactive
   $fesub = New-AzVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
   $gwsub = New-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
   ```

1. Создание виртуальной сети.

   В этом примере параметр сервера -DnsServer является необязательным. Если указать значение, DNS-сервер не создается. Необходимо указать IP-адрес DNS-сервера, который может разрешать имена для ресурсов, к которым вы подключаетесь из своей виртуальной сети. В этом примере мы использовали частный IP-адрес, но это не IP-адрес DNS-сервера. Подставьте собственные значения. Указанное значение используется для ресурсов, развертываемых в виртуальной сети, а не для подключений "точка — сеть" или VPN-клиентов.

   ```azurepowershell-interactive
       New-AzVirtualNetwork `
      -ResourceGroupName $RG `
      -Location $Location `
      -Name $VNetName `
      -AddressPrefix $VNetPrefix `
      -Subnet $fesub, $gwsub `
      -DnsServer $DNS
   ```

1. Укажите переменные для созданной виртуальной сети.

   ```azurepowershell-interactive
   $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
   $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
   ```

1. VPN-шлюз должен иметь общедоступный IP-адрес. Сначала запросите ресурс IP-адреса, а затем укажите его при создании шлюза виртуальной сети. IP-адрес динамически назначается ресурсу при создании VPN-шлюза. В настоящее время VPN-шлюз поддерживает выделение *динамических* общедоступных IP-адресов. Вы не можете запросить назначение статического общедоступного IP-адреса. Однако это не означает, что IP-адрес изменяется после назначения VPN-шлюзу. Общедоступный IP-адрес изменяется только после удаления и повторного создания шлюза. При изменении размера, сбросе или других внутренних операциях обслуживания или обновления IP-адрес VPN-шлюза не изменяется.

   Запросите динамически назначенный общедоступный IP-адрес.

   ```azurepowershell-interactive
   $pip = New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
   $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
   ```

## <a name="4-create-the-vpn-gateway"></a><a name="creategateway"></a>4. Создание VPN-шлюза

На этом шаге вы настраиваете и создаете шлюз виртуальной сети для своей сети.

* У параметра -GatewayType должно быть значение **Vpn**, а у параметра -VpnType — **RouteBased**.
* Параметр -VpnClientProtocol используется, чтобы указать типы туннелей, которые необходимо включить. Параметры туннеля: **опенвпн, SSTP** и **IKEv2**. Вы можете включить одну из них или любую поддерживаемую комбинацию. Если требуется включить несколько типов, укажите имена, разделенные запятыми. Опенвпн и SSTP нельзя включить одновременно. Для подключения клиент strongSwan в Android и Linux и собственный VPN-клиент IKEv2 в iOS и OSX используют только туннель IKEv2. Клиенты Windows сначала пытаются подключиться через IKEv2, и если им это не удается, возвращаются к SSTP. Клиент Опенвпн можно использовать для подключения к типу туннеля Опенвпн.
* SKU "базовый" шлюза виртуальной сети не поддерживает проверку подлинности IKEv2, Опенвпн или RADIUS. Если вы планируете подключение к своей виртуальной сети клиентов Mac, не используйте SKU "Базовый".
* Создание VPN-шлюза может занять до 45 минут в зависимости от выбранного [номера SKU шлюза](vpn-gateway-about-vpn-gateway-settings.md). В этом примере используется IKEv2.

1. Настройте и создайте шлюз для своей виртуальной сети. Создание шлюза занимает примерно 45 минут.

   ```azurepowershell-interactive
   New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
   -Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
   -VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
   ```

1. После создания шлюза его можно просмотреть, используя следующий пример. Если вы закрыли PowerShell или истекло время ожидания при создании шлюза, вы можете [объявить переменные](#declare) еще раз.

   ```azurepowershell-interactive
   Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroup $RG
   ```

## <a name="5-add-the-vpn-client-address-pool"></a><a name="addresspool"></a>5. Добавление пула адресов VPN-клиента

По завершении создания сетевого VPN-шлюза можно добавить пул адресов клиента VPN. Пул адресов VPN-клиента представляет собой диапазон, из которого VPN-клиенты будут получать IP-адреса при подключении. Используйте диапазон частных IP-адресов, который не пересекается с локальным расположением, из которого выполнено подключение, или с виртуальной сетью, к которой вы хотите подключиться.

В этом примере пул адресов VPN-клиента объявляется как [переменная](#declare) на предыдущем шаге.

```azurepowershell-interactive
$Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
```

## <a name="6-generate-certificates"></a><a name="Certificates"></a>6. Создание сертификатов

>[!IMPORTANT]
> Вы не можете создавать сертификаты с помощью Azure Cloud Shell. Необходимо использовать один из методов, описанных в этом разделе. Если вы хотите использовать PowerShell, необходимо установить его локально.
>

Сертификаты используются в Azure для проверки подлинности VPN-клиентов для виртуальных частных сетей типа "точка — сеть". Необходимо отправить сведения об открытом ключе корневого сертификата в Azure. После этого открытый ключ считается доверенным. Сертификаты клиентов должны создаваться из доверенного корневого сертификата, а затем устанавливаться на каждом клиентском компьютере в хранилище сертификатов Certificates-Current User/Personal. Сертификат используется для проверки подлинности клиента, когда он инициирует подключение к виртуальной сети. 

Используемые самозаверяющие сертификаты должны быть созданы с помощью определенных параметров. Чтобы создать самозаверяющий сертификат, см. инструкции по [PowerShell и Windows 10](vpn-gateway-certificates-point-to-site.md). Если у вас нет Windows 10, вы можете использовать [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Во время создания самозаверяющих корневых сертификатов и сертификатов клиента обязательно следуйте инструкциям. В противном случае создаваемые сертификаты не будут совместимы с подключениями "точка — сеть", и вы получите сообщение об ошибке подключения.

### <a name="root-certificate"></a><a name="cer"></a>Корневой сертификат

1. [!INCLUDE [Root certificate](../../includes/vpn-gateway-p2s-rootcert-include.md)]

1. После создания корневого сертификата [экспортируйте](vpn-gateway-certificates-point-to-site.md#cer) данные общедоступного сертификата (не закрытый ключ) в виде файла X. 509. cer в кодировке Base64.

### <a name="client-certificate"></a><a name="generate"></a>Сертификат клиента

1. [!INCLUDE [Generate a client certificate](../../includes/vpn-gateway-p2s-clientcert-include.md)]

1. После создания сертификата клиента [экспортируйте](vpn-gateway-certificates-point-to-site.md#clientexport) его. Сертификат клиента будет распространяться на клиентские компьютеры, которые будут подключаться.

## <a name="7-upload-the-root-certificate-public-key-information"></a><a name="upload"></a>7. Отправка сведений о открытом ключе корневого сертификата

Убедитесь, что создание VPN-шлюза завершено. После создания шлюза вы можете передать CER-файл (который содержит сведения об открытом ключе) для доверенного корневого сертификата в Azure. После отправки CER-файла Azure сможет использовать его для проверки подлинности клиентов, на которых установлен клиентский сертификат, созданный из доверенного корневого сертификата. При необходимости позже можно отправить дополнительные файлы доверенных корневых сертификатов (не более 20).

>[!NOTE]
> Невозможно передать CER файл с помощью Azure Cloud Shell. Можно либо использовать PowerShell локально на компьютере, либо выполнить [портал Azure действия](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile).
>

1. Объявите переменную для имени сертификата, заменив существующее значение собственным.

   ```azurepowershell
   $P2SRootCertName = "P2SRootCert.cer"
   ```

1. Добавьте собственный путь к файлу, а затем выполните командлеты.

   ```azurepowershell
   $filePathForCert = "C:\cert\P2SRootCert.cer"
   $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
   $CertBase64 = [system.convert]::ToBase64String($cert.RawData)
   ```

1. Отправьте сведения об открытом ключе в Azure. После отправки сведений о сертификате Azure рассматривает его как доверенный корневой сертификат. При отправке убедитесь, что вы используете PowerShell локально на компьютере или вместо этого можете использовать [портал Azure действия](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile). Невозможно отправить с помощью Azure Cloud Shell.

   ```azurepowershell
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64
   ```

## <a name="8-install-an-exported-client-certificate"></a><a name="clientcertificate"></a>8. Установка экспортированного сертификата клиента

Следующие шаги помогут установить в клиенте Windows. Дополнительные сведения о клиентах см. [в разделе Установка сертификата клиента](point-to-site-how-to-vpn-client-install-azure-cert.md).

[!INCLUDE [Install on Windows](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

Убедитесь, что сертификат клиента был экспортирован как PFX-файл вместе со всей цепочкой сертификатов (это значение по умолчанию). В противном случае данные корневого сертификата будут отсутствовать на клиентском компьютере и клиент не сможет пройти проверку должным образом.

## <a name="9-configure-the-vpn-client"></a><a name="clientconfig"></a>9. Настройка VPN-клиента

В этом разделе вы настроите собственный клиент для компьютера, чтобы подключиться к шлюзу виртуальной сети. Например, при переходе к параметрам VPN на компьютере Windows можно добавить VPN-подключения. Для подключения типа "точка — сеть" требуются определенные параметры конфигурации. Эти действия помогут вам создать пакет с конкретными параметрами, которым должен быть подключен собственный VPN-клиент для подключения к виртуальной сети через подключение типа "точка — сеть".

Для создания и установки пакета конфигурации клиента можно использовать следующие краткие примеры. Дополнительные сведения о содержимом пакета и дополнительные инструкции по созданию и установке файлов конфигурации VPN-клиента см. в разделе [Создание и установка файлов конфигурации VPN-](point-to-site-vpn-client-configuration-azure-cert.md)клиента.

Если вам нужно еще раз объявить переменные, их можно найти [здесь](#declare).

### <a name="to-generate-configuration-files"></a>Создание файлов конфигурации

```azurepowershell-interactive
$profile=New-AzVpnClientConfiguration -ResourceGroupName $RG -Name $GWName -AuthenticationMethod "EapTls"

$profile.VPNProfileSASUrl
```

### <a name="to-install-the-client-configuration-package"></a>Установка пакета конфигурации клиента

[!INCLUDE [Windows instructions](../../includes/vpn-gateway-p2s-client-configuration-windows.md)]

## <a name="10-connect-to-azure"></a><a name="connect"></a>10. подключение к Azure

### <a name="windows-vpn-client"></a>VPN-клиент Windows

[!INCLUDE [Connect from Windows client](../../includes/vpn-gateway-p2s-connect-windows-client.md)]

[!INCLUDE [Client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

### <a name="mac-vpn-client"></a>VPN-клиент для Mac

В диалоговом окне Network (Сеть) найдите необходимый клиентский профиль и нажмите кнопку **Connect** (Подключиться).
Ознакомьтесь с подробными инструкциями в разделе [Установка Mac (OS X)](./point-to-site-vpn-client-configuration-azure-cert.md#installmac). Если у вас возникают неполадки подключения, убедитесь, что шлюз виртуальной сети не использует SKU "Базовый". SKU "Базовый" не поддерживается для клиентов Mac.

  ![Подключение для Mac](./media/vpn-gateway-howto-point-to-site-rm-ps/applyconnect.png)

## <a name="to-verify-a-connection"></a><a name="verify"></a>Проверка подключения

Эти инструкции применимы к клиентам Windows.

1. Чтобы проверить, активно ли VPN-подключение, откройте окно командной строки от имени администратора и выполните команду *ipconfig/all*.
2. Просмотрите результаты. Обратите внимание, что полученный IP-адрес является одним из адресов в пуле адресов VPN-клиента типа "точка — сеть", указанном в конфигурации. Вы должны увидеть результат, аналогичный приведенному ниже.

   ```
   PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.13(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
   ```

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Подключение к виртуальной машине

Эти инструкции применимы к клиентам Windows.

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm.md)]

* Убедитесь, что пакет конфигурации VPN-клиента был создан после IP-адресов DNS-сервера, заданных для виртуальной сети. Если вы обновили IP-адреса DNS-сервера, создайте и установите новый пакет конфигурации VPN-клиента.

* Используйте ipconfig, чтобы проверить IPv4-адрес, назначенный Ethernet-адаптеру на компьютере, с которого выполняется подключение. Если IP-адрес находится в диапазоне адресов виртуальной сети, к которой выполняется подключение, или в диапазоне адресов VPNClientAddressPool, адресное пространство перекрывается. В таком случае сетевой трафик не достигает Azure и остается в локальной сети.

## <a name="to-add-or-remove-a-root-certificate"></a><a name="addremovecert"></a>Добавление и удаление корневого сертификата

Вы можете добавлять доверенные корневые сертификаты в Azure, а также удалять их из Azure. При удалении корневого сертификата клиенты, использующие цифровой сертификат, созданный из этого корневого сертификата, не смогут пройти проверку подлинности и поэтому не смогут подключиться. Чтобы клиенты могли проходить аутентификацию и подключаться, необходимо установить новый сертификат клиента, созданный на основе корневого сертификата, который является доверенным для Azure (то есть он передан в Azure). Для выполнения этих действий требуются Azure PowerShell командлеты, установленные локально на компьютере (не Azure Cloud Shell). Можно также использовать портал Azure для добавления корневых сертификатов.

**Чтобы добавить:**

В Azure можно добавить до 20 CER-файлов корневых сертификатов. Следующие шаги помогут добавить корневой сертификат. 

1. Подготовьте CER-файл для отправки:

   ```azurepowershell
   $filePathForCert = "C:\cert\P2SRootCert3.cer"
   $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
   $CertBase64_3 = [system.convert]::ToBase64String($cert.RawData)
   ```

1. Отправьте файл. Вы можете отправить только один файл за раз.

   ```azurepowershell
   Add-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG1" -PublicCertData $CertBase64_3
   ```

1. Убедитесь, что файл сертификата загружен:

   ```azurepowershell
   Get-AzVpnClientRootCertificate -ResourceGroupName "TestRG1" `
   -VirtualNetworkGatewayName "VNet1GW"
   ```

**Для удаления:**

1. Объявите переменные. Измените переменные в примере, чтобы они совпадали с сертификатом, который требуется удалить.

   ```azurepowershell-interactive
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   $P2SRootCertName2 = "ARMP2SRootCert2.cer"
   $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
   ```

1. Удалите сертификат.

   ```azurepowershell-interactive
   Remove-AzVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
   ```

1. Используйте приведенный ниже пример, чтобы убедиться, что сертификат успешно удален.

   ```azurepowershell-interactive
   Get-AzVpnClientRootCertificate -ResourceGroupName "TestRG1" `
   -VirtualNetworkGatewayName "VNet1GW"
   ```

## <a name="to-revoke-or-reinstate-a-client-certificate"></a><a name="revoke"></a>Отзыв или восстановление сертификата клиента

Можно отозвать сертификаты клиента. Список отзыва сертификатов позволяет выборочно запретить подключение "точка — сеть" на основе отдельных сертификатов клиента. Эта процедура отличается от удаления доверенного корневого сертификата. При удалении доверенного корневого сертификата (CER-файл) из Azure будет запрещен доступ для всех сертификатов клиента, созданных на основе отозванного корневого сертификата или подписанных им. Отзыв сертификата клиента, а не корневого сертификата, позволяет по-прежнему использовать другие сертификаты, созданные на основе корневого сертификата, для проверки подлинности.

Обычно корневой сертификат используется для управления доступом на уровнях группы или организации, а отозванный сертификат клиента — для точного контроля доступа для отдельных пользователей.

**Для отмены:**

1. Получите отпечаток сертификата клиента. Дополнительные сведения см. в разделе [Получение отпечатка сертификата](/dotnet/framework/wcf/feature-details/how-to-retrieve-the-thumbprint-of-a-certificate).

1. Скопируйте данные в текстовый редактор и удалите все пробелы, чтобы предоставить отпечаток в виде непрерывной строки. Далее эта строка будет объявлена в качестве переменной.

1. Объявите переменные. Обязательно объявите отпечаток, полученный на предыдущем шаге.

   ```azurepowershell-interactive
   $RevokedClientCert1 = "NameofCertificate"
   $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   ```

1. Добавьте отпечаток в список отозванных сертификатов. После добавления отпечатка отобразится Succeeded.

   ```azurepowershell-interactive
   Add-AzVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
   -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG `
   -Thumbprint $RevokedThumbprint1
   ```

1. Убедитесь, что отпечаток добавлен в список отзыва сертификатов.

   ```azurepowershell-interactive
   Get-AzVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
   ```

1. Теперь сертификат нельзя использовать для подключения. Клиенты, пытающиеся подключиться с помощью этого сертификата, получат сообщение, что он недействителен.

**Чтобы возобновить:**

Можно возобновить использование сертификата клиента, удалив отпечаток из списка отозванных сертификатов клиента.

1. Объявите переменные. Обязательно объявите правильный отпечаток сертификата, который требуется возобновить.

   ```azurepowershell-interactive
   $RevokedClientCert1 = "NameofCertificate"
   $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
   $GWName = "Name_of_virtual_network_gateway"
   $RG = "Name_of_resource_group"
   ```

1. Удалите отпечаток сертификата из списка отзыва сертификатов.

   ```azurepowershell-interactive
   Remove-AzVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
   -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1
   ```

1. Проверьте, удален ли отпечаток из списка отозванных отпечатков.

   ```azurepowershell-interactive
   Get-AzVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
   ```

## <a name="point-to-site-faq"></a><a name="faq"></a>Часто задаваемые вопросы о подключениях типа "точка — сеть"

Дополнительные сведения о возможностях подключения "точка — сеть" см. в статье [часто задаваемые вопросы о VPN-шлюзе](vpn-gateway-vpn-faq.md#P2S) .

## <a name="next-steps"></a>Дальнейшие действия

Установив подключение, можно добавить виртуальные машины в виртуальные сети. Дополнительные сведения см. в статье [виртуальные машины](../index.yml). Дополнительные сведения о сетях и виртуальных машинах см. в статье [Azure и Linux: обзор сетей виртуальных машин](../virtual-machines/network-overview.md).

Дополнительные сведения об устранении неполадок подключения "точка — сеть" см. в руководстве [Устранение неполадок подключения типа "точка — сеть" Azure](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).