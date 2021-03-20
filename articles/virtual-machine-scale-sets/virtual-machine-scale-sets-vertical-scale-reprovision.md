---
title: Вертикальное масштабирование масштабируемых наборов виртуальных машин Azure
description: Сведения о вертикальном масштабировании виртуальной машины в ответ на оповещения c помощью службы автоматизации Azure
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 04/18/2019
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: b172f1f7137b53e98384d92c9c709694eaf0b7e9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100594498"
---
# <a name="vertical-autoscale-with-virtual-machine-scale-sets"></a>Вертикальное автомасштабирование масштабируемых наборов виртуальных машин

В этой статье описывается, как выполнить вертикальное масштабирование [масштабируемых наборов виртуальных машин](https://azure.microsoft.com/services/virtual-machine-scale-sets/) Azure с повторной подготовкой или без нее. 

Вертикальное масштабирование — это изменение размеров виртуальной машины (*увеличение масштаба* или его *уменьшение*) в ответ на рабочую нагрузку. Сравните это с [горизонтальным масштабированием](virtual-machine-scale-sets-autoscale-overview.md) (также называется *горизонтальным увеличением масштаба* и *свертыванием* количества экземпляров), при котором в зависимости от рабочей нагрузки изменяется количество виртуальных машин.

Повторная подготовка означает удаление существующей виртуальной машины и ее замену новой. Когда вы увеличиваете или уменьшаете размер виртуальных машин в масштабируемом наборе, как правило, нужно либо изменить размер имеющихся виртуальных машин и сохранить данные, либо развернуть новые виртуальные машины другого размера. В этом документе описываются оба сценария.

Когда полезно выполнять вертикальное масштабирование:

* Если служба, работающая на базе виртуальных машин, мало используется (например, в выходные). Уменьшение размера виртуальной машины может сократить ежемесячные расходы.
* Чтобы увеличить размер виртуальной машины в соответствии с возросшими потребностями, не создавая дополнительные виртуальные машины.

Вы можете настроить активацию вертикального масштабирования с помощью оповещений на основе метрик масштабируемого набора виртуальных машин. Активированное оповещение запускает веб-перехватчик, который, в свою очередь, активирует модуль runbook. Этот модуль и изменяет размер масштабируемого набора. Вертикальное масштабирование можно настроить так:

1. Создайте учетную запись службы автоматизации Azure с возможностью запуска от имени.
2. Импортируйте в подписку модули runbook службы автоматизации Azure для масштабируемого набора виртуальных машин.
3. Добавьте в модуль Runbook веб-перехватчик.
4. Добавьте в свой масштабируемый набор виртуальных машин правило оповещения с использованием системы уведомлений веб-перехватчика.

> [!NOTE]
> Размеры, до которых можно масштабировать виртуальную машину, могут быть ограничены из-за размера первой виртуальной машины ввиду поддерживаемых размеров кластера, в котором развернута текущая виртуальная машина. В этой статье мы учли эту особенность и в опубликованных модулях Runbook службы автоматизации использовали для масштабирования только приведенные ниже примеры размеров виртуальных машин. Это означает, что нельзя внезапно увеличить виртуальную машину размером Standard_D1v2 до размера Standard_G5 или уменьшить до Basic_A0. Кроме того, размеры виртуальных машин с вертикальным увеличением/уменьшением масштаба не поддерживаются. Вы можете выбрать для масштабирования следующие пары размеров:
> 
> | Размер виртуальной машины. масштабирование элемента пары | Член |
> | --- | --- |
> | Basic_A0 |Basic_A4 |
> | Standard_A0 |Standard_A4 |
> | Standard_A5 |Standard_A7 |
> | Standard_A8 |Standard_A9 |
> | Standard_A10 |Standard_A11 |
> | Standard_A1_v2 |Standard_A8_v2 |
> | Standard_A2m_v2 |Standard_A8m_v2  |
> | Standard_B1s |Standard_B2s |
> | Standard_B1ms |Standard_B8ms |
> | Standard_D1 |Standard_D4 |
> | Standard_D11 |Standard_D14 |
> | Standard_DS1 |Standard_DS4 |
> | Standard_DS11 |Standard_DS14 |
> | Standard_D1_v2 |Standard_D5_v2 |
> | Standard_D11_v2 |Standard_D14_v2 |
> | Standard_DS1_v2 |Standard_DS5_v2 |
> | Standard_DS11_v2 |Standard_DS14_v2 |
> | Standard_D2_v3 |Standard_D64_v3 |
> | Standard_D2s_v3 |Standard_D64s_v3 |
> | Standard_DC2s |Standard_DC4s |
> | Standard_E2_v3 |Standard_E64_v3 |
> | Standard_E2s_v3 |Standard_E64s_v3 |
> | Standard_F1 |Standard_F16 |
> | Standard_F1s |Standard_F16s |
> | Standard_F2sv2 |Standard_F72sv2 |
> | Standard_G1 |Standard_G5 |
> | Standard_GS1 |Standard_GS5 |
> | Standard_H8 |Standard_H16 |
> | Standard_H8m |Standard_H16m |
> | Standard_L4s |Standard_L32s |
> | Standard_L8s_v2 |Standard_L80s_v2 |
> | Standard_M8ms  |Standard_M128ms |
> | Standard_M32ls  |Standard_M64ls |
> | Standard_M64s  |Standard_M128s |
> | Standard_M64  |Standard_M128 |
> | Standard_M64m  |Standard_M128m |
> | Standard_NC6 |Standard_NC24 |
> | Standard_NC6s_v2 |Standard_NC24s_v2 |
> | Standard_NC6s_v3 |Standard_NC24s_v3 |
> | Standard_ND6s |Standard_ND24s |
> | Standard_NV6 |Standard_NV24 |
> | Standard_NV6s_v2 |Standard_NV24s_v2 |
> | Standard_NV12s_v3 |Standard_NV48s_v3 |
> 

## <a name="create-an-azure-automation-account-with-run-as-capability"></a>Создание учетной записи службы автоматизации Azure с возможностью запуска от имени
Сначала необходимо создать учетную запись службы автоматизации Azure, в которой будут размещаться модули runbook, используемые в масштабируемых наборах виртуальных машин. Недавно в [службе автоматизации Azure](https://azure.microsoft.com/services/automation/) появился компонент "Учетная запись запуска от имени". Этот компонент позволяет настраивать субъект-службу для автоматического запуска модулей runbook от имени пользователя. Дополнительные сведения см. в разделе:

* [Проверка подлинности модулей Runbook в Azure с помощью учетной записи запуска от имени](../automation/manage-runas-account.md)

## <a name="import-azure-automation-vertical-scale-runbooks-into-your-subscription"></a>Импорт в подписку модулей Runbook службы автоматизации Azure для вертикального масштабирования

Модули runbook, необходимые для вертикального масштабирования масштабируемых наборов виртуальных машин, уже опубликованы в коллекции runbook службы автоматизации Azure. Вот как импортировать их в подписку:

* [Runbook и коллекции модулей для службы автоматизации Azure](../automation/automation-runbook-gallery.md)

Щелкните в меню модулей Runbook значок "Обзор коллекции".

![Модули Runbook, которые нужно импортировать][runbooks]

На рисунке ниже показаны модули Runbook, которые нужно импортировать. Выберите модуль Runbook в зависимости от того, какой вариант вертикального масштабирования вы хотите выполнить — с повторной подготовкой или без нее.

![Коллекция модулей Runbook][gallery]

## <a name="add-a-webhook-to-your-runbook"></a>Добавление веб-перехватчика в модуль Runbook.

После импорта модулей runbook в них необходимо добавить веб-перехватчик. Этого может потребовать система оповещений масштабируемого набора виртуальных машин. Дополнительные сведения о создании веб-перехватчика для модуля Runbook см. в этой статье:

* [Объекты Webhook в службе автоматизации Azure](../automation/automation-webhooks.md)

> [!NOTE]
> Прежде чем закрывать диалоговое окно, обязательно скопируйте универсальный код ресурса (URI) веб-перехватчика. Этот адрес понадобится в следующем разделе.
> 
> 

## <a name="add-an-alert-to-your-virtual-machine-scale-set"></a>Добавление правила оповещения в масштабируемый набор виртуальных машин

Ниже приведен сценарий PowerShell, с помощью которого вы можете добавить в масштабируемый набор виртуальных машин правило оповещения. Дополнительные сведения о получении имени метрики для активации оповещения см. в статье [Общие метрики автомасштабирования Azure Monitor](../azure-monitor/autoscale/autoscale-common-metrics.md).

```powershell
$actionEmail = New-AzAlertRuleEmail -CustomEmail user@contoso.com
$actionWebhook = New-AzAlertRuleWebhook -ServiceUri <uri-of-the-webhook>
$threshold = <value-of-the-threshold>
$rg = <resource-group-name>
$id = <resource-id-to-add-the-alert-to>
$location = <location-of-the-resource>
$alertName = <name-of-the-resource>
$metricName = <metric-to-fire-the-alert-on>
$timeWindow = <time-window-in-hh:mm:ss-format>
$condition = <condition-for-the-threshold> # Other valid values are LessThanOrEqual, GreaterThan, GreaterThanOrEqual
$description = <description-for-the-alert>

Add-AzMetricAlertRule  -Name  $alertName `
                            -Location  $location `
                            -ResourceGroup $rg `
                            -TargetResourceId $id `
                            -MetricName $metricName `
                            -Operator  $condition `
                            -Threshold $threshold `
                            -WindowSize  $timeWindow `
                            -TimeAggregationOperator Average `
                            -Actions $actionEmail, $actionWebhook `
                            -Description $description
```

> [!NOTE]
> Мы рекомендуем взвешенно подходить к определению временного окна для правила оповещения. Так вы сможете избежать слишком частой активации вертикального масштабирования, а также связанных с этой процедурой перерывов в работе службы. Начните с интервала минимум в 20–30 минут. Если перерывы в работе службы недопустимы, рассмотрите возможность горизонтального масштабирования.
> 
> 

Дополнительные сведения о создании правил оповещений см. в следующих статьях:

* [Примеры для Azure Monitor PowerShell](../azure-monitor/powershell-samples.md)
* [Примеры команд для кроссплатформенного интерфейса командной строки в Azure Monitor](../azure-monitor/cli-samples.md)

## <a name="summary"></a>Сводка

В этой статье описан простой способ вертикального масштабирования. Указанные блоки (работа с учетной записью службы автоматизации, модулями Runbook, веб-перехватчиками и оповещениями) позволяют включить в настраиваемый набор действий самые разные события.

[runbooks]: ./media/virtual-machine-scale-sets-vertical-scale-reprovision/runbooks.png
[gallery]: ./media/virtual-machine-scale-sets-vertical-scale-reprovision/runbooks-gallery.png
