---
title: включить файл
description: включить файл
services: container-service
author: mlearned
ms.service: container-service
ms.topic: include
ms.date: 11/22/2019
ms.author: mlearned
ms.custom: include file
ms.openlocfilehash: a42bba1b6524825aa571e4c18319b61b97829792
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96584603"
---
| Ресурс | Ограничение |
| --- | :--- |
| Максимальное количество кластеров на подписку | 1000 |
| Максимальное количество узлов на кластер с группами доступности виртуальных машин и номером SKU Load Balancer категории "Базовый"  | 100 |
| Максимальное количество узлов на кластер с Масштабируемыми наборами виртуальных машин и [номером SKU Load Balancer категории "Базовый"][standard-load-balancer] | 1000 (100 узлов на [пул узлов][node-pool]) |
| Максимальное количество модулей pod на узел: [Базовая организация сети][basic-networking] с использованием Kubenet | 110 |
| Максимальное количество модулей pod на узел: [Расширенные возможности работы в сети][advanced-networking] благодаря Сетевому интерфейсу контейнеров Azure | Развертывание с помощью Azure CLI: 30<sup>1</sup><br />Шаблон Azure Resource Manager: 30<sup>1</sup><br />Развертывание портала: 30 |

<sup>1</sup> При развертывании кластера Службы Azure Kubernetes (AKS) с помощью Azure CLI или шаблона Resource Manager это значение можно увеличить до 250 модулей pod на узел. Вы не сможете увеличить количество модулей pod на узел, если кластер AKS уже развернут или вы развернули его на портале Azure.<br />

<!-- LINKS - Internal -->
[basic-networking]: ../articles/aks/concepts-network.md#kubenet-basic-networking
[advanced-networking]: ../articles/aks/concepts-network.md#azure-cni-advanced-networking
[standard-load-balancer]: ../articles/load-balancer/load-balancer-overview.md
[node-pool]: ../articles/aks/use-multiple-node-pools.md

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest