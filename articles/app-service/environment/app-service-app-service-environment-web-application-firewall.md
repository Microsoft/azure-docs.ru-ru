---
title: Настройка WAF
description: Узнайте, как настроить брандмауэр веб-приложения (WAF) перед Средой службы приложений с использованием Шлюза приложений Azure или WAF стороннего производителя.
author: ccompy
ms.assetid: a2101291-83ba-4169-98a2-2c0ed9a65e8d
ms.topic: tutorial
ms.date: 03/03/2018
ms.author: stefsch
ms.custom: mvc, seodec18
ms.openlocfilehash: 56d931f2346e5a0b615d3f11dce3b06396e586b4
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588723"
---
# <a name="configuring-a-web-application-firewall-waf-for-app-service-environment"></a>Настройка брандмауэра веб-приложения (WAF) для среды службы приложений
## <a name="overview"></a>Обзор

Брандмауэры веб-приложения (WAF) защищают веб-приложения, проверяя входящий трафик Интернета и блокируя атаки с использованием кода SQL и межсайтовых скриптов, передачи вредоносных программ, атаки DDoS на приложения и другие атаки. Они также проверяют ответы от внутренних веб-серверов, предотвращая потерю данных (DLP). В сочетании с изоляцией и дополнительным масштабированием, предоставляемым средами службы приложений, это обеспечивает идеальную среду для размещения важных коммерческих веб-приложений с большим трафиком, способных противостоять вредоносным запросам. Azure предоставляет возможности WAF со [шлюзом приложений](../../application-gateway/overview.md).  См. документацию по [интеграции ILB ASE со шлюзом приложений](./integrate-with-application-gateway.md).

Помимо Шлюза приложений Azure, в [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/barracudanetworks.waf?tab=PlansAndPrice) доступны такие возможности, как [Barracuda WAF для Azure](https://www.barracuda.com/programs/azure). В остальной части этого документа описано, как интегрировать среду службы приложений с устройством Barracuda WAF.

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)] 

## <a name="setup"></a>Настройка
Мы настроим среду службы приложений за несколькими экземплярами Barracuda WAF с балансировкой нагрузки, чтобы только трафик из WAF мог достичь среды службы приложений. Таким образом эта среда не будет доступна из сети периметра. Мы также настроим диспетчер трафика Azure перед экземплярами Barracuda WAF, чтобы распределить нагрузку между несколькими центрами обработки данных и регионами Azure. Общая схема конфигурации показана на рисунке ниже.

![Схема, на которой показан необязательный диспетчер трафика Azure, который подключается к экземплярам брандмауэра веб-приложения, при этом списки контроля доступа разрешают только передачу трафика из брандмауэра в среду службы приложений, которая содержит веб-приложение, API и мобильное приложение для двух регионов.][Architecture] 

> [!NOTE]
> С появлением [поддержки внутренней подсистемы балансировки нагрузки в среде службы приложений](app-service-environment-with-internal-load-balancer.md) вы можете закрыть доступ к среде службы приложений из сети периметра, чтобы она была доступна только для частной сети. 
> 
> 

## <a name="configuring-your-app-service-environment"></a>Настройка среды службы приложений
Инструкции по настройке среды службы приложений см. в соответствующей [документации](app-service-web-how-to-create-an-app-service-environment.md). После создания среды службы приложений в ней можно создавать веб-приложения, приложения API и [мобильные приложения](/previous-versions/azure/app-service-mobile/app-service-mobile-value-prop), которые будут защищены брандмауэром веб-приложения (WAF), который мы настроим в следующем разделе.

## <a name="configuring-your-barracuda-waf-cloud-service"></a>Настройка облачной службы Barracuda WAF
У нас есть [подробная статья](https://campus.barracuda.com/product/webapplicationfirewall/article/WAF/DeployWAFInAzure) о развертывании Barracuda WAF на виртуальной машине в Azure. Так как нам требуется отказоустойчивость и отсутствие единой точки отказа, необходимо развернуть по крайней мере 2 экземпляра виртуальной машины WAF в той же облачной службе, следуя этим инструкциям.

### <a name="adding-endpoints-to-cloud-service"></a>Добавление конечных точек в облачной службе
После создания в облачной службе двух и более экземпляров виртуальной машины WAF используйте [классический портал Azure](https://portal.azure.com/), чтобы добавить конечные точки HTTP и HTTPS, которые будут использоваться вашим приложением, как показано на рисунке ниже.

![Настройка конечной точки][ConfigureEndpoint]

Если в приложениях используются другие конечные точки, их также необходимо добавить в этот список. 

### <a name="configuring-barracuda-waf-through-its-management-portal"></a>Настройка Barracuda WAF через портал управления
Barracuda WAF использует TCP-порт 8000 для настройки с помощью портала управления. Если имеется несколько экземпляров виртуальной машины WAF, необходимо повторить приведенные здесь шаги для каждого из них. 

> [!NOTE]
> После завершения настройки WAF удалите конечную точку TCP/8000 из всех виртуальных машин WAF, чтобы обеспечить безопасность WAF.
> 
> 

Добавьте конечную точку управления, как показано на рисунке ниже, для настройки Barracuda WAF.

![Добавление конечной точки управления][AddManagementEndpoint]

С помощью браузера перейдите к конечной точке управления в своей облачной службе. Если имя вашей облачной службы — test.cloudapp.net, доступ к этой конечной точке можно получить по адресу `http://test.cloudapp.net:8000`. Отобразится страница входа (как на рисунке ниже), на которой можно выполнить вход с учетными данными, которые вы указали на этапе установки виртуальной машины WAF.

![Страница управления входом][ManagementLoginPage]

После входа отобразится панель мониторинга (как на рисунке ниже) с основной статистикой защиты WAF.

![Панель управления][ManagementDashboard]

На вкладке **Службы** можно настроить WAF для защиты служб. Дополнительные сведения о настройке Barracuda WAF можно найти в соответствующей [документации](https://campus.barracuda.com/product/webapplicationfirewall/doc/4259884/configure-the-barracuda-web-application-firewall-from-the-web-interface/). В примере ниже приведена настройка приложения Службы приложений, обслуживающего трафик HTTP и HTTPS.

![Управление: добавление служб][ManagementAddServices]

> [!NOTE]
> В зависимости от настроек приложений и функций, используемых в Среде службы приложений, будет необходимо перенаправлять трафик для TCP-портов, отличных от 80 и 443, например, при использовании TLS на основе IP для приложения Службы приложений. Список сетевых портов, используемых в средах службы приложений, см. в разделе "Входящие сетевые порты, используемые в среде службы приложений" [документации по управлению входящим трафиком](app-service-app-service-environment-control-inbound-traffic.md).
> 
> 

## <a name="configuring-microsoft-azure-traffic-manager-optional"></a>Настройка диспетчера трафика Microsoft Azure (необязательно)
Если приложение доступно в нескольких регионах, необходимо сбалансировать нагрузку с помощью [диспетчера трафика Azure](../../traffic-manager/traffic-manager-overview.md). Для этого можно добавить конечную точку на [классическом портале Azure](https://portal.azure.com), указав имя облачной службы для WAF в профиле диспетчера трафика, как показано на рисунке ниже. 

![Конечная точка диспетчера трафика][TrafficManagerEndpoint]

Если приложение требует проверки подлинности, убедитесь, что у вас есть ресурсы для проверки доступности приложения, которые не требуют проверки подлинности через диспетчер трафика. URL-адрес можно настроить на странице **Конфигурация** на [портале Azure](https://portal.azure.com), как показано на следующем рисунке.

![Настройка диспетчера трафика][ConfigureTrafficManager]

Чтобы перенаправлять проверки связи диспетчера трафика из WAF в приложение, необходимо установить Website Translations на Barracuda WAF для перенаправления трафика к приложению, как показано в следующем примере.

![Переводы веб-сайта][WebsiteTranslations]

## <a name="securing-traffic-to-app-service-environment-using-network-security-groups-nsg"></a>Защита трафика в среде службы приложений с помощью групп безопасности сети (NSG)
Дополнительные сведения об ограничении трафика в среду службы приложений только от WAF с помощью виртуального IP-адреса облачной службы см. в [документации по управлению входящим трафиком](app-service-app-service-environment-control-inbound-traffic.md). Ниже приведен пример команды PowerShell для выполнения этой задачи для TCP-порта 80.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "RestrictWestUSAppAccess" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP Barracuda" -Type Inbound -Priority 201 -Action Allow -SourceAddressPrefix '191.0.0.1'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
```

Замените SourceAddressPrefix на виртуальный IP-адрес (VIP) облачной службы WAF.

> [!NOTE]
> Виртуальный IP-адрес облачной службы изменится, если удалить и повторно создать эту облачную службу. Не забудьте после этого обновить IP-адрес в группе ресурсов сети. 
> 
> 

<!-- IMAGES -->
[Architecture]: ./media/app-service-app-service-environment-web-application-firewall/Architecture.png
[ConfigureEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/ConfigureEndpoint.png
[AddManagementEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/AddManagementEndpoint.png
[ManagementAddServices]: ./media/app-service-app-service-environment-web-application-firewall/ManagementAddServices.png
[ManagementDashboard]: ./media/app-service-app-service-environment-web-application-firewall/ManagementDashboard.png
[ManagementLoginPage]: ./media/app-service-app-service-environment-web-application-firewall/ManagementLoginPage.png
[TrafficManagerEndpoint]: ./media/app-service-app-service-environment-web-application-firewall/TrafficManagerEndpoint.png
[ConfigureTrafficManager]: ./media/app-service-app-service-environment-web-application-firewall/ConfigureTrafficManager.png
[WebsiteTranslations]: ./media/app-service-app-service-environment-web-application-firewall/WebsiteTranslations.png
