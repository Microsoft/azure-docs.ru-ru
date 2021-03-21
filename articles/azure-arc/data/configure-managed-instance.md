---
title: Настройка управляемого экземпляра SQL с поддержкой ARC в Azure
description: Настройка управляемого экземпляра SQL с поддержкой ARC в Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 6e2443014f6788504a11784945078187a5a72de4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98985875"
---
# <a name="configure-azure-arc-enabled-sql-managed-instance"></a>Настройка управляемого экземпляра SQL с поддержкой ARC в Azure

В этой статье объясняется, как настроить управляемый экземпляр SQL с поддержкой ARC.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="configure-resources"></a>Настройка ресурсов

### <a name="configure-using-azure-data-cli-azdata"></a>Настройка с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

Вы можете изменить конфигурацию управляемых экземпляров SQL, включенных в службу "Дуга Azure", с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] . Выполните следующую команду, чтобы просмотреть параметры конфигурации. 

```
azdata arc sql mi edit --help
```

В следующем примере задаются ограничения на ядро ЦП и запросы памяти.

```
azdata arc sql mi edit --cores-limit 4 --cores-request 2 --memory-limit 4Gi --memory-request 2Gi -n <NAME_OF_SQL_MI>
```

Чтобы просмотреть изменения, внесенные в управляемый экземпляр SQL, можно использовать следующие команды для просмотра файла конфигурации YAML:

```
azdata arc sql mi show -n <NAME_OF_SQL_MI>
```

## <a name="configure-server-options"></a>Настройка параметров сервера

Вы можете настроить параметры конфигурации сервера для управляемого экземпляра SQL, поддерживающего Arc, после времени создания. В этой статье описывается, как настроить такие параметры, как включение или отключение агента MSSQL, включение конкретных флагов трассировки в сценарии устранения неполадок.

Чтобы изменить какие либо из этих параметров, выполните следующие действия.

1. Создайте пользовательский файл `mssql-custom.conf`, включающий нужные параметры. В следующем примере включается агент SQL и включается флаг трассировки 1204.:

   ```
   [sqlagent]
   enabled=true
   
   [traceflag]
   traceflag0 = 1204
   ```

1. Скопируйте файл `mssql-custom.conf` в каталог `/var/opt/mssql` в контейнере в `mssql-miaa` pod `master-0`. Замените `<namespaceName>` именем кластера больших данных.

   ```bash
   kubectl cp mssql-custom.conf master-0:/var/opt/mssql/mssql-custom.conf -c mssql-server -n <namespaceName>
   ```

1. Перезапустите экземпляр SQL Server.  Замените `<namespaceName>` именем кластера больших данных.

   ```bash
   kubectl exec -it master-0  -c mssql-server -n <namespaceName> -- /bin/bash
   supervisorctl restart mssql-server
   exit
   ```


**Известные ограничения**
- Для выполнения приведенных выше действий требуются разрешения администратора кластера Kubernetes.
- Это может измениться на протяжении всего предварительной версии
