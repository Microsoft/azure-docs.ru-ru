---
title: Получение журналов для устранения неполадок служб данных, включенных в службу Arc Azure
description: Узнайте, как получить файлы журналов из контроллера данных для устранения неполадок в службах данных, включенных в службу Arc Azure.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0c4cff7583f08fe27649cee464fcef802cddd88f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93234055"
---
# <a name="get-logs-to-troubleshoot-azure-arc-enabled-data-services"></a>Получение журналов для устранения неполадок служб данных, включенных в службу Arc Azure

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Предварительные условия

Прежде чем продолжать, вам потребуется:

* [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]. Дополнительные сведения см. в [статье Установка клиентских средств для развертывания служб данных Azure Arc и управления ими](./install-client-tools.md).
* Учетная запись администратора для входа в контроллер данных с поддержкой дуги Azure.

## <a name="get-log-files"></a>Получение файлов журнала

Для устранения неполадок можно получить журналы служб для всех модулей Pod или отдельных модулей. Один из способов — использовать стандартные средства Kubernetes, такие как `kubectl logs` команда. В этой статье вы будете использовать [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] средство, которое упрощает получение всех журналов.

1. Войдите в контроллер данных с помощью учетной записи администратора.

   ```console
   azdata login
   ```

2. Выполните следующую команду, чтобы создать дамп журналов:

   ```console
   azdata arc dc debug copy-logs --namespace <namespace name> --exclude-dumps --skip-compress
   ```

   Пример:

   ```console
   #azdata arc dc debug copy-logs --namespace arc --exclude-dumps --skip-compress
   ```

Контроллер данных создает файлы журнала в текущем рабочем каталоге в подкаталоге с именем `logs` . 

## <a name="options"></a>Параметры

`azdata arc dc debug copy-logs`Команда предоставляет следующие параметры для управления выходными данными:

* Выводит файлы журнала в другой каталог с помощью `--target-folder` параметра.
* Сжатие файлов путем пропуска `--skip-compress` параметра.
* Активируйте и включите дампы памяти, опустив `--exclude-dumps` . Этот метод не рекомендуется использовать, если только служба поддержки Майкрософт не запросил дампы памяти. Для получения дампа памяти необходимо, чтобы параметр контроллера данных `allowDumps` был установлен в значение `true` при создании контроллера данных.
* Фильтр для получения журналов только для определенного Pod ( `--pod` ) или контейнера ( `--container` ) по имени.
* Фильтр для получения журналов для определенного настраиваемого ресурса путем передачи `--resource-kind` параметров и `--resource-name` . `resource-kind`Значение параметра должно быть одним из имен настраиваемых определений ресурсов. Эти имена можно получить с помощью команды `kubectl get customresourcedefinition` .

С помощью этих параметров можно заменить `<parameters>` в следующем примере: 

```console
azdata arc dc debug copy-logs --target-folder <desired folder> --exclude-dumps --skip-compress -resource-kind <custom resource definition name> --resource-name <resource name> --namespace <namespace name>
```

Пример:

```console
#azdata arc dc debug copy-logs --target-folder C:\temp\logs --exclude-dumps --skip-compress --resource-kind postgresql-12 --resource-name pg1 --namespace arc
```

Примером является следующая иерархия папок. Он упорядочивается по имени POD, затем контейнеру, а затем по иерархии каталогов в контейнере.

```output
<export directory>
├───debuglogs-arc-20200827-180403
│   ├───bootstrapper-vl8j2
│   │   └───bootstrapper
│   │       ├───apt
│   │       └───fsck
│   ├───control-j2dm5
│   │   ├───controller
│   │   │   └───controller
│   │   │       ├───2020-08-27
│   │   │       └───2020-08-28
│   │   └───fluentbit
│   │       ├───agent
│   │       ├───fluentbit
│   │       └───supervisor
│   │           └───log
│   ├───controldb-0
│   │   ├───fluentbit
│   │   │   ├───agent
│   │   │   ├───fluentbit
│   │   │   └───supervisor
│   │   │       └───log
│   │   └───mssql-server
│   │       ├───agent
│   │       ├───mssql
│   │       ├───mssql-server
│   │       └───supervisor
│   │           └───log
│   ├───controlwd-ln6j8
│   │   └───controlwatchdog
│   │       └───controlwatchdog
│   ├───logsdb-0
│   │   └───elasticsearch
│   │       ├───agent
│   │       ├───elasticsearch
│   │       ├───provisioner
│   │       └───supervisor
│   │           └───log
│   ├───logsui-7gg2d
│   │   └───kibana
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───kibana
│   │       └───supervisor
│   │           └───log
│   ├───metricsdb-0
│   │   └───influxdb
│   │       ├───agent
│   │       ├───influxdb
│   │       └───supervisor
│   │           └───log
│   ├───metricsdc-2f62t
│   │   └───telegraf
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───supervisor
│   │       │   └───log
│   │       └───telegraf
│   ├───metricsdc-jznd2
│   │   └───telegraf
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───supervisor
│   │       │   └───log
│   │       └───telegraf
│   ├───metricsdc-n5vnx
│   │   └───telegraf
│   │       ├───agent
│   │       ├───apt
│   │       ├───fsck
│   │       ├───supervisor
│   │       │   └───log
│   │       └───telegraf
│   ├───metricsui-h748h
│   │   └───grafana
│   │       ├───agent
│   │       ├───grafana
│   │       └───supervisor
│   │           └───log
│   └───mgmtproxy-r5zxs
│       ├───fluentbit
│       │   ├───agent
│       │   ├───fluentbit
│       │   └───supervisor
│       │       └───log
│       └───service-proxy
│           ├───agent
│           ├───nginx
│           └───supervisor
│               └───log
└───debuglogs-kube-system-20200827-180431
    ├───coredns-8bbb65c89-kklt7
    │   └───coredns
    ├───coredns-8bbb65c89-z2vvr
    │   └───coredns
    ├───coredns-autoscaler-5585bf8c9f-g52nt
    │   └───autoscaler
    ├───kube-proxy-5c9s2
    │   └───kube-proxy
    ├───kube-proxy-h6x56
    │   └───kube-proxy
    ├───kube-proxy-nd2b7
    │   └───kube-proxy
    ├───metrics-server-5f54b8994-vpm5r
    │   └───metrics-server
    └───tunnelfront-db87f4cd8-5xwxv
        ├───tunnel-front
        │   ├───apt
        │   └───journal
        └───tunnel-probe
            ├───apt
            ├───journal
            └───openvpn
```

## <a name="next-steps"></a>Дальнейшие действия

[azdata arc dc debug copy-logs](/sql/azdata/reference/reference-azdata-arc-dc-debug#azdata-arc-dc-debug-copy-logs?toc=/azure/azure-arc/data/toc.json&bc=/azure/azure-arc/data/breadcrumb/toc.json)
