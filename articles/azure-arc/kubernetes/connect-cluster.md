---
title: Подключение кластера Kubernetes с поддержкой дуги Azure (Предварительная версия)
services: azure-arc
ms.service: azure-arc
ms.date: 02/09/2021
ms.topic: article
author: mlearned
ms.author: mlearned
description: Подключение кластера Kubernetes с поддержкой дуги Azure с помощью дуги Azure
keywords: Kubernetes, Arc, Azure, K8s, контейнеры
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: e68eccf998592aa7d1ebfea51e4ca66d577b3c7f
ms.sourcegitcommit: d4734bc680ea221ea80fdea67859d6d32241aefc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2021
ms.locfileid: "100390561"
---
# <a name="connect-an-azure-arc-enabled-kubernetes-cluster-preview"></a>Подключение кластера Kubernetes с поддержкой дуги Azure (Предварительная версия)

В этой статье описывается процесс подключения любого Kubernetes кластера, сертифицированного для машинного кода (КНКФ), например, AKS Engine в Azure, AKS-Engine в центре Azure Stack, ГКЕ, ЕКС и VMware vSphere кластера в дугу Azure.

## <a name="before-you-begin"></a>Перед началом

Убедитесь, что вы подготовили следующие необходимые компоненты:

* Работающий кластер Kubernetes. Если у вас нет кластера Kubernetes, вы можете создать тестовый кластер с помощью одного из следующих руководств:
  * Создание кластера Kubernetes с помощью [Kubernetes в DOCKER (Kind)](https://kind.sigs.k8s.io/).
  * Создайте кластер Kubernetes с помощью DOCKER для [Mac](https://docs.docker.com/docker-for-mac/#kubernetes) или [Windows](https://docs.docker.com/docker-for-windows/#kubernetes).
* Kubeconfig-файл для доступа к кластеру и роли администратора кластера в кластере для развертывания с помощью Arc с включенными агентами Kubernetes.
* Пользователь или субъект-служба, которые будут указаны в командах `az login` и `az connectedk8s connect`, имеют разрешения на чтение и запись для типа ресурса Microsoft.Kubernetes/connectedclusters. Роль "кластер Kubernetes с подключением ARC в Azure" имеет эти разрешения и может использоваться для назначений ролей пользователя или субъекта-службы.
* Helm 3 для адаптации кластера с помощью расширения connectedk8s. [Установите последнюю версию Helm 3](https://helm.sh/docs/intro/install) для удовлетворения этого требования.
* Azure CLI версии 2.15 + для установки расширений CLI Kubernetes для службы "Дуга Azure". [Установите Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true) или обновите последнюю версию.
* Установите расширения CLI Kubernetes с поддержкой ARC:
  
  * Установите расширение `connectedk8s`, которое поможет подключить кластеры Kubernetes к Azure.
  
  ```azurecli
  az extension add --name connectedk8s
  ```
  
   * Установите расширение `k8sconfiguration`:
  
  ```azurecli
  az extension add --name k8sconfiguration
  ```

  * Если вы хотите обновить эти расширения позже, выполните следующие команды:
  
  ```azurecli
  az extension update --name connectedk8s
  az extension update --name k8sconfiguration
  ```

## <a name="supported-regions"></a>Поддерживаемые регионы

* Восточная часть США
* Западная Европа

## <a name="network-requirements"></a>Требования к сети

Для работы агентов Azure Arc требуются следующие протоколы, порты и исходящие URL-адреса:

* TCP через порт 443: `https://:443`
* TCP через порт 9418: `git://:9418`

| Конечная точка (DNS)                                                                                               | Описание                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `https://management.azure.com`                                                                                 | Требуется для подключения агента к Azure и регистрации кластера.                                                        |
| `https://eastus.dp.kubernetesconfiguration.azure.com`, `https://westeurope.dp.kubernetesconfiguration.azure.com` | Конечная точка плоскости данных для агента, чтобы отправить сведения о состоянии и получить информацию о конфигурации.                                      |
| `https://login.microsoftonline.com`                                                                            | Требуется для выборки и обновления маркеров Azure Resource Manager.                                                                                    |
| `https://mcr.microsoft.com`                                                                            | Требуется для извлечения образов контейнеров для агентов Arc Azure.                                                                  |
| `https://eus.his.arc.azure.com`, `https://weu.his.arc.azure.com`                                                                            |  Требуется для извлечения назначенных системой сертификатов управляемых удостоверений.                                                                  |

## <a name="register-the-two-providers-for-azure-arc-enabled-kubernetes"></a>Регистрация двух поставщиков для Kubernetes с поддержкой Azure Arc:

```console
az provider register --namespace Microsoft.Kubernetes

az provider register --namespace Microsoft.KubernetesConfiguration
```

Регистрация — это асинхронный процесс, который может занять около 10 минут. Процесс регистрации можно контролировать, выполняя следующие команды:

```console
az provider show -n Microsoft.Kubernetes -o table
```

```console
az provider show -n Microsoft.KubernetesConfiguration -o table
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группу ресурсов мы будем использовать для хранения метаданных кластера.

Для начала создайте группу ресурсов, где будет размещен ресурс подключенного кластера.

```console
az group create --name AzureArcTest -l EastUS -o table
```

**Выходные данные:**

```console
Location    Name
----------  ------------
eastus      AzureArcTest
```

## <a name="connect-a-cluster"></a>Подключение кластера

Далее мы подключим наш кластер Kubernetes к Azure, используя `az connectedk8s connect` :

1. Проверьте подключение к кластеру Kubernetes одним из следующих способов:
   * `KUBECONFIG`
   * `~/.kube/config`
   * `--kube-config`
1. Разверните агенты Arc Azure для Kubernetes, используя Helm 3 в `azure-arc` пространстве имен:

    ```console
    az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
    ```

**Выходные данные:**

```console
Command group 'connectedk8s' is in preview. It may be changed/removed in a future release.
Helm release deployment succeeded

{
  "aadProfile": {
    "clientAppId": "",
    "serverAppId": "",
    "tenantId": ""
  },
  "agentPublicKeyCertificate": "...",
  "agentVersion": "0.1.0",
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1",
  "identity": {
    "principalId": null,
    "tenantId": null,
    "type": "None"
  },
  "kubernetesVersion": "v1.15.0",
  "location": "eastus",
  "name": "AzureArcTest1",
  "resourceGroup": "AzureArcTest",
  "tags": {},
  "totalNodeCount": 1,
  "type": "Microsoft.Kubernetes/connectedClusters"
}
```

## <a name="verify-connected-cluster"></a>Проверка подключенного кластера

Чтобы получить список подключенных кластеров, используйте следующую команду:

```console
az connectedk8s list -g AzureArcTest -o table
```

**Выходные данные:**

```console
Command group 'connectedk8s' is in preview. It may be changed/removed in a future release.
Name           Location    ResourceGroup
-------------  ----------  ---------------
AzureArcTest1  eastus      AzureArcTest
```

Этот ресурс также можно просмотреть на [портал Azure](https://portal.azure.com/). Откройте портал в браузере и перейдите к ресурсу группы ресурсов и Kubernetes в службе "Дуга Azure" на основе имен ресурсов и имен групп ресурсов, которые использовались ранее в `az connectedk8s connect` команде.  
> [!NOTE]
> После подключения кластера потребуется от 5 до 10 минут для метаданных кластера (версия кластера, версия агента, число узлов и т. д.) на странице обзора ресурса Kubernetes в службе "Дуга Azure", включенной в портал Azure.

## <a name="connect-using-an-outbound-proxy-server"></a>Подключение с использованием исходящего прокси-сервера

Если кластер находится за исходящим прокси-сервером, Azure CLI и Kubernetes агенты должны маршрутизировать запросы через исходящий прокси-сервер:

1. Проверьте версию расширения, `connectedk8s` установленного на компьютере:

    ```console
    az -v
    ```

    `connectedk8s`Для настройки агентов с исходящим прокси-сервером требуется расширение 0.2.5 +. Если на компьютере установлена версия 0.2.3 или более старая, выполните инструкции по [обновлению](#before-you-begin) , чтобы получить последнюю версию расширения на компьютере.

2. Задайте переменные среды, необходимые для Azure CLI для использования исходящего прокси-сервера:

    * Если вы используете bash, выполните следующую команду с соответствующими значениями:

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

3. Выполните команду Connect с указанными параметрами прокси-сервера:

    ```console
    az connectedk8s connect -n <cluster-name> -g <resource-group> --proxy-https https://<proxy-server-ip-address>:<port> --proxy-http http://<proxy-server-ip-address>:<port> --proxy-skip-range <excludedIP>,<excludedCIDR> --proxy-cert <path-to-cert-file>
    ```

> [!NOTE]
> * Указание `excludedCIDR` в разделе `--proxy-skip-range` важно для того, чтобы обеспечить передачу данных в кластере для агентов не разбивается.
> * Несмотря на то, что `--proxy-http` `--proxy-https` `--proxy-skip-range` для большинства сред исходящих прокси-серверов ожидается, и, `--proxy-cert` требуется только в том случае, если доверенные сертификаты от прокси-сервера необходимо внедрить в хранилище доверенных сертификатов модулей Pod агента.
> * Приведенная выше спецификация прокси-сервера в настоящее время применяется только для агентов Arc, а не для модулей Flux Pod, используемых в Саурцеконтролконфигуратион. Группа Kubernetes с поддержкой дуги активно работает над этой функцией, и вскоре она будет доступна.

## <a name="azure-arc-agents-for-kubernetes"></a>Агенты Azure Arc для Kubernetes

Kubernetes с поддержкой Azure Arc развертывает в пространстве имен `azure-arc` ряд операторов. Для просмотра этих развертываний и модулей Pod можно использовать:

```console
kubectl -n azure-arc get deployments,pods
```

**Выходные данные:**

```console
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

Kubernetes с поддержкой Azure Arc состоит из нескольких агентов (операторов), которые выполняются в кластере, развернутом в пространстве имен `azure-arc`.

| Агенты (операторы)                                                                                               | Описание                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `deployment.apps/config-agent`                                                                                 | Наблюдает за подключенным кластером для ресурсов конфигурации системы управления версиями, применяемых в кластере, и обновляет состояние соответствия.                                                        |
| `deployment.apps/controller-manager` | Оператор операторов, который управляет взаимодействием между компонентами дуги Azure.                                      |
| `deployment.apps/metrics-agent`                                                                            | Собирает метрики производительности других агентов ARC.                                                                                    |
| `deployment.apps/cluster-metadata-operator`                                                                            | Собирает метаданные кластера, такие как версия кластера, число узлов и версия агента Arc Azure.                                                                  |
| `deployment.apps/resource-sync-agent`                                                                            |  Синхронизирует упомянутые выше метаданные кластера с Azure.                                                                  |
| `deployment.apps/clusteridentityoperator`                                                                            |  В настоящее время Kubernetes с поддержкой дуги Azure поддерживает назначенное системой удостоверение. `clusteridentityoperator` хранит сертификат управляемого удостоверения службы (MSI), используемый другими агентами для взаимодействия с Azure.                                                                  |
| `deployment.apps/flux-logs-agent`                                                                            |  Собирает журналы из операторов Flux, развернутых в составе конфигурации системы управления версиями.                                                                  |

## <a name="delete-a-connected-cluster"></a>Удаление подключенного кластера

Ресурс `Microsoft.Kubernetes/connectedcluster` можно удалить с помощью Azure CLI или портала Azure.


* **Удаление с помощью Azure CLI**: выполните следующую команду Azure CLI, чтобы инициировать удаление ресурса Kubernetes с включенной службой "Дуга Azure".
  ```console
  az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
  ```
  Эта команда удаляет `Microsoft.Kubernetes/connectedCluster` ресурс и все связанные `sourcecontrolconfiguration` с ним ресурсы в Azure. Azure CLI использует `helm uninstall` для удаления агентов, выполняющихся в кластере.

* **Удаление портал Azure**: Удаление ресурса Kubernetes для службы "Дуга Azure" на портал Azure удаление `Microsoft.Kubernetes/connectedcluster` ресурса и всех связанных `sourcecontrolconfiguration` с ним ресурсов в Azure, но *не* удаление агентов, выполняющихся в кластере. 

  Чтобы удалить агенты, запущенные в кластере, выполните следующую команду:

  ```console
  az connectedk8s delete --name AzureArcTest1 --resource-group AzureArcTest
  ```

## <a name="next-steps"></a>Дальнейшие действия

* [Использование GitOps в подключенном кластере](./use-gitops-connected-cluster.md)
* [Использование Политики Azure для управления конфигурацией кластера](./use-azure-policy.md)
