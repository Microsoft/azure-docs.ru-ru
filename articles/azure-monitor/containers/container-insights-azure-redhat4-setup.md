---
title: Настройка Azure Red Hat OpenShift v4. x с помощью Container Insights | Документация Майкрософт
description: В этой статье описывается настройка мониторинга для кластера Kubernetes с Azure Monitor, размещенной в Azure Red Hat OpenShift версии 4 или более поздней.
ms.topic: conceptual
ms.date: 03/05/2021
ms.openlocfilehash: 02cb794463b965ebafef0b6861477dbf69227511
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102506418"
---
# <a name="configure-azure-red-hat-openshift-v4x-with-container-insights"></a>Настройка Azure Red Hat OpenShift v4. x с помощью Container Insights

Служба "аналитика Azure" предоставляет широкие возможности мониторинга для кластеров Kubernetes Service (AKS) и AKS Engine. В этой статье описывается, как добиться аналогичного процесса мониторинга, включив мониторинг кластеров Kubernetes, размещенных в [Azure Red Hat OpenShift](../../openshift/intro-openshift.md) версии 4. x.

>[!NOTE]
>Поддержка Azure Red Hat OpenShift в настоящее время является функцией в общедоступной предварительной версии.
>

Вы можете включить аналитику контейнера для одного или нескольких существующих развертываний Azure Red Hat OpenShift v4. x, используя Поддерживаемые методы, описанные в этой статье.

Для существующего кластера запустите этот [скрипт Bash в Azure CLI](/cli/azure/openshift#az-openshift-create&preserve-view=true).

## <a name="supported-and-unsupported-features"></a>Поддерживаемые и неподдерживаемые функции

Служба "аналитика контейнеров" поддерживает мониторинг Azure Red Hat OpenShift v4. x, как описано в статье [Общие сведения о контейнере](container-insights-overview.md), за исключением следующих возможностей.

- Данные в режиме реального времени (Предварительная версия)
- [Сбор метрик](container-insights-update-metrics.md) из узлов кластера и модулей Pod и их сохранение в базе данных метрик Azure Monitor

## <a name="prerequisites"></a>Предварительные требования

- Azure CLI версии 2.0.72 или более поздней.  

- Средство CLI [Helm 3](https://helm.sh/docs/intro/install/)

- [Bash версии 4](https://www.gnu.org/software/bash/)

- Программа командной строки [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- [Рабочая область Log Analytics](../logs/design-logs-deployment.md).

    Служба "аналитика контейнеров" поддерживает Log Analytics рабочей области в регионах, перечисленных в списке [продуктов Azure по регионам](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=monitor). Чтобы создать собственную рабочую область, ее можно создать с помощью [Azure Resource Manager](../logs/resource-manager-workspace.md), [PowerShell](../logs/powershell-sample-create-workspace.md?toc=%2fpowershell%2fmodule%2ftoc.json)или в [портал Azure](../logs/quick-create-workspace.md).

- Для включения и доступа к функциям в контейнере аналитики необходимо, как минимум, роль *участника* Azure в подписке Azure и роль [*участника log Analytics*](../logs/manage-access.md#manage-access-using-azure-permissions) в рабочей области log Analytics, настроенную с помощью аналитики контейнеров.

- Для просмотра данных мониторинга необходимо иметь роль [*log Analytics читателя*](../logs/manage-access.md#manage-access-using-azure-permissions) в рабочей области log Analytics, настроенную с помощью аналитики контейнера.

## <a name="enable-monitoring-for-an-existing-cluster"></a>Включение мониторинга для существующего кластера

Чтобы включить мониторинг для кластера Azure Red Hat OpenShift версии 4 или более поздней, который развернут в Azure с помощью предоставленного скрипта bash, выполните следующие действия.

1. Войдите в Azure, выполнив следующую команду.

    ```azurecli
    az login
    ```

1. Скачайте и сохраните в локальной папке скрипт, который настраивает кластер с помощью надстройки мониторинга, выполнив следующую команду:

    `curl -o enable-monitoring.sh -L https://aka.ms/enable-monitoring-bash-script`

1. Подключитесь к кластеру АТО v4 с помощью инструкций в разделе [учебник. подключение к кластеру Azure Red Hat OpenShift 4](../../openshift/tutorial-connect-cluster.md).


### <a name="integrate-with-an-existing-workspace"></a>Интеграция с существующей рабочей областью

В этом разделе описано, как включить мониторинг кластера с помощью скрипта bash, скачанного ранее. Чтобы выполнить интеграцию с существующей рабочей областью Log Analytics, начните с определения полного идентификатора ресурса Log Analytics рабочей области, необходимой для `logAnalyticsWorkspaceResourceId` параметра, а затем выполните команду, чтобы включить надстройку мониторинга для указанной рабочей области.

Если у вас нет рабочей области для указания, можно перейти к разделу [Интеграция с рабочей областью по умолчанию](#integrate-with-the-default-workspace) и позволить скрипту создать новую рабочую область.

1. Перечислите все подписки, к которым у вас есть доступ, выполнив следующую команду:

    ```azurecli
    az account list --all -o table
    ```

    Выходные данные будут выглядеть следующим образом:

    ```azurecli
    Name                                  CloudName    SubscriptionId                        State    IsDefault
    ------------------------------------  -----------  ------------------------------------  -------  -----------
    Microsoft Azure                       AzureCloud   0fb60ef2-03cc-4290-b595-e71108e8f4ce  Enabled  True
    ```

1. Скопируйте значение для **SubscriptionId**.

1. Переключитесь на подписку, в которой размещена рабочая область Log Analytics, выполнив следующую команду:

    ```azurecli
    az account set -s <subscriptionId of the workspace>
    ```

1. Отобразите список рабочих областей в подписках в формате JSON по умолчанию, выполнив следующую команду:

    ```
    az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json
    ```

1. В выходных данных найдите имя рабочей области, а затем скопируйте полный идентификатор ресурса этой Log Analytics рабочей области под **идентификатором** поля.

1. Чтобы включить мониторинг, выполните следующую команду. Замените значения для `azureAroV4ClusterResourceId` `logAnalyticsWorkspaceResourceId` параметров и.

    ```bash
    export azureAroV4ClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.RedHatOpenShift/OpenShiftClusters/<clusterName>"
    export logAnalyticsWorkspaceResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>" 
    ```

    Ниже приведена команда, которую необходимо выполнить после заполнения 3 переменных командами Export:

    `bash enable-monitoring.sh --resource-id $azureAroV4ClusterResourceId --workspace-id $logAnalyticsWorkspaceResourceId`

После включения мониторинга для просмотра метрик работоспособности кластера может потребоваться около 15 минут.

### <a name="integrate-with-the-default-workspace"></a>Интеграция с рабочей областью по умолчанию

В этом разделе описано, как включить мониторинг кластера Azure Red Hat OpenShift v4. x с помощью скачанного скрипта bash.

В этом примере вам не требуется предварительно создавать или указывать существующую рабочую область. Эта команда упрощает процесс, создавая рабочую область по умолчанию в группе ресурсов по умолчанию подписки на кластер, если она еще не существует в регионе.

Созданная Рабочая область по умолчанию имеет формат *DefaultWorkspace- \<GUID> - \<Region>*.  

Замените значение для `azureAroV4ClusterResourceId` параметра.

```bash
export azureAroV4ClusterResourceId="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.RedHatOpenShift/OpenShiftClusters/<clusterName>"
```

Пример:

"Bash enable-monitoring.sh--Resource-id $azureAroV 4ClusterResourceId 

После включения мониторинга может пройти около 15 минут, прежде чем вы сможете просмотреть метрики работоспособности кластера.

### <a name="enable-monitoring-from-the-azure-portal"></a>Включение мониторинга из портал Azure

В представлении с несколькими кластерами в контейнере аналитики для кластеров Azure Red Hat OpenShift, для которых не включен мониторинг, используется вкладка **Неотслеживаемые кластеры** . Параметр **Enable (включить** ) рядом с кластером не инициирует подключение мониторинга с портала. Вы будете перенаправлены к этой статье, чтобы включить мониторинг вручную, выполнив действия, описанные ранее в этой статье.

1. Войдите на [портал Azure](https://portal.azure.com).

1. На левой панели или на домашней странице выберите **Azure Monitor**.

1. В разделе **аналитика** выберите **контейнеры**.

1. На странице **мониторинг — контейнеры** выберите **Неотслеживаемые кластеры**.

1. В списке неотслеживаемых кластеров выберите кластер, а затем щелкните **включить**.

    Результаты в списке можно найти, выполнив поиск значения **АТО** в столбце **тип кластера** . После выбора **включить** вы будете перенаправлены к этой статье.

## <a name="next-steps"></a>Дальнейшие действия

- Теперь, когда вы включили функцию мониторинга для получения сведений о работоспособности и использовании ресурсов в кластере RedHat OpenShift версии 4. x и рабочих нагрузках, которые выполняются на них, Узнайте, [как использовать](container-insights-analyze.md) аналитику контейнера.

- По умолчанию контейнерный агент собирает журналы контейнеров *stdout* и *stderr* всех контейнеров, которые выполняются во всех пространствах имен, за исключением KUBE-System. Чтобы настроить коллекцию журналов контейнеров, относящуюся к определенному пространству имен или пространству имен, ознакомьтесь с разделом [Конфигурация агента аналитики контейнеров](container-insights-agent-config.md) , чтобы настроить необходимые параметры сбора данных для файла конфигурации *ConfigMap* .

- Чтобы отбракировать и проанализировать метрики Prometheus из кластера, см. раздел [Настройка брака Prometheus метрик](container-insights-prometheus-integration.md).

- Дополнительные сведения о том, как отключить мониторинг кластера с помощью Azure Insights, см. в статье [как отключить мониторинг кластера OpenShift для Red Hat](./container-insights-optout-openshift-v3.md).
