---
title: 'Мониторинг диагностики Azure: Аттестация Azure'
description: Мониторинг диагностики Azure для Аттестации Azure
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: d01e7817906927295591353b710afe2899aacdf1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101726484"
---
# <a name="setting-up-diagnostics-with-trusted-platform-module-tpm-endpoint-of-azure-attestation"></a>Настройка диагностики с использованием конечной точки доверенного платформенного модуля Аттестации Azure

[Журналы платформы](../azure-monitor/essentials/platform-logs-overview.md) в Azure, в том числе журнал действий Azure и журналы ресурсов, предоставляют подробные сведения о диагностике и аудите для ресурсов Azure и платформы Azure, от которых они зависят. [Метрики платформы](../azure-monitor/essentials/data-platform-metrics.md) собираются по умолчанию и обычно хранятся в базе данных метрик Azure Monitor. В этой статье приведены сведения о создании и настройке параметров диагностики для отправки метрик и журналов платформы в разные назначения. 

Служба конечной точки доверенного платформенного модуля поддерживает параметр диагностики и позволяет выполнять мониторинг действий. Чтобы настроить [мониторинг Azure](../azure-monitor/overview.md) для конечной точки службы доверенного платформенного модуля с помощью PowerShell, выполните указанные ниже действия. 

Настройте службу "Аттестация Azure". 

[Настройка службы "Аттестация Azure" с помощью Azure PowerShell](./quickstart-powershell.md)

```powershell

 Connect-AzAccount 

 Set-AzContext -Subscription <Subscription id> 

 $attestationProviderName=<Name of the attestation provider> 

 $attestationResourceGroup=<Name of the resource Group> 

 $attestationProvider=Get-AzAttestation -Name $attestationProviderName -ResourceGroupName $attestationResourceGroup 

 $storageAccount=New-AzStorageAccount -ResourceGroupName $attestationProvider.ResourceGroupName -Name <Storage Account Name> -SkuName Standard_LRS -Location <Location> 

 Set-AzDiagnosticSetting -ResourceId $ attestationProvider.Id -StorageAccountId $ storageAccount.Id -Enabled $true 

```
Журналы действий можно найти в разделе "Контейнеры" учетной записи хранения. Подробные сведения можно найти в статье о [сборе журналов ресурсов из ресурса Azure и их анализе в Azure Monitor](../azure-monitor/essentials/tutorial-resource-logs.md).
