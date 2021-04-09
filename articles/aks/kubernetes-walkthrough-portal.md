---
title: Краткое руководство. Развертывание кластера AKS с помощью портала Azure
titleSuffix: Azure Kubernetes Service
description: Узнайте, как быстро создать кластер Kubernetes, развертывать приложение и отслеживать производительность в Службе Azure Kubernetes (AKS) с помощью портала Azure.
services: container-service
ms.topic: quickstart
ms.date: 03/15/2021
ms.custom: mvc, seo-javascript-october2019, contperf-fy21q3
ms.openlocfilehash: aa07dd84cbd107aca77236f4d084ef95a7f1005b
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105544287"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-aks-cluster-using-the-azure-portal"></a>Краткое руководство. Развертывание кластера Службы Azure Kubernetes (AKS) с помощью портала Azure

Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. В этом кратком руководстве вы выполните указанные ниже задачи.
* Развертывание кластера AKS с помощью портала Azure. 
* Запуск многоконтейнерного приложения, которое включает в себя веб-интерфейс и экземпляр Redis в кластере. 
* Мониторинг работоспособности кластера и pod, на которых выполняется приложение.

![Изображение перехода к примеру приложения Azure для голосования](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

В этом руководстве предполагается, что у вас есть некоторое представление о функциях Kubernetes. Дополнительные сведения см. в статье [Ключевые концепции Kubernetes для службы Azure Kubernetes (AKS)][kubernetes-concepts].

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com).

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

1. На **домашней странице** или в меню портала Azure выберите **Создать ресурс**.

2. Выберите **Контейнеры** > **Служба Kubernetes**.

3. На странице **Основные** настройте следующие параметры:
    - **Сведения о проекте**. 
        * Выберите **подписку Azure**.
        * Выберите или создайте **группу ресурсов** Azure, например *myResourceGroup*.
    - **Сведения о кластере**. 
        * Введите **Имя кластера Kubernetes**, например *myAKSCluster*. 
        * Выберите **регион** и **версию Kubernetes** для кластера AKS.
    - **Пул первичных узлов**: 
        * Выберите **размер узла** виртуальной машины для узлов AKS. Размер виртуальной машины *невозможно* изменить после развертывания кластера с AKS.
        * Выберите количество узлов для развертывания в кластере. В этом кратком руководстве задайте параметру **Число узлов** значение *1*. Количество узлов *можно* изменить после развертывания кластера.
    
    ![Создание кластера AKS — предоставление основных сведений](media/kubernetes-walkthrough-portal/create-cluster-basics.png)

4. По завершении выберите **Next: Пулы узлов**.

5. Сохраните значения параметров **Пулы узлов**, предложенные по умолчанию. В нижней части экрана щелкните **Next: Authentication** (Далее: аутентификация).
    > [!CAUTION]
    > При создании новых субъектов-служб Azure AD может потребоваться несколько минут на их распространение и доступность. До завершения этого процесса на портале Azure будут появляться ошибки вида "субъект-служба не найден" и сбои проверки. Если вы столкнулись с этой проблемой, обратитесь к [статье по устранению неполадок](troubleshooting.md#received-an-error-saying-my-service-principal-wasnt-found-or-is-invalid-when-i-try-to-create-a-new-cluster).

6. На странице **Аутентификация** настройте следующие параметры.
    - Создайте новый идентификатор кластера, выполнив любое из следующих действий:
        * Сохраните для поля **Проверка подлинности** значение **System-assinged managed identity** (Назначаемое системой управляемое удостоверение).
        * Выберите в поле **Субъект-служба** необходимый субъект-службу. 
            * Щелкните *Субъект-служба по умолчанию (новый)* , чтобы создать субъект-службу по умолчанию.
            * Либо выберите *Настроить субъект-службу*, чтобы использовать существующий субъект-службу. Предоставьте идентификатор клиента и секрет для имени субъекта службы, если используете существующий.
    - Включите управление доступом на основе ролей Kubernetes (RBAC Kubernetes), чтобы обеспечить более детальный контроль доступа к ресурсам Kubernetes, развернутым в кластере AKS.

    *Базовые* сетевые подключения используются по умолчанию, и включена служба Azure Monitor для контейнеров. 

7. Выберите **Review + create** (Проверить и создать), а по завершении проверки щелкните **Создать**. 


8. Создание кластера AKS занимает несколько минут. Когда развертывание завершится, перейдите к этому ресурсу любым из следующих способов.
    * Щелкните **Переход к ресурсу**.
    * Либо перейдите к группе ресурсов кластера AKS и выберите ресурс AKS. 
        * В представленном ниже примере панели мониторинга кластера это группа *myResourceGroup* и ресурс *myAKSCluster*.

        ![Пример панелей мониторинга AKS на портале Azure](media/kubernetes-walkthrough-portal/aks-portal-dashboard.png)

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Кластером Kubernetes можно управлять при помощи [kubectl][kubectl] клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. 

1. Откройте Cloud Shell с помощью кнопки `>_` в верхней части портала Azure.

    ![Открытие Azure Cloud Shell на портале](media/kubernetes-walkthrough-portal/aks-cloud-shell.png)

    > [!NOTE]
    > Вы можете выполнить эти операции в локальной установке оболочки следующим образом.
    > 1. Убедитесь, что установлен интерфейс командной строки Azure.
    > 2. Подключитесь к Azure с помощью команды `az login`.

2. Настройте в `kubectl` подключение к кластеру Kubernetes, выполнив команду [az aks get-credentials][az-aks-get-credentials]. Следующая команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования;

    ```azurecli
    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    ```

3. Проверьте подключение к кластеру, получив список узлов кластера с помощью команды `kubectl get`.

    ```console
    kubectl get nodes
    ```

    В выходных данных будет представлен один узел, созданный на предыдущих шагах. Убедитесь, что этот узел находится в состоянии *готовности*:

    ```output
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-agentpool-14693408-0   Ready     agent     15m       v1.11.5
    ```

## <a name="run-the-application"></a>Выполнение приложения

Файл манифеста Kubernetes определяет требуемое состояние кластера, например список выполняемых в нем образов контейнеров. 

В этом кратком руководстве вы примените манифест для создания всех объектов, необходимых для запуска приложения Azure для голосования. Этот манифест содержит два развертывания Kubernetes:
* пример приложения Azure для голосования на языке Python;
* экземпляр Redis. 

Также создаются две службы Kubernetes:
* внутренняя служба для экземпляра Redis;
* внешняя служба для доступа к приложению Azure для голосования из Интернета.

1. В Cloud Shell с помощью редактора создайте файл с именем `azure-vote.yaml`, например так:
    * `code azure-vote.yaml`
    * `nano azure-vote.yaml`или 
    * `vi azure-vote.yaml`. 

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

1. Разверните приложение с помощью команды `kubectl apply` и укажите имя манифеста YAML:

    ```console
    kubectl apply -f azure-vote.yaml
    ```

    Выходные данные подтверждают успешное создание развертываний и служб:

    ```output
    deployment "azure-vote-back" created
    service "azure-vote-back" created
    deployment "azure-vote-front" created
    service "azure-vote-front" created
    ```

## <a name="test-the-application"></a>Тестирование приложения

При запуске приложения Служба Kubernetes предоставляет внешний интерфейс приложения в Интернете. Процесс создания может занять несколько минут.

Чтобы отслеживать ход выполнения, используйте команду `kubectl get service` с аргументом `--watch`.

```console
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

![Изображение перехода к примеру приложения Azure для голосования](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

## <a name="monitor-health-and-logs"></a>Мониторинг работоспособности и журналов

При создании кластера была включена Azure Monitor для контейнеров. Azure Monitor для контейнеров предоставляет метрики работоспособности для кластера AKS и pod, запущенных в этом кластере.

Передача данных метрик на портал Azure занимает несколько минут. Чтобы увидеть текущее состояние работоспособности, время доступности и потребление ресурсов для pod приложения Azure для голосования, выполните следующую процедуру:

1. Вернитесь к ресурсу AKS на портале Azure.
1. В разделе **Отслеживание** в левой части окна выберите **Аналитика**.
1. В верхней части щелкните **+Добавить фильтр**.
1. Выберите свойство **Пространство имен** и укажите значение *\<All but kube-system\>* .
1. Щелкните **Контейнеры**, чтобы просмотреть список.

Отобразятся контейнеры `azure-vote-back` и `azure-vote-front`, как показано в следующем примере.

![Представление работоспособности запущенных контейнеров в AKS](media/kubernetes-walkthrough-portal/monitor-containers.png)

Чтобы увидеть журналы для pod `azure-vote-front`, выберите действие **Просмотреть журналы контейнера** в раскрывающемся списке контейнеров. Эти журналы содержат потоки *stdout* и *stderr* из контейнера.

![Представление журналов контейнеров в AKS](media/kubernetes-walkthrough-portal/monitor-container-logs.png)

## <a name="delete-cluster"></a>Удаление кластера

Чтобы не оплачивать ненужные ресурсы Azure, удалите их. Нажмите кнопку **Удалить** на панели мониторинга для кластера AKS. Вы также можете использовать команду [az aks delete][az-aks-delete] в Cloud Shell:

```azurecli
az aks delete --resource-group myResourceGroup --name myAKSCluster --no-wait
```
> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями][sp-delete].
> 
> Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="get-the-code"></a>Получение кода

В этом кратком руководстве для создания развертывания Kubernetes вы применили предварительно созданные образы контейнеров. Вы можете получить код приложений, файл Dockerfile и файл манифеста Kubernetes для этих образов [на сайте GitHub.][azure-vote-app]

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы развернули кластер Kubernetes и многоконтейнерное приложение в нем. Откройте веб-панель мониторинга Kubernetes для кластера AKS.


Чтобы получить дополнительные сведения об AKS с полным примером создания приложения, развертыванием из Реестра контейнеров Azure, обновления работающего приложения, а также масштабирования и обновления кластера, ознакомьтесь с учебником по кластеру Kubernetes.

> [!div class="nextstepaction"]
> [Руководство по AKS][aks-tutorial]

<!-- LINKS - external -->
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-documentation]: https://kubernetes.io/docs/home/

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-delete]: /cli/azure/aks#az-aks-delete
[aks-monitor]: ../azure-monitor/containers/container-insights-overview.md
[aks-network]: ./concepts-network.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[http-routing]: ./http-application-routing.md
[sp-delete]: kubernetes-service-principal.md#additional-considerations