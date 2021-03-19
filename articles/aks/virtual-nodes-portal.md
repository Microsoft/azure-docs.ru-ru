---
title: Создание виртуальных узлов в Службе Azure Kubernetes (AKS) с помощью портала
description: Сведения о том, как с помощью портала Azure создать кластер Службы Azure Kubernetes (AKS), который использует виртуальные узлы для запуска групп pod.
services: container-service
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: c1ecaa88dd5329d86818565983a6ba891a6d8424
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104577837"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-in-the-azure-portal"></a>Создание и настройка кластера Службы Azure Kubernetes (AKS) для использования виртуальных узлов на портале Azure

В этой статье показано, как использовать портал Azure для создания и настройки ресурсов виртуальной сети и кластера AKS с включенными виртуальными узлами.

> [!NOTE]
> В [этой статье](virtual-nodes.md) приводятся общие сведения о доступности региона и ограничениях с помощью виртуальных узлов.

## <a name="before-you-begin"></a>Перед началом

Виртуальные узлы обеспечивают сетевое взаимодействие контейнеров pod, которые выполняются в Экземплярах контейнеров Azure (ACI) и кластере AKS. Для обеспечения этого взаимодействия создается подсеть виртуальной сети и назначаются делегированные разрешения. Виртуальные узлы работают только с кластерами AKS, созданными с помощью *Advanced* Networking (Azure CNI). По умолчанию кластеры AKS создаются с помощью *базовых* сетей (кубенет). В этой статье показано, как создать виртуальную сеть и подсети, а затем развернуть кластер AKS, использующий сетевое взаимодействие уровня "Расширенный".

Если вы не использовали ACI ранее, зарегистрируйте поставщик служб в подписке. Вы можете проверить состояние регистрации поставщика ACI с помощью команды [az provider list][az-provider-list], как показано в следующем примере.

```azurecli-interactive
az provider list --query "[?contains(namespace,'Microsoft.ContainerInstance')]" -o table
```

Поставщик *Microsoft.ContainerInstance* должен иметь состояние *Registered*, как показано в следующем примере выходных данных.

```output
Namespace                    RegistrationState    RegistrationPolicy
---------------------------  -------------------  --------------------
Microsoft.ContainerInstance  Registered           RegistrationRequired
```

Если поставщик имеет состояние *NotRegistered*, зарегистрируйте этот поставщик с помощью команды [az provider register][az-provider-register], как показано в следующем примере.

```azurecli-interactive
az provider register --namespace Microsoft.ContainerInstance
```

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу https://portal.azure.com.

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

В верхнем левом углу портала Azure выберите **Создать ресурс** > **Служба Kubernetes**.

На странице **Основные** настройте следующие параметры:

- *Сведения о проекте*. Выберите подписку Azure, а затем выберите или создайте группу ресурсов Azure, например *myResourceGroup*. Введите **Имя кластера Kubernetes**, например *myAKSCluster*.
- *Сведения о кластере*: Выберите регион и версию Kubernetes для кластера AKS.
- *Пул первичных узлов*. Выберите размер виртуальной машины для узлов AKS. Размер виртуальной машины **невозможно** изменить после развертывания кластера с AKS.
     - Выберите количество узлов для развертывания в кластере. Для задач в этой статье задайте для параметра **Число узлов** значение *1*. Количество узлов **можно** изменить после развертывания кластера.

Щелкните **Далее: пулы узлов**.

На странице **Пулы узлов** выберите *Включить виртуальные узлы*.

:::image type="content" source="media/virtual-nodes-portal/enable-virtual-nodes.png" alt-text="В браузере показано, как создать кластер с включенными виртуальными узлами на портал Azure. Параметр &quot;включить виртуальные узлы&quot; выделен.":::

По умолчанию создается удостоверение кластера. Это удостоверение кластера используется для связи с кластером и интеграции с другими службами Azure. По умолчанию это удостоверение кластера является управляемым удостоверением. Дополнительные сведения см. в статье о том, [как использовать управляемые удостоверения](use-managed-identity.md). Вы также можете использовать субъект-службу в качестве удостоверения кластера.

Также для кластера настраивается расширенная поддержка сети. Для виртуальных узлов настраивается отдельная подсеть виртуальной сети Azure. Эта подсеть получает делегированные права для подключения ресурсов Azure из кластеров AKS. Если у вас еще нет делегированной подсети, портал Azure создаст и настроит виртуальную сеть Azure и подсеть для использования с виртуальными узлами.

Выберите **Review + create** (Просмотреть и создать). После завершения проверки щелкните **Создать**.

Для создания кластера AKS и его подготовки к использованию потребуется несколько минут.

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. Управлять кластером Kubernetes можно при помощи [kubectl][kubectl], клиента командной строки Kubernetes. Клиент `kubectl` предварительно установлен в Azure Cloud Shell.

Чтобы открыть Cloud Shell, выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

Выполните команду [az aks get-credentials][az-aks-get-credentials], чтобы настроить подключение `kubectl` к кластеру Kubernetes. В следующем примере возвращаются учетные данные для имени кластера *myAKSCluster* в группе ресурсов *myResourceGroup*.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Чтобы проверить подключение к кластеру, используйте команду [kubectl get][kubectl-get] для получения списка узлов кластера.

```console
kubectl get nodes
```

В следующем примере выходных данных показано создание одного узла виртуальной машины и одного виртуального узла для Linux с именем *virtual-node-aci-linux*:

```output
NAME                           STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux         Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0       Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>Развертывание примера приложения

Создайте в Azure Cloud Shell файл с именем `virtual-node.yaml` и скопируйте в него следующий код YAML. Чтобы назначить контейнер для узла, определите параметры [nodeSelector][node-selector] и [toleration][toleration]. Они включают выполнение группы pod в этом виртуальном узле и позволяют убедиться, что функция успешно включена.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aci-helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aci-helloworld
  template:
    metadata:
      labels:
        app: aci-helloworld
    spec:
      containers:
      - name: aci-helloworld
        image: mcr.microsoft.com/azuredocs/aci-helloworld
        ports:
        - containerPort: 80
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
```

Запустите приложение, выполнив команду [kubectl apply][kubectl-apply].

```azurecli-interactive
kubectl apply -f virtual-node.yaml
```

Чтобы вывести список pod и назначенный узел, выполните команду [kubectl get pods][kubectl-get] с аргументом `-o wide`. Обратите внимание, что pod `virtual-node-helloworld` был назначен на узел `virtual-node-linux`.

```console
kubectl get pods -o wide
```

```output
NAME                                     READY     STATUS    RESTARTS   AGE       IP           NODE
virtual-node-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

Группа pod получает внутренний IP-адрес из подсети виртуальной сети Azure, которая делегирована для использования с виртуальными узлами.

> [!NOTE]
> При использовании образов, хранящихся в Реестре контейнеров Azure, [настройте и используйте секрет Kubernetes][acr-aks-secrets]. Текущее ограничение виртуальных узлов заключается в том, что невозможно использовать встроенную аутентификацию субъекта-службы Azure AD. Если вы не используете секрет, модули pod, запланированные на виртуальные узлы, не будут запускаться и сообщат об ошибке `HTTP response status code 400 error code "InaccessibleImage"`.

## <a name="test-the-virtual-node-pod"></a>Тестирование группы pod на виртуальном узле

Чтобы протестировать выполнение группы pod на виртуальном узле, откройте демонстрационную версию приложения в веб-клиенте. Так как группа pod получает внутренний IP-адрес, вы можете быстро проверить это подключение с другой группой pod в том же кластере AKS. Создайте тестовый системный модуль pod и свяжите с ним сеанс терминала.

```console
kubectl run -it --rm virtual-node-test --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11
```

Установите `curl` в модуле pod с помощью `apt-get`:

```console
apt-get update && apt-get install -y curl
```

Теперь откройте адрес этой группы pod с помощью `curl`, например, *http://10.241.0.4* . Укажите внутренний IP-адрес, который вы получили от предыдущей команды `kubectl get pods`.

```console
curl -L http://10.241.0.4
```

Отобразится демонстрационное приложение, как показано в следующем сокращенном примере выходных данных:

```output
<html>
<head>
  <title>Welcome to Azure Container Instances!</title>
</head>
[...]
```

Закройте сеанс подключения терминала к проверяемой группе pod, выполнив `exit`. Когда сеанс завершится, группа pod автоматически удаляется.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье мы назначили выполнение группы pod в виртуальном узле и присвоили частный внутренний IP-адрес. В качестве альтернативы вы можете создать развертывание службы и направлять трафик в группу pod через подсистему балансировки нагрузки или контроллер входящего трафика. Дополнительные сведения см. в статье [Создание контроллера входящего трафика в Службе Azure Kubernetes (AKS)][aks-basic-ingress].

Виртуальные узлы — это один из компонентов, необходимых для масштабирования решения в AKS. Дополнительные сведения о таких решениях см. в следующих статьях:

- [Масштабирование приложений в Службе Azure Kubernetes (AKS)][aks-hpa]
- [Автомасштабирование кластера с помощью службы Azure Kubernetes (AKS)][aks-cluster-autoscaler]
- [Пример автомасштабирования для виртуальных узлов][virtual-node-autoscale]
- [Дополнительные сведения о библиотеке Virtual Kubelet с открытым кодом][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[aks-github]: https://github.com/azure/aks/issues]
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
[acr-aks-secrets]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

<!-- LINKS - internal -->
[aks-network]: ./configure-azure-cni.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: cluster-autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[az-provider-list]: /cli/azure/provider#az-provider-list
[az-provider-register]: /cli/azure/provider#az-provider-register
