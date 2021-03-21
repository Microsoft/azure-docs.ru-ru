---
title: Профилирование Live Service Fabric приложений Azure с помощью Application Insights
description: Включение Profiler для приложения Service Fabric
ms.topic: conceptual
ms.custom: devx-track-dotnet
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 67e7765a1f46c2be5790c11687e06ea624702b9b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589558"
---
# <a name="profile-live-azure-service-fabric-applications-with-application-insights"></a>Профилирование запущенных приложений Azure Service Fabric с помощью Application Insights

Вы можете развернуть Application Insights Profiler для следующих служб:
* [Служба приложений Azure](profiler.md?toc=/azure/azure-monitor/toc.json)
* [Oблачныe службы Azure2](profiler-cloudservice.md?toc=/azure/azure-monitor/toc.json)
* [Виртуальные машины Azure](profiler-vm.md?toc=/azure/azure-monitor/toc.json)

## <a name="set-up-the-environment-deployment-definition"></a>Настройка определения развертывания среды

Application Insights Profiler поставляется в комплекте с системой диагностики Azure. Вы можете установить расширение системы диагностики Azure, используя шаблон Azure Resource Manager для кластера Service Fabric. Получите шаблон [, который устанавливает систему диагностики Azure в кластер Service Fabric](https://github.com/Azure/azure-docs-json-samples/blob/master/application-insights/ServiceFabricCluster.json).

Чтобы настроить среду, выполните следующие действия.

1. Profiler поддерживает платформа .NET Framework и .Net Core. Если вы используете платформа .NET Framework, убедитесь, что вы используете [платформа .NET Framework 4.6.1](/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed) или более поздней версии. Достаточно подтвердить, что развернутая ОС является `Windows Server 2012 R2` или более поздней. Profiler поддерживает приложения .NET Core 2,1 и более поздние версии.

1. Найдите расширение [системы диагностики Azure](../agents/diagnostics-extension-overview.md) в файле шаблона развертывания.

1. Добавьте следующий раздел `SinksConfig` в качестве дочернего элемента `WadCfg`. Замените значение свойства `ApplicationInsightsProfiler` собственным ключом инструментирования Application Insights:  

      ```json
      "SinksConfig": {
        "Sink": [
          {
            "name": "MyApplicationInsightsProfilerSink",
            "ApplicationInsightsProfiler": "00000000-0000-0000-0000-000000000000"
          }
        ]
      }
      ```

      Сведения о добавлении расширения диагностики в шаблон развертывания см. в статье [Использование мониторинга и системы диагностики с виртуальной машиной Windows и шаблонами Azure Resource Manager](../../virtual-machines/extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json).

1. Разверните кластер Service Fabric с помощью шаблона Azure Resource Manager.  
  Если все параметры верны, при установке расширения системы диагностики Azure будет установлен и включен Application Insights Profiler. 

1. Добавьте Application Insights в приложение Service Fabric.  
  Чтобы профилировщик составил профили для запросов, приложение должно отслеживать операции с Application Insights. Для API без отслеживания состояния можно ознакомиться с инструкциями по [отслеживанию запросов профилирования](profiler-trackrequests.md?toc=/azure/azure-monitor/toc.json). Дополнительные сведения об отслеживании пользовательских операций в других видах приложений см. в разделе [Отслеживание настраиваемых операций с помощью пакета SDK для .net Application Insights](custom-operations-tracking.md?toc=/azure/azure-monitor/toc.json).

1. Разверните приложение заново.


## <a name="next-steps"></a>Дальнейшие действия

* Создайте трафик к приложению (например, запустите [тест доступности](monitor-web-app-availability.md)). Подождите 10–15 минут, пока трассировки не начнут отправляться в экземпляр Application Insights.
* См. раздел [трассировки профайлера](profiler-overview.md?toc=/azure/azure-monitor/toc.json) в портал Azure.
* Сведения об устранении неполадок профилировщика см. в статье [Устранение неполадок по включению и просмотру Application Insights Profiler](profiler-troubleshooting.md?toc=/azure/azure-monitor/toc.json).
