---
title: Общие сведения о кластере Kubernetes на устройстве с пограничным Pro Microsoft Azure Stack | Документация Майкрософт
description: Описывает, как Kubernetes реализуется на устройстве Azure Stack ребра Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 03/01/2021
ms.author: alkohli
ms.openlocfilehash: 72ba07090e6ce67501761d97876aa136f146d61c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102437933"
---
# <a name="kubernetes-on-your-azure-stack-edge-pro-gpu-device"></a>Kubernetes на устройстве GPU Pro на Azure Stack

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Kubernetes — это популярная платформа с открытым исходным кодом для управления контейнерными приложениями. В этой статье приводится обзор Kubernetes, а затем описывается, как Kubernetes работает на устройстве Azure Stack пограничной Pro. 

## <a name="about-kubernetes"></a>О Kubernetes 

Kubernetes предоставляет простую и надежную платформу для управления приложениями на основе контейнеров и связанными с ними компонентами сети и хранилища. Вы можете быстро создавать, доставлять и масштабировать контейнерные приложения с помощью Kubernetes.

Как открытая платформа, вы можете использовать Kubernetes для создания приложений с использованием предпочтительного языка программирования, библиотек ОС или шины обмена сообщениями. Для планирования и развертывания выпусков Kubernetes может интегрироваться с существующими средствами непрерывной интеграции и непрерывной поставки.

Дополнительные сведения см. в разделе [как работает Kubernetes](https://www.youtube.com/watch?v=q1PcAawa4Bg&list=PLLasX02E8BPCrIhFrc_ZiINhbRkYMKdPT&index=2&t=0s).

## <a name="kubernetes-on-azure-stack-edge-pro"></a>Kubernetes на Azure Stack пограничных Pro

На устройстве Azure Stack пограничной Pro можно создать кластер Kubernetes, настроив вычисление. При настройке роли вычислений кластер Kubernetes, включая главный и рабочий узлы, развертывается и настраивается для вас. Затем этот кластер используется для развертывания рабочей нагрузки `kubectl` с помощью, IOT EDGE или дуги Azure.

Устройство Azure Stack погранично Pro доступно как конфигурация с 1 узлом, которая образует кластер инфраструктуры. Кластер Kubernetes отделен от кластера инфраструктуры и развертывается поверх кластера инфраструктуры. Кластер инфраструктуры обеспечивает постоянное хранилище для устройства Azure Stack ребра Pro, а кластер Kubernetes — исключительно для оркестрации приложений. 

Кластер Kubernetes в этом случае имеет главный узел и рабочий узел. Узлы Kubernetes в кластере — это виртуальные машины, на которых работают ваши приложения и облачные рабочие процессы. 

Главный узел Kubernetes отвечает за поддержание требуемого состояния кластера. Главный узел также управляет рабочим узлом, который, в свою очередь, запускает контейнерные приложения. 

На следующей схеме показана реализация Kubernetes на 1-узловом Azure Stack устройстве Pro. Устройство с одним узлом не является высокодоступным, и в случае сбоя одного узла устройство выйдет из строя. Кластер Kubernetes также выходит из строя.

![Kubernetes архитектура для 1-узлового устройства Azure Stack пограничной Pro](media/azure-stack-edge-gpu-kubernetes-overview/kubernetes-architecture-1-node.png)

Дополнительные сведения об архитектуре кластера Kubernetes см. на странице Основные [понятия Kubernetes](https://kubernetes.io/docs/concepts/architecture/).


<!--The Kubernetes cluster control plane components make global decisions about the cluster. The control plane has:

- *kubeapiserver* that is the front end of the Kubernetes API and exposes the API.
- *etcd* that is a highly available key value store that backs up all the Kubernetes cluster data.
- *kube-scheduler* that makes scheduling decisions.
- *kube-controller-manager* that runs controller processes such as those for node controllers, replications controllers, endpoint controllers, and service account and token controllers. -->

## <a name="storage-volume-provisioning"></a>Подготовка тома хранилища

Для поддержки рабочих нагрузок приложений можно подключать тома хранилища для постоянных данных на Azure Stackных общих ресурсах устройства Pro. Можно использовать статические и динамические тома. 

Дополнительные сведения см. в статье варианты подготовки хранилища для приложений в [хранилище Kubernetes для устройства на Azure Stack пограничной Pro](azure-stack-edge-gpu-kubernetes-storage.md).

## <a name="networking"></a>Сеть

Kubernetes Networking позволяет настраивать обмен данными внутри сети Kubernetes, включая сети контейнеров в контейнеры, сети Pod-Pod, сети Pod-To-Service и сети "Интернет-обслуживание". Дополнительные сведения см. в статье сетевая модель в [Kubernetes Networking для устройства Azure Stack ребра Pro](azure-stack-edge-gpu-kubernetes-networking.md).

## <a name="updates"></a>Обновления

По мере появления новых версий Kubernetes можно обновить кластер, используя стандартные обновления, доступные для устройства Azure Stack пограничной Pro. Инструкции по обновлению см. в статье [Apply Updates for a Azure Stack ребр Pro](azure-stack-edge-gpu-install-update.md).

## <a name="access-monitoring"></a>Доступ, мониторинг

Кластер Kubernetes на устройстве Azure Stack ребра Pro позволяет Kubernetes управление доступом на основе ролей (Kubernetes RBAC). Дополнительные сведения см. [в статье Kubernetes управления доступом на основе ролей на устройстве Azure Stack ребра Pro GPU](azure-stack-edge-gpu-kubernetes-rbac.md).

Вы также можете отслеживать работоспособность кластера и ресурсов с помощью панели мониторинга Kubernetes. Также доступны журналы контейнеров. Дополнительные сведения см. в статье [Использование панели мониторинга Kubernetes для мониторинга работоспособности кластера Kubernetes на устройстве Azure Stack ребра Pro](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).

Azure Monitor также доступен в качестве надстройки для получения данных о работоспособности из контейнеров, узлов и контроллеров. Дополнительные сведения см. в разделе [Azure Monitor обзор](../azure-monitor/overview.md) .

<!--## Private container registry

Kubernetes on Azure Stack Edge Pro device allows for the private storage of your images by providing a local container registry.-->

## <a name="application-management"></a>Управление приложениями

После создания кластера Kubernetes на устройстве Azure Stack пограничной Pro можно управлять приложениями, развернутыми в этом кластере, с помощью любого из следующих методов.

- Собственный доступ через `kubectl`
- IoT Edge 
- Azure Arc

Эти методы описаны в следующих разделах.


### <a name="kubernetes-and-kubectl"></a>Kubernetes и kubectl

После развертывания кластера Kubernetes можно управлять приложениями, развернутыми в кластере локально, с клиентского компьютера. Для взаимодействия с приложениями используется собственное средство, например *kubectl* , через командную строку. 

Дополнительные сведения о развертывании кластера Kubernetes см. в подразделе [развертывание кластера Kubernetes на Azure Stack пограничном устройстве Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Дополнительные сведения об управлении см. в подразделе [Использование kubectl для управления кластером Kubernetes на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).


### <a name="kubernetes-and-iot-edge"></a>Kubernetes и IoT Edge

Kubernetes также можно интегрировать с рабочими нагрузками IoT Edge на Azure Stack Device Pro, где Kubernetes обеспечивает масштабирование, а экосистема и IoT предоставляет экосистему, ориентированную на IoT. Слой Kubernetes используется в качестве уровня инфраструктуры для развертывания рабочих нагрузок Azure IoT Edge. Время существования модуля и балансировка сетевой нагрузки управляются Kubernetes, в то время как платформа пограничной приложения управляется IoT Edge.

Дополнительные сведения о развертывании приложений в кластере Kubernetes с помощью IoT Edge см. по адресу: 

- [Предоставление приложений без отслеживания состояния на устройстве Pro Azure Stack с помощью IOT Edge](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md).


### <a name="kubernetes-and-azure-arc"></a>Kubernetes и дуга Azure

Служба "Дуга Azure" — это гибридное средство управления, которое позволяет развертывать приложения в кластерах Kubernetes. Служба "Дуга Azure" также позволяет использовать Azure Monitor для контейнеров для просмотра и мониторинга кластеров. Дополнительные сведения см. в [разделе что такое Azure-Arc Enabled Kubernetes?](../azure-arc/kubernetes/overview.md). Сведения о ценах на ARC Azure см. на странице [цен на дугу Azure](https://azure.microsoft.com/services/azure-arc/#pricing).

Начиная с 2021 марта Azure с включенной службой Arc Kubernetes будет общедоступна для пользователей и стандартных расходов на использование. В качестве клиента для предварительного ознакомления Azure Kubernetes с поддержкой Arc будет доступен бесплатно для Azure Stack пограничных устройств. Чтобы воспользоваться предварительной версией предложения, создайте [Поддержка](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest):

1. В разделе **Тип проблемы** выберите **Выставление счетов**.
2. В разделе **Подписка** выберите свою подписку.
3. В разделе **Служба** выберите **Мои службы**, а затем выберите **Azure Stack ребро**.
4. В разделе **ресурс** выберите свой ресурс.
5. В разделе **Сводка** введите описание проблемы.
6. В разделе **тип проблемы** выберите **непредвиденные расходы**.
7. В разделе **проблемный подтип** выберите **помощь в понимании расходов на бесплатную пробную версию**.


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о хранилище Kubernetes см. на [устройстве с Azure Stack пограничным Pro](azure-stack-edge-gpu-kubernetes-storage.md).
- Общие сведения о модели Kubernetes Networking на [устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-kubernetes-networking.md).
- Разверните [Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md) на портале Azure.
