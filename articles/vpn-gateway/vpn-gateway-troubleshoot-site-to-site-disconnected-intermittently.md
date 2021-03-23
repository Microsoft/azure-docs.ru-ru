---
title: Временное устранение неполадок при подключении VPN типа "сеть — сеть" Azure
description: Сведения об устранении регулярных разрывов подключений VPN типа "сеть — сеть".
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: chadmath
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 03/22/2021
ms.author: genli
ms.openlocfilehash: 38846bbe717912092ccfe2b236b717770b79302f
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867259"
---
# <a name="troubleshooting-azure-site-to-site-vpn-disconnects-intermittently"></a>Устранение проблемы периодических разрывов подключений VPN типа "сеть — сеть" Azure

У вас может возникнуть проблема: новое или существующее VPN-подключение типа "сеть — сеть" Microsoft Azure может быть нестабильным или регулярно отключаться. В этой статье приведены действия, которые помогут определить и устранить причину проблемы. 

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок

### <a name="prerequisite-step"></a>Предварительные действия

Проверьте тип шлюза виртуальной сети Azure.

1. Перейдите на [портал Azure](https://portal.azure.com).
2. Ознакомьтесь со страницей **обзора** шлюза виртуальной сети, чтобы получить сведения о типе шлюза.
    
    ![Обзор шлюза](media/vpn-gateway-troubleshoot-site-to-site-disconnected-intermittently/gatewayoverview.png)

### <a name="step-1-check-whether-the-on-premises-vpn-device-is-validated"></a>Шаг 1. Узнайте, проверено ли локальное устройство VPN

1. Узнайте, используете ли вы [проверенную версию VPN-устройства и операционной системы](vpn-gateway-about-vpn-devices.md#devicetable). Если VPN-устройство непроверенное, может потребоваться обратиться к изготовителю устройства, чтобы узнать о возможных проблемах совместимости.
2. Убедитесь, что VPN-устройство настроено правильно. Дополнительные сведения см. на странице об [изменении примеров конфигурации устройств](vpn-gateway-about-vpn-devices.md#editing).

### <a name="step-2-check-the-security-association-settingsfor-policy-based-azure-virtual-network-gateways"></a>Шаг 2. Проверьте параметры сопоставления безопасности (для шлюзов виртуальной сети Azure на основе политик)

1. Убедитесь, что виртуальная сеть, подсети и диапазоны в определении **шлюза локальной сети** в Microsoft Azure соответствуют конфигурации локального VPN-устройства.
2. Убедитесь, что параметры сопоставления безопасности совпадают.

### <a name="step-3-check-for-user-defined-routes-or-network-security-groups-on-gateway-subnet"></a>Шаг 3. Проверьте определяемые пользователем маршруты или группы безопасности сети в подсети шлюза

Определяемый пользователем маршрут в подсети шлюза может ограничивать часть трафика и разрешать другой трафик. Таким образом VPN-подключение является ненадежным для одной части трафика и хорошим для другой. 

### <a name="step-4-check-the-one-vpn-tunnel-per-subnet-pair-setting-for-policy-based-virtual-network-gateways"></a>Шаг 4. Проверьте параметр One VPN Tunnel per Subnet Pair (Один VPN-туннель на пару подсетей) (для шлюзов виртуальной сети на основе политик)

Убедитесь, что для локального VPN-устройства задан параметр **One VPN tunnel per subnet pair** (Один VPN-туннель на пару подсетей) для шлюзов виртуальной сети на основе политик.

### <a name="step-5-check-for-security-association-limitation-for-policy-based-virtual-network-gateways"></a>Шаг 5. Проверьте ограничения сопоставления безопасности (для шлюзов виртуальной сети на основе политик)

Шлюз виртуальной сети на основе политик имеет ограничение на сопоставления безопасности подсети в 200 пар. Если количество подсетей виртуальной сети Azure, умноженное на количество локальных подсетей, превышает 200, могут случайно отключаться подсети.

### <a name="step-6-check-on-premises-vpn-device-external-interface-address"></a>Шаг 6. Проверьте внешний адрес интерфейса для локального VPN-устройства

Если IP-адрес для Интернета VPN-устройства включен в определение **шлюза локальной сети** в Azure, вы можете столкнуться с нерегулярными отключениями.

### <a name="step-7-check-whether-the-on-premises-vpn-device-has-perfect-forward-secrecy-enabled"></a>Шаг 7. Проверьте, включена ли для локального VPN-устройства функция полной безопасности пересылки

Функция **полной безопасности пересылки** (PFS) может вызвать проблемы отключения. Если для VPN-устройства **включена полная PFS,** отключите ее. Затем [Обновите политику IPSec шлюза виртуальной сети](vpn-gateway-ipsecikepolicy-rm-powershell.md#managepolicy).

## <a name="next-steps"></a>Дальнейшие действия

- [Создание подключения типа "сеть — сеть" на портале Azure](./tutorial-site-to-site-portal.md)
- [Настройка политики IPsec/IKE для VPN-подключений типа "сеть — сеть"](vpn-gateway-ipsecikepolicy-rm-powershell.md)
