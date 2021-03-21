---
title: Настройка мониторинга GPU с помощью Container Insights | Документация Майкрософт
description: В этой статье описывается, как настроить мониторинг кластеров Kubernetes с помощью узлов NVIDIA и AMD с поддержкой GPU с помощью аналитики контейнеров.
ms.topic: conceptual
ms.date: 03/27/2020
ms.openlocfilehash: 2958b000ac0dabcd7fddf75a58f553b705a95e9a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101731873"
---
# <a name="configure-gpu-monitoring-with-container-insights"></a>Настройка мониторинга GPU с помощью Container Insights

Начиная с версии агента *ciprod03022019*, интегрированный агент аналитики контейнеров теперь поддерживает мониторинг использования GPU (графических единиц обработки) на узлах кластера Kubernetes, поддерживающих GPU, и отслеживание блоков и контейнеров, запрашивающих и использующих ресурсы GPU.

## <a name="supported-gpu-vendors"></a>Поддерживаемые поставщики GPU

Аналитика контейнеров поддерживает мониторинг кластеров GPU от следующих поставщиков GPU:

- [ПЛАТ](https://developer.nvidia.com/kubernetes-gpu)

- [AMD](https://github.com/RadeonOpenCompute/k8s-device-plugin)

Аналитика контейнеров автоматически начинает отслеживать использование GPU на узлах и GPU, запрашивающих модули Pod и рабочие нагрузки, собирая следующие метрики через 60sec интервалы и сохраняя их в таблице **инсигхтметрикс** .

>[!NOTE]
>После подготовки кластера с узлами GPU убедитесь, что [драйвер GPU](../../aks/gpu-cluster.md) установлен в соответствии с требованиями AKS для запуска рабочих нагрузок GPU. Служба "аналитика контейнеров" соберет метрики GPU с помощью модулей драйвера GPU, выполняющихся на узле. 

|Имя метрики |Измерение метрики (Теги) |Описание |
|------------|------------------------|------------|
|контаинергпудутицикле |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName, gpuId, Гпумодел, Гпувендор|Процент времени за последний период выборки (60 секунд), в течение которого GPU был занят или активно обработан для контейнера. Рабочий цикл — это число от 1 до 100. |
|контаинергпулимитс |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName |Каждый контейнер может задавать ограничения как один или несколько GPU. Невозможно запросить или ограничить долю GPU. |
|контаинергпурекуестс |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName |Каждый контейнер может запрашивать один или несколько GPU. Невозможно запросить или ограничить долю GPU.|
|контаинергпумеморитоталбитес |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName, gpuId, Гпумодел, Гпувендор |Объем памяти GPU в байтах, доступной для использования в определенном контейнере. |
|контаинергпумеморюседбитес |container.azm.ms/clusterId, container.azm.ms/clusterName, containerName, gpuId, Гпумодел, Гпувендор |Объем памяти GPU в байтах, используемый указанным контейнером. |
|нодегпуаллокатабле |container.azm.ms/clusterId, container.azm.ms/clusterName, Гпувендор |Количество GPU в узле, которые могут использоваться Kubernetes. |
|нодегпукапаЦити |container.azm.ms/clusterId, container.azm.ms/clusterName, Гпувендор |Общее число GPU в узле. |

## <a name="gpu-performance-charts"></a>Диаграммы производительности GPU 

Контейнерная аналитика включает предварительно настроенные диаграммы для метрик, перечисленных выше в таблице в виде книги GPU для каждого кластера. Описание книг, доступных для аналитики контейнеров, см. [в книге в контейнере аналитики](../insights/container-insights-reports.md) .

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о развертывании кластера AKS, включающего узлы с поддержкой GPU, см. в статье [Использование GPU для рабочих нагрузок с большим количеством вычислений в службе Azure Kubernetes Service](../../aks/gpu-cluster.md) (AKS).

- Дополнительные сведения о номерах [SKU виртуальных машин, оптимизированных для GPU](../../virtual-machines/sizes-gpu.md), см. в Microsoft Azure.

- Ознакомьтесь с [поддержкой GPU в Kubernetes](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/) , чтобы узнать больше о Kubernetes экспериментальной поддержке управления GPU на одном или нескольких узлах в кластере.
