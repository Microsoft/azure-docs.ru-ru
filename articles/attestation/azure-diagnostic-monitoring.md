---
title: Мониторинг диагностики Azure для Аттестации Azure
description: Мониторинг диагностики Azure для Аттестации Azure
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.openlocfilehash: d2773be4bc67e125c18d5d38c951685e4f4fceaf
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106168354"
---
# <a name="set-up-diagnostics-with-a-trusted-platform-module-tpm-endpoint-of-azure-attestation"></a>Настройка диагностики с использованием конечной точки доверенного платформенного модуля Аттестации Azure

В этой статье рассмотрена процедура создания и настройки параметров диагностики для отправки метрик и журналов платформы в разные назначения. [Журналы платформы](/azure/azure-monitor/platform/platform-logs-overview) в Azure, включая журнал действий Azure и журналы ресурсов, содержат подробные сведения о диагностике и аудите для ресурсов Azure и платформы Azure, от которых они зависят. [Метрики платформы](/azure/azure-monitor/platform/data-platform-metrics) собираются по умолчанию и хранятся в базе данных метрик Azure Monitor.

Прежде чем начать, убедитесь, что вы [настроили аттестацию Azure с помощью Azure PowerShell](quickstart-powershell.md).

Служба конечной точки доверенного платформенного модуля (TPM) включена в параметрах диагностики и может использоваться для наблюдения за действиями. Настройте [мониторинг Azure](/azure/azure-monitor/overview) для конечной точки службы TPM, используя следующий код.

```powershell

 Connect-AzAccount 

 Set-AzContext -Subscription <Subscription id> 

 $attestationProviderName=<Name of the attestation provider> 

 $attestationResourceGroup=<Name of the resource Group> 

 $attestationProvider=Get-AzAttestation -Name $attestationProviderName -ResourceGroupName $attestationResourceGroup 

 $storageAccount=New-AzStorageAccount -ResourceGroupName $attestationProvider.ResourceGroupName -Name <Storage Account Name> -SkuName Standard_LRS -Location <Location> 

 Set-AzDiagnosticSetting -ResourceId $ attestationProvider.Id -StorageAccountId $ storageAccount.Id -Enabled $true 

```

Журналы действий находятся в разделе **Контейнеры** учетной записи хранения. Дополнительные сведения см. в разделе [Сбор и анализ журналов ресурсов из ресурса Azure](/azure/azure-monitor/learn/tutorial-resource-logs).
