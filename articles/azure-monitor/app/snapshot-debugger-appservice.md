---
title: Включение Snapshot Debugger для приложений .NET в службе приложений Azure | Документация Майкрософт
description: Включение Snapshot Debugger для приложений .NET в службе приложений Azure
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 03/26/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 421f80493a9cb88e8bbbddc06aa9a24042b64b17
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695474"
---
# <a name="enable-snapshot-debugger-for-net-apps-in-azure-app-service"></a>Включение Snapshot Debugger для приложений .NET в службе приложений Azure

Snapshot Debugger в настоящее время работает для приложений ASP.NET и ASP.NET Core, работающих в службе приложений Azure в планах служб Windows. При использовании отладчика моментальных снимков рекомендуется запускать приложение на уровне служб Basic или выше. Для большинства приложений уровни обслуживания "бесплатный" и "общий" не содержат достаточно памяти или места на диске для сохранения моментальных снимков.

## <a name="enable-snapshot-debugger"></a><a id="installation"></a> Включить Snapshot Debugger
Чтобы включить Snapshot Debugger для приложения, следуйте приведенным ниже инструкциям.

Если вы используете другой тип службы Azure, ниже приведены инструкции по включению Snapshot Debugger на других поддерживаемых платформах:
* [Функция Azure](snapshot-debugger-function-app.md?toc=/azure/azure-monitor/toc.json)
* [Oблачныe службы Azure2](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Службы Service Fabric Azure](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Профилирование веб-приложений, работающих на виртуальной машине Azure или в масштабируемом наборе виртуальных машин, с помощью Application Insights Profiler](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Локальные виртуальные или физические компьютеры](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)

> [!NOTE]
> Если вы используете предварительную версию .NET Core или приложение ссылается на Application Insights SDK, прямо или косвенно с помощью зависимой сборки, следуйте инструкциям по [включению snapshot Debugger для других сред](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) сначала включите пакет NuGet [Microsoft. ApplicationInsights. SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) в приложение, а затем выполните остальные инструкции ниже. 

Snapshot Debugger предварительно устанавливается в составе среды выполнения служб приложений, но его необходимо включить, чтобы получить моментальные снимки для приложения службы приложений.

После развертывания приложения выполните следующие действия, чтобы включить отладчик моментальных снимков:

1. Перейдите на панель управления Azure для службы приложений.
2. Перейдите на страницу **параметров > Application Insights** .

   ![Включение Application Insights на портале служб приложений](./media/snapshot-debugger/applicationinsights-appservices.png)

3. Либо следуйте инструкциям на странице, чтобы создать новый ресурс, либо выберите существующий ресурс App Insights для мониторинга приложения. Также убедитесь, что оба параметра для Snapshot Debugger **включены.**

   ![Добавление расширения сайта Application Insights][Enablement UI]

4. Snapshot Debugger теперь включена с помощью параметра приложения служб приложений.

    ![Параметр приложения для Snapshot Debugger][snapshot-debugger-app-setting]

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

## <a name="next-steps"></a>Дальнейшие действия

- Создание трафика для приложения, которое может вызвать исключение. Затем подождите 10 – 15 минут, чтобы моментальные снимки отправлялись на экземпляр Application Insights.
- См. раздел [моментальные снимки](snapshot-debugger.md?toc=/azure/azure-monitor/toc.json#view-snapshots-in-the-portal) в портал Azure.
- Справку по устранению неполадок Snapshot Debugger см. в разделе [snapshot Debugger устранение неполадок](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

[Enablement UI]: ./media/snapshot-debugger/enablement-ui.png
[snapshot-debugger-app-setting]:./media/snapshot-debugger/snapshot-debugger-app-setting.png

