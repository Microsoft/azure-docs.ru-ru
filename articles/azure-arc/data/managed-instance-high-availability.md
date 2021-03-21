---
title: Управляемый экземпляр с поддержкой Azure Arc и высокой доступностью
titleSuffix: Deploy Azure Arc enabled Managed Instance with high availability
description: Узнайте, как развернуть Управляемый экземпляр с поддержкой ARC в Azure с высокой доступностью.
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 03/02/2021
ms.topic: conceptual
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.openlocfilehash: 92f5c900238fc5d40e22870e2f00f8adeb5d335f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102032200"
---
# <a name="azure-arc-enabled-managed-instance-high-availability"></a>Управляемый экземпляр с поддержкой Azure Arc и высокой доступностью

Управляемый экземпляр с поддержкой дуги Azure развертывается на Kubernetes как контейнерное приложение и использует конструкции Kubernetes, такие как наборы с отслеживанием состояния и постоянное хранилище, для обеспечения встроенного мониторинга работоспособности, обнаружения сбоев и механизмов отработки отказа для поддержания работоспособности служб. Для повышения надежности можно также настроить Управляемый экземпляр, включенную в дугу Azure, для развертывания с дополнительными репликами в конфигурации высокой доступности. Мониторинг, обнаружение сбоев и автоматический переход на другой ресурс управляются контроллером данных служб данных ARC. Эта служба обеспечивает все действия без участия пользователя: от настройки группы доступности и конечных точек зеркального отображения баз данных до добавления баз данных в группу доступности или координации отработки отказа и обновления. В этом документе рассматриваются оба типа обеспечения высокой доступности.

## <a name="built-in-high-availability"></a>высокую доступность; 

Встроенная Высокая доступность обеспечивается Kubernetes, когда удаленное постоянное хранилище настроено и используется совместно с узлами, используемыми при развертывании службы данных ARC. В этой конфигурации Kubernetes играет роль оркестратора кластера. При сбое управляемого экземпляра в контейнере или базовом узле Orchestrator загружает другой экземпляр контейнера и подключается к тому же постоянному хранилищу. Этот тип включен по умолчанию при развертывании Управляемый экземпляр с поддержкой дуги Azure.

### <a name="verify-built-in-high-availability"></a>Проверка высокого уровня доступности

В этом разделе вы проверите встроенную высокую доступность, предоставляемую Kubernetes. Когда вы выполните действия по тестированию этой функции, удалите модуль существующего управляемого экземпляра и убедитесь, что Kubernetes восстанавливается из этого действия. 

### <a name="prerequisites"></a>Предварительные условия

- Кластер Kubernetes должен иметь [Общее удаленное хранилище](storage-configuration.md#factors-to-consider-when-choosing-your-storage-configuration) 
- Управляемый экземпляр с поддержкой дуги Azure, развернутой с одной реплики (по умолчанию)

1. Просмотр модулей Pod. 

   ```console
   kubectl get pods -n <namespace of data controller>
   ```

2. Удалите экземпляр Pod управляемого экземпляра.

   ```console
   kubectl delete pod <name of managed instance>-0 -n <namespace of data controller>
   ```

   Например:

   ```output
   user@pc:/# kubectl delete pod sql1-0 -n arc
   pod "sql1-0" deleted
   ```

3. Просмотрите модули Pod, чтобы убедиться, что управляемый экземпляр восстанавливается.

   ```console
   kubectl get pods -n <namespace of data controller>
   ```

   Например:

   ```output
   user@pc:/# kubectl get pods -n arc
   NAME                 READY   STATUS    RESTARTS   AGE
   sql1-0               2/3     Running   0          22s
   ```

После восстановления всех контейнеров в Pod можно подключиться к управляемому экземпляру.

## <a name="deploy-with-always-on-availability-groups"></a>Развертывание с помощью группы доступности Always On

Для повышения надежности можно настроить Управляемый экземпляр, включенную в дугу Azure, для развертывания с дополнительными репликами в конфигурации высокой доступности. 

Возможности, которые включает группа доступности.

- При развертывании с несколькими репликами создается одна группа доступности с именем `containedag` . По умолчанию группа доступности `containedag` имеет три реплики, включая первичную. Все операции CRUD для группы доступности управляются внутренним образом, в том числе создание группы доступности или присоединение реплик к созданной группе доступности. Невозможно создать дополнительные группы доступности в Управляемый экземпляр, включенной в дугу Azure.

- Все базы данных автоматически добавляются в группу доступности, в том числе все пользовательские и системные базы данных, такие как `master` и `msdb`. Эта возможность обеспечивает представление единой системы во всех репликах группы доступности. Обратите внимание `containedag_master` и на `containedag_msdb` базы данных, и при подключении напрямую к экземпляру. Базы данных `containedag_*` представляют `master` и `msdb` в группе доступности.

- Внешняя конечная точка автоматически подготавливается для подключения к базам данных в группе доступности. Эта конечная точка `<managed_instance_name>-svc-external` выполняет роль прослушивателя группы доступности.

### <a name="deploy"></a>Развернуть

Чтобы развернуть управляемый экземпляр с группами доступности, выполните следующую команду.

```console
azdata arc sql mi create -n <name of instance> --replicas 3
```

### <a name="check-status"></a>Проверка состояния
После развертывания экземпляра выполните следующие команды, чтобы проверить состояние экземпляра:

```console
azdata arc sql mi list
azdata arc sql mi show -n <name of instance>
```

Выходные данные примера:

```output
user@pc:/# azdata arc sql mi list
ExternalEndpoint    Name    Replicas    State
------------------  ------  ----------  -------
20.131.31.58,1433   sql2    3/3         Ready

user@pc:/#  azdata arc sql mi show -n sql2
{
...
  "status": {
    "AGStatus": "Healthy",
    "externalEndpoint": "20.131.31.58,1433",
    "logSearchDashboard": "link to logs dashboard",
    "metricsDashboard": "link to metrics dashboard",
    "readyReplicas": "3/3",
    "state": "Ready"
  }
}
```

Обратите внимание на дополнительное число `Replicas` и `AGstatus` поле, указывающее на работоспособность группы доступности. Если все реплики синхронизированы, то это значение равно `healthy` . 

### <a name="restore-a-database"></a>Восстановление базы данных 
Для восстановления базы данных в группу доступности требуются дополнительные действия. В следующих шагах показано, как восстановить базу данных в управляемый экземпляр и добавить ее в группу доступности. 

1. Предоставьте внешнюю конечную точку первичного экземпляра, создав новую службу Kubernetes.

    Определите Pod, на котором размещена первичная реплика, подключившись к управляемому экземпляру и запустите:

    ```sql
    SELECT @@SERVERNAME
    ```
    Создайте службу kubernetes для основного экземпляра, выполнив следующую команду, если кластер kubernetes использует службы Нодепорт. Замените на `podName` имя сервера, возвращенного на предыдущем шаге, `serviceName` с предпочтительным именем для созданной службы Kubernetes.

    ```bash
    kubectl -n <namespaceName> expose pod <podName> --port=1533  --name=<serviceName> --type=NodePort
    ```

    Для службы балансировщика нагрузки выполните ту же команду, за исключением того, что созданная служба имеет тип `LoadBalancer` . Пример: 

    ```bash
    kubectl -n <namespaceName> expose pod <podName> --port=1533  --name=<serviceName> --type=LoadBalancer
    ```

    Ниже приведен пример выполнения этой команды в службе Azure Kubernetes, где в качестве основного модуля будет размещаться `sql2-0` :

    ```bash
    kubectl -n arc-cluster expose pod sql2-0 --port=1533  --name=sql2-0-p --type=LoadBalancer
    ```

    Получите IP-адрес созданной службы Kubernetes:

    ```bash
    kubectl get services -n <namespaceName>
    ```
2. Восстановите базу данных в конечной точке основного экземпляра.

    Добавьте файл резервной копии базы данных в контейнер основного экземпляра.

    ```console
    kubectl cp <source file location> <pod name>:var/opt/mssql/data/<file name> -n <namespace name>
    ```

    Пример

    ```console
    kubectl cp /home/WideWorldImporters-Full.bak sql2-1:var/opt/mssql/data/WideWorldImporters-Full.bak -c arc-sqlmi -n arc
    ```

    Восстановите файл резервной копии базы данных, выполнив приведенную ниже команду.

    ```sql 
    RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/<file name>.bak'
    WITH MOVE '<database name>' to '/var/opt/mssql/data/<file name>.mdf'  
    ,MOVE '<database name>' to '/var/opt/mssql/data/<file name>_log.ldf'  
    ,RECOVERY, REPLACE, STATS = 5;  
    GO
    ```
    
    Пример

    ```sql
    RESTORE Database WideWorldImporters
    FROM DISK = '/var/opt/mssql/data/WideWorldImporters-Full.BAK'
    WITH
    MOVE 'WWI_Primary' TO '/var/opt/mssql/data/WideWorldImporters.mdf',
    MOVE 'WWI_UserData' TO '/var/opt/mssql/data/WideWorldImporters_UserData.ndf',
    MOVE 'WWI_Log' TO '/var/opt/mssql/data/WideWorldImporters.ldf',
    MOVE 'WWI_InMemory_Data_1' TO '/var/opt/mssql/data/WideWorldImporters_InMemory_Data_1',
    RECOVERY, REPLACE, STATS = 5;  
    GO
    ```

3. Добавьте базу данных в группу доступности.

    Чтобы база данных была добавлена в группу доступности, она должна быть запущена в режиме полного восстановления и необходимо было создать резервную копию журнала. Выполните приведенные ниже инструкции TSQL, чтобы добавить восстановленную базу данных в группу доступности.

    ```sql
    ALTER DATABASE <databaseName> SET RECOVERY FULL;
    BACKUP DATABASE <databaseName> TO DISK='<filePath>'
    ALTER AVAILABILITY GROUP containedag ADD DATABASE <databaseName>
    ```

    В следующем примере добавляется база данных с именем `WideWorldImporters`, которая была восстановлена в экземпляре.

    ```sql
    ALTER DATABASE WideWorldImporters SET RECOVERY FULL;
    BACKUP DATABASE WideWorldImporters TO DISK='/var/opt/mssql/data/WideWorldImporters.bak'
    ALTER AVAILABILITY GROUP containedag ADD DATABASE WideWorldImporters
    ```

> [!IMPORTANT]
> Рекомендуется выполнить очистку, удалив созданную выше службу Kubernetes с помощью следующей команды:
>
>```bash
>kubectl delete svc sql2-0-p -n arc
>```

### <a name="limitations"></a>Ограничения

Служба "Дуга Azure", включенная Управляемый экземпляр группы доступности, имеет те же [ограничения, что и группы доступности кластера больших данных. Щелкните здесь, чтобы получить дополнительные сведения.](/sql/big-data-cluster/deployment-high-availability#known-limitations)

## <a name="next-steps"></a>Следующие шаги

Подробнее о [функциях и возможностях Управляемого экземпляра SQL с поддержкой Azure Arc](managed-instance-features.md).
