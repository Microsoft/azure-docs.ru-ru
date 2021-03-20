---
title: 'Создание и экспорт сертификатов для "точка — сеть": Linux: CLI'
description: Создание самозаверяющего корневого сертификата, экспорт открытого ключа и создание сертификатов клиента с помощью Linux CLI и strongSwan.
titleSuffix: Azure VPN Gateway
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: alzam
ms.openlocfilehash: 0be5bb042649b0fe425077b5b3feb3cea728218c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89394012"
---
# <a name="generate-and-export-certificates"></a>Создание и экспорт сертификатов

Для аутентификации подключений типа "точка — сеть" используются сертификаты. Эта статья поможет создать самозаверяющий корневой сертификат, а также сертификаты клиента с помощью Linux CLI и strongSwan. Если вы ищете инструкции для других сертификатов, ознакомьтесь со статьями, посвященными [PowerShell](vpn-gateway-certificates-point-to-site.md) или [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md). Инструкции по установке strongSwan с помощью графического пользовательского интерфейса вместо интерфейса командной строки приведены в разделе [Установка и настройка](point-to-site-vpn-client-configuration-azure-cert.md#install).

## <a name="install-strongswan"></a>Установка strongSwan

[!INCLUDE [strongSwan Install](../../includes/vpn-gateway-strongswan-install-include.md)]

## <a name="generate-and-export-certificates"></a>Создание и экспорт сертификатов

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Продолжайте настраивать параметры конфигурации типа "точка — сеть", чтобы [создать и установить файлы конфигурации VPN-клиента](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli).
