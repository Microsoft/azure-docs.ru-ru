---
title: Мониторинг производительности служб приложений Azure | Документация Майкрософт
description: Мониторинг производительности приложений для служб приложений Azure. Загрузка диаграммы и время отклика, сведения о зависимостях и Настройка оповещений о производительности.
ms.topic: conceptual
ms.date: 08/06/2020
ms.custom: devx-track-js, devx-track-dotnet
ms.openlocfilehash: 7661066bc2666070c8b3ed9263b1223c09d6c720
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101734729"
---
# <a name="monitor-azure-app-service-performance"></a>Мониторинг производительности Службы приложений Azure

Включение мониторинга для веб-приложений на основе ASP.NET, ASP.NET Core и Node.js, работающих в [службах приложений Azure](../../app-service/index.yml) , стало еще проще, чем когда бы то ни было. В то время как раньше требовалось вручную установить расширение сайта, в образ службы приложений теперь по умолчанию встроено Последнее расширение или агент. В этой статье описано, как включить мониторинг Application Insights, а также приводятся предварительные рекомендации по автоматизации процесса для крупномасштабных развертываний.

> [!NOTE]
> Добавление расширения Application Insights сайта вручную с помощью **расширений средств разработки**  >   является устаревшим. Этот метод установки расширения был зависеть от ручных обновлений для каждой новой версии. Последний стабильный выпуск расширения теперь  [предварительно установлен](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions) в составе образа службы приложений. Файлы находятся в `d:\Program Files (x86)\SiteExtensions\ApplicationInsightsAgent` и автоматически обновляются в каждом стабильном выпуске. Если вы выполните инструкции на основе агента, чтобы включить наблюдение ниже, оно автоматически удалит устаревшее расширение.

## <a name="enable-application-insights"></a>Включение Application Insights

Включить мониторинг приложений для приложений, размещенных в службах приложений Azure, можно двумя способами:

* **Мониторинг приложений на основе агента** (аппликатионинсигхтсажент).  
    * Этот метод является самым простым для включения и не требует дополнительной настройки. Часто это называется мониторингом среды выполнения. Для служб приложений Azure рекомендуется как минимум включить этот уровень мониторинга, а затем в зависимости от конкретного сценария можно оценить, требуется ли более сложный мониторинг с помощью инструментирования вручную.

* **Ручное инструментирование приложения с помощью кода** путем установки пакета SDK для Application Insights.

    * Этот подход гораздо более настраиваемый, но требует [добавления зависимости от Application Insights пакетов NUGET SDK](./asp-net.md). Этот метод также означает, что вам нужно самостоятельно управлять обновлениями для последних версий пакетов.

    * Если необходимо выполнить пользовательские вызовы API для отслеживания событий или зависимостей, не отслеживаемых по умолчанию с помощью мониторинга на основе агентов, необходимо использовать этот метод. Дополнительные сведения см. в [статье об API для получения пользовательских событий и метрик](./api-custom-events-metrics.md) . В настоящее время это единственный поддерживаемый вариант для рабочих нагрузок на основе Linux.

> [!NOTE]
> Если обнаруживается и отслеживание на основе агентов, и инструментирование вручную на основе пакета SDK, будут учитываться только параметры инструментирования вручную. Это позволяет предотвратить отправку повторяющихся данных. Чтобы получить дополнительные сведения об этом, ознакомьтесь с [разделом устранение неполадок](#troubleshooting) ниже.

## <a name="enable-agent-based-monitoring"></a>Включить мониторинг на основе агентов

# <a name="aspnet"></a>[ASP.NET](#tab/net)

> [!NOTE]
> Сочетание APPINSIGHTS_JAVASCRIPT_ENABLED и urlCompression не поддерживается. Дополнительные сведения см. в описании [раздела Устранение неполадок](#troubleshooting).


1. На панели управления Azure **выберите Application Insights** для своей службы приложений.

    ![В разделе "Параметры" выберите пункт Application Insights](./media/azure-web-apps/settings-app-insights-01.png)

   * Нажмите переключатель "Создать новый ресурс", если вы еще не настроили ресурс Application Insights для этого приложения. 

     > [!NOTE]
     > Нажав кнопку **OK** для создания нового ресурса, вам будет предложено выбрать параметр **Применение параметров мониторинга**. Нажав кнопку **Продолжить**, вы свяжете новый ресурс Application Insights со службой приложений, а также **активируете перезапуск самой службы приложений**. 

     ![Инструментирование веб-приложения](./media/azure-web-apps/create-resource-01.png)

2. Указав ресурс для использования, вы можете выбрать способ сбора данных в Application Insights для каждой платформы для приложения. Мониторинг приложений ASP.NET по умолчанию имеет два разных уровня коллекции.

    ![На снимке экрана показана страница расширения сайта Application Insights с выбранным параметром создать новый ресурс.](./media/azure-web-apps/choose-options-new.png)
 
 Ниже приведена сводка данных, собираемых для каждого маршрута.
        
| Данные | Базовая коллекция ASP.NET | Рекомендуемая коллекция ASP.NET |
| --- | --- | --- |
| Добавляет тенденции использования ЦП, памяти и операций ввода-вывода. |Да |Да |
| Собирает тенденции использования и обеспечивает сопоставление от результатов доступности до транзакций. | Да |Да |
| Собирает исключения, необработанные хост-процессом. | Да |Да |
| Улучшает точность метрик APM под нагрузкой при использовании выборки. | Да |Да |
| Сопоставляет микрослужбы в пределах границ запросов или зависимостей. | Нет (только возможности APM с одним экземпляром) |Да |

3. Чтобы настроить такие параметры, как выборка, которые можно было бы контролировать с помощью файла applicationinsights.config теперь можно взаимодействовать с теми же параметрами через параметры приложения с соответствующим префиксом. 

    * Например, чтобы изменить начальную долю выборки, можно создать параметр приложения: `MicrosoftAppInsights_AdaptiveSamplingTelemetryProcessor_InitialSamplingPercentage` и значение `100` .

    * Список поддерживаемых параметров обработчика данных телеметрии адаптивной выборки можно найти в [коде](https://github.com/microsoft/ApplicationInsights-dotnet/blob/master/BASE/Test/ServerTelemetryChannel.Test/TelemetryChannel.Tests/AdaptiveSamplingTelemetryProcessorTest.cs) и [связанной документации](./sampling.md).

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/netcore)

> [!IMPORTANT]
> Поддерживаются следующие версии ASP.NET Core: ASP.NET Core 2,1 и 3,1. Версии 2,0, 2,2 и 3,0 были прекращены и больше не поддерживаются. Выполните обновление до [поддерживаемой версии](https://dotnet.microsoft.com/platform/support/policy/dotnet-core) .NET Core, чтобы автоматическое инструментирование работало.

Для полной платформы, предназначенной для ASP.NET Core, автономного развертывания и приложений на основе Linux в настоящее время **не поддерживается** мониторинг на основе агента или расширения. ([Инструментирование вручную](./asp-net-core.md) с помощью кода будет работать во всех предыдущих сценариях.)

1. На панели управления Azure **выберите Application Insights** для своей службы приложений.

    ![В разделе "Параметры" выберите пункт Application Insights](./media/azure-web-apps/settings-app-insights-01.png)

   * Нажмите переключатель "Создать новый ресурс", если вы еще не настроили ресурс Application Insights для этого приложения. 

     > [!NOTE]
     > Нажав кнопку **OK** для создания нового ресурса, вам будет предложено выбрать параметр **Применение параметров мониторинга**. Нажав кнопку **Продолжить**, вы свяжете новый ресурс Application Insights со службой приложений, а также **активируете перезапуск самой службы приложений**. 

     ![Инструментирование веб-приложения](./media/azure-web-apps/create-resource-01.png)

2. Указав, какой ресурс следует использовать, можно выбрать способ, с помощью которого Application Insights собирает данные для каждой платформы приложения. ASP.NET Core предлагает **рекомендуемый сбор** или **отключение** для ASP.NET Core 2,1 и 3,1.

    ![Выбор параметров для каждой платформы](./media/azure-web-apps/choose-options-new-net-core.png)

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Мониторинг на основе агента Windows не поддерживается. чтобы включить его в Linux, обратитесь к [ документации по службе приложенийNode.js](../../app-service/configure-language-nodejs.md?pivots=platform-linux#monitor-with-application-insights).

# <a name="java"></a>[Java](#tab/java)

Следуйте рекомендациям для [Application Insights агента java 3,0](./java-in-process-agent.md) , чтобы включить автоматическое инструментирование для приложений Java без изменения кода.
Автоматическая интеграция еще не доступна для службы приложений.

# <a name="python"></a>[Python](#tab/python)

Веб-приложения на основе службы приложений Python в настоящее время не поддерживают автоматическое отслеживание на основе агентов и расширений. Чтобы включить мониторинг для приложения Python, необходимо [вручную выполнить инструментирование приложения](./opencensus-python.md).

---

## <a name="enable-client-side-monitoring"></a>Включение наблюдения на стороне клиента

# <a name="aspnet"></a>[ASP.NET](#tab/net)

Наблюдение на стороне клиента — это согласие на ASP.NET. Чтобы включить наблюдение на стороне клиента:

* **Параметры** **>** **Конфигурация**
   * В разделе Параметры приложения создайте **новый параметр приложения**.

     Имя: `APPINSIGHTS_JAVASCRIPT_ENABLED`

     Значение: `true`

   * **Сохраните** параметры и **перезапустите** приложение.

Чтобы отключить наблюдение на стороне клиента, удалите связанную пару "ключ-значение" из параметров приложения или установите значение false.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/netcore)

Наблюдение на стороне клиента **включено по умолчанию** для ASP.NET Core приложений с **рекомендуемой коллекцией**, независимо от наличия параметра приложения "APPINSIGHTS_JAVASCRIPT_ENABLED".

Если по какой-либо причине вы хотели бы отключить наблюдение на стороне клиента:

* **Параметры** **>** **Конфигурация**
   * В разделе Параметры приложения создайте **новый параметр приложения**.

     безымян `APPINSIGHTS_JAVASCRIPT_ENABLED`

     Значение: `false`

   * **Сохраните** параметры и **перезапустите** приложение.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Чтобы включить мониторинг на стороне клиента для Node.js приложения, необходимо [вручную добавить клиентский пакет SDK JavaScript в приложение](./javascript.md).

# <a name="java"></a>[Java](#tab/java)

Чтобы включить мониторинг на стороне клиента для приложения Java, необходимо [вручную добавить пакет SDK JavaScript на стороне клиента в приложение](./javascript.md).

# <a name="python"></a>[Python](#tab/python)

Чтобы включить мониторинг на стороне клиента для приложения Python, необходимо [вручную добавить пакет SDK JavaScript на стороне клиента в приложение](./javascript.md).

---

## <a name="automate-monitoring"></a>Автоматизация мониторинга

Чтобы включить сбор данных телеметрии с Application Insights, необходимо задать только параметры приложения:

   ![Параметры приложения службы приложений с доступными параметрами Application Insights](./media/azure-web-apps/application-settings.png)

### <a name="application-settings-definitions"></a>Определения параметров приложения

|Имя параметра приложения |  Определение | Значение |
|-----------------|:------------|-------------:|
|ApplicationInsightsAgent_EXTENSION_VERSION | Главное расширение, которое управляет мониторингом среды выполнения. | `~2` |
|XDT_MicrosoftApplicationInsights_Mode |  В режиме по умолчанию для обеспечения оптимальной производительности включены только функции, обеспечивающие их работу. | `default` или `recommended`. |
|InstrumentationEngine_EXTENSION_VERSION | Определяет, будет ли включен модуль двоичной перезаписи `InstrumentationEngine` . Этот параметр оказывает влияние на производительность и влияет на время холодного запуска и запуска. | `~1` |
|XDT_MicrosoftApplicationInsights_BaseExtensions | Определяет, будет ли записываться текст таблицы SQL & Azure вместе с вызовами зависимостей. Предупреждение о производительности: время холодного запуска приложения будет затронуто. Для этого параметра требуется `InstrumentationEngine` . | `~1` |
|XDT_MicrosoftApplicationInsights_PreemptSdk | Только для ASP.NET Core приложений. Включает взаимодействие (взаимодействий) с пакетом SDK для Application Insights. Загружает расширение параллельно с пакетом SDK и использует его для отправки данных телеметрии (отключает пакет SDK для Application Insights). |`1`|

### <a name="app-service-application-settings-with-azure-resource-manager"></a>Параметры приложения службы приложений с Azure Resource Manager

Параметры приложения для служб приложений можно управлять и настраивать с помощью [шаблонов Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md). Этот метод можно использовать при развертывании новых ресурсов службы приложений с помощью Azure Resource Manager Automation или изменении параметров существующих ресурсов.

Ниже показана базовая структура JSON параметров приложения для службы приложений.

```JSON
      "resources": [
        {
          "name": "appsettings",
          "type": "config",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites', variables('webSiteName'))]"
          ],
          "tags": {
            "displayName": "Application Insights Settings"
          },
          "properties": {
            "key1": "value1",
            "key2": "value2"
          }
        }
      ]
```

Чтобы получить пример шаблона Azure Resource Manager с параметрами приложения, настроенными для Application Insights, этот [шаблон](https://github.com/Andrew-MSFT/BasicImageGallery) может быть полезен, в частности раздел, начинающийся в [строке 238](https://github.com/Andrew-MSFT/BasicImageGallery/blob/c55ada54519e13ce2559823c16ca4f97ddc5c7a4/CoreImageGallery/Deploy/CoreImageGalleryARM/azuredeploy.json#L238).

### <a name="automate-the-creation-of-an-application-insights-resource-and-link-to-your-newly-created-app-service"></a>Автоматизируйте создание Application Insights ресурса и ссылку на созданную службу приложений.

Чтобы создать шаблон Azure Resource Manager со всеми настроенными параметрами Application Insights по умолчанию, начните процесс, как если бы вы создавали новое веб-приложение с включенным Application Insights.

Выбор **параметров автоматизации**

   ![Меню создания веб-приложения службы приложений](./media/azure-web-apps/create-web-app.png)

Этот параметр создает последний шаблон Azure Resource Manager со всеми настройками обязательных параметров.

  ![Шаблон веб-приложения службы приложений](./media/azure-web-apps/arm-template.png)

Ниже приведен пример, замените все экземпляры  `AppMonitoredSite` именем своего сайта:

```json
{
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "Microsoft.Web/sites",
            "properties": {
                "siteConfig": {
                    "appSettings": [
                        {
                            "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
                            "value": "[reference('microsoft.insights/components/AppMonitoredSite', '2015-05-01').InstrumentationKey]"
                        },
                        {
                            "name": "APPLICATIONINSIGHTS_CONNECTION_STRING",
                            "value": "[reference('microsoft.insights/components/AppMonitoredSite', '2015-05-01').ConnectionString]"
                        },
                        {
                            "name": "ApplicationInsightsAgent_EXTENSION_VERSION",
                            "value": "~2"
                        }
                    ]
                },
                "name": "[parameters('name')]",
                "serverFarmId": "[concat('/subscriptions/', parameters('subscriptionId'),'/resourcegroups/', parameters('serverFarmResourceGroup'), '/providers/Microsoft.Web/serverfarms/', parameters('hostingPlanName'))]",
                "hostingEnvironment": "[parameters('hostingEnvironment')]"
            },
            "dependsOn": [
                "[concat('Microsoft.Web/serverfarms/', parameters('hostingPlanName'))]",
                "microsoft.insights/components/AppMonitoredSite"
            ],
            "apiVersion": "2016-03-01",
            "location": "[parameters('location')]"
        },
        {
            "apiVersion": "2016-09-01",
            "name": "[parameters('hostingPlanName')]",
            "type": "Microsoft.Web/serverfarms",
            "location": "[parameters('location')]",
            "properties": {
                "name": "[parameters('hostingPlanName')]",
                "workerSizeId": "[parameters('workerSize')]",
                "numberOfWorkers": "1",
                "hostingEnvironment": "[parameters('hostingEnvironment')]"
            },
            "sku": {
                "Tier": "[parameters('sku')]",
                "Name": "[parameters('skuCode')]"
            }
        },
        {
            "apiVersion": "2015-05-01",
            "name": "AppMonitoredSite",
            "type": "microsoft.insights/components",
            "location": "West US 2",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Request_Source": "IbizaWebAppExtensionCreate"
            }
        }
    ],
    "parameters": {
        "name": {
            "type": "string"
        },
        "hostingPlanName": {
            "type": "string"
        },
        "hostingEnvironment": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "sku": {
            "type": "string"
        },
        "skuCode": {
            "type": "string"
        },
        "workerSize": {
            "type": "string"
        },
        "serverFarmResourceGroup": {
            "type": "string"
        },
        "subscriptionId": {
            "type": "string"
        }
    },
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0"
}
```

### <a name="enabling-through-powershell"></a>Включение с помощью PowerShell

Чтобы включить мониторинг приложений с помощью PowerShell, необходимо изменить только базовые параметры приложения. Ниже приведен пример, который позволяет отслеживать приложения для веб-сайта с именем "Аппмониторедсите" в группе ресурсов "Аппмониторедрг" и настраивает данные для отправки в ключ инструментирования "012345678-ABCD-ef01-2345-6789abcd".

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```powershell
$app = Get-AzWebApp -ResourceGroupName "AppMonitoredRG" -Name "AppMonitoredSite" -ErrorAction Stop
$newAppSettings = @{} # case-insensitive hash map
$app.SiteConfig.AppSettings | %{$newAppSettings[$_.Name] = $_.Value} # preserve non Application Insights application settings.
$newAppSettings["APPINSIGHTS_INSTRUMENTATIONKEY"] = "012345678-abcd-ef01-2345-6789abcd"; # set the Application Insights instrumentation key
$newAppSettings["APPLICATIONINSIGHTS_CONNECTION_STRING"] = "InstrumentationKey=012345678-abcd-ef01-2345-6789abcd"; # set the Application Insights connection string
$newAppSettings["ApplicationInsightsAgent_EXTENSION_VERSION"] = "~2"; # enable the ApplicationInsightsAgent
$app = Set-AzWebApp -AppSettings $newAppSettings -ResourceGroupName $app.ResourceGroup -Name $app.Name -ErrorAction Stop
```

## <a name="upgrade-monitoring-extensionagent"></a>Обновить расширение или агент мониторинга

### <a name="upgrading-from-versions-289-and-up"></a>Обновление с версий 2.8.9 и выше

Обновление с версии 2.8.9 происходит автоматически без каких бы то ни было дополнительных действий. Новые биты мониторинга доставляются в фоновом режиме в целевую службу приложений, и при перезапуске приложения они будут отобраны.

Чтобы проверить, какая версия расширения используется, посетите `http://yoursitename.scm.azurewebsites.net/ApplicationInsights`

![Снимок экрана: URL-путь http://yoursitename.scm.azurewebsites.net/ApplicationInsights](./media/azure-web-apps/extension-version.png)

### <a name="upgrade-from-versions-100---265"></a>Обновление с версий 1.0.0 – 2.6.5

Начиная с версии 2.8.9 используется предварительно установленное расширение сайта. Если вы используете более раннюю версию, можно выполнить обновление одним из двух способов:

* [Обновление путем включения через портал](#enable-application-insights). (Даже если у вас установлен модуль Application Insights для службы приложений Azure, в пользовательском интерфейсе отображается только кнопка **включить** . В фоновом режиме старое расширение частного сайта будет удалено.)

* [Обновление с помощью PowerShell](#enabling-through-powershell):

    1. Задайте параметры приложения, чтобы включить предварительно установленное расширение сайта Аппликатионинсигхтсажент. См. раздел [Включение с помощью PowerShell](#enabling-through-powershell).
    2. Вручную удалите расширение частного сайта с именем Application Insights расширение для службы приложений Azure.

Если обновление выполняется из версии до 2.5.1, убедитесь, что библиотеки DLL Аппликатионинсигсс удалены из папки Bin приложения см. в [разделе действия по устранению неполадок](#troubleshooting).

## <a name="troubleshooting"></a>Устранение неполадок

Ниже приведено пошаговое руководство по устранению неполадок для мониторинга на основе расширений и агентов для ASP.NET и ASP.NET Core приложений, работающих в службах приложений Azure.

> [!NOTE]
> Для мониторинга приложений Java рекомендуется использовать автоматическое инструментирование без изменения кода. Следуйте указаниям для [Application Insights агента Java 3,0](./java-in-process-agent.md).


1. Убедитесь, что приложение отслеживается через `ApplicationInsightsAgent` .
    * Убедитесь, что `ApplicationInsightsAgent_EXTENSION_VERSION` для параметра приложения задано значение "~ 2".
2. Убедитесь, что приложение соответствует требованиям, которые необходимо отслеживать.
    * Перейдите по адресу `https://yoursitename.scm.azurewebsites.net/ApplicationInsights`.

    ![Снимок экрана со https://yoursitename.scm.azurewebsites/applicationinsights страницей результатов](./media/azure-web-apps/app-insights-sdk-status.png)

    * Убедитесь `Application Insights Extension Status` , что `Pre-Installed Site Extension, version 2.8.12.1527, is running.` 
    * Если она не запущена, следуйте инструкциям по [включению мониторинга Application Insights](#enable-application-insights)

    * Убедитесь, что источник состояния существует и выглядит следующим образом: `Status source D:\home\LogFiles\ApplicationInsights\status\status_RD0003FF0317B6_4248_1.json`
        * Если такое же значение отсутствует, это означает, что приложение в данный момент не выполняется или не поддерживается. Чтобы убедиться, что приложение выполняется, попробуйте вручную посетить конечные точки URL-адреса приложения или приложения, что позволит получить доступ к сведениям о среде выполнения.

    * Убедитесь, `IKeyExists` что `true`
        * Если это так `false` , добавьте `APPINSIGHTS_INSTRUMENTATIONKEY` в `APPLICATIONINSIGHTS_CONNECTION_STRING` Параметры приложения и с идентификатором GUID iKey.

    * Убедитесь в отсутствии записей для `AppAlreadyInstrumented` , `AppContainsDiagnosticSourceAssembly` и `AppContainsAspNetTelemetryCorrelationAssembly` .
        * Если любая из этих записей существует, удалите из приложения следующие пакеты: `Microsoft.ApplicationInsights` , `System.Diagnostics.DiagnosticSource` и `Microsoft.AspNet.TelemetryCorrelation` .
        * Только для ASP.NET Core приложений. Если приложение ссылается на какие-либо пакеты Application Insights, например если вы ранее выполнили инструментирование (или предпринимали попытку инструментирования) приложения с помощью [пакета SDK для ASP.NET Core](./asp-net-core.md), включение интеграции службы приложений может не действовать и данные могут не отображаться в Application Insights. Чтобы устранить эту проблему, на портале включите взаимодействие с пакетом SDK для Application Insights, и вы начнете Просмотр данных в Application Insights 
        > [!IMPORTANT]
        > Эта функция доступна в предварительной версии 

        ![Включение параметра для существующего приложения](./media/azure-web-apps/netcore-sdk-interop.png)

        Теперь данные будут отправляться с использованием бескод подхода, даже если изначально использовался Application Insights SDK или была предпринята попытка его использования.

        > [!IMPORTANT]
        > Если приложение использовало Application Insights SDK для отправки каких-либо данных телеметрии, такие данные телеметрии будут отключены. Иными словами, пользовательская телеметрия (если таковая имеется), например, любые методы отслеживания * () и пользовательские параметры, такие как выборка, будут отключены. 


### <a name="php-and-wordpress-are-not-supported"></a>PHP и WordPress не поддерживаются

Сайты PHP и WordPress не поддерживаются. В настоящее время официально поддерживаемый пакет SDK или агент не поддерживается для наблюдения за этими рабочими нагрузками на стороне сервера. Однако ручное инструментирование транзакций на стороне клиента на сайте PHP или WordPress путем добавления клиентского сценария JavaScript к веб-страницам можно выполнить с помощью [пакета SDK для JavaScript](./javascript.md).

В таблице ниже приведено более подробное описание этих значений, их основных причин и Рекомендуемые исправления.

|Значение проблемы|Пояснение|Fix
|---- |----|---|
| `AppAlreadyInstrumented:true` | Это значение указывает, что расширение обнаружило, что некоторые аспекты пакета SDK уже имеются в приложении и будут отключаться. Это может быть вызвано ссылкой на `System.Diagnostics.DiagnosticSource` ,  `Microsoft.AspNet.TelemetryCorrelation` или `Microsoft.ApplicationInsights`  | Удалите ссылки. Некоторые из этих ссылок добавляются по умолчанию из определенных шаблонов Visual Studio, а в более ранних версиях Visual Studio могут быть добавлены ссылки на `Microsoft.ApplicationInsights` .
|`AppAlreadyInstrumented:true` | Если приложение предназначено для ASP.NET Core 2,1 или 2,2, это значение указывает, что расширение обнаружило, что некоторые аспекты пакета SDK уже имеются в приложении и будут отключаться. | Клиентам в .NET Core 2.1, 2.2 [рекомендуется](https://github.com/aspnet/Announcements/issues/287) использовать вместо него мета-пакет Microsoft.AspNetCore.app. Кроме того, включите "взаимодействие с Application Insightsным пакетом SDK" на портале (см. инструкции выше).|
|`AppAlreadyInstrumented:true` | Это значение также может быть вызвано наличием указанных выше библиотек DLL в папке приложения из предыдущего развертывания. | Очистите папку приложения, чтобы убедиться, что эти библиотеки DLL удалены. Проверьте каталог Bin локального приложения и каталог wwwroot в службе приложений. (Чтобы проверить каталог wwwroot веб-приложения службы приложений: Advanced Tools (KUDU) > консоль отладки > CMD > хоме\сите\ввврут).
|`AppContainsAspNetTelemetryCorrelationAssembly: true` | Это значение указывает, что расширение обнаружило ссылки на `Microsoft.AspNet.TelemetryCorrelation` приложение и будет отключаться. | Удаление ссылки.
|`AppContainsDiagnosticSourceAssembly**:true`|Это значение указывает, что расширение обнаружило ссылки на `System.Diagnostics.DiagnosticSource` приложение и будет отключаться.| Для ASP.NET удалите ссылку. 
|`IKeyExists:false`|Это значение указывает, что ключ инструментирования отсутствует в AppSetting, `APPINSIGHTS_INSTRUMENTATIONKEY` . Возможные причины. возможно, значения были случайно удалены, забыли установить значения в скрипте автоматизации и т. д. | Убедитесь, что параметр есть в параметрах приложения службы приложений.

### <a name="appinsights_javascript_enabled-and-urlcompression-is-not-supported"></a>APPINSIGHTS_JAVASCRIPT_ENABLED и urlCompression не поддерживаются

При использовании APPINSIGHTS_JAVASCRIPT_ENABLED = true в случаях, когда содержимое кодируется, могут возникать такие ошибки: 

- 500 ошибка переопределения URL-адреса
- Ошибка модуля переопределения URL-адресов 500,53. правила перезаписи исходящих сообщений не могут быть применены, если содержимое HTTP-ответа закодировано (gzip). 

Это происходит из-за того, что для параметра приложения APPINSIGHTS_JAVASCRIPT_ENABLED задано значение true и кодировка содержимого представлена одновременно. Этот сценарий пока не поддерживается. Обходной путь состоит в том, чтобы удалить APPINSIGHTS_JAVASCRIPT_ENABLED из параметров приложения. К сожалению, это означает, что если на стороне клиента и браузера по-прежнему требуется инструментирование JavaScript, для веб-страниц требуются ручные ссылки на пакеты SDK. Следуйте [инструкциям](https://github.com/Microsoft/ApplicationInsights-JS#snippet-setup-ignore-if-using-npm-setup) по ручному инструментированию с помощью пакета SDK для JavaScript.

Последние сведения об агенте или расширении Application Insights см. в [заметках о выпуске](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/app-insights-web-app-extensions-releasenotes.md).

### <a name="default-website-deployed-with-web-apps-does-not-support-automatic-client-side-monitoring"></a>Веб-сайт по умолчанию, развернутый с помощью веб-приложений, не поддерживает автоматический мониторинг на стороне клиента

При создании веб-приложения со `ASP.NET` `ASP.NET Core` средами выполнения или в службах приложений Azure она развертывает одну статическую HTML-страницу в качестве начального веб-сайта. Статическая веб-страница также загружает управляемую ASP.NET Web Part в IIS. Это позволяет тестировать мониторинг на стороне сервера без кода, но не поддерживает автоматическое наблюдение на стороне клиента.

Если вы хотите протестировать серверный и клиентский мониторинг для ASP.NET или ASP.NET Core в веб-приложении служб приложений Azure, мы рекомендуем использовать официальные руководства по [созданию веб](../../app-service/quickstart-dotnetcore.md) -приложения ASP.NET Core и [созданию веб-приложения ASP.NET Framework](../../app-service/quickstart-dotnet-framework.md) , а затем выполнить инструкции в текущей статье, чтобы включить мониторинг.

### <a name="connection-string-and-instrumentation-key"></a>Строка подключения и ключ инструментирования

При использовании мониторинга без кода требуется только строка подключения. Однако мы по-прежнему рекомендуем задать ключ инструментирования, чтобы сохранить обратную совместимость с более старыми версиями пакета SDK при выполнении инструментирования вручную.

### <a name="difference-between-standard-metrics-from-application-insights-vs-azure-app-service-metrics"></a>Разница между стандартными метриками из Application Insights и метрик службы приложений Azure?

Application Insights собирает данные телеметрии для запросов, которые были сделаны в приложении. Если в веб-приложениях или IIS произошла ошибка, и запрос не получил доступ к пользовательскому приложению, Application Insights не будет получать данные телеметрии о нем.

Продолжительность `serverresponsetime` вычисления Application Insights не обязательно соответствует времени отклика сервера, наблюдаемого в веб-приложениях. Это происходит потому, что Application Insights подсчитывает только время, когда фактический запрос достигает пользовательского приложения. Если запрос зависает или помещается в очередь в службах IIS, это время ожидания будет включено в метрики веб-приложения, но не в метриках Application Insights.

## <a name="release-notes"></a>Заметки о выпуске

Последние обновления и исправления ошибок см. [в заметках о выпуске](./web-app-extension-release-notes.md).

## <a name="next-steps"></a>Дальнейшие действия
* [Запуск профилировщика в живом приложении](./profiler.md).
* [Функции Azure.](https://github.com/christopheranderson/azure-functions-app-insights-sample) Отслеживайте функции Azure с помощью Application Insights.
* [Включите отправку данных диагностики Azure](../agents/diagnostics-extension-to-application-insights.md) в Application Insights.
* [Отслеживайте метрики состояния службы](../data-platform.md), чтобы убедиться, что служба доступна и отвечает на запросы.
* [Получайте уведомления](../alerts/alerts-overview.md) при возникновении операционных событий или превышении пороговых значений метрик.
* Используйте [расширение Application Insights для приложений JavaScript и веб-страниц](javascript.md), чтобы получать данные телеметрии клиентов из браузеров, которые используются для открытия веб-страницы.
* [Настройте веб-тесты доступности](monitor-web-app-availability.md), чтобы получать уведомления о сбоях в работе сайта.
