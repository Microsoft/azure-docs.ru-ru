---
title: Потоковая передача журналов выполнения в функциях Azure
description: 115-145 символов, включая пробелы. Эта аннотация отображается в результатах поиска.
ms.date: 9/1/2020
ms.topic: how-to
ms.custom: contperf-fy21q2, devx-track-azurecli
ms.openlocfilehash: 085849386fce929ceaec4536844cf31b94fe7539
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97033057"
---
# <a name="enable-streaming-execution-logs-in-azure-functions"></a>Включение журналов выполнения потоковой передачи в функциях Azure

При разработке приложения часто требуется узнать, какие записи записываются в журналы практически в реальном времени при работе в Azure.

Существует два способа просмотра потока файлов журнала, создаваемых при выполнении функций.

* **Встроенная потоковая передача журналов**. Платформа Службы приложений позволяет просматривать поток файлов журналов приложений. Он эквивалентен выходным данным, которые отображаются при отладке функций во время [локальной разработки](functions-develop-local.md) и при использовании вкладки **Тест** на портале. При этом отображаются все данные в журнале. Дополнительные сведения см. в разделе [Потоковая передача журналов](../app-service/troubleshoot-diagnostic-logs.md#stream-logs). Этот потоковый метод поддерживает только один экземпляр и не может использоваться с приложением, которое выполняется в Linux в плане потребления.

* **Live Metrics Stream**. Если приложение-функция [подключено к Application Insights](configure-monitoring.md#enable-application-insights-integration), данные журнала и другие метрики можно просматривать почти в реальном времени на портале Azure с помощью [Live Metrics Stream](../azure-monitor/app/live-stream.md). Используйте этот метод при мониторинге функций, выполняемых в нескольких экземплярах или в Linux в плане потребления. Этот метод использует [данные выборки](configure-monitoring.md#configure-sampling).

Потоки журнала можно просматривать как на портале, так и в большинстве локальных сред разработки. 

## <a name="portal"></a>Портал

На портале можно просматривать оба типа потоков журналов.

### <a name="built-in-log-streaming"></a>Встроенная потоковая передача журналов

Чтобы просмотреть журналы потоковой передачи на портале, выберите в приложении-функции вкладку **Функции платформы**. Затем в разделе **Мониторинг** выберите **Потоковая передача журнала**.

![Включение потоковой передачи журналов на портале](./media/functions-monitoring/enable-streaming-logs-portal.png)

При этом приложение будет подключено к службе потоковой передачи журналов, а журналы приложений отобразятся в окне. Вы можете переключаться между режимом отображения журналов для **приложения** и **веб-сервера**.  

![Просмотр потоковой передачи журналов на портале](./media/functions-monitoring/streaming-logs-window.png)

### <a name="live-metrics-stream"></a>Динамический поток метрик

Чтобы просмотреть Live Metrics Stream для приложения, перейдите на вкладку **Обзор** приложения-функции. Если компонент Application Insights включен, в разделе **Настроенные функции** отображается ссылка **Application Insights**. По этой ссылке можно перейти на страницу Application Insights приложения.

В Application Insights выберите **Live Metrics Stream**. В разделе **Пример телеметрии** отображаются [записи выборки из журнала](configure-monitoring.md#configure-sampling).

![Просмотр Live Metrics Stream на портале](./media/functions-monitoring/live-metrics-stream.png) 

## <a name="visual-studio-code"></a>Visual Studio Code

[!INCLUDE [functions-enable-log-stream-vs-code](../../includes/functions-enable-log-stream-vs-code.md)]

## <a name="core-tools"></a>Core Tools

[!INCLUDE [functions-streaming-logs-core-tools](../../includes/functions-streaming-logs-core-tools.md)]

## <a name="azure-cli"></a>Azure CLI

Вы можете включить потоковую передачу журналов с помощью [Azure CLI](/cli/azure/install-azure-cli). Используйте следующие команды, чтобы войти, выбрать подписку и включить потоковую передачу файлов журнала.

```azurecli
az login
az account list
az account set --subscription <subscriptionNameOrId>
az webapp log tail --resource-group <RESOURCE_GROUP_NAME> --name <FUNCTION_APP_NAME>
```

## <a name="azure-powershell"></a>Azure PowerShell

Вы можете включить потоковую передачу журналов с помощью [Azure PowerShell](/powershell/azure/). Для PowerShell используйте команду [Set-азвебапп](/powershell/module/az.websites/set-azwebapp) , чтобы включить ведение журнала в приложении-функции, как показано в следующем фрагменте кода: 

:::code language="powershell" source="~/powershell_scripts/app-service/monitor-with-logs/monitor-with-logs.ps1" range="19-20":::

Дополнительные сведения см. в [полном примере кода](../app-service/scripts/powershell-monitor.md#sample-script). 

## <a name="next-steps"></a>Дальнейшие действия

+ [Мониторинг Функций Azure](functions-monitoring.md)
+ [Анализ телеметрии функций Azure в Application Insights](analyze-telemetry-data.md)
