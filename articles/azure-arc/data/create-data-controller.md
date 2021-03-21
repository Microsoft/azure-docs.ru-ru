---
title: Создание контроллера данных
description: Создайте контроллер данных Arc Azure в типичном кластере Kubernetes с несколькими узлами, который вы уже развернули.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 03/02/2021
ms.topic: how-to
ms.openlocfilehash: 329df78bb5829695b95fcca5b7ed7e1439ced821
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101688371"
---
# <a name="create-the-azure-arc-data-controller"></a>Создание контроллера данных ARC в Azure

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="overview-of-creating-the-azure-arc-data-controller"></a>Общие сведения о создании контроллера данных ARC в Azure

Службы данных с поддержкой дуги Azure можно создавать на различных типах кластеров Kubernetes и управляемых служб Kubernetes с помощью различных подходов.

В настоящее время в списке поддерживаемых служб и дистрибутивов Kubernetes поддерживается следующее:

- Служба Azure Kubernetes (AKS)
- Подсистема Azure Kubernetes Service Engine (подсистема AKS) на Azure Stack
- Служба Azure Kubernetes в Azure Stack HCI
- Azure RedHat OpenShift (ARO)
- Платформа контейнеров OpenShift (OCP)
- AWS Elastic Kubernetes Service (EKS)
- Подсистема Google Cloud Kubernetes (ГКЕ)
- Служба с открытым исходным кодом Kubernetes обычно развертывается с помощью kubeadm

> [!IMPORTANT]
> * Минимальная поддерживаемая версия Kubernetes — v 1.17. Дополнительные сведения см. в статье [Известные проблемы](./release-notes.md#known-issues) . 
> * Минимальная поддерживаемая версия OCP — 4,5.
> * Ознакомьтесь с [требованиями к подключению](connectivity.md) , чтобы узнать, какое подключение требуется для вашей среды и Azure.
> * Сведения о настройке постоянного хранилища см. в [руководстве по настройке хранилища](storage-configuration.md) .
> * Если вы используете службу Azure Kubernetes, размер виртуальной машины рабочего узла кластера должен быть не менее **Standard_D8s_v3** и использовать диски уровня " **Премиум".** Кластер не должен охватывать несколько зон доступности. 
> * Если вы используете другое Kubernetesное распространение или службу, следует убедиться в наличии минимального размера узла (8 ГБ ОЗУ и 4 ядер) и общей емкости 32 ГБ ОЗУ для всех узлов Kubernetes. Например, у вас может быть 1 узел с объемом ОЗУ 32 ГБ и 4 ядрами или иметь 2 узла с 16 ГБ ОЗУ и 4 ядрами.

> [!NOTE]
> Если вы используете платформу контейнеров Red Hat OpenShift в Azure, рекомендуется использовать последнюю доступную версию.

В зависимости от выбранного варианта _требуются_ некоторые средства, но рекомендуется [установить все клиентские средства](./install-client-tools.md) , прежде чем приступать к созданию контроллера данных ARC в Azure.

Независимо от выбранного варианта в процессе создания необходимо предоставить следующие сведения:

- **Имя контроллера данных** — описательное имя для контроллера данных, например "Рабочий контроллер данных", "контроллер данных в Сиэтле".
- **Username контроллера данных** — любое имя пользователя администратора контроллера данных.
- **Пароль контроллера данных** — пароль для пользователя администратора контроллера данных.
- **Имя пространства имен Kubernetes** — имя пространства имен Kubernetes, в котором требуется создать контроллер данных.
- **Режим подключения** — режим подключения определяет степень подключения из вашей среды служб данных, включенной в службу "Дуга Azure", в Azure. В настоящее время предварительная версия поддерживает только косвенно подключенные и напрямую подключенные режимы.  Дополнительные сведения см. в разделе [режим подключения](./connectivity.md). 
- **Идентификатор подписки Azure** — GUID подписки Azure, в которой требуется создать ресурс контроллера данных в Azure.
- **Имя группы ресурсов Azure** — имя группы ресурсов, в которой должен быть создан ресурс контроллера данных в Azure.
- **Расположение Azure** — расположение Azure, в котором будут храниться метаданные ресурсов контроллера данных в Azure. Список доступных регионов см. в статье [Глобальная инфраструктура и продукты Azure по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=azure-arc).

## <a name="next-steps"></a>Дальнейшие действия

Существует несколько вариантов создания контроллера данных Arc Azure.

> **Хотите попробовать?**  
> Быстро Начните работу с помощью [Azure Arc JumpStart](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) в службе Azure Kubernetes Service (AKS), AWS эластичной Kubernetes Service (ЕКС), Google Cloud Kubernetes Engine (гке) или на виртуальной машине Azure.
> 
- [Создание контроллера данных с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]](create-data-controller-using-azdata.md)
- [Создание контроллера данных с Azure Data Studio](create-data-controller-azure-data-studio.md)
- [Создание контроллера данных из портал Azure через записную книжку Jupyter в Azure Data Studio](create-data-controller-resource-in-azure-portal.md)
- [Создание контроллера данных с помощью таких средств Kubernetes, как kubectl или OC](create-data-controller-using-kubernetes-native-tools.md)
- [Создание контроллера данных с помощью Azure Arc JumpStart для ускоренного взаимодействия с тестовым развертыванием](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/)
