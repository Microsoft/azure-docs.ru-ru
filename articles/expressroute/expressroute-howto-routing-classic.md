---
title: 'Azure ExpressRoute: Настройка пиринга: классическая'
description: В этой статье описана процедура создания и подготовки частного пиринга, общедоступного пиринга и пиринга Microsoft для канала ExpressRoute, а также показано, как проверить состояние, обновить или удалить пиринги для канала.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: duau
ms.openlocfilehash: a4a3bad1e868fa0e75611630ffb5db5ba13126b6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89395559"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit-classic"></a>Создание и изменение пиринга для канала ExpressRoute (классическая модель)
> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [Видео — частный пиринг](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Видео — общедоступный пиринг](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [Видео — пиринг Майкрософт](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-routing-classic.md)
> 

В этой статье описано, как созвать конфигурацию маршрутизации или пиринга для канала ExpressRoute и управлять ею, используя командлеты PowerShell и классическую модель развертывания. Ниже описывается, как проверить состояние, обновить или удалить и отозвать пиринги для канала ExpressRoute. Для каждого канала ExpressRoute можно настроить один, два или все три пиринга (частный пиринг Azure, общедоступный пиринг Azure и пиринг Microsoft). Пиринги можно настраивать в любом порядке, главное, выполнять их конфигурацию по очереди. 

Эти инструкции распространяются только на каналы от поставщиков, предоставляющих услуги подключения второго уровня. Если ваш поставщик услуг подключения предлагает услуги третьего уровня (обычно это IPVPN, например MPLS), то он возьмет на себя настройку маршрутизации и управление ею.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**О моделях развертывания Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="configuration-prerequisites"></a>Предварительные требования для настройки

* Прежде чем приступать к настройке, обязательно изучите [предварительные требования](expressroute-prerequisites.md), [требования к маршрутизации](expressroute-routing.md) и [рабочие процессы](expressroute-workflows.md).
* Вам потребуется активный канал ExpressRoute. Прежде чем продолжить, следуйте инструкциям по [созданию канала ExpressRoute](expressroute-howto-circuit-classic.md) и каналу, включенному поставщиком услуг подключения. Для выполнения описанных ниже командлетов канал ExpressRoute должен быть подготовлен и включен.

### <a name="download-the-latest-powershell-cmdlets"></a>Скачивание последних версий командлетов PowerShell

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="azure-private-peering"></a>Частный пиринг Azure

В этом разделе описано, как создать, получить, обновить и удалить конфигурацию частного пиринга Azure для канала ExpressRoute. 

### <a name="to-create-azure-private-peering"></a>Создание частного пиринга Azure

1. **Создайте канал ExpressRoute.**

   Выполните инструкции по созданию [канала ExpressRoute](expressroute-howto-circuit-classic.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения оказывает услуги третьего уровня, он может включить для вас частный пиринг Azure. В этом случае инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией за вас, после создания канала выполните приведенные ниже инструкции.
2. **Проверьте, подготовлен ли канал ExpressRoute.**
   
   Убедитесь, что канал ExpressRoute подготовлен и включен.

   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Выходные данные:

   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Убедитесь, что канал отображается как подготовленный и включенный. Если он не подготовлен и отключен, обратитесь к поставщику услуг подключения, чтобы перевести канал в нужное состояние.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
3. **Настройте для канала частный пиринг Azure.**

   Прежде чем продолжить, убедитесь в наличии следующих элементов:
   
   * Подсеть /30 для основной ссылки. Она не должна входить в адресное пространство, зарезервированное для виртуальных сетей.
   * Подсеть /30 для дополнительной ссылки. Она не должна входить в адресное пространство, зарезервированное для виртуальных сетей.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор виртуальной локальной сети не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS. Для этого пиринга можно использовать частный номер AS. Не используйте номер 65515.
   * Хэш MD5, если вы решите его использовать. **Необязательно**.
     
   Вы можете использовать код из следующего примера, чтобы настроить для канала частный пиринг Azure:

   ```powershell
   New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100
   ```    

   Если нужно использовать хэш MD5, используйте код из следующего примера, чтобы настроить для канала частный пиринг:

   ```powershell
   New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100 -SharedKey "A1B2C3D4"
   ```
     
   > [!IMPORTANT]
   > Убедитесь, что номер AS указан в качестве ASN пиринга, а не ASN клиента.
   > 

### <a name="to-view-azure-private-peering-details"></a>Просмотр сведений о частном пиринге Azure

Для просмотра сведений о конфигурации можно использовать следующий командлет:

```powershell
Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

Выходные данные:

```
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 100
```

### <a name="to-update-azure-private-peering-configuration"></a>Обновление конфигурации частного пиринга Azure

С помощью указанного ниже командлета можно обновить любую часть конфигурации. В приведенном ниже примере идентификатор виртуальной локальной сети канала изменяется со 100 на 500.

```powershell
Set-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 500 -SharedKey "A1B2C3D4"
```

### <a name="to-delete-azure-private-peering"></a>Удаление частного пиринга Azure

Для удаления конфигурации пиринга выполните следующий командлет: Перед выполнением этого командлета отсоедините от канала ExpressRoute все виртуальные сети.

```powershell
Remove-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

## <a name="azure-public-peering"></a>Общедоступный пиринг Azure

В этом разделе описано, как создать, получить, обновить и удалить конфигурацию общедоступного пиринга Azure для канала ExpressRoute.

> [!NOTE]
> Общедоступный пиринг Azure устарел для новых каналов.
>

### <a name="to-create-azure-public-peering"></a>Создание общедоступного пиринга Azure

1. **Создание канала ExpressRoute**

   Выполните инструкции по созданию [канала ExpressRoute](expressroute-howto-circuit-classic.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения оказывает услуги третьего уровня, он может включить для вас частный пиринг Azure. В этом случае инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией за вас, после создания канала выполните приведенные ниже инструкции.
2. **Проверьте, подготовлен ли канал ExpressRoute.**

   Сначала убедитесь, что канал ExpressRoute подготовлен и включен.

   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Выходные данные:

   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Убедитесь, что канал отображается как подготовленный и включенный. Если он не подготовлен и отключен, обратитесь к поставщику услуг подключения, чтобы перевести канал в нужное состояние.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
4. **Настройка общедоступного пиринга Azure для канала**
   
   Перед началом работы убедитесь, что у вас есть следующие сведения.
   
   * Подсеть /30 для основной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4.
   * Подсеть /30 для дополнительной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор виртуальной локальной сети не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS.
   * Хэш MD5, если вы решите его использовать. **Необязательно**.

   > [!IMPORTANT]
   > Номер AS должен быть указан в качестве ASN пиринга, а не ASN клиента.
   >  
     
   Чтобы настроить для канала общедоступный пиринг Azure, можно использовать код из примера ниже:

   ```powershell
   New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200
   ```
     
   Если нужно использовать хэш MD5, используйте код из следующего примера, чтобы настроить канал:
     
   ```powershell
   New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200 -SharedKey "A1B2C3D4"
   ```
     
### <a name="to-view-azure-public-peering-details"></a>Просмотр сведений об общедоступном пиринге Azure

Чтобы просмотреть сведения о конфигурации, используйте следующий командлет:

```powershell
Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

Выходные данные:

```powershell
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 131.107.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 131.107.0.4/30
State                          : Enabled
VlanId                         : 200
```

### <a name="to-update-azure-public-peering-configuration"></a>Обновление конфигурации общедоступного пиринга Azure

С помощью указанного ниже командлета можно обновить любую часть конфигурации. В приведенном ниже примере значение идентификатора виртуальной локальной сети для канала изменяется с 200 на 600.

```powershell
Set-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 600 -SharedKey "A1B2C3D4"
```

Убедитесь, что канал отображается как подготовленный и включенный. 
### <a name="to-delete-azure-public-peering"></a>Удаление общедоступного пиринга Azure

Для удаления конфигурации пиринга выполните следующий командлет.

```powershell
Remove-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

## <a name="microsoft-peering"></a>Пиринг Майкрософт

В этом разделе описано, как создать, получить, обновить и удалить конфигурацию пиринга Майкрософт для канала ExpressRoute. 

### <a name="to-create-microsoft-peering"></a>Создание пиринга Майкрософт

1. **Создание канала ExpressRoute**
  
   Выполните инструкции по созданию [канала ExpressRoute](expressroute-howto-circuit-classic.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения оказывает услуги третьего уровня, он может включить для вас частный пиринг Azure. В этом случае инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией за вас, после создания канала выполните приведенные ниже инструкции.
2. **Проверьте, подготовлен ли канал ExpressRoute.**

   Убедитесь, что канал отображается как подготовленный и включенный. 
   
   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Выходные данные:
   
   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Убедитесь, что канал отображается как подготовленный и включенный. Если он не подготовлен и отключен, обратитесь к поставщику услуг подключения, чтобы перевести канал в нужное состояние.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
3. **Настройка пиринга Майкрософт для канала**
   
    Перед началом работы убедитесь, что у вас есть следующие сведения.
   
   * Подсеть /30 для основной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4, принадлежащего вам и зарегистрированного в RIR/IRR.
   * Подсеть /30 для дополнительной ссылки. Это должен быть допустимый префикс общедоступного адреса IPv4, принадлежащего вам и зарегистрированного в RIR/IRR.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор виртуальной локальной сети не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS.
   * Объявленные префиксы: необходимо предоставить список всех префиксов, которые вы планируете объявить во время сеанса BGP. Допускаются только общедоступные префиксы IP-адресов. Если планируется отправить набор префиксов, можно отправить список с разделителями-запятыми. Эти префиксы должны быть зарегистрированы в RIR/IRR на ваше имя.
   * Клиент ASN: для объявления префиксов, не зарегистрированных с номером AS для пиринга, можно указать номер AS, с которым они зарегистрированы. **Необязательно**.
   * Имя реестра маршрутизации: можно указать RIR/IRR, в котором зарегистрированы номер AS и префиксы.
   * Хэш MD5, если вы решите его использовать. **Необязательный элемент.**
     
   Чтобы настроить пиринг Майкрософт для своего канала, выполните следующий командлет:
 
   ```powershell
   New-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
   ```

### <a name="to-view-microsoft-peering-details"></a>Просмотр сведений о пиринге Майкрософт

Для просмотра сведений о конфигурации можно использовать следующий командлет:

```powershell
Get-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```
Выходные данные:

```powershell
AdvertisedPublicPrefixes       : 123.0.0.0/30
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 2245
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : ARIN
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 300
```

### <a name="to-update-microsoft-peering-configuration"></a>Обновление конфигурации пиринга Майкрософт

С помощью указанного ниже командлета можно обновить любую часть конфигурации.

```powershell
Set-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
```

### <a name="to-delete-microsoft-peering"></a>Удаление пиринга Майкрософт

Для удаления конфигурации пиринга выполните следующий командлет.

```powershell
Remove-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```

## <a name="next-steps"></a>Дальнейшие действия

Затем необходимо выполнить [связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-classic.md).

* Дополнительные сведения о рабочих процессах см. в разделе [Рабочие процессы ExpressRoute](expressroute-workflows.md).
* Дополнительную информацию о пиринге канала см. в статье [Каналы ExpressRoute и домены маршрутизации](expressroute-circuit-peerings.md).
