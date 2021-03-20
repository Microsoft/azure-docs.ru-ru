---
title: Высокий уровень доступности для контейнеров Azure SQL с границами — Azure SQL
description: Дополнительные сведения о высокой доступности для контейнеров Azure SQL
keywords: SQL ребро, контейнеры, высокий уровень доступности
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 56b9f06547f737bc05d573f98ce1dbac2ba48758
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90937625"
---
# <a name="high-availability-for-azure-sql-edge-containers"></a>Высокий уровень доступности для контейнеров Azure SQL

Создавайте собственные экземпляры Azure SQL и управляйте ими в Kubernetes. Разверните Azure SQL ребро в контейнерах DOCKER, управляемых с помощью [Kubernetes](https://kubernetes.io/). В Kubernetes контейнер с экземпляром Azure SQL погранично может автоматически восстанавливаться в случае сбоя узла кластера. Вы можете настроить образ контейнера SQL с помощью утверждения Kubernetes постоянного тома (PVC). Kubernetes отслеживает пограничные процессы Azure SQL в контейнере. В случае сбоя процесса, pod, контейнера или узла Kubernetes автоматически загружает другой экземпляр и восстанавливает подключение к хранилищу.

## <a name="azure-sql-edge-containers-on-kubernetes"></a>Контейнеры Azure SQL в Kubernetes

Kubernetes 1,6 и более поздней версии поддерживают [*классы хранения*](https://kubernetes.io/docs/concepts/storage/storage-classes/), [*утверждения Постоянного тома*](https://kubernetes.io/docs/concepts/storage/storage-classes/#persistentvolumeclaims).

В этой конфигурации Kubernetes играет роль оркестратора контейнера. 

![Схема Azure SQL ребра в кластере Kubernetes](media/deploy-kubernetes/kubernetes-sql-edge.png)

На предыдущей схеме `azure-sql-edge` — это контейнер внутри модуля [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/). Kubernetes управляет ресурсами в кластере. Применение [набора реплик](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) гарантирует автоматическое восстановление pod в случае отказа узла. Приложения подключаются к службе. В этом случае служба представляет подсистему балансировки нагрузки, которая содержит IP-адрес, остающийся неизменным в случае отказа `azure-sql-edge`.

На следующей схеме в контейнере `azure-sql-edge` происходит сбой. Как оркестратор Kubernetes гарантирует правильное количество работоспособных экземпляров в наборе реплик и запускает новый контейнер в соответствии с конфигурацией. Оркестратор запускает новый модуль Pod на том же узле, и `azure-sql-edge` повторно подключается к тому же постоянному хранилищу. Служба подключается к повторно созданному `azure-sql-edge`.

![SQL Azure для пограничных вычислений в кластере Kubernetes после сбоя модуля pod](media/deploy-kubernetes/kubernetes-sql-edge-after-pod-fail.png)

На следующей схеме сбой затронул узел, на котором размещен контейнер `azure-sql-edge`. Оркестратор запускает новый модуль Pod на другом узле, и `azure-sql-edge` повторно подключается к тому же постоянному хранилищу. Служба подключается к повторно созданному `azure-sql-edge`.

![SQL Azure для пограничных вычислений в кластере Kubernetes после сбоя узла](media/deploy-kubernetes/kubernetes-sql-edge-after-node-fail.png).

Сведения о создании контейнера в Kubernetes см. [в статье Развертывание контейнера ребра Azure SQL в Kubernetes](deploy-Kubernetes.md)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы развернуть контейнеры Azure SQL в службе Azure Kubernetes (AKS), см. следующие статьи:
- [Развертывание контейнера Azure SQL в Kubernetes](deploy-Kubernetes.md)
- [Машинное обучение и искусственный интеллект с использованием ONNX в SQL для пограничных вычислений](onnx-overview.md)
- [Создание комплексного решения для Интернета вещей с использованием SQL для пограничных вычислений на основе IoT Edge](tutorial-deploy-azure-resources.md).
- [Потоковая передача данных в SQL Azure для пограничных вычислений](stream-data.md)