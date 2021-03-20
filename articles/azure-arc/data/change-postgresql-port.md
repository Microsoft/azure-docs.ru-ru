---
title: Изменение порта PostgreSQL
description: Измените порт, на котором PostgreSQLа группа серверов Microsoft Azure Arc Enabled.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 11/02/2020
ms.topic: how-to
ms.openlocfilehash: 45424408c790e4921be48464818c55fe74313fd3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93328756"
---
# <a name="change-the-port-on-which-the-server-group-is-listening"></a>Изменение порта, на котором прослушивается группа серверов 

Изменение порта является стандартной операцией по изменению группы серверов. Чтобы изменить порт, выполните следующую команду:
```console
 azdata arc postgres server edit -n <server group name> --port <desired port number>
```

Например, предположим, что имя группы серверов — _postgres01_ , и вы хотите, чтобы он прослушивает порт _866_. Выполните следующую команду:
```console
 azdata arc postgres server edit -n postgres01 --port 866
```

## <a name="verify-that-the-port-was-changed"></a>Убедитесь, что порт был изменен

Чтобы убедиться, что порт был изменен, выполните следующую команду, чтобы отобразить конфигурацию группы серверов:
```console
azdata arc postgres server show -n <server group name>
```

В выходных данных этой команды найдите номер порта, отображаемый для элемента "порт" в разделе "служба" спецификации вашей группы серверов.
Кроме того, можно проверить в элементе Екстерналендпоинт в разделе состояние спецификаций вашей группы серверов, за которым следует номер порта, который вы настроили.

В качестве иллюстрации, если мы продолжим приведенный выше пример, выполните команду:
```console
azdata arc postgres server show -n postgres01
```

Вы увидите порт 866, на который указывает ссылка:

```console
"service": {
      "port": 866,
      "type": "LoadBalancer"
    },
```
и здесь.

```console
"status": {
    "externalEndpoint": "12.678.345.09:866",
    "logSearchDashboard": "https://12.345.678.90:30777/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:postgres01'))",
    "metricsDashboard": "https://12.345.678.90:30777/grafana/d/postgres-metrics?var-Namespace=arc&var-Name=postgres01",
    "readyPods": "3/3",
    "state": "Ready"
  }
```
## <a name="next-steps"></a>Дальнейшие действия
- Узнайте [, как подключиться к группе серверов](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md).
- Дополнительные сведения о настройке других аспектов группы серверов см. в разделе хов-то\манаже\конфигуре & Scale раздела документации.
