---
title: 'Azure ExpressRoute: привязка виртуальной сети к цепи: классический'
description: В этом документе содержатся общие сведения о связывании виртуальных сетей с каналами ExpressRoute с помощью классической модели развертывания и PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: duau
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0e8a1d48934d73f1035c6f2c4c76e56d9a0e4052
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92206976"
---
# <a name="connect-a-virtual-network-to-an-expressroute-circuit-using-powershell-classic"></a>Подключение виртуальной сети к каналу ExpressRoute с помощью PowerShell (классическая модель)
> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [Azure CLI](howto-linkvnet-cli.md)
> * [Видео — портал Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-linkvnet-classic.md)
>

Эта статья поможет вам связать виртуальные сети с каналами Azure ExpressRoute с помощью PowerShell. Отдельную виртуальную сеть можно связать не более чем с четырьмя каналами ExpressRoute. Чтобы создать связи с каждым каналом ExpressRoute, к которому вы подключаетесь, выполните следующие шаги. Каналы ExpressRoute могут быть размещены в той же подписке, в других подписках или и там, и там. Эта статья относится к виртуальным сетям, созданным с помощью классической модели развертывания.

Вы можете связать с каналом ExpressRoute до 10 виртуальных сетей включительно. Все виртуальные машины должны находиться в одном геополитическом регионе. При использовании надстройки Premium вы сможете связать больше виртуальных сетей с каналом ExpressRoute или связать виртуальные сети, которые находятся в других геополитических регионах. Дополнительную информацию о надстройке Premium см. в разделе [Вопросы и ответы](expressroute-faqs.md).

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**О моделях развертывания Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]


## <a name="configuration-prerequisites"></a>Предварительные требования для настройки

* Прежде чем приступить к настройке, изучите [предварительные требования](expressroute-prerequisites.md), [требования к маршрутизации](expressroute-routing.md) и [рабочие процессы](expressroute-workflows.md).
* Вам потребуется активный канал ExpressRoute.
   * Следуйте инструкциям, чтобы [создать канал ExpressRoute](expressroute-howto-circuit-classic.md) и включить его на стороне поставщика услуг подключения.
   * Убедитесь, что для вашего канала настроен частный пиринг Azure. Инструкции по маршрутизации см. в статье [Настройка маршрутизации](expressroute-howto-routing-classic.md).
   * Для создания сквозного подключения обязательно настройте частный пиринг Azure, а также пиринг BGP между вашей сетью и сетью Майкрософт.
   * Вам необходимо иметь созданные и полностью подготовленные виртуальную сеть и шлюз виртуальной сети. Следуйте инструкциям по [настройке виртуальной сети для ExpressRoute](./expressroute-howto-add-gateway-portal-resource-manager.md).

### <a name="download-the-latest-powershell-cmdlets"></a>Скачивание последних версий командлетов PowerShell

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>Подключение к каналу виртуальной сети в той же подписке
Вы можете связать виртуальную сеть с каналом ExpressRoute, используя следующий командлет. Убедитесь в наличии шлюза виртуальной сети и его готовности к связыванию, прежде чем запускать командлет.

```powershell
New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
Provisioned
```
    
## <a name="remove-a-virtual-network-link-to-a-circuit"></a>Удаление связи виртуальной сети с каналом
Вы можете удалить связь виртуальной сети с каналом ExpressRoute, используя командлет ниже. Для данной виртуальной сети выберите текущую подписку. 

```powershell
Remove-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
```
 

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>Подключение к каналу виртуальной сети в другой подписке
Канал ExpressRoute может совместно использоваться несколькими подписками. На рисунке ниже схематично показан способ совместного использования каналов ExpressRoute несколькими подписками.

Каждое маленькое облако внутри большого облака представляет подписки, принадлежащие различным подразделениям одной организации. Любое подразделение в организации может использовать свою собственную подписку для развертывания своих служб. Кроме того, подразделения могут совместно использовать один выделенный канал ExpressRoute для подключения к корпоративной сети. Владельцем канала ExpressRoute может выступать одно подразделение (в данном примере — ИТ-подразделение). Другие подписки в организации могут использовать канал ExpressRoute.

> [!NOTE]
> Плата за подключение выделенного канала ExpressRoute и использование полосы пропускания будет взиматься с владельца выделенного канала. Полоса пропускания распределяется между всеми виртуальными сетями.
> 
> 

![Подключение между подписками](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### <a name="administration"></a>Администрирование
*Владелец канала* — это администратор или соадминистратор подписки, в которой создан канал ExpressRoute. Владелец канала может разрешить использовать свой выделенный канал администраторам и соадминистраторам других подписок (на схеме рабочего процесса они называются *пользователями канала*). Пользователи, получившие разрешение на использование канала ExpressRoute организации, смогут связать виртуальную сеть в своей подписке с этим каналом ExpressRoute.

Владелец канала имеет право изменить или отменить авторизацию в любое время. Отмена авторизации приводит к удалению всех ссылок из подписки, доступ которой был отменен.

### <a name="circuit-owner-operations"></a>Действия владельца канала

**Создание разрешения**

Владелец канала разрешает администраторам других подписок использовать указанный канал. В приведенном ниже примере администратор канала (Contoso IT) разрешает администратору другой подписки (Dev-Test) привязать к этому каналу две виртуальные сети. Администратор Contoso IT разрешает это, указав идентификатор Майкрософт подписки Dev-Test. Командлет не отправляет по электронной почте указанный идентификатор Майкрософт. Владелец канала должен сам уведомить владельца другой подписки о том, что авторизация завершена.

```powershell
New-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -Description "Dev-Test Links" -Limit 2 -MicrosoftIds 'devtest@contoso.com'
```

  Выходные данные:

  ```powershell
  Description         : Dev-Test Links
  Limit               : 2
  LinkAuthorizationId : **********************************
  MicrosoftIds        : devtest@contoso.com
  Used                : 0
  ```

**Просмотр разрешений**

Владелец канала может просмотреть все разрешения, выданные для определенного канала, выполнив следующий командлет.

```powershell
Get-AzureDedicatedCircuitLinkAuthorization -ServiceKey: "**************************"
```
  Выходные данные:

  ```powershell
  Description         : EngineeringTeam
  Limit               : 3
  LinkAuthorizationId : ####################################
  MicrosoftIds        : engadmin@contoso.com
  Used                : 1

  Description         : MarketingTeam
  Limit               : 1
  LinkAuthorizationId : @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
  MicrosoftIds        : marketingadmin@contoso.com
  Used                : 0

  Description         : Dev-Test Links
  Limit               : 2
  LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
  MicrosoftIds        : salesadmin@contoso.com
  Used                : 2
  ```

**Изменение разрешений**

Владелец канала может изменять разрешения с помощью следующего командлета.

```powershell
Set-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -AuthorizationId "&&&&&&&&&&&&&&&&&&&&&&&&&&&&"-Limit 5
```

  Выходные данные:

  ```powershell
  Description         : Dev-Test Links
  Limit               : 5
  LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
  MicrosoftIds        : devtest@contoso.com
  Used                : 0
  ```

**Удаление разрешений**

Владелец канала может отзывать (удалять) разрешения, выданные пользователю, с помощью следующего командлета.

```powershell
Remove-AzureDedicatedCircuitLinkAuthorization -ServiceKey "*****************************" -AuthorizationId "###############################"
```

### <a name="circuit-user-operations"></a>Действия пользователя канала

**Просмотр разрешений**

Пользователь канала может просматривать разрешения с помощью следующего командлета.

```powershell
Get-AzureAuthorizedDedicatedCircuit
```

  Выходные данные:

  ```powershell
  Bandwidth                        : 200
  CircuitName                      : ContosoIT
  Location                         : Washington DC
  MaximumAllowedLinks              : 2
  ServiceKey                       : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
  ServiceProviderName              : equinix
  ServiceProviderProvisioningState : Provisioned
  Status                           : Enabled
  UsedLinks                        : 0
  ```

**Активация разрешений на связь**

Пользователь канала может активировать разрешение на связь, выполнив следующий командлет.

```powershell
New-AzureDedicatedCircuitLink –servicekey "&&&&&&&&&&&&&&&&&&&&&&&&&&" –VnetName 'SalesVNET1'
```

  Выходные данные:

  ```powershell
  State VnetName
  ----- --------
  Provisioned SalesVNET1
  ```

Выполните следующую команду в только что связанной подписке для виртуальной сети:

```powershell
New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об ExpressRoute см. в статье [Вопросы и ответы по ExpressRoute](expressroute-faqs.md).