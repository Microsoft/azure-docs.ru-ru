---
title: Краткое руководство. Развертывание кластера AKS с помощью PowerShell
description: Узнайте, как быстро создать кластер Kubernetes, развернуть приложение и отслеживать производительность в Службе Azure Kubernetes (AKS) с помощью PowerShell.
services: container-service
ms.topic: quickstart
ms.date: 03/15/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2b61c791390200beac4a18422a4de58dd94fa711
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103492903"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-cluster-using-powershell"></a>Краткое руководство. Развертывание кластера Службы Azure Kubernetes с помощью PowerShell

Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. В этом кратком руководстве вы выполните указанные ниже задачи.
* Развертывание кластера AKS с помощью PowerShell. 
* Запуск многоконтейнерного приложения, которое включает в себя веб-интерфейс и экземпляр Redis в кластере. 
* Мониторинг работоспособности кластера и pod, на которых выполняется приложение.

Дополнительные сведения о создании пула узлов Windows Server см. в разделе [Создание кластера AKS, поддерживающего контейнеры Windows Server][windows-container-powershell].

![Приложение для голосования, развернутое в Службе Azure Kubernetes](./media/kubernetes-walkthrough-powershell/voting-app-deployed-in-azure-kubernetes-service.png)

В этом руководстве предполагается, что у вас есть некоторое представление о функциях Kubernetes. Дополнительные сведения см. в статье [Ключевые концепции Kubernetes для службы Azure Kubernetes (AKS)][kubernetes-concepts].

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили запустить PowerShell локально, установите модуль Az PowerShell и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). См. сведения об [установке модуля Azure PowerShell][install-azure-powershell].

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Если вы используете несколько подписок Azure, выберите идентификатор той подписки, в которой будут выставляться счета за ресурсы c помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[Группа ресурсов Azure](../azure-resource-manager/management/overview.md) — это логическая группа, в которой развертываются и управляются ресурсы Azure. В процессе создания группы ресурсов вам будет предложено указать расположение. Это расположение определяет следующее: 
* место хранения метаданных группы ресурсов;
* место выполнения ресурсов в Azure, если при их создании не указан другой регион. 

В следующем примере создается группа ресурсов с именем **myResourceGroup** в регионе **eastus**.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup][new-azresourcegroup].

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location eastus
```

Выходные данные для успешно созданной группы ресурсов:

```plaintext
ResourceGroupName : myResourceGroup
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup
```

## <a name="create-aks-cluster"></a>Создание кластера AKS

1. Создайте пару ключей SSH с помощью служебной программы командной строки `ssh-keygen`. 
    * Дополнительные сведения см. в статье [Краткая инструкция. Создание и использование пары из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

1. Создайте кластер AKS с помощью командлета [New-AzAks][new-azaks]. Служба Azure Monitor для контейнеров также включена по умолчанию.

    В следующем примере создается кластер **myAKSCluster** с одним узлом. 

    ```azurepowershell-interactive
    New-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster -NodeCount 1
    ```

Через несколько минут выполнение команды завершается и отображаются сведения о кластере.

> [!NOTE]
> При создании кластера AKS для хранения ресурсов AKS автоматически создается вторая группа ресурсов. Дополнительные сведения см. в разделе [Почему с AKS создаются две группы ресурсов?](./faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Кластером Kubernetes можно управлять при помощи [kubectl][kubectl] клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. 

1. Установите `kubectl` локально с помощью командлета `Install-AzAksKubectl`:

    ```azurepowershell
    Install-AzAksKubectl
    ```

2. Настройте `kubectl` на подключение к кластеру Kubernetes с помощью командлета [ Import-AzAksCredential][import-azakscredential]. В следующем командлете выполняется скачивание учетных данных и настройка интерфейса командной строки Kubernetes для их использования.

    ```azurepowershell-interactive
    Import-AzAksCredential -ResourceGroupName myResourceGroup -Name myAKSCluster
    ```

3. Проверьте подключение к кластеру, выполнив команду [kubectl get][kubectl-get]. Эта команда возвращает список узлов кластера.

    ```azurepowershell-interactive
    .\kubectl get nodes
    ```

    В выходных данных будет представлен один узел, созданный на предыдущих шагах. Убедитесь, что этот узел находится в состоянии *готовности*:

    ```plaintext
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.15.10
    ```

## <a name="run-the-application"></a>Выполнение приложения

[Файл манифеста Kubernetes][kubernetes-deployment] используется для определения требуемого состояния кластера, например выполняемых в нем образов контейнеров. 

В этом кратком руководстве вы примените манифест для создания всех объектов, необходимых для запуска [приложения Azure для голосования][azure-vote-app]. Этот манифест содержит два [развертывания Kubernetes][kubernetes-deployment]:
* пример приложения Azure для голосования на языке Python;
* экземпляр Redis. 

Кроме того, создаются две [Службы Kubernetes][kubernetes-service]:
* внутренняя служба для экземпляра Redis;
* внешняя служба для доступа к приложению Azure для голосования из Интернета.

1. Создайте файл с именем `azure-vote.yaml`.
    * Если вы используете Azure Cloud Shell, этот файл можно создать с помощью `vi` или `nano`, как при обычной работе в виртуальной или физической системе.
1. Скопируйте в него следующее определение YAML:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-back
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-back
      template:
        metadata:
          labels:
            app: azure-vote-back
        spec:
          nodeSelector:
            "beta.kubernetes.io/os": linux
          containers:
          - name: azure-vote-back
            image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
            env:
            - name: ALLOW_EMPTY_PASSWORD
              value: "yes"
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
            - containerPort: 6379
              name: redis
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-back
    spec:
      ports:
      - port: 6379
      selector:
        app: azure-vote-back
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-front
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-front
      template:
        metadata:
          labels:
            app: azure-vote-front
        spec:
          nodeSelector:
            "beta.kubernetes.io/os": linux
          containers:
          - name: azure-vote-front
            image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
            - containerPort: 80
            env:
            - name: REDIS
              value: "azure-vote-back"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-front
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        app: azure-vote-front
    ```

1. Разверните приложение с помощью команды [kubectl apply][kubectl-apply] и укажите имя манифеста YAML:

    ```azurepowershell-interactive
    .\kubectl apply -f azure-vote.yaml
    ```

    В выходных данных показаны успешно созданные развертывания и службы:

    ```plaintext
    deployment.apps/azure-vote-back created
    service/azure-vote-back created
    deployment.apps/azure-vote-front created
    service/azure-vote-front created
    ```

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете. Процесс создания может занять несколько минут.

Ход выполнения можно отслеживать с помощью команды [kubectl get service][kubectl-get] с аргументом `--watch`.

```azurepowershell-interactive
.\kubectl get service azure-vote-front --watch
```

Параметр **EXTERNAL-IP** для службы `azure-vote-front` в выходных данных изначально будут иметь значение *pending* (Ожидается).

```plaintext
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда параметр **EXTERNAL-IP** вместо *pending* примет значение общедоступного IP-адреса, выполните команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```plaintext
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Чтобы увидеть приложение для голосования Azure в действии, откройте в веб-браузере внешний IP-адрес вашей службы.

![Приложение для голосования, развернутое в Службе Azure Kubernetes](./media/kubernetes-walkthrough-powershell/voting-app-deployed-in-azure-kubernetes-service.png)

Проверьте на портале Azure метрики работоспособности узлов кластера и модулей pod, собранные службой Azure Monitor для контейнеров. 

## <a name="delete-the-cluster"></a>Удаление кластера

Чтобы не оплачивать ненужные ресурсы Azure, очистите их. Чтобы удалить группу ресурсов, службу контейнеров и все связанные с ней ресурсы, используйте командлет [Remove-AzResourceGroup][remove-azresourcegroup].

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями][sp-delete].
> 
> Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="get-the-code"></a>Получение кода

В этом кратком руководстве для создания развертывания Kubernetes вы применили предварительно созданные образы контейнеров. Вы можете получить код приложений, файл Dockerfile и файл манифеста Kubernetes для этих образов [на сайте GitHub.][azure-vote-app]

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы развернули кластер Kubernetes, а затем многоконтейнерное приложение в нем. Узнайте, как [открыть веб-панель мониторинга Kubernetes][kubernetes-dashboard] для кластера AKS.

Дополнительные сведения о AKS и инструкции по созданию полного кода для примера развертывания см. в руководстве по кластерам Kubernetes.

> [!div class="nextstepaction"]
> [Руководство по AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[azure-dev-spaces]: ../dev-spaces/index.yml
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git

<!-- LINKS - internal -->
[windows-container-powershell]: windows-container-powershell.md
[kubernetes-concepts]: concepts-clusters-workloads.md
[install-azure-powershell]: /powershell/azure/install-az-ps
[new-azresourcegroup]: /powershell/module/az.resources/new-azresourcegroup
[new-azaks]: /powershell/module/az.aks/new-azaks
[import-azakscredential]: /powershell/module/az.aks/import-azakscredential
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[remove-azresourcegroup]: /powershell/module/az.resources/remove-azresourcegroup
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[kubernetes-dashboard]: kubernetes-dashboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
