---
title: Устранение распространенных проблем Kubernetes с поддержкой ARC в Azure
services: azure-arc
ms.service: azure-arc
ms.date: 03/02/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Устранение распространенных проблем с кластерами Kubernetes с поддержкой Arc.
keywords: Kubernetes, Arc, Azure, контейнеры
ms.openlocfilehash: 992ea75c48b2630032e1314610986fbc610eec7b
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105025787"
---
# <a name="azure-arc-enabled-kubernetes-troubleshooting"></a>Устранение неполадок Kubernetes с поддержкой ARC в Azure

В этом документе содержатся руководства по устранению проблем с подключением, разрешениями и агентами.

## <a name="general-troubleshooting"></a>Общие действия по устранению неполадок

### <a name="azure-cli"></a>Azure CLI

Перед использованием `az connectedk8s` `az k8s-configuration` команд или интерфейса командной строки убедитесь, что Azure CLI настроена для работы с правильной подпиской Azure.

```azurecli
az account set --subscription 'subscriptionId'
az account show
```

### <a name="azure-arc-agents"></a>Агенты Arc Azure

Все агенты для Kubernetes с поддержкой Azure Arc развертываются в виде объектов pod в пространстве имен `azure-arc`. Все модули Pod должны работать и передавать их проверки работоспособности.

Сначала проверьте выпуск Azure Arc Helm.

```console
$ helm --namespace default status azure-arc
NAME: azure-arc
LAST DEPLOYED: Fri Apr  3 11:13:10 2020
NAMESPACE: default
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

Если выпуск Helm не найден или отсутствует, попробуйте [подключить кластер к службе "Дуга" Azure](./quickstart-connect-cluster.md) еще раз.

Если выпуск Helm имеется в `STATUS: deployed` , проверьте состояние агентов с помощью `kubectl` :

```console
$ kubectl -n azure-arc get deployments,pods
NAME                                       READY  UP-TO-DATE  AVAILABLE  AGE
deployment.apps/clusteridentityoperator     1/1       1          1       16h
deployment.apps/config-agent                1/1       1          1       16h
deployment.apps/cluster-metadata-operator   1/1       1          1       16h
deployment.apps/controller-manager          1/1       1          1       16h
deployment.apps/flux-logs-agent             1/1       1          1       16h
deployment.apps/metrics-agent               1/1       1          1       16h
deployment.apps/resource-sync-agent         1/1       1          1       16h

NAME                                            READY   STATUS  RESTART  AGE
pod/cluster-metadata-operator-7fb54d9986-g785b  2/2     Running  0       16h
pod/clusteridentityoperator-6d6678ffd4-tx8hr    3/3     Running  0       16h
pod/config-agent-544c4669f9-4th92               3/3     Running  0       16h
pod/controller-manager-fddf5c766-ftd96          3/3     Running  0       16h
pod/flux-logs-agent-7c489f57f4-mwqqv            2/2     Running  0       16h
pod/metrics-agent-58b765c8db-n5l7k              2/2     Running  0       16h
pod/resource-sync-agent-5cf85976c7-522p5        3/3     Running  0       16h
```

Все модули Pod должны показываться, `STATUS` как и в столбце, либо `Running` `3/3` `2/2` в нем `READY` . Извлечение журналов и описание модулей Pod, возвращающих `Error` или `CrashLoopBackOff` . В случае зависания каких либо модулей Pod в `Pending` состоянии может быть недостаточно ресурсов на узлах кластера. [Увеличение масштаба кластера](https://kubernetes.io/docs/tasks/administer-cluster/) может стать переходом к `Running` состоянию.

## <a name="connecting-kubernetes-clusters-to-azure-arc"></a>Подключение кластеров Kubernetes к службе "Дуга Azure"

Для подключения кластеров к Azure требуется как доступ к подписке Azure, так и `cluster-admin` доступ к целевому кластеру. Если вы не можете подключиться к кластеру или у вас недостаточно разрешений, подключение кластера к службе "Дуга" Azure завершится сбоем.

### <a name="insufficient-cluster-permissions"></a>Недостаточно разрешений для кластера

Если предоставленный файл kubeconfig не имеет достаточных разрешений для установки агентов Arc Azure, команда Azure CLI вернет ошибку.

```azurecli
$ az connectedk8s connect --resource-group AzureArc --name AzureArcCluster
Ensure that you have the latest helm version installed before proceeding to avoid unexpected errors.
This operation might take a while...

Error: list: failed to list: secrets is forbidden: User "myuser" cannot list resource "secrets" in API group "" at the cluster scope
```

Пользователю, подключающемуся к кластеру в службе "Дуга Azure", должна быть `cluster-admin` назначена роль в кластере.

### <a name="installation-timeouts"></a>Время ожидания установки

Подключение кластера Kubernetes к службе Arc Azure Kubernetes требует установки агентов ARC для Azure в кластере. Если кластер работает с медленным подключением к Интернету, извлечение образа контейнера для агентов может занять больше времени, чем время ожидания Azure CLI.

```azurecli
$ az connectedk8s connect --resource-group AzureArc --name AzureArcCluster
Ensure that you have the latest helm version installed before proceeding to avoid unexpected errors.
This operation might take a while...
```

### <a name="helm-issue"></a>Helm проблемы

В версии Helm возникла ошибка, при `v3.3.0-rc.1` которой установка или обновление Helm (используемое [](https://github.com/helm/helm/pull/8527) `connectedk8s` расширением CLI) приводит к запуску всех обработчиков, ведущих к следующей ошибке:

```console
$ az connectedk8s connect -n shasbakstest -g shasbakstest
Ensure that you have the latest helm version installed before proceeding.
This operation might take a while...

Please check if the azure-arc namespace was deployed and run 'kubectl get pods -n azure-arc' to check if all the pods are in running state. A possible cause for pods stuck in pending state could be insufficientresources on the kubernetes cluster to onboard to arc.
ValidationError: Unable to install helm release: Error: customresourcedefinitions.apiextensions.k8s.io "connectedclusters.arc.azure.com" not found
```

Чтобы устранить эту ошибку, выполните следующие действия.

1. Удалите ресурс Kubernetes с включенной службой "Дуга Azure" в портал Azure.
2. Выполните следующие команды на своем компьютере:
    
    ```console
    kubectl delete ns azure-arc
    kubectl delete clusterrolebinding azure-arc-operator
    kubectl delete secret sh.helm.release.v1.azure-arc.v1
    ```

3. [Установите стабильную версию](https://helm.sh/docs/intro/install/) Helm 3 на компьютере, а не версию-кандидат.
4. Выполните `az connectedk8s connect` команду с соответствующими значениями, чтобы подключить кластер к службе "Дуга Azure".

## <a name="configuration-management"></a>Управление конфигурацией

### <a name="general"></a>Общие сведения
Чтобы помочь в устранении неполадок, связанных с ресурсом конфигурации, выполните команды AZ с `--debug` указанным параметром.

```console
az provider show -n Microsoft.KubernetesConfiguration --debug
az k8s-configuration create <parameters> --debug
```

### <a name="create-configurations"></a>Создание конфигураций

Разрешения на запись в ресурсе Kubernetes с включенной службой Arc Azure ( `Microsoft.Kubernetes/connectedClusters/Write` ) необходимы и достаточно для создания конфигураций в этом кластере.

### <a name="configuration-remains-pending"></a>Конфигурация остается `Pending`

```console
kubectl -n azure-arc logs -l app.kubernetes.io/component=config-agent -c config-agent
$ k -n pending get gitconfigs.clusterconfig.azure.com  -o yaml
apiVersion: v1
items:
- apiVersion: clusterconfig.azure.com/v1beta1
  kind: GitConfig
  metadata:
    creationTimestamp: "2020-04-13T20:37:25Z"
    generation: 1
    name: pending
    namespace: pending
    resourceVersion: "10088301"
    selfLink: /apis/clusterconfig.azure.com/v1beta1/namespaces/pending/gitconfigs/pending
    uid: d9452407-ff53-4c02-9b5a-51d55e62f704
  spec:
    correlationId: ""
    deleteOperator: false
    enableHelmOperator: false
    giturl: git@github.com:slack/cluster-config.git
    helmOperatorProperties: null
    operatorClientLocation: azurearcfork8s.azurecr.io/arc-preview/fluxctl:0.1.3
    operatorInstanceName: pending
    operatorParams: '"--disable-registry-scanning"'
    operatorScope: cluster
    operatorType: flux
  status:
    configAppliedTime: "2020-04-13T20:38:43.081Z"
    isSyncedWithAzure: true
    lastPolledStatusTime: ""
    message: 'Error: {exit status 1} occurred while doing the operation : {Installing
      the operator} on the config'
    operatorPropertiesHashed: ""
    publicKey: ""
    retryCountPublicKey: 0
    status: Installing the operator
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```
## <a name="monitoring"></a>Мониторинг

Azure Monitor для контейнеров требуется, чтобы его демон был запущен в привилегированном режиме. Чтобы успешно настроить канонический Kubernetes кластер для мониторинга, выполните следующую команду:

```console
juju config kubernetes-worker allow-privileged=true
```