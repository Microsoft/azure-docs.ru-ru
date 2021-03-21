---
title: 'Azure ExpressRoute: Настройка Direct ExpressRoute'
description: Узнайте, как использовать Azure PowerShell, чтобы настроить Azure ExpressRoute Direct для прямого подключения к глобальной сети Майкрософт.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/14/2020
ms.author: duau
ms.openlocfilehash: f54c22a0c2f7bf89d790dbd33f748446a871d224
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102099953"
---
# <a name="how-to-configure-expressroute-direct"></a>Настройка Direct для ExpressRoute

ExpressRoute Direct дает возможность напрямую подключаться к глобальной сети Майкрософт через равноправные расположения, стратегически распределенные по всему миру. См. дополнительные сведения об [ExpressRoute Direct](expressroute-erdirect-about.md).

## <a name="before-you-begin"></a>Перед началом

Прежде чем использовать ExpressRoute Direct, сначала необходимо зарегистрировать подписку. Прежде чем использовать ExpressRoute Direct, сначала необходимо зарегистрировать подписку. Для регистрации выполните следующие действия с помощью Azure PowerShell.
1.  Войдите в Azure и выберите подписку, которую хотите зарегистрировать.

    ```azurepowershell-interactive
    Connect-AzAccount 

    Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
    ```

2. Зарегистрируйте подписку для общедоступной предварительной версии с помощью следующей команды:
    ```azurepowershell-interactive
    Register-AzProviderFeature -FeatureName AllowExpressRoutePorts -ProviderNamespace Microsoft.Network
    ```

После регистрации убедитесь, что поставщик ресурсов **Microsoft. Network** зарегистрирован в вашей подписке. Регистрация поставщика ресурсов настраивает подписку для работы с поставщиком ресурсов.

## <a name="create-the-resource"></a><a name="resources"></a>Создание ресурса

1. Войдите в Azure и выберите подписку. Ресурс ExpressRoute Direct и каналы ExpressRoute должны находиться в одной подписке.

   ```powershell
   Connect-AzAccount 

   Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
   ```
   
2. Повторно зарегистрируйте подписку в Microsoft. Network, чтобы получить доступ к API-интерфейсам експрессраутепортслокатион и експрессраутепорт.

   ```powershell
   Register-AzResourceProvider -ProviderNameSpace "Microsoft.Network"
   ```   
3. Выведите список всех расположений, где поддерживается ExpressRoute Direct.
  
   ```powershell
   Get-AzExpressRoutePortsLocation
   ```

   **Пример выходных данных**
  
   ```powershell
   Name                : Equinix-Ashburn-DC2
   Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Ashburn-D
                        C2
   ProvisioningState   : Succeeded
   Address             : 21715 Filigree Court, DC2, Building F, Ashburn, VA 20147
   Contact             : support@equinix.com
   AvailableBandwidths : []

   Name                : Equinix-Dallas-DA3
   Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-Dallas-DA
                        3
   ProvisioningState   : Succeeded
   Address             : 1950 N. Stemmons Freeway, Suite 1039A, DA3, Dallas, TX 75207
   Contact             : support@equinix.com
   AvailableBandwidths : []

   Name                : Equinix-San-Jose-SV1
   Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-
                        SV1
   ProvisioningState   : Succeeded
   Address             : 11 Great Oaks Blvd, SV1, San Jose, CA 95119
   Contact             : support@equinix.com
   AvailableBandwidths : []
   ```
4. Определение, имеет ли место в приведенном выше списке доступную пропускную способность

   ```powershell
   Get-AzExpressRoutePortsLocation -LocationName "Equinix-San-Jose-SV1"
   ```

   **Пример выходных данных**

   ```powershell
   Name                : Equinix-San-Jose-SV1
   Id                  : /subscriptions/<subscriptionID>/providers/Microsoft.Network/expressRoutePortsLocations/Equinix-San-Jose-
                        SV1
   ProvisioningState   : Succeeded
   Address             : 11 Great Oaks Blvd, SV1, San Jose, CA 95119
   Contact             : support@equinix.com
   AvailableBandwidths : [
                          {
                            "OfferName": "100 Gbps",
                            "ValueInGbps": 100
                          }
                        ]
   ```
5. Создание ресурса ExpressRoute Direct на основе выбранного выше расположения

   ExpressRoute Direct поддерживает инкапсуляцию как QinQ, так и Dot1Q. Если выбран QinQ, каждому каналу ExpressRoute динамически назначается S-тег и каналы будут уникальными в пределах ресурса ExpressRoute Direct. Каждый C-тег в канале должен быть уникальным в пределах канала, но не для всего ExpressRoute Direct.  

   Если выбрана инкапсуляция Dot1Q, необходимо достичь уникальности C-тега (VLAN) в пределах всего ресурса ExpressRoute Direct.  

   > [!IMPORTANT]
   > ExpressRoute Direct может иметь только один тип инкапсуляции. После создания ExpressRoute Direct изменить инкапсуляцию невозможно.
   > 
 
   ```powershell 
   $ERDirect = New-AzExpressRoutePort -Name $Name -ResourceGroupName $ResourceGroupName -PeeringLocation $PeeringLocationName -BandwidthInGbps 100.0 -Encapsulation QinQ | Dot1Q -Location $AzureRegion
   ```

   > [!NOTE]
   > Атрибуту инкапсуляции также можно присвоить значение Dot1Q. 
   >

   **Пример результата:**

   ```powershell
   Name                       : Contoso-Direct
   ResourceGroupName          : Contoso-Direct-rg
   Location                   : westcentralus
   Id                         : /subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/exp
                               ressRoutePorts/Contoso-Direct
   Etag                       : W/"<etagnumber> "
   ResourceGuid               : <number>
   ProvisioningState          : Succeeded
   PeeringLocation            : Equinix-Seattle-SE2
   BandwidthInGbps            : 100
   ProvisionedBandwidthInGbps : 0
   Encapsulation              : QinQ
   Mtu                        : 1500
   EtherType                  : 0x8100
   AllocationDate             : Saturday, September 1, 2018
   Links                      : [
                                 {
                                   "Name": "link1",
                                   "Etag": "W/\"<etagnumber>\"",
                                   "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                               Network/expressRoutePorts/Contoso-Direct/links/link1",
                                   "RouterName": "tst-09xgmr-cis-1",
                                   "InterfaceName": "HundredGigE2/2/2",
                                   "PatchPanelId": "PPID",
                                   "RackId": "RackID",
                                   "ConnectorType": "SC",
                                   "AdminState": "Disabled",
                                   "ProvisioningState": "Succeeded"
                                 },
                                 {
                                   "Name": "link2",
                                   "Etag": "W/\"<etagnumber>\"",
                                   "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                               Network/expressRoutePorts/Contoso-Direct/links/link2",
                                   "RouterName": "tst-09xgmr-cis-2",
                                   "InterfaceName": "HundredGigE2/2/2",
                                   "PatchPanelId": "PPID",
                                   "RackId": "RackID",
                                   "ConnectorType": "SC",
                                   "AdminState": "Disabled",
                                   "ProvisioningState": "Succeeded"
                                 }
                               ]
   Circuits                   : []
   ```

## <a name="generate-the-letter-of-authorization-loa"></a><a name="authorization"></a>Создание письма авторизации (определения гарантии)

Создайте ссылку на недавно созданный прямой ресурс ExpressRoute, введите имя клиента для записи определения гарантии в и (необязательно) Определите расположение файла для хранения документа. Если на путь к файлу нет ссылки, документ будет загружен в текущий каталог.

  ```powershell 
   New-AzExpressRoutePortLOA -ExpressRoutePort $ERDirect -CustomerName TestCustomerName -Destination "C:\Users\SampleUser\Downloads" 
   ```
 **Пример выходных данных**

   ```powershell
   Written Letter of Authorization To: C:\Users\SampleUser\Downloads\LOA.pdf
   ```

## <a name="change-admin-state-of-links"></a><a name="state"></a>Изменение параметра AdminState для ссылок
   
Этот процесс следует использовать для проведения тестирования уровня 1, что в свою очередь обеспечит правильное исправление перекрестных подключений в каждом маршрутизаторе для основного и дополнительного подключения.
1. Получите подробные сведения об ExpressRoute Direct.

   ```powershell
   $ERDirect = Get-AzExpressRoutePort -Name $Name -ResourceGroupName $ResourceGroupName
   ```
2. Установите параметр ссылки как Enabled (включена). Повторите этот шаг, чтобы установить состояние каждой ссылки как Enabled (включена).

   Ссылки [0] — это основной порт, а ссылки [1] — это дополнительный порт.

   ```powershell
   $ERDirect.Links[0].AdminState = "Enabled"
   Set-AzExpressRoutePort -ExpressRoutePort $ERDirect
   $ERDirect = Get-AzExpressRoutePort -Name $Name -ResourceGroupName $ResourceGroupName
   $ERDirect.Links[1].AdminState = "Enabled"
   Set-AzExpressRoutePort -ExpressRoutePort $ERDirect
   ```
   **Пример результата:**

   ```powershell
   Name                       : Contoso-Direct
   ResourceGroupName          : Contoso-Direct-rg
   Location                   : westcentralus
   Id                         : /subscriptions/<number>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Network/exp
                             ressRoutePorts/Contoso-Direct
   Etag                       : W/"<etagnumber> "
   ResourceGuid               : <number>
   ProvisioningState          : Succeeded
   PeeringLocation            : Equinix-Seattle-SE2
   BandwidthInGbps            : 100
   ProvisionedBandwidthInGbps : 0
   Encapsulation              : QinQ
   Mtu                        : 1500
   EtherType                  : 0x8100
   AllocationDate             : Saturday, September 1, 2018
   Links                      : [
                               {
                                 "Name": "link1",
                                 "Etag": "W/\"<etagnumber>\"",
                                 "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                             Network/expressRoutePorts/Contoso-Direct/links/link1",
                                 "RouterName": "tst-09xgmr-cis-1",
                                 "InterfaceName": "HundredGigE2/2/2",
                                 "PatchPanelId": "PPID",
                                 "RackId": "RackID",
                                 "ConnectorType": "SC",
                                 "AdminState": "Enabled",
                                 "ProvisioningState": "Succeeded"
                               },
                               {
                                 "Name": "link2",
                                 "Etag": "W/\"<etagnumber>\"",
                                 "Id": "/subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.
                             Network/expressRoutePorts/Contoso-Direct/links/link2",
                                 "RouterName": "tst-09xgmr-cis-2",
                                 "InterfaceName": "HundredGigE2/2/2",
                                 "PatchPanelId": "PPID",
                                 "RackId": "RackID",
                                 "ConnectorType": "SC",
                                 "AdminState": "Enabled",
                                 "ProvisioningState": "Succeeded"
                               }
                             ]
   Circuits                   : []
   ```

   Используйте эту же процедуру с `AdminState = "Disabled"`, чтобы выключить порты.

## <a name="create-a-circuit"></a><a name="circuit"></a>Создание цепи

По умолчанию можно создать до 10 каналов в подписке, в которой есть ресурс ExpressRoute Direct. Это ограничение можно увеличить за счет поддержки. Вы несете ответственность за отслеживание как подготовленной, так и используемой пропускной способности. Подготовленная пропускная способность — это сумма пропускных способностей всех каналов ресурса ExpressRoute Direct, а использованная пропускная способность — это физическое использование базовых физических интерфейсов.

Существует дополнительная полоса пропускания канала, которую можно использовать в ExpressRoute Direct для поддержки только описанных выше сценариев. Эти пропускная способность составляет 40 Гбит/с и 100 Гбит/с.

**Скутиер** может быть локальным, стандартным или Premium.

**Скуфамили** может быть только MeteredData. Неограниченное число не поддерживается в ExpressRoute Direct.

Создайте канал в ресурсе ExpressRoute Direct.

  ```powershell
  New-AzExpressRouteCircuit -Name $Name -ResourceGroupName $ResourceGroupName -ExpressRoutePort $ERDirect -BandwidthinGbps 100.0  -Location $AzureRegion -SkuTier Premium -SkuFamily MeteredData 
  ```

  Другие значения пропускной способности включают в себя: 5.0, 10.0 и 40.0

  **Пример результата:**

  ```powershell
  Name                             : ExpressRoute-Direct-ckt
  ResourceGroupName                : Contoso-Direct-rg
  Location                         : westcentralus
  Id                               : /subscriptions/<subscriptionID>/resourceGroups/Contoso-Direct-rg/providers/Microsoft.Netwo
                                   rk/expressRouteCircuits/ExpressRoute-Direct-ckt
  Etag                             : W/"<etagnumber>"
  ProvisioningState                : Succeeded
  Sku                              : {
                                     "Name": "Premium_MeteredData",
                                     "Tier": "Premium",
                                     "Family": "MeteredData"
                                   }
  CircuitProvisioningState         : Enabled
  ServiceProviderProvisioningState : Provisioned
  ServiceProviderNotes             : 
    ServiceProviderProperties        : null
  ExpressRoutePort                 : {
                                     "Id": "/subscriptions/<subscriptionID>n/resourceGroups/Contoso-Direct-rg/providers/Micros
                                   oft.Network/expressRoutePorts/Contoso-Direct"
                                   }
  BandwidthInGbps                  : 10
  Stag                             : 2
  ServiceKey                       : <number>
  Peerings                         : []
  Authorizations                   : []
  AllowClassicOperations           : False
  GatewayManagerEtag     
  ```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании ExpressRoute Direct см. в [обзоре](expressroute-erdirect-about.md).
