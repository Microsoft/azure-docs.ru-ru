---
title: Отображение конфигурации PostgreSQL, включенной в группу серверов масштабирования.
titleSuffix: Azure Arc enabled data services
description: Отображение конфигурации PostgreSQL, включенной в группу серверов масштабирования.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: a268cd6b2fa3da6846554e3d1b170298abec7f18
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93279407"
---
# <a name="show-the-configuration-of-an-arc-enabled-postgresql-hyperscale-server-group"></a>Отображение конфигурации PostgreSQL, включенной в группу серверов масштабирования.

В этой статье объясняется, как отобразить конфигурацию групп серверов. Это достигается за счет того, что некоторые вопросы могут запрашивать себя и отвечать на них. Иногда может быть несколько допустимых ответов. В этой статье описываются наиболее распространенные или полезные условия. Он группирует эти вопросы по теме:

- с точки зрения Kubernetes
- из точки зрения служб данных с поддержкой дуги Azure

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="from-a-kubernetes-point-of-view"></a>С точки зрения Kubernetes

### <a name="how-many-pods-are-used-by-azure-arc-enabled-postgresql-hyperscale"></a>Сколько модулей, используемых в службе "Дуга Azure", PostgreSQL с помощью масштабирования?

Перечислите ресурсы Kubernetes типа postgres. Выполните приведенную ниже команду.

```console
kubectl get postgresqls [-n <namespace name>]
```

Выходные данные этой команды показывают список созданных групп серверов. Для каждого из них указывается число модулей Pod. Пример:

```output
NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      6h34m
postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d7h
```

В этом примере показано, что создаются две группы серверов, каждая из которых выполняется в 3 модулях (1 координатор и 2 рабочих роли). Это означает, что группы серверов, созданные в этом контроллере данных Arc Azure, используют 6 модулей.

### <a name="what-pods-are-used-by-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Какие модули используются в службе "Дуга Azure" с PostgreSQL масштабируемыми группами серверов?

Выполните команду:

```console
kubectl get pods [-n <namespace name>]
```

Он возвращает список модулей Pod. Вы увидите модули, используемые группами серверов, на основе имен, присвоенных этим группам серверов. Пример:

```console 
NAME                 READY   STATUS    RESTARTS   AGE
bootstrapper-vdltm   1/1     Running   0          6d8h
control-h6kc9        2/2     Running   0          6d8h
controldb-0          2/2     Running   0          6d8h
controlwd-96sbn      1/1     Running   0          6d8h
logsdb-0             1/1     Running   0          6d8h
logsui-7wkg2         1/1     Running   0          6d8h
metricsdb-0          1/1     Running   0          6d8h
metricsdc-28ffl      1/1     Running   0          6d8h
metricsui-k7qsh      1/1     Running   0          6d8h
mgmtproxy-gd84z      2/2     Running   0          6d8h
postgres01-0         3/3     Running   0          6h50m
postgres01-1         3/3     Running   0          6h50m
postgres01-2         3/3     Running   0          6h50m
postgres02-0         3/3     Running   0          22h
postgres02-1         3/3     Running   0          22h
postgres02-2         3/3     Running   0          22h
```

В этом примере шесть модулей Pod, в которых размещаются две создаваемые группы серверов,:
- `postgres01-0`
- `postgres01-1`
- `postgres01-2`
- `postgres02-0`
- `postgres02-1`
- `postgres02-2`  

### <a name="what-server-group-pod-is-used-for-what-role-the-server-group"></a>Какой модуль группы серверов используется для роли группы серверов?

Любое имя Pod с суффиксом `-0` представляет собой узел координатора. Любое имя узла с суффиксом, `-x` которое равно 1 или больше, является рабочим узлом. В приведенном выше примере:
- Используются следующие координаторы: `postgres01-0` , `postgres02-0`
- Рабочие роли: `postgres01-2` , `postgres01-2` , `postgres02-1` , `postgres02-2`

### <a name="what-is-the-status-of-the-pods"></a>Каково состояние модулей Pod?

Запустите `kubectl get pods` и взгляните на столбец `STATUS`

### <a name="what-persistent-volume-claims-pvcs-are-being-used"></a>Какие требования к постоянному тому используются (PVC)? 

Чтобы понять, какие виртуальные каналы используются, а также какие данные используются для данных, журналов и резервных копий, выполните: 

```console
kubectl get pvc [-n <namespace name>]
```

По умолчанию префикс для имени PVC указывает на его использование:

- `backups-`... — это PVC, используемый для резервного копирования.
- `data-`... — это PVC, используемый для файлов данных.
- `logs-`... — это PVC, используемый для файлов журналов транзакций и WAL.

Пример:

```output
NAME                                            STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS    AGE
backups-few7hh0k4npx9phsiobdc3hq-postgres01-0   Bound    local-pv-485e37db   1938Gi     RWO            local-storage   6d6h
backups-few7hh0k4npx9phsiobdc3hq-postgres01-1   Bound    local-pv-9d3d4a15   1938Gi     RWO            local-storage   6d6h
backups-few7hh0k4npx9phsiobdc3hq-postgres01-2   Bound    local-pv-7b8dd819   1938Gi     RWO            local-storage   6d6h
...
data-few7hh0k4npx9phsiobdc3hq-postgres01-0      Bound    local-pv-3c1a8cc5   1938Gi     RWO            local-storage   6d6h
data-few7hh0k4npx9phsiobdc3hq-postgres01-1      Bound    local-pv-8303ab19   1938Gi     RWO            local-storage   6d6h
data-few7hh0k4npx9phsiobdc3hq-postgres01-2      Bound    local-pv-55572fe6   1938Gi     RWO            local-storage   6d6h
...
logs-few7hh0k4npx9phsiobdc3hq-postgres01-0      Bound    local-pv-5e852b76   1938Gi     RWO            local-storage   6d6h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-1      Bound    local-pv-55d309a7   1938Gi     RWO            local-storage   6d6h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-2      Bound    local-pv-5ccd02e6   1938Gi     RWO            local-storage   6d6h
...
```


## <a name="from-an-azure-arc-enabled-data-services-point-of-view"></a>С точки зрения служб данных с поддержкой дуги Azure:

* Сколько групп серверов создается в контроллере данных Arc?
* Что такое имена?
* Сколько рабочих узлов они используют?
* Какую версию postgres запускать?

Используйте одну из следующих команд.
- **С kubectl:**

   ```console
   kubectl get postgresqls [-n <namespace name>]
   ``` 

   Например, он создает приведенный ниже результат, где каждая строка является `servergroup` . Структура имени в приведенном ниже виде имеет следующий вид:

   `<Name-Of-Custom-Resource-Definition>`/`<Server-Group-Name>`

   ```output
   NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
   postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      7h15m
   postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d7h
   ```

   В выходных данных выше показаны две группы серверов postgres версии 12. Если версия была postgres 11, вместо нее будет указано имя CRD postgresql-11.arcdata.microsoft.com.

   Каждый из них выполняется на трех узлах и в модулях Pod: 1 координатор и 2 работника.

- **С аздата:**

Выполните следующую команду. В выходных данных отображаются похожие сведения о том, что kubectl показывает:

   ```console
   azdata arc postgres server list

   `output
   Name        State    Workers
   ----------  -------  ---------
   postgres01  Ready    2
   postgres02  Ready    2
   ```


### <a name="how-much-memory-and-vcores-are-being-used-and-what-extensions-were-created-for-a-group"></a>Сколько памяти и виртуальных ядер используются и какие расширения были созданы для группы?

Выполните одну из следующих команд.

**С Kubectl:**

Используйте kubectl для описания ресурсов postgres. Для этого необходимо иметь свой тип (имя ресурса Kubernetes (CRD) для соответствующей версии postgres, как показано выше) и имя группы серверов.
Общий формат этой команды:

```console
kubectl describe <CRD name>/<server group name> [-n <namespace name>]
```

Пример:

```console
kubectl describe postgresql-12/postgres02
```

Эта команда отображает конфигурацию группы серверов.

```output
Name:         postgres02
Namespace:    arc
Labels:       <none>
Annotations:  <none>
API Version:  arcdata.microsoft.com/v1alpha1
Kind:         postgresql-12
Metadata:
  Creation Timestamp:  2020-08-31T21:01:07Z
  Generation:          10
  Resource Version:    569516
  Self Link:           /apis/arcdata.microsoft.com/v1alpha1/namespaces/arc/postgresql-12s/postgres02
  UID:                 8a9cd118-361b-4a2e-8a9d-5f9257bf6abb
Spec:
  Engine:
    Extensions:
      Name:  citus
      Name:  pg_stat_statements
  Scale:
    Workers:  2
  Scheduling:
    Default:
      Resources:
        Limits:
          Cpu:     4
          Memory:  1024Mi
        Requests:
          Cpu:     1
          Memory:  512Mi
  Service:
    Type:  NodePort
  Storage:
    Data:
      Class Name:  local-storage
      Size:        5Gi
    Logs:
      Class Name:  local-storage
      Size:        5Gi
Status:
  External Endpoint:  10.0.0.4:31066
  Ready Pods:         3/3
  State:              Ready
Events:               <none>
```

>[!NOTE]
>До выпуска 2020 октября `Workers` существовали `Shards` в предыдущем примере. Дополнительные сведения см. в разделе [заметки о выпуске Azure Arc Data Services (Предварительная версия)](release-notes.md) .

Давайте выберем некоторые конкретные интересующие моменты в описании, приведенном `servergroup` выше. Что сообщает нам об этой группе серверов?

- Версия postgres версии 12. 
   > ```json
   > Kind:         `postgresql-12`
   > ```
- Он был создан в течение месяца 2020 августа:
   > ```json
   > Creation Timestamp:  `2020-08-31T21:01:07Z`
   > ```
- В этой группе серверов создано два расширения postgres: `citus` и `pg_stat_statements`
   > ```json
   > Engine:
   >    Extensions:
   >      Name:  `citus`
   >      Name:  `pg_stat_statements`
   > ```
- В ней используются два рабочих узла
   > ```json
   > Scale:
   >    Workers:  `2`
   > ```
- Для каждого узла гарантируется использование 1 ЦП/Виртуальное ядро и 512 МБ ОЗУ. Он будет использовать более 4 ЦП/виртуальных ядер и 1024 МБ памяти:
   > ```json
   > Scheduling:
   >    Default: 
   >      Resources:
   >        Limits:
   >          Cpu:     4
   >          Memory:  1024Mi
   >        Requests:
   >          Cpu:     1
   >          Memory:  512Mi
   > ```
 - Она доступна для запросов и не имеет проблем. Все узлы работают и выполняются:
   > ```json
   > Status:
   >  ...
   >  Ready Pods:         3/3
   >  State:              Ready
   > ```

**С аздата:**

Общий формат команды:

```console
azdata arc postgres server show -n <server group name>
```

Пример:

```console
azdata arc postgres server show -n postgres02
```

Возвращает приведенный ниже результат в формате и содержимом, очень похожем на тот, который возвращается функцией kubectl.

```console
{
  "apiVersion": "arcdata.microsoft.com/v1alpha1",
  "kind": "postgresql-12",
  "metadata": {
    "creationTimestamp": "2020-08-31T21:01:07Z",
    "generation": 10,
    "name": "postgres02",
    "namespace": "arc",
    "resourceVersion": "569516",
    "selfLink": "/apis/arcdata.microsoft.com/v1alpha1/namespaces/arc/postgresql-12s/postgres02",
    "uid": "8a9cd118-361b-4a2e-8a9d-5f9257bf6abb"
  },
  "spec": {
    "engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_stat_statements"
        }
      ]
    },
    "scale": {
      "workers": 2
    },
    "scheduling": {
      "default": {
        "resources": {
          "limits": {
            "cpu": "4",
            "memory": "1024Mi"
          },
          "requests": {
            "cpu": "1",
            "memory": "512Mi"
          }
        }
      }
    },
    "service": {
      "type": "NodePort"
    },
    "storage": {
      "data": {
        "className": "local-storage",
        "size": "5Gi"
      },
      "logs": {
        "className": "local-storage",
        "size": "5Gi"
      }
    }
  },
  "status": {
    "externalEndpoint": "10.0.0.4:31066",
    "readyPods": "3/3",
    "state": "Ready"
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия
- [Узнайте о концепциях PostgreSQL с поддержкой ARC в Azure.](concepts-distributed-postgres-hyperscale.md)
- [Дополнительные сведения о масштабировании (Добавление рабочих узлов) в группу серверов](scale-out-postgresql-hyperscale-server-group.md)
- [Узнайте, как масштабировать группу серверов (увеличить или уменьшить объем памяти и/или виртуальных ядер).](scale-up-down-postgresql-hyperscale-server-group-using-cli.md)
- [Сведения о конфигурации хранилища](storage-configuration.md)
- [Сведения о мониторинге экземпляра базы данных](monitor-grafana-kibana.md)
- [Использование расширений PostgreSQL в службе "Дуга Azure", в которой включена PostgreSQLная группа серверов](using-extensions-in-postgresql-hyperscale-server-group.md)
- [Настройка безопасности для группы серверов Гипермасштабирования PostgreSQL с поддержкой Azure Arc](configure-security-postgres-hyperscale.md)
