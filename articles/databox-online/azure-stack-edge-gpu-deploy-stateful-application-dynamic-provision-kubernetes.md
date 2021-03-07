---
title: Использование kubectl для развертывания приложения с отслеживанием состояния Kubernetes с помощью динамически подготовленного общего ресурса на устройстве Azure Stack ребра Pro GPU | Документация Майкрософт
description: Описывает создание Kubernetes развертывания приложений с отслеживанием состояния с помощью динамически подготовленного общего ресурса с использованием kubectl на устройстве Microsoft Azure Stack с ГРАФИЧЕСКИм стандартом Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: ea316205e7dfb4069c41987e785881512735a4d6
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102438036"
---
# <a name="use-kubectl-to-run-a-kubernetes-stateful-application-with-storageclass-on-your-azure-stack-edge-pro-gpu-device"></a>Использование kubectl для запуска приложения Kubernetes с отслеживанием состояния с помощью Сторажекласс на устройстве Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье показано, как развернуть приложение с отслеживанием состояния с одним экземпляром в Kubernetes с помощью Сторажекласс, чтобы динамически подготавливать хранилище и развертывание. Развертывание использует `kubectl` команды в существующем кластере Kubernetes и развертывает приложение MySQL. 

Эта процедура предназначена для тех, кто проверил [хранилище Kubernetes на устройстве с Azure Stack пограничным Pro](azure-stack-edge-gpu-kubernetes-storage.md) и знаком с концепциями [хранилища Kubernetes](https://kubernetes.io/docs/concepts/storage/).


## <a name="prerequisites"></a>Предварительные требования

Перед развертыванием приложения с отслеживанием состояния выполните следующие предварительные требования на устройстве и клиенте, которые будут использоваться для доступа к устройству.

### <a name="for-device"></a>Для устройств

- У вас есть учетные данные для входа на 1 узел Azure Stack пограничным устройством Pro.
    - Устройство активировано. См. раздел [Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
    - Устройство имеет роль вычислений, настроенную через портал Azure и имеющую кластер Kubernetes. См. раздел [Настройка вычислений](azure-stack-edge-gpu-deploy-configure-compute.md).

### <a name="for-client-accessing-the-device"></a>Для клиента, обращающегося к устройству

- У вас есть клиентская система Windows, которая будет использоваться для доступа к устройству Azure Stack погранично Pro.
    - Клиент работает под управлением Windows PowerShell 5,0 или более поздней версии. Чтобы скачать последнюю версию Windows PowerShell, перейдите к разделу [Установка Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-7&preserve-view=true).
    
    - Также можно использовать любой другой клиент с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device) . В этой статье описывается процедура использования клиента Windows. 
    
    - Вы выполнили процедуру, описанную в статье [доступ к кластеру Kubernetes на устройстве с Azure Stack ребр Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Вы выполнили следующие задачи:
      - Создание `userns1` пространства имен с помощью `New-HcsKubernetesNamespace` команды. 
      - Создание пользователя с `user1` помощью `New-HcsKubernetesUser` команды. 
      - Предоставлен `user1` доступ к `userns1` через `Grant-HcsKubernetesNamespaceAccess` команду.       
      - Установлен `kubectl` на клиенте и сохранил `kubeconfig` файл с пользовательской конфигурацией в C: \\ Users \\ &lt; &gt; \\ . KUBE. 
    
    - Убедитесь, что `kubectl` версия клиента отклонена, но не имеет более одной версии из главной версии Kubernetes, работающей на устройстве Azure Stack погранично Pro. 
        - Используйте `kubectl version` для проверки версии kubectl, работающей на клиенте. Запишите полную версию.
        - В локальном пользовательском интерфейсе устройства Azure Stack ребра Pro перейдите к **обзору** и запишите номер Kubernetes Software. 
        - Проверьте эти две версии на совместимость с сопоставлением, указанным в поддерживаемой версии Kubernetes.<!-- insert link-->. 


Вы готовы к развертыванию приложения с отслеживанием состояния на устройстве Azure Stack ребра Pro. 


## <a name="deploy-mysql"></a>Развертывание MySQL

Теперь вы запустите приложение с отслеживанием состояния, создав развертывание Kubernetes и подключив его к встроенной Сторажекласс с помощью Персистентволумеклаим (PVC). 

Все `kubectl` команды, используемые для создания развертываний приложений с отслеживанием состояния и управления ими, должны указывать пространство имен, связанное с конфигурацией. Чтобы указать пространство имен в команде kubectl, используйте `kubectl <command> -n <your-namespace>` .

1. Получите список модулей Pod, выполняющихся в кластере Kubernetes в пространстве имен. Pod — это контейнер приложения или процесс, выполняющийся в кластере Kubernetes.

   ```powershell
   kubectl get pods -n <your-namespace>
   ```
    
   Ниже приведен пример применения этой команды:
    
   ```powershell
    C:\Users\user>kubectl get pods -n "userns1"
    No resources found in userns1 namespace.    
    C:\Users\user>
   ```
    
   В выходных данных должно быть состояние, что ресурсы (Pod) не найдены, так как в вашем кластере нет приложений, выполняющихся на вашем компьютере.

1. Вы будете использовать следующие файлы YAML. В этом `mysql-deployment.yml` файле описывается развертывание, которое запускает MySQL и ссылается на PVC. Файл определяет подключение тома для `/var/lib/mysql` , а затем создает PVC, который выполняет поиск тома размером 20 ГБ. Динамический ПС подготавливается, а PVC привязан к этой ПС.

    Скопируйте и сохраните следующий `mysql-deployment.yml` файл в папке клиента Windows, который используется для доступа к устройству Azure Stack погранично Pro.
    
    ```yml
    apiVersion: v1
    kind: Service
    metadata:
      name: mysql
    spec:
      ports:
      - port: 3306
      selector:
        app: mysql
      clusterIP: None
    ---
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: mysql
    spec:
      selector:
        matchLabels:
          app: mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - image: mysql:5.6
            name: mysql
            env:
              # Use secret in real usage
            - name: MYSQL_ROOT_PASSWORD
              value: password
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
          volumes:
          - name: mysql-persistent-storage
            persistentVolumeClaim:
              claimName: mysql-pv-claim-sc
    ```
    
2. Скопируйте и сохраните в виде `mysql-pvc.yml` файла в той же папке, где был сохранен `mysql-deployment.yml` . Чтобы использовать встроенные Сторажекласс, Azure Stack Device Pro на подключенном диске данных, задайте `storageClassName` для поля в ОБЪЕКТЕ PVC значение `ase-node-local` и акцессмодес `ReadWriteOnce` . 

    > [!NOTE] 
    > Убедитесь, что YAML файлы имеют правильный отступ. Проверить и сохранить можно с помощью [YAML Lint](http://www.yamllint.com/) .
   
    ```yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim-sc
    spec:
      storageClassName: ase-node-local
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

3. Разверните `mysql-pvc.yaml` файл.

    `kubectl apply -f <URI path to the mysql-pv.yml file> -n <your-user-namespace>`
    
    Ниже приведен пример выходных данных развертывания.

    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-pvc.yml" -n userns1
    persistentvolumeclaim/mysql-pv-claim-sc created
    C:\Users\user>
    ```
   Запишите имя созданной PVC, в этом примере — `mysql-pv-claim-sc` . Вы будете использовать его на более позднем этапе.

4. Разверните содержимое `mysql-deployment.yml` файла.

    `kubectl apply -f <URI path to mysql-deployment.yml file> -n <your-user-namespace>`

    Ниже приведен пример выходных данных развертывания.
    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-deployment.yml" -n userns1
    service/mysql created
    deployment.apps/mysql created
    C:\Users\user>
    ```
    
5. Отображает сведения о развертывании.

    `kubectl describe deployment <app-label> -n <your-user-namespace>`
    
    ```powershell
    C:\Users\user>kubectl describe deployment mysql -n userns1
    Name:               mysql
    Namespace:          userns1
    CreationTimestamp:  Thu, 20 Aug 2020 11:14:25 -0700
    Labels:             <none>
    Annotations:        deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"mysql","namespace":"userns1"},"spec":{"selector":{"matchL...
    Selector:           app=mysql
    Replicas:           1 desired | 1 updated | 1 total | 1 available | 0 unavailable
    StrategyType:       Recreate
    MinReadySeconds:    0
    Pod Template:
      Labels:  app=mysql
      Containers:
       mysql:
        Image:      mysql:5.6
        Port:       3306/TCP
        Host Port:  0/TCP
        Environment:
          MYSQL_ROOT_PASSWORD:  password
        Mounts:
          /var/lib/mysql from mysql-persistent-storage (rw)
      Volumes:
       mysql-persistent-storage:
        Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName:  mysql-pv-claim-sc
        ReadOnly:   false
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-695c4d9dcd (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set mysql-695c4d9dcd to 1
    C:\Users\user>
    ```
    

6. Список модулей Pod, созданных при развертывании.

    `kubectl get pods -l <app=label> -n <your-user-namespace>`

    Ниже приведен пример выходных данных.

    
    ```powershell
    C:\Users\user>kubectl get pods -l app=mysql -n userns1
    NAME                     READY   STATUS    RESTARTS   AGE
    mysql-695c4d9dcd-rvzff   1/1     Running   0          40s
    C:\Users\user>
    ```
    
7. Проверьте Персистентволумеклаим.

    `kubectl describe pvc <your-pvc-name>`

    Ниже приведен пример выходных данных.

    
    ```powershell
    C:\Users\user>kubectl describe pvc mysql-pv-claim-sc -n userns1
    Name:          mysql-pv-claim-sc
    Namespace:     userns1
    StorageClass:  ase-node-local
    Status:        Bound
    Volume:        pvc-dc48253c-82dc-42a4-a7c6-aaddc97c9b8a
    Labels:        <none>
    Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"mysql-pv-claim-sc","namespace":"userns1"},"spec":{"...
                   pv.kubernetes.io/bind-completed: yes
                   pv.kubernetes.io/bound-by-controller: yes
                   volume.beta.kubernetes.io/storage-provisioner: rancher.io/local-path
                   volume.kubernetes.io/selected-node: k8s-3q7lhq2cl-3q7lhq2
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      20Gi
    Access Modes:  RWO
    VolumeMode:    Filesystem
    Mounted By:    mysql-695c4d9dcd-rvzff
    Events:
      Type    Reason                 Age                From                                                                                                Message
      ----    ------                 ----               ----                                                                                                -------
      Normal  WaitForFirstConsumer   71s (x2 over 77s)  persistentvolume-controller                                                                         waiting for first consumer to be created before binding
      Normal  ExternalProvisioning   62s                persistentvolume-controller                                                                         waiting for a volume to be created, either by external provisioner "rancher.io/local-path" or manually created by system administrator
      Normal  Provisioning           62s                rancher.io/local-path_local-path-provisioner-6b84988bf9-tx8mz_1896d824-f862-4cbf-912a-c8cc0ca05574  External provisioner is provisioning volume for claim "userns1/mysql-pv-claim-sc"
      Normal  ProvisioningSucceeded  60s                rancher.io/local-path_local-path-provisioner-6b84988bf9-tx8mz_1896d824-f862-4cbf-912a-c8cc0ca05574  Successfully provisioned volume pvc-dc48253c-82dc-42a4-a7c6-aaddc97c9b8a
    C:\Users\user>
    ```
    

## <a name="verify-mysql-is-running"></a>Проверка выполнения MySQL

Чтобы убедиться, что приложение выполняется, введите:

`kubectl exec <your-pod-with-the-app> -i -t -n <your-namespace> -- mysql -p`

При появлении запроса введите пароль. Пароль находится в `mysql-deployment` файле.

Ниже приведен пример выходных данных.

```powershell
C:\Users\user>kubectl exec mysql-695c4d9dcd-rvzff -i -t -n userns1 -- mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.49 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```

## <a name="delete-a-deployment"></a>Удаление развертывания

Чтобы удалить развертывание, удалите развернутые объекты по имени. К таким объектам относятся развертывание, Обслуживание и PVC.
 
```powershell
kubectl delete deployment <deployment-name>,svc <service-name> -n <your-namespace>
kubectl delete pvc <your-pvc-name> -n <your-namespace>
```

Ниже приведен пример выходных данных при удалении развертывания и службы.

```powershell
C:\Users\user>kubectl delete deployment,svc mysql -n userns1
deployment.apps "mysql" deleted
service "mysql" deleted
C:\Users\user>
```
Ниже приведен пример выходных данных при удалении PVC.

```powershell
C:\Users\user>kubectl delete pvc mysql-pv-claim-sc -n userns1
persistentvolumeclaim "mysql-pv-claim-sc" deleted
C:\Users\user>
```                                                                                         


## <a name="next-steps"></a>Дальнейшие действия

Сведения о настройке сети с помощью kubectl см. в статье [развертывание приложения без отслеживания состояния на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md) .