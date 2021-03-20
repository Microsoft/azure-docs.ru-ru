---
title: Проверка применения рекомендаций в развертываниях
titleSuffix: Azure Kubernetes Service
description: Сведений о проверке применения рекомендаций в развертываниях в Службе Azure Kubernetes с помощью kube-advisor
services: container-service
author: seanmck
ms.topic: troubleshooting
ms.date: 11/05/2018
ms.author: seanmck
ms.openlocfilehash: 7730146f30487eb5d20f0d3138e9e5ba799daa99
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94681522"
---
# <a name="checking-for-kubernetes-best-practices-in-your-cluster"></a>Проверка применения рекомендаций Kubernetes в кластере

Чтобы обеспечить максимальную производительность и устойчивость приложений, следует соблюдать ряд рекомендаций в отношении развертываний Kubernetes. С помощью средства kube-advisor можно определить развертывания, которые не соответствуют этим рекомендациям.

## <a name="about-kube-advisor"></a>Сведения о kube-advisor

[Средство kube-advisor][kube-advisor-github] представляет собой контейнер, предназначенный для выполнения в кластере. Он запрашивает с сервера API Kubernetes сведения о ваших развертываниях и возвращает набор предложений по оптимизации.

Средство kube-advisor может сообщать о запросах на ресурсы и отсутствующих ограничениях в PodSpecs для приложений Windows, а также приложений Linux, но средство kube-advisor должно при этом быть запланировано в объекте pod Linux. Вы можете запланировать запуск объекта pod в пуле узлов с определенной ОС с помощью [селектора узла][k8s-node-selector] в конфигурации объекта pod.

> [!NOTE]
> Средство kube-advisor поддерживается корпорацией Майкрософт по мере возможности. Проблемы и предложения следует отправлять в GitHub.

## <a name="running-kube-advisor"></a>Запуск kube-advisor

Чтобы запустить средство в кластере, настроенном для [управления доступом на основе ролей Kubernetes (KUBERNETES RBAC)](./azure-ad-integration-cli.md), используйте следующие команды. Первая команда создает учетную запись службы Kubernetes. Вторая команда запускает средство в pod с помощью этой учетной записи службы и настраивает удаление pod после выхода. 

```bash
kubectl apply -f https://raw.githubusercontent.com/Azure/kube-advisor/master/sa.yaml

kubectl run --rm -i -t kubeadvisor --image=mcr.microsoft.com/aks/kubeadvisor --restart=Never --overrides="{ \"apiVersion\": \"v1\", \"spec\": { \"serviceAccountName\": \"kube-advisor\" } }" --namespace default
```

Если вы не используете Kubernetes RBAC, можно выполнить команду следующим образом:

```bash
kubectl run --rm -i -t kubeadvisor --image=mcr.microsoft.com/aks/kubeadvisor --restart=Never
```

Через несколько секунд должна появиться таблица с описанием возможных улучшений развертываний.

![Выходные данные kube-advisor](media/kube-advisor-tool/kube-advisor-output.png)

## <a name="checks-performed"></a>Выполняемые проверки

Средство проверяет выполнение ряда рекомендаций Kubernetes, предлагая для каждого собственный способ исправления.

### <a name="resource-requests-and-limits"></a>Запросы ресурсов и ограничения

Kubernetes поддерживает определение [запросов ресурсов и ограничений в спецификациях pod][kube-cpumem]. Запрос определяет минимальные ресурсы ЦП и памяти, необходимые для выполнения контейнера. Ограничение определяет максимальные допустимые ресурсы ЦП и памяти.

По умолчанию запросы и ограничения в спецификациях pod не заданы. Это может привести к чрезмерной загрузке узлов и нехватке ресурсов для контейнеров. Средство kube-advisor выявляет группы pod, для которых не заданы запросы и ограничения.

## <a name="cleaning-up"></a>Очистка.

Если в кластере включен Kubernetes RBAC, можно очистить `ClusterRoleBinding` после запуска средства с помощью следующей команды:

```bash
kubectl delete -f https://raw.githubusercontent.com/Azure/kube-advisor/master/sa.yaml
```

Если вы используете средство для кластера, который не поддерживает Kubernetes RBAC, очистка не требуется.

## <a name="next-steps"></a>Дальнейшие действия

- [Устранение неполадок со службой Azure Kubernetes](troubleshooting.md)

<!-- RESOURCES -->

[kube-cpumem]: https://github.com/Azure/azure-quickstart-templates
[kube-advisor-github]: https://github.com/azure/kube-advisor
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
