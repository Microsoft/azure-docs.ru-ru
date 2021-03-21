---
title: Настройка кластера Kubernetes с включенной службой "Дуга Azure" с помощью Container Insights | Документация Майкрософт
description: В этой статье описывается, как настроить мониторинг с помощью Container Insights в кластерах Kubernetes с включенной службой ARC.
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 307f9d9928042410dc9b4443aba5c019c592980c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711303"
---
# <a name="enable-monitoring-of-azure-arc-enabled-kubernetes-cluster"></a>Включение мониторинга кластеров Kubernetes с поддержкой Azure Arc

Служба "аналитика Azure" предоставляет широкие возможности мониторинга для кластеров Kubernetes Service (AKS) и AKS Engine. В этой статье описывается, как включить мониторинг кластеров Kubernetes, размещенных за пределами Azure, которые включены с помощью дуги Azure, для обеспечения аналогичного процесса мониторинга.

Аналитика контейнера может быть включена для одного или нескольких существующих развертываний Kubernetes с помощью скрипта PowerShell или bash.

## <a name="supported-configurations"></a>Поддерживаемые конфигурации

Служба "аналитика Azure" поддерживает мониторинг дуги Kubernetes (Предварительная версия), как описано в [обзорной](container-insights-overview.md) статье, за исключением следующих возможностей.

- Данные в режиме реального времени (Предварительная версия)

Следующие сведения официально поддерживаются в контейнере "аналитика":

- Версии Kubernetes и политики поддержки совпадают с [поддерживаемыми версиями AKS](../../aks/supported-kubernetes-versions.md).

- Поддерживаются следующие среды выполнения контейнеров: DOCKER, значок Кита и CRI совместимые среды выполнения, такие как CRI-O и контейнеры.

- Поддерживается выпуск ОС Linux для основных и рабочих узлов: Ubuntu (18,04 LTS и 16,04 LTS).

## <a name="prerequisites"></a>Предварительные условия

Чтобы начать, у вас должны быть следующие компоненты:

- Рабочая область Log Analytics.

    Служба "аналитика контейнеров" поддерживает Log Analytics рабочей области в регионах, перечисленных в списке [продуктов Azure по регионам](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor). Чтобы создать собственную рабочую область, ее можно создать с помощью [Azure Resource Manager](../logs/resource-manager-workspace.md), [PowerShell](../logs/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)или в [портал Azure](../logs/quick-create-workspace.md).

- Чтобы включить и получить доступ к функциям в контейнере аналитики, необходимо быть членом роли *участника* Azure в подписке Azure и членом роли [*участника log Analytics*](../logs/manage-access.md#manage-access-using-azure-permissions) в log Analytics рабочей области, настроенной с помощью аналитики контейнеров.

- Вы являетесь членом роли " [участник](../../role-based-access-control/built-in-roles.md#contributor) " в ресурсе кластера Arc Azure.

- Чтобы просмотреть данные мониторинга, вы являетесь членом разрешения роли [*log Analytics читатель*](../logs/manage-access.md#manage-access-using-azure-permissions) с рабочей областью log Analytics, настроенной с помощью Container Insights.

- [Helm клиент](https://helm.sh/docs/using_helm/) , чтобы подключить диаграмму Container Insights для указанного кластера Kubernetes.

- Следующие сведения о конфигурации прокси-сервера и брандмауэра необходимы для того, чтобы контейнерная версия агента Log Analytics для Linux могла взаимодействовать с Azure Monitor:

    |Ресурс агента|порты; |
    |------|---------|
    |`*.ods.opinsights.azure.com` |Порт 443 |
    |`*.oms.opinsights.azure.com` |Порт 443 |
    |`*.dc.services.visualstudio.com` |Порт 443 |

- Контейнерный агент требует, чтобы Kubelet `cAdvisor secure port: 10250` или `unsecure port :10255` был открыт на всех узлах кластера для сбора метрик производительности. Рекомендуется настроить `secure port: 10250` для CAdvisor Kubelet, если он еще не настроен.

- Контейнерный агент требует, чтобы в контейнере были указаны следующие переменные среды для взаимодействия со службой API Kubernetes в кластере для сбора данных инвентаризации — `KUBERNETES_SERVICE_HOST` и `KUBERNETES_PORT_443_TCP_PORT` .

    >[!IMPORTANT]
    >Минимальная версия агента, поддерживаемая для мониторинга кластеров Kubernetes с поддержкой Arc, — это ciprod04162020 или более поздней версии.

- При включении мониторинга с помощью метода PowerShell необходимо использовать [PowerShell Core](/powershell/scripting/install/installing-powershell?view=powershell-6&preserve-view=true) .

- При включении мониторинга с помощью метода Bash в скрипте требуется [Версия 4 Bash](https://www.gnu.org/software/bash/) .

## <a name="identify-workspace-resource-id"></a>Указание идентификатора ресурса рабочей области

Чтобы включить мониторинг кластера с помощью сценария PowerShell или bash, скачанного ранее, и интегрировать его с существующей рабочей областью Log Analytics, выполните следующие действия, чтобы сначала найти полный идентификатор ресурса для Log Analytics рабочей области. Это необходимо для `workspaceResourceId` параметра при выполнении команды, чтобы включить надстройку мониторинга для указанной рабочей области. Если у вас нет рабочей области для указания, можно пропустить включение `workspaceResourceId` параметра и позволить скрипту создать новую рабочую область.

1. Перечислите все подписки, к которым у вас есть доступ, с помощью следующей команды:

    ```azurecli
    az account list --all -o table
    ```

    Результат должен выглядеть так:

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   0fb60ef2-03cc-4290-b595-e71108e8f4ce  Enabled  True
    ```

    Скопируйте значение для **SubscriptionId**.

2. Перейдите к подписке, в которой размещена рабочая область Log Analytics, с помощью следующей команды:

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

3. В следующем примере отображается список рабочих областей в подписках в формате JSON по умолчанию.

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    В выходных данных найдите имя рабочей области, а затем скопируйте полный идентификатор ресурса этой Log Analytics рабочей области под **идентификатором** поля.

## <a name="enable-monitoring-using-powershell"></a>Включение мониторинга с помощью PowerShell

1. Скачайте и сохраните скрипт в локальной папке, которая настраивает кластер с помощью надстройки мониторинга, используя следующие команды:

    ```powershell
    Invoke-WebRequest https://aka.ms/enable-monitoring-powershell-script -OutFile enable-monitoring.ps1
    ```

2. Настройте `$azureArcClusterResourceId` переменную, задав соответствующие значения для `subscriptionId` и указав `resourceGroupName` идентификатор ресурса `clusterName` кластера Kubernetes с поддержкой дуги Azure.

    ```powershell
    $azureArcClusterResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Настройте `$kubeContext` переменную с помощью **KUBE-context** в кластере, выполнив команду `kubectl config get-contexts` . Если вы хотите использовать текущий контекст, задайте для него значение `""` .

    ```powershell
    $kubeContext = "<kubeContext name of your k8s cluster>"
    ```

4. Если вы хотите использовать существующую рабочую область Azure Monitor Log Analytics, настройте переменную `$logAnalyticsWorkspaceResourceId` с соответствующим значением, представляющим идентификатор ресурса рабочей области. В противном случае задайте для переменной значение `""` , и скрипт создаст рабочую область по умолчанию в группе ресурсов по умолчанию подписки на кластер, если она еще не существует в регионе. Созданная Рабочая область по умолчанию напоминает формат *DefaultWorkspace- \<SubscriptionID> - \<Region>*.

    ```powershell
    $logAnalyticsWorkspaceResourceId = "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.operationalinsights/workspaces/<workspaceName>"
    ```

5. Если кластер Kubernetes с поддержкой Arc взаимодействует через прокси-сервер, настройте переменную `$proxyEndpoint` URL-адресом прокси-сервера. Если кластер не обменивается данными через прокси-сервер, можно задать для него значение `""` .  Дополнительные сведения см. в разделе [Настройка конечной точки прокси-сервера](#configure-proxy-endpoint) далее в этой статье.

6. Выполните следующую команду, чтобы включить мониторинг.

    ```powershell
    .\enable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -kubeContext $kubeContext -workspaceResourceId $logAnalyticsWorkspaceResourceId -proxyEndpoint $proxyEndpoint
    ```

После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера.

### <a name="using-service-principal"></a>Использование субъекта-службы
В *enable-monitoring.ps1* сценария используется имя входа интерактивного устройства. Если вы предпочитаете неинтерактивное имя входа, можно использовать существующий субъект-службу или создать новый, имеющий необходимые разрешения, как описано в разделе [Предварительные требования](#prerequisites). Чтобы использовать субъект-службу, необходимо передать параметры $servicePrincipalClientId, $servicePrincipalClientSecret и $tenantId, указав значения субъекта-службы, которые будут использоваться для *enable-monitoring.ps1* сценария.

```powershell
$subscriptionId = "<subscription Id of the Azure Arc connected cluster resource>"
$servicePrincipal = New-AzADServicePrincipal -Role Contributor -Scope "/subscriptions/$subscriptionId"
```

Приведенные ниже назначения ролей применимы только при использовании существующей рабочей области Log Analytics в другой подписке Azure, чем кластерный ресурс Arc K8s.

```powershell
$logAnalyticsWorkspaceResourceId = "<Azure Resource Id of the Log Analytics Workspace>" # format of the Azure Log Analytics workspace should be /subscriptions/<subId>/resourcegroups/<rgName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>
New-AzRoleAssignment -RoleDefinitionName 'Log Analytics Contributor'  -ObjectId $servicePrincipal.Id -Scope  $logAnalyticsWorkspaceResourceId

$servicePrincipalClientId =  $servicePrincipal.ApplicationId.ToString()
$servicePrincipalClientSecret = [System.Net.NetworkCredential]::new("", $servicePrincipal.Secret).Password
$tenantId = (Get-AzSubscription -SubscriptionId $subscriptionId).TenantId
```

Пример:

```powershell
.\enable-monitoring.ps1 -clusterResourceId $azureArcClusterResourceId -servicePrincipalClientId $servicePrincipalClientId -servicePrincipalClientSecret $servicePrincipalClientSecret -tenantId $tenantId -kubeContext $kubeContext -workspaceResourceId $logAnalyticsWorkspaceResourceId -proxyEndpoint $proxyEndpoint
```



## <a name="enable-using-bash-script"></a>Включить использование Bash скрипта

Чтобы включить мониторинг с помощью предоставленного скрипта bash, выполните следующие действия.

1. Скачайте и сохраните скрипт в локальной папке, которая настраивает кластер с помощью надстройки мониторинга, используя следующие команды:

    ```bash
    curl -o enable-monitoring.sh -L https://aka.ms/enable-monitoring-bash-script
    ```

2. Настройте `azureArcClusterResourceId` переменную, задав соответствующие значения для `subscriptionId` и указав `resourceGroupName` идентификатор ресурса `clusterName` кластера Kubernetes с поддержкой дуги Azure.

    ```bash
    export azureArcClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
    ```

3. Настройте `kubeContext` переменную с помощью **KUBE-context** в кластере, выполнив команду `kubectl config get-contexts` . Если вы хотите использовать текущий контекст, задайте для него значение `""` .

    ```bash
    export kubeContext="<kubeContext name of your k8s cluster>"
    ```

4. Если вы хотите использовать существующую рабочую область Azure Monitor Log Analytics, настройте переменную `logAnalyticsWorkspaceResourceId` с соответствующим значением, представляющим идентификатор ресурса рабочей области. В противном случае задайте для переменной значение `""` , и скрипт создаст рабочую область по умолчанию в группе ресурсов по умолчанию подписки на кластер, если она еще не существует в регионе. Созданная Рабочая область по умолчанию напоминает формат *DefaultWorkspace- \<SubscriptionID> - \<Region>*.

    ```bash
    export logAnalyticsWorkspaceResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.operationalinsights/workspaces/<workspaceName>"
    ```

5. Если кластер Kubernetes с поддержкой Arc взаимодействует через прокси-сервер, настройте переменную `proxyEndpoint` URL-адресом прокси-сервера. Если кластер не обменивается данными через прокси-сервер, можно задать для него значение `""` . Дополнительные сведения см. в разделе [Настройка конечной точки прокси-сервера](#configure-proxy-endpoint) далее в этой статье.

6. Чтобы включить мониторинг в кластере, на основе сценария развертывания существуют различные команды.

    Выполните следующую команду, чтобы включить мониторинг с параметрами по умолчанию, такими как использование текущего KUBE контекста, создание рабочей области Log Analytics по умолчанию и без указания прокси-сервера.

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId
    ```

    Выполните следующую команду, чтобы создать рабочую область Log Analytics по умолчанию и без указания прокси-сервера.

    ```bash
   bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext
    ```

    Выполните следующую команду, чтобы использовать существующую рабочую область Log Analytics и без указания прокси-сервера:

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId
    ```

    Выполните следующую команду, чтобы использовать существующую рабочую область Log Analytics и указать прокси-сервер:

    ```bash
    bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId --proxy $proxyEndpoint
    ```

После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера.

### <a name="using-service-principal"></a>Использование субъекта-службы
В скрипте Bash *Enable-Monitoring.sh* используется имя входа интерактивного устройства. Если вы предпочитаете неинтерактивное имя входа, можно использовать существующий субъект-службу или создать новый, имеющий необходимые разрешения, как описано в разделе [Предварительные требования](#prerequisites). Чтобы использовать субъект-службу, необходимо передать значения типа "Client-ID", "--Client-Secret" и "--Клиент-ID" субъекта службы, который будет использоваться для *Enable-Monitoring.sh* Bash-скрипта.

```bash
subscriptionId="<subscription Id of the Azure Arc connected cluster resource>"
servicePrincipal=$(az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/${subscriptionId}")
servicePrincipalClientId=$(echo $servicePrincipal | jq -r '.appId')
```

Приведенные ниже назначения ролей применимы только при использовании существующей рабочей области Log Analytics в другой подписке Azure, чем кластерный ресурс Arc K8s.

```bash
logAnalyticsWorkspaceResourceId="<Azure Resource Id of the Log Analytics Workspace>" # format of the Azure Log Analytics workspace should be /subscriptions/<subId>/resourcegroups/<rgName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>
az role assignment create --role 'Log Analytics Contributor' --assignee $servicePrincipalClientId --scope $logAnalyticsWorkspaceResourceId

servicePrincipalClientSecret=$(echo $servicePrincipal | jq -r '.password')
tenantId=$(echo $servicePrincipal | jq -r '.tenant')
```

Пример:

```bash
bash enable-monitoring.sh --resource-id $azureArcClusterResourceId --client-id $servicePrincipalClientId --client-secret $servicePrincipalClientSecret  --tenant-id $tenantId --kube-context $kubeContext  --workspace-id $logAnalyticsWorkspaceResourceId --proxy $proxyEndpoint
```

## <a name="configure-proxy-endpoint"></a>Настройка конечной точки прокси-сервера

С контейнерным агентом для аналитики контейнера можно настроить конечную точку прокси, чтобы разрешить ей взаимодействовать через прокси-сервер. Обмен данными между контейнерным агентом и Azure Monitor может быть прокси-сервером HTTP или HTTPS, и анонимная и обычная проверка подлинности (имя пользователя и пароль) поддерживаются.

Значение конфигурации прокси-сервера имеет следующий синтаксис: `[protocol://][user:password@]proxyhost[:port]`

> [!NOTE]
>Если прокси-сервер не требует проверки подлинности, необходимо указать имя пользователя и пароль псуедо. Вы можете указать любое имя пользователя или пароль.

|Свойство| Описание |
|--------|-------------|
|Протокол | HTTP или HTTPS |
|пользователь | Необязательное имя пользователя для аутентификации прокси-сервера |
|password | Необязательный пароль для аутентификации прокси-сервера |
|proxyhost | Адрес или полное доменное имя прокси-сервера |
|порт | Необязательный номер порта прокси-сервера |

Пример: `http://user01:password@proxy01.contoso.com:3128`

Если указать протокол как **http**, HTTP-запросы создаются с помощью защищенного соединения SSL/TLS. Прокси-сервер должен поддерживать протоколы SSL/TLS.

### <a name="configure-using-powershell"></a>Настройка с помощью PowerShell

Укажите имя пользователя и пароль, IP-адрес или полное доменное имя, а также номер порта для прокси-сервера. Пример:

```powershell
$proxyEndpoint = https://<user>:<password>@<proxyhost>:<port>
```

### <a name="configure-using-bash"></a>Настройка с помощью bash

Укажите имя пользователя и пароль, IP-адрес или полное доменное имя, а также номер порта для прокси-сервера. Пример:

```bash
export proxyEndpoint=https://<user>:<password>@<proxyhost>:<port>
```

## <a name="next-steps"></a>Дальнейшие действия

- С включенным наблюдением для получения сведений о работоспособности и использовании ресурсов для кластера Kubernetes с включенной системой Arc и рабочих нагрузок на них вы узнаете, [как использовать](container-insights-analyze.md) аналитику контейнера.

- По умолчанию контейнерный агент собирает журналы контейнеров stdout/stderr всех контейнеров, работающих во всех пространствах имен, за исключением KUBE-System. Чтобы настроить сбор журналов контейнеров для конкретного пространства имен или пространств имен, ознакомьтесь с разрешениями [Конфигурация агента аналитики контейнеров](container-insights-agent-config.md) , чтобы настроить необходимые параметры сбора данных в файле конфигурации ConfigMap.

- Чтобы отбракировать и проанализировать метрики Prometheus из кластера, см. раздел [Настройка брака/отхода метрик Prometheus](container-insights-prometheus-integration.md)

- Сведения о том, как отключить мониторинг кластера Kubernetes с включенной поддержкой ARC с помощью Container Insights, см. в статье [как прерывать мониторинг гибридного кластера](container-insights-optout-hybrid.md#how-to-stop-monitoring-on-arc-enabled-kubernetes).