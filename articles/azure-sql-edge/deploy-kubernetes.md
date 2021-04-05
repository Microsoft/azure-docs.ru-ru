---
title: Развертывание контейнера SQL Azure для пограничных вычислений в Kubernetes — SQL Azure для пограничных вычислений
description: Cведения о развертывании контейнера SQL Azure для пограничных вычислений в Kubernetes.
keywords: sql для пограничных вычислений, контейнер, kubernetes
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 31a454c93ad5192f387306a8ec557c4e4d3ae991
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93395297"
---
# <a name="deploy-an-azure-sql-edge-container-in-kubernetes"></a>Развертывание контейнера SQL Azure для пограничных вычислений в Kubernetes

SQL Azure для пограничных вычислений можно развернуть в кластере Kubernetes как модуль IoT Edge с помощью Azure IoT Edge, запущенного в Kubernetes или в качестве изолированного модуля pod контейнера. В оставшейся части этой статьи мы рассмотрим автономное развертывание контейнеров в кластере Kubernetes. Сведения о развертывании Azure IoT Edge в Kubernetes см. в [этой статье](https://microsoft.github.io/iotedge-k8s-doc/introduction.html).

В этом учебнике показано, как настроить высокодоступный экземпляр SQL Azure для пограничных вычислений в контейнере в кластере Kubernetes. 

> [!div class="checklist"]
> * Создание пароля SA
> * Создание хранилища
> * Создание развертывания
> * Подключение в SQL Server Management Studio (SSMS)
> * Проверка сбоя и восстановление

В Kubernetes версии 1.6 и более поздних поддерживаются [классы хранилища](https://kubernetes.io/docs/concepts/storage/storage-classes/), [утверждения постоянного тома](https://kubernetes.io/docs/concepts/storage/storage-classes/#persistentvolumeclaims), а также [тип тома диска Azure](https://github.com/kubernetes/examples/tree/master/staging/volumes/azure_disk). Поддерживается создание собственных экземпляров SQL Azure для пограничных вычислений в собственном коде Kubernetes и управление ими. В примере, приведенном в этой статье, показано, как создать [развертывание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) для достижения высокой доступности, аналогичной экземпляру отказоустойчивого кластера общего диска. В этой конфигурации Kubernetes играет роль оркестратора кластера. При сбое экземпляра SQL Azure для пограничных вычислений в контейнере оркестратор загружает другой экземпляр контейнера, который подключается к тому же постоянному хранилищу.

![SQL Azure для пограничных вычислений в кластере Kubernetes](media/deploy-kubernetes/kubernetes-sql-edge.png)

На предыдущей схеме `azure-sql-edge` — это контейнер внутри модуля [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/). Kubernetes управляет ресурсами в кластере. Применение [набора реплик](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) гарантирует автоматическое восстановление pod в случае отказа узла. Приложения подключаются к службе. В этом случае служба представляет подсистему балансировки нагрузки, которая содержит IP-адрес, остающийся неизменным в случае отказа `azure-sql-edge`.

На следующей схеме в контейнере `azure-sql-edge` происходит сбой. Как оркестратор Kubernetes гарантирует правильное количество работоспособных экземпляров в наборе реплик и запускает новый контейнер в соответствии с конфигурацией. Оркестратор запускает новый модуль Pod на том же узле, и `azure-sql-edge` повторно подключается к тому же постоянному хранилищу. Служба подключается к повторно созданному `azure-sql-edge`.

![SQL Azure для пограничных вычислений в кластере Kubernetes после сбоя модуля pod](media/deploy-kubernetes/kubernetes-sql-edge-after-pod-fail.png)

На следующей схеме сбой затронул узел, на котором размещен контейнер `azure-sql-edge`. Оркестратор запускает новый модуль Pod на другом узле, и `azure-sql-edge` повторно подключается к тому же постоянному хранилищу. Служба подключается к повторно созданному `azure-sql-edge`.

![SQL Azure для пограничных вычислений в кластере Kubernetes после сбоя узла](media/deploy-kubernetes/kubernetes-sql-edge-after-node-fail.png)

## <a name="prerequisites"></a>Предварительные требования

* **Кластер Kubernetes**
   - Для работы с этим руководством требуется кластер Kubernetes. В шагах для управления кластером используется [kubectl](https://kubernetes.io/docs/user-guide/kubectl/). 

   - В этом учебнике мы будем использовать службу Azure Kubernetes для развертывания SQL Azure для пограничных вычислений. См. раздел [Развертывание кластера службы Azure Kubernetes (AKS)](../aks/tutorial-kubernetes-deploy-cluster.md) для создания кластера Kubernetes с одним узлом в AKS с помощью `kubectl`. 

   >[!NOTE]
   >Для защиты от сбоя узла кластер Kubernetes требует наличия нескольких узлов.

* **Azure CLI**
   - Инструкции в этом учебнике проверены на соответствие Azure CLI 2.10.1.

## <a name="create-a-kubernetes-namespace-for-sql-edge-deployment"></a>Создание пространства имен Kubernetes для развертывания SQL для пограничных вычислений

Создайте пространство имен в кластере Kubernetes. Это пространство имен будет использоваться для развертывания SQL для пограничных вычислений и всех необходимых артефактов. Дополнительные сведения см. в [документации по пространствам имен](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/).

   ```azurecli
   kubectl create namespace <namespace name>
   ```  

## <a name="create-an-sa-password"></a>Создание пароля SA

Создайте пароль SA в кластере Kubernetes. Kubernetes может управлять конфиденциальными сведениями о конфигурации, например паролями, в качестве [секретов](https://kubernetes.io/docs/concepts/configuration/secret/).

Следующая команда создает пароль для учетной записи SA:

   ```azurecli
   kubectl create secret generic mssql --from-literal=SA_PASSWORD="MyC0m9l&xP@ssw0rd" -n <namespace name>
   ```  

   Замените `MyC0m9l&xP@ssw0rd` надежным паролем.

## <a name="create-storage"></a>Создание хранилища

Настройте [постоянный том](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) и [утверждение постоянного тома](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volume-claim-protection) в кластере Kubernetes. Необходимо выполнить следующие шаги. 

1. Создайте манифест, чтобы определить класс хранения и утверждение постоянного тома.  В манифесте описана подготовка хранилища, параметры и [политика обработки заявок на хранение](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming). Кластер Kubernetes использует этот манифест для создания постоянного хранилища. 

   В следующем примере на языке YAML определяется класс хранения и утверждение постоянного тома. Подготовка класса хранилища — `azure-disk`, так как этот кластер Kubernetes находится в Azure. Тип учетной записи хранения — `Standard_LRS`. Утверждение постоянного тома называется `mssql-data`. Метаданные утверждения постоянного тома содержат заметку, которая относит его к классу хранения. 

   ```yaml
   kind: StorageClass
   apiVersion: storage.k8s.io/v1
   metadata:
        name: azure-disk
   provisioner: kubernetes.io/azure-disk
   parameters:
     storageaccounttype: Standard_LRS
     kind: managed
   ---
   kind: PersistentVolumeClaim
   apiVersion: v1
   metadata:
     name: mssql-data
     annotations:
       volume.beta.kubernetes.io/storage-class: azure-disk
   spec:
     accessModes:
     - ReadWriteOnce
     resources:
       requests:
         storage: 8Gi
   ```

   Сохраните файл (например, **pvc.yaml**).

2. Создайте утверждение постоянного тома в Kubernetes.

   ```azurecli
   kubectl apply -f <Path to pvc.yaml file> -n <namespace name>
   ```

   `<Path to pvc.yaml file>` — это расположение, в котором сохранен файл.

   Постоянный том создается автоматически как учетная запись хранения Azure и связывается с утверждением постоянного тома. 

    ![Снимок экрана: команда утверждения постоянного тома](media/deploy-kubernetes/pvc-cmd.png)

3. Проверьте утверждение постоянного тома.

   ```azurecli
   kubectl describe pvc <PersistentVolumeClaim>  -n <name of the namespace>
   ```

   Утверждение постоянного тома теперь называется `<PersistentVolumeClaim>`.

   На предыдущем шаге утверждение постоянного тома называется `mssql-data`. Чтобы просмотреть метаданные об утверждении постоянного тома, выполните следующую команду:

   ```azurecli
   kubectl describe pvc mssql-data  -n <namespace name>
   ```

   Возвращаемые метаданные содержат значение с именем `Volume`. Это значение сопоставляется с именем большого двоичного объекта.

   ![Снимок экрана возвращенных метаданных, включающих том](media/deploy-kubernetes/describe-volume.png)

4. Проверьте постоянный том.

   ```azurecli
   kubectl describe pv -n <namespace name>
   ```

   `kubectl` возвращает метаданные постоянного тома, который создается автоматически и связывается с утверждением постоянного тома. 

## <a name="create-the-deployment"></a>Создание развертывания

В этом примере контейнер, в котором размещен экземпляр SQL Azure для пограничных вычислений, описан как объект развертывания Kubernetes. При развертывании создается набор реплик. Набор реплик создает Pod. 

На этом шаге создайте манифест, описывающий контейнер на основе образа SQL Azure для пограничных вычислений в Docker. Манифест ссылается на утверждение постоянного тома `mssql-data` и секрет `mssql`, который вы уже применили к кластеру Kubernetes. Манифест также описывает [службу](https://kubernetes.io/docs/concepts/services-networking/service/). Эта служба является подсистемой балансировки нагрузки. Она гарантирует, что IP-адрес будет сохранен после восстановления экземпляра SQL Azure для пограничных вычислений. 

1. Создайте манифест (YAML-файл) для описания развертывания. В следующем примере описывается развертывание, включая контейнер, основанный на образе контейнера SQL Azure для пограничных вычислений.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sqledge-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sqledge
  template:
    metadata:
      labels:
        app: sqledge
    spec:
      volumes:
        - name: sqldata
          persistentVolumeClaim:
            claimName: mssql-data
      containers:
        - name: azuresqledge
          image: mcr.microsoft.com/azure-sql-edge:latest
          ports:
            - containerPort: 1433
          volumeMounts:
            - name: sqldata
              mountPath: /var/opt/mssql
          env:
            - name: MSSQL_PID
              value: "Developer"
            - name: ACCEPT_EULA
              value: "Y"
            - name: SA_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mssql
                  key: SA_PASSWORD
            - name: MSSQL_AGENT_ENABLED
              value: "TRUE"
            - name: MSSQL_COLLATION
              value: "SQL_Latin1_General_CP1_CI_AS"
            - name: MSSQL_LCID
              value: "1033"
      terminationGracePeriodSeconds: 30
      securityContext:
        fsGroup: 10001
---
apiVersion: v1
kind: Service
metadata:
  name: sqledge-deployment
spec:
  selector:
    app: sqledge
  ports:
    - protocol: TCP
      port: 1433
      targetPort: 1433
      name: sql
  type: LoadBalancer
```

   Скопируйте приведенный выше код в новый файл с именем `sqldeployment.yaml`. Измените следующие значения: 

   * MSSQL_PID `value: "Developer"`: задает контейнер для запуска SQL Azure для пограничных вычислений выпуска Developer Edition. Выпуск Developer Edition не лицензирован для рабочих данных. Если развертывание предназначено для использования в рабочей среде, задайте выпуск `Premium`. 

      >[!NOTE]
      >Дополнительные сведения см. в статье о [лицензировании SQL Azure для пограничных вычислений](https://azure.microsoft.com/pricing/details/sql-edge/).

   * `persistentVolumeClaim`: Это значение требует наличия записи для `claimName:`, сопоставленного с именем, используемым для утверждения постоянного тома. В этом учебнике используется `mssql-data`. 

   * `name: SA_PASSWORD`: настраивает образ контейнера для установки пароля SA, определенного в этом разделе.

     ```yaml
     valueFrom:
       secretKeyRef:
         name: mssql
         key: SA_PASSWORD 
     ```

     Когда Kubernetes развертывает контейнер, он обращается к секрету с именем `mssql`, чтобы получить значение пароля. 

   >[!NOTE]
   >Используя тип службы `LoadBalancer`, экземпляр SQL Azure для пограничных вычислений будет доступен удаленно (через Интернет) через порт 1433.

   Сохраните файл (например, **sqledgedeploy.yaml**).

2. Создайте развертывание.

   ```azurecli
   kubectl apply -f <Path to sqledgedeploy.yaml file> -n <namespace name>
   ```

   `<Path to sqldeployment.yaml file>` — это расположение, в котором сохранен файл.

   ![Снимок экрана команды развертывания](media/deploy-kubernetes/deploy-sql-cmd.png)

   Развертывание и служба будут созданы. Экземпляр SQL Azure для пограничных вычислений находится в контейнере, подключенном к постоянному хранилищу.

   Для просмотра состояния модуля Pod введите `kubectl get pod -n <namespace name>`.

   ![Снимок экрана команды get pod](media/deploy-kubernetes/get-sql-pod-cmd.png)

   На предыдущем изображении модуль имеет состояние `Running`. Это состояние указывает, что контейнер готов. Это может занять несколько минут.

   >[!NOTE]
   >После создания развертывания может пройти несколько минут, прежде чем будет виден модуль. Задержка связана с тем, что кластер извлекает образ контейнера SQL Azure для пограничных вычислений из центра Docker. После первого извлечения образа последующие развертывания могут выполняться быстрее, если развертывание выполняется на узле, где уже есть кэшированный образ. 

3. Убедитесь в том, что службы запущены. Выполните следующую команду:

   ```azurecli
   kubectl get services -n <namespace name>
   ```

   Эта команда возвращает службы, которые выполняются, а также внутренние и внешние IP-адреса этих служб. Запишите внешний IP-адрес для службы `mssql-deployment`. Используйте этот IP-адрес для подключения к SQL Azure для пограничных вычислений. 

   ![Снимок экрана команды get service](media/deploy-kubernetes/get-service-cmd.png)

   Чтобы получить дополнительные сведения о состоянии объектов в кластере Kubernetes, выполните команду:

   ```azurecli
   az aks browse --resource-group <MyResourceGroup> --name <MyKubernetesClustername>
   ```  

## <a name="connect-to-the-azure-sql-edge-instance"></a>Подключение к экземпляру SQL Azure для пограничных вычислений

Если вы настроили контейнер согласно описанию, вы можете подключиться к приложению извне виртуальной сети Azure. Используйте учетную запись `sa` и внешний IP-адрес для службы. Используйте пароль, настроенный в качестве секрета Kubernetes. Дополнительные сведения о подключении к экземпляру SQL Azure для пограничных вычислений см. в [этой статье](connect.md).

## <a name="verify-failure-and-recovery"></a>Проверка сбоя и восстановление

Чтобы проверить сбой и восстановление, можно удалить модуль Pod. Выполните следующие шаги:

1. Перечислите модули Pod, в которых выполняется SQL Azure для пограничных вычислений.

   ```azurecli
   kubectl get pods -n <namespace name>
   ```

   Запишите имя модуля, на котором выполняется SQL Azure для пограничных вычислений.

2. Удалите модуль.

   ```azurecli
   kubectl delete pod sqledge-deployment-7df66c9999-rc9xl
   ```
   `sqledge-deployment-7df66c9999-rc9xl` — это значение, возвращенное на предыдущем шаге для имени Pod. 

Kubernetes автоматически повторно создает модуль pod для восстановления экземпляра SQL Azure для пограничных вычислений и подключения к постоянному хранилищу. Используйте `kubectl get pods` для проверки развертывания нового Pod. Используйте `kubectl get services` для проверки того, что IP-адрес контейнера не изменился. 

## <a name="summary"></a>Сводка

Их этого учебника вы узнали, как развертывать контейнеры SQL Azure для пограничных вычислений в кластере Kubernetes для обеспечения высокой доступности. 

> [!div class="checklist"]
> * Создание пароля SA
> * Создание хранилища
> * Создание развертывания
> * Подключение в SQL Azure для пограничных вычислений Management Studio (SSMS)
> * Проверка сбоя и восстановление

## <a name="next-steps"></a>Дальнейшие действия

- [Введение в Kubernetes](../aks/intro-kubernetes.md)
- [Машинное обучение и искусственный интеллект с использованием ONNX в SQL для пограничных вычислений](onnx-overview.md)
- [Создание комплексного решения для Интернета вещей с использованием SQL для пограничных вычислений на основе IoT Edge](tutorial-deploy-azure-resources.md).
- [Потоковая передача данных в SQL Azure для пограничных вычислений](stream-data.md)