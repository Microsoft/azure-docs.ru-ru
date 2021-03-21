---
title: Создание контроллера входящего трафика с помощью существующего шлюза приложений
description: В этой статье содержатся сведения о развертывании контроллера входящего трафика шлюза приложений с помощью существующего шлюза приложений.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 2d64766c754c0ea104ae83fde799a514e9da6d68
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97693735"
---
# <a name="install-an-application-gateway-ingress-controller-agic-using-an-existing-application-gateway"></a>Установка контроллера входящего трафика шлюза приложений (АГИК) с помощью существующего шлюза приложений

Контроллер входящего трафика шлюза приложений (АГИК) — это Pod в кластере Kubernetes.
АГИК отслеживает [Входящие ресурсы Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/) и создает и применяет конфигурацию шлюза приложений в зависимости от состояния кластера Kubernetes.

## <a name="outline"></a>Обзор
- [Предварительные требования](#prerequisites)
- [Проверка подлинности Azure Resource Manager (ARM)](#azure-resource-manager-authentication)
    - Вариант 1. [Настройка AAD-Pod-Identity](#set-up-aad-pod-identity) и создание удостоверения Azure в подлокотниках
    - Вариант 2. [Использование субъекта-службы](#using-a-service-principal)
- [Установка контроллера входящего трафика с помощью Helm](#install-ingress-controller-as-a-helm-chart)
- [Шлюз приложений с несколькими кластерами и общим доступом](#multi-cluster--shared-application-gateway). Установите агик в среде, где шлюз приложений является общим для одного или нескольких кластеров AKS и (или) других компонентов Azure.

## <a name="prerequisites"></a>Предварительные условия
В этом документе предполагается, что у вас уже установлены следующие средства и инфраструктура:
- [AKS](https://azure.microsoft.com/services/kubernetes-service/) с включенной поддержкой [расширенных сетей](../aks/configure-azure-cni.md)
- [Шлюз приложений версии 2](./tutorial-autoscale-ps.md) в той же виртуальной сети, что и AKS
- [Удостоверение Pod AAD](https://github.com/Azure/aad-pod-identity) , установленное в КЛАСТЕРе AKS
- [Cloud Shell](https://shell.azure.com/) — это среда оболочки Azure, которая имеет интерфейс `az` командной строки, `kubectl` и `helm` установлен. Эти средства необходимы для приведенных ниже команд.

Перед установкой АГИК создайте __резервную копию конфигурации шлюза приложений__ :
  1. с помощью [портал Azure](https://portal.azure.com/) перейдите к своему `Application Gateway` экземпляру
  2. из `Export template` Click `Download`

Скачанный ZIP-файл будет содержать шаблоны JSON, bash и скрипты PowerShell, которые можно использовать для восстановления шлюза приложений, если это необходимо.

## <a name="install-helm"></a>Установка Helm
[Helm](../aks/kubernetes-helm.md) — это диспетчер пакетов для Kubernetes. Мы будем использовать его для установки `application-gateway-kubernetes-ingress` пакета.
Для установки Helm используйте [Cloud Shell](https://shell.azure.com/) :

1. Установите [Helm](../aks/kubernetes-helm.md) и выполните следующую команду, чтобы добавить `application-gateway-kubernetes-ingress` пакет Helm:

    - *KUBERNETES RBAC включен* Кластер AKS

    ```bash
    kubectl create serviceaccount --namespace kube-system tiller-sa
    kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller-sa
    helm init --tiller-namespace kube-system --service-account tiller-sa
    ```

    - *KUBERNETES RBAC отключен* Кластер AKS

    ```bash
    helm init
    ```

1. Добавьте репозиторий Helm AGIC:
    ```bash
    helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    helm repo update
    ```

## <a name="azure-resource-manager-authentication"></a>Проверка подлинности Azure Resource Manager

АГИК взаимодействует с сервером API Kubernetes и Azure Resource Manager. Для доступа к этим API-интерфейсам требуется удостоверение.

## <a name="set-up-aad-pod-identity"></a>Настройка удостоверения Pod для AAD

[Удостоверение типа POD AAD](https://github.com/Azure/aad-pod-identity) — это контроллер, аналогичный агик, который также работает на AKS. Он привязывает Azure Active Directory удостоверения к модулям Pod Kubernetes. Удостоверение требуется для того, чтобы приложение в Kubernetes Pod могло взаимодействовать с другими компонентами Azure. В конкретном случае для модуля АГИК требуется авторизация для выполнения HTTP-запросов к [ARM](../azure-resource-manager/management/overview.md).

Чтобы добавить этот компонент в AKS, следуйте [инструкциям по установке удостоверения Pod AAD](https://github.com/Azure/aad-pod-identity#deploy-the-azure-aad-identity-infra) .

Далее нам нужно создать удостоверение Azure и дать ему права на ARM.
Используйте [Cloud Shell](https://shell.azure.com/) , чтобы выполнить все приведенные ниже команды и создать удостоверение.

1. Создайте удостоверение Azure **в той же группе ресурсов, что и узлы AKS**. Важно выбрать правильную группу ресурсов. Группа ресурсов, необходимая в приведенной ниже команде, *не* является ссылкой на область портала AKS. Это группа ресурсов `aks-agentpool` виртуальных машин. Обычно эта группа ресурсов начинается с `MC_` и содержит имя AKS. Например: `MC_resourceGroup_aksABCD_westus`

    ```azurecli
    az identity create -g <agent-pool-resource-group> -n <identity-name>
    ```

1. Для приведенных ниже команд назначения ролей необходимо получить `principalId` для вновь созданного удостоверения:

    ```azurecli
    az identity show -g <resourcegroup> -n <identity-name>
    ```

1. Предоставьте удостоверению `Contributor` доступ к шлюзу приложений. Для этого вам потребуется идентификатор шлюза приложений, который будет выглядеть примерно так: `/subscriptions/A/resourceGroups/B/providers/Microsoft.Network/applicationGateways/C`

    Получите список идентификаторов шлюзов приложений в подписке с помощью: `az network application-gateway list --query '[].id'`

    ```azurecli
    az role assignment create \
        --role Contributor \
        --assignee <principalId> \
        --scope <App-Gateway-ID>
    ```

1. Предоставьте удостоверению `Reader` доступ к группе ресурсов шлюза приложений. Идентификатор группы ресурсов будет выглядеть следующим образом: `/subscriptions/A/resourceGroups/B` . Все группы ресурсов можно получить с помощью: `az group list --query '[].id'`

    ```azurecli
    az role assignment create \
        --role Reader \
        --assignee <principalId> \
        --scope <App-Gateway-Resource-Group-ID>
    ```

## <a name="using-a-service-principal"></a>Использование субъекта-службы
Также можно предоставить АГИК доступ к ARM через секрет Kubernetes.

1. Создайте субъект-службу Active Directory и закодировать его с помощью Base64. Для сохранения большого двоичного объекта JSON в Kubernetes требуется кодировка Base64.

```azurecli
az ad sp create-for-rbac --sdk-auth | base64 -w0
```

2. Добавьте в файл большой двоичный объект JSON в кодировке Base64 `helm-config.yaml` . Дополнительные сведения о `helm-config.yaml` параметре см. в следующем разделе.
```yaml
armAuth:
    type: servicePrincipal
    secretJSON: <Base64-Encoded-Credentials>
```

## <a name="install-ingress-controller-as-a-helm-chart"></a>Установка контроллера входящего трафика как Helm диаграммы
В первых нескольких шагах мы устанавливаем Helm в кластере Kubernetes. Для установки пакета Helm АГИК используйте [Cloud Shell](https://shell.azure.com/) :

1. Добавление `application-gateway-kubernetes-ingress` репозитория Helm и выполнение обновления Helm

    ```bash
    helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    helm repo update
    ```

1. Скачайте Helm-config. YAML, который будет настраивать АГИК:
    ```bash
    wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/sample-helm-config.yaml -O helm-config.yaml
    ```
    Или скопируйте файл YAML ниже: 
    
    ```yaml
    # This file contains the essential configs for the ingress controller helm chart

    # Verbosity level of the App Gateway Ingress Controller
    verbosityLevel: 3
    
    ################################################################################
    # Specify which application gateway the ingress controller will manage
    #
    appgw:
        subscriptionId: <subscriptionId>
        resourceGroup: <resourceGroupName>
        name: <applicationGatewayName>
    
        # Setting appgw.shared to "true" will create an AzureIngressProhibitedTarget CRD.
        # This prohibits AGIC from applying config for any host/path.
        # Use "kubectl get AzureIngressProhibitedTargets" to view and change this.
        shared: false
    
    ################################################################################
    # Specify which kubernetes namespace the ingress controller will watch
    # Default value is "default"
    # Leaving this variable out or setting it to blank or empty string would
    # result in Ingress Controller observing all acessible namespaces.
    #
    # kubernetes:
    #   watchNamespace: <namespace>
    
    ################################################################################
    # Specify the authentication with Azure Resource Manager
    #
    # Two authentication methods are available:
    # - Option 1: AAD-Pod-Identity (https://github.com/Azure/aad-pod-identity)
    armAuth:
        type: aadPodIdentity
        identityResourceID: <identityResourceId>
        identityClientID:  <identityClientId>
    
    ## Alternatively you can use Service Principal credentials
    # armAuth:
    #    type: servicePrincipal
    #    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --sdk-auth | base64 -w0" >>
    
    ################################################################################
    # Specify if the cluster is Kubernetes RBAC enabled or not
    rbac:
        enabled: false # true/false
    
    # Specify aks cluster related information. THIS IS BEING DEPRECATED.
    aksClusterConfiguration:
        apiServerAddress: <aks-api-server-address>
    ```

1. Измените Helm-config. YAML и заполните значения для `appgw` и `armAuth` .
    ```bash
    nano helm-config.yaml
    ```

    > [!NOTE] 
    > `<identity-resource-id>`Свойства и `<identity-client-id>` являются свойствами удостоверения Azure AD, настроенного в предыдущем разделе. Эти сведения можно получить, выполнив следующую команду: `az identity show -g <resourcegroup> -n <identity-name>` , где `<resourcegroup>` — это группа ресурсов, в которой развернуты объект кластера AKS верхнего уровня, шлюз приложений и управляемая оценка.

1. Установите диаграмму Helm `application-gateway-kubernetes-ingress` с `helm-config.yaml` конфигурацией из предыдущего шага.

    ```bash
    helm install -f <helm-config.yaml> application-gateway-kubernetes-ingress/ingress-azure
    ```

    Кроме того, можно объединить `helm-config.yaml` команду и Helm в одном шаге:
    ```bash
    helm install ./helm/ingress-azure \
         --name ingress-azure \
         --namespace default \
         --debug \
         --set appgw.name=applicationgatewayABCD \
         --set appgw.resourceGroup=your-resource-group \
         --set appgw.subscriptionId=subscription-uuid \
         --set appgw.shared=false \
         --set armAuth.type=servicePrincipal \
         --set armAuth.secretJSON=$(az ad sp create-for-rbac --sdk-auth | base64 -w0) \
         --set rbac.enabled=true \
         --set verbosityLevel=3 \
         --set kubernetes.watchNamespace=default \
         --set aksClusterConfiguration.apiServerAddress=aks-abcdefg.hcp.westus2.azmk8s.io
    ```

1. Проверьте журнал созданного модуля, чтобы проверить, правильно ли он запущен.

Ознакомьтесь с [этим руководством](ingress-controller-expose-service-over-http-https.md) , чтобы понять, как можно предоставить службу AKS через HTTP или HTTPS в Интернете с помощью шлюза приложений Azure.



## <a name="multi-cluster--shared-application-gateway"></a>Шлюз приложений с несколькими кластерами или общим доступом
По умолчанию АГИК предполагает полное владение шлюзом приложений, с которым он связан. АГИК версии 0.8.0 и более поздних версий могут совместно использовать один шлюз приложений с другими компонентами Azure. Например, можно использовать один и тот же шлюз приложений для приложения, размещенного в масштабируемом наборе виртуальных машин, а также в кластере AKS.

Перед включением этого параметра создайте __резервную копию конфигурации шлюза приложений__ :
  1. с помощью [портал Azure](https://portal.azure.com/) перейдите к своему `Application Gateway` экземпляру
  2. из `Export template` Click `Download`

Скачанный ZIP-файл будет содержать шаблоны JSON, bash и сценарии PowerShell, которые можно использовать для восстановления шлюза приложений.

### <a name="example-scenario"></a>Пример сценария
Рассмотрим воображаемый шлюз приложений, который управляет трафиком для двух веб-сайтов:
  - `dev.contoso.com` — размещается в новом AKS с использованием шлюза приложений и АГИК
  - `prod.contoso.com`— размещено в [масштабируемом наборе виртуальных машин Azure](https://azure.microsoft.com/services/virtual-machine-scale-sets/) ;

При использовании параметров по умолчанию АГИК предполагает владение шлюзом приложений на 100%, на который указывает. АГИК перезаписывает все настройки шлюза приложений. Если необходимо вручную создать прослушиватель для `prod.contoso.com` (в шлюзе приложений), не определяя его в Kubernetes входящих данных, агик удалит конфигурацию в течение нескольких `prod.contoso.com` секунд.

Чтобы установить АГИК, а также выполнить обслуживание на виртуальных машинах `prod.contoso.com` с масштабируемыми наборами, необходимо ограничить агик только конфигурацией `dev.contoso.com` . Это упрощается путем создания экземпляра следующих [CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/):

```bash
cat <<EOF | kubectl apply -f -
apiVersion: "appgw.ingress.k8s.io/v1"
kind: AzureIngressProhibitedTarget
metadata:
  name: prod-contoso-com
spec:
  hostname: prod.contoso.com
EOF
```

Приведенная выше команда создает `AzureIngressProhibitedTarget` объект. Это делает АГИК (версии 0.8.0 и более поздней) осведомленной о существовании конфигурации шлюза приложений для `prod.contoso.com` и явно инструктирует его, чтобы избежать изменения конфигурации, связанной с этим именем узла.


### <a name="enable-with-new-agic-installation"></a>Включить при установке новой АГИК
Чтобы ограничить АГИК (Version 0.8.0 и более поздние версии) подмножеством конфигурации шлюза приложений, измените `helm-config.yaml` шаблон.
В `appgw:` разделе добавьте `shared` раздел и задайте для него значение `true` .

```yaml
appgw:
    subscriptionId: <subscriptionId>    # existing field
    resourceGroup: <resourceGroupName>  # existing field
    name: <applicationGatewayName>      # existing field
    shared: true                        # <<<<< Add this field to enable shared Application Gateway >>>>>
```

Применить изменения Helm:
  1. Убедитесь, что `AzureIngressProhibitedTarget` CRD установлен с помощью:
      ```bash
      kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/ae695ef9bd05c8b708cedf6ff545595d0b7022dc/crds/AzureIngressProhibitedTarget.yaml
      ```
  2. Обновление Helm:
      ```bash
      helm upgrade \
          --recreate-pods \
          -f helm-config.yaml \
          ingress-azure application-gateway-kubernetes-ingress/ingress-azure
      ```

В результате в AKS будет создан новый экземпляр с `AzureIngressProhibitedTarget` именем `prohibit-all-targets` :
```bash
kubectl get AzureIngressProhibitedTargets prohibit-all-targets -o yaml
```

Объект `prohibit-all-targets` , как и предполагает имя, запрещает агик изменять конфигурацию для *любого* узла и пути.
Helm install с `appgw.shared=true` выполнит развертывание агик, но не внесет никаких изменений в шлюз приложений.


### <a name="broaden-permissions"></a>Расширить разрешения
Так как Helm с `appgw.shared=true` и блоки по умолчанию `prohibit-all-targets` агик от применения любой конфигурации.

Расширить разрешения АГИК с помощью:
1. Создайте новый `AzureIngressProhibitedTarget` с помощью конкретной настройки:
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: "appgw.ingress.k8s.io/v1"
    kind: AzureIngressProhibitedTarget
    metadata:
      name: your-custom-prohibitions
    spec:
      hostname: your.own-hostname.com
    EOF
    ```

2. Вы можете удалить значение по умолчанию, которое является слишком широким, только после создания собственного пользовательского запрета.

    ```bash
    kubectl delete AzureIngressProhibitedTarget prohibit-all-targets
    ```

### <a name="enable-for-an-existing-agic-installation"></a>Включить для существующей установки АГИК
Предположим, что у нас уже есть рабочий AKS, шлюз приложений и настроенный АГИК в нашем кластере. У нас есть `prod.contoso.com` входящий трафик для и успешно обслуживать его в AKS. Мы хотим добавить `staging.contoso.com` к нашему существующему шлюзу приложений, но необходимо разместить его на [виртуальной машине](https://azure.microsoft.com/services/virtual-machines/). Мы будем повторно использовать существующий шлюз приложений и вручную настроить прослушиватель и серверные пулы для `staging.contoso.com` . Но Настройка конфигурации шлюза приложений вручную (через [портал](https://portal.azure.com), [API ARM](/rest/api/resources/) или [terraform](https://www.terraform.io/)) противоречит предположениям полного владения агик. Вскоре после применения изменений АГИК будет перезаписывать или удалять их.

Мы можем запретить АГИК внесение изменений в подмножество конфигурации.

1. Создайте `AzureIngressProhibitedTarget` объект:
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: "appgw.ingress.k8s.io/v1"
    kind: AzureIngressProhibitedTarget
    metadata:
      name: manually-configured-staging-environment
    spec:
      hostname: staging.contoso.com
    EOF
    ```

2. Просмотрите только что созданный объект:
    ```bash
    kubectl get AzureIngressProhibitedTargets
    ```

3. Изменение конфигурации шлюза приложений с помощью портала — Добавление прослушивателей, правил маршрутизации, серверной точки и т. д. Новый объект, который мы создали ( `manually-configured-staging-environment` ), не позволит агик перезаписывать конфигурацию шлюза приложений, связанную с `staging.contoso.com` .
