---
title: Включение Управления обновлениями с помощью шаблона Azure Resource Manager
description: В этой статье описывается, как использовать шаблон Azure Resource Manager для включения Управления обновлениями.
services: automation
ms.subservice: update-management
ms.topic: conceptual
ms.date: 09/18/2020
ms.openlocfilehash: 95ef52acedc9171ba86110a665d08ea97c59bfbb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100575830"
---
# <a name="enable-update-management-using-azure-resource-manager-template"></a>Включение Управления обновлениями с помощью шаблона Azure Resource Manager

Для включения Управления обновлениями службы автоматизации Azure в группе ресурсов можно использовать [шаблоны Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md). В этой статье приводится пример шаблона, который автоматизирует выполнение следующих задач:

* автоматизирует создание рабочей области Log Analytics Azure Monitor;
* автоматизирует создание учетной записи службы автоматизации Azure;
* связывает учетную запись службы автоматизации с рабочей областью Log Analytics.
* добавляет примеры модулей Runbook автоматизации в учетную запись.
* Включает функцию Управление обновлениями.

Этот шаблон не позволяет автоматизировать включение Управление обновлениями на одной или нескольких виртуальных машинах Azure или не в Azure.

Если у вас уже есть рабочая область Log Analytics и учетная запись службы автоматизации, развернутая в поддерживаемом регионе в вашей подписке, они не связаны между собой. Использование этого шаблона позволяет успешно создавать ссылку и развертывать Управление обновлениями.

>[!NOTE]
>Создание учетной записи запуска от имени службы автоматизации на основе шаблона Azure Resource Manager не поддерживается. Сведения о создании учетной записи запуска от имени вручную на портале или с помощью PowerShell см. в статье [Создание учетной записи запуска от имени](../create-run-as-account.md).

После выполнения этих действий необходимо [настроить параметры диагностики](../automation-manage-send-joblogs-log-analytics.md), чтобы учетная запись службы автоматизации могла отправлять состояния заданий Runbook и потоки заданий в связанную рабочую область Log Analytics.

## <a name="api-versions"></a>Версии API

В следующей таблице перечислены версии API для ресурсов, используемых в этом примере.

| Ресурс | Тип ресурса | Версия API |
|:---|:---|:---|
| [Рабочая область](/azure/templates/microsoft.operationalinsights/workspaces) | workspaces | 2020-03-01-preview |
| [Учетная запись службы автоматизации](/azure/templates/microsoft.automation/automationaccounts) | служба автоматизации | 2020-01-13-preview |
| [Связанные службы рабочей области](/azure/templates/microsoft.operationalinsights/workspaces/linkedservices) | workspaces | 2020-03-01-preview |
| [Решения](/azure/templates/microsoft.operationsmanagement/solutions) | solutions | 2015-11-01-preview |

## <a name="before-using-the-template"></a>Перед применением шаблона

Шаблон JSON настроен так, чтобы осталось только указать следующие сведения:

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

Шаблон в формате JSON указывает значения по умолчанию для других параметров, которые, скорее всего, будут использоваться в качестве стандартной конфигурации в вашей среде. Шаблон можно сохранить в учетной записи хранения Azure для совместного использования в пределах организации. Дополнительную информацию о работе с шаблонами см. в статье [Развертывание ресурсов с использованием шаблонов Resource Manager и Azure CLI](../../azure-resource-manager/templates/deploy-cli.md).

Если вы не знакомы со службами автоматизации Azure и Azure Monitor, важно понимать следующие моменты, относящиеся к конфигурации. Они могут помочь избежать ошибок при создании, настройке и использовании рабочей области Log Analytics, связанной с новой учетной записью службы автоматизации.

* Ознакомьтесь с [дополнительными сведениями](../../azure-monitor/logs/resource-manager-workspace.md#create-a-log-analytics-workspace), чтобы полностью понять возможности конфигурации рабочей области, такие как режим управления доступом, ценовая категория, срок хранения и уровень резервирования мощности.

* Изучите [сопоставления рабочей области](../how-to/region-mappings.md), чтобы указывать поддерживаемые регионы в строке или файле параметров. Только определенные регионы поддерживают связывание рабочей области Log Analytics и учетной записи службы автоматизации в подписке.

* Если вы еще не работали с журналами Azure Monitor и пока не развернули рабочую область, ознакомьтесь с [руководством по проектированию рабочей области](../../azure-monitor/logs/design-logs-deployment.md). В нем содержатся сведения об управлении доступом и приводятся стратегии реализации разработок, которые рекомендуется внедрить в организации.

## <a name="deploy-template"></a>Развертывание шаблона

1. Скопируйте и вставьте в него следующий синтаксис JSON:

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string",
                "metadata": {
                    "description": "Workspace name"
                }
            },
            "sku": {
                "type": "string",
                "allowedValues": [
                    "pergb2018",
                    "Free",
                    "Standalone",
                    "PerNode",
                    "Standard",
                    "Premium"
                ],
                "defaultValue": "pergb2018",
                "metadata": {
                    "description": "Pricing tier: perGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium), which are not available to all customers."
                }
            },
            "dataRetention": {
                "type": "int",
                "defaultValue": 30,
                "minValue": 7,
                "maxValue": 730,
                "metadata": {
                    "description": "Number of days to retain data."
                }
            },
            "location": {
                "type": "string",
                "defaultValue": "[resourceGroup().location]",
                "metadata": {
                    "description": "Specifies the location in which to create the workspace."
                }
            },
            "automationAccountName": {
                "type": "string",
                "metadata": {
                    "description": "Automation account name"
                }
            },
            "automationAccountLocation": {
                "type": "string",
                "metadata": {
                    "description": "Specifies the location in which to create the Automation account."
                }
            },
            "sampleGraphicalRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorial"
            },
            "sampleGraphicalRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePowerShellRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialScript"
            },
            "samplePowerShellRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePython2RunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialPython2"
            },
            "samplePython2RunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "_artifactsLocation": {
                "type": "string",
                "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-automation/",
                "metadata": {
                    "description": "URI to artifacts location"
                }
            },
            "_artifactsLocationSasToken": {
                "type": "securestring",
                "defaultValue": "",
                "metadata": {
                    "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated"
                }
            }
        },
        "variables": {
        "Updates": {
            "name": "[concat('Updates', '(', parameters('workspaceName'), ')')]",
            "galleryName": "Updates"
          }
        },
        "resources": [
            {
                "type": "Microsoft.OperationalInsights/workspaces",
                "apiVersion": "2020-03-01-preview",
                "name": "[parameters('workspaceName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "sku": {
                        "name": "[parameters('sku')]"
                    },
                    "retentionInDays": "[parameters('dataRetention')]",
                    "features": {
                        "searchVersion": 1,
                        "legacy": 0
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "location": "[parameters('location')]",
                "name": "[variables('Updates').name]",
                "type": "Microsoft.OperationsManagement/solutions",
                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.OperationsManagement/solutions/', variables('Updates').name)]",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                ],
                "properties": {
                    "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                },
                "plan": {
                    "name": "[variables('Updates').name]",
                    "publisher": "Microsoft",
                    "promotionCode": "",
                    "product": "[concat('OMSGallery/', variables('Updates').galleryName)]"
                }
            },
            {
                "type": "Microsoft.Automation/automationAccounts",
                "apiVersion": "2020-01-13-preview",
                "name": "[parameters('automationAccountName')]",
                "location": "[parameters('automationAccountLocation')]",
                "dependsOn": [
                    "[parameters('workspaceName')]"
                ],
                "properties": {
                    "sku": {
                        "name": "Basic"
                    }
                },
                "resources": [
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('sampleGraphicalRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "GraphPowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('sampleGraphicalRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.graphrunbook', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePowerShellRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "PowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePowerShellRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.ps1', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePython2RunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "Python2",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePython2RunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorialPython2.py', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    }
                ]
            },
            {
                "type": "Microsoft.OperationalInsights/workspaces/linkedServices",
                "apiVersion": "2020-03-01-preview",
                "name": "[concat(parameters('workspaceName'), '/' , 'Automation')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[parameters('workspaceName')]",
                    "[parameters('automationAccountName')]"
                ],
                "properties": {
                    "resourceId": "[resourceId('Microsoft.Automation/automationAccounts', parameters('automationAccountName'))]"
                }
            }
        ]
    }
    ```

2. Отредактируйте шаблон с учетом ваших требований. Вместо встраивания параметров в строку запуска попробуйте создать [файл параметров Resource Manager](../../azure-resource-manager/templates/parameter-files.md).

3. Сохраните этот файл с именем **deployUMSolutiontemplate.json** в локальной папке.

4. Теперь вы можете развернуть этот шаблон. Это можно сделать с помощью PowerShell или Azure CLI. При появлении запроса на ввод имени рабочей области и учетной записи службы автоматизации укажите имя, глобально уникальное во всех подписках Azure.

    **PowerShell**

    ```powershell
    New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile deployUMSolutiontemplate.json
    ```

    **Azure CLI**

    ```azurecli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deployUMSolutiontemplate.json
    ```

    Развертывание может занять несколько минут. По завершении выполнения появится сообщение с результатами наподобие приведенного ниже.

    ![Пример результатов по завершении развертывания](media/enable-from-template/template-output.png)

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Войдите на [портал Azure](https://portal.azure.com).

2. В портал Azure откройте созданную учетную запись службы автоматизации.

3. В области слева выберите **Модули Runbook**. На странице **модулей Runbook** перечислены три учебных модуля Runbook, созданные с помощью учетной записи службы автоматизации.

    ![Учебные модули Runbook, созданные с помощью учетной записи службы автоматизации](../media/quickstart-create-automation-account-template/automation-sample-runbooks.png)

4. В области слева выберите **Связанная рабочая область**. На странице **связанной рабочей области** отображается рабочая область Log Analytics, указанная ранее, которая связана с учетной записью службы автоматизации.

    ![Учетная запись службы автоматизации, связанная с рабочей областью Log Analytics](../media/quickstart-create-automation-account-template/automation-account-linked-workspace.png)

5. В левой области выберите **Управление обновлениями**. На странице **Управление обновлениями** отображается страница оценка без каких-либо сведений в результате включения только что включенных, а компьютеры не настроены для управления.

    ![Управление обновлениями представление оценки возможностей](./media/enable-from-template/update-management-assessment-view.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если они больше не нужны, удалите решение " **обновления** " в рабочей области log Analytics, снимите связь между учетной записью службы автоматизации и рабочей областью, а затем удалите учетную запись службы автоматизации и рабочую область.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об использовании Управление обновлениями для виртуальных машин см. в статье [Управление обновлениями и исправлениями для виртуальных машин](manage-updates-for-vm.md).

* Если вы больше не хотите использовать Управление обновлениями и хотите удалить его, см. инструкции по [удалению Управление обновлениями компонента](remove-feature.md).

* Дополнительные сведения см. в статье [Удаление виртуальной машины из Управления обновлениями](remove-vms.md).