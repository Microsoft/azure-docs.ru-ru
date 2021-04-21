---
title: Создание контейнера Windows Server в кластере AKS с помощью PowerShell
description: Узнайте, как быстро создать кластер Kubernetes и развернуть приложение в контейнере Windows Server в Службе Azure Kubernetes (AKS) с помощью PowerShell.
services: container-service
ms.topic: article
ms.date: 03/12/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b877ecbdca06ff73d152e1b491e993798a99f98a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103233520"
---
# <a name="create-a-windows-server-container-on-an-azure-kubernetes-service-aks-cluster-using-powershell"></a>Создание контейнера Windows Server в кластере Службы Azure Kubernetes (AKS) с помощью PowerShell

Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. В этой статье показано, как развернуть кластер AKS с помощью PowerShell. Затем вы развернете в кластере пример приложения `ASP.NET`, помещенный в контейнер Windows Server.

![Изображение: поиск примера приложения ASP.NET](media/windows-container-powershell/asp-net-sample-app.png)

В этой статье предполагается базовое понимание концепций Kubernetes. Дополнительные сведения см. в статье [Ключевые концепции Kubernetes для службы Azure Kubernetes (AKS)][kubernetes-concepts].

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). См. сведения об [установке модуля Azure PowerShell][install-azure-powershell]. Также необходимо установить модуль PowerShell Az.Aks: 

```azurepowershell-interactive
Install-Module Az.Aks
```

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите идентификатор требуемой подписки с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="limitations"></a>Ограничения

При создании и администрировании кластеров AKS, поддерживающих несколько пулов узлов, действуют следующие ограничения:

* Невозможно удалить первый пул узлов.

К пулам узлов Windows Server применяются следующие дополнительные ограничения:

* Кластер AKS не может содержать больше 10 пулов узлов.
* Кластер AKS не может содержать больше 100 узлов в каждом пуле узлов.
* Длина имени пула узлов Windows Server ограничена шестью символами.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[Группа ресурсов Azure](../azure-resource-manager/management/overview.md) — это логическая группа, в которой развертываются и управляются ресурсы Azure. Во время создания группы ресурсов вам будет предложено указать расположение. В этом расположении сохраняются метаданные группы ресурсов, а также выполняется их работа в Azure, если во время создания ресурса вы не указали другой регион. Создайте группу ресурсов с помощью командлета [New-AzResourceGroup][new-azresourcegroup].

В следующем примере создается группа ресурсов с именем **myResourceGroup** в расположении **eastus**.

> [!NOTE]
> В этом руководстве используется синтаксис команд, используемый в оболочке PowerShell. Если вы используете Azure Cloud Shell, в раскрывающемся списке в левом верхнем углу окна Cloud Shell обязательно установите значение **PowerShell**.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location eastus
```

В следующем примере выходных данных показано, что группа ресурсов успешно создана:

```plaintext
ResourceGroupName : myResourceGroup
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup
```

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

Создайте пару ключей SSH с помощью служебной программы командной строки `ssh-keygen`. Дополнительные сведения см. в статье [Краткая инструкция. Создание и использование пары из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

Чтобы запустить кластер AKS, который поддерживает пулы узлов для контейнеров Windows Server, такой кластер должен использовать сетевую политику с подключаемым сетевым модулем [Azure CNI][azure-cni-about] (расширенным). Более подробные сведения о том, как планировать диапазоны подсетей и оценивать требования к сети, см в статье о [настройке сетевого взаимодействия Azure CNI][use-advanced-networking]. Приведенный ниже командлет [New-AzAks][new-azaks] позволяет создать кластер AKS с именем **myAKSCluster**. Следующий пример создает обязательные сетевые ресурсы, если они еще не существуют.

> [!NOTE]
> Чтобы обеспечить надежную работу кластера, создайте не менее 2 (двух) узлов в пуле узлов по умолчанию.

```azurepowershell-interactive
$Username = Read-Host -Prompt 'Please create a username for the administrator credentials on your Windows Server containers: '
$Password = Read-Host -Prompt 'Please create a password for the administrator credentials on your Windows Server containers: ' -AsSecureString
New-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster -NodeCount 2 -NetworkPlugin azure -NodeVmSetType VirtualMachineScaleSets -WindowsProfileAdminUserName $Username -WindowsProfileAdminUserPassword $Password
```

> [!Note]
> Если кластер AKS невозможно создать из-за того, что нужная версия не поддерживается в вашем регионе, выполните команду `Get-AzAksVersion -Location eastus`, чтобы получить список поддерживаемых версий для этого региона.

Через несколько минут выполнение команды завершается и отображаются сведения о кластере. Инициализация кластера иногда может выполняться не сразу же. В таком случае подождите примерно 10 минут.

## <a name="add-a-windows-server-node-pool"></a>Добавление пула узлов Windows Server

По умолчанию кластер AKS создается с пулом узлов, который может выполнять контейнеры Linux. Используйте командлет `New-AzAksNodePool`, чтобы добавить к пулу узлов Linux пул узлов, который может выполнять контейнеры Windows Server.

```azurepowershell-interactive
New-AzAksNodePool -ResourceGroupName myResourceGroup -ClusterName myAKSCluster -VmSetType VirtualMachineScaleSets -OsType Windows -Name npwin
```

Приведенная выше команда создает новый пул узлов с именем **npwin** и добавляет его в **myAKSCluster**. При создании пула узлов для запуска контейнеров Windows Server для параметра **VmSize** по умолчанию задается значение **Standard_D2s_v3**. Если вы хотите указать другое значение для параметра **VmSize**, не забудьте проверить список [ограниченных размеров виртуальных машин][restricted-vm-sizes]. Минимальное рекомендуемое значение размера: **Standard_D2s_v3**. Приведенная выше команда также использует подсеть по умолчанию в виртуальной сети по умолчанию, созданной при запуске `New-AzAks`.

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Управлять кластером Kubernetes можно c помощью [kubectl][kubectl], клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Чтобы установить `kubectl` локально, используйте командлет `Install-AzAksKubectl`.

```azurepowershell-interactive
Install-AzAksKubectl
```

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, используйте командлет [Import-AzAksCredential][import-azakscredential]. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurepowershell-interactive
Import-AzAksCredential -ResourceGroupName myResourceGroup -Name myAKSCluster
```

Чтобы проверить подключение к кластеру, используйте команду [kubectl get][kubectl-get] для получения списка узлов кластера.

```azurepowershell-interactive
kubectl get nodes
```

В следующем примере выходных данных содержится полный список узлов кластера. Убедитесь, что все узлы находятся в состоянии **Ready** (Готово):

```plaintext
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.16.7
aksnpwin987654                      Ready    agent   108s   v1.16.7
```

## <a name="run-the-application"></a>Выполнение приложения

Файл манифеста Kubernetes определяет требуемое состояние для кластера, включая образы контейнеров, которые нужно запустить. В этой статье предполагается, что манифест используется для создания всех объектов, необходимых для запуска примера приложения ASP.NET в контейнере Windows Server. Этот манифест включает [развертывание Kubernetes][kubernetes-deployment] для примера приложения ASP.NET и внешнюю [службу Kubernetes][kubernetes-service] для получения доступа к приложению через Интернет.

Пример приложения ASP.NET предоставляется в составе пакета [примеров для .NET Framework][dotnet-samples] и выполняется в контейнере Windows Server. AKS требует использовать контейнеры Windows Server на основе образов **Windows Server 2019** или более поздней версии. Файл манифеста Kubernetes также должен определять [средство выбора узла][node-selector], чтобы кластер AKS выполнял pod этого примера приложения ASP.NET на узле с поддержкой контейнеров Windows Server.

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

```azurepowershell-interactive
kubectl apply -f sample.yaml
```

Следующий пример выходных данных указывает, что развертывание и служба успешно созданы.

```plaintext
deployment.apps/sample created
service/sample created
```

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете.
Процесс создания может занять несколько минут. Инициализация службы иногда может выполняться не сразу же. В таком случае подождите примерно 10 минут.

Чтобы отслеживать ход выполнения, используйте команду [kubectl get service][kubectl-get] с аргументом `--watch`.

```azurepowershell-interactive
kubectl get service sample --watch
```

Изначально для параметра **EXTERNAL-IP** службы **sample** отображается состояние **pending**.

```plaintext
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
sample             LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда значение **EXTERNAL-IP** изменится с состояния **pending** на фактический общедоступный IP-адрес, используйте команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```plaintext
sample  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Чтобы увидеть пример приложения в действии, откройте в веб-браузере внешний IP-адрес вашей службы.

![Изображение: поиск примера приложения ASP.NET](media/windows-container-powershell/asp-net-sample-app.png)

> [!Note]
> Если при попытке загрузить страницу время ожидания истекает, проверьте готовность примера приложения с помощью следующей команды `kubectl get pods --watch`. Иногда контейнер Windows запускается несколько позже, чем становится доступным внешний IP-адрес.

## <a name="delete-cluster"></a>Удаление кластера

Чтобы удалить ненужные кластер, группу ресурсов, службу контейнеров и все связанные с ней ресурсы, используйте командлет [Remove-AzResourceGroup][remove-azresourcegroup].

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями][sp-delete]. Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье показано, как развернуть кластер Kubernetes, а в нем — пример приложения `ASP.NET`, размещенного в контейнере Windows Server. [Получите доступ к веб-панели Kubernetes][kubernetes-dashboard] для кластера, который вы создали.

Дополнительные сведения о AKS и инструкции по созданию полного кода для примера развертывания см. в руководстве по кластерам Kubernetes.

> [!div class="nextstepaction"]
> [Руководство по AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[dotnet-samples]: https://hub.docker.com/_/microsoft-dotnet-framework-samples/
[node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[install-azure-powershell]: /powershell/azure/install-az-ps
[new-azresourcegroup]: /powershell/module/az.resources/new-azresourcegroup
[azure-cni-about]: concepts-network.md#azure-cni-advanced-networking
[use-advanced-networking]: configure-azure-cni.md
[new-azaks]: /powershell/module/az.aks/new-azaks
[restricted-vm-sizes]: quotas-skus-regions.md#restricted-vm-sizes
[import-azakscredential]: /powershell/module/az.aks/import-azakscredential
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[remove-azresourcegroup]: /powershell/module/az.resources/remove-azresourcegroup
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[kubernetes-dashboard]: kubernetes-dashboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
