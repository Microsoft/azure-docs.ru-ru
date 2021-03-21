---
title: Создание контроллера входящего трафика с новым шлюзом приложений
description: В этой статье содержатся сведения о развертывании контроллера входящего трафика шлюза приложений с помощью нового шлюза приложений.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 8be5ac75e2da3eaeae300fd36e152a24c9777e64
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99593745"
---
# <a name="how-to-install-an-application-gateway-ingress-controller-agic-using-a-new-application-gateway"></a>Установка контроллера входящего трафика шлюза приложений (АГИК) с помощью нового шлюза приложений

В приведенных ниже инструкциях предполагается, что контроллер входящего трафика шлюза приложений (АГИК) будет установлен в среде без уже существующих компонентов.

## <a name="required-command-line-tools"></a>Обязательные средства командной строки

Рекомендуется использовать [Azure Cloud Shell](https://shell.azure.com/) для всех операций командной строки, приведенных ниже. Запустите оболочку из shell.azure.com или щелкните ссылку:

[![Внедрение запуска](https://shell.azure.com/images/launchcloudshell.png "Запуск Azure Cloud Shell")](https://shell.azure.com)

Кроме того, можно запустить Cloud Shell из портал Azure, используя следующий значок:

![Запуск с помощью портала](./media/application-gateway-ingress-controller-install-new/portal-launch-icon.png)

У [Azure Cloud Shell](https://shell.azure.com/) уже есть все необходимые средства. Если вы решили использовать другую среду, убедитесь, что установлены следующие программы командной строки:

* `az` -Azure CLI: [инструкции по установке](/cli/azure/install-azure-cli)
* `kubectl` -Kubernetes программа командной строки: [инструкции по установке](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* `helm` -Диспетчер пакетов Kubernetes: [инструкции по установке](https://github.com/helm/helm/releases/latest)
* `jq` -процессор командной строки JSON: [инструкции по установке](https://stedolan.github.io/jq/download/)


## <a name="create-an-identity"></a>Создание удостоверения

Выполните следующие действия, чтобы создать [объект субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object)Azure Active Directory (AAD). Запишите `appId` значения, `password` и, `objectId` которые будут использоваться в следующих шагах.

1. Создайте субъект-службу AD ([Подробнее об Azure RBAC](../role-based-access-control/overview.md)):
    ```azurecli
    az ad sp create-for-rbac --skip-assignment -o json > auth.json
    appId=$(jq -r ".appId" auth.json)
    password=$(jq -r ".password" auth.json)
    ```
    `appId`Значения и `password` из выходных данных JSON будут использоваться в следующих шагах.


1. `appId`Чтобы получить новый субъект-службу, используйте команду из вывода предыдущей команды `objectId` :
    ```azurecli
    objectId=$(az ad sp show --id $appId --query "objectId" -o tsv)
    ```
    Выходные данные этой команды `objectId` будут использоваться в шаблоне Azure Resource Manager ниже.

1. Создайте файл параметров, который будет использоваться в развертывании шаблона Azure Resource Manager позже.
    ```bash
    cat <<EOF > parameters.json
    {
      "aksServicePrincipalAppId": { "value": "$appId" },
      "aksServicePrincipalClientSecret": { "value": "$password" },
      "aksServicePrincipalObjectId": { "value": "$objectId" },
      "aksEnableRBAC": { "value": false }
    }
    EOF
    ```
    Чтобы развернуть кластер с поддержкой **KUBERNETES RBAC** , задайте `aksEnableRBAC` для поля значение. `true`

## <a name="deploy-components"></a>Развертывание компонентов
На этом шаге в подписку будут добавлены следующие компоненты:

- [Служба Azure Kubernetes](../aks/intro-kubernetes.md)
- [Шлюз приложений](./overview.md) версии 2
- [Виртуальная сеть](../virtual-network/virtual-networks-overview.md) с двумя [подсетями](../virtual-network/virtual-networks-overview.md)
- [Общедоступный IP-адрес](../virtual-network/virtual-network-public-ip-address.md)
- [Управляемое удостоверение](../active-directory/managed-identities-azure-resources/overview.md), которое будет использоваться [удостоверением "AAD Pod](https://github.com/Azure/aad-pod-identity/blob/master/README.md) "

1. Скачайте шаблон Azure Resource Manager и при необходимости измените шаблон.
    ```bash
    wget https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/deploy/azuredeploy.json -O template.json
    ```

1. Разверните шаблон Azure Resource Manager с помощью `az cli` . Это может занять до 5 минут.
    ```azurecli
    resourceGroupName="MyResourceGroup"
    location="westus2"
    deploymentName="ingress-appgw"

    # create a resource group
    az group create -n $resourceGroupName -l $location

    # modify the template as needed
    az deployment group create \
            -g $resourceGroupName \
            -n $deploymentName \
            --template-file template.json \
            --parameters parameters.json
    ```

1. После завершения развертывания Скачайте выходные данные развертывания в файл с именем `deployment-outputs.json` .
    ```azurecli
    az deployment group show -g $resourceGroupName -n $deploymentName --query "properties.outputs" -o json > deployment-outputs.json
    ```

## <a name="set-up-application-gateway-ingress-controller"></a>Настройка входящего контроллера шлюза приложений

Следуя инструкциям в предыдущем разделе, мы создали и настроили новый кластер AKS и шлюз приложений. Теперь мы готовы к развертыванию примера приложения и входного контроллера в нашей новой инфраструктуре Kubernetes.

### <a name="setup-kubernetes-credentials"></a>Настройка учетных данных Kubernetes
Для выполнения следующих действий требуется команда [kubectl](https://kubectl.docs.kubernetes.io/) Setup, которая будет использоваться для подключения к нашему новому кластеру Kubernetes. [Cloud Shell](https://shell.azure.com/) `kubectl` уже установлен. Мы будем использовать `az` CLI для получения учетных данных для Kubernetes.

Получите учетные данные для вновь развернутых AKS (см.[Дополнительные](../aks/kubernetes-walkthrough.md#connect-to-the-cluster)сведения):
```azurecli
# use the deployment-outputs.json created after deployment to get the cluster name and resource group name
aksClusterName=$(jq -r ".aksClusterName.value" deployment-outputs.json)
resourceGroupName=$(jq -r ".resourceGroupName.value" deployment-outputs.json)

az aks get-credentials --resource-group $resourceGroupName --name $aksClusterName
```

### <a name="install-aad-pod-identity"></a>Установка удостоверения Pod для AAD
  Azure Active Directory удостоверение Pod предоставляет доступ на основе маркеров к [Azure Resource Manager (ARM)](../azure-resource-manager/management/overview.md).

  [Удостоверение для AAD Pod](https://github.com/Azure/aad-pod-identity) добавит в кластер Kubernetes следующие компоненты:
   * [определения пользовательских ресурсов](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) Kubernetes: `AzureIdentity`, `AzureAssignedIdentity`, `AzureIdentityBinding`;
   * компонент [Контроллер управляемых удостоверений (MIC)](https://github.com/Azure/aad-pod-identity#managed-identity-controllermic);
   * компонент [Node Managed Identity (NMI)](https://github.com/Azure/aad-pod-identity#node-managed-identitynmi).


Чтобы установить удостоверение AAD Pod в кластер, выполните следующие действия.

   - *KUBERNETES RBAC включен* Кластер AKS

     ```bash
     kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment-rbac.yaml
     ```

   - *KUBERNETES RBAC отключен* Кластер AKS

     ```bash
     kubectl create -f https://raw.githubusercontent.com/Azure/aad-pod-identity/master/deploy/infra/deployment.yaml
     ```

### <a name="install-helm"></a>Установка Helm
[Helm](../aks/kubernetes-helm.md) — это диспетчер пакетов для Kubernetes. Мы будем использовать его для установки `application-gateway-kubernetes-ingress` пакета:

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

### <a name="install-ingress-controller-helm-chart"></a>Установка диаграммы Helm контроллера входящего трафика

1. Используйте `deployment-outputs.json` созданный выше файл и создайте следующие переменные.
    ```bash
    applicationGatewayName=$(jq -r ".applicationGatewayName.value" deployment-outputs.json)
    resourceGroupName=$(jq -r ".resourceGroupName.value" deployment-outputs.json)
    subscriptionId=$(jq -r ".subscriptionId.value" deployment-outputs.json)
    identityClientId=$(jq -r ".identityClientId.value" deployment-outputs.json)
    identityResourceId=$(jq -r ".identityResourceId.value" deployment-outputs.json)
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
    #    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --subscription <subscription-uuid> --sdk-auth | base64 -w0" >>
    
    ################################################################################
    # Specify if the cluster is Kubernetes RBAC enabled or not
    rbac:
        enabled: false # true/false
    
    # Specify aks cluster related information. THIS IS BEING DEPRECATED.
    aksClusterConfiguration:
        apiServerAddress: <aks-api-server-address>
    ```

1. Измените только что скачанный файл Helm-config. YAML и заполните разделы `appgw` и `armAuth` .
    ```bash
    sed -i "s|<subscriptionId>|${subscriptionId}|g" helm-config.yaml
    sed -i "s|<resourceGroupName>|${resourceGroupName}|g" helm-config.yaml
    sed -i "s|<applicationGatewayName>|${applicationGatewayName}|g" helm-config.yaml
    sed -i "s|<identityResourceId>|${identityResourceId}|g" helm-config.yaml
    sed -i "s|<identityClientId>|${identityClientId}|g" helm-config.yaml

    # You can further modify the helm config to enable/disable features
    nano helm-config.yaml
    ```

   Значения:
     - `verbosityLevel`: задает уровень детализации инфраструктуры ведения журналов AGIC. Возможные значения можно найти в разделе [Уровни ведения журнала](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/463a87213bbc3106af6fce0f4023477216d2ad78/docs/troubleshooting.md#logging-levels).
     - `appgw.subscriptionId`— Идентификатор подписки Azure, в которой находится шлюз приложений. Например, `a123b234-a3b4-557d-b2df-a0bc12de1234`.
     - `appgw.resourceGroup`: Имя группы ресурсов Azure, в которой был создан шлюз приложений. Например, `app-gw-resource-group`.
     - `appgw.name`: имя Шлюза приложений. Например, `applicationgatewayd0f0`.
     - `appgw.shared`: этот логический флаг должен иметь значение по умолчанию `false`. Задайте значение, `true` Если вам нужен [Общий шлюз приложений](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/072626cb4e37f7b7a1b0c4578c38d1eadc3e8701/docs/setup/install-existing.md#multi-cluster--shared-app-gateway).
     - `kubernetes.watchNamespace`: укажите пространство имен, которое должно отслеживать AGIC. Это может быть одно строковое значение или разделенный запятыми список пространств имен.
    - `armAuth.type`: может быть `aadPodIdentity` или `servicePrincipal`
    - `armAuth.identityResourceID`: Идентификатор ресурса управляемого удостоверения Azure.
    - `armAuth.identityClientId`: идентификатор клиента удостоверения. Дополнительные сведения об удостоверении см. ниже.
    - `armAuth.secretJSON`: Требуется только при выборе типа секрета субъекта-службы (если `armAuth.type` для задано значение `servicePrincipal` ) 


   > [!NOTE]
   > `identityResourceID`И `identityClientID` — это значения, которые были созданы во время действий по [развертыванию компонентов](ingress-controller-install-new.md#deploy-components) и могут быть получены повторно с помощью следующей команды:
   > ```azurecli
   > az identity show -g <resource-group> -n <identity-name>
   > ```
   > `<resource-group>` в приведенной выше команде — это группа ресурсов шлюза приложений. `<identity-name>` имя созданного удостоверения. Все удостоверения для данной подписки могут быть перечислены с помощью: `az identity list`


1. Установите пакет контроллера входящего трафика Шлюза приложений:

    ```bash
    helm install -f helm-config.yaml application-gateway-kubernetes-ingress/ingress-azure
    ```

## <a name="install-a-sample-app"></a>Установка примера приложения
Теперь, когда у нас установлен шлюз приложений, AKS и АГИК, мы можем установить пример приложения с помощью [Azure Cloud Shell](https://shell.azure.com/):

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: aspnetapp
  labels:
    app: aspnetapp
spec:
  containers:
  - image: "mcr.microsoft.com/dotnet/core/samples:aspnetapp"
    name: aspnetapp-image
    ports:
    - containerPort: 80
      protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
  name: aspnetapp
spec:
  selector:
    app: aspnetapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: aspnetapp
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: aspnetapp
          servicePort: 80
EOF
```

Кроме того, можно:

* Скачайте файл YAML, приведенный выше:

```bash
curl https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml -o aspnetapp.yaml
```

* Примените файл YAML:

```bash
kubectl apply -f aspnetapp.yaml
```


## <a name="other-examples"></a>Другие образцы
В этом [пошаговом руководство](ingress-controller-expose-service-over-http-https.md) содержатся дополнительные примеры предоставления службы AKS через протокол HTTP или HTTPS для доступа к Интернету с помощью шлюза приложений.
