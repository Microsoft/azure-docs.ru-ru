---
title: Использование kubectl для развертывания приложения с отслеживанием состояния Kubernetes с помощью статически подготовленного общего ресурса на Azure Stack пограничном устройстве Pro | Документация Майкрософт
description: Описывает создание Kubernetes развертывания приложений с отслеживанием состояния с помощью статически подготовленного общего ресурса с использованием kubectl на устройстве Azure Stack с ГРАФИЧЕСКИм стандартом Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/09/2021
ms.author: alkohli
ms.openlocfilehash: 01ba8e1f22deb376fd461be24d3f66f0a7f5e1ae
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102610490"
---
# <a name="use-kubectl-to-run-a-kubernetes-stateful-application-with-a-persistentvolume-on-your-azure-stack-edge-pro-device"></a>Использование kubectl для запуска приложения Kubernetes с отслеживанием состояния с помощью Персистентволуме на устройстве Azure Stack ребра Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье показано, как развернуть приложение с отслеживанием состояния с одним экземпляром в Kubernetes с помощью Персистентволуме (ПС) и развертывания. Развертывание использует `kubectl` команды в существующем кластере Kubernetes и развертывает приложение MySQL. 

Эта процедура предназначена для тех, кто проверил [хранилище Kubernetes на устройстве с Azure Stack пограничным Pro](azure-stack-edge-gpu-kubernetes-storage.md) и знаком с концепциями [хранилища Kubernetes](https://kubernetes.io/docs/concepts/storage/). 

Azure Stack пограничной Pro также поддерживает запуск контейнеров Azure SQL, которые можно развернуть подобным образом, как описано в разделе MySQL. Дополнительные сведения см. в разделе [Azure SQL ребр](../azure-sql-edge/overview.md).


## <a name="prerequisites"></a>Предварительные требования

Перед развертыванием приложения с отслеживанием состояния выполните следующие предварительные требования на устройстве и клиенте, которые будут использоваться для доступа к устройству.

### <a name="for-device"></a>Для устройств

- У вас есть учетные данные для входа на 1 узел Azure Stack пограничным устройством Pro.
    - Устройство активировано. См. раздел [Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
    - Устройство имеет роль вычислений, настроенную через портал Azure и имеющую кластер Kubernetes. См. раздел [Настройка вычислений](azure-stack-edge-gpu-deploy-configure-compute.md).

### <a name="for-client-accessing-the-device"></a>Для клиента, обращающегося к устройству

- У вас есть клиентская система Windows, которая будет использоваться для доступа к устройству Azure Stack погранично Pro.
    - Клиент работает под управлением Windows PowerShell 5,0 или более поздней версии. Чтобы скачать последнюю версию Windows PowerShell, перейдите к разделу [Установка Windows PowerShell](/powershell/scripting/install/installing-windows-powershell).
    
    - Также можно использовать любой другой клиент с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device) . В этой статье описывается процедура использования клиента Windows. 
    
    - Вы выполнили процедуру, описанную в статье [доступ к кластеру Kubernetes на устройстве с Azure Stack ребр Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Вы выполнили следующие задачи:
      - Создание `userns1` пространства имен с помощью `New-HcsKubernetesNamespace` команды. 
      - Создание пользователя с `user1` помощью `New-HcsKubernetesUser` команды. 
      - Предоставлен `user1` доступ к `userns1` через `Grant-HcsKubernetesNamespaceAccess` команду.       
      - Установлен `kubectl` на клиенте и сохранил `kubeconfig` файл с пользовательской конфигурацией в C: \\ Users \\ &lt; &gt; \\ . KUBE. 
    
    - Убедитесь, что `kubectl` версия клиента отклонена, но не имеет более одной версии из главной версии Kubernetes, работающей на устройстве Azure Stack погранично Pro. 
        - Используйте `kubectl version` для проверки версии kubectl, работающей на клиенте. Запишите полную версию.
        - В локальном пользовательском интерфейсе устройства Azure Stack ребра Pro перейдите к **обзору** и запишите номер Kubernetes Software. 
        - Проверьте эти две версии на совместимость с сопоставлением, предоставленным в поддерживаемой версии Kubernetes.<!-- insert link--> 


Вы готовы к развертыванию приложения с отслеживанием состояния на устройстве Azure Stack ребра Pro. 

## <a name="provision-a-static-pv"></a>Подготавливается статическая ПС

Чтобы статически подготавливать ПС, необходимо создать общую папку на устройстве. Выполните следующие действия, чтобы предоставить ПС для общего ресурса SMB. 

> [!NOTE]
> - Конкретный пример, используемый в этой статье, не работает с общими папками NFS. Как правило, общие папки NFS можно подготовить на Azure Stack пограничном устройстве с приложениями, не являющимися базами данных.
> - Для развертывания приложений с отслеживанием состояния, которые используют тома хранилища для обеспечения постоянного хранения, рекомендуется использовать `StatefulSet` . В этом примере используется `Deployment` только с одной репликой и подходит для разработки и тестирования. 

1. Выберите, следует ли создать пограничные или пограничные локальные папки. Следуйте инструкциям в разделе [Добавление общей папки](azure-stack-edge-manage-shares.md#add-a-share) для создания общей папки. Убедитесь, что установлен флажок **использовать общий ресурс с границей COMPUTE**.

    ![Пограничная локальная общая папка для ПС](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/edge-local-share-static-provision-1.png)

    1. Если вы решили использовать существующую общую папку, вместо создания новой общей папки необходимо подключить общую папку.
    
        В портал Azure для ресурса Azure Stackного периметра перейдите в раздел **Общие ресурсы**. В существующем списке общих папок выберите и щелкните общую папку, которую вы хотите использовать.

        ![Выберите существующий локальный общий ресурс для ПС](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/mount-edge-share-1.png)

    1. При появлении запроса выберите **Подключение** и подтверждение подключения.  

        ![Подключение существующего локального общего ресурса для ПС](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/mount-edge-share-2.png)

1. Запишите имя общей папки. При создании этой общей папки в кластере Kubernetes, соответствующем созданной общей папке SMB, автоматически создается объект постоянного тома. 

## <a name="deploy-mysql"></a>Развертывание MySQL

Теперь вы сможете запустить приложение с отслеживанием состояния, создав развертывание Kubernetes и подключив его к ПС, созданному на предыдущем шаге, с помощью Персистентволумеклаим (PVC). 

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

1. Вы будете использовать следующие файлы YAML. В этом `mysql-deployment.yml` файле описывается развертывание, которое запускает MySQL и ссылается на PVC. Файл определяет подключение тома для `/var/lib/mysql` , а затем создает PVC, который выполняет поиск тома размером 20 ГБ. 

    Это утверждение удовлетворяет любому существующему ПС, который был статически подготовлен при создании общего ресурса на предыдущем шаге. На вашем устройстве для каждой общей папки создается большая 32 ТБ. ПС соответствует требованиям, заданным в PVC, и PVC должен быть привязан к этой ПС.

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
              claimName: mysql-pv-claim
    ```
    
2. Скопируйте и сохраните в виде `mysql-pv.yml` файла в той же папке, где был сохранен `mysql-deployment.yml` . Чтобы использовать общий ресурс SMB, созданный ранее с помощью `kubectl` , задайте в `volumeName` поле в объекте PVC имя общей папки. 

    > [!NOTE] 
    > Убедитесь, что YAML файлы имеют правильный отступ. Проверить и сохранить можно с помощью [YAML Lint](http://www.yamllint.com/) .
   
    ```yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim
    spec:
      volumeName: <smb-share-name-here>
      storageClassName: ""
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

3. Разверните `mysql-pv.yaml` файл.

    `kubectl apply -f <URI path to the mysql-pv.yml file> -n <your-user-namespace>`
    
    Ниже приведен пример выходных данных развертывания.

    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-pv.yml" -n userns1
    persistentvolumeclaim/mysql-pv-claim created
    
    C:\Users\user>
    ```
   Запишите имя созданной PVC. Вы будете использовать его на более позднем этапе. 

4. Разверните содержимое `mysql-deployment.yml` файла.

    `kubectl apply -f <URI path to mysql-deployment.yml file> -n <your-user-namespace>`

    Ниже приведен пример выходных данных развертывания.
    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-deployment.yml" -n userns1
        service/mysql created
        deployment.apps/mysql created
    ```
    
5. Отображает сведения о развертывании.

    `kubectl describe deployment <app-label> -n <your-user-namespace>`
    
    ```powershell
    C:\Users\user>kubectl describe deployment mysql -n userns1
    Name:               mysql
    Namespace:          userns1
    CreationTimestamp:  Tue, 18 Aug 2020 09:44:58 -0700
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
        ClaimName:  mysql-pv-claim
        ReadOnly:   false
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Progressing    True    NewReplicaSetAvailable
      Available      True    MinimumReplicasAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-c85f7f79c (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set mysql-c85f7f79c to 1
    
    C:\Users\user>
    ```
    

6. Список модулей Pod, созданных при развертывании.

    `kubectl get pods -l <app=label> -n <your-user-namespace>`

    Ниже приведен пример выходных данных.

    
    ```powershell
    C:\Users\user>kubectl get pods -l app=mysql -n userns1
    NAME                    READY   STATUS    RESTARTS   AGE
    mysql-c85f7f79c-vzz7j   1/1     Running   1          14m
    
    C:\Users\user>
    ```
    
7. Проверьте Персистентволумеклаим.

    `kubectl describe pvc <your-pvc-name>`

    Ниже приведен пример выходных данных.

    
    ```powershell
    C:\Users\user>kubectl describe pvc mysql-pv-claim -n userns1
    Name:          mysql-pv-claim
    Namespace:     userns1
    StorageClass:
    Status:        Bound
    Volume:        mylocalsmbshare1
    Labels:        <none>
    Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"mysql-pv-claim","namespace":"userns1"},"spec":{"acc...
                   pv.kubernetes.io/bind-completed: yes
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      32Ti
    Access Modes:  RWO,RWX
    VolumeMode:    Filesystem
    Mounted By:    mysql-c85f7f79c-vzz7j
    Events:        <none>
    
    C:\Users\user>
    ```
    

## <a name="verify-mysql-is-running"></a>Проверка выполнения MySQL


Чтобы выполнить команду для контейнера в модуле Pod, работающем под управлением MySQL, введите:

`kubectl exec <your-pod-with-the-app> -i -t -n <your-namespace> -- mysql`

Ниже приведен пример выходных данных.

```powershell
C:\Users\user>kubectl exec mysql-c85f7f79c-vzz7j -i -t -n userns1 -- mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
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
C:\Users\user>kubectl delete pvc mysql-pv-claim -n userns1
persistentvolumeclaim "mysql-pv-claim" deleted
C:\Users\user>
```

ПС больше не привязан к виртуальной цепи, так как PVC удален. Так как ПС был подготовлен при создании общей папки, необходимо удалить общую папку. Выполните следующие действия.

1. Отключите общий ресурс. В портал Azure перейдите к **ресурсу Azure Stack ребра > общие ресурсы** и выберите общую папку, которую нужно отключить. Выберите **Отключить** и подтвердите операцию. Подождите, пока общий ресурс не будет отключен. При отключении общая папка (и, следовательно, связанная Персистентволуме) удаляется из кластера Kubernetes. 

    ![Отсоединение локального общего ресурса для ПС](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/unmount-edge-local-share-1.png)

1. Теперь можно выбрать **Удалить** и подтвердить удаление, чтобы удалить общую папку. Он также должен удалить общую папку и соответствующий ПС.

    ![Удаление локального общего ресурса для ПС](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/delete-edge-local-share-1.png)


## <a name="next-steps"></a>Дальнейшие действия

Сведения о динамической подготовке хранилища см. в статье [развертывание приложения с отслеживанием состояния с помощью динамической подготовки на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md) .
