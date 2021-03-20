---
title: Создание кластера Kubernetes с включенной Azure Dev Spaces Azure Cloud Shell
services: azure-dev-spaces
ms.date: 10/04/2018
ms.topic: conceptual
description: Узнайте, как быстро создать кластер Kubernetes, поддерживающий Azure Dev Spaces, напрямую из браузера без какой-либо установки.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, containers, Helm, service mesh, service mesh routing, kubectl, k8s
ms.openlocfilehash: ce73f46a2451dfa87751d90f9cd6b31d5c80683f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91963556"
---
# <a name="create-a-kubernetes-cluster-with-azure-dev-spaces-enabled-with-azure-cloud-shell"></a>Создание кластера Kubernetes с включенной Azure Dev Spaces с Azure Cloud Shell

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Вы можете использовать [Azure Cloud Shell](/azure/cloud-shell) для создания кластера службы Azure Kubernetes, нажав кнопку " **попробовать** " на этой странице. Если вы не выполнили вход, следуйте инструкциям, чтобы войти с помощью учетной записи Azure, а затем введите команды в командной строке Azure Cloud Shell, когда она появится.

## <a name="create-the-cluster"></a>Создайте кластер.

Сначала создайте группу ресурсов в [регионе, который поддерживает Azure dev Spaces][supported-regions].

```azurecli-interactive
az group create --name MyResourceGroup --location <region>
```

Чтобы создать кластер Kubernetes, выполните следующую команду:

```azurecli-interactive
az aks create -g MyResourceGroup -n MyAKS --location <region> --generate-ssh-keys
```

Создание кластера занимает несколько минут.  По завершении выходные данные отображаются в формате JSON. Найдите `provisioningState` и убедитесь, что значение равно `Succeeded`.

## <a name="next-steps"></a>Дальнейшие действия

Ссылки на полные руководства см. в разделе [Рабочие среды Azure для разработчиков](../index.yml).

> [!IMPORTANT]
> Многие из кратких руководств Azure Dev Spaces и учебников используют Azure Dev Spaces CLI для выполнения операций. Интерфейс командной строки Azure Dev Spaces невозможно установить в Azure Cloud Shell.


[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
