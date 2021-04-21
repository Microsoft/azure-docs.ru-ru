---
title: Краткое руководство. Создание учетной записи автоматизации с помощью шаблона Azure
titleSuffix: Azure Automation
description: В этом кратком руководстве показано, как создать учетную запись автоматизации с помощью шаблона Azure Resource Manager.
services: automation
ms.author: magoedte
ms.date: 01/07/2021
ms.topic: quickstart
ms.workload: infrastructure-services
ms.custom:
- mvc
- subject-armqs
- mode-arm
ms.openlocfilehash: 50a8c057a6fdf6350a18225cd2dc538942d6686d
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538403"
---
# <a name="quickstart-create-an-automation-account-by-using-arm-template"></a>Краткое руководство. Создание учетной записи автоматизации с помощью шаблона Resource Manager

Служба автоматизации Azure — это облачная служба автоматизации и настройки, которая поддерживает согласованное управление в Azure и других средах. В этом кратком руководстве показано, как развернуть шаблон Azure Resource Manager (шаблон ARM), который создает учетную запись автоматизации. При использовании шаблона Resource Manager выполняется меньшее количество действий по сравнению с другими методами развертывания.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-automation%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

В этом примере шаблона выполняются следующие действия:

* автоматизирует создание рабочей области Log Analytics Azure Monitor;
* автоматизирует создание учетной записи службы автоматизации Azure;
* связывает учетную запись службы автоматизации с рабочей областью Log Analytics.
* добавляет примеры модулей Runbook автоматизации в учетную запись.

>[!NOTE]
>Создание учетной записи запуска от имени службы автоматизации на основе шаблона Azure Resource Manager не поддерживается. Сведения о создании учетной записи запуска от имени вручную на портале или с помощью PowerShell см. в статье [Создание учетной записи запуска от имени](create-run-as-account.md).

После выполнения этих действий необходимо [настроить параметры диагностики](automation-manage-send-joblogs-log-analytics.md), чтобы учетная запись службы автоматизации могла отправлять состояния заданий Runbook и потоки заданий в связанную рабочую область Log Analytics.

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-automation/).

:::code language="json" source="~/quickstart-templates/101-automation/azuredeploy.json":::

### <a name="api-versions"></a>Версии API

В следующей таблице перечислены версии API для ресурсов, используемых в этом примере.

| Ресурс | Тип ресурса | Версия API |
|:---|:---|:---|
| [Рабочая область](/azure/templates/microsoft.operationalinsights/workspaces) | workspaces | 2020-03-01-preview |
| [Учетная запись службы автоматизации](/azure/templates/microsoft.automation/automationaccounts) | служба автоматизации | 2020-01-13-preview |
| [Связанные службы рабочей области](/azure/templates/microsoft.operationalinsights/workspaces/linkedservices) | workspaces | 2020-03-01-preview |

### <a name="before-you-use-the-template"></a>Перед использованием шаблона

Шаблон параметров JSON настраивается, чтобы можно было указать:

* имя рабочей области;
* регион, в котором будет создана рабочая область;
* имя учетной записи службы автоматизации;
* регион, в котором будет создана учетная запись автоматизации.

Для следующих параметров в шаблоне задано значение по умолчанию для рабочей области Log Analytics:

* *sku* — по умолчанию используется ценовая категория с платой за гигабайт, выпущенная в апреле 2018 года.
* *dataRetention* — по умолчанию задано 30 дней.

>[!WARNING]
>Если вы хотите создать или настроить рабочую область Log Analytics в подписке, использующей модель ценообразования от апреля 2018 года, будет доступна только ценовая категория *PerGB2018*.
>

Шаблон в формате JSON указывает значения по умолчанию для других параметров, которые, скорее всего, будут использоваться в качестве стандартной конфигурации в вашей среде. Шаблон можно сохранить в учетной записи хранения Azure для совместного использования в пределах организации. Дополнительную информацию о работе с шаблонами см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../azure-resource-manager/templates/deploy-cli.md).

Если вы не знакомы со службами автоматизации Azure и Azure Monitor, важно понимать следующие моменты, относящиеся к конфигурации. Они могут помочь избежать ошибок при создании, настройке и использовании рабочей области Log Analytics, связанной с новой учетной записью службы автоматизации.

* Ознакомьтесь с [дополнительными сведениями](../azure-monitor/logs/resource-manager-workspace.md#create-a-log-analytics-workspace), чтобы полностью понять возможности конфигурации рабочей области, такие как режим управления доступом, ценовая категория, срок хранения и уровень резервирования мощности.

* Изучите [сопоставления рабочей области](how-to/region-mappings.md), чтобы указывать поддерживаемые регионы в строке или файле параметров. Только определенные регионы поддерживают связывание рабочей области Log Analytics и учетной записи службы автоматизации в подписке.

* Если вы еще не работали с журналами Azure Monitor и пока не развернули рабочую область, ознакомьтесь с [руководством по проектированию рабочей области](../azure-monitor/logs/design-logs-deployment.md). В нем содержатся сведения об управлении доступом и приводятся стратегии реализации разработок, которые рекомендуется внедрить в организации.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает учетную запись службы автоматизации Azure, рабочую область Log Analytics и связывает учетную запись службы автоматизации с рабочей областью.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-automation%2Fazuredeploy.json)

2. Введите значения.

3. Развертывание может занять несколько минут. По завершении выходные данные должны быть следующего вида:

    ![Пример результатов по завершении развертывания](media/quickstart-create-automation-account-template/template-output.png)

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Войдите на [портал Azure](https://portal.azure.com).

2. На портале Azure откройте учетную запись службы автоматизации, которую вы только что создали. 

3. В области слева выберите **Модули Runbook**. На странице **модулей Runbook** перечислены три учебных модуля Runbook, созданные с помощью учетной записи службы автоматизации.

    ![Учебные модули Runbook, созданные с помощью учетной записи службы автоматизации](./media/quickstart-create-automation-account-template/automation-sample-runbooks.png)

4. В области слева выберите **Связанная рабочая область**. На странице **связанной рабочей области** отображается рабочая область Log Analytics, указанная ранее, которая связана с учетной записью службы автоматизации.

    ![Учетная запись службы автоматизации, связанная с рабочей областью Log Analytics](./media/quickstart-create-automation-account-template/automation-account-linked-workspace.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если они больше не нужны, удалите связь между учетной записью службы автоматизации и рабочей области Log Analytics, а затем — учетную запись и рабочую область службы автоматизации.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы создали учетную запись службы автоматизации, рабочую область Log Analytics и связали их.

Чтобы узнать больше о службе автоматизации Azure, ознакомьтесь с соответствующими учебниками.

> [!div class="nextstepaction"]
> [Учебники по созданию службы автоматизации Azure](learn/automation-tutorial-runbook-graphical.md)
