---
title: Настройка гибридных кластеров Kubernetes с помощью Container Insights | Документация Майкрософт
description: В этой статье описывается, как настроить контейнерную аналитику для мониторинга кластеров Kubernetes, размещенных на Azure Stack или в другой среде.
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: d2692b4a634d60ef62339f68277591d711260712
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711252"
---
# <a name="configure-hybrid-kubernetes-clusters-with-container-insights"></a>Настройка гибридных кластеров Kubernetes с помощью аналитики контейнеров

Служба "аналитика контейнеров" предоставляет широкие возможности мониторинга службы Azure Kubernetes Service (AKS) и [подсистемы AKS в Azure](https://github.com/Azure/aks-engine), которая представляет собой управляемый кластер Kubernetes, размещенный в Azure. В этой статье описывается, как включить мониторинг кластеров Kubernetes, размещенных за пределами Azure, и добиться аналогичного процесса мониторинга.

## <a name="supported-configurations"></a>Поддерживаемые конфигурации

Следующие конфигурации официально поддерживаются в контейнере аналитики. При наличии другой версии Kubernetes и версии операционной системы отправьте сообщение по адресу askcoin@microsoft.com .

- Возможным

    - Kubernetes в локальной среде
    - Подсистема AKS в Azure и Azure Stack. Дополнительные сведения см. [в разделе AKS Engine on Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
    - [OpenShift](https://docs.openshift.com/container-platform/4.3/welcome/index.html) версии 4 или более поздней, локальной или другой облачной среды.

- Версии Kubernetes и политики поддержки совпадают с [поддерживаемыми версиями AKS](../../aks/supported-kubernetes-versions.md).

- Поддерживаются следующие среды выполнения контейнеров: DOCKER, значок Кита и CRI совместимые среды выполнения, такие как CRI-O и контейнеры.

- Поддерживается выпуск ОС Linux для основных и рабочих узлов: Ubuntu (18,04 LTS и 16,04 LTS) и Red Hat Enterprise Linux CoreOS 43,81.

- Поддерживается контроль доступа: Kubernetes RBAC и non-RBAC.

## <a name="prerequisites"></a>Предварительные условия

Чтобы начать, у вас должны быть следующие компоненты:

- [Рабочая область Log Analytics](../logs/design-logs-deployment.md).

    Служба "аналитика контейнеров" поддерживает Log Analytics рабочей области в регионах, перечисленных в списке [продуктов Azure по регионам](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor). Чтобы создать собственную рабочую область, ее можно создать с помощью [Azure Resource Manager](../logs/resource-manager-workspace.md), [PowerShell](../logs/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)или в [портал Azure](../logs/quick-create-workspace.md).

    >[!NOTE]
    >Включение мониторинга нескольких кластеров с одним и тем же именем кластера в одной рабочей области Log Analytics не поддерживается. Имена кластеров должны быть уникальными.
    >

- Вы являетесь членом **роли участника log Analytics** , чтобы включить мониторинг контейнеров. Дополнительные сведения об управлении доступом к рабочей области Log Analytics см. в разделе [Управление доступом к рабочей области и данным журнала](../logs/manage-access.md).

- Для просмотра данных мониторинга необходимо иметь роль [*log Analytics читателя*](../logs/manage-access.md#manage-access-using-azure-permissions) в рабочей области log Analytics, настроенную с помощью аналитики контейнера.

- [Helm клиент](https://helm.sh/docs/using_helm/) , чтобы подключить диаграмму Container Insights для указанного кластера Kubernetes.

- Следующие сведения о конфигурации прокси-сервера и брандмауэра необходимы для того, чтобы контейнерная версия агента Log Analytics для Linux могла взаимодействовать с Azure Monitor:

    |Ресурс агента|порты; |
    |------|---------|
    |*.ods.opinsights.azure.com |Порт 443 |
    |*.oms.opinsights.azure.com |Порт 443 |
    |*. dc.services.visualstudio.com |Порт 443 |

- Контейнерный агент требует, чтобы Kubelet `cAdvisor secure port: 10250` или `unsecure port :10255` был открыт на всех узлах кластера для сбора метрик производительности. Рекомендуется настроить `secure port: 10250` для CAdvisor Kubelet, если он еще не настроен.

- Контейнерный агент требует, чтобы в контейнере были указаны следующие переменные среды для взаимодействия со службой API Kubernetes в кластере для сбора данных инвентаризации — `KUBERNETES_SERVICE_HOST` и `KUBERNETES_PORT_443_TCP_PORT` .

>[!IMPORTANT]
>Минимальная версия агента, поддерживаемая для мониторинга гибридных кластеров Kubernetes, — ciprod10182019 или более поздняя.

## <a name="enable-monitoring"></a>Включение мониторинга

Включение аналитики контейнера для гибридного кластера Kubernetes состоит из выполнения следующих действий по порядку.

1. Настройте рабочую область Log Analytics с помощью решения Container Insights.   

2. Включите диаграмму HELM Container Insights с Log Analytics рабочей областью.

Дополнительные сведения о решениях мониторинга в Azure Monitor см. [здесь](../../azure-monitor/insights/solutions.md).

### <a name="how-to-add-the-azure-monitor-containers-solution"></a>Добавление решения "контейнеры Azure Monitor"

Вы можете развернуть решение с помощью указанного шаблона Azure Resource Manager, используя командлет Azure PowerShell `New-AzResourceGroupDeployment` или с помощью Azure CLI.

Если вы не знакомы с концепцией развертывания ресурсов с помощью шаблона, ознакомьтесь со статьями:

- [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)

- [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../../azure-resource-manager/templates/deploy-cli.md)

Если вы решили использовать Azure CLI, необходимо сначала установить интерфейс командной строки и использовать его локально. Необходимо запустить Azure CLI версии 2.0.59 или более поздней. Для определения версии выполните `az --version`. Если вам необходимо установить или обновить Azure CLI, ознакомьтесь со статьей [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

В этом методе используются два шаблона JSON. Один шаблон задает конфигурацию для включения мониторинга, а другой содержит значения параметров, которые можно настроить, чтобы указать следующее:

- **воркспацересаурцеид** — полный идентификатор ресурса рабочей области log Analytics.
- **воркспацерегион** — регион, в котором создана Рабочая область, которая также называется **расположением** в свойствах рабочей области при просмотре из портал Azure.

Чтобы сначала указать полный идентификатор ресурса Log Analytics рабочей области, необходимой для `workspaceResourceId` значения параметра в **containerSolutionParams.jsв** файле, выполните следующие действия, а затем выполните командлет PowerShell или команду Azure CLI, чтобы добавить решение.

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

    ```azurecli
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

    В выходных данных найдите имя рабочей области, а затем скопируйте полный идентификатор ресурса этой Log Analytics рабочей области под **идентификатором** поля.

4. Скопируйте и вставьте в него следующий синтаксис JSON:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace Resource ID"
            }
        },
        "workspaceRegion": {
            "type": "string",
            "metadata": {
                "description": "Azure Monitor Log Analytics Workspace region"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "name": "[Concat('ContainerInsights', '-',  uniqueString(parameters('workspaceResourceId')))]",
            "apiVersion": "2017-05-10",
            "subscriptionId": "[split(parameters('workspaceResourceId'),'/')[2]]",
            "resourceGroup": "[split(parameters('workspaceResourceId'),'/')[4]]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "apiVersion": "2015-11-01-preview",
                            "type": "Microsoft.OperationsManagement/solutions",
                            "location": "[parameters('workspaceRegion')]",
                            "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                            "properties": {
                                "workspaceResourceId": "[parameters('workspaceResourceId')]"
                            },
                            "plan": {
                                "name": "[Concat('ContainerInsights', '(', split(parameters('workspaceResourceId'),'/')[8], ')')]",
                                "product": "[Concat('OMSGallery/', 'ContainerInsights')]",
                                "promotionCode": "",
                                "publisher": "Microsoft"
                            }
                        }
                    ]
                },
                "parameters": {}
            }
         }
      ]
   }
    ```

5. Сохраните этот файл как containerSolution.jsв локальной папке.

6. Вставьте в него следующий синтаксис JSON:

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "workspaceResourceId": {
          "value": "<workspaceResourceId>"
      },
      "workspaceRegion": {
        "value": "<workspaceRegion>"
      }
     }
    }
    ```

7. Измените значения для **воркспацересаурцеид** , используя значение, скопированное на шаге 3, а для **Воркспацерегион** скопируйте значение **региона** после выполнения команды Azure CLI [AZ Monitor журнала-Analytics Workspace](/cli/azure/monitor/log-analytics/workspace#az-monitor-log-analytics-workspace-list&preserve-view=true).

8. Сохраните этот файл как containerSolutionParams.jsв локальной папке.

9. Теперь вы можете развернуть этот шаблон.

   - Чтобы выполнить развертывание с Azure PowerShell, используйте следующие команды в папке, содержащей шаблон:

       ```powershell
       # configure and login to the cloud of Log Analytics workspace.Specify the corresponding cloud environment of your workspace to below command.
       Connect-AzureRmAccount -Environment <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       ```

       ```powershell
       # set the context of the subscription of Log Analytics workspace
       Set-AzureRmContext -SubscriptionId <subscription Id of Log Analytics workspace>
       ```

       ```powershell
       # execute deployment command to add Container Insights solution to the specified Log Analytics workspace
       New-AzureRmResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <resource group of Log Analytics workspace> -TemplateFile .\containerSolution.json -TemplateParameterFile .\containerSolutionParams.json
       ```

       Изменение конфигурации может занять несколько минут. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

       ```powershell
       provisioningState       : Succeeded
       ```

   - Чтобы выполнить развертывание с Azure CLI, выполните следующие команды:

       ```azurecli
       az login
       az account set --name <AzureCloud | AzureChinaCloud | AzureUSGovernment>
       az login
       az account set --subscription "Subscription Name"
       # execute deployment command to add container insights solution to the specified Log Analytics workspace
       az deployment group create --resource-group <resource group of log analytics workspace> --name <deployment name> --template-file  ./containerSolution.json --parameters @./containerSolutionParams.json
       ```

       Изменение конфигурации может занять несколько минут. После завершения настройки конфигурации появится сообщение, похожее на приведенное ниже, с таким результатом:

       ```azurecli
       provisioningState       : Succeeded
       ```

       После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера.

## <a name="install-the-helm-chart"></a>Установка диаграммы HELM

В этом разделе вы устанавливаете контейнерный агент для аналитики контейнера. Прежде чем продолжать, необходимо указать идентификатор рабочей области, необходимый для `omsagent.secret.wsid` параметра, и первичный ключ, необходимый для `omsagent.secret.key` параметра. Эти сведения можно найти, выполнив следующие действия, а затем выполнить команды для установки агента с помощью диаграммы HELM.

1. Выполните следующую команду, чтобы обозначить идентификатор рабочей области:

    `az monitor log-analytics workspace list --resource-group <resourceGroupName>`

    В выходных данных найдите имя рабочей области под **именем** поля, а затем скопируйте идентификатор рабочей области log Analytics рабочей области в поле **CustomerID**.

2. Выполните следующую команду, чтобы обозначить первичный ключ для рабочей области:

    `az monitor log-analytics workspace get-shared-keys --resource-group <resourceGroupName> --workspace-name <logAnalyticsWorkspaceName>`

    В выходных данных найдите первичный ключ в поле **примаришаредкэй**, а затем скопируйте значение.

>[!NOTE]
>Следующие команды применимы только для Helm версии 2. Использование `--name` параметра не применимо к Helm версии 3. 

>[!NOTE]
>Если кластер Kubernetes обменивается данными через прокси-сервер, настройте параметр `omsagent.proxy` с URL-адресом прокси-сервера. Если кластер не обменивается данными через прокси-сервер, этот параметр указывать не нужно. Дополнительные сведения см. в разделе [Настройка конечной точки прокси-сервера](#configure-proxy-endpoint) далее в этой статье.

3. Добавьте репозиторий диаграмм Azure в локальный список, выполнив следующую команду:

    ```
    helm repo add microsoft https://microsoft.github.io/charts/repo
    ````

4. Установите диаграмму, выполнив следующую команду:

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<my_prod_cluster> microsoft/azuremonitor-containers
    ```

    Если Рабочая область Log Analytics находится в Azure для Китая (21Vianet), выполните следующую команду:

    ```
    $ helm install --name myrelease-1 \
     --set omsagent.domain=opinsights.azure.cn,omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

    Если рабочая область Log Analytics находится в Azure для государственных организаций США, выполните следующую команду.

    ```
    $ helm install --name myrelease-1 \
    --set omsagent.domain=opinsights.azure.us,omsagent.secret.wsid=<logAnalyticsWorkspaceId>,omsagent.secret.key=<logAnalyticsWorkspaceKey>,omsagent.env.clusterName=<your_cluster_name> incubator/azuremonitor-containers
    ```

### <a name="enable-the-helm-chart-using-the-api-model"></a>Включение диаграммы Helm с помощью модели API

Можно указать надстройку в файле JSON спецификации кластера AKS Engine, которая также называется моделью API. В этой надстройке укажите версию и Log Analytics рабочей области в кодировке Base64, в `WorkspaceGUID` `WorkspaceKey` которой хранятся собранные данные мониторинга. Вы можете найти `WorkspaceGUID` и `WorkspaceKey` использовать шаги 1 и 2 в предыдущем разделе.

Поддерживаемые определения API для кластера Azure Stack Hub можно найти в этом примере — [kubernetes-container-monitoring_existing_workspace_id_and_key.js](https://github.com/Azure/aks-engine/blob/master/examples/addons/container-monitoring/kubernetes-container-monitoring_existing_workspace_id_and_key.json). В частности, найдите свойство **addons** в **kubernetesConfig**:

```json
"orchestratorType": "Kubernetes",
       "kubernetesConfig": {
         "addons": [
           {
             "name": "container-monitoring",
             "enabled": true,
             "config": {
               "workspaceGuid": "<Azure Log Analytics Workspace Id in Base-64 encoded>",
               "workspaceKey": "<Azure Log Analytics Workspace Key in Base-64 encoded>"
             }
           }
         ]
       }
```

## <a name="configure-agent-data-collection"></a>Настройка сбора данных коллекции

С помощью схемы версии 1.0.0 параметры сбора данных агента контролируются из ConfigMap. См. документацию о параметрах сбора данных агента [здесь](container-insights-agent-config.md).

После успешного развертывания диаграммы вы можете просматривать данные для гибридного кластера Kubernetes в контейнере аналитики из портал Azure.  

>[!NOTE]
>Задержка приема составляет от агента до десяти минут до фиксации в рабочей области Azure Log Analytics. Состояние кластера отображает значение **нет данных** или **неизвестно** , пока все необходимые данные мониторинга не будут доступны в Azure Monitor.

## <a name="configure-proxy-endpoint"></a>Настройка конечной точки прокси-сервера

Начиная с версии диаграммы 2.7.1, диаграмма будет поддерживать Указание конечной точки прокси-сервера с помощью `omsagent.proxy` параметра диаграммы. Это позволяет ему взаимодействовать через прокси-сервер. Обмен данными между агентом Container Insights и Azure Monitor может быть прокси-сервером HTTP или HTTPS, и анонимная и обычная проверка подлинности (имя пользователя и пароль) поддерживаются.

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

Пример: `omsagent.proxy=http://user01:password@proxy01.contoso.com:8080`

Если указать протокол как **http**, HTTP-запросы создаются с помощью защищенного соединения SSL/TLS. Прокси-сервер должен поддерживать протоколы SSL/TLS.

## <a name="troubleshooting"></a>Устранение неполадок

Если при попытке включить мониторинг для гибридного кластера Kubernetes возникает ошибка, скопируйте сценарий PowerShell [TroubleshootError_nonAzureK8s.ps1](https://aka.ms/troubleshoot-non-azure-k8s) и сохраните его в папку на компьютере. Этот сценарий предназначен для обнаружения и устранения обнаруженных проблем. Проблемы, с которыми он предназначен для обнаружения и попыток исправления, приведены ниже.

- Указана допустимая рабочая область Log Analytics
- Рабочая область Log Analytics настроена с помощью решения Container Insights. В противном случае настройте рабочую область.
- Работают модули OmsAgent реплики
- Работают модули OmsAgent управляющей программы
- Служба работоспособности OmsAgent запущена
- ИДЕНТИФИКАТОР рабочей области Log Analytics и ключ, настроенные на контейнерном агенте, совпадают с рабочей областью, для которой настроена аналитика.
- Проверьте, что все рабочие узлы Linux имеют `kubernetes.io/role=agent` метку, чтобы запланировать модуль RS. Если он не существует, добавьте его.
- Проверка `cAdvisor secure port:10250` или `unsecure port: 10255` Открытие на всех узлах в кластере.

Для выполнения с Azure PowerShell используйте следующие команды в папке, содержащей скрипт:

```powershell
.\TroubleshootError_nonAzureK8s.ps1 - azureLogAnalyticsWorkspaceResourceId </subscriptions/<subscriptionId>/resourceGroups/<resourcegroupName>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName> -kubeConfig <kubeConfigFile> -clusterContextInKubeconfig <clusterContext>
```

## <a name="next-steps"></a>Дальнейшие действия

С включенным наблюдением для получения сведений о работоспособности и использовании ресурсов гибридного кластера Kubernetes и рабочих нагрузок на них вы узнаете, [как использовать](container-insights-analyze.md) аналитику контейнера.