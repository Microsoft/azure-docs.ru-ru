---
title: Краткое руководство. Подключение существующего кластера Kubernetes к Azure Arc
description: В этом кратком руководстве описывается, как подключить кластер Kubernetes с поддержкой Azure Arc.
author: mgoedtel
ms.author: magoedte
ms.service: azure-arc
ms.topic: quickstart
ms.date: 02/24/2021
ms.custom: template-quickstart
keywords: Kubernetes, Arc, Azure, кластер
ms.openlocfilehash: c50266ce0afd6dd3f5860e3259d2b22af817834c
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101689255"
---
# <a name="quickstart-connect-an-existing-kubernetes-cluster-to-azure-arc"></a>Краткое руководство. Подключение существующего кластера Kubernetes к Azure Arc 

В этом кратком руководстве рассказывается, как воспользоваться возможностями кластера Kubernetes с поддержкой Azure Arc и подключить существующий кластер Kubernetes к службе Azure Arc. Основы подключения кластеров к Azure Arc см. в статье [Архитектура агентов Kubernetes с поддержкой Azure Arc](./conceptual-agent-architecture.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

* Убедитесь, что у вас есть следующее:
    * Работающий кластер Kubernetes.
    * Файл `kubeconfig`, указывающий на кластер, который необходимо подключить к службе Azure Arc.
    * Разрешения на чтение и запись для пользователя или субъекта-службы, который создает тип ресурса Kubernetes с поддержкой Azure Arc (`Microsoft.Kubernetes/connectedClusters`).
* Установите [последнюю версию Helm 3](https://helm.sh/docs/intro/install).
* Установите следующие расширения CLI для Kubernetes с поддержкой Azure Arc версии не ниже 1.0.0:
  
  ```azurecli
  az extension add --name connectedk8s
  az extension add --name k8s-configuration
  ```
  * Чтобы обновить эти расширения до последних версий, выполните следующие команды:
  
  ```azurecli
  az extension update --name connectedk8s
  az extension update --name k8s-configuration
  ```

>[!NOTE]
>**Поддерживаемые регионы:**
>* Восточная часть США
>* Западная Европа
>* центрально-западная часть США
>* Центрально-южная часть США
>* Юго-Восточная Азия
>* южная часть Соединенного Королевства
>* Западная часть США 2
>* Восточная Австралия
>* восточная часть США 2
>* Северная Европа

## <a name="meet-network-requirements"></a>Выполнение требований к сети

>[!IMPORTANT]
>Для работы агентов Azure Arc требуются следующие протоколы, порты и исходящие URL-адреса:
>* TCP на порту 443 (`https://:443`).
>* TCP на порту 9418 (`git://:9418`).
  
| Конечная точка (DNS) | Описание |  
| ----------------- | ------------- |  
| `https://management.azure.com`                                                                                 | Требуется для подключения агента к Azure и регистрации кластера.                                                        |  
| `https://eastus.dp.kubernetesconfiguration.azure.com`, `https://westeurope.dp.kubernetesconfiguration.azure.com`, `https://westcentralus.dp.kubernetesconfiguration.azure.com`, `https://southcentralus.dp.kubernetesconfiguration.azure.com`, `https://southeastasia.dp.kubernetesconfiguration.azure.com`, `https://uksouth.dp.kubernetesconfiguration.azure.com`, `https://westus2.dp.kubernetesconfiguration.azure.com`, `https://australiaeast.dp.kubernetesconfiguration.azure.com`, `https://eastus2.dp.kubernetesconfiguration.azure.com`, `https://northeurope.dp.kubernetesconfiguration.azure.com` | Конечная точка плоскости данных, через которую агент будет отправлять сведения о состоянии и извлекать сведения о конфигурации.                                      |  
| `https://login.microsoftonline.com`                                                                            | Требуется для извлечения и обновления маркеров Azure Resource Manager.                                                                                    |  
| `https://mcr.microsoft.com`                                                                            | Требуется агентам Azure Arc для извлечения образов контейнеров.                                                                  |  
| `https://eus.his.arc.azure.com`, `https://weu.his.arc.azure.com`, `https://wcus.his.arc.azure.com`, `https://scus.his.arc.azure.com`, `https://sea.his.arc.azure.com`, `https://uks.his.arc.azure.com`, `https://wus2.his.arc.azure.com`, `https://ae.his.arc.azure.com`, `https://eus2.his.arc.azure.com`, `https://ne.his.arc.azure.com` |  Требуется для получения назначенных системой сертификатов MSI (управляемого удостоверения службы).                                                                  |

## <a name="register-the-two-providers-for-azure-arc-enabled-kubernetes"></a>Регистрация двух поставщиков для Kubernetes с поддержкой Azure Arc

1. Введите следующие команды:
    ```azurecli
    az provider register --namespace Microsoft.Kubernetes
    az provider register --namespace Microsoft.KubernetesConfiguration
    ```
2. Отслеживайте ход процесса регистрации. Она может занять до 10 минут.
    ```azurecli
    az provider show -n Microsoft.Kubernetes -o table
    az provider show -n Microsoft.KubernetesConfiguration -o table    
    ```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов:  

```console
az group create --name AzureArcTest -l EastUS -o table
```

```output
Location    Name
----------  ------------
eastus      AzureArcTest
```

## <a name="connect-an-existing-kubernetes-cluster"></a>Подключение существующего кластера Kubernetes

1. Подключите кластер Kubernetes к Azure Arc с помощью следующей команды:
    ```console
    az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
    ```

    ```output
    Helm release deployment succeeded

    {
      "aadProfile": {
        "clientAppId": "",
        "serverAppId": "",
        "tenantId": ""
      },
      "agentPublicKeyCertificate": "xxxxxxxxxxxxxxxxxxx",
      "agentVersion": null,
      "connectivityStatus": "Connecting",
      "distribution": "gke",
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1",
      "identity": {
        "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "type": "SystemAssigned"
      },
      "infrastructure": "gcp",
      "kubernetesVersion": null,
      "lastConnectivityTime": null,
      "location": "eastus",
      "managedIdentityCertificateExpirationTime": null,
      "name": "AzureArcTest1",
      "offering": null,
      "provisioningState": "Succeeded",
      "resourceGroup": "AzureArcTest",
      "tags": {},
      "totalCoreCount": null,
      "totalNodeCount": null,
      "type": "Microsoft.Kubernetes/connectedClusters"
    }
    ```

> [!TIP]
> Команда, указанная выше без параметра location, создает ресурс Kubernetes с поддержкой Azure Arc в том же расположении, что и группа ресурсов. Чтобы создать ресурс Kubernetes с поддержкой Azure Arc в другом расположении, укажите `--location <region>` или `-l <region>` при выполнении команды `az connectedk8s connect`.

## <a name="verify-cluster-connection"></a>Проверка подключения к кластеру

Просмотрите список подключенных кластеров с помощью следующей команды:  

```console
az connectedk8s list -g AzureArcTest -o table
```

```output
Name           Location    ResourceGroup
-------------  ----------  ---------------
AzureArcTest1  eastus      AzureArcTest
```

> [!NOTE]
> После подключения кластера потребуется от 5 до 10 минут, чтобы метаданные этого кластера (версия кластера, версия агента, число узлов и т. д.) появились на странице обзорных сведений для ресурса Kubernetes с поддержкой Azure Arc на портале Azure.

## <a name="connect-using-an-outbound-proxy-server"></a>Подключение с использованием исходящего прокси-сервера

Если кластер расположен за исходящим прокси-сервером, Azure CLI и агенты Kubernetes с поддержкой Azure Arc должны направлять все запросы через этот прокси-сервер. 


1. Задайте переменные среды, которые нужны Azure CLI для использования исходящего прокси-сервера:

    * Если вы используете оболочку bash, выполните следующую команду с соответствующими значениями:

        ```bash
        export HTTP_PROXY=<proxy-server-ip-address>:<port>
        export HTTPS_PROXY=<proxy-server-ip-address>:<port>
        export NO_PROXY=<cluster-apiserver-ip-address>:<port>
        ```

    * Если вы используете PowerShell, выполните следующую команду с соответствующими значениями:

        ```powershell
        $Env:HTTP_PROXY = "<proxy-server-ip-address>:<port>"
        $Env:HTTPS_PROXY = "<proxy-server-ip-address>:<port>"
        $Env:NO_PROXY = "<cluster-apiserver-ip-address>:<port>"
        ```

2. Выполните команду connect с указанными параметрами прокси-сервера:

    ```console
    az connectedk8s connect -n <cluster-name> -g <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --proxy-cert <path-to-cert-file>
    ```

> [!NOTE]
> * Укажите `excludedCIDR` для параметра `--proxy-skip-range`, чтобы взаимодействие внутри кластера не нарушалось для агентов.
> * Для большинства сред с исходящим прокси-сервером ожидаются параметры `--proxy-http`, `--proxy-https` и `--proxy-skip-range`. Параметр `--proxy-cert` обязателен *только* в том случае, если вам нужно внедрять доверенные сертификаты, ожидаемые прокси-сервером, в хранилище доверенных сертификатов для модулей pod агентов.

## <a name="view-azure-arc-agents-for-kubernetes"></a>Просмотр агентов Azure Arc для Kubernetes

Kubernetes с поддержкой Azure Arc развертывает в пространстве имен `azure-arc` ряд операторов. 

1. Эти развертывания и модули pod можно просмотреть следующим образом:

    ```console
    kubectl -n azure-arc get deployments,pods
    ```

1. Убедитесь, что все модули pod находятся в состоянии `Running`.

    ```output
    NAME                                        READY      UP-TO-DATE  AVAILABLE  AGE
    deployment.apps/cluster-metadata-operator     1/1             1        1      16h
    deployment.apps/clusteridentityoperator       1/1             1        1      16h
    deployment.apps/config-agent                  1/1             1        1      16h
    deployment.apps/controller-manager            1/1             1        1      16h
    deployment.apps/flux-logs-agent               1/1             1        1      16h
    deployment.apps/metrics-agent                 1/1             1        1      16h
    deployment.apps/resource-sync-agent           1/1             1        1      16h

    NAME                                           READY    STATUS   RESTART AGE
    pod/cluster-metadata-operator-7fb54d9986-g785b  2/2     Running  0       16h
    pod/clusteridentityoperator-6d6678ffd4-tx8hr    3/3     Running  0       16h
    pod/config-agent-544c4669f9-4th92               3/3     Running  0       16h
    pod/controller-manager-fddf5c766-ftd96          3/3     Running  0       16h
    pod/flux-logs-agent-7c489f57f4-mwqqv            2/2     Running  0       16h
    pod/metrics-agent-58b765c8db-n5l7k              2/2     Running  0       16h
    pod/resource-sync-agent-5cf85976c7-522p5        3/3     Running  0       16h
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ресурс Kubernetes с поддержкой Azure Arc, все связанные с ним ресурсы конфигурации *и* все агенты, работающие в кластере, с помощью следующей команды Azure CLI:

```azurecli
az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
```

>[!NOTE]
>При удалении ресурса Kubernetes с поддержкой Azure Arc с помощью портала Azure удаляются все связанные с ним ресурсы конфигурации, но *не удаляются* агенты, работающие в кластере. Мы рекомендуем удалять ресурс Kubernetes с поддержкой Azure Arc с помощью команды `az connectedk8s delete`, а не на портале Azure.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье, чтобы узнать, как развернуть конфигурации в подключенном кластере Kubernetes с помощью GitOps.
> [!div class="nextstepaction"]
> [Развертывание конфигураций с помощью GitOps](use-gitops-connected-cluster.md)
