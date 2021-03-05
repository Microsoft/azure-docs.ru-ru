---
title: Профилирование динамических приложений службы приложений Azure с помощью Application Insights | Документация Майкрософт
description: Профилирование динамических приложений в службе приложений Azure с помощью Application Insights Profiler.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 08/06/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 2a5ba6cb0778e0c013f150d787d694b23f0497bd
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102218015"
---
# <a name="profile-live-azure-app-service-apps-with-application-insights"></a>Профилирование динамических приложений службы приложений Azure с помощью Application Insights

Вы можете использовать Profiler на приложениях ASP.NET и ASP.NET Core, работающих в Службе приложений Azure, с использованием уровня служб "базовый" или выше. Сейчас включить средство Profiler на платформе Linux можно с помощью [этого метода](profiler-aspnetcore-linux.md).

## <a name="enable-profiler-for-your-app"></a><a id="installation"></a> Включение Profiler для приложений
Чтобы включить Profiler для приложения, сделайте следующее. Далее приведены сведения по включению Profiler на других поддерживаемых платформах при выполнении службы Azure другого типа.
* [Облачные службы](./profiler-cloudservice.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Приложения Service Fabric](./profiler-servicefabric.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
* [Виртуальные машины](./profiler-vm.md?toc=%2fazure%2fazure-monitor%2ftoc.json)

Application Insights Profiler устанавливается в составе среды выполнения служб приложений. Ниже показано, как включить его для своей Службы приложений. Выполните эти действия, даже если вы включили пакет SDK для App Insights в свое приложение во время сборки.

> [!NOTE]
> Установка Application Insights Profiler, не поддерживающая код, соответствует политике поддержки .NET Core.
> Дополнительные сведения о поддерживаемых средах выполнения см. в разделе [Политика поддержки .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

1. Перейдите на панель управления Azure для службы приложений.
1. Установите параметр "Всегда включена" для службы приложений. Этот параметр можно найти в разделе **Параметры**, страница **конфигурации** (см. снимок экрана на следующем шаге) и перейдите на вкладку **Общие параметры** .
1. Перейдите в раздел **параметры > Application Insights** странице.

   ![Включение Application Insights на портале служб приложений](./media/profiler/AppInsights-AppServices.png)

1. Следуйте инструкциям на панели, чтобы создать ресурс Application Insights или выбрать имеющийся ресурс для отслеживания приложения. Также убедитесь, что средство Profiler **включено**. Если ресурс Application Insights находится в другой подписке, чем Служба приложений, вы не сможете настроить Application Insights с помощью этой страницы. Это по-прежнему можно сделать вручную, создав необходимые параметры приложения вручную. [В следующем разделе приведены инструкции по включению Profiler вручную.](#enable-profiler-manually-or-with-azure-resource-manager) 

   ![Добавление расширения сайта Application Insights][Enablement UI]

1. Теперь средство Profiler включается с помощью параметра приложения Службы приложений.

    ![Параметр приложения для профилировщика][profiler-app-setting]

## <a name="enable-profiler-manually-or-with-azure-resource-manager"></a>Включение Profiler вручную или с помощью Azure Resource Manager
Application Insights Profiler можно включить, создав параметры приложения для Cлужбы приложений Azure. Страница с параметрами, приведенная выше, создает эти параметры приложения. Но вы можете автоматизировать создание этих параметров с помощью шаблона или других средств. Эти параметры также будут работать, если ресурс Application Insights находится не в той же подписке, что и Служба приложений Azure.
Ниже приведены параметры, необходимые для включения профилировщика.

|Параметр приложения    | Значение    |
|---------------|----------|
|APPINSIGHTS_INSTRUMENTATIONKEY         | iKey для ресурса Application Insights    |
|APPINSIGHTS_PROFILERFEATURE_VERSION | 1.0.0 |
|DiagnosticServices_EXTENSION_VERSION | ~3 |


Эти значения можно задать с помощью [шаблонов Azure Resource Manager](./azure-web-apps.md#app-service-application-settings-with-azure-resource-manager), [Azure PowerShell](/powershell/module/az.websites/set-azwebapp)  [Azure CLI](/cli/azure/webapp/config/appsettings).

## <a name="enable-profiler-for-other-clouds"></a>Включение профилировщика для других облаков

Сейчас только регионы, требующие внесения изменений в конечную точку, — это [Azure для государственных организаций](https://docs.microsoft.com/azure/azure-government/compare-azure-government-global-azure#application-insights) и [Azure для Китая](https://docs.microsoft.com/azure/china/resources-developer-guide).

|Параметр приложения    | Облако для государственных организаций США | Облако для Китая |   
|---------------|---------------------|-------------|
|ApplicationInsightsProfilerEndpoint         | `https://profiler.monitor.azure.us`    | `https://profiler.monitor.azure.cn` |
|ApplicationInsightsEndpoint | `https://dc.applicationinsights.us` | `https://dc.applicationinsights.azure.cn` |

## <a name="disable-profiler"></a>Отключение профилировщика

Чтобы прерывать или перезапускать профилировщик для экземпляра отдельного приложения, на левой боковой панели выберите веб- **задания** и прервите задание с именем `ApplicationInsightsProfiler3` .

  ![Отключение профилировщика для веб-задания][disable-profiler-webjob]

Мы рекомендуем включить Profiler для всех приложений, чтобы обеспечить максимально быстрое обнаружение проблем производительности.

Файлы Profiler можно удалить при использовании WebDeploy для развертывания изменений в веб-приложении. Можно предотвратить удаление, исключив папку App_Data из числа удаляемых во время развертывания. 


## <a name="next-steps"></a>Дальнейшие действия

* [Работа с Azure Application Insights в Visual Studio](./visual-studio.md)

[Enablement UI]: ./media/profiler/Enablement_UI.png
[profiler-app-setting]:./media/profiler/profiler-app-setting.png
[disable-profiler-webjob]: ./media/profiler/disable-profiler-webjob.png

