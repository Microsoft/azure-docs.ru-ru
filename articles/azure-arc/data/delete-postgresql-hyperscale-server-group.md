---
title: Удаление группы горизонтального масштабирования сервера PostgreSQL в службе "Дуга Azure"
description: Удаление группы горизонтального масштабирования сервера postgres в службе "Дуга Azure"
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 7932ad3b30910e539acfbff2329a03f80a4d1a0b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104670364"
---
# <a name="delete-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Удаление группы горизонтального масштабирования сервера PostgreSQL в службе "Дуга Azure"

В этом документе описаны действия по удалению группы серверов из настройки дуги Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="delete-the-server-group"></a>Удаление группы серверов

Например, рассмотрим, что мы хотим удалить экземпляр _postgres01_ из следующей установки:

```console
azdata arc postgres server list
Name        State    Workers
----------  -------  ---------
postgres01  Ready    3
```

Общий формат команды Delete:
```console
azdata arc postgres server delete -n <server group name>
```
При выполнении этой команды будет запрошено подтверждение удаления группы серверов. При использовании скриптов для автоматизации удаления необходимо использовать параметр--Force, чтобы обойти запрос на подтверждение. Например, можно выполнить команду следующим образом: 
```console
azdata arc postgres server delete -n <server group name> --force
```

Чтобы получить дополнительные сведения о команде delete, выполните команду:
```console
azdata arc postgres server delete --help
```

### <a name="delete-the-server-group-used-in-this-example"></a>Удаление группы серверов, используемой в этом примере

```console
azdata arc postgres server delete -n postgres01
```

## <a name="reclaim-the-kubernetes-persistent-volume-claims-pvcs"></a>Освобождение утверждений о постоянном томе Kubernetes (PVC)

При удалении группы серверов связанные с ней [постоянные виртуальные каналы](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)не удаляются. Это сделано намеренно. Это необходимо, чтобы помочь пользователю получить доступ к файлам базы данных в случае случайного удаления экземпляра. Удалять утверждения PVC не обязательно, но рекомендуется. Если вы не освободите эти утверждения PVC, в итоге будут происходить ошибки, так как кластер Kubernetes считает, что на нем заканчивается дисковое пространство. Для освобождения утверждений PVC сделайте следующее.

### <a name="1-list-the-pvcs-for-the-server-group-you-deleted"></a>1. Выведите список постоянных виртуальных цепей для удаленной группы серверов.

Чтобы получить список постоянных виртуальных цепей, выполните следующую команду:

```console
kubectl get pvc [-n <namespace name>]
```

Он возвращает список постоянных виртуальных цепей, в частности, PVC для удаленной группы серверов. Пример:

```output
kubectl get pvc
NAME                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-few7hh0k4npx9phsiobdc3hq-postgres01-0   Bound    pvc-72ccc225-dad0-4dee-8eae-ed352be847aa   5Gi        RWO            default        2d18h
data-few7hh0k4npx9phsiobdc3hq-postgres01-1   Bound    pvc-ce6f0c51-faed-45ae-9472-8cdf390deb0d   5Gi        RWO            default        2d18h
data-few7hh0k4npx9phsiobdc3hq-postgres01-2   Bound    pvc-5a863ab9-522a-45f3-889b-8084c48c32f8   5Gi        RWO            default        2d18h
data-few7hh0k4npx9phsiobdc3hq-postgres01-3   Bound    pvc-00e1ace3-1452-434f-8445-767ec39c23f2   5Gi        RWO            default        2d15h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-0   Bound    pvc-8b810f4c-d72a-474a-a5d7-64ec26fa32de   5Gi        RWO            default        2d18h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-1   Bound    pvc-51d1e91b-08a9-4b6b-858d-38e8e06e60f9   5Gi        RWO            default        2d18h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-2   Bound    pvc-8e5ad55e-300d-4353-92d8-2e383b3fe96e   5Gi        RWO            default        2d18h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-3   Bound    pvc-f9e4cb98-c943-45b0-aa07-dd5cff7ea585   5Gi        RWO            default        2d15h
```
Для этой группы серверов имеется 8 постоянных виртуальных цепей.

### <a name="2-delete-each-of-the-pvcs"></a>2. Удалите все виртуальные каналы

Удалите данные и записи PVC для каждого из PostgreSQLных узлов (координаторов и рабочих ролей) в группе серверов, которую вы удалили.

Общий формат этой команды: 

```console
kubectl delete pvc <name of pvc>  [-n <namespace name>]
```

Пример:

```console
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-0
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-1
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-2
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-3
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-0
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-1
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-2
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-3
```

Каждая из этих команд kubectl будет подтверждать успешное удаление PVC. Пример:

```output
persistentvolumeclaim "data-postgres01-0" deleted
```
  

>[!NOTE]
> Как указано, удаление постоянных виртуальных цепей может в конечном итоге получить кластер Kubernetes в ситуации, когда она выдаст ошибки. Некоторые из этих ошибок могут включать в себя невозможность входа в кластер Kubernetes с помощью аздата, так как эти модули могут быть исключены из-за этой проблемы с хранилищем (нормальное поведение Kubernetes).
>
> Например, в журналах могут отображаться сообщения следующего вида:  
> ```output
> Annotations:    microsoft.com/ignore-pod-health: true  
> Status:         Failed  
> Reason:         Evicted  
> Message:        The node was low on resource: ephemeral-storage. Container controller was using 16372Ki, which exceeds its request of 0.
> ```
    
## <a name="next-step"></a>Следующий шаг
Создание [PostgreSQL в Azure ARC с включенным масштабированием](create-postgresql-hyperscale-server-group.md)
