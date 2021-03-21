---
title: Мониторинг производительности на виртуальных машинах Azure с использованием Azure Application Insights
description: Мониторинг производительности приложений для ВИРТУАЛЬНОЙ машины Azure и масштабируемых наборов виртуальных машин Azure. Загрузка диаграммы и время отклика, сведения о зависимостях и Настройка оповещений о производительности.
ms.topic: conceptual
ms.date: 08/26/2019
ms.openlocfilehash: 0951d1d622f59de4780735fad78ac73649ea2369
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711487"
---
# <a name="deploy-the-azure-monitor-application-insights-agent-on-azure-virtual-machines-and-azure-virtual-machine-scale-sets"></a>Развертывание агента Application Insights службы Azure Monitor на виртуальных машинах Azure и в их масштабируемых наборах

Включение мониторинга для веб-приложений на основе .NET или Java, работающих на [виртуальных машинах Azure](https://azure.microsoft.com/services/virtual-machines/) и [масштабируемых наборах виртуальных машин Azure](../../virtual-machine-scale-sets/index.yml) , теперь стало проще, чем когда бы то ни было. Получите все преимущества использования Application Insights без изменения кода.

В этой статье описывается включение мониторинга Application Insights с помощью агента Application Insights и предоставляются предварительные рекомендации по автоматизации процесса для крупномасштабных развертываний.
> [!IMPORTANT]
> Приложения на основе **Java** , работающие на виртуальных машинах Azure и VMSS, отслеживаются с помощью **[Application Insights агента Java 3,0](./java-in-process-agent.md)**, который является общедоступным.

> [!IMPORTANT]
> Агент Application Insights Azure для приложений ASP.NET, работающих на **виртуальных машинах Azure и VMSS** , в настоящее время находится в общедоступной предварительной версии. Для мониторинга приложений ASP.Net, работающих **в локальной среде**, используйте [Агент Azure Application Insights для локальных серверов](./status-monitor-v2-overview.md), который является общедоступным и полностью поддерживаемым.
> Предварительная версия для виртуальных машин Azure и VMSS предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок в рабочей среде. Некоторые функции могут не поддерживаться, а некоторые могут иметь ограниченные возможности.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="enable-application-insights"></a>Включение Application Insights

Существует два способа включения мониторинга приложений для виртуальных машин Azure и масштабируемых наборов виртуальных машин Azure, размещенных в приложениях:

### <a name="auto-instrumentation-via-application-insights-agent"></a>Автоматическое инструментирование с помощью агента Application Insights

* Этот метод является самым простым для включения и не требует дополнительной настройки. Часто это называется мониторингом среды выполнения.

* Для виртуальных машин Azure и масштабируемых наборов виртуальных машин Azure рекомендуется как минимум включить этот уровень мониторинга. После этого в зависимости от конкретного сценария можно оценить, требуется ли ручное инструментирование.

> [!NOTE]
> Автоматическое Инструментирование в настоящее время доступно только для приложений, размещенных в среде .NET и Java. Используйте пакет SDK для инструментирования приложений ASP.NET Core, Node.js и Python, размещенных на виртуальных машинах Azure и масштабируемых наборах виртуальных машин.


#### <a name="net"></a>.NET

  * Агент Application Insights выполняет автоматическую сбор одинаковых зависимых сигналов в виде пакета SDK для .NET. Дополнительные сведения см. в разделе [Автоматическая коллекция зависимостей](./auto-collect-dependencies.md#net) .
        
#### <a name="java"></a>Java
  * Для Java предпочтительным подходом является **[Application Insights агента java 3,0](./java-in-process-agent.md)** . Самые популярные библиотеки и платформы, а также журналы и зависимости [собираются](./java-in-process-agent.md#auto-collected-requests-dependencies-logs-and-metrics)с множеством [дополнительных конфигураций](./java-standalone-config.md) .

### <a name="code-based-via-sdk"></a>Пакет SDK на основе кода
    
#### <a name="net"></a>.NET
  * Для приложений .NET этот подход гораздо более настраиваемый, но он требует [добавления зависимости от Application Insights пакетов NUGET SDK](./asp-net.md). Этот метод также означает, что вам нужно самостоятельно управлять обновлениями для последних версий пакетов.

  * Если необходимо выполнить пользовательские вызовы API для отслеживания событий или зависимостей, не отслеживаемых по умолчанию с помощью мониторинга на основе агентов, необходимо использовать этот метод. Дополнительные сведения см. в [статье об API для получения пользовательских событий и метрик](./api-custom-events-metrics.md) .

    > [!NOTE]
    > Только для приложений .NET — при обнаружении средств мониторинга на основе агента и ручного инструментирования на основе пакета SDK будут учитываться только параметры инструментирования вручную. Это позволяет предотвратить отправку повторяющихся данных. Чтобы узнать больше об этом, ознакомьтесь с [разделом устранение неполадок](#troubleshooting) ниже.

#### <a name="net-core"></a>.NET Core
Для мониторинга приложений .NET Core используйте [пакет SDK](./asp-net-core.md) 

#### <a name="java"></a>Java 

Если вам требуются дополнительные пользовательские данные телеметрии для приложений Java, см. статью [доступность](./java-in-process-agent.md#send-custom-telemetry-from-your-application), добавление [пользовательских измерений](./java-standalone-config.md#custom-dimensions)или использование [обработчиков данных телеметрии](./java-standalone-telemetry-processors.md). 

#### <a name="nodejs"></a>Node.js

Для инструментирования приложения Node.js используйте [пакет SDK](./nodejs.md).

#### <a name="python"></a>Python

Для мониторинга приложений Python используйте [пакет SDK](./opencensus-python.md).

## <a name="manage-application-insights-agent-for-net-applications-on-azure-virtual-machines-using-powershell"></a>Управление агентом Application Insights для приложений .NET на виртуальных машинах Azure с помощью PowerShell

> [!NOTE]
> Перед установкой агента Application Insights вам потребуется строка подключения. [Создайте новый Application Insights ресурс](./create-new-resource.md) или скопируйте строку подключения из существующего ресурса Application Insights.

> [!NOTE]
> Не знакомы с PowerShell? Ознакомьтесь с [руководством по началу работы](/powershell/azure/get-started-azureps).

Установка или обновление агента Application Insights как расширения для виртуальных машин Azure
```powershell
$publicCfgJsonString = '
{
  "redfieldConfiguration": {
    "instrumentationKeyMap": {
      "filters": [
        {
          "appFilter": ".*",
          "machineFilter": ".*",
          "virtualPathFilter": ".*",
          "instrumentationSettings" : {
            "connectionString": "InstrumentationKey=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          }
        }
      ]
    }
  }
}
';
$privateCfgJsonString = '{}';

Set-AzVMExtension -ResourceGroupName "<myVmResourceGroup>" -VMName "<myVmName>" -Location "<myVmLocation>" -Name "ApplicationMonitoring" -Publisher "Microsoft.Azure.Diagnostics" -Type "ApplicationMonitoringWindows" -Version "2.8" -SettingString $publicCfgJsonString -ProtectedSettingString $privateCfgJsonString
```

> [!NOTE]
> Вы можете установить или обновить агент Application Insights как расширение на нескольких виртуальных машинах при масштабировании с помощью цикла PowerShell.

Удаление расширения агента Application Insights из виртуальной машины Azure
```powershell
Remove-AzVMExtension -ResourceGroupName "<myVmResourceGroup>" -VMName "<myVmName>" -Name "ApplicationMonitoring"
```

Запрос состояния расширения агента Application Insights для виртуальной машины Azure
```powershell
Get-AzVMExtension -ResourceGroupName "<myVmResourceGroup>" -VMName "<myVmName>" -Name ApplicationMonitoring -Status
```

Получение списка установленных расширений для виртуальной машины Azure
```powershell
Get-AzResource -ResourceId "/subscriptions/<mySubscriptionId>/resourceGroups/<myVmResourceGroup>/providers/Microsoft.Compute/virtualMachines/<myVmName>/extensions"

# Name              : ApplicationMonitoring
# ResourceGroupName : <myVmResourceGroup>
# ResourceType      : Microsoft.Compute/virtualMachines/extensions
# Location          : southcentralus
# ResourceId        : /subscriptions/<mySubscriptionId>/resourceGroups/<myVmResourceGroup>/providers/Microsoft.Compute/virtualMachines/<myVmName>/extensions/ApplicationMonitoring
```
Вы также можете просмотреть установленные расширения в [колонке виртуальной машины Azure](../../virtual-machines/extensions/overview.md) на портале.

> [!NOTE]
> Проверьте установку, щелкнув Live Metrics Stream в ресурсе Application Insights, связанном со строкой подключения, которая использовалась для развертывания расширения агента Application Insights. При отправке данных из нескольких виртуальных машин выберите целевые виртуальные машины Azure в разделе Имя сервера. Начало потока данных может занять до минуты.

## <a name="manage-application-insights-agent-for-net-applications-on-azure-virtual-machine-scale-sets-using-powershell"></a>Управление агентом Application Insights для приложений .NET в масштабируемых наборах виртуальных машин Azure с помощью PowerShell

Установка или обновление агента Application Insights как расширения для масштабируемого набора виртуальных машин Azure
```powershell
$publicCfgHashtable =
@{
  "redfieldConfiguration"= @{
    "instrumentationKeyMap"= @{
      "filters"= @(
        @{
          "appFilter"= ".*";
          "machineFilter"= ".*";
          "virtualPathFilter": ".*",
          "instrumentationSettings" : {
            "connectionString": "InstrumentationKey=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" # Application Insights connection string, create new Application Insights resource if you don't have one. https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/microsoft.insights%2Fcomponents
          }
        }
      )
    }
  }
};
$privateCfgHashtable = @{};

$vmss = Get-AzVmss -ResourceGroupName "<myResourceGroup>" -VMScaleSetName "<myVmssName>"

Add-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ApplicationMonitoring" -Publisher "Microsoft.Azure.Diagnostics" -Type "ApplicationMonitoringWindows" -TypeHandlerVersion "2.8" -Setting $publicCfgHashtable -ProtectedSetting $privateCfgHashtable

Update-AzVmss -ResourceGroupName $vmss.ResourceGroupName -Name $vmss.Name -VirtualMachineScaleSet $vmss

# Note: depending on your update policy, you might need to run Update-AzVmssInstance for each instance
```

Удаление расширения мониторинга приложений из масштабируемых наборов виртуальных машин Azure
```powershell
$vmss = Get-AzVmss -ResourceGroupName "<myResourceGroup>" -VMScaleSetName "<myVmssName>"

Remove-AzVmssExtension -VirtualMachineScaleSet $vmss -Name "ApplicationMonitoring"

Update-AzVmss -ResourceGroupName $vmss.ResourceGroupName -Name $vmss.Name -VirtualMachineScaleSet $vmss

# Note: depending on your update policy, you might need to run Update-AzVmssInstance for each instance
```

Запрос состояния расширения мониторинга приложений для масштабируемых наборов виртуальных машин Azure
```powershell
# Not supported by extensions framework
```

Получение списка установленных расширений для масштабируемых наборов виртуальных машин Azure
```powershell
Get-AzResource -ResourceId /subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.Compute/virtualMachineScaleSets/<myVmssName>/extensions

# Name              : ApplicationMonitoringWindows
# ResourceGroupName : <myResourceGroup>
# ResourceType      : Microsoft.Compute/virtualMachineScaleSets/extensions
# Location          :
# ResourceId        : /subscriptions/<mySubscriptionId>/resourceGroups/<myResourceGroup>/providers/Microsoft.Compute/virtualMachineScaleSets/<myVmssName>/extensions/ApplicationMonitoringWindows
```

## <a name="troubleshooting"></a>Устранение неполадок

Найдите советы по устранению неполадок для Application Insights расширения агента мониторинга для приложений .NET, выполняющихся на виртуальных машинах Azure и в масштабируемых наборах виртуальных машин.

> [!NOTE]
> Приложения .NET Core, Node.js и Python поддерживаются только на виртуальных машинах Azure и масштабируемых наборах виртуальных машин Azure с помощью инструментирования вручную на основе пакета SDK, поэтому приведенные ниже действия не применяются к этим сценариям.

Выходные данные выполнения расширения регистрируются в файле, расположенном в следующих каталогах:
```Windows
C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.ApplicationMonitoringWindows\<version>\
```

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как [развернуть приложение в масштабируемом наборе виртуальных машин Azure](../../virtual-machine-scale-sets/virtual-machine-scale-sets-deploy-app.md).
* [Настройте веб-тесты доступности](monitor-web-app-availability.md) , которые будут оповещены, если ваша конечная точка не работает.