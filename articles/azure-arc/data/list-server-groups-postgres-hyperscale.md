---
title: Вывод списка групп серверов PostgreSQL с поддержкой дуги Azure, созданных в контроллере данных ARC в Azure
description: Вывод списка групп серверов PostgreSQL с поддержкой дуги Azure, созданных в контроллере данных ARC в Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 7577ca4b8a1d7db7ea99aadfef4fd2a445b66425
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90940787"
---
# <a name="list-the-azure-arc-enabled-postgresql-hyperscale-server-groups-created-in-an-azure-arc-data-controller"></a>Вывод списка групп серверов PostgreSQL с поддержкой дуги Azure, созданных в контроллере данных ARC в Azure

В этой статье объясняется, как получить список групп серверов, созданных в контроллере данных ARC.

Чтобы получить этот список, используйте один из следующих методов после подключения к контроллеру данных ARC:

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="from-cli-with-azdata"></a>Из CLI с аздата
Общий формат команды:
```console
azdata arc postgres server list
```

Он возвратит выходные данные следующего вида:
```console
Name        State    Workers
----------  -------  ---------
postgres01  Ready    2
postgres02  Ready    2
```
Чтобы получить дополнительные сведения о параметрах, доступных для этой команды, выполните следующую команду:
```console
azdata arc postgres server list --help
```

## <a name="from-cli-with-kubectl"></a>Из CLI с kubectl
Выполните одну из следующих команд.

**Чтобы вывести список групп серверов независимо от версии postgres, выполните:**
```console
kubectl get postgresqls
```
Он возвратит выходные данные следующего вида:
```console
NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      51s
postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d
```

**Чтобы получить список групп серверов определенной версии postgres, выполните:**
```console
kubectl get postgresql-12
```

Чтобы получить список групп серверов, работающих под управлением версии 11 из postgres, замените _PostgreSQL-12_ на _PostgreSQL-11_.

## <a name="next-steps"></a>Дальнейшие действия

* [Прочитайте статью о том, как получить конечные точки подключения и сформировать строки подключения для подключения к группе серверов.](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md)
* [Прочитайте статью о том, как настроить группу серверов PostgreSQL в службе "Дуга Azure" с поддержкой ARC.](show-configuration-postgresql-hyperscale-server-group.md)
