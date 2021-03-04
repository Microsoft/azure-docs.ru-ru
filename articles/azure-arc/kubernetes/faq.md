---
title: Часто задаваемые вопросы о службе Arc Azure Kubernetes
services: azure-arc
ms.service: azure-arc
ms.date: 02/19/2021
ms.topic: conceptual
author: shashankbarsin
ms.author: shasb
description: Эта статья содержит список часто задаваемых вопросов, связанных с включенной службой Arc Kubernetes.
keywords: Kubernetes, Arc, Azure, контейнеры, конфигурация, Гитопс, вопросы и ответы
ms.openlocfilehash: d7facb842509bdf852957d46d91dfcb3912e99ba
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102121717"
---
# <a name="frequently-asked-questions---azure-arc-enabled-kubernetes"></a>Часто задаваемые вопросы об Azure Arc Enabled Kubernetes

В этой статье рассматриваются часто задаваемые вопросы о Kubernetes с поддержкой Arc Azure.

## <a name="what-is-the-difference-between-azure-arc-enabled-kubernetes-and-azure-kubernetes-service-aks"></a>В чем разница между Kubernetes с поддержкой Azure Arc и Службой Azure Kubernetes (AKS)?

AKS — это управляемое предложение Kubernetes в Azure. AKS упрощает развертывание управляемого кластера Kubernetes в Azure путем разгрузки значительной сложности и эксплуатационных расходов в Azure. Так как главными узлами Kubernetes управляет платформа Azure, вы администрируете и обслуживаете только узлы агентов.

Служба "Дуга Azure" с поддержкой Kubernetes позволяет расширить возможности управления Azure (например, Azure Monitor и политику Azure), подключив кластеры Kubernetes к Azure. Вы сохраняете базовый кластер Kubernetes.

## <a name="do-i-need-to-connect-my-aks-clusters-running-on-azure-to-azure-arc"></a>Нужно ли подключать кластеры AKS, работающие в Azure, к службе "Дуга" Azure?

Нет. Все функции Kubernetes, включенные в службу "Дуга Azure", включая Azure Monitor и политику Azure (привратник), доступны в AKS (собственный ресурс в Azure Resource Manager).
    
## <a name="should-i-connect-my-aks-hci-cluster-and-kubernetes-clusters-on-azure-stack-hub-and-azure-stack-edge-to-azure-arc"></a>Следует ли подключить кластер AKS-ХЦИ и кластеры Kubernetes в центре Azure Stack, а Azure Stack ребра к службе "Дуга Azure"?

Да, подключение кластера AKS-ХЦИ или кластеров Kubernetes на Azure Stack пограничном или Azure Stackном концентраторе к Azure Arc обеспечивает кластеры с представлением ресурсов в Azure Resource Manager. Это представление ресурсов расширяет такие возможности, как конфигурация кластера, Azure Monitor и политика Azure (привратник) для подключенных кластеров Kubernetes.

Если кластер Kubernetes, включенный в дугу Azure, находится на Azure Stack пограничном, AKS на Azure Stack ХЦИ (>= апрель 2021 обновление) или AKS в Windows Server 2019 Datacenter (>= апрель 2021 обновление), то конфигурация Kubernetes будет включена бесплатно.

## <a name="how-to-address-expired-azure-arc-enabled-kubernetes-resources"></a>Как устранить срок действия ресурсов Kubernetes в службе "Дуга Azure" с истекшим периодом?

В сертификате Управляемое удостоверение службы (MSI), связанном с включенной службой "Дуга Azure" Kubernetes, срок действия составляет 90 дней. После истечения срока действия этого сертификата считается, что ресурс рассматривается `Expired` и все компоненты (такие как конфигурация, мониторинг и политика) перестают работать в этом кластере. Чтобы снова получить кластер Kubernetes с помощью дуги Azure, сделайте следующее:

1. Удалите в кластере ресурс и агенты Kubernetes с поддержкой дуги Azure. 

    ```console
    az connectedk8s delete -n <name> -g <resource-group>
    ```

1. Повторно создайте ресурс Kubernetes с включенной службой "Дуга Azure", развернув агенты в кластере.
    
    ```console
    az connectedk8s connect -n <name> -g <resource-group>
    ```

> [!NOTE]
> `az connectedk8s delete` также будет удалять конфигурации поверх кластера. После выполнения `az connectedk8s connect` повторно создайте конфигурации в кластере вручную или с помощью политики Azure.

## <a name="if-i-am-already-using-cicd-pipelines-can-i-still-use-azure-arc-enabled-kubernetes-and-configurations"></a>Если я уже использую конвейеры CI/CD, могу ли я по-прежнему использовать Kubernetes и конфигурации с включенной службой Arc?

Да, вы по-прежнему можете использовать конфигурации в кластере, принимающем развертывания с помощью конвейера CI/CD. По сравнению с традиционными конвейерами CI/CD, конфигурации имеют два дополнительных преимущества:

**Выверка смещений**

Конвейер CI/CD применяет изменения только один раз во время выполнения конвейера. Однако оператор Гитопс в кластере постоянно опрашивает репозиторий Git для выборки требуемого состояния ресурсов Kubernetes в кластере. Если оператор Гитопс находит требуемое состояние ресурсов, отличающееся от фактического состояния ресурсов в кластере, это смещение выверено.

**Применение Гитопс в масштабе**

Конвейеры CI/CD полезны при развертывании на основе событий в кластере Kubernetes (например, при отправке в репозиторий Git). Однако если вы хотите развернуть ту же конфигурацию для всех кластеров Kubernetes, потребуется вручную настроить учетные данные каждого кластера Kubernetes для конвейера CI/CD. 

Так как Azure Resource Manager управляет конфигурациями Azure Arc, вы можете автоматизировать создание одинаковой конфигурации для всех ресурсов Kubernetes с поддержкой ARC в Azure с помощью политики Azure в области подписки или группы ресурсов. Эта возможность применима даже к ресурсам Kubernetes, включенным в службу Arc Azure, созданным после назначения политики.

Эта функция применяет базовые конфигурации (например, политики сети, привязки ролей и политики безопасности Pod) во всей инвентаризации кластера Kubernetes для удовлетворения требований соответствия и управления.

## <a name="next-steps"></a>Дальнейшие действия

* Пошаговое руководство по [подключению кластера Kubernetes к службе "Дуга Azure](./connect-cluster.md)".
* У вас уже есть кластер Kubernetes, подключенный к службе "Дуга Azure"? [Создайте конфигурации в кластере Kubernetes с включенной Arc](./use-gitops-connected-cluster.md).
* Узнайте, как [использовать политику Azure для применения конфигураций в масштабе](./use-azure-policy.md).