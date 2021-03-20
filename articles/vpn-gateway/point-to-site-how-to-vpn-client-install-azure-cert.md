---
title: VPN-шлюз Azure. Установка сертификата клиента "точка — сеть"
description: Установка сертификата клиента для аутентификации на основе сертификата для подключения "точка — сеть" (Windows, Mac, Linux).
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 8c4d37f67d89ad1b7973a85fb6c2a9f7e818a028
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89421668"
---
# <a name="install-client-certificates-for-p2s-certificate-authentication-connections"></a>Установка сертификатов клиента для аутентификации Azure на основе сертификата для подключения "точка — сеть"

Для настройки подключений типа "точка — сеть" между клиентами и виртуальной сетью с использованием аутентификации Azure на основе сертификата требуется сертификат клиента. В этой статье описано, как установить сертификат клиента для аутентификации при подключении типа "точка — сеть" к виртуальной сети.

## <a name="acquire-a-client-certificate"></a><a name="generate"></a>Получение сертификата клиента

Независимо от того, из какой клиентской операционной системы вам нужно установить подключение, обязательно требуется сертификат клиента. Вы можете создать сертификат клиента либо на основе корневого сертификата, созданного с помощью корпоративного ЦС, либо на основе самозаверяющего сертификата. Ознакомьтесь с инструкциями по созданию сертификата клиента для [PowerShell](vpn-gateway-certificates-point-to-site.md), [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) или [Linux](vpn-gateway-certificates-point-to-site-linux.md). 

## <a name="windows"></a><a name="installwin"></a>Windows

[!INCLUDE [Install on Windows](../../includes/vpn-gateway-certificates-install-client-cert-include.md)]

## <a name="mac"></a><a name="installmac"></a>Адрес

>[!NOTE]
>VPN-клиенты Mac поддерживают только модель развертывания на основе Resource Manager. Классическую модель использовать нельзя.
>
>

[!INCLUDE [Install on Mac](../../includes/vpn-gateway-certificates-install-mac-client-cert-include.md)]

## <a name="linux"></a><a name="installlinux"></a>Linux

Сертификат клиента для Linux устанавливается в клиенте в рамках настройки. Ознакомьтесь с инструкциями по [настройке клиента для Linux](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli).

## <a name="next-steps"></a>Дальнейшие действия

Продолжайте настраивать параметры конфигурации "точка — сеть", чтобы [создать и установить файлы конфигурации VPN-клиента](point-to-site-vpn-client-configuration-azure-cert.md).
