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
ms.openlocfilehash: 1c2dec106ae72ddead7bda54792fa74e38eb6660
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106081054"
---
| Ресурс                                                                                                           | Ограничение                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Максимальное количество кластеров на подписку                                                                                  | 1000                                                                                                                                                                                                        |
| Максимальное количество узлов на кластер с группами доступности виртуальных машин и номером SKU Load Balancer категории "Базовый"                       | 100                                                                                                                                                                                                         |
| Максимальное количество узлов на кластер с Масштабируемыми наборами виртуальных машин и [номером SKU Load Balancer категории "Базовый"][standard-load-balancer] | 1000 (100 узлов на [пул узлов][node-pool])                                                                                                                                                                 |
| Максимальное количество модулей pod на узел: [Базовая организация сети][basic-networking] с использованием Kubenet                                           | 110                                                                                                                                                                                                         |
| Максимальное количество модулей pod на узел: [Расширенные возможности работы в сети][advanced-networking] благодаря Сетевому интерфейсу контейнеров Azure        | Развертывание с помощью Azure CLI: 30<sup>1</sup><br />Шаблон Azure Resource Manager: 30<sup>1</sup><br />Развертывание портала: 30                                                                                        |
| Предварительная версия надстройки Open Service Mesh (OSM) AKS                                                                          | Версия кластера Kubernetes: 1.19+<sup>2</sup><br />Контроллеров OSM в кластере: 1<sup>2</sup><br />Объектов Pod в контроллере OSM: 500<sup>2</sup><br />Учетные записи службы Kubernetes, управляемые посредством OSM: 50<sup>2</sup> |

<sup>1</sup> При развертывании кластера Службы Azure Kubernetes (AKS) с помощью Azure CLI или шаблона Resource Manager это значение можно увеличить до 250 модулей pod на узел. Вы не сможете увеличить количество модулей pod на узел, если кластер AKS уже развернут или вы развернули его на портале Azure.<br />

<sup>2</sup> Надстройка OSM для AKS находится в состоянии предварительной версии и будет подвергнута дополнительным усовершенствованиям до перевода в состояние общедоступной версии. На этапе предварительной версии рекомендуется не превышать указанные предельные значения.<br />

<!-- LINKS - Internal -->

[basic-networking]: ../articles/aks/concepts-network.md#kubenet-basic-networking
[advanced-networking]: ../articles/aks/concepts-network.md#azure-cni-advanced-networking
[standard-load-balancer]: ../articles/load-balancer/load-balancer-overview.md
[node-pool]: ../articles/aks/use-multiple-node-pools.md

<!-- LINKS - External -->

[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
