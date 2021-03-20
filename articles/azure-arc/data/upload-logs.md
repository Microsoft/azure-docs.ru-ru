---
title: Отправка журналов в Azure Monitor
description: Отправка журналов для служб данных, включенных в службу Arc Azure, в Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: f3f29ae1ab868a96e6f70ed964f79c47bc591c4d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92378145"
---
# <a name="upload-logs-to-azure-monitor"></a>Отправка журналов в Azure Monitor

Периодически можно экспортировать журналы, а затем передать их в Azure. При экспорте и отправке журналов также создаются и обновляются контроллеры данных, управляемый экземпляр SQL и PostgreSQL в Azure ресурсы группы серверов масштабирования.

> [!NOTE] 
> В течение периода действия предварительной версии не взимается плата за использование служб данных, включенных в службу Arc Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="before-you-begin"></a>Перед началом

Перед отправкой журналов необходимо выполнить следующие действия. 

1. [Создание рабочей области log Analytics](#create-a-log-analytics-workspace)
1. [Назначение ИДЕНТИФИКАТОРов и общего ключа переменным среды](#assign-id-and-shared-key-to-environment-variables)

## <a name="create-a-log-analytics-workspace"></a>Создание рабочей области log Analytics

Чтобы создать рабочую область log Analytics, выполните следующие команды, чтобы создать рабочую область Log Analytics и задать сведения о доступе в переменных среды.

> [!NOTE]
> Пропустите этот шаг, если у вас уже есть рабочая область.

```azurecli
az monitor log-analytics workspace create --resource-group <resource group name> --workspace-name <some name you choose>
```

Выходные данные примера:

```output
{
  "customerId": "d6abb435-2626-4df1-b887-445fe44a4123",
  "eTag": null,
  "id": "/subscriptions/<Subscription ID>/resourcegroups/user-arc-demo/providers/microsoft.operationalinsights/workspaces/user-logworkspace",
  "location": "eastus",
  "name": "user-logworkspace",
  "portalUrl": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "user-arc-demo",
  "retentionInDays": 30,
  "sku": {
    "lastSkuUpdate": "Thu, 30 Jul 2020 22:37:53 GMT",
    "maxCapacityReservationLevel": 3000,
    "name": "pergb2018"
  },
  "source": "Azure",
  "tags": null,
  "type": "Microsoft.OperationalInsights/workspaces"
}
```

## <a name="assign-id-and-shared-key-to-environment-variables"></a>Назначение ИДЕНТИФИКАТОРов и общего ключа переменным среды

Сохраните журнал рабочей области журнала в `customerId` качестве переменной среды, которая будет использоваться позже:

::: zone pivot="client-operating-system-windows-command"

```console
SET WORKSPACE_ID=<customerId>
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:WORKSPACE_ID='<customerId>'
```
::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export WORKSPACE_ID='<customerId>'
```
::: zone-end

Эта команда возвращает ключи доступа, необходимые для подключения к рабочей области log Analytics:

```azurecli
az monitor log-analytics workspace get-shared-keys --resource-group MyResourceGroup --workspace-name MyLogsWorkpace
```

Выходные данные примера:

```output
{
  "primarySharedKey": "JXzQp1RcGgjXFCDS3v0sXoxPvbgCoGaIv35lf11Km2WbdGFvLXqaydpaj1ByWGvKoCghL8hL4BRoypXxkLr123==",
  "secondarySharedKey": "p2XHSxLJ4o9IAqm2zINcEmx0UWU5Z5EZz8PQC0OHpFjdpuVaI0zsPbTv5VyPFgaCUlCZb2yEbkiR4eTuTSF123=="
}
```

Сохраните первичный ключ в переменной среды, которая будет использоваться позже:

::: zone pivot="client-operating-system-windows-command"

```console
SET WORKSPACE_SHARED_KEY=<primarySharedKey>
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```console
$Env:WORKSPACE_SHARED_KEY='<primarySharedKey>'
```
```
::: zone-end


::: zone pivot="client-operating-system-macos-and-linux"

```console
export WORKSPACE_SHARED_KEY='<primarySharedKey>'
```

::: zone-end

## <a name="set-final-environment-variables-and-confirm"></a>Задание конечных переменных среды и подтверждение

Задайте URL-адрес центра SPN в переменной среды:

::: zone pivot="client-operating-system-windows-command"

```console
SET SPN_AUTHORITY=https://login.microsoftonline.com
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```console
$Env:SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export SPN_AUTHORITY='https://login.microsoftonline.com'
```

::: zone-end


## <a name="verify-environment-variables"></a>Проверка переменных среды

Убедитесь, что все необходимые переменные среды заданы при необходимости.


::: zone pivot="client-operating-system-windows-command"

```console
echo %WORKSPACE_ID%
echo %WORKSPACE_SHARED_KEY%
echo %SPN_TENANT_ID%
echo %SPN_CLIENT_ID%
echo %SPN_CLIENT_SECRET%
echo %SPN_AUTHORITY%
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```PowerShell
$Env:WORKSPACE_ID
$Env:WORKSPACE_SHARED_KEY
$Env:SPN_TENANT_ID
$Env:SPN_CLIENT_ID
$Env:SPN_CLIENT_SECRET
$Env:SPN_AUTHORITY
```


::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
echo $WORKSPACE_ID
echo $WORKSPACE_SHARED_KEY
echo $SPN_TENANT_ID
echo $SPN_CLIENT_ID
echo $SPN_CLIENT_SECRET
echo $SPN_AUTHORITY
```

::: zone-end

После установки переменных среды можно передать журналы в рабочую область журнала. 

## <a name="upload-logs-to-azure-monitor"></a>Отправка журналов в Azure Monitor

 Чтобы отправить журналы для управляемых экземпляров SQL, включенных в службу "Дуга Azure", и Азуреарк Enabled PostgreSQL. группы серверов масштабирования выполняют следующие команды CLI

1. Войдите в систему на контроллере данных ARC в Azure с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] .

   ```console
   azdata login
   ```

   Следуйте инструкциям на экране, чтобы задать пространство имен, имя пользователя администратора и пароль. 

1. Экспортировать все журналы в указанный файл:

   ```console
   azdata arc dc export --type logs --path logs.json
   ```

2. Отправка журналов в рабочую область журнала Azure Monitor Analytics:

   ```console
   azdata arc dc upload --path logs.json
   ```

## <a name="view-your-logs-in-azure-portal"></a>Просмотр журналов в портал Azure

К отправленным журналам можно обращаться с помощью обозревателя запросов журналов следующим образом:

1. Откройте портал Azure и выполните поиск рабочей области по имени в строке поиска вверху, а затем выберите ее.
2. Выберите журналы на панели слева.
3. Выберите начало работы (или щелкните ссылки на странице начало работы, чтобы узнать больше о Log Analytics, если вы еще не знакомы с ней).
4. Следуйте указаниям учебника, чтобы получить дополнительные сведения о Log Analytics, если вы впервые используете Log Analytics.
5. Разверните раздел "Пользовательские журналы" в нижней части списка таблиц. Вы увидите таблицу с именем sql_instance_logs_CL.
6. Щелкните значок «глаз» рядом с именем таблицы.
7. Нажмите кнопку "просмотреть в редакторе запросов".
8. Теперь у вас есть запрос в редакторе запросов, который будет отображать последние 10 событий в журнале.
9. Здесь можно поэкспериментировать с отправкой запросов к журналам с помощью редактора запросов, настроить оповещения и т. д.

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

## <a name="next-steps"></a>Дальнейшие действия

[Передача метрик и журналов в Azure Monitor](upload-metrics.md)

[Передача данных об использовании, метрик и журналов в Azure Monitor](upload-usage-data.md)

[Отправьте данные о выставлении счетов в Azure и просмотрите их в портал Azure](view-billing-data-in-azure.md)

[Просмотр ресурса контроллера данных Arc Azure в портал Azure](view-data-controller-in-azure-portal.md)
