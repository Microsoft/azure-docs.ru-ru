---
title: Создание контроллера данных с помощью Azure Data CLI (azdata)
description: Описывается создание контроллера данных Arc Azure в типичном кластере Kubernetes с несколькими узлами, который вы уже создавали с помощью Azure Data CLI (azdata).
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 04/07/2021
ms.topic: how-to
ms.openlocfilehash: f7bc90f2748d230ad50868cff5d7a8f7b69d850a
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029545"
---
# <a name="create-azure-arc-data-controller-using-the-azure-data-cli-azdata"></a>Создание контроллера данных Azure Arc с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Ознакомьтесь с разделом [Создание контроллера данных Azure Arc](create-data-controller.md) для получения общих сведений.

Чтобы создать контроллер данных Azure Arc с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)], необходимо установить [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)].

   [Установите [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]](install-client-tools.md)

Независимо от выбранной целевой платформы перед созданием учетной записи администратора контроллера данных необходимо задать следующие переменные среды. Эти учетные данные можно предоставить другим пользователям, которым может потребоваться доступ к контроллеру данных с привилегиями администратора.

**AZDATA_USERNAME** — имя пользователя, выбранного в качестве администратора контроллера данных. Пример: `arcadmin`

**AZDATA_PASSWORD** — пароль, выбранный для пользователя контроллера данных с привилегиями администратора. Пароль должен быть не короче восьми символов и содержать символы трех из следующих четырех групп: прописные буквы, строчные буквы, цифры, специальные символы.

### <a name="linux-or-macos"></a>Linux и macOS

```console
export AZDATA_USERNAME="<your username of choice>"
export AZDATA_PASSWORD="<your password of choice>"
```

### <a name="windows-powershell"></a>Windows PowerShell

```console
$ENV:AZDATA_USERNAME="<your username of choice>"
$ENV:AZDATA_PASSWORD="<your password of choice>"
```

Перед началом создания контроллера данных Azure Arc вам нужно подключиться к кластеру Kubernetes и пройти проверку подлинности, после чего будет выбран существующий контекст Kubernetes. Способ подключения к кластеру или службе Kubernetes может быть разным. См. документацию по дистрибутиву Kubernetes или службе, которую вы используете для подключения к серверу API Kubernetes.

Вы можете убедиться в наличии текущего подключения к Kubernetes и проверить текущий контекст с помощью следующих команд.

```console
kubectl get namespace
kubectl config current-context
```

## <a name="create-the-azure-arc-data-controller"></a>Создание контроллера данных Azure Arc

> [!NOTE]
> Для параметра `--namespace` команды azdata arc dc create из следующих примеров можно применить другое значение, однако нужно использовать это имя пространства имен для `--namespace parameter` во всех других командах, приводимых далее.

- [Создание в службе Azure Kubernetes (AKS)](#create-on-azure-kubernetes-service-aks)
- [Создание в обработчике AKS в Azure Stack Hub](#create-on-aks-engine-on-azure-stack-hub)
- [Создание в AKS в Azure Stack HCI](#create-on-aks-on-azure-stack-hci)
- [Создание в Azure Red Hat OpenShift (ARO)](#create-on-azure-red-hat-openshift-aro)
- [Создание на платформе контейнеров Red Hat OpenShift (OCP)](#create-on-red-hat-openshift-container-platform-ocp)
- [Создание в вышестоящем Kubernetes с открытым кодом (kubeadm)](#create-on-open-source-upstream-kubernetes-kubeadm)
- [Создание в службе AWS Elastic Kubernetes Service (EKS)](#create-on-aws-elastic-kubernetes-service-eks)
- [Создание в службе Google Cloud Kubernetes Engine Service (GKE)](#create-on-google-cloud-kubernetes-engine-service-gke)

### <a name="create-on-azure-kubernetes-service-aks"></a>Создание в службе Azure Kubernetes (AKS)

В профиле развертывания AKS по умолчанию используется класс хранения `managed-premium`. Класс хранения `managed-premium` будет работать только при наличии виртуальных машин, развернутых с помощью образов виртуальных машин, имеющих диски класса Premium.

Если вы хотите использовать `managed-premium` в качестве класса хранения, для создания контроллера данных можно выполнить следующую. Замените заполнители в этой команде именем группы ресурсов, идентификатором подписки и расположением Azure.

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Если вы не знаете, какой класс хранения использовать, нужно применить класс хранения `default`, который поддерживается независимо от типа используемой виртуальной машины. Только этот класс хранения не обеспечивает максимальную производительность.

Если вы хотите использовать класс хранения `default`, можно выполнить следующую команду:

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

### <a name="create-on-aks-engine-on-azure-stack-hub"></a>Создание в обработчике AKS в Azure Stack Hub

В профиле развертывания по умолчанию используется класс хранения `managed-premium`. Класс хранения `managed-premium` будет работать только при наличии виртуальных машин, развернутых с помощью образов виртуальных машин, имеющих диски класса "Премиум" на Azure Stack Hub.

Вы можете выполнить следующую команду, чтобы создать контроллер данных с классом хранения "Управляемый премиум":

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Если вы не знаете, какой класс хранения использовать, нужно применить класс хранения `default`, который поддерживается независимо от типа используемой виртуальной машины. В Azure Stack Hub в основе дисков класса "Премиум" и стандартных дисков лежит одна и та же инфраструктура системы хранения. Поэтому они должны обеспечивать одинаковую общую производительность, но с разными ограничениями по операциям ввода-вывода.

Если вы хотите использовать класс хранения `default`, можно выполнить следующую команду.

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

### <a name="create-on-aks-on-azure-stack-hci"></a>Создание в AKS в Azure Stack HCI

По умолчанию профиль развертывания использует класс хранения с именем `default` и тип службы `LoadBalancer`.

Вы можете выполнить следующую команду, чтобы создать контроллер данных с классом хранения `default` и типом службы `LoadBalancer`.

```console
azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).


### <a name="create-on-azure-red-hat-openshift-aro"></a>Создание в Azure Red Hat OpenShift (ARO)

Для Azure Red Hat OpenShift требуется ограничение контекста безопасности.

#### <a name="apply-the-security-context"></a>Применение контекста безопасности

Перед созданием контроллера данных в Azure Red Hat OpenShift необходимо применить определенные ограничения контекста безопасности (SCC). В предварительной версии такое применение ослабляет ограничения безопасности. В будущих выпусках будет предоставляться обновленная версия SCC.

[!INCLUDE [apply-security-context-constraint](includes/apply-security-context-constraint.md)]

#### <a name="create-custom-deployment-profile"></a>Создание настраиваемого профиля развертывания

Для Azure RedHat Open Shift используйте профиль `azure-arc-azure-openshift`.

```console
azdata arc dc config init --source azure-arc-azure-openshift --path ./custom
```

#### <a name="create-data-controller"></a>Создание контроллера данных

Для создания контроллера данных можно выполнить следующую команду:

> [!NOTE]
> Используйте одно и то же пространство имен здесь и в приведенной ниже команде `oc adm policy add-scc-to-user`. Пример: `arc`.

```console
azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example
#azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

### <a name="create-on-red-hat-openshift-container-platform-ocp"></a>Создание на платформе контейнеров Red Hat OpenShift (OCP)

> [!NOTE]
> Рекомендуется использовать последнюю доступную версию платформы контейнеров Red Hat OpenShift в Azure.

Перед созданием контроллера данных в Red Hat OCP необходимо применить определенные ограничения контекста безопасности. 

#### <a name="apply-the-security-context-constraint"></a>Применение ограничения контекста безопасности

[!INCLUDE [apply-security-context-constraint](includes/apply-security-context-constraint.md)]

#### <a name="determine-storage-class"></a>Определение класса хранилища

Кроме того, необходимо определить, какой класс хранения следует использовать, выполнив следующую команду.

```console
kubectl get storageclass
```

#### <a name="create-custom-deployment-profile"></a>Создание настраиваемого профиля развертывания

Создайте новый файл настраиваемого профиля развертывания на основе профиля развертывания `azure-arc-openshift`, выполнив следующую команду. Эта команда создает каталог `custom` в текущем рабочем каталоге и файл настраиваемого профиля развертывания `control.json` в этом каталоге.

Используйте профиль `azure-arc-openshift` для платформы контейнеров OpenShift.

```console
azdata arc dc config init --source azure-arc-openshift --path ./custom
```

#### <a name="set-storage-class"></a>Задание класса хранения 

Теперь задайте необходимый класс хранения, заменив `<storageclassname>` в приведенной ниже команде именем класса хранения, который вы хотите использовать и который был определен при выполнении команды `kubectl get storageclass` ранее.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

#### <a name="set-loadbalancer-optional"></a>Задание балансировщика нагрузки (необязательно)

По умолчанию в профиле развертывания `azure-arc-openshift` в качестве типа службы используется `NodePort`. Если вы используете кластер OpenShift, интегрированный с подсистемой балансировки нагрузки, можно изменить конфигурацию для использования типа службы `LoadBalancer` с помощью следующей команды:

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

#### <a name="verify-security-policies"></a>Проверка политик безопасности

При использовании OpenShift вы можете продолжить работу с политиками безопасности по умолчанию в OpenShift или ввести более строгие ограничения среды, чем обычно. По выбору можно отключить некоторые функции, чтобы минимизировать уровень привилегий, необходимых во время развертывания и работы, выполнив следующие команды.

Эта команда отключает сбор метрик для модулей Pod. Если эта функция отключена, метрики для модулей Pod не отображаются на панелях мониторинга Grafana. Значение по умолчанию — true.

```console
azdata arc dc config replace -p ./custom/control.json --json-values spec.security.allowPodMetricsCollection=false
```

Эта команда отключает сбор метрик для узлов. Если эта функция отключена, метрики для узлов не отображаются на панелях мониторинга Grafana. Значение по умолчанию — true.

```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowNodeMetricsCollection=false
```

Эта команда отключает возможность создания дампов памяти для устранения неполадок.
```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowDumps=false
```

#### <a name="create-data-controller"></a>Создание контроллера данных

Теперь все готово для создания контроллера данных с помощью следующей команды.

> [!NOTE]
>   Используйте одно и то же пространство имен здесь и в приведенной ниже команде `oc adm policy add-scc-to-user`. Пример: `arc`.

> [!NOTE]
>   Параметр `--path` должен указывать на _каталог_, содержащий файл control.json, а не на сам файл control.json.


```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

### <a name="create-on-open-source-upstream-kubernetes-kubeadm"></a>Создание в вышестоящем Kubernetes с открытым кодом (kubeadm)

По умолчанию в профиле развертывания kubeadm использует класс хранения с именем `local-storage` и тип службы `NodePort`. Если это устраивает вас, можно пропустить приведенные ниже инструкции, которые задают необходимый класс хранения и тип службы, и сразу же выполнить следующую команду `azdata arc dc create`.

Если вы хотите настроить профиль развертывания, указав конкретный класс хранения и (или) тип службы, начните с создания нового файла настраиваемого профиля развертывания на основе профиля развертывания kubeadm, выполнив следующую команду. Эта команда создает каталог `custom` в текущем рабочем каталоге и файл настраиваемого профиля развертывания `control.json` в этом каталоге.

```console
azdata arc dc config init --source azure-arc-kubeadm --path ./custom
```

Доступные классы хранения можно найти, выполнив следующую команду.

```console
kubectl get storageclass
```

Теперь задайте необходимый класс хранения, заменив `<storageclassname>` в приведенной ниже команде именем класса хранения, который вы хотите использовать и который был определен при выполнении команды `kubectl get storageclass` ранее.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

По умолчанию в профиле развертывания kubeadm в качестве типа службы используется `NodePort`. Если вы используете кластер Kubernetes, интегрированный с подсистемой балансировки нагрузки, конфигурацию можно изменить с помощью следующей команды.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

Теперь все готово для создания контроллера данных с помощью следующей команды.

```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

### <a name="create-on-aws-elastic-kubernetes-service-eks"></a>Создание в службе AWS Elastic Kubernetes Service (EKS)

По умолчанию класс хранения EKS имеет значение `gp2`, а тип службы — `LoadBalancer`.

Выполните следующую команду, чтобы создать контроллер данных с помощью указанного профиля развертывания EKS.

```console
azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

### <a name="create-on-google-cloud-kubernetes-engine-service-gke"></a>Создание в службе Google Cloud Kubernetes Engine Service (GKE)

По умолчанию класс хранения GKE имеет значение `standard`, а тип службы — `LoadBalancer`.

Выполните следующую команду, чтобы создать контроллер данных с помощью указанного профиля развертывания GKE.

```console
azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

После выполнения этой команды перейдите в раздел [Наблюдение за состоянием создания](#monitoring-the-creation-status).

## <a name="monitoring-the-creation-status"></a>Мониторинг состояния создания

Создание контроллера занимает несколько минут. Вы можете отслеживать ход выполнения этой операции в другом окне терминала с помощью следующих команд:

> [!NOTE]
>  В нижеследующих примерах команд предполагается, что вы создали контроллер данных и пространство имен Kubernetes с именем `arc`. Если вы использовали другое имя пространства имен или контроллера данных, вы можете заменить `arc` использованным вами именем.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Вы также можете проверить состояние создания любого модуля Pod, выполнив команду, как показано ниже. Это особенно полезно для устранения неполадок.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Устранение неполадок при создании

Если при создании у вас возникли какие-либо проблемы, ознакомьтесь с [Руководством по устранению неполадок](troubleshoot-guide.md).