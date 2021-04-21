---
title: Использование Application Insights для облачных служб Azure | Документация Майкрософт
description: Эффективное отслеживание веб-ролей и рабочих ролей с помощью Application Insights
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 09/05/2018
ms.openlocfilehash: 264d6d4b0b397a29b5dc1db4bb299297c1e30584
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103419241"
---
# <a name="application-insights-for-azure-cloud-services"></a>Использование Application Insights для облачных служб Azure
С помощью [Application Insights][start] можно отслеживать [приложения облачной службы Azure](https://azure.microsoft.com/services/cloud-services/) на предмет доступности, производительности, сбоев и использования, объединяя данные из пакета SDK Application Insights с данными [системы диагностики Azure](../agents/diagnostics-extension-overview.md) из облачных служб. Благодаря получаемым данным о производительности и эффективности работы приложения на практике вы можете принимать осознанные решения о направлении разработки в каждом жизненном цикле.

![Панель мониторинга с общими сведениями](./media/cloudservices/overview-graphs.png)

## <a name="prerequisites"></a>Предварительные требования
Для этого потребуются следующие компоненты.

* Подписка [Azure](https://azure.com). Войдите с помощью учетной записи Майкрософт для Windows, XBox Live или других облачных служб Майкрософт. 
* Инструменты Microsoft Azure 2.9 или более поздней версии.
* Developer Analytics Tools 7.10 или более поздней версии.

## <a name="get-started-quickly"></a>Быстрое начало работы
Самый быстрый и простой способ настроить мониторинг облачной службы с помощью Application Insights — выбрать этот вариант при публикации своей службы в Azure.

![Пример страницы "Параметры диагностики"](./media/cloudservices/azure-cloud-application-insights.png)

Этот вариант позволяет инструментировать приложение во время выполнения, что предоставляет все данные телеметрии, необходимые для мониторинга запросов, исключений и зависимостей в веб-роли. Вы также можете отслеживать счетчики производительности рабочих ролей. Все диагностические трассировки, создаваемые вашим приложением, также отправляются в Application Insights.

Возможно, вас удовлетворит только этот вариант. 

Следующие шаги — [просмотр метрик приложения](../essentials/metrics-charts.md), [отправка запроса данных с помощью Analytics](../logs/log-query-overview.md). 

Чтобы отслеживать производительность в браузере, вы можете [настроить тесты доступности](./monitor-web-app-availability.md) и [добавить код в веб-страницы](./javascript.md).

В следующих разделах обсуждаются дополнительные возможности:

* отправка данных от разных компонентов и конфигураций сборки в отдельные ресурсы;
* добавить пользовательские данные телеметрии из своего приложения.

## <a name="sample-app-instrumented-with-application-insights"></a>Пример приложения, инструментированного с помощью Application Insights
В этом [примере приложения](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService) служба Application Insights добавлена в облачную службу с двумя рабочими ролями, размещенными в Azure. 

В следующем разделе описано, как адаптировать собственный проект облачной службы аналогичным образом.

## <a name="plan-resources-and-resource-groups"></a>Планирование ресурсов и групп ресурсов
Данные телеметрии из приложения сохраняются, анализируются и отображаются в ресурсе Azure типа Application Insights. 

Каждый ресурс относится к группе ресурсов. Группы ресурсов используются для управления затратами, предоставления доступа участникам команды и развертывания обновлений в отдельной согласованной транзакции. Например, вы можете [написать скрипт для развертывания](../../azure-resource-manager/templates/deploy-powershell.md) облачной службы Azure и ресурсов мониторинга Application Insights в одной операции.

### <a name="resources-for-components"></a>Ресурсы для компонентов

Рекомендуется [добавить свойство измерения к каждому элементу телеметрии](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer), определяющему его исходную роль. При таком подходе диаграммы метрик (например, исключений) обычно отображают агрегированные данные для нескольких ролей. При необходимости вы можете сегментировать диаграмму по идентификатору роли. Поиск также можно фильтровать по одному измерению. Этот вариант делает более удобным одновременный просмотр информации, но может привести к некоторой путанице с ролями.

Данные телеметрии браузера обычно добавляются в тот же ресурс, что и данные соответствующей серверной веб-роли.

Поместите ресурсы Application Insights для разных компонентов в одну группу ресурсов. Такой подход позволяет легко управлять ими одновременно. 

### <a name="separate-development-test-and-production"></a>Разделение среды разработки, тестирования и рабочей среды
При разработке пользовательских событий для следующей версии компонента в то время, когда предыдущая версия еще активна, необходимо отправлять данные телеметрии приложения в отдельный ресурс Application Insights.  В противном случае будет трудно найти данные телеметрии тестирования среди всего трафика, поступающего от активного сайта.

Чтобы избежать такой ситуации, создайте отдельные ресурсы для каждой конфигурации сборки или метки (разработка, тестирование, эксплуатация и т. д.) в своей системе. Поместите ресурсы для каждой конфигурации сборки в отдельную группу ресурсов. 

Чтобы отправлять данные телеметрии в соответствующие ресурсы, можно настроить пакет SDK для Application Insights, чтобы он использовал разные ключи инструментирования в зависимости от конфигурации сборки. 

Узнайте, как [динамически задать ключ инструментирования](./separate-resources.md#dynamic-ikey) для различных этапов. 

## <a name="create-an-application-insights-resource-for-each-role"></a>Создание ресурса Application Insights для каждой роли

Если вы решили создать отдельный ресурс для каждой роли и, возможно, отдельный набор для каждой конфигурации сборки, проще это сделать на портале Application Insights. Если создается много ресурсов, можно [автоматизировать этот процесс](./powershell.md).

1. На [портале Azure][portal] выберите **Создать** > **Службы разработки** > **Application Insights**.  

    ![Панель Application Insights](./media/cloudservices/01-new.png)

1. В раскрывающемся списке **Тип приложения** выберите **Веб-приложение ASP.NET**.

Каждый ресурс идентифицируется по ключу инструментирования. Он может понадобиться позже, если вы захотите вручную настроить или проверить конфигурацию пакета SDK.


## <a name="set-up-azure-diagnostics-for-each-role"></a>Настройка системы диагностики Azure для каждой роли
Настройте мониторинг приложения с помощью Application Insights. Для веб-ролей этот вариант поддерживает мониторинг производительности, создание оповещений и диагностику, а также анализ сведений об использовании. Для других ролей можно искать и отслеживать диагностические данные Azure, например о событиях перезапуска, показаниях счетчиков производительности и вызовах System.Diagnostics.Trace. 

1. В обозревателе решений Visual Studio в разделе **\<YourCloudService>**  > **Роли** откройте свойства каждой роли.

1. В разделе **Конфигурация** установите флажок **Отправка диагностических данных в Application Insights** и выберите созданный ранее ресурс Application Insights.

Если вы решили использовать отдельный ресурс Application Insights для каждой конфигурации сборки, сначала выберите эту конфигурацию.

![Настройка Application Insights](./media/cloudservices/configure-azure-diagnostics.png)

Это приведет к добавлению ключей инструментирования Application Insights в файлы *ServiceConfiguration.\*.cscfg*. Вот [пример кода](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/AzureEmailService/ServiceConfiguration.Cloud.cscfg).

Если требуется изменить уровень диагностических сведений, отправляемых в Application Insights, это можно сделать, [напрямую изменив файлы *.cscfg*](../agents/diagnostics-extension-to-application-insights.md).

## <a name="install-the-sdk-in-each-project"></a><a name="sdk"></a>Установка пакета SDK в каждый проект
Этот вариант добавляет пользовательскую бизнес-телеметрию в любую роль. Так вы получите подробные аналитические данные об использовании и производительности приложения.

В Visual Studio настройте пакет SDK для Application Insights для каждого проекта облачного приложения.

1. Чтобы настроить **веб-роли**, щелкните проект правой кнопкой мыши и выберите **Настроить Application Insights** или **Добавить > Телеметрия Application Insights**.

1. Чтобы настроить **рабочие ролей**, сделайте следующее: 

    а. Щелкните проект правой кнопкой мыши и выберите **Управление пакетами Nuget**.

    b. Добавьте [Application Insights для Windows Servers](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/).

1. Чтобы настроить пакет SDK для отправки данных в ресурс Application Insights, сделайте следующее:

    а. В соответствующей функции запуска задайте ключ инструментирования, указанный в параметре конфигурации из файла *.cscfg*.
 
    ```csharp
        TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
        configuration.InstrumentationKey = RoleEnvironment.GetConfigurationSettingValue("APPINSIGHTS_INSTRUMENTATIONKEY");
        var telemetryClient = new TelemetryClient(configuration);
    ```
   
    b. Повторите предыдущий шаг (a) для каждой роли в приложении. См. указанные ниже примеры.
   
    * [Веб-роль](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Global.asax.cs#L27)
    * [Рабочая роль](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L232)
    * [Веб-страницы](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Views/Shared/_Layout.cshtml#L13) 

1. Настройте для файла *ApplicationInsights.config* копирование в выходной каталог.

   В сообщении в файле *.config* будет предложено поместить в него ключ инструментирования. Но для облачных приложений лучше задать его в файле *.cscfg*. Такой подход позволит правильно идентифицировать роли на портале.

## <a name="set-up-status-monitor-to-collect-full-sql-queries-optional"></a>Настройка монитора состояния для получения полных SQL-запросов (необязательно)

Этот шаг необходим только в том случае, если требуется захватить полные SQL-запросы на платформа .NET Framework. 

1. В файле `\*.csdef` добавьте [задачу запуска](../../cloud-services/cloud-services-startup-tasks.md) для каждой роли, аналогичной 

    ```xml
    <Startup>
      <Task commandLine="AppInsightsAgent\InstallAgent.bat" executionContext="elevated" taskType="simple">
        <Environment>
          <Variable name="ApplicationInsightsAgent.DownloadLink" value="http://go.microsoft.com/fwlink/?LinkID=522371" />
          <Variable name="RoleEnvironment.IsEmulated">
            <RoleInstanceValue xpath="/RoleEnvironment/Deployment/@emulated" />
          </Variable>
        </Environment>
      </Task>
    </Startup>
    ```
    
2. Скачайте [InstallAgent.bat](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/AppInsightsAgent/InstallAgent.bat) и [InstallAgent.ps1](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/AppInsightsAgent/InstallAgent.ps1), вставьте их в папку `AppInsightsAgent` в каждом проекте роли. Не забудьте скопировать их в выходной каталог с помощью свойств файла Visual Studio или скриптов сборки.

3. Добавьте переменные среды для всех рабочих ролей: 

    ```xml
      <Environment>
        <Variable name="COR_ENABLE_PROFILING" value="1" />
        <Variable name="COR_PROFILER" value="{324F817A-7420-4E6D-B3C1-143FBED6D855}" />
        <Variable name="MicrosoftInstrumentationEngine_Host" value="{CA487940-57D2-10BF-11B2-A3AD5A13CBC0}" />
      </Environment>
    ```
    
## <a name="run-and-publish-the-app"></a>Запуск и публикация приложения

1. Запустите приложение и войдите в Azure. 

1. Откройте созданные ранее ресурсы Application Insights.

   В разделе [Поиск][diagnostic] отображаются отдельные точки данных, а в [обозревателе метрик](../essentials/metrics-charts.md) — агрегированные данные.

1. Добавьте дополнительные данные телеметрии (как описано в следующем разделе), а затем опубликуйте приложение для оперативного получения данных диагностики и сведений об использовании. 

Если данные не отображаются, сделайте следующее:

1. Откройте плитку [Поиск][diagnostic], чтобы просмотреть отдельные события.
1. В приложении откройте несколько разных страниц, чтобы создать некоторый объем данных телеметрии.
1. Подождите несколько секунд и щелкните **Обновить**.  

Дополнительные сведения см. в [руководстве по устранению неполадок][qna].

## <a name="view-azure-diagnostics-events"></a>Просмотр событий диагностики Azure
Источники данных [диагностики Azure](../agents/diagnostics-extension-overview.md) в Application Insights можно найти в следующих расположениях:

* Счетчики производительности отображаются в виде настраиваемых метрик. 
* Журналы событий Windows отображаются в виде трассировок и настраиваемых событий.
* Журналы приложений, журналы трассировки событий Windows и все журналы инфраструктуры диагностики отображаются в виде трассировок.

Чтобы просмотреть счетчики производительности и счетчики событий, откройте [обозреватель метрик](../essentials/metrics-charts.md) и добавьте следующую диаграмму:

![Данные системы диагностики Azure](./media/cloudservices/23-wad.png)

Для поиска по журналам трассировки, отправляемым системой диагностики Azure, используйте [Поиск](./diagnostic-search.md) или [Запрос аналитики](../logs/log-analytics-tutorial.md). Предположим, что возникло необработанное исключение, которое вызвало сбой и перезапуск роли. Эта информация будет отображена в канале приложения журнала событий Windows. Можно использовать Поиск, чтобы просмотреть ошибку в журнале событий Windows и получить полную трассировку стека для этого исключения. Такой вариант поможет вам найти причину проблемы.

![Поиск данных системы диагностики Azure](./media/cloudservices/25-wad.png)

## <a name="more-telemetry"></a>Дополнительные данные телеметрии
В следующих разделах описано, как получать дополнительные данные телеметрии по разным аспектам приложения.

## <a name="track-requests-from-worker-roles"></a>Отслеживание запросов из рабочих ролей
В веб-ролях модуль запросов собирает данные об HTTP-запросах автоматически. Примеры переопределения поведения коллекции по умолчанию см. на странице с [примером роли MVCWebRole](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/MvcWebRole). 

Производительность вызовов для рабочих ролей можно регистрировать, отслеживая их таким же образом, как запросы HTTP. В Application Insights тип телеметрии запроса определяет единицу работы для указанного сервера, которую можно учесть и которая может быть независимо выполнена или не выполнена. HTTP-запросы автоматически перехватываются пакетом SDK, но вы можете добавить собственный код для отслеживания запросов к рабочим ролям.

Ознакомьтесь с двумя примерами рабочих ролей, которые инструментированы для отправки отчетов по запросам: 
* [WorkerRoleA](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleA)
* [WorkerRoleB](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService/WorkerRoleB)

## <a name="exceptions"></a>Исключения
Сведения о сборе необработанных исключений от веб-приложений разных типов см. в руководстве по [настройке Application Insights и диагностике исключений](./asp-net-exceptions.md).

Образец веб-роли имеет контроллеры MVC5 и веб-API 2. Необработанные исключения, поступающие из них, регистрируются следующими обработчиками:

* объектом [AiHandleErrorAttribute](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiHandleErrorAttribute.cs), [настроенным для контроллеров MVC5](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/FilterConfig.cs#L12); 
* объектом [AiWebApiExceptionLogger](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/Telemetry/AiWebApiExceptionLogger.cs), [настроенным для контроллеров веб-API 2](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/MvcWebRole/App_Start/WebApiConfig.cs#L25). 

Для рабочих ролей есть два способа отслеживать исключения.

* TrackException(ex)
* Если вы добавили пакет NuGet прослушивателя трассировки Application Insights, вы можете [вносить исключения в журнал с помощью System.Diagnostics.Trace](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L107).

## <a name="performance-counters"></a>Счетчики производительности
По умолчанию собираются приведенные ниже счетчики.

* \Process(??APP_WIN32_PROC??)\% Процессорное время
* \Память\доступные байты
* \.NET CLR Exceptions(??APP_CLR_PROC??)\# Исключений в секунду
* \Process(??APP_WIN32_PROC??)\Байт исключительного пользования
* \Process(??APP_WIN32_PROC??)\I/O — обмен данными, байт в секунду
* \Процессор (_общий объем ресурсов)\% загруженности процессора

Эти счетчики также собираются для веб-ролей.

* \ASP.NET Applications(??APP_W3SVC_PROC??)\Запросов в секунду
* \ASP.NET Applications(??APP_W3SVC_PROC??)\Время выполнения запросов
* \ASP.NET Applications(??APP_W3SVC_PROC??)\Запросы в очереди приложений

Можно указать дополнительные пользовательские или другие счетчики производительности Windows, изменив файл [*ApplicationInsights.config*](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/ApplicationInsights.config#L14).

  ![Счетчики производительности](./media/cloudservices/002-servers.png)

## <a name="correlated-telemetry-for-worker-roles"></a>Коррелированная телеметрия для рабочих ролей
Вы можете расширить возможности диагностики, просматривая причины неудавшихся запросов или запросов с высокой задержкой. При работе с веб-ролями пакет SDK автоматически настраивает корреляцию между связанными данными телеметрии. 

Для рабочих ролей вы можете использовать пользовательский инициализатор телеметрии, чтобы задать атрибут общего контекста Operation.Id для всех данных телеметрии. Это позволит с первого взгляда понять, связаны ли сбой или задержка с некоторой зависимостью или ошибкой в коде. 

Вот как это сделать.

* [Настройте идентификатор корреляции в объекте CallContext](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L36). В нашем примере мы используем идентификатор запроса как идентификатор корреляции.
* Добавьте пользовательскую реализацию TelemetryInitializer, которая сохраняет в Operation.Id заданное ранее значение correlationId. Пример: [ItemCorrelationTelemetryInitializer](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/Telemetry/ItemCorrelationTelemetryInitializer.cs#L13).
* Добавьте пользовательский инициализатор телеметрии. Это можно сделать в файле *ApplicationInsights.config* или [в коде](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/Samples/AzureEmailService/WorkerRoleA/WorkerRoleA.cs#L233).

## <a name="client-telemetry"></a>Данные телеметрии клиента
[Добавьте пакет SDK JavaScript на веб-страницы][client], чтобы получать браузерные данные телеметрии, такие как число просмотров страниц, время загрузки страницы, исключения скриптов, и записывать пользовательскую телеметрию в скрипты страниц.

## <a name="availability-tests"></a>Тесты доступности
[Настройте веб-тесты][availability], которые обеспечат работоспособность приложения и его правильное реагирование на запросы.

## <a name="display-everything-together"></a>Отображение всей информации вместе
Чтобы получить общее представление о системе, можно отобразить ключевые диаграммы мониторинга на одной [панели мониторинга](./overview-dashboard.md). Например, можно закрепить количество запросов и сбоев для каждой роли. 

Если в системе используются другие службы Azure, такие как Stream Analytics, добавьте и их диаграммы мониторинга. 

Если у вас есть клиентское мобильное приложение, воспользуйтесь [Центром приложений](../app/mobile-center-quickstart.md). Создайте запросы в [Аналитике](../logs/log-query-overview.md) для отображения числа событий и закрепите эти показатели на панели мониторинга.

## <a name="example"></a>Пример
[примере](https://github.com/MohanGsk/ApplicationInsights-Home/tree/master/Samples/AzureEmailService) отслеживается служба, которая имеет веб-роль и две рабочие роли.

## <a name="exception-method-not-found-on-running-in-azure-cloud-services"></a>Исключение "метод не найден" при выполнении в облачных службах Azure
Выполняется сборка для .NET 4.6? .NET 4.6 не поддерживается автоматически в ролях облачных служб Azure. [Установите .NET 4.6 в каждой роли](../../cloud-services/cloud-services-dotnet-install-dotnet.md), прежде чем выполнять приложение.

## <a name="video"></a>Видеоролик

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/100/player]

## <a name="next-steps"></a>Дальнейшие действия
* [Настройка системы диагностики Azure для отправки данных в Application Insights](../agents/diagnostics-extension-to-application-insights.md)
* [Создание ресурсов Application Insights с помощью PowerShell](./powershell.md)
* [Настройка Application Insights для веб-приложения Azure с помощью PowerShell](./powershell-azure-diagnostics.md)
* [Функции Azure](https://github.com/christopheranderson/azure-functions-app-insights-sample)

[api]: ./api-custom-events-metrics.md
[availability]: ./monitor-web-app-availability.md
[azure]: ./app-insights-overview.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[netlogs]: ./asp-net-trace-logs.md
[portal]: https://portal.azure.com/
[qna]: ../faq.md
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md