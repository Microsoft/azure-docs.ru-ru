---
title: Профилирование облачных служб реального времени Azure с помощью Application Insights | Документация Майкрософт
description: Включите Application Insights Profiler для облачных служб Azure.
ms.topic: conceptual
ms.custom: devx-track-dotnet
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 5eb6680bba44872821a2f0d094d31d0ad994f8a0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589590"
---
# <a name="profile-live-azure-cloud-services-with-application-insights"></a>Профилирование облачных служб реального времени Azure с помощью Application Insights

Вы можете развернуть Application Insights Profiler для следующих служб:
* [Служба приложений Azure](profiler.md?toc=/azure/azure-monitor/toc.json)
* [Приложения Azure Service Fabric](profiler-servicefabric.md?toc=/azure/azure-monitor/toc.json)
* [Виртуальные машины Azure](profiler-vm.md?toc=/azure/azure-monitor/toc.json)

Application Insights Profiler поставляется с расширением системы диагностики Azure. Вам достаточно настроить систему диагностики Azure, чтобы установить Profiler и отправлять профили в ресурс Application Insights.

## <a name="enable-profiler-for-azure-cloud-services"></a>Включение Profiler для облачных служб Azure
1. Убедитесь, что вы используете [платформа .NET Framework 4.6.1](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) или более поздней версии. Если используется семейство ОС 4, необходимо установить платформа .NET Framework 4.6.1 или более поздней версии с [задачей запуска](../../cloud-services/cloud-services-dotnet-install-dotnet.md). Семейство ОС версии 5 включает совместимую версию платформа .NET Framework по умолчанию. 

1. Добавьте [Application Insights для облачных служб Azure](./cloudservices.md?toc=%2fazure%2fazure-monitor%2ftoc.json).

    **Исправлена ошибка в профилировщике, который поставляется в WAD для облачных служб.** Последняя версия WAD (1.12.2.0) для облачных служб работает со всеми последними версиями пакета SDK для App Insights. Узлы облачных служб будут автоматически обновлять WAD, но это немедленный процесс. Чтобы принудительно выполнить обновление, можно повторно развернуть службу или перезагрузить узел.

1. Отслеживание запросов с помощью Application Insights.

    * Application Insights может автоматически отслеживает запросы для веб-ролей ASP.NET.

    * Для рабочих ролей [добавьте код для трассировки запросов](profiler-trackrequests.md?toc=/azure/azure-monitor/toc.json).

1. Настройте расширение система диагностики Azure, чтобы включить профилировщик:

    а. Выберите файл [система диагностики Azure](../agents/diagnostics-extension-overview.md) *Diagnostics. wadcfgx* для роли приложения, как показано ниже:  

      ![Расположение файла конфигурации диагностики](./media/profiler-cloudservice/cloudservice-solutionexplorer.png)  

      Если вы не можете найти файл, см. статью [Настройка системы диагностики для облачных служб и виртуальных машин Azure](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines).

    b. Добавьте следующий раздел `SinksConfig` в качестве дочернего элемента `WadCfg`:  

      ```xml
      <WadCfg>
        <DiagnosticMonitorConfiguration>...</DiagnosticMonitorConfiguration>
        <SinksConfig>
          <Sink name="MyApplicationInsightsProfiler">
            <!-- Replace with your own Application Insights instrumentation key. -->
            <ApplicationInsightsProfiler>00000000-0000-0000-0000-000000000000</ApplicationInsightsProfiler>
          </Sink>
        </SinksConfig>
      </WadCfg>
      ```

    > [!NOTE]
    > Если файл *diagnostics.wadcfgx* содержит другой приемник типа ApplicationInsights, все три ключа инструментирования должны совпадать.  
    > * Ключ, используемый в приложении. 
    > * Ключ, используемый в приемнике ApplicationInsights. 
    > * Ключ, используемый в приемнике ApplicationInsightsProfiler. 
    >
    > Фактическое значения ключа инструментирования, который используется приемником `ApplicationInsights`, можно найти в файлах *ServiceConfiguration.\*.cscfg*. 
    > Начиная с выпуска пакета SDK Azure для Visual Studio 15.5, совпадать должны только ключи инструментирования, используемые приложением и приемником ApplicationInsightsProfiler.

1. Разверните службу с новой конфигурацией диагностики. При этом Application Insights Profiler настраивается для этой службы.
 
## <a name="next-steps"></a>Дальнейшие действия

* Создайте трафик к приложению (например, запустите [тест доступности](monitor-web-app-availability.md)). Подождите 10–15 минут, пока трассировки не начнут отправляться в экземпляр Application Insights.
* См. раздел [трассировки профайлера](profiler-overview.md?toc=/azure/azure-monitor/toc.json) в портал Azure.
* Дополнительные сведения об устранении неполадок Profiler см. в статье [Устранение неполадок по включению и просмотру Application Insights Profiler](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).

