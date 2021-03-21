---
title: Агент Application Insights Azure — Приступая к работе | Документация Майкрософт
description: Краткое руководство по Application Insightsу агенту. Отслеживайте производительность веб-сайта без повторного развертывания веб-сайта. Работает с веб-приложениями ASP.NET, размещенными локально, в виртуальных машинах или в Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 01/22/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 21184e1623fd47e8367d4c5dfbc2c85debe93124
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100587391"
---
# <a name="get-started-with-azure-monitor-application-insights-agent-for-on-premises-servers"></a>Начало работы с агентом Azure Monitor Application Insights для локальных серверов

Эта статья содержит команды краткого руководства, которые должны работать в большинстве сред.
Инструкции зависят от коллекция PowerShell распространения обновлений.
Эти команды поддерживают параметр PowerShell `-Proxy` .

Описание этих команд, инструкции по настройке и сведения об устранении неполадок см. в [подробных инструкциях](status-monitor-v2-detailed-instructions.md).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="download-and-install-via-powershell-gallery"></a>Загрузка и установка с помощью коллекция PowerShell

### <a name="install-prerequisites"></a>Установка необходимых компонентов

> [!NOTE]
> По состоянию на апрель 2020 коллекция PowerShell имеет устаревшие TLS 1,1 и 1,0.
>
> Необходимые условия для аддитионнал см. в разделе [поддержка коллекция POWERSHELL TLS](https://devblogs.microsoft.com/powershell/powershell-gallery-tls-support).
>

Запустите PowerShell от имени администратора.
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
Install-Module -Name PowerShellGet -Force
``` 
Закройте PowerShell.

### <a name="install-application-insights-agent"></a>Установка агента Application Insights
Запустите PowerShell от имени администратора.
```powershell   
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
Install-Module -Name Az.ApplicationMonitor -AllowPrerelease -AcceptLicense
``` 

### <a name="enable-monitoring"></a>Включение мониторинга
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
Enable-ApplicationInsightsMonitoring -ConnectionString xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
    
        
## <a name="download-and-install-manually-offline-option"></a>Скачать и установить вручную (автономный режим)
### <a name="download-the-module"></a>Скачать модуль
Вручную Скачайте последнюю версию модуля из [коллекция PowerShell](https://www.powershellgallery.com/packages/Az.ApplicationMonitor).

### <a name="unzip-and-install-application-insights-agent"></a>Распакуйте и установите агент Application Insights
```powershell
$pathToNupkg = "C:\Users\t\Desktop\Az.ApplicationMonitor.0.3.0-alpha.nupkg"
$pathToZip = ([io.path]::ChangeExtension($pathToNupkg, "zip"))
$pathToNupkg | rename-item -newname $pathToZip
$pathInstalledModule = "$Env:ProgramFiles\WindowsPowerShell\Modules\Az.ApplicationMonitor"
Expand-Archive -LiteralPath $pathToZip -DestinationPath $pathInstalledModule
```
### <a name="enable-monitoring"></a>Включение мониторинга
```powershell
Enable-ApplicationInsightsMonitoring -ConnectionString xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```



## <a name="next-steps"></a>Дальнейшие действия

 Просмотр телеметрии:

- [Изучите метрики](../essentials/metrics-charts.md) для мониторинга производительности и использования.
- [Поиск событий и журналов](./diagnostic-search.md) для диагностики проблем.
- [Используйте аналитику](../logs/log-query-overview.md) для более сложных запросов.
- [Создание панелей мониторинга](./overview-dashboard.md).

 Добавление данных телеметрии:

- [Создайте веб-тесты](monitor-web-app-availability.md), чтобы убедиться, что ваш сайт продолжает работать.
- [Добавьте данные телеметрии веб-клиента](./javascript.md) , чтобы просмотреть исключения из кода веб-страницы и включить вызовы трассировки.
- [Добавьте в код пакет SDK для Application Insights](./asp-net.md) , чтобы можно было вставить вызовы трассировки и журнала.

Другие действия с агентом Application Insights:

- Ознакомьтесь с [подробными инструкциями](status-monitor-v2-detailed-instructions.md) для объяснения команд, найденных здесь.
- Используйте наше справочное по для [устранения неполадок](status-monitor-v2-troubleshoot.md) агента Application Insights.

