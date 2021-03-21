---
title: Выбор архитектуры для развертывания Application Insights — один или несколько ресурсов
description: Отправка телеметрии к различным ресурсам для меток разработки, тестирования и эксплуатации.
ms.topic: conceptual
ms.date: 05/11/2020
ms.openlocfilehash: 9a60981e692a45dd3630073300b206289cfd2a30
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102424671"
---
# <a name="how-many-application-insights-resources-should-i-deploy"></a>Выбор количества ресурсов Application Insights для развертывания

Разрабатывая следующую версию веб-приложения, вы не хотели бы путать данные телеметрии [Application Insights](../../azure-monitor/app/app-insights-overview.md), полученные для новой версии, с данными уже выпущенной. Чтобы избежать путаницы, отправляйте данные телеметрии разных стадий разработки в отдельные ресурсы Application Insights, используя отдельные ключи инструментирования. Чтобы облегчить изменение ключа инструментирования в зависимости от стадии разработки, вы можете задать ключи инструментирования в коде, а не в файле конфигурации.

(Если вы используете облачную службу Azure, есть [другой метод настройки отдельных ключей инструментирования ikey](../../azure-monitor/app/cloudservices.md).)

## <a name="about-resources-and-instrumentation-keys"></a>Ресурсы и ключи инструментирования

При настройке мониторинга Application Insights для веб-приложения вы создаете *ресурс* Application Insights в Microsoft Azure. Вы открываете этот ресурс на портале Azure для просмотра и анализа телеметрии, собранной из приложения. Каждый ресурс идентифицируется с помощью *ключа инструментирования* (ikey). При установке пакета Application Insights для мониторинга приложения вы можете настроить для него ключ инструментирования. Так ему будет известно, куда отправлять телеметрию.

Каждый ресурс Application Insights в стандартной конфигурации поддерживает метрики. Если компоненты, никак не связанные друг с другом, будут отправлять отчеты в один ресурс Application Insights, данные этих метрик на панели мониторинга или для оповещений могут оказаться бессмысленными.

### <a name="when-to-use-a-single-application-insights-resource"></a>Когда лучше использовать один ресурс Application Insights

-   Для компонентов приложения, которые развертываются совместно. Обычно это означает, что они разрабатываются одной командой и управляются одним подмножеством пользователей DevOps или ITOps.
-   Если есть смысл усреднять на панели мониторинга такие ключевые показатели эффективности (KPI), как длительность ответа, количество сбоев и т. д. (вы можете настроить сегментирование по имени роли в интерфейсе обозревателя метрик).
-   Если нет необходимости управлять управлением доступом на основе ролей Azure (Azure RBAC) по-разному между компонентами приложения.
-   Если не требуется применять разные критерии оповещения по метрикам для разных компонентов.
-   Если не требуется управлять непрерывным экспортом разными способами для разных компонентов.
-   Если не требуется управлять выставлением счетов и (или) квотами разными способами для разных компонентов.
-   Если вас устраивает наличие одного ключа API с одинаковым доступом к данным от всех компонентов. И при этом вам достаточно десяти ключей API на все компоненты.
-   Если вас устраивают одинаковые параметры интеллектуального обнаружения и интеграции рабочих элементов для всех ролей.

> [!NOTE]
> Если вы хотите консолидировать несколько Application Insightsных ресурсов, вы можете указать существующие компоненты приложения на новый, Объединенный Application Insights ресурс. Данные телеметрии, хранящиеся в старом ресурсе, не будут переданы в новый ресурс, поэтому удалите старый ресурс только при наличии достаточного количества данных телеметрии в новом ресурсе для обеспечения непрерывности бизнес-процессов.

### <a name="other-things-to-keep-in-mind"></a>Что еще нужно учитывать

-   Возможно, потребуется добавить пользовательский код для настройки значимых значений в атрибуте [Cloud_RoleName](./app-map.md?tabs=net#set-or-override-cloud-role-name). Если значения этого атрибута будут неверными, интерфейс портала *НЕ БУДЕТ* работать.
- Для приложений Service Fabric и классических облачных служб пакет SDK автоматически считывает данные из среды роли Azure и настраивает эти значения. Но для приложений всех других типов вам придется, скорее всего, задать его явным образом.
-   Интерфейс Live Metrics не поддерживает разделение по именам ролей.

## <a name="dynamic-instrumentation-key"></a><a name="dynamic-ikey"></a> Динамический ключ инструментирования

Чтобы упростить изменение iKey по мере перемещения кода между стадиями производства, следует динамически ссылаться на ключ в коде вместо использования жестко закодированного или статического значения.

Задайте ключ в методе инициализации, таком как global.aspx.cs, в службе ASP.NET:

```csharp
protected void Application_Start()
{
  Microsoft.ApplicationInsights.Extensibility.
    TelemetryConfiguration.Active.InstrumentationKey = 
      // - for example -
      WebConfigurationManager.AppSettings["ikey"];
  ...
```

В этом примере ключи инструментирования (ikey) для различных ресурсов приводятся в различных версиях файла веб-конфигурации. При смене файла веб-конфигурации, что можно сделать в рамках сценария выпуска, сменится и целевой ресурс.

### <a name="web-pages"></a>Веб-страницы
Ключ инструментирования iKey также используется для веб-страниц приложения в [скрипте, который вы получили на панели быстрого запуска](../../azure-monitor/app/javascript.md). Вместо того чтобы вставлять его в код сценария напрямую, генерируйте его из состояния сервера. Например, в приложении ASP.NET:

```javascript
<script type="text/javascript">
// Standard Application Insights web page script:
var appInsights = window.appInsights || function(config){ ...
// Modify this part:
}({instrumentationKey:  
  // Generate from server property:
  "@Microsoft.ApplicationInsights.Extensibility.
     TelemetryConfiguration.Active.InstrumentationKey"
  }
 )
//...
```

## <a name="create-additional-application-insights-resources"></a>Создание дополнительных ресурсов Application Insights

Создание ресурса Application Insights описано в [этом руководстве](./create-new-resource.md).

### <a name="getting-the-instrumentation-key"></a>Получение ключа инструментирования
Ключ инструментирования идентифицирует созданный вами ресурс.

Вам потребуются ключи инструментирования всех ресурсов, в которые приложение будет отправлять данные.

## <a name="filter-on-build-number"></a>Фильтрация по номеру сборки
При публикации новой версии приложения имеет смысл отделить телеметрию от других сборок.

Для этого можно настроить свойство "Версия приложения" для фильтрации результатов [поиска](../../azure-monitor/app/diagnostic-search.md) и [обозревателя метрик](../../azure-monitor/essentials/metrics-charts.md).

Свойство "Версия приложения" можно настроить различными способами.

* Напрямую:

    `telemetryClient.Context.Component.Version = typeof(MyProject.MyClass).Assembly.GetName().Version;`
* Вставьте эту строку в [инициализатор телеметрии](../../azure-monitor/app/api-custom-events-metrics.md#defaults) , чтобы обеспечить согласованность всех экземпляров TelemetryClient.
* [ASP.NET] Задайте версию в `BuildInfo.config`. Веб-модуль берет номер версии из узла BuildLabel. Включите этот файл в проект и не забудьте установить свойство «Всегда копировать» в обозревателе решений.

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <DeploymentEvent xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/VisualStudio/DeploymentEvent/2013/06">
      <ProjectName>AppVersionExpt</ProjectName>
      <Build type="MSBuild">
        <MSBuild>
          <BuildLabel kind="label">1.0.0.2</BuildLabel>
        </MSBuild>
      </Build>
    </DeploymentEvent>

    ```
* [ASP.NET] Настройте автоматическое создание файла BuildInfo.config в MSBuild. Для этого добавьте в файл `.csproj` несколько строк:

    ```XML
    <PropertyGroup>
      <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>    <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
    </PropertyGroup>
    ```

    Вы получите файл *Имя_проекта*.BuildInfo.config. В процессе публикации он переименовывается в BuildInfo.config.

    При создании сборки с помощью Visual Studio в подпись включается заполнитель (AutoGen_...). Если используется MSBuild, в подписи указывается правильный номер версии.

    Чтобы разрешить MSBuild генерировать номера версий, задайте версию вида `1.0.*` в файле AssemblyReference.cs.

## <a name="version-and-release-tracking"></a>Отслеживание версии и выпуска
Для отслеживания версии приложения убедитесь, что во время выполнения процесса Microsoft Build Engine создается `buildinfo.config`. Добавьте в файл `.csproj` следующее:  

```XML
<PropertyGroup>
  <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>
  <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
</PropertyGroup>
```

При наличии данных сборки веб-модуль Application Insights автоматически добавляет **версию приложения** как свойство для каждого элемента телеметрии. Это позволяет применить фильтр по версии при [диагностическом поиске](../../azure-monitor/app/diagnostic-search.md) или [изучении метрик](../../azure-monitor/essentials/metrics-charts.md).

Обратите внимание, что номер версии сборки создается только в Microsoft Build Engine, но не в процессе сборки из Visual Studio.

### <a name="release-annotations"></a>Примечания к выпуску
Если используется Azure DevOps, можно настроить [добавление маркера заметки](../../azure-monitor/app/annotations.md) к диаграммам при выпуске новой версии. 

## <a name="next-steps"></a>Дальнейшие действия

* [Monitor multi-component applications with Application Insights (preview)](../../azure-monitor/app/app-map.md) (Мониторинг приложений с несколькими компонентами с помощью Application Insights (предварительная версия))
* [Добавление свойств: ITelemetryInitializer](../../azure-monitor/app/api-filtering-sampling.md#add-properties)
