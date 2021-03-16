---
title: Устранение неполадок, связанных с тем, что в Application Insights для .NET не отображаются данные
description: Не отображаются данные в Azure Application Insights? Попробуйте здесь.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/21/2020
ms.openlocfilehash: 40fbe4d08676d7cc56478d3740424fccaa7addc0
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103562201"
---
# <a name="troubleshooting-no-data---application-insights-for-netnet-core"></a>Устранение неполадок, связанных с тем, что в Application Insights для .NET/.NET Core не отображаются данные

## <a name="some-of-my-telemetry-is-missing"></a>Некоторая телеметрия отсутствует
*В Application Insights отображается только часть событий, создаваемых приложением.*

* Если постоянно отображается одна и та же часть, это может быть связано с адаптивной [выборкой](./sampling.md). Чтобы проверить это, откройте область поиска (из колонки "Обзор") и посмотрите на экземпляр запроса или другого события. В нижней части раздела свойств нажмите кнопку "...", чтобы отобразить все сведения свойств. Если число запросов > 1, то выборка работает.
* В противном случае, возможно, превышено [ограничение на скорость передачи данных](./pricing.md#limits-summary) для вашего плана ценообразования. Эти ограничения применяются в пропорционально по минутам.

*Я столкнулся с потерей данных случайным образом.*

* Проверьте, возникают ли потери данных в [канале телеметрии](telemetry-channels.md#does-the-application-insights-channel-guarantee-telemetry-delivery-if-not-what-are-the-scenarios-in-which-telemetry-can-be-lost).

* Проверьте наличие известных проблем в [репозитории GitHub](https://github.com/Microsoft/ApplicationInsights-dotnet/issues) Канала телеметрии.

*Я столкнулся с потерей данных в консольном приложении или в веб-приложении, когда приложение собирается закончить работу.*

* Канал пакета SDK хранит данные телеметрии в буфере и отправляет их в пакетах. Если приложение завершает работу, может потребоваться явным образом вызвать [Flush()](api-custom-events-metrics.md#flushing-data). Поведение `Flush()` зависит от фактически используемого [канала](telemetry-channels.md#built-in-telemetry-channels).

## <a name="no-data-from-my-server"></a>Нет данных с моего сервера
*На моем веб-сервере установлено приложение, но данные телеметрии сервера не отображаются. На компьютере разработки все работало хорошо.*

* Возможно, проблема в брандмауэре. [Установите исключения брандмауэра для отправки данных Application Insights](./ip-addresses.md).
* На сервере IIS могут отсутствовать некоторые предварительные условия: расширяемость .NET 4.5 и ASP.NET 4.5.

*На моем веб-сервере [установлен монитор состояния](./monitor-performance-live-website-now.md) для мониторинга существующих приложений. Я не вижу никаких результатов.*

* См. раздел [Устранение неполадок](./monitor-performance-live-website-now.md#troubleshoot).

> [!IMPORTANT]
> Новые регионы Azure **должны** использовать строки подключения вместо ключей инструментирования. [Строка подключения](./sdk-connection-string.md?tabs=net) определяет ресурс, с которым необходимо связать данные телеметрии. Она также позволяет изменить конечные точки, которые ресурс будет использовать в качестве назначения для данных телеметрии. Необходимо скопировать строку подключения и добавить ее в код приложения или переменную среды.


## <a name="filenotfoundexception-could-not-load-file-or-assembly-microsoftaspnet-telemetrycorrelation"></a>FileNotFoundException: не удалось загрузить файл или сборку "Microsoft. AspNet Телеметрикоррелатион

Дополнительные сведения об этой ошибке см. в [номере GitHub 1610] ( https://github.com/microsoft/ApplicationInsights-dotnet/issues/1610) .

При обновлении с пакетов SDK старше (2,4) необходимо убедиться, что следующие изменения применены к `web.config` и `ApplicationInsights.config` :

1. Два HTTP-модуля, а не один. В `web.config` необходимо иметь два модуля HTTP. Порядок важен в некоторых сценариях:

    ``` xml
    <system.webServer>
      <modules>
          <add name="TelemetryCorrelationHttpModule" type="Microsoft.AspNet.TelemetryCorrelation.TelemetryCorrelationHttpModule, Microsoft.AspNet.TelemetryCorrelation" preCondition="integratedMode,managedHandler" />
          <add name="ApplicationInsightsHttpModule" type="Microsoft.ApplicationInsights.Web.ApplicationInsightsHttpModule, Microsoft.AI.Web" preCondition="managedHandler" />
      </modules>
    </system.webServer>
    ```

2. В `ApplicationInsights.config` кроме `RequestTrackingTelemetryModule` того, у вас должен быть следующий модуль телеметрии:

    ``` xml
    <TelemetryModules>
      <Add Type="Microsoft.ApplicationInsights.Web.AspNetDiagnosticTelemetryModule, Microsoft.AI.Web"/>
    </TelemetryModules>
    ```

***Неправильное обновление может привести к непредвиденным исключениям или телеметрии, которые не будут собираться.***


## <a name="no-add-application-insights-option-in-visual-studio"></a><a name="q01"></a>Параметр "Добавление Application Insights" в Visual Studio не отображается
*Когда я щелкаю существующий проект в обозревателе решений правой кнопкой мыши, параметры Application Insights не отображаются.*

* Средства Application Insights поддерживают не все типы проектов .NET. Проекты WCF и веб-проекты поддерживаются. В проекты других типов, например в приложения-службы и классические приложения, можно [вручную добавить пакет SDK службы Application Insights](./windows-desktop.md).
* Убедитесь в том, что установлена версия [Visual Studio 2013 с обновлением 3 или более поздним](/visualstudio/releasenotes/vs2013-update3-rtm-vs). Решение поставляется предварительно установленным с аналитическими средствами для разработчиков в рамках пакета Application Insights SDK.
* Выберите элементы **Сервис**, **Расширения и обновления** и убедитесь, что пакет **Аналитические средства для разработчиков** установлен и включен. Если это так, щелкните пункт **Обновления** , чтобы проверить, доступно ли обновление.
* Откройте диалоговое окно "Новый проект" и выберите "Веб-приложение ASP.NET". Если параметр Application Insights отображается, соответствующие средства установлены. Если нет, попробуйте удалить и заново установить средства Developer Analytics Tools.

## <a name="adding-application-insights-failed"></a><a name="q02"></a>Сбой при добавлении Application Insights
*При попытке добавить Application Insights в существующий проект я вижу сообщение об ошибке.*

Вероятные причины:

* Сбой связи с порталом Application Insights.
* В вашей учетной записи Azure возникли неполадки.
* У вас есть только [доступ на чтение в подписке или группе, в которой вы пытаетесь создать новый ресурс](./resources-roles-access-control.md).

Исправление:

* Убедитесь, что вы указали правильные учетные данные для входа в учетную запись Azure.
* В браузере убедитесь, что у вас есть доступ к [порталу Azure](https://portal.azure.com). Откройте параметры и проверьте, нет ли каких-либо ограничений.
* [Добавьте Application Insights в существующий проект](./asp-net.md). В обозревателе решений щелкните проект правой кнопкой мыши и выберите пункт "Добавить Application Insights".

## <a name="i-get-an-error-instrumentation-key-cannot-be-empty"></a><a name="emptykey"></a>Появляется сообщение об ошибке «ключ инструментирования не может быть пустым»
Вероятно, произошла ошибка при установке Application Insights или же адаптера ведения журнала.

В обозревателе решений щелкните проект правой кнопкой мыши и выберите **Application Insights > Настроить Application Insights**. Появится диалоговое окно с предложением войти в Azure и создать новый ресурс Application Insights или повторно использовать уже существующий.

## <a name="nuget-packages-are-missing-on-my-build-server"></a><a name="NuGetBuild"></a> На сервере сборки возникает ошибка NuGet package(s) are missing (Пакеты NuGet отсутствуют).
*Когда я выполняю отладку на компьютере для разработки, сборка выполняется без ошибок, но на сервере сборки отображается ошибка NuGet.*

См. сведения о [восстановлении пакетов NuGet](https://docs.nuget.org/Consume/Package-Restore) и [автоматическом восстановлении пакетов](https://docs.nuget.org/Consume/package-restore/migrating-to-automatic-package-restore).

## <a name="missing-menu-command-to-open-application-insights-from-visual-studio"></a>Отсутствует команда меню для открытия Application Insights из Visual Studio
*Когда я щелкаю правой кнопкой мыши свой проект в обозревателе решений, не отображаются команды Application Insights или команда "Открыть Application Insights".*

Вероятные причины:

* Ресурс Application Insights создан вручную, либо тип проекта не поддерживается средствами Application Insights.
* Аналитические средства для разработчиков отключены в Visual Studio.
* У вас установлена более старая версия Visual Studio, чем версия 2013 с обновлением 3.

Исправление:

* Обновите Visual Studio до версии 2013 с обновлением 3 или более поздней.
* Выберите элементы **Сервис**, **Расширения и обновления** и убедитесь, что пакет **Аналитические средства для разработчиков** установлен и включен. Если это так, щелкните пункт **Обновления** , чтобы проверить, доступно ли обновление.
* В обозревателе решений щелкните проект правой кнопкой мыши. Если вы видите команду **Application Insights > Настроить Application Insights**, используйте ее, чтобы подключить проект к ресурсу в службе Application Insights.

В противном случае тип проекта не будет поддерживаться напрямую средствами Developer Analytics Tools. Для просмотра данных телеметрии войдите на [портал Azure](https://portal.azure.com), выберите пункт Application Insights на панели навигации слева и выберите приложение.

## <a name="access-denied-on-opening-application-insights-from-visual-studio"></a>При открытии Application Insights из Visual Studio возникает ошибка "Доступ запрещен"
*При выборе команды меню "Открыть Application Insights" выполняется переход на портал Azure, но при этом появляется сообщение об ошибке "Доступ запрещен".*

При вводе учетных данных Майкрософт, которые вы в последний раз использовали в стандартном браузере, отсутствует доступ к [ресурсу, который был создан при добавлении Application Insights в это приложение](./asp-net.md). Есть две вероятные причины:

* Возможно, у вас не одна учетная запись Майкрософт, а несколько — например, рабочая и личная. В последний раз вы использовали в браузере по умолчанию не ту учетную запись, с помощью которой [добавляли Application Insights в проект](./asp-net.md).
  * Исправление: щелкните свое имя в верхней правой части окна браузера и выполните выход. Затем войдите с учетной записью, необходимой для доступа. Затем на панели навигации слева щелкните Application Insights и выберите приложение.
* Кто-то другой добавил Application Insights в проект и забыл предоставить вам [доступ к группе ресурсов](./resources-roles-access-control.md) , в которой он был создан.
  * Исправление: если этот пользователь указал учетную запись организации, возможно, он добавил вас в группу или предоставил вам индивидуальный доступ к группе ресурсов.

## <a name="asset-not-found-on-opening-application-insights-from-visual-studio"></a>При открытии Application Insights из Visual Studio возникает ошибка "Ресурс-контейнер не найден"
*При выборе команды меню "Открыть Application Insights" выполняется переход на портал Azure, но при этом появляется сообщение об ошибке "Ресурс-контейнер не найден".*

Вероятные причины:

* Ресурс Application Insights для приложения был удален.
* Ключ инструментирования был задан или изменен непосредственно в файле ApplicationInsights.config, при этом файл проекта не был обновлен.

Ключ инструментирования в файле ApplicationInsights.config контролирует, куда отправляются данные телеметрии. Строка в файле проекта контролирует, какой ресурс открывается при использовании команды в Visual Studio.

Исправление:

* В обозревателе решений щелкните проект правой кнопкой мыши и выберите пункт Application Insights, а затем — "Настроить Application Insights". В открывшемся диалоговом окне можно отправить данные телеметрии в существующий ресурс или создать новый. Или сделайте так:
* Откройте ресурс напрямую. Войдите на [портал Azure](https://portal.azure.com), выберите пункт Application Insights на панели навигации слева и выберите приложение.

## <a name="where-do-i-find-my-telemetry"></a>Где найти мои данные телеметрии
*После входа на [портал Microsoft Azure](https://portal.azure.com) я вижу панель мониторинга на домашней странице Azure. Итак, где найти мои данные Application Insights?*

* На панели навигации слева щелкните Application Insights и выберите имя приложения. Если проекты не отображаются, [добавьте или настройте Application Insights в своем веб-проекте](./asp-net.md).  
  Вы увидите несколько сводных диаграмм. Щелкните любую из них, чтобы увидеть более подробные данные.
* В Visual Studio во время отладки приложения нажмите кнопку Application Insights.

## <a name="no-server-data-or-no-data-at-all"></a><a name="q03"></a> Не отображаются данные сервера (или вообще какие-либо данные)
*После запуска приложения и открытия службы Application Insights в Microsoft Azure на всех диаграммах отображаются сообщения "Сведения о сборе…" или "Не настроено".* Или отображаются *только представление страницы и пользовательские данные, но нет данных сервера.*

* Запустите приложение в режиме отладки в Visual Studio (F5). Используйте приложение таким образом, чтобы создать данные телеметрии. Проверьте, можете ли вы просматривать события, зарегистрированные в окне вывода Visual Studio.  
  ![Снимок экрана, показывающий запуск приложения в режиме отладки в Visual Studio.](./media/asp-net-troubleshoot-no-data/output-window.png)
* На портале Application Insights откройте колонку [Diagnostic Search](./diagnostic-search.md)(Поиск по журналу диагностики). Как правило, сначала данные отображаются здесь.
* Нажмите кнопку «Обновить». Колонка периодически обновляется, однако ее можно обновить и вручную. При значительных значениях временного диапазона интервал обновления увеличивается.
* Проверьте соответствие ключей инструментирования. В основной колонке приложения на портале Application Insights в раскрывающемся списке **Основные компоненты** найдите **ключ инструментирования**. Затем в Visual Studio в своем проекте откройте файл ApplicationInsights.config и найдите `<instrumentationkey>`. Ключи должны совпадать. В противном случае выполните одно из таких действий:  
  * На портале щелкните Application Insights и найдите ресурс приложения с правильным ключом.
  * В обозревателе решений в Visual Studio щелкните проект правой кнопкой мыши и выберите пункт Application Insights, а затем щелкните "Настроить". Сбросьте настройки приложения, чтобы данные телеметрии отправлялись в нужный ресурс.
  * Если не удается найти совпадающие ключи, убедитесь, что для входа в Visual Studio и на портал используются одни и те же учетные данные.
* На [панели мониторинга на домашней странице Microsoft Azure](https://portal.azure.com)посмотрите на карту работоспособности службы. Если есть какие-либо предупреждения, дождитесь возвращения всех модулей в состояние ОК, затем закройте и заново откройте модуль приложения Application Insights.
* Кроме того, просмотрите [наш блог о состояниях](https://techcommunity.microsoft.com/t5/azure-monitor-status/bg-p/AzureMonitorStatusBlog).
* Возможно, вы написали код для [серверного пакета SDK](./api-custom-events-metrics.md), который мог изменить ключ инструментирования в экземплярах `TelemetryClient` или в `TelemetryContext`. Или вы создали [конфигурацию фильтра или выборки](./api-filtering-sampling.md) , которая отфильтровывает слишком много данных.
* Если вы вносили изменения в файл ApplicationInsights.config, внимательно проверьте конфигурацию [TelemetryInitializer и TelemetryProcessor](./api-filtering-sampling.md). Тип или параметр с неправильным именем может стать причиной неотправки данных пакетом SDK.

## <a name="no-data-on-page-views-browsers-usage"></a><a name="q04"></a>Нет данных об использовании, браузере и просмотрах страниц
*Данные отображаются на диаграммах "Время ответа сервера" и "Запросы сервера", но отсутствуют в колонках "Время загрузки страницы", "Браузер" или "Использование".*

Данные поступают из сценариев на веб-страницах. 

* Если вы добавили Application Insights в существующий веб-проект, [нужно добавить сценарии вручную](./javascript.md).
* Убедитесь, что Internet Explorer не отображает ваш сайт в режиме совместимости.
* Воспользуйтесь функцией отладки в браузере (в некоторых браузерах нужно нажать клавишу F12 и выбрать "Сеть"), чтобы убедиться, что данные отправляются по адресу `dc.services.visualstudio.com`.

## <a name="no-dependency-or-exception-data"></a>Нет данных о зависимостях или исключениях
См. статьи, посвященные [телеметрии зависимостей](./asp-net-dependencies.md) и [телеметрии исключений](asp-net-exceptions.md).

## <a name="no-performance-data"></a>Нет данных о производительности
Данные о производительности (ЦП, скорость ввода-вывода и т. д.) доступны для [веб-служб Java](./java-collectd.md), [классических приложений Windows](./windows-desktop.md), [веб-приложений и служб IIS, если установлен монитор состояния](./monitor-performance-live-website-now.md), а также [облачных служб Azure](./app-insights-overview.md). Их можно найти, открыв раздел "Параметры" и выбрав "Серверы".

## <a name="no-server-data-since-i-published-the-app-to-my-server"></a>После публикации приложения на сервере данные на нем отсутствуют
* Проверьте, что вы фактически скопировали все библиотеки DLL Microsoft. ApplicationInsights на сервер вместе с Microsoft.Diagnostics.Instrumentation.Extensions.Intercept.dll
* В брандмауэре может потребоваться [открыть некоторые порты TCP](./ip-addresses.md).
* Если для отправки из корпоративной сети необходимо использовать прокси-сервер, установите значение [defaultProxy](/previous-versions/dotnet/netframework-1.1/aa903360(v=vs.71)) в файле Web.config.
* Для Windows Server 2008. Убедитесь, что установлены следующие обновления: [KB2468871](https://support.microsoft.com/kb/2468871), [KB2533523](https://support.microsoft.com/kb/2533523), [KB2600217](https://web.archive.org/web/20150129090641/http://support.microsoft.com/kb/2600217).

## <a name="i-used-to-see-data-but-it-has-stopped"></a>Ранее видимые данные перестали отображаться
* Вы достигли месячной квоты точек данных? Чтобы выяснить это, последовательно выберите «Параметры», «Квота и расценки». Если вы достигли квоты, вы можете изменить свой тарифный план или заплатить за дополнительную емкость. См. [таблицу цен](https://azure.microsoft.com/pricing/details/application-insights/).

## <a name="i-dont-see-all-the-data-im-expecting"></a>Не отображаются все данные, которые ожидалось увидеть
Если ваше приложение отправляет большие объемы данных, а вы используете Application Insights SDK для ASP.NET версии 2.0.0-beta3 или более поздней, может сработать функция [адаптивной выборки](./sampling.md) и будет отправлена только часть данных телеметрии.

Ее можно отключить, но это не рекомендуется. Выборка разработана таким образом, чтобы связанные данные телеметрии правильно передавались для диагностических целей.

## <a name="client-ip-address-is-0000"></a>IP-адрес клиента — 0.0.0.0

О прекращении ведения журналов с IP-адресами клиентов было объявлено 5 февраля 2018 г. Это не влияет на географическое расположение.

> [!NOTE]
> Если вам требуются первые три октета IP-адресов, можно использовать [инициализатор телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) для добавления пользовательского свойства.
> Это изменение не влияет на данные, собранные до 5 февраля 2018 г.

## <a name="wrong-geographical-data-in-user-telemetry"></a>Неправильные географические данные в телеметрии пользователя
Город, область и страна определяются по IP-адресу и не всегда являются точными. Эти IP-адреса сначала обрабатываются для расположения, а затем изменяются на IP-адрес 0.0.0.0 для хранения.

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>Исключение "метод не найден" при выполнении в облачных службах Azure
Выполняется сборка для .NET 4.6? Версия 4.6 не поддерживается автоматически в ролях облачных служб Azure. [установите версию 4.6 в каждой роли](../../cloud-services/cloud-services-dotnet-install-dotnet.md) .

## <a name="troubleshooting-logs"></a>Журналы для устранения неполадок

Следуйте этим инструкциям, чтобы записать журналы устранения неполадок для своей платформы.

### <a name="net-framework"></a>.NET Framework

1. Установите пакет [Microsoft.AspNet.ApplicationInsights.HostingStartup](https://www.nuget.org/packages/Microsoft.AspNet.ApplicationInsights.HostingStartup) из NuGet. Устанавливаемая версия должна соответствовать текущей установленной версии `Microsoft.ApplicationInsighs`

2. Измените файл applicationinsights.config, чтобы включить следующее:

    ```xml
    <TelemetryModules>
      <Add Type="Microsoft.ApplicationInsights.Extensibility.Implementation.Tracing.FileDiagnosticsTelemetryModule, Microsoft.ApplicationInsights">
        <Severity>Verbose</Severity>
        <LogFileName>mylog.txt</LogFileName>
        <LogFilePath>C:\\SDKLOGS</LogFilePath>
      </Add>
    </TelemetryModules>
    ```
    У приложения должны быть разрешения на запись в настроенное расположение.

3. Перезапустите процесс таким образом, чтобы эти новые параметры были выбраны пакетом SDK.

4. Когда вы закончите, отмените эти изменения.

### <a name="net-core"></a>.NET Core

1. Установите [пакет nuget Application Insights SDK для пакета ASP.NET Core](https://nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) из NuGet. Устанавливаемая версия должна соответствовать текущей установленной версии `Microsoft.ApplicationInsights` .

   Последняя версия Microsoft. ApplicationInsights. AspNetCore — 2.14.0, и она ссылается на Microsoft. ApplicationInsights версии 2.14.0. Таким образом, устанавливаемая версия Microsoft. ApplicationInsights. AspNetCore должна быть 2.14.0.

2. Изменить `ConfigureServices` метод в вашем `Startup.cs` классе:

    ```csharp
    services.AddSingleton<ITelemetryModule, FileDiagnosticsTelemetryModule>();
    services.ConfigureTelemetryModule<FileDiagnosticsTelemetryModule>( (module, options) => {
        module.LogFilePath = "C:\\SDKLOGS";
        module.LogFileName = "mylog.txt";
        module.Severity = "Verbose";
    } );
    ```
    У приложения должны быть разрешения на запись в настроенное расположение.

3. Перезапустите процесс таким образом, чтобы эти новые параметры были выбраны пакетом SDK.

4. Когда вы закончите, отмените эти изменения.


## <a name="collect-logs-with-perfview"></a><a name="PerfView"></a> Получение журналов с помощью PerfView
[PerfView](https://github.com/Microsoft/perfview) — это бесплатное средство диагностики и анализа производительности, помогающее изолировать проблемы ЦП, памяти и другие, собирая и визуализируя диагностические сведения из многих источников.

Пакет SDK Application Insights ведет журналы самостоятельного устранения неполадок EventSource, которые можно записать с помощью PerfView.

Чтобы получить журналы, скачайте PerfView и выполните следующую команду.
```cmd
PerfView.exe collect -MaxCollectSec:300 -NoGui /onlyProviders=*Microsoft-ApplicationInsights-Core,*Microsoft-ApplicationInsights-Data,*Microsoft-ApplicationInsights-WindowsServer-TelemetryChannel,*Microsoft-ApplicationInsights-Extensibility-AppMapCorrelation-Dependency,*Microsoft-ApplicationInsights-Extensibility-AppMapCorrelation-Web,*Microsoft-ApplicationInsights-Extensibility-DependencyCollector,*Microsoft-ApplicationInsights-Extensibility-HostingStartup,*Microsoft-ApplicationInsights-Extensibility-PerformanceCollector,*Microsoft-ApplicationInsights-Extensibility-EventCounterCollector,*Microsoft-ApplicationInsights-Extensibility-PerformanceCollector-QuickPulse,*Microsoft-ApplicationInsights-Extensibility-Web,*Microsoft-ApplicationInsights-Extensibility-WindowsServer,*Microsoft-ApplicationInsights-WindowsServer-Core,*Microsoft-ApplicationInsights-LoggerProvider,*Microsoft-ApplicationInsights-Extensibility-EventSourceListener,*Microsoft-ApplicationInsights-AspNetCore
```

Эти параметры можно менять по мере необходимости.
- **MaxCollectSec**. Задайте этот параметр, чтобы предотвратить постоянное выполнение PerfView и соответствующее влияние на производительность сервера.
- **OnlyProviders**. Установите этот параметр, чтобы получать только журналы из пакета SDK. Этот список можно настроить в соответствии с конкретным расследованием. 
- **NoGui**. Установите этот параметр, чтобы получать журналы без графического пользовательского интерфейса.


Дополнительные сведения см.:
- [Запись трассировок производительности с помощью PerfView](https://github.com/dotnet/roslyn/wiki/Recording-performance-traces-with-PerfView).
- [Источники событий в Application Insights](https://github.com/microsoft/ApplicationInsights-dotnet/tree/develop/examples/ETW)

## <a name="collect-logs-with-dotnet-trace"></a>Получение журналов с помощью dotnet-trace

Альтернативный способ сбора журналов для устранения неполадок, который может быть особенно полезен для сред под управлением Linux, — это [`dotnet-trace`](/dotnet/core/diagnostics/dotnet-trace).

```bash
dotnet-trace collect --process-id <PID> --providers Microsoft-ApplicationInsights-Core,Microsoft-ApplicationInsights-Data,Microsoft-ApplicationInsights-WindowsServer-TelemetryChannel,Microsoft-ApplicationInsights-Extensibility-AppMapCorrelation-Dependency,Microsoft-ApplicationInsights-Extensibility-AppMapCorrelation-Web,Microsoft-ApplicationInsights-Extensibility-DependencyCollector,Microsoft-ApplicationInsights-Extensibility-HostingStartup,Microsoft-ApplicationInsights-Extensibility-PerformanceCollector,Microsoft-ApplicationInsights-Extensibility-EventCounterCollector,Microsoft-ApplicationInsights-Extensibility-PerformanceCollector-QuickPulse,Microsoft-ApplicationInsights-Extensibility-Web,Microsoft-ApplicationInsights-Extensibility-WindowsServer,Microsoft-ApplicationInsights-WindowsServer-Core,Microsoft-ApplicationInsights-LoggerProvider,Microsoft-ApplicationInsights-Extensibility-EventSourceListener,Microsoft-ApplicationInsights-AspNetCore
```

## <a name="how-to-remove-application-insights"></a>Как удалить Application Insights

Узнайте, как удалить Application Insights в Visual Studio, выполнив действия, описанные в [статье об удалении](./remove-application-insights.md).

## <a name="still-not-working"></a>По-прежнему не работает...
* [Страница Майкрософт с вопросами и ответами об Application Insights](/answers/topics/azure-monitor.html)

