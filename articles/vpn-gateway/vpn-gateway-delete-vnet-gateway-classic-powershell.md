---
title: 'Удаление шлюза виртуальной сети: классический класс Azure'
description: Удаление шлюза виртуальной сети с помощью PowerShell в классической модели развертывания.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/08/2020
ms.author: cherylmc
ms.openlocfilehash: 39585a68c5cddc50cd04e82caca71209270f7b68
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91874124"
---
# <a name="delete-a-virtual-network-gateway-using-powershell-classic"></a>Удаление шлюза виртуальной сети с помощью PowerShell (классическая модель)

> [!div class="op_single_selector"]
> * [Resource Manager — портал Azure](vpn-gateway-delete-vnet-gateway-portal.md)
> * [Resource Manager — PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [Классическая модель: PowerShell](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>

В этой статье описано, как удалить VPN-шлюз в классической модели развертывания с помощью PowerShell. После удаления шлюза виртуальной сети измените файл конфигурации сети, чтобы удалить элементы, которые больше не используются.

## <a name="step-1-connect-to-azure"></a><a name="connect"></a>Шаг 1. Подключение к Azure

### <a name="1-install-the-latest-powershell-cmdlets"></a>1. Установите последние командлеты PowerShell.

[!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

### <a name="2-connect-to-your-azure-account"></a>2. Подключитесь к учетной записи Azure.

Откройте консоль PowerShell с повышенными правами и подключитесь к своей учетной записи. Для подключения используйте следующий пример кода:

1. Откройте консоль PowerShell с повышенными правами.
2. Подключитесь к учетной записи. Для подключения используйте следующий пример кода:

   ```powershell
   Add-AzureAccount
   ```

## <a name="step-2-export-and-view-the-network-configuration-file"></a><a name="export"></a>Шаг 2. Экспорт и просмотр файла конфигурации сети

Создайте каталог на компьютере, а затем экспортируйте в него файл конфигурации сети. С помощью этого файла можно просмотреть сведения о текущей конфигурации, а также изменить конфигурацию сети.

В этом примере файл конфигурации сети экспортируется в каталог C:\AzureNet.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

Откройте файл в текстовом редакторе и просмотрите имя для классической виртуальной сети. При создании виртуальной сети на портале Azure полное имя, которое используется Azure, не отображается на портале. Например, если виртуальной сеть отображается на портале Azure с именем ClassicVNet1, то в файле конфигурации сети она может иметь гораздо более длинное имя. Это имя может выглядеть примерно следующим образом: Group ClassicRG1 ClassicVNet1. Имена виртуальных сетей перечислены как **"VirtualNetworkSite Name ="**. При выполнении командлетов PowerShell используйте имена из файла конфигурации сети.

## <a name="step-3-delete-the-virtual-network-gateway"></a><a name="delete"></a>Шаг 3. Удаление шлюза виртуальной сети

При удалении шлюза виртуальной сети все подключения к виртуальной сети через шлюз будут отключены. Если к виртуальной сети подключены клиенты P2S, то они будут отключены без предупреждения.

В этом примере удаляется шлюз виртуальной сети. Обязательно используйте полное имя виртуальной сети из файла конфигурации сети.

```powershell
Remove-AzureVNetGateway -VNetName "Group ClassicRG1 ClassicVNet1"
```

При успешном выполнении возвращается следующий результат:

```
Status : Successful
```

## <a name="step-4-modify-the-network-configuration-file"></a><a name="modify"></a>Шаг 4. Изменение файла конфигурации сети

При удалении шлюза виртуальной сети командлет не изменяет файл конфигурации сети. Необходимо изменить файл, чтобы удалить элементы, которые больше не используются. В следующих разделах описывается, как изменить скачанный файл конфигурации сети.

### <a name="local-network-site-references"></a><a name="lnsref"></a>Ссылки на сайты локальной сети

Чтобы удалить данные ссылок на сайты, внесите изменения в элемент **ConnectionsToLocalNetwork/LocalNetworkSiteRef**. Удаление ссылки на локальный сайт Azure инициирует удаление туннеля. В зависимости от созданной конфигурации у вас может не быть **LocalNetworkSiteRef** в списке.

```
<Gateway>
   <ConnectionsToLocalNetwork>
     <LocalNetworkSiteRef name="D1BFC9CB_Site2">
       <Connection type="IPsec" />
     </LocalNetworkSiteRef>
   </ConnectionsToLocalNetwork>
 </Gateway>
```

Пример

```
<Gateway>
   <ConnectionsToLocalNetwork>
   </ConnectionsToLocalNetwork>
 </Gateway>
```

### <a name="local-network-sites"></a><a name="lns"></a>Сайты локальной сети

Удалите все локальные сайты, которые больше не используются. В зависимости от созданной конфигурации элемент **LocalNetworkSite** может отсутствовать.

```
<LocalNetworkSites>
  <LocalNetworkSite name="Site1">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
  </LocalNetworkSite>
  <LocalNetworkSite name="Site3">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>57.179.18.164</VPNGatewayAddress>
  </LocalNetworkSite>
 </LocalNetworkSites>
```

В этом примере мы удалили только Site3.

```
<LocalNetworkSites>
  <LocalNetworkSite name="Site1">
    <AddressSpace>
      <AddressPrefix>192.168.0.0/16</AddressPrefix>
    </AddressSpace>
    <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
  </LocalNetworkSite>
 </LocalNetworkSites>
```

### <a name="client-addresspool"></a><a name="clientaddresss"></a>Пул адресов клиента

Если вы P2S подключение к виртуальной сети, то у вас будет **VPNClientAddressPool**. Удалите пулы адресов клиента, которые соответствуют удаленному шлюзу виртуальной сети.

```
<Gateway>
    <VPNClientAddressPool>
      <AddressPrefix>10.1.0.0/24</AddressPrefix>
    </VPNClientAddressPool>
  <ConnectionsToLocalNetwork />
 </Gateway>
```

Пример

```
<Gateway>
  <ConnectionsToLocalNetwork />
 </Gateway>
```

### <a name="gatewaysubnet"></a><a name="gwsub"></a>GatewaySubnet

Удалите **GatewaySubnet** , соответствующий виртуальной сети.

```
<Subnets>
   <Subnet name="FrontEnd">
     <AddressPrefix>10.11.0.0/24</AddressPrefix>
   </Subnet>
   <Subnet name="GatewaySubnet">
     <AddressPrefix>10.11.1.0/29</AddressPrefix>
   </Subnet>
 </Subnets>
```

Пример

```
<Subnets>
   <Subnet name="FrontEnd">
     <AddressPrefix>10.11.0.0/24</AddressPrefix>
   </Subnet>
 </Subnets>
```

## <a name="step-5-upload-the-network-configuration-file"></a><a name="upload"></a>Шаг 5. Передача файла конфигурации сети

Сохраните изменения и передайте файл конфигурации сети в Azure. Обязательно измените путь к файлу в соответствии с вашей средой.

```powershell
Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml
```

При успешном выполнении возвращается примерно следующий результат:

```
OperationDescription        OperationId                      OperationStatus                                                
--------------------        -----------                      ---------------                                           
Set-AzureVNetConfig         e0ee6e66-9167-cfa7-a746-7casb9   Succeeded
```
