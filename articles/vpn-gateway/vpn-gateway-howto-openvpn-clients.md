---
title: Как настроить клиенты OpenVPN для VPN-шлюза Azure | Документация Майкрософт
description: Узнайте, как настроить Опенвпн для VPN-шлюза Azure для клиентов операционной системы Windows, Linux и Mac.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: 4be903fa17ce95e96c82241249b421e1d794c80f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89435785"
---
# <a name="configure-openvpn-clients-for-azure-vpn-gateway"></a>Настройка клиентов OpenVPN для VPN-шлюза Azure

Эта статья поможет вам настроить **клиенты &reg; протокола опенвпн** .

## <a name="before-you-begin"></a>Перед началом

Убедитесь, что шаги по настройке OpenVPN для вашего VPN-шлюза выполнены. Дополнительные сведения см. в разделе [Настройка OpenVPN для VPN-шлюза Azure](vpn-gateway-howto-openvpn.md).

[!INCLUDE [configuration steps](../../includes/vpn-gateway-vwan-config-openvpn-clients.md)]

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите, чтобы VPN-клиенты могли получать доступ к ресурсам в другой виртуальной сети, следуйте инструкциям в статье " [Виртуальная](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) сеть-vnet", чтобы настроить подключение между виртуальными сетями. Обязательно включите протокол BGP на шлюзах и соединениях, иначе трафик не будет считываться.

**"Опенвпн" является товарным знаком Опенвпн Inc.**
