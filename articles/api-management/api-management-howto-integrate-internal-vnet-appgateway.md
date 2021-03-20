---
title: Как использовать управление API в виртуальной сети с помощью шлюза приложений
titleSuffix: Azure API Management
description: Узнайте, как установить и настроить службу управления API Azure во внутренней виртуальной сети с интерфейсным шлюзом приложений (WAF)
services: api-management
documentationcenter: ''
author: solankisamir
manager: kjoshi
editor: vlvinogr
ms.assetid: a8c982b2-bca5-4312-9367-4a0bbc1082b1
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: sasolank
ms.openlocfilehash: 3db1c8bfc3a11151342589af0873d88e3d90c6a1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91825634"
---
# <a name="integrate-api-management-in-an-internal-vnet-with-application-gateway"></a>Интеграция службы управления API во внутреннюю сеть со шлюзом приложений

## <a name="overview"></a><a name="overview"></a> Общие сведения

Если настроить службу управления API для работы в виртуальной сети в режиме внутренней сети, она будет доступна только из этой виртуальной сети. Шлюз приложений Azure — это служба PaaS, выполняющая функции подсистемы балансировки нагрузки на сетевом уровне 7. Это служба обратного прокси-сервера, которая содержит также брандмауэр веб-приложения (WAF).

Объединяя возможности службы управления API, работающей во внутренней виртуальной сети, и интерфейса шлюза приложений, вы можете реализовать следующие сценарии.

* Использовать один ресурс управления API одновременно и для внешних, и для внутренних потребителей.
* Использовать один ресурс управления API, определив для него в службе управления API подмножество API-интерфейсов, доступных для внешних потребителей.
* Создать простой способ включать и отключать доступ из Интернета к управлению API.

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы выполнить действия, описанные в этой статье, необходимо следующее:

* Активная подписка Azure.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

* Сертификаты — PFX- и CER-файл для имени узла API и PFX-файл для имени узла на портале разработчика.

## <a name="scenario"></a><a name="scenario"></a> Сценарий

В этой статье описывается, как использовать одну службу управления API как для внутренних, так и для внешних потребителей, а также как единый интерфейс для локальных и облачных API. Также вы узнаете, как предоставить некоторое подмножество этих API-интерфейсов (в нашем примере они выделены зеленым цветом) для внешнего использования, применив функцию маршрутизации, предоставляемую Шлюзом приложений.

В первом примере конфигурации все API-интерфейсы управляются только из виртуальной сети. Внутренние потребители (выделены оранжевым цветом) могут обращаться ко всем внутренним и внешним API. Трафик никогда не переходит в Интернет. Высокопроизводительное подключение доставляется через каналы Express Route.

![Маршрут URL-адреса](./media/api-management-howto-integrate-internal-vnet-appgateway/api-management-howto-integrate-internal-vnet-appgateway.png)

## <a name="before-you-begin"></a><a name="before-you-begin"></a> Перед началом

* Убедитесь, что у вас установлена последняя версия Azure PowerShell. Инструкции по установке см. в статье [установка Azure PowerShell](/powershell/azure/install-az-ps). 

## <a name="what-is-required-to-create-an-integration-between-api-management-and-application-gateway"></a>Что нужно для того, чтобы интегрировать управление API со шлюзом приложений?

* **Пул внутренних серверов:** Это внутренний виртуальный IP-адрес службы управления API.
* **Параметры пула внутренних серверов:** У каждого пула есть такие параметры, как порт, протокол и сходство на основе файлов cookie. Эти параметры применяются ко всем серверам в этом пуле.
* **Внешний порт**. Общедоступный порт, открытый в шлюзе приложений. Трафик, поступающий на этот порт, перенаправляется на один из тыловых серверов.
* **Прослушиватель:** Прослушиватель имеет интерфейсный порт, протокол (HTTP или HTTPS, эти значения чувствительны к регистру) и имя сертификата TLS/SSL (при настройке разгрузки TLS).
* **Правило**. Это правило связывает прослушиватель с пулом тыловых серверов.
* **Пользовательские проверки работоспособности**. Шлюз приложений по умолчанию использует проверки на основе IP-адреса, чтобы найти активные серверы в пуле BackendAddressPool. Служба управления API отвечает только на те запросы, которые имеют правильный заголовок узла, поэтому стандартные проверки завершаются ошибкой. Вам следует определить пользовательскую проверку работоспособности, чтобы шлюз приложений мог определять работоспособность службы и передавать в нее запросы.
* **Сертификат для личного домена**. Чтобы осуществлять доступ из Интернета в службу управления API, создайте сопоставление CNAME, связывающее имя узла с DNS-именем внешнего интерфейса Шлюза приложений. Это позволит службе управления API распознавать допустимость заголовка hostname и сертификата, который передан в шлюз приложений и пересылается в управление API. В этом примере мы будем использовать два сертификата — для внутренней службы и портала разработчика.  

## <a name="steps-required-for-integrating-api-management-and-application-gateway"></a><a name="overview-steps"> </a> Действия по интеграции управления API со шлюзом приложений

1. Создание группы ресурсов для диспетчера ресурсов.
2. Создание виртуальной сети, подсети и общедоступного IP-адреса для шлюза приложений. Создайте еще одну подсеть для управления API.
3. Создайте службу управления API в той подсети виртуальной сети, которую вы создали ранее. Должен использоваться режим внутренней сети.
4. Настройте имя личного домена в службе управления API.
5. Создайте объект конфигурации шлюза приложений.
6. Создайте ресурс шлюза приложений.
7. Создайте сопоставление CNAME, связывающее DNS-имя шлюза приложений с именем узла прокси-сервера управления API.

## <a name="exposing-the-developer-portal-externally-through-application-gateway"></a>Внешнее предоставление портала разработчика через Шлюз приложений

В этом руководстве мы также предоставим **портал разработчика** для внешней аудитории через Шлюз приложений. Для этого нужно создать прослушиватель, пробу, параметры и правила портала разработчика. Все подробности предоставляются на соответствующих этапах.

> [!WARNING]
> Если вы используете Azure AD или стороннюю аутентификацию, включите [сходство сеансов на основе файлов cookie](../application-gateway/features.md#session-affinity) в Шлюзе приложений.

> [!WARNING]
> Чтобы предотвратить нарушение загрузки спецификации OpenAPI на портале разработчика WAF шлюза приложений, необходимо отключить правило брандмауэра `942200 - "Detects MySQL comment-/space-obfuscated injections and backtick termination"` .
> 
> Правила WAF шлюза приложений, которые могут нарушить работу портала, включают:
> 
> - `920300`,,, `920330` `931130` `942100` , `942110` , `942180` , `942200` , `942260` , `942340` , `942370` для режима администрирования
> - `942200`, `942260` , `942370` , `942430` , `942440` для опубликованного портала

## <a name="create-a-resource-group-for-resource-manager"></a>Создание группы ресурсов для диспетчера ресурсов.

### <a name="step-1"></a>Шаг 1

Вход в Azure

```powershell
Connect-AzAccount
```

Выполните аутентификацию со своими учетными данными.

### <a name="step-2"></a>Шаг 2

Выберите нужную подписку.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000" # GUID of your Azure subscription
Get-AzSubscription -Subscriptionid $subscriptionId | Select-AzSubscription
```

### <a name="step-3"></a>Шаг 3

Создайте группу ресурсов. Если вы используете существующую группу, пропустите этот шаг.

```powershell
$resGroupName = "apim-appGw-RG" # resource group name
$location = "West US"           # Azure region
New-AzResourceGroup -Name $resGroupName -Location $location
```

Диспетчер ресурсов Azure требует, чтобы все группы ресурсов указывали расположение. Оно используется в качестве расположения по умолчанию для всех ресурсов данной группы. Убедитесь, что во всех командах для создания шлюза приложений используется одна группа ресурсов.

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Создание виртуальной сети и подсети для шлюза приложений

В следующем примере показано, как создать виртуальную сеть с помощью диспетчер ресурсов.

### <a name="step-1"></a>Шаг 1

Создайте переменную подсети с диапазоном адресов 10.0.0.0/24, которая будет использоваться для шлюза приложений при создании виртуальной сети.

```powershell
$appgatewaysubnet = New-AzVirtualNetworkSubnetConfig -Name "apim01" -AddressPrefix "10.0.0.0/24"
```

### <a name="step-2"></a>Шаг 2

Создайте переменную подсети с диапазоном адресов 10.0.1.0/24, которая будет использоваться для управления API при создании виртуальной сети.

```powershell
$apimsubnet = New-AzVirtualNetworkSubnetConfig -Name "apim02" -AddressPrefix "10.0.1.0/24"
```

### <a name="step-3"></a>Шаг 3

Создайте виртуальную сеть с именем **appgwvnet** в группе ресурсов **apim-appGw-RG** для региона "Западная часть США", назначив ей префикс 10.0.0.0/16. Создайте в ней подсети 10.0.0.0/24 и 10.0.1.0/24.

```powershell
$vnet = New-AzVirtualNetwork -Name "appgwvnet" -ResourceGroupName $resGroupName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $appgatewaysubnet,$apimsubnet
```

### <a name="step-4"></a>Шаг 4

Назначьте переменную для подсети, которая будет использоваться далее.

```powershell
$appgatewaysubnetdata = $vnet.Subnets[0]
$apimsubnetdata = $vnet.Subnets[1]
```

## <a name="create-an-api-management-service-inside-a-vnet-configured-in-internal-mode"></a>Создание службы управления API в виртуальной сети, настроенной в режиме внутренней сети

В следующем примере демонстрируется создание службы управления API в виртуальной сети, настроенной только для внутреннего доступа.

### <a name="step-1"></a>Шаг 1

Создайте объект виртуальной сети для управления API, используя созданную выше переменную подсети $apimsubnetdata.

```powershell
$apimVirtualNetwork = New-AzApiManagementVirtualNetwork -SubnetResourceId $apimsubnetdata.Id
```

### <a name="step-2"></a>Шаг 2

Создайте службу управления API в виртуальной сети.

```powershell
$apimServiceName = "ContosoApi"       # API Management service instance name
$apimOrganization = "Contoso"         # organization name
$apimAdminEmail = "admin@contoso.com" # administrator's email address
$apimService = New-AzApiManagement -ResourceGroupName $resGroupName -Location $location -Name $apimServiceName -Organization $apimOrganization -AdminEmail $apimAdminEmail -VirtualNetwork $apimVirtualNetwork -VpnType "Internal" -Sku "Developer"
```

Когда завершится выполнение предыдущей команды, выполните инструкции из раздела [DNS Configuration ](api-management-using-with-internal-vnet.md#apim-dns-configuration) (Настройка DNS) для доступа к службе управления API во внутренней виртуальной сети. Этот шаг может занять более получаса.

## <a name="set-up-a-custom-domain-name-in-api-management"></a>Настройка пользовательского доменного имени в службе управления API

> [!IMPORTANT]
> [Новый портал разработчика](api-management-howto-developer-portal.md) также требует включения подключения к конечной точке управления управления API в дополнение к следующим шагам.

### <a name="step-1"></a>Шаг 1

Инициализируйте следующие переменные с подробными сведениями о сертификатах с закрытыми ключами для доменов. В этом примере используется `api.contoso.net` и `portal.contoso.net`.  

```powershell
$gatewayHostname = "api.contoso.net"                 # API gateway host
$portalHostname = "portal.contoso.net"               # API developer portal host
$gatewayCertCerPath = "C:\Users\Contoso\gateway.cer" # full path to api.contoso.net .cer file
$gatewayCertPfxPath = "C:\Users\Contoso\gateway.pfx" # full path to api.contoso.net .pfx file
$portalCertPfxPath = "C:\Users\Contoso\portal.pfx"   # full path to portal.contoso.net .pfx file
$gatewayCertPfxPassword = "certificatePassword123"   # password for api.contoso.net pfx certificate
$portalCertPfxPassword = "certificatePassword123"    # password for portal.contoso.net pfx certificate

$certPwd = ConvertTo-SecureString -String $gatewayCertPfxPassword -AsPlainText -Force
$certPortalPwd = ConvertTo-SecureString -String $portalCertPfxPassword -AsPlainText -Force
```

### <a name="step-2"></a>Шаг 2

Создайте и задайте объекты конфигурации имени узла для прокси-сервера и для портала.  

```powershell
$proxyHostnameConfig = New-AzApiManagementCustomHostnameConfiguration -Hostname $gatewayHostname -HostnameType Proxy -PfxPath $gatewayCertPfxPath -PfxPassword $certPwd
$portalHostnameConfig = New-AzApiManagementCustomHostnameConfiguration -Hostname $portalHostname -HostnameType DeveloperPortal -PfxPath $portalCertPfxPath -PfxPassword $certPortalPwd

$apimService.ProxyCustomHostnameConfiguration = $proxyHostnameConfig
$apimService.PortalCustomHostnameConfiguration = $portalHostnameConfig
Set-AzApiManagement -InputObject $apimService
```

> [!NOTE]
> Чтобы настроить подключение к устаревшему порталу разработчика, необходимо заменить на `-HostnameType DeveloperPortal` `-HostnameType Portal` .

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>Создание общедоступного IP-адреса для конфигурации интерфейсной части

Создайте общедоступный IP-ресурс **publicIP01** в группе ресурсов.

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName $resGroupName -name "publicIP01" -location $location -AllocationMethod Dynamic
```

IP-адрес назначается шлюзу приложений при запуске службы.

## <a name="create-application-gateway-configuration"></a>Создание конфигурации шлюза приложений

Перед созданием шлюза приложений необходимо настроить все элементы конфигурации. В ходе следующих шагов создаются необходимые элементы конфигурации для ресурса шлюза приложений.

### <a name="step-1"></a>Шаг 1

Создайте IP-конфигурацию шлюза приложений с именем **gatewayIP01**. При запуске шлюз приложений получает IP-адрес из настроенной подсети. Затем шлюз маршрутизирует сетевой трафик на IP-адреса из пула внутренних IP-адресов. Помните, что для каждого экземпляра требуется отдельный IP-адрес.

```powershell
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name "gatewayIP01" -Subnet $appgatewaysubnetdata
```

### <a name="step-2"></a>Шаг 2

Настройте интерфейсный порт IP для конечной точки с общедоступным IP-адресом. Это порт, к которому подключаются пользователи.

```powershell
$fp01 = New-AzApplicationGatewayFrontendPort -Name "port01"  -Port 443
```

### <a name="step-3"></a>Шаг 3

Настройте внешний IP-адрес, используя конечную точку с общедоступным IP-адресом.

```powershell
$fipconfig01 = New-AzApplicationGatewayFrontendIPConfig -Name "frontend1" -PublicIPAddress $publicip
```

### <a name="step-4"></a>Шаг 4

Настройте в Шлюзе приложений сертификаты для повторного шифрования и расшифровки проходящего трафика.

```powershell
$cert = New-AzApplicationGatewaySslCertificate -Name "cert01" -CertificateFile $gatewayCertPfxPath -Password $certPwd
$certPortal = New-AzApplicationGatewaySslCertificate -Name "cert02" -CertificateFile $portalCertPfxPath -Password $certPortalPwd
```

### <a name="step-5"></a>Шаг 5

Создайте прослушиватели HTTP для Шлюза приложений. Назначьте им клиентскую конфигурацию IP-адресов, порт и TLS/SSL-сертификаты.

```powershell
$listener = New-AzApplicationGatewayHttpListener -Name "listener01" -Protocol "Https" -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $cert -HostName $gatewayHostname -RequireServerNameIndication true
$portalListener = New-AzApplicationGatewayHttpListener -Name "listener02" -Protocol "Https" -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $certPortal -HostName $portalHostname -RequireServerNameIndication true
```

### <a name="step-6"></a>Шаг 6

Создайте пользовательские пробы для конечной точки домена прокси-сервера `ContosoApi` службы управления API. По умолчанию на всех службах управления API для конечной точки проверки работоспособности используется путь `/status-0123456789abcdef`. Задайте в `api.contoso.net` качестве имени узла пользовательской пробы, чтобы защитить его с помощью сертификата TLS/SSL.

> [!NOTE]
> Когда в общедоступной службе Azure создается служба с именем `contosoapi.azure-api.net`, для ее прокси-сервера по умолчанию назначается имя узла `contosoapi`.
>

```powershell
$apimprobe = New-AzApplicationGatewayProbeConfig -Name "apimproxyprobe" -Protocol "Https" -HostName $gatewayHostname -Path "/status-0123456789abcdef" -Interval 30 -Timeout 120 -UnhealthyThreshold 8
$apimPortalProbe = New-AzApplicationGatewayProbeConfig -Name "apimportalprobe" -Protocol "Https" -HostName $portalHostname -Path "/internal-status-0123456789abcdef" -Interval 60 -Timeout 300 -UnhealthyThreshold 8
```

### <a name="step-7"></a>Шаг 7

Отправьте сертификат, который будет использоваться в ресурсах внутреннего пула с поддержкой TLS. Это тот же сертификат, который вы указали выше на шаге 4.

```powershell
$authcert = New-AzApplicationGatewayAuthenticationCertificate -Name "whitelistcert1" -CertificateFile $gatewayCertCerPath
```

### <a name="step-8"></a>Шаг 8

Настройте HTTP для внутреннего пула шлюза приложений. К настройкам относится и предел времени ожидания, по истечении которого запрос к внутренним серверам отменяется. Это значение отличается от времени ожидания проверки работоспособности.

```powershell
$apimPoolSetting = New-AzApplicationGatewayBackendHttpSettings -Name "apimPoolSetting" -Port 443 -Protocol "Https" -CookieBasedAffinity "Disabled" -Probe $apimprobe -AuthenticationCertificates $authcert -RequestTimeout 180
$apimPoolPortalSetting = New-AzApplicationGatewayBackendHttpSettings -Name "apimPoolPortalSetting" -Port 443 -Protocol "Https" -CookieBasedAffinity "Disabled" -Probe $apimPortalProbe -AuthenticationCertificates $authcert -RequestTimeout 180
```

### <a name="step-9"></a>Шаг 9.

Настройте пул IP-адресов серверной части с именем **apimbackend**, указав для него внутренний виртуальный IP-адрес, созданный ранее для службы управления API.

```powershell
$apimProxyBackendPool = New-AzApplicationGatewayBackendAddressPool -Name "apimbackend" -BackendIPAddresses $apimService.PrivateIPAddresses[0]
```

### <a name="step-10"></a>Шаг 10

Создайте правила, чтобы Шлюз приложений использовал базовую маршрутизацию.

```powershell
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "rule1" -RuleType Basic -HttpListener $listener -BackendAddressPool $apimProxyBackendPool -BackendHttpSettings $apimPoolSetting
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "rule2" -RuleType Basic -HttpListener $portalListener -BackendAddressPool $apimProxyBackendPool -BackendHttpSettings $apimPoolPortalSetting
```

> [!TIP]
> Измените -RuleType и маршрутизацию, чтобы ограничить доступ к определенным страницам портала разработчика.

### <a name="step-11"></a>Шаг 11

Настройте число экземпляров и размер шлюза приложений. Здесь мы используем [WAF SKU](../web-application-firewall/ag/ag-overview.md) для повышения уровня безопасности ресурса управления API.

```powershell
$sku = New-AzApplicationGatewaySku -Name "WAF_Medium" -Tier "WAF" -Capacity 2
```

### <a name="step-12"></a>Шаг 12

Настройте WAF в режиме предотвращения.

```powershell
$config = New-AzApplicationGatewayWebApplicationFirewallConfiguration -Enabled $true -FirewallMode "Prevention"
```

## <a name="create-application-gateway"></a>Создание шлюза приложений

Создайте шлюз приложений, используя все созданные ранее объекты конфигурации.

```powershell
$appgwName = "apim-app-gw"
$appgw = New-AzApplicationGateway -Name $appgwName -ResourceGroupName $resGroupName -Location $location -BackendAddressPools $apimProxyBackendPool -BackendHttpSettingsCollection $apimPoolSetting, $apimPoolPortalSetting  -FrontendIpConfigurations $fipconfig01 -GatewayIpConfigurations $gipconfig -FrontendPorts $fp01 -HttpListeners $listener, $portalListener -RequestRoutingRules $rule01, $rule02 -Sku $sku -WebApplicationFirewallConfig $config -SslCertificates $cert, $certPortal -AuthenticationCertificates $authcert -Probes $apimprobe, $apimPortalProbe
```

## <a name="cname-the-api-management-proxy-hostname-to-the-public-dns-name-of-the-application-gateway-resource"></a>Создание сопоставления CNAME для связи имени узла прокси-сервера управления API с общедоступным DNS-именем ресурса шлюза приложений

После создания шлюза следует настроить внешний интерфейс для обмена данными. Если вы используете общедоступный IP-адрес, шлюзу приложений требуется динамически назначаемое имя DNS, которое может быть неудобным для использования.

Для DNS-имени шлюза приложений следует создать запись CNAME, которая будет связывать имя узла прокси-сервера (в примере выше это `api.contoso.net`) с этим DNS-именем. Чтобы настроить запись CNAME интерфейсного IP-адреса, получите сведения о шлюзе приложений и соответствующее IP- или DNS-имя с помощью элемента PublicIPAddress. Мы не рекомендуем использовать записи типа A, так как виртуальный IP-адрес может измениться после перезапуска шлюза.

```powershell
Get-AzPublicIpAddress -ResourceGroupName $resGroupName -Name "publicIP01"
```

## <a name="summary"></a><a name="summary"></a> Сводка по
Служба управления API Azure, настроенная в виртуальной сети, предоставляет один интерфейс шлюза для всех настроенных интерфейсов API, размещенных в локальной среде или в облаке. Интеграция шлюза приложений с управлением API дает вам дополнительную гибкость, позволяя избирательно предоставлять доступ к API-интерфейсам через Интернет. Также вы можете использовать брандмауэр веб-приложения в качестве интерфейсного компонента для экземпляра службы управления API.

## <a name="next-steps"></a><a name="next-steps"></a> Дальнейшие действия
* Дополнительные сведения о шлюзе приложений Azure:
  * [Обзор шлюза приложений](../application-gateway/overview.md)
  * [Брандмауэр веб-приложения шлюза приложений](../web-application-firewall/ag/ag-overview.md)
  * [Создание шлюза приложений с помощью маршрутизации на основе пути](../application-gateway/tutorial-url-route-powershell.md)
* См. дополнительные сведения о службе управлении API и виртуальных сетях
  * [Использование службы управления API Azure совместно с внутренней виртуальной сетью](api-management-using-with-internal-vnet.md)
  * [How to use Azure API Management with virtual networks](api-management-using-with-vnet.md) (Использование управления API Azure в виртуальных сетях)
