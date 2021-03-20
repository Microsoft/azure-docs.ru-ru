---
title: Использование GPU в службе Azure Kubernetes (AKS)
description: Узнайте, как использовать GPU для ресурсоемких вычислений или интенсивных графических рабочих нагрузок в Службе Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 08/21/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: d7e312f049acc0b74aa0a253864bfce6100044bd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96929146"
---
# <a name="use-gpus-for-compute-intensive-workloads-on-azure-kubernetes-service-aks"></a>Использование процессоров GPU для интенсивных вычислительных рабочих нагрузок в Службе Azure Kubernetes (AKS)

Графические процессоры (GPU) обычно используются для ресурсоемких рабочих нагрузок, например графической обработки и визуализаций. AKS поддерживает создание пулов узлов с поддержкой GPU для выполнения этих ресурсоемких рабочих нагрузок в Kubernetes. Дополнительные сведения о доступных виртуальных машинах с поддержкой GPU см. в разделе [Размеры виртуальных машин, оптимизированных для GPU][gpu-skus]. Для узлов AKS мы рекомендуем использовать как минимум размер *Standard_NC6*.

> [!NOTE]
> Виртуальные машины с поддержкой GPU содержат специализированное оборудование, на которое предусмотрена более высокая цена и которое зависит от доступности в регионе. Дополнительные сведения можно получить с помощью [калькулятора цен][azure-pricing] и данных о [доступности в регионах][azure-availability].

В настоящее время использование пулов узлов с поддержкой GPU доступно только для пулов узлов Linux.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS с узлами, поддерживающими процессоры GPU. Кластер AKS должен работать на платформе Kubernetes 1.10 или более поздней версии. Если вам нужен кластер AKS, соответствующий этим требованиям, обратитесь к первой части этой статьи, чтобы [создать кластер AKS](#create-an-aks-cluster).

Также требуется Azure CLI версии 2.0.64 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

Если вам нужен кластер AKS, соответствующий минимальным требованиям (узел с поддержкой GPU и Kubernetes 1.10 или более поздней версии), выполните приведенные ниже действия. Если у вас уже есть кластер AKS, соответствующий этим требованиям, [перейдите к следующему разделу](#confirm-that-gpus-are-schedulable).

Сначала создайте группу ресурсов для кластера с помощью команды [AZ Group Create][az-group-create] . В следующем примере создается группа ресурсов *myResourceGroup* в регионе *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Теперь создайте кластер AKS с помощью команды [AZ AKS Create][az-aks-create] . В следующем примере создается кластер с одним узлом размером `Standard_NC6` :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-vm-size Standard_NC6 \
    --node-count 1
```

Получите учетные данные для кластера AKS с помощью команды [AZ AKS Get-Credential][az-aks-get-credentials] :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="install-nvidia-device-plugin"></a>Установка подключаемого модуля устройства NVIDIA

Прежде чем можно будет использовать GPU на узлах, необходимо развернуть управляющий набор для подключаемого модуля устройства NVIDIA. Этот DaemonSet запускает pod на каждом узле, чтобы предоставить необходимые драйверы для процессоров GPU.

Сначала с помощью команды [kubectl create namespace][kubectl-create] создайте пространство имен, например *gpu-resources*.

```console
kubectl create namespace gpu-resources
```

Создайте файл *nvidia-device-plugin-ds.yaml* и вставьте в него приведенный ниже манифест YAML. Этот манифест предоставляется как часть [подключаемого модуля устройства NVIDIA для проекта Kubernetes][nvidia-github].

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: gpu-resources
spec:
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
      # reserves resources for critical add-on pods so that they can be rescheduled after
      # a failure.  This annotation works in tandem with the toleration below.
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
      # This, along with the annotation above marks this pod as a critical add-on.
      - key: CriticalAddonsOnly
        operator: Exists
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      containers:
      - image: mcr.microsoft.com/oss/nvidia/k8s-device-plugin:1.11
        name: nvidia-device-plugin-ctr
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
```

Теперь используйте команду [kubectl Apply][kubectl-apply] , чтобы создать управляющую программу и убедиться, что подключаемый модуль устройства NVIDIA успешно создан, как показано в следующем примере выходных данных:

```console
$ kubectl apply -f nvidia-device-plugin-ds.yaml

daemonset "nvidia-device-plugin" created
```

## <a name="use-the-aks-specialized-gpu-image-preview"></a>Использование специализированного образа GPU AKS (Предварительная версия)

В качестве альтернативы этим шагам AKS предоставляет полностью настроенный образ AKS, который уже содержит [подключаемый модуль устройства NVIDIA для Kubernetes][nvidia-github].

> [!WARNING]
> Не следует вручную устанавливать набор управляющих программ подключаемого модуля устройства NVIDIA для кластеров с помощью нового специализированного образа GPU AKS.


Зарегистрируйте компонент `GPUDedicatedVHDPreview`:

```azurecli
az feature register --name GPUDedicatedVHDPreview --namespace Microsoft.ContainerService
```

Состояние **Registered** (Зарегистрировано) может появиться через несколько минут. Состояние регистрации можно проверить с помощью команды [az feature list](/cli/azure/feature#az-feature-list):

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/GPUDedicatedVHDPreview')].{Name:name,State:properties.state}"
```

Когда отобразится правильный статус, обновите регистрацию поставщика ресурсов `Microsoft.ContainerService` с помощью команды [az provider register](/cli/azure/provider#az-provider-register):

```azurecli
az provider register --namespace Microsoft.ContainerService
```

Чтобы установить расширение CLI AKS-Preview, используйте следующие Azure CLI команды:

```azurecli
az extension add --name aks-preview
```

Чтобы обновить расширение aks-preview, выполните следующие команды Azure CLI:

```azurecli
az extension update --name aks-preview
```

### <a name="use-the-aks-specialized-gpu-image-on-new-clusters-preview"></a>Использование специализированного образа GPU AKS в новых кластерах (Предварительная версия)    

Настройте кластер для использования специализированного образа GPU AKS при создании кластера. Используйте `--aks-custom-headers` флаг для узлов агента GPU в новом кластере, чтобы использовать специализированный образ GPU AKS.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true
```

Если вы хотите создать кластер с помощью обычных образов AKS, это можно сделать, опустив пользовательский `--aks-custom-headers` тег. Вы также можете добавить дополнительные специализированные пулы узлов GPU в соответствии с разрядом.


### <a name="use-the-aks-specialized-gpu-image-on-existing-clusters-preview"></a>Использование специализированного образа GPU AKS в существующих кластерах (Предварительная версия)

Настройте новый пул узлов для использования специализированного образа GPU AKS. Используйте `--aks-custom-headers` флаг флага для узлов агента GPU в новом пуле узлов, чтобы использовать специализированный образ GPU AKS.

```azurecli
az aks nodepool add --name gpu --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true
```

Если вы хотите создать пул узлов с помощью обычных образов AKS, это можно сделать, опустив пользовательский `--aks-custom-headers` тег. 

> [!NOTE]
> Если номер SKU GPU требует виртуальных машин поколения 2, можно создать:
> ```azurecli
> az aks nodepool add --name gpu --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6s_v2 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true,usegen2vm=true
> ```

## <a name="confirm-that-gpus-are-schedulable"></a>Подтверждение того, что процессоры GPU доступны для планирования

Создав кластер AKS, убедитесь, что процессоры GPU доступны для планирования в Kubernetes. Сначала получите список узлов в кластере с помощью команды [kubectl get nodes][kubectl-get].

```console
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-28993262-0   Ready    agent   13m   v1.12.7
```

Теперь с помощью команды [kubectl describe node][kubectl-describe] подтвердите, что процессоры GPU доступны для планирования. В разделе *Емкость* PU должен отображаться как `nvidia.com/gpu:  1`.

В следующем сокращенном примере показано, что GPU доступен на узле *aks-nodepool1-18821093-0*.

```console
$ kubectl describe node aks-nodepool1-28993262-0

Name:               aks-nodepool1-28993262-0
Roles:              agent
Labels:             accelerator=nvidia

[...]

Capacity:
 attachable-volumes-azure-disk:  24
 cpu:                            6
 ephemeral-storage:              101584140Ki
 hugepages-1Gi:                  0
 hugepages-2Mi:                  0
 memory:                         57713784Ki
 nvidia.com/gpu:                 1
 pods:                           110
Allocatable:
 attachable-volumes-azure-disk:  24
 cpu:                            5916m
 ephemeral-storage:              93619943269
 hugepages-1Gi:                  0
 hugepages-2Mi:                  0
 memory:                         51702904Ki
 nvidia.com/gpu:                 1
 pods:                           110
System Info:
 Machine ID:                 b0cd6fb49ffe4900b56ac8df2eaa0376
 System UUID:                486A1C08-C459-6F43-AD6B-E9CD0F8AEC17
 Boot ID:                    f134525f-385d-4b4e-89b8-989f3abb490b
 Kernel Version:             4.15.0-1040-azure
 OS Image:                   Ubuntu 16.04.6 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://1.13.1
 Kubelet Version:            v1.12.7
 Kube-Proxy Version:         v1.12.7
PodCIDR:                     10.244.0.0/24
ProviderID:                  azure:///subscriptions/<guid>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Compute/virtualMachines/aks-nodepool1-28993262-0
Non-terminated Pods:         (9 in total)
  Namespace                  Name                                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                     ------------  ----------  ---------------  -------------  ---
  kube-system                nvidia-device-plugin-daemonset-bbjlq     0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m39s

[...]
```

## <a name="run-a-gpu-enabled-workload"></a>Запуск рабочей нагрузки с поддержкой GPU

Чтобы увидеть, как используется GPU, запланируйте рабочую нагрузку с поддержкой GPU с помощью соответствующего запроса ресурса. В этом примере давайте выполним задание [Tensorflow](https://www.tensorflow.org/) с [набором данных MNIST](http://yann.lecun.com/exdb/mnist/).

Создайте файл *samples-tf-mnist-demo.yaml* и вставьте в него приведенный ниже манифест YAML. Следующий манифест задания включает в себя ограничение ресурсов: `nvidia.com/gpu: 1`.

> [!NOTE]
> Если при вызове драйверов возникла ошибка несоответствия версий, например версия драйвера CUDA недостаточна для версии среды выполнения CUDA, см. диаграмму совместимости с матричным драйвером NVIDIA. [https://docs.nvidia.com/deploy/cuda-compatibility/index.html](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: samples-tf-mnist-demo
  name: samples-tf-mnist-demo
spec:
  template:
    metadata:
      labels:
        app: samples-tf-mnist-demo
    spec:
      containers:
      - name: samples-tf-mnist-demo
        image: mcr.microsoft.com/azuredocs/samples-tf-mnist-demo:gpu
        args: ["--max_steps", "500"]
        imagePullPolicy: IfNotPresent
        resources:
          limits:
           nvidia.com/gpu: 1
      restartPolicy: OnFailure
```

Используйте команду [kubectl apply][kubectl-apply], чтобы запустить задание. Эта команда анализирует файл манифеста и создает заданные объекты Kubernetes.

```console
kubectl apply -f samples-tf-mnist-demo.yaml
```

## <a name="view-the-status-and-output-of-the-gpu-enabled-workload"></a>Просмотр состояния и выходных данных рабочей нагрузки с поддержкой GPU

Отслеживайте ход выполнения задания с помощью команды [kubectl get jobs][kubectl-get] с аргументом `--watch`. Извлечение изображения и последующая обработка набора данных может занять несколько минут. Когда в столбце *завершения* отображается *1/1*, задание успешно завершено. Выйдите из `kubetctl --watch` команды с помощью *клавиш CTRL-C*:

```console
$ kubectl get jobs samples-tf-mnist-demo --watch

NAME                    COMPLETIONS   DURATION   AGE

samples-tf-mnist-demo   0/1           3m29s      3m29s
samples-tf-mnist-demo   1/1   3m10s   3m36s
```

Чтобы просмотреть выходные данные рабочей нагрузки с поддержкой GPU, сначала получите имя Pod с помощью команды [kubectl Get][kubectl-get] Pod:

```console
$ kubectl get pods --selector app=samples-tf-mnist-demo

NAME                          READY   STATUS      RESTARTS   AGE
samples-tf-mnist-demo-mtd44   0/1     Completed   0          4m39s
```

Теперь выполните команду [kubectl logs][kubectl-logs], чтобы просмотреть журналы pod. В следующем примере журналов pod указано, что обнаружено соответствующее устройство GPU, `Tesla K80`. Укажите имя собственного pod.

```console
$ kubectl logs samples-tf-mnist-demo-smnr6

2019-05-16 16:08:31.258328: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2019-05-16 16:08:31.396846: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties: 
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: 2fd7:00:00.0
totalMemory: 11.17GiB freeMemory: 11.10GiB
2019-05-16 16:08:31.396886: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: 2fd7:00:00.0, compute capability: 3.7)
2019-05-16 16:08:36.076962: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.1081
Accuracy at step 10: 0.7457
Accuracy at step 20: 0.8233
Accuracy at step 30: 0.8644
Accuracy at step 40: 0.8848
Accuracy at step 50: 0.8889
Accuracy at step 60: 0.8898
Accuracy at step 70: 0.8979
Accuracy at step 80: 0.9087
Accuracy at step 90: 0.9099
Adding run metadata for 99
Accuracy at step 100: 0.9125
Accuracy at step 110: 0.9184
Accuracy at step 120: 0.922
Accuracy at step 130: 0.9161
Accuracy at step 140: 0.9219
Accuracy at step 150: 0.9151
Accuracy at step 160: 0.9199
Accuracy at step 170: 0.9305
Accuracy at step 180: 0.9251
Accuracy at step 190: 0.9258
Adding run metadata for 199
Accuracy at step 200: 0.9315
Accuracy at step 210: 0.9361
Accuracy at step 220: 0.9357
Accuracy at step 230: 0.9392
Accuracy at step 240: 0.9387
Accuracy at step 250: 0.9401
Accuracy at step 260: 0.9398
Accuracy at step 270: 0.9407
Accuracy at step 280: 0.9434
Accuracy at step 290: 0.9447
Adding run metadata for 299
Accuracy at step 300: 0.9463
Accuracy at step 310: 0.943
Accuracy at step 320: 0.9439
Accuracy at step 330: 0.943
Accuracy at step 340: 0.9457
Accuracy at step 350: 0.9497
Accuracy at step 360: 0.9481
Accuracy at step 370: 0.9466
Accuracy at step 380: 0.9514
Accuracy at step 390: 0.948
Adding run metadata for 399
Accuracy at step 400: 0.9469
Accuracy at step 410: 0.9489
Accuracy at step 420: 0.9529
Accuracy at step 430: 0.9507
Accuracy at step 440: 0.9504
Accuracy at step 450: 0.951
Accuracy at step 460: 0.9512
Accuracy at step 470: 0.9539
Accuracy at step 480: 0.9533
Accuracy at step 490: 0.9494
Adding run metadata for 499
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить связанные объекты Kubernetes, созданные в этой статье, выполните команду [kubectl delete job][kubectl delete], как показано ниже.

```console
kubectl delete jobs samples-tf-mnist-demo
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы выполнять задания Apache Spark, ознакомьтесь с разделом [Запуск заданий Apache Spark в AKS][aks-spark].

Чтобы получить дополнительные сведения о выполнении рабочих нагрузок машинного обучения машины в Kubernetes, ознакомьтесь с [лабораториями Kubeflow][kubeflow-labs].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeflow-labs]: https://github.com/Azure/kubeflow-labs
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[kubectl delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[azure-pricing]: https://azure.microsoft.com/pricing/
[azure-availability]: https://azure.microsoft.com/global-infrastructure/services/
[nvidia-github]: https://github.com/NVIDIA/k8s-device-plugin

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[aks-spark]: spark-job.md
[gpu-skus]: ../virtual-machines/sizes-gpu.md
[install-azure-cli]: /cli/azure/install-azure-cli
