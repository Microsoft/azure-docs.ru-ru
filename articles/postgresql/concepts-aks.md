---
title: Подключение к службе Azure Kubernetes. база данных Azure для PostgreSQL — один сервер
description: Дополнительные сведения о подключении службы Kubernetes Azure (AKS) к базе данных Azure для PostgreSQL-Single Server
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.date: 07/14/2020
ms.topic: conceptual
ms.openlocfilehash: 9b7da2fcc1310f03f894e048089658f25be3a149
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91708855"
---
# <a name="connecting-azure-kubernetes-service-and-azure-database-for-postgresql---single-server"></a>Подключение службы Kubernetes Azure и базы данных Azure для PostgreSQL-Single Server

Служба Azure Kubernetes (AKS) предоставляет управляемый кластер Kubernetes для использования в Azure. Ниже приведены некоторые настройки, которые можно использовать при создании приложения с помощью AKS и Базы данных Azure для PostgreSQL.


## <a name="accelerated-networking"></a>Ускорение работы в сети
Используйте в кластере AKS базовые виртуальные машины с поддержкой ускоренной работы в сети. Если на виртуальной машине включена ускоренная работа в сети, на машине наблюдается снижение задержки, дрожания и загрузки ЦП. Дополнительные сведения об ускоренной работе в сети, поддерживаемых версиях ОС и экземплярах виртуальных машин Linux см. в статье [Создание виртуальной машины Linux с ускоренной сетью](../virtual-network/create-vm-accelerated-networking-cli.md).

С ноября 2018 года AKS поддерживает ускоренные сети на поддерживаемых экземплярах виртуальных машин. Для новых кластеров AKS, которые используют эти виртуальные машины, ускоренная работа в сети включена по умолчанию.

Чтобы проверить, использует ли ваш кластер AKS ускоренную сеть, сделайте следующее:
1. Перейдите на портал Azure и выберите свой кластер AKS.
2. Выберите вкладку "Свойства".
3. Скопируйте имя **группы ресурсов инфраструктуры**.
4. С помощью панели поиска на портале найдите и откройте группу ресурсов инфраструктуры.
5. Выберите в этой группе ресурсов виртуальную машину.
6. Перейдите на вкладку **Сети** для виртуальной машины.
7. Проверьте, включена ли **ускоренная сеть**.

Или с помощью Azure CLI выполните следующие две команды.
```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query "nodeResourceGroup"
```
Будет получена группа ресурсов, созданная службой AKS и содержащая сетевой интерфейс. Скопируйте имя nodeResourceGroup и используйте его в следующей команде. Значением **EnableAcceleratedNetworking** будет true или false.
```azurecli
az network nic list --resource-group nodeResourceGroup -o table
```


## <a name="connection-pooling"></a>Организация пулов соединений
Пул подключений позволяет свести к минимуму затраты и время, связанные с созданием и закрытием новых подключений к базе данных. Пул — это коллекция подключений, которые можно использовать повторно. 

С PostgreSQL можно использовать несколько пулов подключений. Один из них — [PgBouncer](https://pgbouncer.github.io/). В Реестре контейнеров Майкрософт доступен упрощенный контейнерный пул PgBouncer, который можно использовать в расширении для объединения подключений AKS к Базе данных Azure для PostgreSQL. Перейдите на [страницу центра Docker](https://hub.docker.com/r/microsoft/azureossdb-tools-pgbouncer/), чтобы просмотреть сведения о получении доступа к этому образу и его использовании. 


## <a name="next-steps"></a>Дальнейшие действия
-  [Создание кластера службы Azure Kubernetes](../aks/kubernetes-walkthrough.md)
