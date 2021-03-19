---
title: Удаление SQL Управляемый экземпляр с поддержкой дуги Azure
description: Удаление SQL Управляемый экземпляр с поддержкой дуги Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: e531349e8f404380d9f0601caa3b66557c297062
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90940847"
---
# <a name="delete-azure-arc-enabled-sql-managed-instance"></a>Удаление SQL Управляемый экземпляр с поддержкой дуги Azure
В этой статье объясняется, как можно удалить SQL Управляемый экземпляр с поддержкой службы "Дуга Azure".

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="view-existing-azure-arc-enabled-sql-managed-instances"></a>Просмотр существующих управляемых экземпляров SQL с поддержкой ARC в Azure
Чтобы просмотреть управляемые экземпляры SQL, выполните следующую команду:

```console
azdata arc sql mi list
```

Результат должен выглядеть примерно так:

```console
Name    Replicas    ServerEndpoint    State
------  ----------  ----------------  -------
demo-mi 1/1         10.240.0.4:32023  Ready
```

## <a name="delete-a-azure-arc-enabled-sql-managed-instance"></a>Удаление SQL Управляемый экземпляр с поддержкой дуги Azure
Чтобы удалить Управляемый экземпляр SQL, выполните следующую команду:

```console
azdata arc sql mi delete -n <NAME_OF_INSTANCE>
```

Результат должен выглядеть примерно так:

```console
# azdata arc sql mi delete -n demo-mi
Deleted demo-mi from namespace arc
```

## <a name="reclaim-the-kubernetes-persistent-volume-claims-pvcs"></a>Освобождение утверждений о постоянном томе Kubernetes (PVC)

При удалении Управляемый экземпляр SQL связанные с ней [постоянные виртуальные каналы](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)не удаляются. Это сделано намеренно. Это необходимо, чтобы помочь пользователю получить доступ к файлам базы данных в случае случайного удаления экземпляра. Удалять утверждения PVC не обязательно, но рекомендуется. Если вы не освободите эти виртуальные каналы, в итоге будут возникнет ошибка, так как кластеру Kubernetes не хватает места на диске. Для освобождения утверждений PVC сделайте следующее.

### <a name="1-list-the-pvcs-for-the-server-group-you-deleted"></a>1. Выведите список постоянных виртуальных цепей для удаленной группы серверов.
Чтобы получить список постоянных виртуальных цепей, выполните следующую команду:
```console
kubectl get pvc
```

В приведенном ниже примере обратите внимание на постоянные виртуальные каналы для удаленных управляемых экземпляров SQL.
```console
# kubectl get pvc -n arc

NAME                    STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-demo-mi-0        Bound     pvc-1030df34-4b0d-4148-8986-4e4c20660cc4   5Gi        RWO            managed-premium   13h
logs-demo-mi-0        Bound     pvc-11836e5e-63e5-4620-a6ba-d74f7a916db4   5Gi        RWO            managed-premium   13h
```

### <a name="2-delete-each-of-the-pvcs"></a>2. Удалите все виртуальные каналы
Удалите данные и журналы PVC для каждого удаленного управляемого экземпляра SQL.
Общий формат этой команды: 
```console
kubectl delete pvc <name of pvc>
```

Пример:
```console
kubectl delete pvc data-demo-mi-0 -n arc
kubectl delete pvc logs-demo-mi-0 -n arc
```

Каждая из этих команд kubectl будет подтверждать успешное удаление PVC. Пример:
```console
persistentvolumeclaim "data-demo-mi-0" deleted
persistentvolumeclaim "logs-demo-mi-0" deleted
```
  

> [!NOTE]
> Как указано, удаление постоянных виртуальных цепей может в конечном итоге получить кластер Kubernetes в ситуации, когда она выдаст ошибки. Некоторые из этих ошибок могут включать в себя невозможность входа в кластер Kubernetes с помощью аздата, так как эти модули могут быть исключены из-за этой проблемы с хранилищем (нормальное поведение Kubernetes).
>
> Например, в журналах могут отображаться сообщения следующего вида:  
> - Аннотации: microsoft.com/ignore-pod-health: true  
> - Состояние: сбой  
> - Причина: удалено  
> - Сообщение: нехватка узла на ресурсе: эфемерное хранилище. Контроллер контейнера использует 16372Ki, что превышает его запрос 0.

## <a name="next-steps"></a>Следующие шаги

Подробнее о [функциях и возможностях Управляемого экземпляра SQL с поддержкой Azure Arc](managed-instance-features.md).

[Начните работу, создав контроллер данных.](create-data-controller.md)

Уже создали контроллер данных? [Создайте Управляемый экземпляр SQL с поддержкой Azure Arc](create-sql-managed-instance.md)