---
title: Включение надстройки мониторинга AKS с помощью политики Azure
description: Описание включения надстройки мониторинга AKS с помощью настраиваемой политики Azure.
ms.topic: conceptual
ms.date: 02/04/2021
ms.openlocfilehash: 2163527cc83e70913e9a6e11bf2e22f9ed9c6690
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101713904"
---
# <a name="enable-aks-monitoring-addon-using-azure-policy"></a>Включение надстройки мониторинга AKS с помощью политики Azure
В этой статье описывается, как включить надстройку мониторинга AKS с помощью настраиваемой политики Azure. Настраиваемую политику надстройки для мониторинга можно назначить в области действия подписки или группы ресурсов. Если Рабочая область Azure Log Analytics и кластер AKS находятся в разных подписках, то управляемое удостоверение, используемое назначением политики, должно иметь необходимые разрешения роли как для подписки, так и для ресурса в Log Analytics рабочей области. Аналогично, если политика ограничена группой ресурсов, то управляемое удостоверение должно иметь необходимые разрешения роли в рабочей области Log Analytics, если Рабочая область не входит в область выбранной группы ресурсов.

Для наблюдения за надстройкой требуются следующие роли в управляемом удостоверении, используемом политикой Azure:

 - [Azure-kubernetes-Service-участник — роль](../../role-based-access-control/built-in-roles.md#azure-kubernetes-service-contributor-role)
 - [Журнал-аналитика-участник](../../role-based-access-control/built-in-roles.md#log-analytics-contributor)

## <a name="create-and-assign-policy-definition-using-azure-portal"></a>Создание и назначение определения политики с помощью портал Azure

### <a name="create-policy-definition"></a>Создание определения политики

1. Скачайте определение пользовательской политики Azure, чтобы включить надстройку мониторинга AKS.
 
    ``` sh
    curl -o azurepolicy.json -L https://aka.ms/aks-enable-monitoring-custom-policy
    ```

3. Перейдите к https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyMenuBlade/Definitions и создайте определение политики со следующими сведениями в диалоговом окне Создание определения политики.
 
    - **Расположение определения**: выберите подписку Azure, в которой должно храниться определение политики.
    - **Имя**: *(Предварительная версия) AKS-Monitoring-надстройка*
    - **Описание**. *настраиваемая политика Azure, позволяющая включить мониторинг надстроек в кластерах Azure Kubernetes в указанной области* .
    - **Категория**: выберите *использовать существующий* и выберите *Kubernetes* из раскрывающегося списка.
    - **Правило политики**. Удалите существующие образцы правил и скопируйте содержимое *azurepolicy.js* , скачанного на шаге #1 выше.

### <a name="assign-policy-definition-to-specified-scope"></a>Назначение определения политики указанной области

> [!NOTE]
>  Управляемое удостоверение будет создано автоматически и назначены указанные роли в определении политики.

1. Выберите только что созданную надстройку "определение политики *(Предварительная версия)" AKS Monitoring* .
4. Щелкните *назначить** * и укажите **область** , в которой должна быть назначена политика. 
5. Нажмите кнопку **Далее** и укажите идентификатор ресурса для рабочей области Azure log Analytics. Идентификатор ресурса должен быть в этом формате `/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>` .
6. Создайте задачу исправления в случае, если вы хотите применить политику к существующим кластерам AKS в выбранной области.
7. Щелкните **проверить и создать** , чтобы создать назначение политики.
   
## <a name="create-and-assign-policy-definition-using-azure-cli"></a>Создание и назначение определения политики с помощью Azure CLI

### <a name="create-policy-definition"></a>Создание определения политики

1. Скачайте правила определения настраиваемой политики Azure и файлы параметров с помощью следующих команд:

    ``` sh
    curl -o azurepolicy.rules.json -L https://aka.ms/aks-enable-monitoring-custom-policy-rules
    curl -o azurepolicy.parameters.json -L https://aka.ms/aks-enable-monitoring-custom-policy-parameters
    ```

2. Создайте определение политики с помощью следующей команды:

    ``` sh
    az cloud set -n <AzureCloud | AzureChinaCloud | AzureUSGovernment> # set the Azure cloud
    az login # login to cloud environment 
    az account set -s <subscriptionId>
    az policy definition create --name "(Preview)AKS-Monitoring-Addon" --display-name "(Preview)AKS-Monitoring-Addon" --mode Indexed --metadata version=1.0.0 category=Kubernetes --rules azurepolicy.rules.json --params azurepolicy.parameters.json
    ```

### <a name="assign-policy-definition-to-specified-scope"></a>Назначение определения политики указанной области

- Создайте назначение политики с помощью следующей команды:

    ``` sh
    az policy assignment create --name aks-monitoring-addon --policy "(Preview)AKS-Monitoring-Addon" --assign-identity --identity-scope /subscriptions/<subscriptionId> --role Contributor --scope /subscriptions/<subscriptionId> --location <locatio> --role Contributor --scope /subscriptions/<subscriptionId> -p "{ \"workspaceResourceId\": { \"value\":  \"/subscriptions/<subscriptionId>/resourcegroups/<resourceGroupName>/providers/microsoft.operationalinsights/workspaces/<workspaceName>\" } }"
    ```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [политике Azure](../../governance/policy/overview.md).
- Узнайте, как [работает защита от исправлений](../../governance/policy/how-to/remediate-resources.md#how-remediation-security-works).
- Дополнительные сведения о [контейнере аналитики](./container-insights-overview.md).
- Установка [Azure CLI](/cli/azure/install-azure-cli).