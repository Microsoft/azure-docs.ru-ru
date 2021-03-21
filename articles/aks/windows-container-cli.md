---
title: Создание контейнера Windows Server в кластере AKS с помощью Azure CLI
description: Узнайте, как быстро создать кластер Kubernetes и развернуть приложение в контейнере Windows Server в Службе Azure Kubernetes (AKS) с помощью Azure CLI.
services: container-service
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: 13b4fbd21bb348d1ef79a3ca68128869115745cc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103200911"
---
# <a name="create-a-windows-server-container-on-an-azure-kubernetes-service-aks-cluster-using-the-azure-cli"></a>Создание контейнера Windows Server в кластере Службы Azure Kubernetes (AKS) с помощью Azure CLI

Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. В этой статье показано, как развернуть кластер AKS с помощью Azure CLI. Затем вы развернете в кластере пример приложения ASP.NET, помещенный в контейнер Windows Server.

![Изображение: поиск примера приложения ASP.NET](media/windows-container/asp-net-sample-app.png)

В этой статье предполагается базовое понимание концепций Kubernetes. Дополнительные сведения см. в статье [Ключевые концепции Kubernetes для службы Azure Kubernetes (AKS)][kubernetes-concepts].

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

### <a name="limitations"></a>Ограничения

При создании и администрировании кластеров AKS, поддерживающих несколько пулов узлов, действуют следующие ограничения:

* Невозможно удалить первый пул узлов.

К пулам узлов Windows Server применяются следующие дополнительные ограничения:

* Кластер AKS не может содержать больше 10 пулов узлов.
* Кластер AKS не может содержать больше 100 узлов в каждом пуле узлов.
* Длина имени пула узлов Windows Server ограничена шестью символами.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure — это логическая группа, в которой развертываются и управляются ресурсы Azure. Во время создания группы ресурсов вам будет предложено указать расположение. В этом расположении сохраняются метаданные группы ресурсов, а также выполняется их работа в Azure, если во время создания ресурса вы не указали другой регион. Создайте группу ресурсов с помощью команды [az group create][az-group-create].

В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

> [!NOTE]
> В этом руководстве используется синтаксис команд, используемый в оболочке Bash.
> Если вы используете Azure Cloud Shell, в раскрывающемся списке в левом верхнем углу окна Cloud Shell обязательно установите значение **Bash**.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

В следующем примере выходных данных показано, что группа ресурсов успешно создана:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

Чтобы запустить кластер AKS, который поддерживает пулы узлов для контейнеров Windows Server, такой кластер должен использовать сетевую политику с подключаемым сетевым модулем [Azure CNI][azure-cni-about] (расширенным). Более подробные сведения о том, как планировать диапазоны подсетей и оценивать требования к сети, см в статье о [настройке сетевого взаимодействия Azure CNI][use-advanced-networking]. Используйте команду [AZ AKS Create][az-aks-create] , чтобы создать кластер AKS с именем *myAKSCluster*. Эта команда создаст обязательные сетевые ресурсы, если они еще не существуют.

* Кластер настраивается с двумя узлами.
* `--windows-admin-password`Параметры и `--windows-admin-username` задают учетные данные администратора для всех контейнеров Windows Server, созданных в кластере, и должны соответствовать [требованиям к паролю Windows Server][windows-server-password]. Если не указать параметр *Windows-Admin-Password* , будет предложено ввести значение.
* Пул узлов использует `VirtualMachineScaleSets` .

> [!NOTE]
> Чтобы обеспечить надежную работу кластера, создайте не менее 2 (двух) узлов в пуле узлов по умолчанию.

Создайте имя пользователя для использования в качестве учетных данных администратора для контейнеров Windows Server в кластере. Следующие команды запрашивают имя пользователя и задают его WINDOWS_USERNAME для использования в более поздней команде (Помните, что команды в этой статье введены в оболочку BASH).

```azurecli-interactive
echo "Please enter the username to use as administrator credentials for Windows Server containers on your cluster: " && read WINDOWS_USERNAME
```

Создайте кластер, убедившись, что задан `--windows-admin-username` параметр. В следующем примере команда создает кластер, используя значение из *WINDOWS_USERNAME* , заданное в предыдущей команде. Кроме того, вместо использования *WINDOWS_USERNAME* можно указать другое имя пользователя непосредственно в параметре. При выполнении следующей команды также будет предложено создать пароль для учетных данных администратора для контейнеров Windows Server в кластере. Кроме того, можно использовать параметр *Windows-Admin-Password* и указать в нем собственное значение.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --windows-admin-username $WINDOWS_USERNAME \
    --vm-set-type VirtualMachineScaleSets \
    --network-plugin azure
```

> [!NOTE]
> Если вы получаете сообщение об ошибке проверки пароля, проверьте, что заданный пароль соответствует [требованиям к паролю Windows Server][windows-server-password]. Если ваш пароль соответствует требованиям, попробуйте создать группу ресурсов в другом регионе. Затем попытайтесь создать кластер с новой группой ресурсов.

Через несколько минут выполнение команды завершается и отображаются сведения о кластере в формате JSON. Инициализация кластера иногда может выполняться не сразу же. В таком случае подождите примерно 10 минут.

## <a name="add-a-windows-server-node-pool"></a>Добавление пула узлов Windows Server

По умолчанию кластер AKS создается с пулом узлов, который может выполнять контейнеры Linux. Выполните команду `az aks nodepool add`, чтобы добавить к пулу узлов Linux дополнительный пул узлов, который может выполнять контейнеры Windows Server.

```azurecli
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --os-type Windows \
    --name npwin \
    --node-count 1
```

Приведенная выше команда создает новый пул узлов с именем *npwin* и добавляет его в *myAKSCluster*. При создании пула узлов для запуска контейнеров Windows Server для параметра *node-vm-size* по умолчанию задается значение *Standard_D2s_v3*. Если вы хотите указать другое значение для параметра *node-vm-size*, не забудьте проверить список [ограниченных размеров виртуальных машин][restricted-vm-sizes]. Минимальное рекомендуемое значение размера: *Standard_D2s_v3*. Приведенная выше команда также использует подсеть по умолчанию в виртуальной сети по умолчанию (создается при запуске `az aks create`).

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Управлять кластером Kubernetes можно c помощью [kubectl][kubectl], клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Чтобы установить `kubectl` локально, используйте команду [az aks install-cli][az-aks-install-cli]:

```azurecli
az aks install-cli
```

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][az-aks-get-credentials]. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Чтобы проверить подключение к кластеру, используйте команду [kubectl get][kubectl-get] для получения списка узлов кластера.

```console
kubectl get nodes
```

В следующем примере выходных данных содержится полный список узлов кластера. Убедитесь, что все узлы находятся в состоянии *Ready* (Готово):

```output
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.16.9
aksnpwin987654                      Ready    agent   108s   v1.16.9
```

## <a name="run-the-application"></a>Выполнение приложения

Файл манифеста Kubernetes определяет требуемое состояние для кластера, включая образы контейнеров, которые нужно запустить. В этой статье предполагается, что манифест используется для создания всех объектов, необходимых для запуска примера приложения ASP.NET в контейнере Windows Server. Этот манифест включает [развертывание Kubernetes][kubernetes-deployment] для примера приложения ASP.NET и внешнюю [службу Kubernetes][kubernetes-service] для получения доступа к приложению через Интернет.

Пример приложения ASP.NET предоставляется в составе пакета [примеров для .NET Framework][dotnet-samples] и выполняется в контейнере Windows Server. AKS требует использовать контейнеры Windows Server на основе образов *Windows Server 2019* или более поздней версии. Файл манифеста Kubernetes также должен определять [средство выбора узла][node-selector], чтобы кластер AKS выполнял pod этого примера приложения ASP.NET на узле с поддержкой контейнеров Windows Server.

Создайте файл `sample.yaml` и скопируйте в него следующее определение YAML. Если вы используете Azure Cloud Shell, этот файл можно создать с помощью `vi` или `nano`, как при работе в виртуальной или физической системе:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
  labels:
    app: sample
spec:
  replicas: 1
  template:
    metadata:
      name: sample
      labels:
        app: sample
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      containers:
      - name: sample
        image: mcr.microsoft.com/dotnet/framework/samples:aspnetapp
        resources:
          limits:
            cpu: 1
            memory: 800M
          requests:
            cpu: .1
            memory: 300M
        ports:
          - containerPort: 80
  selector:
    matchLabels:
      app: sample
---
apiVersion: v1
kind: Service
metadata:
  name: sample
spec:
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: sample
```

Разверните приложение с помощью команды [kubectl apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f sample.yaml
```

Следующий пример выходных данных указывает, что развертывание и служба успешно созданы.

```output
deployment.apps/sample created
service/sample created
```

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете. Процесс создания может занять несколько минут. Инициализация службы иногда может выполняться не сразу же. В таком случае подождите примерно 10 минут.

Чтобы отслеживать ход выполнения, используйте команду [kubectl get service][kubectl-get] с аргументом `--watch`.

```console
kubectl get service sample --watch
```

Изначально для параметра *EXTERNAL-IP* службы *sample* отображается состояние *pending*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
sample             LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда значение *EXTERNAL-IP* изменится с состояния *pending* на фактический общедоступный IP-адрес, используйте команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```output
sample  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Чтобы увидеть пример приложения в действии, откройте в веб-браузере внешний IP-адрес вашей службы.

![Изображение: поиск примера приложения ASP.NET](media/windows-container/asp-net-sample-app.png)

> [!Note]
> Если при попытке загрузить страницу время ожидания истекает, проверьте готовность примера приложения с помощью следующей команды [kubectl get pods --watch]. Иногда контейнер Windows запускается несколько позже, чем становится доступным внешний IP-адрес.

## <a name="delete-cluster"></a>Удаление кластера

Чтобы удалить ненужные кластер, группу ресурсов, службу контейнеров и все связанные с ней ресурсы, выполните команду [az group delete][az-group-delete].

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями][sp-delete]. Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье показано, как развернуть кластер Kubernetes, а в нем — пример приложения ASP.NET, размещенного в контейнере Windows Server. Для нового кластера вы можете открыть [веб-панель мониторинга Kubernetes][kubernetes-dashboard].

Дополнительные сведения о AKS и инструкции по созданию полного кода для примера развертывания см. в руководстве по кластерам Kubernetes.

> [!div class="nextstepaction"]
> [Руководство по AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[dotnet-samples]: https://hub.docker.com/_/microsoft-dotnet-framework-samples/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[aks-monitor]: ../azure-monitor/containers/container-insights-onboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[az-aks-browse]: /cli/azure/aks#az-aks-browse
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-install-cli]: /cli/azure/aks#az-aks-install-cli
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[az-provider-register]: /cli/azure/provider#az-provider-register
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-cni-about]: concepts-network.md#azure-cni-advanced-networking
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[azure-portal]: https://portal.azure.com
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[kubernetes-dashboard]: kubernetes-dashboard.md
[restricted-vm-sizes]: quotas-skus-regions.md#restricted-vm-sizes
[use-advanced-networking]: configure-azure-cni.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[windows-server-password]: /windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference