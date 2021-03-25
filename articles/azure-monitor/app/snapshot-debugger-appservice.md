---
title: Включение Snapshot Debugger для приложений .NET в службе приложений Azure | Документация Майкрософт
description: Включение Snapshot Debugger для приложений .NET в службе приложений Azure
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 03/26/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 26538f48213d025c6fe71fb55abb17a025a23b45
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105025685"
---
# <a name="enable-snapshot-debugger-for-net-apps-in-azure-app-service"></a>Включение Snapshot Debugger для приложений .NET в службе приложений Azure

В настоящее время Snapshot Debugger поддерживает приложения ASP.NET и ASP.NET Core, работающие в службе приложений Azure в планах служб Windows.

При использовании отладчика моментальных снимков рекомендуется запускать приложение на уровне службы "базовый" или выше.

Для большинства приложений уровни обслуживания "бесплатный" и "общий" не содержат достаточно памяти или места на диске для сохранения моментальных снимков.

## <a name="enable-snapshot-debugger"></a><a id="installation"></a> Включить Snapshot Debugger
Чтобы включить Snapshot Debugger для приложения, следуйте приведенным ниже инструкциям.

Если вы используете другой тип службы Azure, ниже приведены инструкции по включению Snapshot Debugger на других поддерживаемых платформах:
* [Функция Azure](snapshot-debugger-function-app.md?toc=/azure/azure-monitor/toc.json)
* [Oблачныe службы Azure2](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Службы Service Fabric Azure](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Профилирование веб-приложений, работающих на виртуальной машине Azure или в масштабируемом наборе виртуальных машин, с помощью Application Insights Profiler](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Локальные виртуальные или физические компьютеры](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)

> [!NOTE]
> Если вы используете предварительную версию .NET Core или приложение ссылается на Application Insights SDK напрямую или косвенно через зависимую сборку, следуйте инструкциям по [включению snapshot Debugger для других сред](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) , чтобы включить пакет NuGet [Microsoft. ApplicationInsights. SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) в приложение, а затем выполните остальные инструкции ниже. 
>
> Установка Application Insights Snapshot Debugger, не поддерживающая код, соответствует политике поддержки .NET Core.
> Дополнительные сведения о поддерживаемых средах выполнения см. в разделе [Политика поддержки .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

Snapshot Debugger предварительно устанавливается в составе среды выполнения служб приложений, но его необходимо включить, чтобы получить моментальные снимки для приложения службы приложений.

После развертывания приложения выполните следующие действия, чтобы включить отладчик моментальных снимков:

1. Перейдите на панель управления Azure для службы приложений.
2. Перейдите на страницу **параметров > Application Insights** .

   ![Включение Application Insights на портале служб приложений](./media/snapshot-debugger/applicationinsights-appservices.png)

3. Либо следуйте инструкциям на странице, чтобы создать новый ресурс, либо выберите существующий ресурс App Insights для мониторинга приложения. Также убедитесь, что оба параметра для Snapshot Debugger **включены.**

   ![Добавление расширения сайта Application Insights][Enablement UI]

4. Snapshot Debugger теперь включена с помощью параметра приложения служб приложений.

    ![Параметр приложения для Snapshot Debugger][snapshot-debugger-app-setting]

## <a name="enable-snapshot-debugger-for-other-clouds"></a>Включение Snapshot Debugger для других облаков

Сейчас единственными регионами, требующими внесения изменений в конечную точку, являются [Azure для государственных организаций](../../azure-government/compare-azure-government-global-azure.md#application-insights) и [Azure для Китая](/azure/china/resources-developer-guide) через строку подключения Application Insights.

|Свойство строки подключения    | Облако для государственных организаций США | Облако для Китая |   
|---------------|---------------------|-------------|
|снапшотендпоинт         | `https://snapshot.monitor.azure.us`    | `https://snapshot.monitor.azure.cn` |

Дополнительные сведения о других переопределениях соединений см. в [документации по Application Insights](./sdk-connection-string.md?tabs=net#connection-string-with-explicit-endpoint-overrides).

## <a name="disable-snapshot-debugger"></a>Отключить Snapshot Debugger

Выполните те же действия, что и для параметра **Enable snapshot Debugger**, но установите оба переключателя для snapshot Debugger в значение **Off**.

Рекомендуется включить Snapshot Debugger для всех приложений, чтобы упростить диагностику исключений приложений.

## <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

Для службы приложений Azure можно задать параметры приложения в шаблоне Azure Resource Manager, чтобы включить Snapshot Debugger и Profiler, см. следующий фрагмент шаблона:

```json
{
  "apiVersion": "2015-08-01",
  "name": "[parameters('webSiteName')]",
  "type": "Microsoft.Web/sites",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[variables('hostingPlanName')]"
  ],
  "tags": { 
    "[concat('hidden-related:', resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName')))]": "empty",
    "displayName": "Website"
  },
  "properties": {
    "name": "[parameters('webSiteName')]",
    "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
  },
  "resources": [
    {
      "apiVersion": "2015-08-01",
      "name": "appsettings",
      "type": "config",
      "dependsOn": [
        "[parameters('webSiteName')]",
        "[concat('AppInsights', parameters('webSiteName'))]"
      ],
      "properties": {
        "APPINSIGHTS_INSTRUMENTATIONKEY": "[reference(resourceId('Microsoft.Insights/components', concat('AppInsights', parameters('webSiteName'))), '2014-04-01').InstrumentationKey]",
        "APPINSIGHTS_PROFILERFEATURE_VERSION": "1.0.0",
        "APPINSIGHTS_SNAPSHOTFEATURE_VERSION": "1.0.0",
        "DiagnosticServices_EXTENSION_VERSION": "~3",
        "ApplicationInsightsAgent_EXTENSION_VERSION": "~2"
      }
    }
  ]
},
```

## <a name="next-steps"></a>Следующие шаги

- Создание трафика для приложения, которое может вызвать исключение. Затем подождите 10 – 15 минут, чтобы моментальные снимки отправлялись на экземпляр Application Insights.
- См. раздел [моментальные снимки](snapshot-debugger.md?toc=/azure/azure-monitor/toc.json#view-snapshots-in-the-portal) в портал Azure.
- Справку по устранению неполадок Snapshot Debugger см. в разделе [snapshot Debugger устранение неполадок](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

[Enablement UI]: ./media/snapshot-debugger/enablement-ui.png
[snapshot-debugger-app-setting]:./media/snapshot-debugger/snapshot-debugger-app-setting.png