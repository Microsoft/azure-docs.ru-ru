---
title: Общие сведения о Kubernetes Network on Azure Stack погранично Pro Device | Документация Майкрософт
description: Описывает, как Kubernetes Networking работает на устройстве с Azure Stack пограничном Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 02/21/2021
ms.author: alkohli
ms.openlocfilehash: 1a0fc569b7d552101c1e0bd1a065b2a91ebfa3c4
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102439618"
---
# <a name="kubernetes-networking-in-your-azure-stack-edge-pro-gpu-device"></a>Kubernetes Networking на устройстве GPU Azure Stack с пограничными устройствами

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

На устройстве Azure Stack пограничной Pro создается кластер Kubernetes при настройке роли вычислений. После создания кластера Kubernetes контейнерные приложения можно развернуть в кластере Kubernetes в модулях Pod. Существует несколько различных способов использования сети для модулей Pod в кластере Kubernetes. 

В этой статье описывается сеть в кластере Kubernetes в целом, а именно в контексте устройства Azure Stack пограничной Pro. 

## <a name="networking-requirements"></a>Требования к сети

Ниже приведен пример типичного 2-уровневого приложения, развернутого в кластере Kubernetes.

- Приложение имеет клиентскую часть веб-сервера и приложение базы данных в серверной части. 
- Каждому модулю назначается IP, но эти IP-адреса могут измениться при перезапуске и отработке отказа модуля Pod. 
- Каждое приложение состоит из нескольких модулей, и балансировка нагрузки трафика между всеми репликами Pod должна быть выполнена. 

![Требования к сети Kubernetes](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-networking-1.png)

Приведенный выше сценарий приводит к следующим требованиям к сети.

 - Пользователь приложения может получить доступ к внешнему приложению за пределами кластера Kubernetes, используя имя или IP-адрес. 
 - Приложения в кластере Kubernetes, например внешний интерфейс и серверные модули, должны иметь возможность взаимодействовать друг с другом.

Чтобы решить обе указанные выше потребности, мы представляем службу Kubernetes. 


## <a name="networking-services"></a>сетевые службы;

Существует два типа служб Kubernetes: 

- **Служба IP-адресов кластера** — Представьте себе, что она предоставляет постоянную конечную точку для модулей Pod приложения. Доступ к любому объекту Pod, связанному с этими службами, невозможен извне кластера Kubernetes. IP-адрес, используемый для этих служб, поступает из адресного пространства в частной сети. 
- **IP-адрес подсистемы балансировки нагрузки** , как и IP-служба кластера, но связанный IP-адрес поступает из внешней сети и к нему можно получить доступ извне кластера Kubernetes.


<!--## Networking example for an app


![Kubernetes networking example](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-networking-2.png)

Each of these applications pods has a label associated with it. For example, the web server application pods have a label `app = WS` and the service has a label selector which the same as `app = WS`. Whenever a service of type load balancer or cluster IP is created, there is a control loop that runs in the master and publishes an endpoint corresponding to this service. This service uses a combination of labels and label selectors to discover the pods associated with this service. As a pod gets created, the new endpoint for the pod is added to the endpoint mapping. Whenever a pod is deleted, it gets deleted from the endpoint mapping. Using this endpoint controller, the service has a most up-to-date view of the pods that make up this application.

For discovery of applications within the cluster, Kubernetes cluster has an inbuilt DNS server pod. This is a cluster DNS that resolves service names to cluster IP. Anytime a cluster IP service is created, a DNS record is added to the DNS server that maps the name of the service to the cluster internal IP. That is how the applications within the cluster can discover each other. For load balancing, there is also the `kube-proxy`. This runs on every node and captures the traffic that comes in through the cluster IP and then distributes the traffic across the pods. 

When an application or the end user would first use the IP address associated with the service of type load balancer to discover the service. Then it would use the label select `app = WS` to discover the pods associated with the application. The `kube-proxy` component would then distribute the traffic and ensure that it hits one of the web server application pods. If the web server app wanted to talk to the database app, then it would simply use the name of the service and using the name and the DNS server pod, resolve the name to an IP address. Again using labels and label selector, it would discover the pods associated with the database application. The `kube-proxy` would then distribute the traffic across each of the database app nodes.-->


## <a name="kubernetes-networking-on-azure-stack-edge-pro"></a>Kubernetes сети на Azure Stack пограничных Pro

Калико, Металлб и Core DNS — это все компоненты, устанавливаемые для работы в сети на Azure Stack пограничных Pro. 

- **Калико** назначает IP-адрес из диапазона частных IP-адресов каждому модулю Pod и настраивает сеть для этих модулей, чтобы модуль Pod на одном узле мог взаимодействовать с модулем на другом узле. 
- **Металлб** работает в модуле Pod в кластере и назначает IP-адрес службам с типом балансировщика нагрузки. IP-адреса балансировщика нагрузки выбираются из диапазона IP-адресов служб, предоставленного через локальный пользовательский интерфейс. 
- **Основной DNS** -сервер — эта надстройка настраивает имя службы СОПОСТАВЛЕНИЯ записей DNS на IP-адрес кластера.

IP-адреса, используемые для узлов Kubernetes и внешних служб, предоставляются через страницу " **вычисленная сеть** " в локальном пользовательском интерфейсе устройства.

![Назначение IP-адресов Kubernetes в локальном пользовательском интерфейсе](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-ip-assignment-local-ui-1.png)

Назначение IP-адресов:

- IP- **адреса узла Kubernetes**. Этот диапазон используется для главного и рабочего узлов Kubernetes. Эти IP-адреса используются, когда узлы Kubernetes обмениваются данными друг с другом.

- IP- **адреса внешней службы Kubernetes**. Этот диапазон используется для внешних служб (также известных как службы Load Balancer), которые доступны за пределами кластера Kubernetes.  


## <a name="next-steps"></a>Дальнейшие действия

Чтобы настроить Kubernetes Network на Azure Stack пограничных Pro, см.:

- [Предоставление приложения без отслеживания состояния извне на Azure Stack пограничных Pro с помощью IOT Edge](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md).

- [Предоставление приложения без отслеживания состояния извне на Azure Stack пограничных Pro через куебктл](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).
