---
title: Краткое руководство. Развертывание кластера AKS с помощью портала Azure
titleSuffix: Azure Kubernetes Service
description: Узнайте, как быстро создать кластер Kubernetes, развертывать приложение и отслеживать производительность в Службе Azure Kubernetes (AKS) с помощью портала Azure.
services: container-service
ms.topic: quickstart
ms.date: 01/13/2021
ms.custom: mvc, seo-javascript-october2019, contperfq3
ms.openlocfilehash: 5f758c0bc50b2d4f22b3dbf0efaa4ecbc3f334cb
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102507812"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-aks-cluster-using-the-azure-portal"></a>Краткое руководство. Развертывание кластера Службы Azure Kubernetes (AKS) с помощью портала Azure

Служба Azure Kubernetes (AKS) — это управляемая служба Kubernetes, которая позволяет быстро развертывать кластеры и управлять ими. В этом кратком руководстве вы развернете кластер AKS с помощью портала Azure. Затем в кластере будет запущено многоконтейнерное приложение, которое включает в себя веб-интерфейс и экземпляр Redis. Затем вы узнаете, как отслеживать работоспособность кластера и модулей pod, на которых выполняется ваше приложение.

![Изображение перехода к примеру приложения Azure для голосования](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

В этом руководстве предполагается, что у вас есть некоторое представление о функциях Kubernetes. Дополнительные сведения см. в статье [Ключевые концепции Kubernetes для службы Azure Kubernetes (AKS)][kubernetes-concepts].

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на портал Azure по адресу [https://portal.azure.com](https://portal.azure.com).

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

Чтобы создать кластеры AKS, выполните следующие действия.

1. На **домашней странице** или в меню портала Azure выберите **Создать ресурс**.

2. Выберите **Контейнеры** >  **Служба Kubernetes**.

3. На странице **Основные** настройте следующие параметры:
    - **Сведения о проекте**. Выберите **подписку** Azure, а затем выберите или создайте **группу ресурсов** Azure, например *myResourceGroup*.
    - **Сведения о кластере**. Введите **Имя кластера Kubernetes**, например *myAKSCluster*. Выберите **регион** и **версию Kubernetes** для кластера AKS.
    - **Пул первичных узлов**: Выберите **Размер узла** виртуальной машины для узлов AKS. Размер виртуальной машины *невозможно* изменить после развертывания кластера с AKS.
            Выберите количество узлов для развертывания в кластере. В этом кратком руководстве задайте параметру **Число узлов** значение *1*. Количество узлов *можно* изменить после развертывания кластера.
    
    ![Создание кластера AKS — предоставление основных сведений](media/kubernetes-walkthrough-portal/create-cluster-basics.png)

    По завершении выберите **Next: Пулы узлов**.

4. На странице **Пулы узлов** оставьте значения по умолчанию. В нижней части экрана щелкните **Next: Authentication** (Далее: аутентификация).
    > [!CAUTION]
    > При создании нового идентификатора кластера может потребоваться несколько минут на его распространение и запуск, а также поиск субъектом-службой ошибок и сбоев проверки на портале Azure. Если вы столкнулись с этой проблемой, посетите страницу [Устранение распространенных проблем Службы Azure Kubernetes](troubleshooting.md#received-an-error-saying-my-service-principal-wasnt-found-or-is-invalid-when-i-try-to-create-a-new-cluster) для устранения неполадок.

5. На странице **Аутентификация** настройте следующие параметры.
    - Создайте новое удостоверение кластера, оставьте поле **Authentication** (Проверка подлинности) с **управляемым удостоверением System--assinged**. В качестве альтернативы можно выбрать **Субъект-службу**, чтобы использовать его. Выберите пункт *(создать) субъект-службу по умолчанию*, чтобы создать субъект-службу по умолчанию или *настроить субъект-службу* для использования существующей. При использовании существующего имени субъекта-службы необходимо указать для него идентификатор клиента и секрет.
    - Включите параметр для элементов управления доступом Kubernetes на основе ролей (Kubernetes RBAC). Это обеспечит более точный контроль доступа к ресурсам Kubernetes, развернутым в кластере AKS.

*Базовые* сетевые подключения используются по умолчанию, и включена служба Azure Monitor для контейнеров. Выберите **Review + create** (Проверить и создать), а по завершении проверки щелкните **Создать**.

Создание кластера AKS занимает несколько минут. После завершения развертывания щелкните **Перейти к ресурсу** или перейдите к группе ресурсов кластера AKS, например *myResourceGroup*, и выберите ресурс AKS, например *myAKSCluster*. Отобразится панель мониторинга кластера AKS, как показано ниже.

![Пример панелей мониторинга AKS на портале Azure](media/kubernetes-walkthrough-portal/aks-portal-dashboard.png)

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Управлять кластером Kubernetes можно c помощью [kubectl][kubectl], клиента командной строки Kubernetes. Клиент `kubectl` предварительно установлен в Azure Cloud Shell.

Откройте Cloud Shell с помощью кнопки `>_` в верхней части портала Azure.

![Открытие Azure Cloud Shell на портале](media/kubernetes-walkthrough-portal/aks-cloud-shell.png)

> [!NOTE]
> Чтобы выполнить эти операции в локальной оболочке, необходимо сначала проверить наличие Azure CLI в системе, а затем подключиться к Azure с помощью команды `az login`.

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][az-aks-get-credentials]. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования. В следующем примере возвращаются учетные данные для имени кластера *myAKSCluster* в группе ресурсов *myResourceGroup*.

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Чтобы проверить подключение к кластеру, используйте команду `kubectl get` для получения списка узлов кластера.

```console
kubectl get nodes
```

В следующем примере показан единый узел, созданный на предыдущих шагах. Убедитесь, что узел находится в состоянии *Ready* (Готово):

```output
NAME                       STATUS    ROLES     AGE       VERSION
aks-agentpool-14693408-0   Ready     agent     15m       v1.11.5
```

## <a name="run-the-application"></a>Выполнение приложения

Файл манифеста Kubernetes определяет требуемое состояние для кластера, включая образы контейнеров, которые нужно запустить. В этом кратком руководстве манифест используется для создания всех объектов, необходимых для запуска приложения Azure для голосования. Этот манифест включает в себя два развертывания Kubernetes. Одно используется для примера приложений Azure для голосования на Python, а другое — для экземпляра Redis. Создаются две службы Kubernetes, внутренняя и внешняя. Внутренняя служба используется для экземпляра Redis, а внешняя — для доступа к приложению Azure для голосования через Интернет.

В Cloud Shell с помощью редактора создайте файл `azure-vote.yaml` (например, выполнив команду `code azure-vote.yaml`, `nano azure-vote.yaml` или `vi azure-vote.yaml`). Затем скопируйте в него следующее определение YAML:

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

Разверните приложение с помощью команды `kubectl apply` и укажите имя манифеста YAML:

```console
kubectl apply -f azure-vote.yaml
```

В следующем примере выходных данных показано, что развертывания и службы успешно созданы.

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

Изначально для параметра *EXTERNAL-IP* (Внешний IP-адрес) службы *azure-vote-front* отображается состояние *pending* (ожидание).

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Когда значение *EXTERNAL-IP* изменится с состояния *pending* на фактический общедоступный IP-адрес, используйте команду `CTRL-C`, чтобы остановить процесс отслеживания `kubectl`. В следующем примере выходных данных показан общедоступный IP-адрес, присвоенный службе.

```output
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Чтобы увидеть приложение для голосования Azure в действии, откройте в веб-браузере внешний IP-адрес вашей службы.

![Изображение перехода к примеру приложения Azure для голосования](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

## <a name="monitor-health-and-logs"></a>Мониторинг работоспособности и журналов

При создании кластера была включена Azure Monitor для контейнеров. Эта функция мониторинга предоставляет метрики работоспособности кластера AKS и модулей, запущенных в кластере.

Он может потребоваться несколько минут, чтобы эти данные отобразились на портале Azure. Чтобы просмотреть текущее состояние, время непрерывной работы и использование ресурсов для модулей pod Azure для голосования, перейдите обратно к ресурсу AKS на портале Azure, например *myAKSCluster*. Сведения о состоянии работоспособности можно получить следующим образом:

1. В разделе **Отслеживание** в левой части окна выберите **Аналитика**.
1. В верхней части выберите **+Добавить фильтр**.
1. Выберите *Пространство имен* как свойство, а затем — *\<All but kube-system\>* .
1. Выберите **Контейнеры** для просмотра.

Отобразятся представления контейнеров *azure-vote-back* и *azure-vote-front*, как показано ниже:

![Представление работоспособности запущенных контейнеров в AKS](media/kubernetes-walkthrough-portal/monitor-containers.png)

Чтобы просмотреть журналы для модуля pod `azure-vote-front`, выберите параметр **Просмотреть журналы контейнера** в раскрывающемся меню списка контейнеров. Эти журналы содержат потоки *stdout* и *stderr* из контейнера.

![Представление журналов контейнеров в AKS](media/kubernetes-walkthrough-portal/monitor-container-logs.png)

## <a name="delete-cluster"></a>Удаление кластера

Если кластер больше не нужен, просто удалите кластерный ресурс. После этого будут удалены все связанные ресурсы. Для этого на портале Azure можно нажать кнопку **Удалить** на панели мониторинга AKS. Вы также можете выполнить команду [az group delete][az-aks-delete] в Cloud Shell.

```azurecli
az aks delete --resource-group myResourceGroup --name myAKSCluster --no-wait
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями][sp-delete]. Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="get-the-code"></a>Получение кода

В этом кратком руководстве для создания развертывания Kubernetes используются предварительно созданные образы контейнеров. Связанный с приложением код, файл Dockerfile и файл манифеста Kubernetes доступны на сайте GitHub.

[https://github.com/Azure-Samples/azure-voting-app-redis][azure-vote-app]

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства мы развернули кластер Kubernetes, а затем развернули в нем многоконтейнерное приложение.

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