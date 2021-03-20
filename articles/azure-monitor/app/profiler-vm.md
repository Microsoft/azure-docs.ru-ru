---
title: Профилирование веб-приложений на виртуальной машине Azure — Application Insights Profiler
description: Профилируйте веб-приложения на виртуальной машине Azure с использованием Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 11/08/2019
ms.reviewer: mbullwin
ms.openlocfilehash: f514dd7b54ac091535aeab43a8a7d2a645b50a09
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87315841"
---
# <a name="profile-web-apps-running-on-an-azure-virtual-machine-or-a-virtual-machine-scale-set-by-using-application-insights-profiler"></a>Профилирование веб-приложений, работающих на виртуальной машине Azure или в масштабируемом наборе виртуальных машин, с использованием Application Insights Profiler

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Вы можете развернуть Azure Application Insights Profiler для таких служб:
* [Служба приложений Azure](./profiler.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Oблачныe службы Azure2](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Service Fabric](?toc=%2fazure%2fazure-monitor%2ftoc.json)

## <a name="deploy-profiler-on-a-virtual-machine-or-a-virtual-machine-scale-set"></a>Развертывание Profiler на виртуальной машине или в масштабируемом наборе виртуальных машин
В этой статье показано, как запустить Application Insights Profiler на виртуальной машине Azure или в масштабируемом наборе виртуальных машин Azure. Profiler поставляется с расширением системы диагностики Azure для виртуальных машин. Настройте расширение для выполнения Profiler и встройте пакет SDK для Application Insights в приложение.

1. Добавьте пакет SDK для Application Insights в [приложение ASP.NET](./asp-net.md).

   Чтобы просмотреть профили запросов, необходимо отправить телеметрию запроса в Application Insights.

1. Установите на виртуальную машину расширение "Диагностика Microsoft Azure". Полные примеры шаблонов Resource Manager можно найти здесь:  
   * [Виртуальная машина](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/WindowsVirtualMachine.json)
   * [Набор масштабирования виртуальных машин](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/WindowsVirtualMachineScaleSet.json)
    
     Ключевой компонент — ApplicationInsightsProfilerSink в WadCfg. Добавьте в этот раздел другой приемник, чтобы система диагностики Azure позволила Profiler отправлять данные в ваш iKey.
    
     ```json
     "SinksConfig": {
       "Sink": [
         {
           "name": "ApplicationInsightsSink",
           "ApplicationInsights": "85f73556-b1ba-46de-9534-606e08c6120f"
         },
         {
           "name": "MyApplicationInsightsProfilerSink",
           "ApplicationInsightsProfiler": "85f73556-b1ba-46de-9534-606e08c6120f"
         }
       ]
     },
     ```

1. Разверните определение развертывания измененной среды.  

   Чтобы применить изменения, обычно нужно полностью развернуть шаблон или опубликовать его в облачных службах с помощью командлетов PowerShell или Visual Studio.  

   Следующие команды PowerShell являются альтернативным способом для существующих виртуальных машин, которые касаются только расширения система диагностики Azure. Добавьте ранее упомянутый Профилерсинк в файл config, возвращаемый командой Get-AzVMDiagnosticsExtension. Затем передайте обновленную конфигурацию в команду Set-AzVMDiagnosticsExtension.

    ```powershell
    $ConfigFilePath = [IO.Path]::GetTempFileName()
    # After you export the currently deployed Diagnostics config to a file, edit it to include the ApplicationInsightsProfiler sink.
    (Get-AzVMDiagnosticsExtension -ResourceGroupName "MyRG" -VMName "MyVM").PublicSettings | Out-File -Verbose $ConfigFilePath
    # Set-AzVMDiagnosticsExtension might require the -StorageAccountName argument
    # If your original diagnostics configuration had the storageAccountName property in the protectedSettings section (which is not downloadable), be sure to pass the same original value you had in this cmdlet call.
    Set-AzVMDiagnosticsExtension -ResourceGroupName "MyRG" -VMName "MyVM" -DiagnosticsConfigurationPath $ConfigFilePath
    ```

1. Если нужное приложение выполняется с помощью [IIS](https://www.microsoft.com/web/downloads/platform.aspx), необходимо включить компонент Windows `IIS Http Tracing`.

   а. Установите удаленный доступ к среде, а затем используйте окно [Добавить функции Windows](/iis/configuration/system.webserver/tracing/) или выполните следующую команду в PowerShell (с правами администратора):  

    ```powershell
    Enable-WindowsOptionalFeature -FeatureName IIS-HttpTracing -Online -All
    ```  
   b. Если не удается установить удаленный доступ, можно использовать [Azure CLI](/cli/azure/get-started-with-azure-cli), чтобы выполнить следующую команду:  

    ```powershell
    az vm run-command invoke -g MyResourceGroupName -n MyVirtualMachineName --command-id RunPowerShellScript --scripts "Enable-WindowsOptionalFeature -FeatureName IIS-HttpTracing -Online -All"
    ```

1. Разверните приложение.

## <a name="set-profiler-sink-using-azure-resource-explorer"></a>Настройка приемника профилировщика с помощью обозреватель ресурсов Azure
У нас еще нет способа задать приемник Application Insights Profiler на портале. Вместо использования PowerShell, как описано выше, для установки приемника можно использовать обозреватель ресурсов Azure. Но обратите внимание, что при повторном развертывании виртуальной машины приемник будет потерян. Чтобы сохранить этот параметр, необходимо обновить конфигурацию, используемую при развертывании виртуальной машины.

1. Убедитесь, что расширение система диагностики Azure Windows установлено, просмотрев расширения, установленные для виртуальной машины.  

    ![Проверьте, установлено ли расширение WAD][wadextension]

2. Найдите расширение диагностики виртуальных машин для виртуальной машины. Перейдите на страницу [https://resources.azure.com](https://resources.azure.com). Разверните группу ресурсов, Microsoft. COMPUTE virtualMachines, имя виртуальной машины и расширения.  

    ![Перейдите к WAD config в обозреватель ресурсов Azure][azureresourceexplorer]

3. Добавьте приемник Application Insights Profiler в узел SinksConfig в разделе WadCfg. Если у вас еще нет раздела SinksConfig, может потребоваться добавить его. Не забудьте указать правильные Application Insights iKey в параметрах. Необходимо переключить режим проводника на чтение и запись в правом верхнем углу и нажать синюю кнопку "Изменить".

    ![Добавление приемника Application Insights Profiler][resourceexplorersinksconfig]

4. Завершив редактирование конфигурации, нажмите кнопку "вставить". Если помещение прошло успешно, в середине экрана появится зеленая галочка.

    ![Отправить запрос на добавление изменений][resourceexplorerput]






## <a name="can-profiler-run-on-on-premises-servers"></a>Можно ли запустить Profiler на локальных серверах?
Поддержка Application Insights Profiler на локальных серверах не планируется.

## <a name="next-steps"></a>Дальнейшие действия

- Создайте трафик к приложению (например, запустите [тест доступности](monitor-web-app-availability.md)). Подождите 10–15 минут, пока трассировки не начнут отправляться в экземпляр Application Insights.
- См. раздел [трассировки профайлера](profiler-overview.md?toc=/azure/azure-monitor/toc.json) в портал Azure.
- Сведения об устранении неполадок профилировщика см. в статье [Устранение неполадок по включению и просмотру Application Insights Profiler](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).

[azureresourceexplorer]: ./media/profiler-vm/azure-resource-explorer.png
[resourceexplorerput]: ./media/profiler-vm/resource-explorer-put.png
[resourceexplorersinksconfig]: ./media/profiler-vm/resource-explorer-sinks-config.png
[wadextension]: ./media/profiler-vm/wad-extension.png

