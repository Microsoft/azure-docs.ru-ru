---
title: 'Устранение неполадок VPN-подключения "сеть — сеть" Azure, которое не может подключить titleSuffix: VPN-шлюз Azure'
description: Узнайте, как устранить проблемы подключения VPN типа "сеть — сеть", которое внезапно завершается сбоем и его невозможно восстановить.
services: vpn-gateway
author: chadmath
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 03/22/2021
ms.author: genli
ms.openlocfilehash: cebb05b35379573fc9797e89dee3c0c2bf3de6e2
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867293"
---
# <a name="troubleshooting-an-azure-site-to-site-vpn-connection-cannot-connect-and-stops-working"></a>Устранение проблемы подключения VPN типа "сеть — сеть" Azure

После настройки подключения VPN типа "сеть — сеть" между локальной сетью и виртуальной сетью Azure VPN-подключение внезапно завершается сбоем и его невозможно восстановить. В этой статье приведены действия по устранению этой проблемы. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок

Чтобы устранить эту проблему, попробуйте сначала [сбросить VPN-шлюз Azure](./reset-gateway.md), а также сбросить туннель в локальном VPN-устройстве. Если проблему устранить не удалось, выполните действия ниже, чтобы определить причину проблемы.

### <a name="prerequisite-step"></a>Предварительные действия

Проверьте тип VPN-шлюза Azure.

1. Перейдите на [портал Microsoft Azure](https://portal.azure.com).

2. Просмотрите тип шлюза на странице **обзора** VPN-шлюза.
    
    ![Обзор шлюза](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>Шаг 1. Проверка того, проверено ли локальное устройство VPN

1. Узнайте, используете ли вы [проверенную версию VPN-устройства и операционной системы](vpn-gateway-about-vpn-devices.md#devicetable). Если устройство является непроверенным VPN-устройством, может потребоваться обратиться к изготовителю устройства, чтобы узнать о возможных проблемах совместимости.

2. Убедитесь, что VPN-устройство настроено правильно. Дополнительные сведения см. на странице об [изменении примеров конфигурации устройств](vpn-gateway-about-vpn-devices.md#editing).

### <a name="step-2-verify-the-shared-key"></a>Шаг 2. Проверка общего ключа

Сравните общий ключ от локального VPN-устройства и VPN виртуальной сети Azure, чтобы убедиться, что они совпадают. 

Чтобы просмотреть общий ключ для подключения VPN Azure, используйте один из методов ниже.

**Портал Azure**

1. Перейдите к созданному вами подключению типа "сеть — сеть" через VPN-шлюз Azure.

2. В разделе **Параметры** щелкните **общий ключ**.
    
    ![Общий ключ](media/vpn-gateway-troubleshoot-site-to-site-cannot-connect/sharedkey.png)

**Azure PowerShell**

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Для модели развертывания с помощью Azure Resource Manager выполните следующую команду:

```azurepowershell
Get-AzVirtualNetworkGatewayConnectionSharedKey -Name <Connection name> -ResourceGroupName <Resource group name>
```

Выполните следующую команду для классической модели развертывания:

```azurepowershell
Get-AzureVNetGatewayKey -VNetName -LocalNetworkSiteName
```

### <a name="step-3-verify-the-vpn-peer-ips"></a>Шаг 3. Проверка IP-адресов VPN-узла

-   Определение IP-адреса в объекте **шлюза локальной сети** в Azure должно совпадать с IP-адресом локального устройства.
-   Определение IP-адреса шлюза Azure, установленное в локальном устройстве, должно совпадать с IP-адресом шлюза Azure.

### <a name="step-4-check-udr-and-nsgs-on-the-gateway-subnet"></a>Шаг 4. Проверка определяемого пользователем маршрута и групп безопасности сети в подсети шлюза

Проверьте, есть ли в подсети шлюза определяемые пользователем маршруты (UDR) или группы безопасности сети (NSG) и удалите их, если они имеются. Затем проверьте, успешно ли выполнена операция удаления. Если проблему удалось устранить, проверьте, соответствующим ли образом настроены параметры для NSG и UDR.

### <a name="step-5-check-the-on-premises-vpn-device-external-interface-address"></a>Шаг 5. Проверка внешнего адреса интерфейса для локального VPN-устройства

Если IP-адрес для Интернета VPN-устройства включен в определение **локальной сети** в Azure, вы можете столкнуться с нерегулярными отключениями.

### <a name="step-6-verify-that-the-subnets-match-exactly-azure-policy-based-gateways"></a>Шаг 6. Проверка полного соответствия подсетей (шлюзов на основе политик Azure)

-   Адресные пространства виртуальной сети между виртуальной сетью Azure и локальными определениями должны полностью совпадать.
-   Подсети между **шлюзом локальной сети** и локальными определениями должны полностью совпадать для локальной сети.

### <a name="step-7-verify-the-azure-gateway-health-probe"></a>Шаг 7. Проверка работоспособности шлюза Azure

1. Перейдя по указанному URL-адресу, вы откроете проверку работоспособности.

    `https://<YourVirtualNetworkGatewayIP>:8081/healthprobe`

2. Просмотрите предупреждение о сертификате.
3. Если вы получите ответ, шлюз VPN считается работоспособным. Если вы не получите ответ, шлюз может быть неработоспособным, или проблема может быть вызвана наличием группы безопасности сети (NSG) в подсети шлюза. Пример ответа приведен ниже:

    ```xml
    <?xml version="1.0"?>
    <string xmlns="http://schemas.microsoft.com/2003/10/Serialization/">Primary Instance: GatewayTenantWorker_IN_1 GatewayTenantVersion: 14.7.24.6</string>
    ```

### <a name="step-8-check-whether-the-on-premises-vpn-device-has-the-perfect-forward-secrecy-feature-enabled"></a>Шаг 8. Проверка, включена ли для локального VPN-устройства функция полной безопасности пересылки

Функция полной безопасности пересылки может вызвать проблемы отключения. Если для VPN-устройства включена функция полной безопасности пересылки, отключите эту функцию. Затем обновите политику IPsec VPN-шлюза.

## <a name="next-steps"></a>Дальнейшие действия

-   [Настройка подключения типа "сеть — сеть" к виртуальной сети](./tutorial-site-to-site-portal.md)
-   [Настройка политики IPsec/IKE для VPN-подключений типа "сеть — сеть" или "виртуальная сеть — виртуальная сеть"](vpn-gateway-ipsecikepolicy-rm-powershell.md)