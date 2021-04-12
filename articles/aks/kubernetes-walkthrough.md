---
title: Краткое руководство. Развертывание кластера AKS с помощью Azure CLI
description: Узнайте, как быстро создать кластер Kubernetes, развернуть приложение и отслеживать производительность в Службе Azure Kubernetes (AKS) с помощью Azure CLI.
services: container-service
ms.topic: quickstart
ms.date: 02/26/2021
ms.custom:
- H1Hack27Feb2017
- mvc
- devcenter
- seo-javascript-september2019
- seo-javascript-october2019
- seo-python-october2019
- devx-track-azurecli
- contperf-fy21q1
ms.openlocfilehash: 8adfd1a6e26a3381653ca9a794b124e201b9d481
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106106744"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-cluster-using-the-azure-cli"></a>Краткое руководство. Развертывание кластера Службы Azure Kubernetes с помощью Azure CLI

Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. В этом кратком руководстве вы выполните указанные ниже задачи.
* Развертывание кластера AKS с помощью Azure CLI. 
* Запуск многоконтейнерного приложения, которое включает в себя веб-интерфейс и экземпляр Redis в кластере. 
* Мониторинг работоспособности кластера и pod, на которых выполняется приложение.

  ![Приложение для голосования, развернутое в Службе Azure Kubernetes](./media/container-service-kubernetes-walkthrough/voting-app-deployed-in-azure-kubernetes-service.png)

В этом руководстве предполагается, что у вас есть некоторое представление о функциях Kubernetes. Дополнительные сведения см. в статье [Ключевые концепции Kubernetes для службы Azure Kubernetes (AKS)][kubernetes-concepts].

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Дополнительные сведения о создании пула узлов Windows Server см. в разделе [Создание кластера AKS, поддерживающего контейнеры Windows Server][windows-container-cli].

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.64 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

> [!NOTE]
> Если вы работаете в локальной среде, а не в Azure Cloud Shell, команды из этого краткого руководства следует выполнять от имени администратора.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[Группа ресурсов Azure](../azure-resource-manager/management/overview.md) — это логическая группа, в которой развертываются и управляются ресурсы Azure. В процессе создания группы ресурсов вам будет предложено указать расположение. Это расположение определяет следующее: 
* место хранения метаданных группы ресурсов;
* место выполнения ресурсов в Azure, если при их создании не указан другой регион. 

В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

Создайте группу ресурсов с помощью команды [az group create][az-group-create].


```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Выходные данные для успешно созданной группы ресурсов:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="enable-cluster-monitoring"></a>Включение мониторинга кластера

1. Убедитесь, что в подписке зарегистрированы *Microsoft.OperationsManagement* и *Microsoft.OperationalInsights*. Чтобы проверить состояние регистрации, сделайте следующее:

    ```azurecli
    az provider show -n Microsoft.OperationsManagement -o table
    az provider show -n Microsoft.OperationalInsights -o table
    ```
 
    Если это не так, зарегистрируйте *Microsoft.OperationsManagement* и *Microsoft.OperationalInsights* следующим образом:
 
    ```azurecli
    az provider register --namespace Microsoft.OperationsManagement
    az provider register --namespace Microsoft.OperationalInsights
    ```

2. Включите [Azure Monitor для контейнеров][azure-monitor-containers] с помощью параметра *--enable-addons monitoring*. 

## <a name="create-aks-cluster"></a>Создание кластера AKS

Теперь создайте кластер AKS с помощью команды [az aks create][az-aks-create]. В следующем примере создается кластер *myAKSCluster* с одним узлом. 

```azurecli-interactive
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
```

Через несколько минут выполнение команды завершается и отображаются сведения о кластере в формате JSON.

> [!NOTE]
> При создании кластера AKS автоматически создается вторая группа ресурсов для хранения ресурсов AKS. Дополнительные сведения см. в разделе [Почему с AKS создаются две группы ресурсов?](./faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Кластером Kubernetes можно управлять при помощи [kubectl][kubectl] клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. 

1. Чтобы установить `kubectl` локально, выполните команду [az aks install-cli][az-aks-install-cli]:

    ```azurecli
    az aks install-cli
    ```

2. Настройте в `kubectl` подключение к кластеру Kubernetes, выполнив команду [az aks get-credentials][az-aks-get-credentials]. Приведенная ниже команда:  
      * скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования;
      * использует `~/.kube/config`, расположение по умолчанию для [файла конфигурации Kubernetes][kubeconfig-file]. Чтобы указать другое расположение файла конфигурации Kubernetes, используйте параметр *--file*;


    ```azurecli-interactive
    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    ```

3. Проверьте подключение к кластеру, выполнив команду [kubectl get][kubectl-get]. Эта команда возвращает список узлов кластера.

    ```azurecli-interactive
    kubectl get nodes
    ```

    В выходных данных будет представлен один узел, созданный на предыдущих шагах. Убедитесь, что этот узел находится в состоянии *готовности*:

    ```output
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.12.8
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
    * Если вы используете Azure Cloud Shell, этот файл можно создать с помощью `code`, `vi` или `nano`, как при работе в виртуальной или физической системе:
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

    ```console
    kubectl apply -f azure-vote.yaml
    ```

    В выходных данных показаны успешно созданные развертывания и службы:

    ```output
    deployment "azure-vote-back" created
    service "azure-vote-back" created
    deployment "azure-vote-front" created
    service "azure-vote-front" created
    ```

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете. Процесс создания может занять несколько минут.

Ход выполнения можно отслеживать с помощью команды [kubectl get service][kubectl-get] с аргументом `--watch`.

```azurecli-interactive
kubectl get service azure-vote-front --watch
```

Параметр **EXTERNAL-IP** для службы `azure-vote-front` в выходных данных изначально будут иметь значение *pending* (Ожидается).

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда параметр **EXTERNAL-IP** вместо *pending* примет значение общедоступного IP-адреса, выполните команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```output
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Чтобы увидеть приложение для голосования Azure в действии, откройте в веб-браузере внешний IP-адрес вашей службы.

![Приложение для голосования, развернутое в Службе Azure Kubernetes](./media/container-service-kubernetes-walkthrough/voting-app-deployed-in-azure-kubernetes-service.png)

Проверьте на портале Azure метрики работоспособности узлов и pod кластера, собранные службой [Azure Monitor для контейнеров][azure-monitor-containers]. 

## <a name="delete-the-cluster"></a>Удаление кластера

Чтобы не оплачивать ненужные ресурсы Azure, удалите их. Используйте команду [az group delete][az-group-delete], чтобы удалить группу ресурсов, службу контейнеров и все связанные с ними ресурсы.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
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
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeconfig-file]: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[aks-monitor]: ../azure-monitor/containers/container-insights-onboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[az-aks-browse]: /cli/azure/aks#az_aks_browse
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-aks-install-cli]: /cli/azure/aks#az_aks_install_cli
[az-group-create]: /cli/azure/group#az_group_create
[az-group-delete]: /cli/azure/group#az_group_delete
[azure-cli-install]: /cli/azure/install_azure_cli
[azure-monitor-containers]: ../azure-monitor/containers/container-insights-overview.md
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[azure-portal]: https://portal.azure.com
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[kubernetes-dashboard]: kubernetes-dashboard.md
[windows-container-cli]: windows-container-cli.md
