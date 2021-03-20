---
title: Использование с внутренней Load Balancer — шлюз приложений Azure
description: На этой странице приводятся инструкции по созданию, настройке, запуску и удалению шлюза приложений Azure с внутренним балансировщиком нагрузки (ILB) с помощью диспетчера ресурсов Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 3d663dc4e2bd860ec9494785ecbf6dbf10a4c5b5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93397762"
---
# <a name="create-an-application-gateway-with-an-internal-load-balancer-ilb"></a>Создание шлюза приложений с внутренней подсистемой балансировки нагрузки (ILB)

Шлюз приложений Azure можно настроить на использование виртуального IP-адреса, доступного в Интернете, или внутренней конечной точки, недоступной в Интернете. Эта точка также называется конечной точкой внутреннего балансировщика нагрузки (ILB). Настройка шлюза с ILB подходит для внутренних бизнес-приложений, недоступных в Интернете. Он также полезен для служб и уровней в многоуровневых приложениях, которые находятся в границах безопасности, которые не доступны в Интернете, но по-прежнему нуждаются в распределении нагрузки с циклическим перебором, прикреплениях сеансов или TLS, ранее известной как SSL (SSL).

В этой статье приводятся пошаговые инструкции по настройке шлюза приложений с использованием ILB.

## <a name="before-you-begin"></a>Перед началом

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

1. Установите последнюю версию модуля Azure PowerShell, следуя [инструкциям по установке](/powershell/azure/install-az-ps).
2. Вы создадите виртуальную сеть и подсеть для шлюза приложений. Убедитесь, что подсеть не используется виртуальной машиной или облачным развертыванием. Сам шлюз приложений должен находиться в подсети виртуальной сети.
3. Для использования шлюза приложений настраиваются существующие серверы или серверы, для которых в виртуальной сети созданы конечные точки либо же назначен общедоступный или виртуальный IP-адрес.

## <a name="what-is-required-to-create-an-application-gateway"></a>Что необходимо для создания шлюза приложений?

* **Пул внутренних серверов:** Список IP-адресов внутренних серверов. Указанные IP-адреса должны относиться к одной виртуальной сети (но не входить в подсеть шлюза приложений) либо представлять собой общедоступные или виртуальные IP-адреса.
* **Параметры пула внутренних серверов:** У каждого пула есть такие параметры, как порт, протокол и сходство на основе файлов cookie. Эти параметры привязываются к пулу и применяются ко всем серверам в этом пуле.
* **Интерфейсный порт**. Общедоступный порт, открытый в шлюзе приложений. Трафик поступает на этот порт, а затем перенаправляется на один из тыловых серверов.
* **Прослушиватель:** Прослушиватель имеет интерфейсный порт, протокол (HTTP или HTTPS, с учетом регистра) и имя SSL-сертификата (при настройке разгрузки SSL).
* **Правило:** Правило привязывает прослушиватель и пул внутренних серверов, а также определяет, к какому внутреннему пулу серверов следует направить трафик при попадании в конкретный прослушиватель. В настоящее время поддерживается только *основное* правило. *Основное* правило предусматривает циклическое распределение нагрузки.

## <a name="create-an-application-gateway"></a>Создание шлюза приложений

Разница между использованием классической модели развертывания Azure и модели Azure Resource Manager заключается в порядке создания шлюза приложения и элементов, которые нужно настроить.
При использовании Resource Manager все элементы, которые будут включены в единый ресурс шлюза приложений, сначала настраиваются по отдельности.

Ниже приведены пошаговые инструкции по созданию шлюза приложений.

1. Создание группы ресурсов для диспетчера ресурсов.
2. Создание виртуальной сети и подсети для шлюза приложений.
3. Создание объекта конфигурации шлюза приложений.
4. Создание ресурса шлюза приложений.

## <a name="create-a-resource-group-for-resource-manager"></a>Создание группы ресурсов для диспетчера ресурсов.

Для работы с командлетами диспетчера ресурсов Azure необходимо перейти в режим PowerShell. Дополнительные сведения см. в статье [Использование Windows PowerShell с диспетчером ресурсов](../azure-resource-manager/management/manage-resources-powershell.md).

### <a name="step-1"></a>Шаг 1

```powershell
Connect-AzAccount
```

### <a name="step-2"></a>Шаг 2

Просмотрите подписки учетной записи.

```powershell
Get-AzSubscription
```

Вам будет предложено указать свои учетные данные для проверки подлинности.

### <a name="step-3"></a>Шаг 3

Выберите, какие подписки Azure будут использоваться.

```powershell
Select-AzSubscription -Subscriptionid "GUID of subscription"
```

### <a name="step-4"></a>Шаг 4

Создайте группу ресурсов (если вы используете существующую группу, пропустите этот шаг).

```powershell
New-AzResourceGroup -Name appgw-rg -location "West US"
```

Диспетчер ресурсов Azure требует, чтобы все группы ресурсов указывали расположение. Оно используется в качестве расположения по умолчанию для всех ресурсов данной группы. Убедитесь, что во всех командах для создания шлюза приложений используется одна группа ресурсов.

В приведенном выше примере мы создали группу ресурсов appgw-rg в расположении West US (западная часть США).

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Создание виртуальной сети и подсети для шлюза приложений.

В следующем примере показано создание виртуальной сети с помощью диспетчера ресурсов.

### <a name="step-1"></a>Шаг 1

```powershell
$subnetconfig = New-AzVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24
```

На этом шаге выполняется назначение диапазона адресов 10.0.0.0/24 переменной подсети, которая будет использоваться для создания виртуальной сети.

### <a name="step-2"></a>Шаг 2

```powershell
$vnet = New-AzVirtualNetwork -Name appgwvnet -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $subnetconfig
```

На этом шаге создается виртуальная сеть appgwvnet в группе ресурсов appgw-rg для региона западная часть США с помощью префикса 10.0.0.0/16 с подсетью 10.0.0.0/24.

### <a name="step-3"></a>Шаг 3

```powershell
$subnet = $vnet.subnets[0]
```

На этом шаге выполняется присвоение объекта подсети переменной $subnet для выполнения следующих действий.

## <a name="create-an-application-gateway-configuration-object"></a>Создание объекта конфигурации шлюза приложений.

### <a name="step-1"></a>Шаг 1

```powershell
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet
```

На этом шаге создается конфигурация IP-адреса шлюза приложений с именем gatewayIP01. При запуске шлюз приложений получает IP-адрес из настроенной подсети. Затем шлюз маршрутизирует сетевой трафик на IP-адреса из пула внутренних IP-адресов. Помните, что для каждого экземпляра требуется отдельный IP-адрес.

### <a name="step-2"></a>Шаг 2

```powershell
$pool = New-AzApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 10.1.1.8,10.1.1.9,10.1.1.10
```

В этом шаге выполняется настройка внутреннего пула IP-адресов pool01 с IP-адресами 10.1.1.8, 10.1.1.9, 10.1.1.10. Эти адреса будут использоваться для получения сетевого трафика от конечной точки с интерфейсным IP-адресом. Замените приведенные выше IP-адреса и добавьте IP-адреса конечных точек своего приложения.

### <a name="step-3"></a>Шаг 3

```powershell
$poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Disabled
```

На этом шаге осуществляется настройка параметров шлюза приложений poolsetting01 для балансировки нагрузки сетевого трафика в пуле серверной части.

### <a name="step-4"></a>Шаг 4

```powershell
$fp = New-AzApplicationGatewayFrontendPort -Name frontendport01  -Port 80
```

На этом шаге выполняется настройка внешнего IP-порта с именем frontendport01 для внутренней подсистемы балансировки нагрузки (ILB).

### <a name="step-5"></a>Шаг 5

```powershell
$fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name fipconfig01 -Subnet $subnet
```

На этом шаге выполняется создание конфигурации внешнего IP-адреса с именем fipconfig01 и ее связывание с частным IP-адресом из текущей подсети виртуальной сети.

### <a name="step-6"></a>Шаг 6

```powershell
$listener = New-AzApplicationGatewayHttpListener -Name listener01  -Protocol Http -FrontendIPConfiguration $fipconfig -FrontendPort $fp
```

На этом шаге выполняется создание прослушивателя listener01 и связывание внешнего порта с конфигурацией внешнего IP-адреса.

### <a name="step-7"></a>Шаг 7

```powershell
$rule = New-AzApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
```

На этом шаге выполняется создание правила маршрутизации rule01 для настройки поведения подсистемы балансировки нагрузки.

### <a name="step-8"></a>Шаг 8

```powershell
$sku = New-AzApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
```

На этом шаге выполняется настройка размера экземпляра шлюза приложений.

> [!NOTE]
> Значение по умолчанию для емкости — 2. Для имени SKU вы можете выбрать значения Standard_Small, Standard_Medium или Standard_Large.

## <a name="create-an-application-gateway-by-using-new-azureapplicationgateway"></a>Создание шлюза приложений с помощью командлета New-AzureApplicationGateway

Создайте шлюз приложений со всеми элементами конфигурации, описанными выше. В этом примере шлюз приложений называется "appgwtest".

```powershell
$appgw = New-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku
```

На этом шаге создается шлюз приложений со всеми элементами конфигурации, описанными выше. В этом примере шлюз приложений называется appgwtest.

## <a name="delete-an-application-gateway"></a>Удаление шлюза приложений

Чтобы удалить шлюз приложений, по порядку выполните указанные ниже действия.

1. Остановите шлюз с помощью командлета `Stop-AzApplicationGateway`.
2. Удалите шлюз с помощью командлета `Remove-AzApplicationGateway`.
3. С помощью командлета `Get-AzureApplicationGateway` проверьте, удален ли шлюз.

### <a name="step-1"></a>Шаг 1

Получите объект шлюза приложений и свяжите его с переменной $getgw.

```powershell
$getgw =  Get-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg
```

### <a name="step-2"></a>Шаг 2

С помощью командлета `Stop-AzApplicationGateway` остановите шлюз приложений. В данном примере командлет `Stop-AzApplicationGateway` показан в первой строке, а за ним следуют выходные данные.

```powershell
Stop-AzApplicationGateway -ApplicationGateway $getgw  
```

```
VERBOSE: 9:49:34 PM - Begin Operation: Stop-AzureApplicationGateway
VERBOSE: 10:10:06 PM - Completed Operation: Stop-AzureApplicationGateway
Name       HTTP Status Code     Operation ID                             Error
----       ----------------     ------------                             ----
Successful OK                   ce6c6c95-77b4-2118-9d65-e29defadffb8
```

Когда шлюз будет остановлен, удалите службу с помощью командлета `Remove-AzApplicationGateway`.

```powershell
Remove-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg -Force
```

```
VERBOSE: 10:49:34 PM - Begin Operation: Remove-AzureApplicationGateway
VERBOSE: 10:50:36 PM - Completed Operation: Remove-AzureApplicationGateway
Name       HTTP Status Code     Operation ID                             Error
----       ----------------     ------------                             ----
Successful OK                   055f3a96-8681-2094-a304-8d9a11ad8301
```

> [!NOTE]
> Если указать параметр **-force**, запрос на подтверждение удаления не появится.

Для проверки того, удалена ли служба, используйте командлет `Get-AzApplicationGateway`. Этот шаг не является обязательным.

```powershell
Get-AzApplicationGateway -Name appgwtest -ResourceGroupName appgw-rg
```

```
VERBOSE: 10:52:46 PM - Begin Operation: Get-AzureApplicationGateway

Get-AzureApplicationGateway : ResourceNotFound: The gateway does not exist.
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы настроить разгрузку SSL, ознакомьтесь с [настройкой шлюза приложений для разгрузки SSL](./tutorial-ssl-powershell.md).

Дополнительные сведения о параметрах балансировки нагрузки в целом см. в статьях:

* [Azure Load Balancer](https://azure.microsoft.com/documentation/services/load-balancer/)
* [Azure Traffic Manager](https://azure.microsoft.com/documentation/services/traffic-manager/)