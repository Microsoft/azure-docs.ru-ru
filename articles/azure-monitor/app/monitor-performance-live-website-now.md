---
title: Мониторинг активного веб-приложения ASP.NET с помощью Azure Application Insights | Документация Майкрософт
description: Мониторинг производительности веб-сайта без необходимости его повторного развертывания. Работает с веб-приложениями ASP.NET, размещенными локально или в виртуальных машинах.
ms.topic: conceptual
ms.date: 08/26/2019
ms.custom: devx-track-dotnet
ms.openlocfilehash: 79e14c171adde89c43c5ea82a60db39133157293
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100576442"
---
# <a name="instrument-web-apps-at-runtime-with-application-insights-codeless-attach"></a>Инструментирование веб-приложений во время выполнения с помощью Application Insights бескодового подключения

> [!IMPORTANT]
> Монитор состояния больше не рекомендуется использовать, и начиная с **1 июня 2021** эта версия монитора состояния не будет поддерживаться. Он был заменен агентом Azure Monitor Application Insights (прежнее название — монитор состояния v2). См. нашу документацию по [развертыванию локальных серверов](./status-monitor-v2-overview.md) , [виртуальным машинам Azure и развертываниям масштабируемых наборов виртуальных машин](./azure-vm-vmss-apps.md).

Действующее веб-приложение можно инструментировать с помощью Azure Application Insights, не прибегая к изменению или повторному развертыванию кода. Вам потребуется подписка [Microsoft Azure](https://azure.com) .

Монитор состояний используется для инструментирования приложения .NET, размещенного в IIS (локально или на виртуальной машине).

- Если приложение развернуто в ВИРТУАЛЬНОЙ машине Azure или в масштабируемом наборе виртуальных машин Azure, следуйте [этим инструкциям](azure-vm-vmss-apps.md).
- Если ваше приложение развернуто в службах приложений Azure, выполните [эти инструкции](azure-web-apps.md).
- Если приложение развернуто на виртуальной машине Azure, вы можете включить мониторинг Application Insights на панели управления Azure.
- (Существуют также отдельные статьи о инструментировании [облачных служб Azure](./cloudservices.md).)


![Снимок экрана App Insights: графики, содержащие сведения о неудачных запросах, времени отклика сервера и запросов сервера](./media/monitor-performance-live-website-now/overview-graphs.png)

Вы можете выбрать один из двух указанных ниже вариантов применения Application Insights для веб-приложений .NET.

* **Во время сборки.** [Добавьте пакет SDK для Application Insights][greenbrown] в код своего веб-приложения.
* **Во время выполнения.** Инструментируйте веб-приложение на сервере, как описано ниже, без повторной сборки и развертывания кода.

> [!NOTE]
> При использовании инструментирования времени сборки инструментирование времени выполнения не будет работать, даже если оно включено.

Ниже представлено общее сравнение предлагаемых вариантов.

|  | Во время сборки | Во время выполнения |
| --- | --- | --- |
| **Запросы, & исключения** |Да |Да |
| **[Более подробные исключения](./asp-net-exceptions.md)** | |Да |
| **[Диагностика зависимостей](./asp-net-dependencies.md)** |На платформе .NET 4.6 или более поздней, неполные сведения |Да, полные сведения: коды результатов, текст команд SQL, HTTP-команда|
| **[Счетчики производительности системы](./performance-counters.md)** |Да |Да |
| **[API для пользовательской телеметрии][api]** |Да |Нет |
| **[Интеграция журнала трассировки](./asp-net-trace-logs.md)** |Да |Нет |
| **[Просмотр страницы & данных пользователя](./javascript.md)** |Да |Нет |
| **Требуется повторная сборка кода** |Да | Нет |



## <a name="monitor-a-live-iis-web-app"></a>Мониторинг активного веб-приложения IIS

Если приложение размещено на сервере IIS, включите Application Insights с помощью монитора состояний.

1. Войдите на веб-сервер IIS с учетными данными администратора.
2. Если монитор состояний Application Insights еще не установлен, [скачайте и запустите установщик](#download).
3. В мониторе состояний выберите установленное веб-приложение или веб-сайт, который требуется отслеживать. Выполните вход с использованием учетных данных Azure.

    Настройте ресурс, в котором должны отображаться результаты на портале Application Insights. (Как правило, проще всего создать ресурс. Выберите имеющийся ресурс, если у вас уже есть [веб-тесты][availability] или [наблюдение за клиентами][client] для этого приложения.) 

    ![Выберите приложение и ресурс.](./media/monitor-performance-live-website-now/appinsights-036-configAIC.png)

4. Перезапустите IIS.

    ![Выберите "Перезапустить" в верхней части диалогового окна.](./media/monitor-performance-live-website-now/appinsights-036-restart.png)

    Работа вашей веб-службы будет ненадолго прервана.

## <a name="customize-monitoring-options"></a>Настройка параметров мониторинга

Если включить Application Insights, в веб-приложение будут добавлены библиотеки DLL и файл ApplicationInsights.config. Вы можете [отредактировать CONFIG-файл](./configuration-with-applicationinsights-config.md), чтобы изменить некоторые параметры.

## <a name="when-you-re-publish-your-app-re-enable-application-insights"></a>При повторной публикации приложения необходимо повторно включить Application Insights

Прежде чем повторно опубликовать приложение, необходимо [добавить Application Insights в код в Visual Studio][greenbrown]. Вы получите более подробные данные телеметрии и возможность написать пользовательскую телеметрию.

Если вы хотите повторно опубликовать приложение, не добавляя Application Insights в код, имейте в виду, что в процессе развертывания библиотеки DLL и файл ApplicationInsights.config могут быть удалены из опубликованного веб-сайта. Таким образом:

1. При редактировании файла ApplicationInsights.config сделайте его копию, прежде чем повторно опубликовать приложение.
2. Повторно опубликуйте приложение.
3. Повторно включите мониторинг Application Insights. Используйте подходящий способ: панель управления веб-приложением Azure или монитор состояния на узле IIS.
4. Восстановите изменения, внесенные в CONFIG-файл.


## <a name="troubleshooting"></a><a name="troubleshoot"></a>Устранение неполадок

### <a name="confirm-a-valid-installation"></a>Подтверждение правильности установки 

Ниже приведены некоторые действия, которые вы можете выполнить, чтобы убедиться в успешности установки.

- Убедитесь, что файл applicationinsights.config находится в целевом каталоге приложения и содержит ikey.

- Если вы считаете, что данные отсутствуют, можно выполнить запрос в [аналитике](../logs/log-analytics-tutorial.md) , чтобы получить список всех облачных ролей, отправляющих данные телеметрии.
  ```Kusto
  union * | summarize count() by cloud_RoleName, cloud_RoleInstance
  ```

- Если необходимо подтвердить, что Application Insights успешно присоединена, можно запустить программу [Sysinternals Handle](/sysinternals/downloads/handle) в окне командной строки, чтобы убедиться, что applicationinsights.dll ЗАГРУЖЕНЫ службами IIS.

  ```console
  handle.exe /p w3wp.exe
  ```


### <a name="cant-connect-no-telemetry"></a>Проблемы с подключением? Отсутствие данных телеметрии

* Чтобы монитор состояний работал, в брандмауэре сервера необходимо открыть [некоторые исходящие порты](./ip-addresses.md#outgoing-ports) .

### <a name="unable-to-login"></a>Не удалось войти в систему

Если монитор состояний не может войти в систему, установите командную строку. Монитор состояний пытается войти в систему для сбора ключа ikey, однако его можно указать вручную с помощью команды:

```powershell
Import-Module 'C:\Program Files\Microsoft Application Insights\Status Monitor\PowerShell\Microsoft.Diagnostics.Agent.StatusMonitor.PowerShell.dll'
Start-ApplicationInsightsMonitoring -Name appName -InstrumentationKey 00000000-000-000-000-0000000
```

### <a name="could-not-load-file-or-assembly-systemdiagnosticsdiagnosticsource"></a>Не удалось загрузить файл или сборку System.Diagnostics.DiagnosticSource

Данная ошибка может возникать после включения Application Insights. Это связано с тем, что программа установки заменяет эту библиотеку DLL в вашем каталоге bin.
Чтобы обновить файл web.config, внеся в него исправления:

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Diagnostics.DiagnosticSource" publicKeyToken="cc7b13ffcd2ddd51"/>
    <bindingRedirect oldVersion="0.0.0.0-4.*.*.*" newVersion="4.0.2.1"/>
</dependentAssembly>
```

Мы отслеживаем эту ошибку [здесь](https://github.com/MohanGsk/ApplicationInsights-Home).


### <a name="application-diagnostic-messages"></a>Диагностические сообщения приложения

* Откройте монитор состояний и на левой панели выберите свое приложение. Проверьте, не появились ли в разделе "Уведомления о конфигурации» сообщения с диагностическими сведениями для этого приложения:

  ![Откройте колонку "Производительность", чтобы отобразить сведения о запросе, времени отклика, зависимости и другие данные.](./media/monitor-performance-live-website-now/appinsights-status-monitor-diagnostics-message.png)
  
### <a name="detailed-logs"></a>Подробные журналы

* По умолчанию монитор состояний будет выводить диагностические журналы в `C:\Program Files\Microsoft Application Insights\Status Monitor\diagnostics.log`.

* Чтобы выводить подробные журналы, измените файл конфигурации `C:\Program Files\Microsoft Application Insights\Status Monitor\Microsoft.Diagnostics.Agent.StatusMonitor.exe.config` и добавьте `<add key="TraceLevel" value="All" />` для `appsettings`.
Затем перезапустите монитор состояний.

* Так как монитор состояния является приложением .NET, можно также включить [трассировку .NET, добавив соответствующие средства диагностики в файл конфигурации](/dotnet/framework/configure-apps/file-schema/trace-debug/system-diagnostics-element). Например, в некоторых сценариях может быть полезно узнать, что происходит на уровне сети, [настроив трассировку сети](/dotnet/framework/network-programming/how-to-configure-network-tracing) .

### <a name="insufficient-permissions"></a>Недостаточные разрешения
  
* Если на сервере отображается сообщение "Недостаточно разрешений", выполните следующее:
  * В диспетчере IIS выберите свой пул приложений, откройте **Дополнительные параметры** и в разделе **Модель процесса** скопируйте значение параметра "Идентификация".
  * На панели управления компьютера добавьте это значение к группе пользователей монитора производительности.

### <a name="conflict-with-systems-center-operations-manager"></a>Конфликт с Systems Center Operations Manager

* Если на вашем сервере установлен MMA/SCOM (Systems Center Operations Manager), некоторые версии могут конфликтовать. Удалите SCOM и монитор состояний и повторно установите последние версии.

### <a name="failed-or-incomplete-installation"></a>Неудачная или неполная установка

Если во время установки происходит сбой монитора состояний, вы можете остаться с неполной установкой, из которой монитор состояний не сможет восстановиться. В результате потребуется выполнить сброс вручную.

Удалите следующие файлы, найденные в каталоге приложения:
- любые библиотеки DLL в каталоге bin, начинающиеся с Microsoft.AI либо с Microsoft.ApplicationInsights;
- библиотеку DLL Microsoft.Web.Infrastructure.dll в каталоге bin;
- библиотеку DLL System.Diagnostics.DiagnosticSource.dll в каталоге bin;
- в каталоге приложения удалите App_Data\packages;
- в каталоге приложения удалите файл applicationinsights.config.


### <a name="additional-troubleshooting"></a>Дополнительные сведения об устранении неполадок

* Ознакомьтесь с дополнительными [сведениями об устранении неполадок][qna].

## <a name="system-requirements"></a>Требования к системе
Операционные системы, которые поддерживаются для монитора состояний Application Insights на сервере:

* Windows Server 2008
* Windows Server 2008 R2
* Windows Server 2012
* Windows Server 2012 R2.
* Windows Server 2016

с последней версией SP и платформа .NET Framework 4,5 (монитор состояния построена на основе этой версии платформы)

На клиентских компьютерах должна быть установлена ОС Windows 7, 8, 8.1 или 10 с платформой .NET Framework 4.5.

Поддерживаются такие версии IIS: 7, 7.5, 8, 8.5 (IIS – обязательный компонент).

## <a name="automation-with-powershell"></a>Автоматизация с помощью PowerShell
Мониторинг можно запускать и останавливать с помощью PowerShell на сервере IIS.

Сначала импортируйте модуль Application Insights:

```powershell
Import-Module 'C:\Program Files\Microsoft Application Insights\Status Monitor\PowerShell\Microsoft.Diagnostics.Agent.StatusMonitor.PowerShell.dll'
```

Узнайте, какие приложения отслеживаются:

`Get-ApplicationInsightsMonitoringStatus [-Name appName]`

* `-Name` (необязательный параметр) — имя веб-приложения.
* Отображает состояние мониторинга Application Insights для каждого веб-приложения (или именованного приложения) на этом сервере IIS.
* Возвращает `ApplicationInsightsApplication` для каждого приложения.

  * `SdkState==EnabledAfterDeployment`: приложение отслеживается. Оно инструментировано во время выполнения с помощью монитора состояния или командлета `Start-ApplicationInsightsMonitoring`.
  * `SdkState==Disabled`: приложение не инструментировано для Application Insights. Оно либо никогда не было инструментировано, либо мониторинг во время выполнения был отключен с помощью монитора состояния или командлета `Stop-ApplicationInsightsMonitoring`.
  * `SdkState==EnabledByCodeInstrumentation`: приложение инструментировано путем добавления пакета SDK в исходный код. Этот пакет SDK нельзя обновить или остановить.
  * `SdkVersion` — отображает версию, которая используется для мониторинга этого приложения.
  * `LatestAvailableSdkVersion`— отображает версию, доступную сейчас в коллекции NuGet. Чтобы обновить приложение до этой версии, используйте командлет `Update-ApplicationInsightsMonitoring`.

`Start-ApplicationInsightsMonitoring -Name appName -InstrumentationKey 00000000-000-000-000-0000000`

* `-Name` — имя приложения на сервере IIS.
* `-InstrumentationKey` — ключ инструментирования ресурса Application Insights, в котором должны отображаться результаты.
* Этот командлет влияет только на приложения, которые еще не инструментированы, то есть SdkState==NotInstrumented.

    Командлет не влияет на приложение, которое уже инструментировано. Не имеет значения, инструментировано ли приложение во время сборки при добавлении пакета SDK в код или с использованием этого же командлета ранее во время выполнения.

    Версия пакета SDK для инструментирования приложения — это последняя версия, загруженная на этот сервер.

    Чтобы загрузить последнюю версию, используйте командлет Update-ApplicationInsightsVersion.
* В случае успешного выполнения возвращает `ApplicationInsightsApplication` . В случае сбоя трассировка записывается в stderr.

   ```output
   Name                      : Default Web Site/WebApp1
   InstrumentationKey        : 00000000-0000-0000-0000-000000000000
   ProfilerState             : ApplicationInsights
   SdkState                  : EnabledAfterDeployment
   SdkVersion                : 1.2.1
   LatestAvailableSdkVersion : 1.2.3
   ```

`Stop-ApplicationInsightsMonitoring [-Name appName | -All]`

* `-Name` — имя приложения на сервере IIS.
* `-All` — останавливает мониторинг всех приложений на этом сервере IIS, для которых `SdkState==EnabledAfterDeployment`.
* Останавливает мониторинг для указанных приложений и удаляет инструментирование. Применимо только для приложений, инструментированных во время выполнения с использованием монитора состояния или командлета Start-ApplicationInsightsApplication. (`SdkState==EnabledAfterDeployment`)
* Возвращает ApplicationInsightsApplication.

`Update-ApplicationInsightsMonitoring -Name appName [-InstrumentationKey "0000000-0000-000-000-0000"`]

* `-Name` — имя веб-приложения на сервере IIS.
* `-InstrumentationKey` (Необязательно.) Используйте этот параметр, чтобы изменить ресурс, в который отправляется телеметрии приложения.
* Этот командлет:
  * Обновляет именованное приложение до последней версии пакета SDK, загруженной на этот компьютер (работает, только если `SdkState==EnabledAfterDeployment`).
  * Если указан ключ инструментирования, именованное приложение повторно настраивается для отправки данных телеметрии в ресурс с этим ключом (работает, если `SdkState != Disabled`).

`Update-ApplicationInsightsVersion`

* Загружает последний пакет SDK для Application Insights на сервер.

## <a name="questions-about-status-monitor"></a><a name="questions"></a>Вопросы о мониторе состояния

### <a name="what-is-status-monitor"></a>Что такое монитор состояния?

Классическое приложение, установленное на веб-сервер IIS. Оно позволяет инструментировать и настраивать веб-приложения. 

### <a name="when-do-i-use-status-monitor"></a>В каких случаях нужно использовать монитор состояния?

* Для инструментирования любого веб-приложения на сервере IIS (даже если оно уже запущено).
* Чтобы включить дополнительные данные телеметрии для веб-приложений, [созданных с помощью пакета SDK для Application Insights](./asp-net.md) во время компиляции. 

### <a name="can-i-close-it-after-it-runs"></a>Можно ли его закрыть после выполнения?

Да. После инструментирования нужных веб-сайтов его можно закрыть.

Монитор состояния не собирает данные телеметрии самостоятельно. Он лишь настраивает веб-приложения и устанавливает некоторые разрешения.

### <a name="what-does-status-monitor-do"></a>Как работает монитор состояния?

При выборе веб-приложения для инструментирования монитор состояния делает следующее:

* Скачивает и помещает сборки Application Insights и файл ApplicationInsights.config в папку с двоичными файлами веб-приложений.
* Включает профилирование среды CLR для сбора вызовов зависимостей.

### <a name="what-version-of-application-insights-sdk-does-status-monitor-install"></a>Какую версию пакета SDK Application Insights устанавливает монитор состояний?

Сейчас монитор состояний устанавливает только пакет SDK Application Insights версии 2.3 или 2.4. 

Пакет SDK для Application Insights версии 2,4 — это [Последняя версия для поддержки .net 4,0](https://github.com/microsoft/ApplicationInsights-dotnet/releases/tag/v2.5.0-beta1) , которая была [конца строки января 2016](https://devblogs.microsoft.com/dotnet/support-ending-for-the-net-framework-4-4-5-and-4-5-1/). Таким образом, теперь монитор состояния можно использовать для инструментирования приложения .NET 4,0. 

### <a name="do-i-need-to-run-status-monitor-whenever-i-update-the-app"></a>Нужно ли запускать монитор состояния при каждом обновлении приложения?

Нет, если повторное развертывание выполняется поэтапно. 

Если в процессе публикации вы выбрали параметр Delete existing files (Удалить существующие файлы), для настройки Application Insights необходимо перезапустить монитор состояния.

### <a name="what-telemetry-is-collected"></a>Какие данные телеметрии собираются?

Для приложений, инструментированных во время выполнения с использованием монитора состояния:

* HTTP-запросы;
* вызовы зависимостей;
* Исключения
* Счетчики производительности

Для уже инструментированных приложений во время компиляции:

 * счетчики процессов;
 * вызовы зависимостей (.NET 4.5) и возвращаемые значения в вызовах зависимостей (.NET 4.6);
 * значения трассировки стека исключений.

[Дополнительные сведения](https://apmtips.com/posts/2016-11-18-how-application-insights-status-monitor-not-monitors-dependencies/)

## <a name="video"></a>Видео

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="download-status-monitor"></a><a name="download"></a>Скачивание монитора состояний

- Использование нового [модуля PowerShell](./status-monitor-v2-overview.md)
- Скачивание и запуск [установщика монитор состояния](https://go.microsoft.com/fwlink/?LinkId=506648)
- Как альтернативный вариант, запустите [установщик веб-платформы](https://www.microsoft.com/web/downloads/platform.aspx) и найдите в нем монитор состояний Application Insights.

## <a name="next-steps"></a><a name="next"></a>Следующие шаги

Просмотр телеметрии:

* [Изучите метрики](../essentials/metrics-charts.md), чтобы отслеживать производительность и использование.
* [Выполняйте поиск событий и журналов][diagnostic] для диагностики неполадок.
* [Аналитика](../logs/log-query-overview.md) для создания расширенных запросов.

Добавление данных телеметрии:

* [Создайте веб-тесты][availability], чтобы убедиться, что ваш сайт продолжает работать.
* [Добавьте телеметрию веб-клиента][usage], чтобы просматривать исключения в коде веб-страницы и вставлять вызовы трассировки.
* [Добавьте пакет SDK для Application Insights в код][greenbrown], чтобы иметь возможность вставлять вызовы трассировки и журналов.

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[availability]: monitor-web-app-availability.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[greenbrown]: ./asp-net.md
[qna]: ../faq.md
[roles]: ./resources-roles-access-control.md
[usage]: ./javascript.md