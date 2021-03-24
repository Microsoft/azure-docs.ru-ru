---
title: Добавление приложения в кластере с помощью PowerShell
description: Пример сценария Azure PowerShell — развертывание приложения из кластера Service Fabric.
services: service-fabric
documentationcenter: ''
author: athinanthny
manager: chackdan
editor: ''
tags: azure-service-management
ms.assetid: ''
ms.service: service-fabric
ms.workload: multiple
ms.topic: sample
ms.date: 01/18/2018
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: 1d25ede5ae871eddd965594224b518ec42525451
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791295"
---
# <a name="deploy-an-application-to-a-service-fabric-cluster"></a>Развертывание приложения в кластера Service Fabric

Этот пример сценария копирует пакет приложения в хранилище образов кластера, регистрирует тип приложения в кластере, удаляет ненужный пакет приложения и создает экземпляр приложения с типом приложения.  Если в манифесте приложения для целевого типа приложения были определены используемые по умолчанию службы, то они также будут созданы. Измените параметры, если это необходимо. 

При необходимости установите модуль PowerShell ServiceFabric вместе с [пакетом SDK для Service Fabric](../service-fabric-get-started.md). 

## <a name="sample-script"></a>Пример скрипта

[!code-powershell[main](../../../powershell_scripts/service-fabric/deploy-application/deploy-application.ps1 "Deploy an application to a cluster")]

## <a name="clean-up-deployment"></a>Очистка развертывания 

После запуска примера сценария можно использовать сценарий из раздела [Удаление приложения](service-fabric-powershell-remove-application.md) для удаления экземпляра приложения, отмены регистрация типа приложения и удаления пакета приложения из хранилища образов.

## <a name="script-explanation"></a>Описание скрипта

Этот скрипт использует следующие команды. Для каждой команды в таблице приведены ссылки на соответствующую документацию.

| Get-Help | Примечания |
|---|---|
|[Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster)| Создает подключение к кластеру Service Fabric. |
|[Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) | Копирует пакет приложения в хранилище образов кластера.  |
|[Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype)| Регистрирует тип и версию приложения в кластере. |
|[New-ServiceFabricApplication](/powershell/module/servicefabric/new-servicefabricapplication)| Создает приложение на основе зарегистрированного типа приложения. |
| [Remove-ServiceFabricApplicationPackage](/powershell/module/servicefabric/remove-servicefabricapplicationpackage) | Удаляет пакет приложения Service Fabric из хранилища образов.|

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Service Fabric PowerShell см. в [документации по Azure PowerShell](/powershell/azure/service-fabric/overview).

Дополнительные примеры сценариев PowerShell для Azure Service Fabric см. в разделе [Примеры сценариев Azure PowerShell](../service-fabric-powershell-samples.md).
