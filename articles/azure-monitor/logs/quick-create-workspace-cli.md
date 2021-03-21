---
title: Создание рабочей области Log Analytics с помощью Azure CLI | Документация Майкрософт
description: Узнайте, как создать рабочую область Log Analytics, чтобы включить решения по управлению и сбор данных из облачной и локальной сред с помощью Azure CLI.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/26/2020
ms.openlocfilehash: 8e1fc816e32e563161e1604bdcd7a7006353e4ed
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102047389"
---
# <a name="create-a-log-analytics-workspace-with-azure-cli-20"></a>Создание рабочей области Log Analytics с помощью Azure CLI 2.0

Azure CLI 2.0 используется для создания ресурсов Azure и управления ими из командной строки или с помощью сценариев. В этом кратком руководстве показано, как развернуть рабочую область Log Analytics в Azure Monitor с помощью Azure CLI 2.0. Рабочая область Log Analytics — это уникальная среда для данных журналов Azure Monitor. Каждая рабочая область имеет свои репозиторий данных и конфигурации, при этом источники данных и решения настроены для хранения данных в определенной рабочей области. Вам понадобится рабочая область Log Analytics, если вы собираете данные из следующих источников:

* ресурсы Azure в подписке;  
* локальные компьютеры, которые отслеживает System Center Operations Manager;  
* коллекции устройств из Configuration Manager;  
* Данные диагностики и журнала из службы хранилища Azure  

Сведения о других источниках, таких как виртуальные машины Azure и виртуальные машины Windows или Linux в вашей среде, см. в статьях ниже.

* [Сбор данных о виртуальных машинах Azure](../vm/quick-collect-azurevm.md)
* [Настройка агента Log Analytics для компьютеров Linux в гибридной среде](../vm/quick-collect-linux-computer.md)
* [Настройка агента Log Analytics для компьютеров Windows в гибридной среде](../vm/quick-collect-windows-computer.md)

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.30 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-workspace"></a>Создание рабочей области
Создайте рабочую область с помощью команды [AZ Deployment Group Create](/cli/azure/deployment/group#az_deployment_group_create). В следующем примере создается рабочая область в регионе *eastus* с помощью шаблона Resource Manager на локальном компьютере. Шаблон в формате JSON настроен так, чтобы осталось только указать имя рабочей области и задать значения по умолчанию для других параметров, которые скорее всего будут использоваться в качестве стандартной конфигурации в вашей среде. Кроме того, шаблон можно хранить в учетной записи хранения Azure для общего доступа в организации. Дополнительную информацию о работе с шаблонами см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../../azure-resource-manager/templates/deploy-cli.md).

Сведения о поддерживаемых регионах см. в Анализе журналов [регионов, доступном в](https://azure.microsoft.com/regions/services/), и найдите Azure Monitor в поле **Поиск продукта**.

Для следующих параметров задаются значения по умолчанию.

* location — по умолчанию используется значение East US.
* sku — по умолчанию используется новый тарифный план с платой за гигабайт, выпущенный в апреле 2018 года.

>[!WARNING]
>При создании или настройке рабочей области Log Analytics в подписке, использующей модель ценообразования от апреля 2018 года, доступна только ценовая категория **PerGB2018**.
>

### <a name="create-and-deploy-template"></a>Создание и развертывание шаблона

1. Скопируйте и вставьте в него следующий синтаксис JSON:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "eastus",
              "westus"
            ],
            "defaultValue": "eastus",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        },
        "sku": {
            "type": "String",
            "allowedValues": [
              "Standalone",
              "PerNode",
              "PerGB2018"
            ],
            "defaultValue": "PerGB2018",
            "metadata": {
            "description": "Specifies the service tier of the workspace: Standalone, PerNode, Per-GB"
        }
          }
    },
    "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "sku": {
                    "Name": "[parameters('sku')]"
                },
                "features": {
                    "searchVersion": 1
                }
            }
          }
       ]
    }
    ```

2. Отредактируйте шаблон с учетом ваших требований. Просмотрите справочник по [шаблону Microsoft.OperationalInsights/workspaces](/azure/templates/microsoft.operationalinsights/2015-11-01-preview/workspaces) с описанием поддерживаемых свойств и значений.
3. Сохраните этот файл как **deploylaworkspacetemplate.json** в локальной папке.   
4. Теперь вы можете развернуть этот шаблон. Используйте следующие команды из папки, содержащей шаблон. При появлении запроса на ввод имени рабочей области укажите имя, глобально уникальное во всех подписках Azure.

    ```azurecli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deploylaworkspacetemplate.json
    ```

Развертывание может занять несколько минут. По завершении выполнения появится сообщение с результатами наподобие приведенного ниже.

![Пример результатов по завершении развертывания](media/quick-create-workspace-cli/template-output-01.png)

## <a name="troubleshooting"></a>Устранение неполадок
При создании рабочей области, которая была удалена в течение последних 14 дней и находится в [состоянии обратимого удаления](../logs/delete-workspace.md#soft-delete-behavior), операция может приводить к разным результатам в зависимости от конфигурации рабочей области.
1. Если вы указали те же параметры имени, группы ресурсов, подписки и региона, которые использовались для удаленной рабочей области, она будет полностью восстановлена с сохранением данных, конфигурации и подключенных агентов.
2. Если вы указали то же имя рабочей области, но другую группу ресурсов, подписку и (или) регион, появится ошибка с сообщением *Имя рабочей области "ИМЯ" не является уникальным* или *сообщение о конфликте*. Чтобы переопределить функцию обратимого удаления, то есть окончательно удалить рабочую область и создать новую рабочую область с тем же именем, выполните следующий процесс, в котором вы восстановите удаленную рабочую область и удалите ее без возможности восстановления:
   * [Восстановите](../logs/delete-workspace.md#recover-workspace) свою рабочую область.
   * [Безвозвратно удалите](../logs/delete-workspace.md#permanent-workspace-delete) рабочую область.
   * Создайте новую рабочую область с именем удаленной рабочей области.

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда рабочая область доступна, вы можете настроить сбор данных телеметрии для мониторинга, выполнять поиск по журналам для анализа этих данных, а также добавить решение по управлению для предоставления дополнительных данных и аналитических сведений.  

* Сведения о том, как включить сбор данных из ресурсов Azure с помощью системы диагностики Azure или службы хранилища Azure, см. в статье [Сбор журналов и метрик для служб Azure для использования в Log Analytics](../essentials/resource-logs.md#send-to-log-analytics-workspace).  
* Добавьте [System Center Operations Manager в качестве источника данных](../agents/om-agents.md), чтобы собирать данные с агентов, которые предоставляют отчеты группе управления Operations Manager, и хранить эти данные в рабочей области Log Analytics.  
* Подключите [Configuration Manager](../logs/collect-sccm.md) для импорта данных с компьютеров, которые являются элементами коллекций в иерархии.  
* Просмотрите список доступных [решений для мониторинга](../insights/solutions.md) и узнайте, как добавить решение в рабочую область или удалить его из нее.

