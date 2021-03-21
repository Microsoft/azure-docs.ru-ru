---
title: Настройка рабочей области Log Analytics для VM Insights
description: Описание создания и настройки рабочей области Log Analytics, используемой службой "аналитика VM".
ms.topic: conceptual
ms.custom: references_regions
author: bwren
ms.author: bwren
ms.date: 12/22/2020
ms.openlocfilehash: 5a0e04772cfc1c9be77c0ad8b32b0e93be8b9f54
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102046726"
---
# <a name="configure-log-analytics-workspace-for-vm-insights"></a>Настройка рабочей области Log Analytics для VM Insights
VM Insights собирает свои данные из одной или нескольких рабочих областей Log Analytics в Azure Monitor. Перед подключением агентов необходимо создать и настроить рабочую область. В этой статье описываются требования к рабочей области и ее настройка для VM Insights.

## <a name="overview"></a>Обзор
Одна подписка может использовать любое количество рабочих областей в зависимости от требований. Единственное требование к рабочей области заключается в том, что она находится в поддерживаемом расположении и настроена с помощью решения *вминсигхтс* .

После настройки рабочей области можно использовать любой из доступных параметров для установки необходимых агентов на виртуальной машине и масштабируемом наборе виртуальных машин, а также указать рабочую область для отправки их данных. VM Insights будет получать данные из любой настроенной рабочей области в своей подписке.

> [!NOTE]
> При включении функции "аналитика VM" на одной виртуальной машине или в масштабируемом наборе виртуальных машин с помощью портал Azure вы можете выбрать существующую рабочую область или создать новую. Решение *вминсигхтс* будет установлено в этой рабочей области, если это еще не сделано. Затем эту рабочую область можно использовать для других агентов.


## <a name="create-log-analytics-workspace"></a>Создание рабочей области Log Analytics

>[!NOTE]
>Сведения, описанные в этом разделе, также применимы к [решению сопоставление служб](service-map.md). 

Доступ к Log Analytics рабочим областям в портал Azure из меню **log Analytics рабочие области** .

[![Рабочие области Анлитикс журналов](media/vminsights-configure-workspace/log-analytics-workspaces.png)](media/vminsights-configure-workspace/log-analytics-workspaces.png#lightbox)

Новую рабочую область Log Analytics можно создать одним из следующих способов. Инструкции по определению количества рабочих областей, которые следует использовать в вашей среде, а также о проектировании стратегии доступа см. в разделе [проектирование развертывания журналов Azure Monitor](../logs/design-logs-deployment.md) .


* [Портал Azure](../logs/quick-create-workspace.md)
* [Azure CLI](../logs/quick-create-workspace-cli.md)
* [PowerShell](../logs/powershell-workspace-configuration.md)
* [Azure Resource Manager](../logs/resource-manager-workspace.md)

## <a name="supported-regions"></a>Поддерживаемые регионы
VM Insights поддерживает Log Analytics рабочей области в любом из [регионов, поддерживаемых log Analytics,](https://azure.microsoft.com/global-infrastructure/services/?products=monitor&regions=all) за исключением следующих:

- Центрально-Западная Германия
- Республика Корея, центральный регион

>[!NOTE]
>Вы можете отслеживать виртуальные машины Azure в любом регионе. Сами виртуальные машины не ограничиваются регионами, поддерживаемыми Log Analytics рабочей областью.

## <a name="azure-role-based-access-control"></a>Управление доступом на основе ролей в Azure
Чтобы включить функции и получить доступ к ним в компоненте "аналитика VM", необходимо иметь [роль участника log Analytics](../logs/manage-access.md#manage-access-using-azure-permissions) в рабочей области. Для просмотра данных о производительности, работоспособности и сопоставлении необходимо иметь [роль "читатель мониторинга](../roles-permissions-security.md#built-in-monitoring-roles) " для виртуальной машины Azure. Дополнительные сведения о том, как управлять доступом к рабочей области Log Analytics, см. в статье [Управление рабочими областями](../logs/manage-access.md).

## <a name="add-vminsights-solution-to-workspace"></a>Добавление решения Вминсигхтс в рабочую область
Прежде чем Log Analytics рабочую область можно будет использовать с VM Insights, на ней должно быть установлено решение *вминсигхтс* . Методы настройки рабочей области описаны в следующих разделах.

> [!NOTE]
> При добавлении решения *вминсигхтс* в рабочую область все существующие виртуальные машины, подключенные к рабочей области, начнут отсылать данные в инсигхтсметрикс. Данные для других типов данных не будут собираться, пока вы не добавите Dependency Agent к этим существующим виртуальным машинам, подключенным к рабочей области.

### <a name="azure-portal"></a>Портал Azure
Существует три варианта настройки существующей рабочей области с помощью портал Azure. Каждая составляющая формулы описана ниже.

Чтобы настроить одну рабочую область, перейдите к параметру **виртуальные машины** в меню **Azure Monitor** , выберите **другие параметры адаптации**, а затем **Настройте рабочую область**. Выберите подписку и рабочую область и нажмите кнопку **настроить**.

[![Настройка рабочей области](../vm/media/vminsights-enable-policy/configure-workspace.png)](../vm/media/vminsights-enable-policy/configure-workspace.png#lightbox)

Чтобы настроить несколько рабочих областей, выберите вкладку **Конфигурация рабочей области** в меню **виртуальные машины** в портал Azure.  Установите значения фильтра, чтобы отобразить список существующих рабочих областей. Установите флажок рядом с каждой рабочей областью, чтобы включить ее, а затем нажмите кнопку **настроить выбранные**.

[![Конфигурация рабочей области](../vm/media/vminsights-enable-policy/workspace-configuration.png)](../vm/media/vminsights-enable-policy/workspace-configuration.png#lightbox)


При включении функции "аналитика VM" на одной виртуальной машине или в масштабируемом наборе виртуальных машин с помощью портал Azure вы можете выбрать существующую рабочую область или создать новую. Решение *вминсигхтс* будет установлено в этой рабочей области, если это еще не сделано. Затем эту рабочую область можно использовать для других агентов.

[![Включение отдельной виртуальной машины на портале](../vm/media/vminsights-enable-portal/enable-vminsights-vm-portal.png)](../vm/media/vminsights-enable-portal/enable-vminsights-vm-portal.png#lightbox)


### <a name="resource-manager-template"></a>Шаблон Resource Manager
Шаблоны Azure Resource Manager для VM Insights предоставляются в файле архива (ZIP-файл), который можно [скачать из нашего репозитория GitHub](https://aka.ms/VmInsightsARMTemplates). Сюда входит шаблон с именем **конфигуреворкспаце** , который настраивает log Analytics рабочую область для VM Insights. Этот шаблон развертывается с помощью любого из стандартных методов, включая примеры команд PowerShell и CLI, приведенные ниже. 

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az deployment group create --name ConfigureWorkspace --resource-group my-resource-group --template-file CreateWorkspace.json  --parameters workspaceResourceId='/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace' workspaceLocation='eastus'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
New-AzResourceGroupDeployment -Name ConfigureWorkspace -ResourceGroupName my-resource-group -TemplateFile ConfigureWorkspace.json -workspaceResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace -location eastus
```

---



## <a name="next-steps"></a>Дальнейшие действия
- Сведения о подключении агентов к службе "аналитика виртуальных машин" см. в статье подключение [агентов к VM Insights](vminsights-enable-overview.md) .
- См. раздел [нацеленность решений мониторинга в Azure Monitor (Предварительная версия)](../insights/solution-targeting.md) , чтобы ограничить объем данных, отправляемых из решения в рабочую область.
