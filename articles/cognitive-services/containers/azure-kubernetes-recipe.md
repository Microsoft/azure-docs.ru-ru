---
title: Запуск распознавание языка контейнера в службе Kubernetes
titleSuffix: Text Analytics -  Azure Cognitive Services
description: Развертывание контейнера распознавания языка с работающим примером в Службе Azure Kubernetes и его тестирование в веб-браузере.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 6918218d8434c06f59b0738e60cad53b94b0a0b5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98939836"
---
# <a name="deploy-the-text-analytics-language-detection-container-to-azure-kubernetes-service"></a>Развертывание контейнера обнаружения языка Анализ текста в службе Kubernetes Azure

Узнайте, как развертывать контейнер распознавания языка. В статье описано, как создавать локальные контейнеры Docker, отправлять их в закрытый реестр контейнеров и запускать в кластере Kubernetes, а также тестировать в веб-браузере.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этой процедуры необходимо установить и запустить несколько средств локально. Не используйте Azure CloudShell.

* Используйте подписку Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services), прежде чем начинать работу.
* [Git](https://git-scm.com/downloads) для вашей операционной системы, чтобы клонировать [пример](https://github.com/Azure-Samples/cognitive-services-containers-samples) для этой процедуры.
* [Azure CLI](/cli/azure/install-azure-cli).
* [Модуль Docker](https://www.docker.com/products/docker-engine), для которого нужно проверить работоспособность Docker CLI в окне консоли.
* [kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.13.1/bin/windows/amd64/kubectl.exe).
* Ресурс Azure с правильной ценовой категорией. Не все ценовые категории поддерживают этот контейнер.
  * Ресурс **Анализ текста** с ценовой категорией F0 или "Стандартный".
  * Ресурс **Cognitive Services** с ценовой категорией S0.

## <a name="running-the-sample"></a>Выполнение примера

Эта процедура загружает и выполняет пример контейнера Cognitive Services для распознавания языка. Пример содержит два контейнера: один для клиентского приложения и второй для контейнера Cognitive Services. Мы будем отправлять оба этих образа в реестр контейнеров Azure. Разместив их в собственном реестре, создайте Службу Azure Kubernetes для доступа к образам и выполнения контейнеров. Когда контейнеры будут запущены, используйте интерфейс командной строки **kubectl** для контроля производительности контейнеров. Создайте HTTP-запрос к клиентскому приложению и проверьте результаты его выполнения.

![Концепция запуска примеров контейнеров](../text-analytics/media/how-tos/container-instance-sample/containers.png)

## <a name="the-sample-containers"></a>Примеры контейнеров

Этот пример содержит два образа контейнеров, один из которых отвечает за интерфейсный веб-сайт. Второй образ — это контейнер распознавания языка, который возвращает сведения о языке (региональных параметрах), обнаруженном для текста. Когда вы завершите работу, оба эти контейнера будут доступны по внешнему IP-адресу.

### <a name="the-language-frontend-container"></a>Контейнер интерфейса для распознавания языка

Этот веб-сайт выполняет те же функции, что и клиентское приложение, то есть направляет запросы к конечной точки распознавания языка. Когда процедура будет завершена, вы сможете определить язык для строки символов, обращаясь к контейнеру веб-сайта через браузер по адресу `http://<external-IP>/<text-to-analyze>`. Примером такого URL-адреса является `http://132.12.23.255/helloworld!`. В браузере отображается результат `English`.

### <a name="the-language-container"></a>Контейнер распознавания языка

В нашем примере контейнер распознавания языка доступен для любых внешних запросов. Мы не изменяли этот контейнер, а значит в нем применяется стандартный API распознавания языка Cognitive Services.

Для этого используется запрос POST для распознавания языка. Как и для других контейнеров Cognitive Services, вы можете получить дополнительные сведения, используя размещенную информацию Swagger: `http://<external-IP>:5000/swagger/index.html`.

Порт 5000 используется по умолчанию со всеми контейнерами Cognitive Services.

## <a name="create-azure-container-registry-service"></a>Создание службы Реестра контейнеров Azure

Чтобы развернуть контейнер в Службе Azure Kubernetes, должны быть доступны образы контейнеров. Создайте собственную службу Реестра контейнеров Azure для размещения образов.

1. Вход в Azure CLI

    ```azurecli-interactive
    az login
    ```

1. Создайте группу ресурсов с именем `cogserv-container-rg`, в которой будут размещены все созданные в этой процедуре ресурсы.

    ```azurecli-interactive
    az group create --name cogserv-container-rg --location westus
    ```

1. Создайте Реестр контейнеров Azure, указав для него имя с постфиксом `registry`, например `pattyregistry`. Не используйте в имени символы дефиса или подчеркивания.

    ```azurecli-interactive
    az acr create --resource-group cogserv-container-rg --name pattyregistry --sku Basic
    ```

    Сохраните результаты, чтобы получить значение свойства **loginServer**. Оно будет частью адреса для размещенного контейнера, который вы позднее будете использовать в файле `language.yml`.

    ```azurecli-interactive
    az acr create --resource-group cogserv-container-rg --name pattyregistry --sku Basic
    ```

    ```output
    {
        "adminUserEnabled": false,
        "creationDate": "2019-01-02T23:49:53.783549+00:00",
        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cogserv-container-rg/providers/Microsoft.ContainerRegistry/registries/pattyregistry",
        "location": "westus",
        "loginServer": "pattyregistry.azurecr.io",
        "name": "pattyregistry",
        "provisioningState": "Succeeded",
        "resourceGroup": "cogserv-container-rg",
        "sku": {
            "name": "Basic",
            "tier": "Basic"
        },
        "status": null,
        "storageAccount": null,
        "tags": {},
        "type": "Microsoft.ContainerRegistry/registries"
    }
    ```

1. Войдите в реестр контейнеров Необходимо войти в систему, чтобы публиковать образы в реестр.

    ```azurecli-interactive
    az acr login --name pattyregistry
    ```

## <a name="get-website-docker-image"></a>Получение образа Docker для веб-сайта

1. Пример кода, используемый в этой процедуре, размещен в репозитории с примерами контейнеров Cognitive Services. Клонируйте этот репозиторий, чтобы получить локальную копию примера.

    ```console
    git clone https://github.com/Azure-Samples/cognitive-services-containers-samples
    ```

    Разместив репозиторий на локальном компьютере, найдите веб-сайт в каталоге [\dotnet\Language\FrontendService](https://github.com/Azure-Samples/cognitive-services-containers-samples/tree/master/dotnet/Language/FrontendService). Этот веб-сайт выступает в качестве клиентского приложения, то есть вызывает API распознавания языка, размещенный в контейнере распознавания языка.  

1. Создайте образ Docker для этого веб-сайта. Убедитесь, что в консоли открыт каталог [\FrontendService](https://github.com/Azure-Samples/cognitive-services-containers-samples/tree/master/dotnet/Language/FrontendService), в котором находится Dockerfile, и выполните следующую команду:

    ```console
    docker build -t language-frontend -t pattiyregistry.azurecr.io/language-frontend:v1 .
    ```

    Чтобы отслеживать версию реестра контейнеров, добавьте тег в формате версии, например `v1`.

1. Отправьте образ в реестр контейнеров. Это может занять несколько минут.

    ```console
    docker push pattyregistry.azurecr.io/language-frontend:v1
    ```

    Если вы получаете ошибку `unauthorized: authentication required`, выполните вход с помощью команды `az acr login --name <your-container-registry-name>`. 

    Когда этот процесс завершится, результаты должны выглядеть примерно так:

    ```output
    The push refers to repository [pattyregistry.azurecr.io/language-frontend]
    82ff52ee6c73: Pushed
    07599c047227: Pushed
    816caf41a9a1: Pushed
    2924be3aed17: Pushed
    45b83a23806f: Pushed
    ef68f6734aa4: Pushed
    v1: digest: sha256:31930445deee181605c0cde53dab5a104528dc1ff57e5b3b34324f0d8a0eb286 size: 1580
    ```

## <a name="get-language-detection-docker-image"></a>Получение образа Docker для распознавания языка

1. Вытяните последнюю версию образа Docker на локальный компьютер. Это может занять несколько минут. Если существует более новая версия этого контейнера, замените значение `1.1.006770001-amd64-preview` номером новой версии.

    ```console
    docker pull mcr.microsoft.com/azure-cognitive-services/language:1.1.006770001-amd64-preview
    ```

1. Присвойте образу тег реестра контейнеров. Проверьте номер последней версии и подставьте его вместо `1.1.006770001-amd64-preview`, если он уже изменился. 

    ```console
    docker tag mcr.microsoft.com/azure-cognitive-services/language pattiyregistry.azurecr.io/language:1.1.006770001-amd64-preview
    ```

1. Отправьте образ в реестр контейнеров. Это может занять несколько минут.

    ```console
    docker push pattyregistry.azurecr.io/language:1.1.006770001-amd64-preview
    ```

## <a name="get-container-registry-credentials"></a>Получение учетных данных Реестра контейнеров

Следующие шаги позволяют получить сведения, необходимые для подключения реестр контейнеров к Службе Azure Kubernetes, которую вы создадите далее в этой процедуре.

1. Создайте субъект-службу.

    ```azurecli-interactive
    az ad sp create-for-rbac --skip-assignment
    ```

    Сохраните из результата значение `appId`, которое вы присвоите параметру `<appId>` (уполномоченный объект) на шаге 3. Сохраните значение `password`, которое вы присвоите параметру `<client-secret>` (секрет клиента) в следующем разделе.

    ```output
    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "azure-cli-2018-12-31-18-39-32",
      "name": "http://azure-cli-2018-12-31-18-39-32",
      "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. Получите идентификатор реестра контейнеров.

    ```azurecli-interactive
    az acr show --resource-group cogserv-container-rg --name pattyregistry --query "id" --o table
    ```

    Сохраните результат, чтобы присвоить его параметру `<acrId>` (область действия) на следующем шаге. Он выглядит следующим образом:

    ```output
    /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/cogserv-container-rg/providers/Microsoft.ContainerRegistry/registries/pattyregistry
    ```

    Сохраните полное значение для шага 3 в этом разделе.

1. Чтобы правильно предоставить кластеру AKS права доступа для использования образов, хранящихся в реестре контейнеров, создайте назначение роли. Замените `<appId>` и `<acrId>` значениями, полученными на двух предыдущих шагах.

    ```azurecli-interactive
    az role assignment create --assignee <appId> --scope <acrId> --role Reader
    ```

## <a name="create-azure-kubernetes-service"></a>Создание Службы Azure Kubernetes

1. Создание кластера Kubernetes. Все значения параметров, кроме имени взяты из предыдущих разделов. Выберите имя, которое описывает понять, кто и зачем его создал, например `patty-kube`.

    ```azurecli-interactive
    az aks create --resource-group cogserv-container-rg --name patty-kube --node-count 2  --service-principal <appId>  --client-secret <client-secret>  --generate-ssh-keys
    ```

    Этот шаг может занять несколько минут. Результат:

    ```output
    {
      "aadProfile": null,
      "addonProfiles": null,
      "agentPoolProfiles": [
        {
          "count": 2,
          "dnsPrefix": null,
          "fqdn": null,
          "maxPods": 110,
          "name": "nodepool1",
          "osDiskSizeGb": 30,
          "osType": "Linux",
          "ports": null,
          "storageProfile": "ManagedDisks",
          "vmSize": "Standard_DS1_v2",
          "vnetSubnetId": null
        }
      ],
      "dnsPrefix": "patty-kube--65a101",
      "enableRbac": true,
      "fqdn": "patty-kube--65a101-341f1f54.hcp.westus.azmk8s.io",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/cogserv-container-rg/providers/Microsoft.ContainerService/managedClusters/patty-kube",
      "kubernetesVersion": "1.9.11",
      "linuxProfile": {
        "adminUsername": "azureuser",
        "ssh": {
          "publicKeys": [
            {
              "keyData": "ssh-rsa AAAAB3NzaC...ohR2d81mFC
            }
          ]
        }
      },
      "location": "westus",
      "name": "patty-kube",
      "networkProfile": {
        "dnsServiceIp": "10.0.0.10",
        "dockerBridgeCidr": "172.17.0.1/16",
        "networkPlugin": "kubenet",
        "networkPolicy": null,
        "podCidr": "10.244.0.0/16",
        "serviceCidr": "10.0.0.0/16"
      },
      "nodeResourceGroup": "MC_patty_westus",
      "provisioningState": "Succeeded",
      "resourceGroup": "cogserv-container-rg",
      "servicePrincipalProfile": {
        "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "keyVaultSecretRef": null,
        "secret": null
      },
      "tags": null,
      "type": "Microsoft.ContainerService/ManagedClusters"
    }
    ```

    Создается служба, которая пока не содержит контейнер веб-сайта или контейнер распознавания языка.  

1. Получите учетные данные кластера Kubernetes.

    ```azurecli-interactive
    az aks get-credentials --resource-group cogserv-container-rg --name patty-kube
    ```

## <a name="load-the-orchestration-definition-into-your-kubernetes-service"></a>Загрузка определения оркестрации в службу Kubernetes

В этом разделе используется интерфейс командной строки **kubectl** для взаимодействия со Службой Azure Kubernetes.

1. Перед загрузкой определения оркестрации убедитесь, что **kubectl** имеет доступ к узлам.

    ```console
    kubectl get nodes
    ```

    Ответ выглядит следующим образом:

    ```output
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-nodepool1-13756812-0   Ready     agent     6m        v1.9.11
    aks-nodepool1-13756812-1   Ready     agent     6m        v1.9.11
    ```

1. Скопируйте следующий файл и присвойте ему имя `language.yml`. Этот файл имеет раздел `service` и раздел `deployment` для контейнеров соответствующих типов: контейнер веб-сайта `language-frontend` и контейнер распознавания `language`.

    [!code-yml[Kubernetes orchestration file for the Cognitive Services containers sample](~/samples-cogserv-containers/Kubernetes/language/language.yml "Kubernetes orchestration file for the Cognitive Services containers sample")]

1. Измените строки развертывания интерфейса распознавания языка в файле `language.yml`, используя данные из следующей таблицы, чтобы добавить имена образов реестров контейнеров, секрет клиента и параметры анализа текста.

    Параметры развертывания интерфейса распознавания языка|Назначение|
    |--|--|
    |Строка 32<br> Свойство`image`|Расположение образа интерфейса в Реестре контейнеров<br>`<container-registry-name>.azurecr.io/language-frontend:v1`|
    |Строка 44<br> Свойство`name`|Секрет Реестра контейнера для образа, который указан в параметре `<client-secret>` в предыдущем разделе.|

1. Измените строки развертывания распознавания языка в файле `language.yml`, используя данные из следующей таблицы, чтобы добавить имена образов реестров контейнеров, секрет клиента и параметры анализа текста.

    |Параметры развертывания распознавания языка|Назначение|
    |--|--|
    |Строка 78<br> Свойство`image`|Расположение образа распознавания языка в Реестре контейнеров<br>`<container-registry-name>.azurecr.io/language:1.1.006770001-amd64-preview`|
    |Строка 95<br> Свойство`name`|Секрет Реестра контейнера для образа, который указан в параметре `<client-secret>` в предыдущем разделе.|
    |Строка 91<br> Свойство`apiKey`|Ключ ресурса анализа текста|
    |Строка 92<br> Свойство`billing`|Конечная точка выставления счетов для ресурса анализа текста.<br>`https://westus.api.cognitive.microsoft.com/text/analytics/v2.1`|

    Параметр **apiKey** и **конечная точка выставления счетов** задаются в определении оркестрации Kubernetes, поэтому контейнеру веб-сайта не нужно знать их значения или передавать их в запросе. Контейнер веб-сайта обращается к контейнеру распознавания языка по имени оркестратора `language`.

1. Загрузите файл определения оркестрации для этого примера из той папки, в которой вы создали и сохранили `language.yml`.

    ```console
    kubectl apply -f language.yml
    ```

    Ответ:

    ```output
    service "language-frontend" created
    deployment.apps "language-frontend" created
    service "language" created
    deployment.apps "language" created
    ```

## <a name="get-external-ips-of-containers"></a>Получение внешних IP-адресов контейнеров

Убедитесь, что для обоих контейнеров запущены службы `language-frontend` и `language`, а затем получите их внешний IP-адрес.

```console
kubectl get all
```

```output
NAME                                     READY     STATUS    RESTARTS   AGE
pod/language-586849d8dc-7zvz5            1/1       Running   0          13h
pod/language-frontend-68b9969969-bz9bg   1/1       Running   1          13h

NAME                        TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
service/kubernetes          ClusterIP      10.0.0.1      <none>          443/TCP          14h
service/language            LoadBalancer   10.0.39.169   104.42.172.68   5000:30161/TCP   13h
service/language-frontend   LoadBalancer   10.0.42.136   104.42.37.219   80:30943/TCP     13h

NAME                                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/language            1         1         1            1           13h
deployment.extensions/language-frontend   1         1         1            1           13h

NAME                                                 DESIRED   CURRENT   READY     AGE
replicaset.extensions/language-586849d8dc            1         1         1         13h
replicaset.extensions/language-frontend-68b9969969   1         1         1         13h

NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/language            1         1         1            1           13h
deployment.apps/language-frontend   1         1         1            1           13h

NAME                                           DESIRED   CURRENT   READY     AGE
replicaset.apps/language-586849d8dc            1         1         1         13h
replicaset.apps/language-frontend-68b9969969   1         1         1         13h
```

Если для службы `EXTERNAL-IP` имеет значение pending, выполните ту же команду еще один или несколько раз, пока не получите значение IP-адреса, прежде чем переходить к следующему шагу.

## <a name="test-the-language-detection-container"></a>Тестирование контейнера распознавания языка

Откройте браузер и перейдите по внешнему IP-адресу контейнера `language`, который вы получили в предыдущем разделе: `http://<external-ip>:5000/swagger/index.html`. Для проверки конечной точки распознавания языка вы можете использовать функцию API `Try it`.

![Просмотр документации Swagger для контейнера](../text-analytics/media/how-tos/container-instance-sample/language-detection-container-swagger-documentation.png)

## <a name="test-the-client-application-container"></a>Тестирование контейнера с клиентским приложением

В адресную строку браузера введите внешний IP-адрес контейнера `language-frontend`, используя следующий формат: `http://<external-ip>/helloworld`. Для текста `helloworld` распознается английский язык: `English`.

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда вы закончите работу с кластером, удалите группу ресурсов Azure.

```azurecli-interactive
az group delete --name cogserv-container-rg
```

## <a name="related-information"></a>Дополнительные сведения

* [kubectl для пользователей Docker](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Контейнеры Cognitive Services](../cognitive-services-container-support.md)
