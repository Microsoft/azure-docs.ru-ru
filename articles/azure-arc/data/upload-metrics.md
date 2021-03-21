---
title: Отправка метрик в Azure Monitor
description: Отправка метрик служб данных, включенных в дугу Azure, в Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: d7c611f1cdb5e3294e38f87c0534003813e50388
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100575689"
---
# <a name="upload-metrics-to-azure-monitor"></a>Отправка метрик в Azure Monitor

Периодически можно экспортировать метрики мониторинга, а затем передать их в Azure. При экспорте и отправке данных также создаются и обновляются контроллеры данных, управляемый экземпляр SQL и PostgreSQL в Azure ресурсы группы серверов масштабирования.

> [!NOTE] 
> В течение периода действия предварительной версии не взимается плата за использование служб данных, включенных в службу Arc Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Предварительные условия

Прежде чем продолжать, убедитесь, что вы создали требуемый субъект-службу и назначили его соответствующей роли. Подробная информация доступна в следующих статьях:
* [Создание субъекта-службы](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal).
* [Назначение ролей субъекту-службе](upload-metrics-and-logs-to-azure-monitor.md#assign-roles-to-the-service-principal)

## <a name="upload-metrics"></a>Отправка метрик

С помощью служб данных Arc Azure можно дополнительно передать метрики в Azure Monitor, чтобы можно было выполнять статистическую обработку и анализ метрик, создавать оповещения, отправлять уведомления или запускать автоматические действия. 

Отправка данных в Azure Monitor позволяет хранить данные метрик вне сайта и в огромном масштабе, обеспечивая долгосрочное хранение данных для расширенной аналитики.

Если у вас есть несколько сайтов со службами данных Arc Azure, вы можете использовать Azure Monitor в качестве центрального расположения для сбора всех журналов и метрик на сайтах.

## <a name="set-final-environment-variables-and-confirm"></a>Задание конечных переменных среды и подтверждение

Задайте URL-адрес центра SPN в переменной среды:

::: zone pivot="client-operating-system-windows-command"

```console
SET SPN_AUTHORITY=https://login.microsoftonline.com
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end

Убедитесь, что все необходимые переменные среды заданы при необходимости.


::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:SPN_TENANT_ID
$Env:SPN_CLIENT_ID
$Env:SPN_CLIENT_SECRET
$Env:SPN_AUTHORITY
```


::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
echo $SPN_TENANT_ID
echo $SPN_CLIENT_ID
echo $SPN_CLIENT_SECRET
echo $SPN_AUTHORITY
```

::: zone-end

::: zone pivot="client-operating-system-windows-command"

```console
echo %SPN_TENANT_ID%
echo %SPN_CLIENT_ID%
echo %SPN_CLIENT_SECRET%
echo %SPN_AUTHORITY%
```

::: zone-end

## <a name="upload-metrics-to-azure-monitor"></a>Отправка метрик в Azure Monitor

Чтобы передать метрики для управляемых экземпляров SQL, включенных в службу "Дуга Azure", и службы "Дуга" PostgreSQL, выполните следующие команды интерфейса командной строки:

1. Войдите в контроллер данных с помощью `azdata` .
 
1. Экспортировать все метрики в указанный файл:

   ```console
   azdata arc dc export --type metrics --path metrics.json
   ```

2. Отправка метрик в Azure Monitor:

   ```console
   azdata arc dc upload --path metrics.json
   ```

   >[!NOTE]
   >Подождите не менее 30 минут после создания экземпляров данных с поддержкой Arc Azure для первой передачи.
   >
   >Убедитесь `upload` , что метрики сразу после `export` Azure Monitor принимают только метрики за последние 30 минут. [Подробнее](../../azure-monitor/essentials/metrics-store-custom-rest-api.md#troubleshooting).


Если при экспорте отображаются ошибки, указывающие на неудачу получения метрик, `true` выполните следующую команду:

```console
azdata arc dc config show
```

В разделе "раздел безопасности"

```output
 "security": {
      "allowDumps": true,
      "allowNodeMetricsCollection": true,
      "allowPodMetricsCollection": true,
      "allowRunAsRoot": false
    },
```

Проверьте, `allowNodeMetricsCollection` установлены ли `allowPodMetricsCollection` для свойств и значение `true` .

## <a name="view-the-metrics-in-the-portal"></a>Просмотр метрик на портале

После отправки метрик можно просмотреть в портал Azure.
> [!NOTE]
> Обратите внимание, что перед просмотром метрик на портале может потребоваться несколько минут для обработки отправленных данных.


Чтобы просмотреть метрики на портале, используйте эту ссылку, чтобы открыть портал. <https://portal.azure.com> затем выполните поиск экземпляра базы данных по имени в строке поиска:

Вы можете просмотреть загрузку ЦП на странице "Обзор" или, если вы хотите получить более подробные метрики, щелкнув метрики на левой навигационной панели.

Выберите SQL Server в качестве пространства имен метрики:

Выберите метрику, которую необходимо визуализировать (можно также выбрать несколько):

Измените частоту до 30 минут:

> [!NOTE]
> Только метрики можно передать только за последние 30 минут. Azure Monitor отклоняет метрики старше 30 минут.

## <a name="automating-uploads-optional"></a>Автоматизация отправки (необязательно)

Если вы хотите отправлять метрики и журналы по расписанию, можно создать сценарий и запустить его через таймер каждые несколько минут. Ниже приведен пример автоматизации отправки с помощью сценария оболочки Linux.

В любом редакторе текста или кода добавьте следующий скрипт в файл и сохраните его как исполняемый файл сценария, например. sh (Linux/Mac) или. cmd,. bat,. ps1.

```console
azdata arc dc export --type metrics --path metrics.json --force
azdata arc dc upload --path metrics.json
```

Сделать исполняемый файл скрипта

```console
chmod +x myuploadscript.sh
```

Запускать скрипт каждые 20 минут:

```console
watch -n 1200 ./myuploadscript.sh
```

Можно также использовать планировщик заданий, например cron или Windows планировщик задач или Orchestrator, например Ansible, Puppet или Chef.

## <a name="general-guidance-on-exporting-and-uploading-usage-metrics"></a>Общие рекомендации по экспорту и передаче сведений об использовании, метриках

Операции создания, чтения, обновления и удаления (CRUD) в службе данных с включенной службой Arc Azure регистрируются в целях выставления счетов и мониторинга. Существуют фоновые службы, которые отслеживают эти операции CRUD и вычисляют потребление соответствующим образом. Фактическое вычисление использования или потребления происходит по расписанию и выполняется в фоновом режиме. 

Во время действия предварительной версии этот процесс происходит ночью. Общее руководство заключается в передаче сведений об использовании только один раз в день. Если сведения об использовании экспортируются и передаются несколько раз в течение одного 24-часового периода, то в портал Azure, но не на использование ресурсов обновляется только Инвентаризация ресурсов.

Для отправки метрик Azure Monitor принимает только последние 30 минут данных (дополнительные[сведения](../../azure-monitor/essentials/metrics-store-custom-rest-api.md#troubleshooting)). Рекомендации по передаче метрик можно передать сразу после создания файла экспорта, чтобы вы могли просмотреть весь набор данных в портал Azure. Например, если вы экспортировали метрики в 2:00 PM и выполнили команду upload в 2:50 РМ. Поскольку Azure Monitor принимает данные только за последние 30 минут, на портале могут не отображаться данные. 

## <a name="next-steps"></a>Дальнейшие действия

[Отправка журналов в Azure Monitor](upload-logs.md)

[Передача данных об использовании, метрик и журналов в Azure Monitor](upload-usage-data.md)

[Отправьте данные о выставлении счетов в Azure и просмотрите их в портал Azure](view-billing-data-in-azure.md)

[Просмотр ресурса контроллера данных Arc Azure в портал Azure](view-data-controller-in-azure-portal.md)
