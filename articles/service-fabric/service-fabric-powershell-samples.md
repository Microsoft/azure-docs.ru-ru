---
title: Примеры сценариев Azure PowerShell — Service Fabric
description: Узнайте о создании кластеров Azure Service Fabric, приложений и служб, а также управлении ими с помощью Powershell.
ms.topic: sample
ms.date: 11/29/2018
ms.custom: mvc
ms.openlocfilehash: 4b85fd604eb27f0963af882b41e823d800005dda
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86187104"
---
# <a name="azure-service-fabric-powershell-samples"></a>Примеры PowerShell Service Fabric

Ниже приведена таблица с ссылками на примеры сценариев PowerShell, которые позволяют создавать кластеры, приложения и службы Service Fabric и управлять ими.

[!INCLUDE [links to azure CLI and service fabric CLI](../../includes/service-fabric-powershell.md)]

| Скрипт | Описание |
|-|-|
| **Создать кластер** ||
| [Создание кластера (Azure)](./scripts/service-fabric-powershell-create-secure-cluster-cert.md)| Создает кластер Azure Service Fabric. |
| **Управление кластером, узлами и инфраструктурой** ||
| [Добавление сертификата приложения](./scripts/service-fabric-powershell-add-application-certificate.md)| Создает сертификат X509 для Key Vault и развертывает его в масштабируемом наборе виртуальных машин в кластере. |
| [Обновление диапазона портов RDP на виртуальных машинах кластера](./scripts/service-fabric-powershell-change-rdp-port-range.md)|Изменяет диапазон портов RDP на виртуальных машинах узла кластера в развернутом кластере.|
| [Обновление имени пользователя и пароля администратора для виртуальных машин узла кластера](./scripts/service-fabric-powershell-change-rdp-user-and-pw.md) | Обновляет имя пользователя и пароль администратора для виртуальных машин узла кластера. |
| [Открытие порта в подсистеме балансировки нагрузки](./scripts/service-fabric-powershell-open-port-in-load-balancer.md) | Открывает порт приложения в Azure Load Balancer, чтобы разрешить входящий трафик через определенный порт. |
| [Создание правила входящего трафика для группы безопасности сети](./scripts/service-fabric-powershell-add-nsg-rule.md) | Создает правило входящего трафика для группы безопасности сети, чтобы разрешить передачу входящего трафика в кластер через определенный порт. |
| **Управление приложениями** ||
| [Развертывание приложения](./scripts/service-fabric-powershell-deploy-application.md)| Развертывание приложения в кластере.|
| [Обновление приложения](./scripts/service-fabric-powershell-upgrade-application.md)| Обновляет приложение.|
| [Удаление приложения](./scripts/service-fabric-powershell-remove-application.md)| Удаление приложения из кластера.|
