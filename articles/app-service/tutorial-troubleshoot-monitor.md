---
title: Руководство по Устранение неполадок с помощью Azure Monitor
description: Сведения о том, как применять Azure Monitor и Log Analytics для мониторинга веб-приложения Службы приложений. Azure Monitor обеспечивает максимальную доступность, предоставляя комплексное решение для мониторинга сред.
author: msangapu-msft
ms.author: msangapu
ms.topic: tutorial
ms.date: 06/20/2020
ms.openlocfilehash: d45a8b8f426df32b9f5ac6f64237107083e0f9ab
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100586286"
---
# <a name="tutorial-troubleshoot-an-app-service-app-with-azure-monitor"></a>Руководство по Устранение неполадок с приложением Службы приложений с помощью Azure Monitor

> [!NOTE]
> Интеграция Azure Monitor со Службой приложений доступна в [предварительной версии](https://aka.ms/appsvcblog-azmon).
>

В этом учебнике описано, как устранять неполадки в приложении [Службы приложений](overview.md) с помощью [Azure Monitor](../azure-monitor/overview.md). Этот пример приложения содержит код, который занимает всю доступную память и приводит к ошибкам HTTP 500. Эту ошибку вы сможете диагностировать и устранить с помощью Azure Monitor. По завершении вы получите тестовое приложение, работающее в Службе приложений в Linux, интегрированной с [Azure Monitor](../azure-monitor/overview.md).

Служба [Azure Monitor](../azure-monitor/overview.md) обеспечивает максимальную доступность и производительность приложений и служб, предоставляя полноценное решение для сбора, анализа и обработки данных телеметрии из облачных и локальных сред.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка веб-приложения для работы с Azure Monitor;
> * отправка журналов консоли в Log Analytics;
> * использование запросов по журналам для обнаружения и устранения ошибок в веб-приложениях.

Шаги, описанные в этом руководстве, можно выполнить для macOS, Linux и Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам понадобится:

- [Подписка Azure.](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
- [Azure CLI](/cli/azure/install-azure-cli)
- [Git](https://git-scm.com/);

## <a name="create-azure-resources"></a>Создание ресурсов Azure

Для начала вы выполните несколько команд локально, чтобы настроить пример приложения для использования с этим руководством. Эти команды клонируют пример приложения, создают ресурсы Azure и пользователя развертывания, а затем развертывают приложение в Azure. Вам будет предложено ввести пароль, предоставленный в процессе создания пользователя развертывания. 

```bash
git clone https://github.com/Azure-Samples/App-Service-Troubleshoot-Azure-Monitor
az group create --name myResourceGroup --location "South Central US"
az webapp deployment user set --user-name <username> --password <password>
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku B1 --is-linux
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.3" --deployment-local-git
git remote add azure <url_from_previous_step>
git push azure main
```

## <a name="configure-azure-monitor-preview"></a>Настройка Azure Monitor (предварительная версия)

### <a name="create-a-log-analytics-workspace"></a>Создание рабочей области Log Analytics.

Теперь, завершив развертывание примера приложения в Службе приложений Azure, вы настроите функцию мониторинга для устранения возникающих в приложении неполадок. Azure Monitor хранит данные журналов в рабочей области Log Analytics. Рабочей областью называется контейнер, который содержит данные и сведения о конфигурации.

На этом шаге вы создадите рабочую область Log Analytics, чтобы настроить применение Azure Monitor для приложения.

```bash
az monitor log-analytics workspace create --resource-group myResourceGroup --workspace-name myMonitorWorkspace
```

> [!NOTE]
> [В Azure Monitor Log Analytics вы платите за прием и хранение данных.](https://azure.microsoft.com/pricing/details/monitor/)
>

### <a name="create-a-diagnostic-setting"></a>Создание параметра диагностики

Параметры диагностики позволяют настроить сбор метрик из определенных служб Azure в журналы Azure Monitor, чтобы анализировать их совместно с другими данными мониторинга с помощью запросов по журналам. Для работы с этим руководством вы включите ведение журналов веб-сервера, а также журналов ошибок и стандартного потока вывода. Полный список типов журналов с описанием см. в статье [Поддерживаемые типы журналов](./troubleshoot-diagnostic-logs.md#supported-log-types).

Чтобы создать параметры диагностики для AppServiceConsoleLogs (стандартный поток вывода и ошибки) и AppServiceHTTPLogs (журналы веб-сервера), нужно выполнить следующие команды. Замените _\<app-name>_ и _\<workspace-name>_ собственными значениями. 

> [!NOTE]
> Первые две команды, `resourceID` и `workspaceID`, создают переменные для использования в команде `az monitor diagnostic-settings create`. Дополнительные сведения об этой команде см. в разделе [Создание параметров диагностики с помощью Azure CLI](../azure-monitor/essentials/diagnostic-settings.md#create-using-azure-cli).
>

```bash
resourceID=$(az webapp show -g myResourceGroup -n <app-name> --query id --output tsv)

workspaceID=$(az monitor log-analytics workspace show -g myResourceGroup --workspace-name <workspace-name> --query id --output tsv)

az monitor diagnostic-settings create --resource $resourceID \
 --workspace $workspaceID \
 -n myMonitorLogs \
 --logs '[{"category": "AppServiceConsoleLogs", "enabled": true},
  {"category": "AppServiceHTTPLogs", "enabled": true}]'
```

## <a name="troubleshoot-the-app"></a>Устранение неполадок в приложении

Перейдите по адресу `http://<app-name>.azurewebsites.net`.

Этот пример приложения ImageConverter преобразует предоставленные изображения из `JPG` в `PNG`. В рамках этого руководства в код намеренно добавлена ошибка. Если вы выбрали несколько изображений, приложение выдает ошибку HTTP 500 при их преобразовании. Предположим, что такой сценарий был упущен на этапе разработки. Для устранения этой ошибки вы примените Azure Monitor.

### <a name="verify-the-app-is-works"></a>Проверка работоспособности приложения

Чтобы преобразовать изображения, щелкните `Tools` (Средства) и выберите `Convert to PNG` (Преобразовать в PNG).

![Выбор элементов Tools (Средства) и Convert to PNG (Преобразовать в PNG)](./media/tutorial-azure-monitor/sample-monitor-app-tools-menu.png)

Выберите первые два изображения и щелкните `convert` (Преобразовать). Эти файлы будут успешно преобразованы.

![Выбор двух изображений](./media/tutorial-azure-monitor/sample-monitor-app-convert-two-images.png)

### <a name="break-the-app"></a>Аварийное завершение приложения

Мы убедились, что приложение успешно преобразовывает два изображения, а теперь попробуем преобразовать первые пять изображений.

![Преобразование первых пяти изображений](./media/tutorial-azure-monitor/sample-monitor-app-convert-five-images.png)

Это действие завершается ошибкой с сообщением `HTTP 500`, которая не была проверена во время разработки.

![Преобразование приводит к ошибке HTTP 500](./media/tutorial-azure-monitor/sample-monitor-app-http-500.png)

## <a name="use-log-query-to-view-azure-monitor-logs"></a>Использование запроса по журналам для просмотра журналов Azure Monitor

Давайте посмотрим, какие журналы доступны в рабочей области Log Analytics. 

Щелкните [эту ссылку на рабочую область Log Analytics](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.OperationalInsights%2Fworkspaces), чтобы открыть ее на портале Azure.

На портале Azure выберите рабочую область Log Analytics.

### <a name="log-queries"></a>Запросы журнала

Запросы по журналам позволяют с пользой применить все данные, собранные в журналах Azure Monitor. Вы можете применять запросы по журналам, собранным в AppServiceHTTPLogs и AppServiceConsoleLogs. Дополнительные сведения о запросах по журналам см. в [этой обзорной статье](../azure-monitor/logs/log-query-overview.md).

### <a name="view-appservicehttplogs-with-log-query"></a>Просмотр AppServiceHTTPLogs с помощью запросов по журналам

Итак, мы получили доступ к приложению, а теперь изучим данные в `AppServiceHTTPLogs`, которые связаны с выполненными HTTP-запросами.

1. Щелкните `Logs` в области навигации слева.

![Журналы рабочей области Log Anlytics](./media/tutorial-azure-monitor/log-analytics-workspace-logs.png)

2. Выполните поиск по строке `appservice` и дважды щелкните элемент `AppServiceHTTPLogs`.

![Таблицы рабочей области Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-app-service-tables.png)

3. Нажмите кнопку `Run`.

![Журналы HTTP для Службы приложений в рабочей области Log Analytics](./media/tutorial-azure-monitor/log-analytics-workspace-app-service-http-logs.png)

Запрос `AppServiceHTTPLogs` возвращает все запросы за последние 24 часа. Столбец `ScStatus` содержит сведения о состоянии HTTP. Чтобы диагностировать ошибки `HTTP 500`, установите для `ScStatus` ограничение 500 и выполните запрос, как показано ниже:

```kusto
AppServiceHTTPLogs
| where ScStatus == 500
```

### <a name="view-appserviceconsolelogs-with-log-query"></a>Просмотр AppServiceConsoleLogs с помощью запросов по журналам

Мы убедились, что возникают ошибки HTTP 500, а теперь перейдем к журналам стандартного потока вывода и ошибок приложения. Эти журналы находятся в AppServiceConsoleLogs.

(1) Щелкните `+`, чтобы создать новый запрос. 

(2) Дважды щелкните таблицу `AppServiceConsoleLogs`, затем выберите `Run`. 

Поскольку преобразование пяти изображений приводит к ошибкам с сервером, вы можете проверить наличие записей ошибок в приложении, установив фильтр по `ResultDescription`, как показано ниже:

```kusto
AppServiceConsoleLogs |
where ResultDescription  contains "error" 
```

В столбце `ResultDescription` отобразится следующая ошибка:

```output
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted 
(tried to allocate 16384 bytes) in /home/site/wwwroot/process.php on line 20, 
referer: http://<app-name>.azurewebsites.net/
```

### <a name="join-appservicehttplogs-and-appserviceconsolelogs"></a>Объединение данных из AppServiceHTTPLogs и AppServiceConsoleLogs

Итак, вы обнаружили как ошибки HTTP 500, так и стандартные ошибки приложения, а теперь нужно проверить наличие связи между этими сообщениями. Для этого вы объедините таблицы по значениям отметки времени (`TimeGenerated`).

> [!NOTE]
> Мы подготовили запрос, который выполняет следующие действия:
>
> - отбор ошибок 500 из журналов HTTP;
> - запрос по журналам консоли;
> - объединение таблиц по значению `TimeGenerated`.
>

Выполните следующий запрос:

```kusto
let myHttp = AppServiceHTTPLogs | where  ScStatus == 500 | project TimeGen=substring(TimeGenerated, 0, 19), CsUriStem, ScStatus;  

let myConsole = AppServiceConsoleLogs | project TimeGen=substring(TimeGenerated, 0, 19), ResultDescription;

myHttp | join myConsole on TimeGen | project TimeGen, CsUriStem, ScStatus, ResultDescription;
```

В столбце `ResultDescription` вы увидите, что следующая ошибка появилась одновременно с ошибками на веб-сервере:

```output
PHP Fatal error:  Allowed memory size of 134217728 bytes exhausted 
(tried to allocate 16384 bytes) in /home/site/wwwroot/process.php on line 20, 
referer: http://<app-name>.azurewebsites.net/
```

Сообщение гласит, что в строке 20 файла `process.php` случилось превышение доступного объема памяти. Теперь вы убедились, что в приложении случилась ошибка в то же время, что и ошибка HTTP 500. Давайте посмотрим на код приложения, чтобы найти проблему.

## <a name="identify-the-error"></a>Определение ошибки

В локальном каталоге откройте файл `process.php` и перейдите в нем к строке 20. 

```php
imagepng($imgArray[$x], $filename);
```

Первый аргумент (`$imgArray[$x]`) является переменной для хранения (в памяти) всех преобразуемых изображений в формате JPG. Но в `imagepng` требуется только одно изображение, которое преобразовывается в текущий момент, а не все сразу. Предварительная загрузка изображений не только не нужна, но и может стать причиной нехватки памяти, приводящей к ошибке HTTP 500. Давайте изменим код, чтобы загружать изображения поочередно, и повторно проверим нашу проблему. Далее вам предстоит улучшить код, чтобы устранить проблему с памятью.

## <a name="fix-the-app"></a>Исправление приложения

### <a name="update-locally-and-redeploy-the-code"></a>Обновление на локальном компьютере и повторное развертывание кода

Внесите следующие изменения в `process.php`, чтобы избавиться от проблемы с нехваткой памяти:

```php
<?php

//Retrieve query parameters
$maxImages = $_GET['images'];
$imgNames  = explode(",",$_GET['imgNames']);

//Load JPEGs into an array (in memory)
for ($x=0; $x<$maxImages; $x++){
    $filename = './images/converted_' . substr($imgNames[$x],0,-4) . '.png';
    imagepng(imagecreatefromjpeg("./images/" . $imgNames[$x]), $filename);
}
```

Зафиксируйте изменения в Git, а затем отправьте изменения кода в Azure.

```bash
git commit -am "Load images on-demand in process.php"
git push azure main
```

### <a name="browse-to-the-azure-app"></a>Переход к приложению Azure

Перейдите по адресу `http://<app-name>.azurewebsites.net`. 

Теперь преобразование изображений не будет вызывать ошибку HTTP 500.

![Приложение PHP, работающее в службе приложений Azure](./media/tutorial-azure-monitor/sample-monitor-app-working.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

Удалите параметры диагностики, выполнив следующую команду:

```bash
az monitor diagnostic-settings delete --resource $resourceID -n myMonitorLogs
```
Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * настройка веб-приложения с помощью Azure Monitor;
> * отправка журналов в Log Analytics;
> * использование запросов по журналам для обнаружения и устранения ошибок в веб-приложениях.

## <a name="next-steps"></a><a name="nextsteps"></a> Дальнейшие действия
* [Запросы по журналам в Azure Monitor](../azure-monitor/logs/log-query-overview.md)
* [Troubleshoot an app in Azure App Service using Visual Studio](troubleshoot-dotnet-visual-studio.md) (Устранение неполадок приложения в Службе приложений Azure с помощью Visual Studio)
* [Анализ журналов приложения в HDInsight](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413)