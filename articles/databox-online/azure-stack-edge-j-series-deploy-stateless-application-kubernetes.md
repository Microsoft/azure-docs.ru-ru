---
title: Развертывание приложения без отслеживания состояния Kubernetes на устройстве с Azure Stack ребра Pro GPU с помощью kubectl | Документация Майкрософт
description: Описание процесса создания развертывания приложений без отслеживания состояния Kubernetes и управления им с помощью kubectl на устройстве Pro Microsoft Azure Stack.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/05/2021
ms.author: alkohli
ms.openlocfilehash: 40065189f3c1dd52f462985aef344e9cbeb12f0b
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102522382"
---
# <a name="deploy-a-kubernetes-stateless-application-via-kubectl-on-your-azure-stack-edge-pro-gpu-device"></a>Развертывание Kubernetes приложения без отслеживания состояния с помощью kubectl на устройстве Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как развернуть приложение без отслеживания состояния с помощью команд kubectl в существующем кластере Kubernetes. В этой статье также описывается процесс создания и настройки модулей Pod в приложении без отслеживания состояния.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем можно будет создать кластер Kubernetes и использовать `kubectl` программу командной строки, необходимо убедиться в том, что:

- У вас есть учетные данные для входа на 1 узел Azure Stack пограничным устройством Pro.

- Windows PowerShell 5,0 или более поздней версии устанавливается в клиентской системе Windows для доступа к устройству Azure Stack погранично Pro. Также можно использовать любой другой клиент с поддерживаемой операционной системой. В этой статье описывается процедура использования клиента Windows. Чтобы скачать последнюю версию Windows PowerShell, перейдите к разделу [Установка Windows PowerShell](/powershell/scripting/install/installing-windows-powershell).

- На устройстве Azure Stack ребра Pro включено вычисление. Чтобы включить вычисление, перейдите на страницу " **Вычисление** " в локальном пользовательском интерфейсе устройства. Затем выберите сетевой интерфейс, который требуется включить для вычислений. Нажмите кнопку **Включить**. Включение вычислений приведет к созданию виртуального коммутатора на устройстве в этом сетевом интерфейсе. Дополнительные сведения см. [в статье Включение вычислений в сети на Azure Stack пограничных Pro](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).

- На устройстве Azure Stack пограничной Pro работает сервер кластера Kubernetes версии 1,9 или более поздней. Дополнительные сведения см. в статье [Создание кластера Kubernetes и управление им на Microsoft Azure Stack пограничном устройстве Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).

- Вы установили `kubectl` .

## <a name="deploy-a-stateless-application"></a>Развертывание приложения без отслеживания состояния

Прежде чем начать, необходимо:

1. Создал кластер Kubernetes.
2. Настройте пространство имен.
3. Связан с пространством имен пользователя.
4. Пользовательская конфигурация сохранена в `C:\Users\<username>\.kube` .
5. Установлено `kubectl` .

Теперь вы можете начать работу и управлять развертываниями приложений без отслеживания состояния на устройстве Azure Stack пограничной Pro. Перед началом использования необходимо `kubectl` убедиться, что у вас установлена правильная версия `kubectl` .

### <a name="verify-you-have-the-correct-version-of-kubectl-and-set-up-configuration"></a>Убедитесь, что у вас установлена правильная версия kubectl и настройка конфигурации.

Чтобы проверить версию `kubectl` :

1. Убедитесь, что версия `kubectl` больше или равна 1,9:

   ```powershell
   kubectl version
   ```
    
   Ниже приведен пример выходных данных.
    
   ```powershell
   PS C:\WINDOWS\system32> C:\windows\system32\kubectl.exe version
   Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
   Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
   ```

   В этом случае клиентская версия kubectl — v 1.15.2 и совместима для продолжения.

2. Получите список модулей Pod, выполняющихся в кластере Kubernetes. Pod — это контейнер приложения или процесс, выполняющийся в кластере Kubernetes.

   ```powershell
   kubectl get pods -n <namespace-string>
   ```
    
   Ниже приведен пример использования команды.
    
   ```powershell
   PS C:\WINDOWS\system32> kubectl get pods -n "test1"
   No resources found.
   PS C:\WINDOWS\system32>
   ```
    
   В выходных данных должно быть состояние, что ресурсы (Pod) не найдены, так как в вашем кластере нет приложений, выполняющихся на вашем компьютере.

   Команда заполнит структуру каталогов "C:\Users \\ &lt; username &gt; \\ . KUBE \" файлами конфигурации. Программа командной строки kubectl будет использовать эти файлы для создания и управления приложениями без отслеживания состояния в кластере Kubernetes.

3. Вручную проверьте структуру каталогов "C:\Users \\ &lt; username &gt; \\ . KUBE \" , чтобы убедиться, что *kubectl* заполнены следующими вложенными папками:

   ```powershell
   PS C:\Users\username> ls .kube
    
    
      Directory: C:\Users\user\.kube
    
   Mode                LastWriteTime         Length Name
   ----                -------------         ------ ----
   d-----         2/18/2020 11:05 AM                cache
   d-----         2/18/2020 11:04 AM                http-cache
   -a----         2/18/2020 10:41 AM           5377 config
   ```

> [!NOTE]
> Чтобы просмотреть список всех команд kubectl, введите `kubectl --help` .

### <a name="create-a-stateless-application-using-a-deployment"></a>Создание приложения без отслеживания состояния с помощью развертывания

Теперь, когда вы проверили правильность версии командной строки kubectl и у вас есть необходимые файлы конфигурации, можно создать развертывание приложения без отслеживания состояния.

Pod — это основная единица выполнения приложения Kubernetes, самая маленькая и простейшая единица в объектной модели Kubernetes, которую вы создаете или развертываете. Pod также инкапсулирует ресурсы хранилища, уникальный IP-адрес сети и параметры, определяющие, как должны выполняться контейнеры.

Тип создаваемого приложения без отслеживания состояния является развертыванием веб-сервера nginx.

Все команды kubectl, используемые для создания и управления развертываниями приложений без отслеживания состояния, должны указывать пространство имен, связанное с конфигурацией. Пространство имен создано при подключении к кластеру на устройстве Azure Stack пограничном Pro в учебнике [Создание и управление кластером Kubernetes на Microsoft Azure Stack пограничных устройствах Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md) с помощью `New-HcsKubernetesNamespace` .

Чтобы указать пространство имен в команде kubectl, используйте `kubectl <command> -n <namespace-string>` .

Чтобы создать развертывание nginx, выполните следующие действия.

1. Применение приложения без отслеживания состояния путем создания объекта развертывания Kubernetes:

   ```powershell
   kubectl apply -f <yaml-file> -n <namespace-string>
   ```

   В этом примере путь к файлу приложения YAML является внешним источником.

   Ниже приведен пример использования команды и ее выходных данных:

   ```powershell
   PS C:\WINDOWS\system32> kubectl apply -f https://k8s.io/examples/application/deployment.yaml -n "test1"
    
   deployment.apps/nginx-deployment created
   ```

   Кроме того, можно сохранить следующий Markdown на локальном компьютере и заменить путь и имя файла в параметре *-f* . Например, "К:\кубернетес\деплоймент.ямл". Конфигурация для развертывания приложения будет выглядеть следующим образом:

   ```markdown
   apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     selector:
       matchLabels:
         app: nginx
     replicas: 2 # tells deployment to run 2 pods matching the template
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:1.7.9
           ports:
           - containerPort: 80
   ```

   Эта команда создает nginx развертывание по умолчанию с двумя модулями Pod для запуска приложения.

2. Получите описание Kubernetes nginx-Deployment, созданного вами:

   ```powershell
   kubectl describe deployment nginx-deployment -n <namespace-string>
   ```

   Ниже приведен пример использования команды с OUTPUT.
    
   ```powershell
   PS C:\Users\user> kubectl describe deployment nginx-deployment -n "test1"
    
   Name:                   nginx-deployment
   Namespace:              test1
   CreationTimestamp:      Tue, 18 Feb 2020 13:35:29 -0800
   Labels:                 <none>
   Annotations:            deployment.kubernetes.io/revision: 1
                           kubectl.kubernetes.io/last-applied-configuration:
                             {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment","namespace":"test1"},"spec":{"repl...
   Selector:               app=nginx
   Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   StrategyType:           RollingUpdate
   MinReadySeconds:        0
   RollingUpdateStrategy:  25% max unavailable, 25% max surge
   Pod Template:
      Labels:  app=nginx
      Containers:
       nginx:
        Image:        nginx:1.7.9
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
   Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
   OldReplicaSets:  <none>
   NewReplicaSet:   nginx-deployment-5754944d6c (2/2 replicas created)
   Events:
     Type    Reason             Age    From                   Message
     ----    ------             ----   ----                   -------
     Normal  ScalingReplicaSet  2m22s  deployment-controller  Scaled up replica set nginx-deployment-5754944d6c to 2
   ```

   Для параметра *реплики* вы увидите:
    
   ```powershell
   Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
   ```

   Параметр *реплики* указывает, что спецификация развертывания требует наличия двух модулей Pod, и что эти модули созданы и обновлены и готовы к использованию.

   > [!NOTE]
   > Набор реплик заменяет удаляемые или завершенные модули Pod по какой-либо причине, например в случае сбоя узла устройства или аварийного обновления устройства. По этой причине рекомендуется использовать набор реплик, даже если приложению требуется только один модуль.

3. Чтобы получить список модулей Pod в развертывании, выполните следующие действия.

   ```powershell
   kubectl get pods -l app=nginx -n <namespace-string>
   ```
    
   Ниже приведен пример использования команды с OUTPUT.
    
   ```powershell
   PS C:\Users\user> kubectl get pods -l app=nginx -n "test1"
    
   NAME                                READY   STATUS    RESTARTS   AGE
   nginx-deployment-5754944d6c-7wqjd   1/1     Running   0          3m13s
   nginx-deployment-5754944d6c-nfj2h   1/1     Running   0          3m13s
   ```

   В выходных данных проверяется наличие двух модулей Pod с уникальными именами, на которые можно сослаться с помощью kubectl.

4. Чтобы просмотреть сведения об отдельном Pod в развертывании, выполните следующие действия.

   ```powershell
   kubectl describe pod <podname-string> -n <namespace-string>
   ```

  Ниже приведен пример использования команды с OUTPUT.

   ```powershell
   PS C:\Users\user> kubectl describe pod "nginx-deployment-5754944d6c-7wqjd" -n "test1"

   Name:           nginx-deployment-5754944d6c-7wqjd
   Namespace:      test1
   Priority:       0
   Node:           k8s-1d9qhq2cl-n1/10.128.46.184
   Start Time:     Tue, 18 Feb 2020 13:35:29 -0800
   Labels:         app=nginx
                   pod-template-hash=5754944d6c
   Annotations:    <none>
   Status:         Running
   IP:             172.17.246.200
   Controlled By:  ReplicaSet/nginx-deployment-5754944d6c
    Containers:
      nginx:
        Container ID:   docker://280b0f76bfdc14cde481dc4f2b8180cf5fbfc90a084042f679d499f863c66979
        Image:          nginx:1.7.9
        Image ID:       docker-pullable://nginx@sha256:e3456c851a152494c3e4ff5fcc26f240206abac0c9d794affb40e0714846c451
        Port:           80/TCP
        Host Port:      0/TCP
        State:          Running
          Started:      Tue, 18 Feb 2020 13:35:35 -0800
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-8gksw (ro)
    Conditions:
      Type              Status
      Initialized       True
      Ready             True
      ContainersReady   True
      PodScheduled      True
    Volumes:
      default-token-8gksw:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-8gksw
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  <none>
    Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                     node.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type    Reason     Age    From                       Message
      ----    ------     ----   ----                       -------
      Normal  Scheduled  4m58s  default-scheduler          Successfully assigned test1/nginx-deployment-5754944d6c-7wqjd to k8s-1d9qhq2cl-n1
      Normal  Pulling    4m57s  kubelet, k8s-1d9qhq2cl-n1  Pulling image "nginx:1.7.9"
      Normal  Pulled     4m52s  kubelet, k8s-1d9qhq2cl-n1  Successfully pulled image "nginx:1.7.9"
      Normal  Created    4m52s  kubelet, k8s-1d9qhq2cl-n1  Created container nginx
      Normal  Started    4m52s  kubelet, k8s-1d9qhq2cl-n1  Started container nginx
   ```

### <a name="rescale-the-application-deployment-by-increasing-the-replica-count"></a>Масштабирование развертывания приложения путем увеличения числа реплик

Каждый модуль Pod предназначен для запуска одного экземпляра данного приложения. Если требуется масштабировать приложение по горизонтали для запуска нескольких экземпляров, можно увеличить число модулей Pod на единицу для каждого экземпляра. В Kubernetes это называется репликацией.
Вы можете увеличить число модулей Pod в развертывании приложения, применив новый файл YAML. Файл YAML изменяет значение параметра реплик на 4, что увеличивает число модулей Pod в развертывании до четырех модулей. Чтобы увеличить число модулей Pod с 2 до 4:

```powershell
PS C:\WINDOWS\system32> kubectl apply -f https://k8s.io/examples/application/deployment-scale.yaml -n "test1"
```

Кроме того, можно сохранить следующий Markdown на локальном компьютере и заменить путь и имя файла для параметра *-f* для `kubectl apply` . Например, "К:\кубернетес\деплоймент-скале.ямл". Конфигурация для масштабирования развертывания приложения будет выглядеть следующим образом:

```markdown
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
```

Чтобы убедиться, что развертывание состоит из четырех модулей:

```powershell
kubectl get pods -l app=nginx
```

Ниже приведен пример выходных данных для развертывания повторного масштабирования из двух до четырех модулей Pod:

```powershell
PS C:\WINDOWS\system32> kubectl get pods -l app=nginx

NAME                               READY     STATUS    RESTARTS   AGE
nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
nginx-deployment-148880595-fxcez   1/1       Running   0          2m
nginx-deployment-148880595-rwovn   1/1       Running   0          2m
```

Как видно из выходных данных, теперь в развертывании есть четыре модуля Pod, которые могут запускать приложение.

### <a name="delete-a-deployment"></a>Удаление развертывания

Чтобы удалить развертывание, включая все модули Pod, необходимо выполнить `kubectl delete deployment` Указание имени развертывания *nginx-Deployment* и имени пространства имен. Чтобы удалить развертывание, выполните следующие действия.

   ```powershell
   kubectl delete deployment nginx-deployment -n <namespace-string>
   ```

Ниже приведен пример использования команды с OUTPUT.

```powershell
PS C:\Users\user> kubectl delete deployment nginx-deployment -n "test1"
deployment.extensions "nginx-deployment" deleted
```

## <a name="next-steps"></a>Дальнейшие действия

[Обзор Kubernetes](azure-stack-edge-gpu-kubernetes-overview.md)