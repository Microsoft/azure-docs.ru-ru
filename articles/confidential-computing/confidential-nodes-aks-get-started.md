---
title: Краткое руководство. Развертывание кластера Службы Azure Kubernetes (AKS) с узлами конфиденциальных вычислений с помощью Azure CLI
description: В этом кратком руководстве вы узнаете, как создать кластер AKS с конфиденциальными узлами и развернуть приложение Hello World с помощью Azure CLI.
author: agowdamsft
ms.service: container-service
ms.subservice: confidential-computing
ms.topic: quickstart
ms.date: 03/18/2020
ms.author: amgowda
ms.custom: contentperf-fy21q3
ms.openlocfilehash: 73770acefc8a153e4a2f2fde146f9afd4c319cd3
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105933140"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-aks-cluster-with-confidential-computing-nodes-dcsv2-using-azure-cli"></a>Краткое руководство. Развертывание кластера Службы контейнеров Azure (AKS) с узлами конфиденциальных вычислений (DCsv2) с помощью Azure CLI

Это краткое руководство предназначено для разработчиков и операторов кластера, которые хотят быстро создать кластер AKS и развернуть приложение для мониторинга приложений на основе управляемой службы Kubernetes в Azure. Также вы можете подготовить кластер и добавить узлы конфиденциальных вычислений на портале Azure.

## <a name="overview"></a>Обзор

В этом кратком руководстве описано, как развернуть кластер Службы контейнеров Azure (AKS) с узлами конфиденциальных вычислений, используя Azure CLI, и запустить в анклаве простейшее приложение Hello World. Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. Дополнительные сведения см. в статьях [Введение в AKS](../aks/intro-kubernetes.md) и [Обзор конфиденциальных узлов AKS](confidential-nodes-aks-overview.md).

> [!NOTE]
> Виртуальные машины DCsv2 для конфиденциальных вычислений работают на основе специализированного оборудования, которое стоит дороже обычного и доступно не во всех регионах. Дополнительные сведения см. на странице, посвященной [доступным ценовым категориям и поддерживаемым регионам](virtual-machine-solutions.md) для виртуальных машин.

### <a name="confidential-computing-node-features-dcsv2"></a>Функции узлов конфиденциальных вычислений (DCsv2)

1. Рабочие узлы Linux с поддержкой контейнеров Linux.
1. Виртуальная машина 2-го поколения с узлами виртуальных машин на основе Ubuntu 18.04.
1. ЦП на основе Intel SGX с поддержкой EPC (кэш в памяти для зашифрованных страниц). Дополнительные сведения см. [здесь](./faq.md).
1. Поддержка Kubernetes версии 1.16+.
1. Драйвер Intel SGX DCAP, предварительно установленный на узлах AKS. Дополнительные сведения см. [здесь](./faq.md).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством вам понадобится:

1. Активная подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
1. Установленное и настроенное на компьютере развертывания решение Azure CLI версии 2.0.64 или более поздней (выполните команду `az --version`, чтобы узнать версию). Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](../container-registry/container-registry-get-started-azure-cli.md).
1. Минимум шесть **ядер DCsv2** в вашей подписке. По умолчанию на каждую подписку Azure выделяется квота в восемь ядер виртуальных машин для конфиденциальных вычислений. Если вы планируете подготовить кластер, в котором будет более восьми ядер, выполните [эти](../azure-portal/supportability/per-vm-quota-requests.md) инструкции, чтобы создать запрос на увеличение квоты.

## <a name="create-a-new-aks-cluster-with-confidential-computing-nodes-and-add-on"></a>Создание нового кластера AKS с узлами конфиденциальных вычислений и надстройкой

Следуйте приведенным ниже инструкциям, чтобы добавить узлы с поддержкой конфиденциальных вычислений и надстройкой.

### <a name="create-an-aks-cluster-with-a-system-node-pool"></a>Создание кластера AKS с пулом системных узлов

Если у вас уже есть кластер AKS, который соответствует указанным выше требованиям, [перейдите к разделу для существующего кластера](#existing-cluster), чтобы добавить новый пул с узлами конфиденциальных вычислений.

Сначала создайте группу ресурсов для кластера, выполнив команду [az group create][az-group-create]. В следующем примере создается группа ресурсов *myResourceGroup* в регионе *westus2*:

```azurecli-interactive
az group create --name myResourceGroup --location westus2
```

Теперь выполните команду [az aks create][az-aks-create], чтобы создать кластер AKS:

```azurecli-interactive
az aks create -g myResourceGroup --name myAKSCluster --generate-ssh-keys --enable-addon confcom
```

Это действие создает новый кластер AKS с пулом системных узлов, для которых включена надстройка. Затем добавьте в кластер AKS пул узлов пользователя с возможностями конфиденциального вычисления.

### <a name="add-a-confidential-computing-node-pool-to-the-aks-cluster"></a>Добавление пула узлов конфиденциальных вычислений в кластер AKS 

Выполните следующую команду, чтобы добавить пул узлов пользователя размером `Standard_DC2s_v2` и тремя узлами. Вы можете выбрать другой SKU из списка поддерживаемых номеров [SKU и регионов DCsv2](../virtual-machines/dcv2-series.md).

```azurecli-interactive
az aks nodepool add --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup --node-vm-size Standard_DC2s_v2
```

После выполнения этой команды должен появиться пул узлов **DCsv2**, для которого установлен набор управляющих программ надстройки конфиденциальных вычислений ([подключаемый модуль устройства SGX](confidential-nodes-aks-overview.md#sgx-plugin)).

### <a name="verify-the-node-pool-and-add-on"></a>Проверка пула узлов и надстройки

Получите учетные данные для кластера AKS с помощью команды [az aks get-credentials][az-aks-get-credentials]:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Убедитесь, что узлы успешно созданы, а наборы управляющих программ, имеющие отношение к SGX, выполняются в пулах узлов **DCsv2** с помощью команд kubectl get pods и kubectl get nodes, как показано ниже:

```console
$ kubectl get pods --all-namespaces

kube-system     sgx-device-plugin-xxxx     1/1     Running
```

Если выходные данные совпадают с примером выше, значит кластер AKS готов к выполнению конфиденциальных приложений.

Перейдите к разделу о [развертывании приложения Hello World из анклава](#hello-world), чтобы протестировать работу приложения в анклаве. Или выполните приведенные ниже инструкции, чтобы добавить в AKS дополнительные пулы узлов (AKS поддерживает сочетание пулов узлов с SGX и без SGX).

## <a name="add-a-confidential-computing-node-pool-to-an-existing-aks-cluster"></a>Добавление пула узлов конфиденциальных вычислений в существующий кластер AKS<a id="existing-cluster"></a>

В этом разделе предполагается, что у вас уже есть работающий кластер AKS, который соответствует критериям из списка в разделе предварительных требований (в применении к надстройке).

### <a name="enable-the-confidential-computing-aks-add-on-on-the-existing-cluster"></a>Включение надстройки AKS для конфиденциальных вычислений в существующем кластере

Выполните приведенную ниже команду, чтобы включить надстройку конфиденциальных вычислений:

```azurecli-interactive
az aks enable-addons --addons confcom --name MyManagedCluster --resource-group MyResourceGroup 
```

### <a name="add-a-dcsv2-user-node-pool-to-the-cluster"></a>Теперь добавьте в кластер пул узлов пользователя **DCsv2**

> [!NOTE]
> Чтобы использовать возможность конфиденциальных вычислений, ваш существующий кластер AKS должен иметь по меньшей мере один пул узлов на основе SKU виртуальной машины **DCsv2**. Дополнительные сведения о номерах SKU виртуальных машин DC v2 для конфиденциальных вычислений см. в [этой статье](virtual-machine-solutions.md).

Чтобы создать пул узлов, выполните следующую команду:

```azurecli-interactive
az aks nodepool add --cluster-name myAKSCluster --name confcompool1 --resource-group myResourceGroup --node-count 1 --node-vm-size Standard_DC4s_v2
```

Проверьте, создан ли пул узлов с именем confcompool1:

```azurecli-interactive
az aks nodepool list --cluster-name myAKSCluster --resource-group myResourceGroup
```

### <a name="verify-that-daemonsets-are-running-on-confidential-node-pools"></a>Проверка работы набора управляющих программ в пулах узлов конфиденциальных вычислений

Войдите в имеющийся кластер AKS, чтобы выполнить проверку, описанную ниже.

```console
kubectl get nodes
```

Выходные данные должны подтвердить добавление confcompool1 в кластер AKS. Вы также можете увидеть другие наборы управляющих программ.

```console
$ kubectl get pods --all-namespaces

kube-system     sgx-device-plugin-xxxx     1/1     Running
```

Если выходные данные совпадают с примером выше, значит кластер AKS готов к выполнению конфиденциальных приложений. Чтобы развернуть пример приложения, выполните приведенные ниже инструкции.

## <a name="hello-world-from-isolated-enclave-application"></a>Выполнение приложения Hello World из изолированного анклава <a id="hello-world"></a>
Создайте файл с именем *hello-world-enclave.yaml* и вставьте в него приведенный ниже манифест YAML. Этот код примера приложения на основе Open Enclave можно найти в [проекте Open Enclave](https://github.com/openenclave/openenclave/tree/master/samples/helloworld). В приведенном ниже развертывании предполагается, что вы развернули надстройку confcom.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sgx-test
  labels:
    app: sgx-test
spec:
  template:
    metadata:
      labels:
        app: sgx-test
    spec:
      containers:
      - name: sgxtest
        image: oeciteam/sgx-test:1.0
        resources:
          limits:
            kubernetes.azure.com/sgx_epc_mem_in_MiB: 5 # This limit will automatically place the job into confidential computing node. Alternatively you can target deployment to nodepools
      restartPolicy: Never
  backoffLimit: 0
  ```

Теперь выполните команду kubectl apply, чтобы создать пример задания для выполнения в защищенном анклаве, как показано в следующем примере выходных данных:

```console
$ kubectl apply -f hello-world-enclave.yaml

job "sgx-test" created
```

Вы можете убедиться, что рабочая нагрузка успешно создала доверенную среду выполнения (анклав), выполнив следующие команды:

```console
$ kubectl get jobs -l app=sgx-test

NAME       COMPLETIONS   DURATION   AGE
sgx-test   1/1           1s         23s
```

```console
$ kubectl get pods -l app=sgx-test

NAME             READY   STATUS      RESTARTS   AGE
sgx-test-rchvg   0/1     Completed   0          25s
```

```console
$ kubectl logs -l app=sgx-test

Hello world from the enclave
Enclave called into host to print: Hello World!
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить кластер AKS или связанные с ним пулы узлов, выполните следующие команды:

### <a name="remove-the-confidential-computing-node-pool"></a>Удаление пула узлов конфиденциальных вычислений

```azurecli-interactive
az aks nodepool delete --cluster-name myAKSCluster --name myNodePoolName --resource-group myResourceGroup
```

### <a name="delete-the-aks-cluster"></a>Удаление кластера AKS

```azurecli-interactive
az aks delete --resource-group myResourceGroup --name myAKSCluster
```

## <a name="next-steps"></a>Дальнейшие действия

* Запустите приложения на Python, Node и т. п. конфиденциальным образом в конфиденциальных контейнерах, используя [эти примеры](https://github.com/Azure-Samples/confidential-container-samples).

* Запустите приложения, поддерживающие анклавы, используя [примеры контейнеров Azure с поддержкой анклава](https://github.com/Azure-Samples/confidential-computing/blob/main/containersamples/).

<!-- LINKS -->
[az-group-create]: /cli/azure/group#az_group_create
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials