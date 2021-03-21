---
title: Отправка данных об использовании в Azure Monitor
description: Отправка данных об использовании Azure ARC с включенными данными служб данных в Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: 0c72eda59f375c70274b17796ca53614ef95505b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104669514"
---
# <a name="upload-usage-data-to-azure-monitor"></a>Отправка данных об использовании в Azure Monitor

Периодически можно экспортировать сведения об использовании. Экспорт и передача этих данных создает и обновляет контроллеры данных, управляемый экземпляр SQL и PostgreSQL масштабируемые ресурсы группы серверов в Azure.

> [!NOTE] 
> В течение периода действия предварительной версии не взимается плата за использование служб данных, включенных в службу Arc Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]


> [!NOTE]
> Подождите не менее 24 часов после создания контроллера данных ARC в Azure перед отправкой данных об использовании.

## <a name="create-service-principal-and-assign-roles"></a>Создание субъекта-службы и назначение ролей

Прежде чем продолжать, убедитесь, что вы создали требуемый субъект-службу и назначили его соответствующей роли. Подробная информация доступна в следующих статьях:
* [Создание субъекта-службы](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal).
* [Назначение ролей субъекту-службе](upload-metrics-and-logs-to-azure-monitor.md#assign-roles-to-the-service-principal)

## <a name="upload-usage-data"></a>Отправка данных об использовании

Сведения об использовании, такие как Инвентаризация и использование ресурсов, можно отправить в Azure следующим образом:

1. Войдите в контроллер данных. Введите значения в командной строке. 

   ```console
   azdata login
   ```

1. Экспортируйте данные об использовании с помощью `azdata arc dc export` команды следующим образом:

   ```console
   azdata arc dc export --type usage --path usage.json
   ```
 
   Эта команда создает `usage.json` файл со всеми ресурсами данных, включенными в дугу Azure, такими как управляемые экземпляры SQL и PostgreSQL экземпляров для масштабирования и т. д., которые создаются на контроллере данных.

2. Отправка данных об использовании с помощью ```azdata upload``` команды

   ```console
   azdata arc dc upload --path usage.json
   ```

## <a name="automating-uploads-optional"></a>Автоматизация отправки (необязательно)

Если вы хотите отправлять метрики и журналы по расписанию, можно создать сценарий и запустить его через таймер каждые несколько минут. Ниже приведен пример автоматизации отправки с помощью сценария оболочки Linux.

В любом редакторе текста или кода добавьте следующий скрипт в файл и сохраните его как исполняемый файл скрипта, например `.sh` (Linux/Mac) или `.cmd` , `.bat` или `.ps1` .

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

[Отправка журналов в Azure Monitor](upload-logs.md)

[Отправьте данные о выставлении счетов в Azure и просмотрите их в портал Azure](view-billing-data-in-azure.md)

[Просмотр ресурса контроллера данных Arc Azure в портал Azure](view-data-controller-in-azure-portal.md)
