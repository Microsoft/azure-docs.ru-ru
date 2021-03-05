---
title: Azure CLI пример скрипта. Создание приложения логики
description: Пример скрипта для создания приложения логики с помощью расширения Logic Apps в Azure CLI.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.custom: mvc, devx-track-azurecli
ms.date: 07/30/2020
ms.openlocfilehash: a4553ceee482fb232e9ab56deca650be93f9dc6b
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102218049"
---
# <a name="azure-cli-script-sample---create-a-logic-app"></a>Azure CLI пример скрипта. Создание приложения логики

Этот скрипт создает пример приложения логики с помощью [расширения Azure CLI Logic Apps](/cli/azure/ext/logic/logic)( `az logic` ). Подробное руководство по созданию приложений логики и управлению ими с помощью Azure CLI см. в [Logic Apps краткого](quickstart-logic-apps-azure-cli.md)руководства по Azure CLI.

> [!WARNING]
> Расширение Azure CLI Logic Apps в настоящее время *экспериментальное*, на него не предоставляется *клиентская поддержка*. Используйте это расширение CLI с осторожностью, особенно в рабочих средах.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Azure CLI](/cli/azure/install-azure-cli), установленный на локальном компьютере.
* Расширение [Logic Apps Azure CLI](/cli/azure/azure-cli-extensions-list), установленное на компьютере. Чтобы установить это расширение, используйте следующую команду: `az extension add --name logic`
* [Определение рабочего процесса](quickstart-logic-apps-azure-cli.md#workflow-definition) для приложения логики. Этот JSON-файл должен соответствовать [схеме языка определения рабочих процессов](logic-apps-workflow-definition-language.md).
* Подключение API к учетной записи электронной почты через поддерживаемый [соединитель Logic Apps](../connectors/apis-list.md) в той же группе ресурсов, что и приложение логики. В этом примере используется соединитель [Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md) , но можно также использовать другие соединители, такие как [Outlook.com](../connectors/connectors-create-api-outlook.md).

### <a name="prerequisite-check"></a>Проверка предварительных условий

Перед началом работы проверьте среду, выполнив следующие действия.

* Войдите на портал Azure и убедитесь, что ваша подписка активна, выполнив команду `az login`.

* Проверьте версию Azure CLI в терминале или в командном окне, выполнив команду `az --version`. Сведения о последней версии см. в [заметках о выпуске](/cli/azure/release-notes-azure-cli).

  * Если у вас нет последней версии, обновите установку, следуя указаниям в [руководстве по установке операционной системы или платформы](/cli/azure/install-azure-cli).

### <a name="sample-workflow-explanation"></a>Пример описания рабочего процесса

В этом примере файл определения рабочего процесса создает то же основное приложение логики, что и [Logic Apps краткого руководства по портал Azure](quickstart-create-first-logic-app-workflow.md). 

Этот пример рабочего процесса: 

1. Указывает схему `$schema` для приложения логики.

1. Определяет триггер для приложения логики в списке триггеров `triggers` . Триггер рекурсии ( `recurrence` ) каждые 3 часа. Действия запускаются при публикации нового элемента веб-канала ( `When_a_feed_item_is_published` ) для УКАЗАННОГО RSS-канала ( `feedUrl` ).

1. Определяет действие для приложения логики в списке действий `actions` . Действие отправляет сообщение электронной почты ( `Send_an_email_(V2)` ) через Microsoft 365 со сведениями из элементов RSS-канала, как указано в разделе текста ( `body` ) входных данных действия ( `inputs` ).

## <a name="sample-workflow-definition"></a>Пример определения рабочего процесса

Перед запуском примера скрипта необходимо сначала создать пример [определения рабочего процесса](#prerequisites).

1. Создайте JSON-файл `testDefinition.json` на своем компьютере. 

1. Скопируйте следующее содержимое в файл JSON: 
    ```json
    
    {
        "definition": {
            "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
            "actions": {
                "Send_an_email_(V2)": {
                    "inputs": {
                        "body": {
                            "Body": "<p>@{triggerBody()?['publishDate']}<br>\n@{triggerBody()?['title']}<br>\n@{triggerBody()?['primaryLink']}</p>",
                            "Subject": "@triggerBody()?['title']",
                            "To": "test@example.com"
                        },
                        "host": {
                            "connection": {
                                "name": "@parameters('$connections')['office365']['connectionId']"
                            }
                        },
                        "method": "post",
                        "path": "/v2/Mail"
                    },
                    "runAfter": {},
                    "type": "ApiConnection"
                }
            },
            "contentVersion": "1.0.0.0",
            "outputs": {},
            "parameters": {
                "$connections": {
                    "defaultValue": {},
                    "type": "Object"
                }
            },
            "triggers": {
                "When_a_feed_item_is_published": {
                    "inputs": {
                        "host": {
                            "connection": {
                                "name": "@parameters('$connections')['rss']['connectionId']"
                            }
                        },
                        "method": "get",
                        "path": "/OnNewFeed",
                        "queries": {
                            "feedUrl": "https://www.pbs.org/now/rss.xml"
                        }
                    },
                    "recurrence": {
                        "frequency": "Hour",
                        "interval": 3
                    },
                    "splitOn": "@triggerBody()?['value']",
                    "type": "ApiConnection"
                }
            }
        },
        "parameters": {
            "$connections": {
                "value": {
                    "office365": {
                        "connectionId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testResourceGroup/providers/Microsoft.Web/connections/office365",
                        "connectionName": "office365",
                        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Web/locations/westus/managedApis/office365"
                    },
                    "rss": {
                        "connectionId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testResourceGroup/providers/Microsoft.Web/connections/rss",
                        "connectionName": "rss",
                        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Web/locations/westus/managedApis/rss"
                    }
                }
            }
        }
    }
    
    ```

1. Замените значения заполнителей собственными сведениями:

    1. Замените адрес электронной почты заполнителя ( `"To": "test@example.com"` ). Необходимо использовать адрес электронной почты, совместимый с соединителями Logic Apps. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites).

    1. Замените дополнительные сведения о соединителе, если вы используете другой соединитель электронной почты, отличный от соединителя Office 365 Outlook.

    1. Замените заполнители значения подписок ( `00000000-0000-0000-0000-000000000000` ) для идентификаторов подключения ( `connectionId` и `id` ) в параметре Connections ( `$connections` ) собственными значениями подписки.

1. Сохраните изменения.

## <a name="sample-script"></a>Пример скрипта

> [!NOTE]
> Этот пример написан для `bash` оболочки. Если вы хотите выполнить этот пример в другой оболочке, такой как Windows PowerShell или Командная строка, может потребоваться внести изменения в скрипт.

Перед запуском этого примера скрипта выполните следующую команду для подключения к Azure:

```azurecli-interactive

az login

```

Затем перейдите к каталогу, в котором вы создали определение рабочего процесса. Например, если вы создали JSON-файл определения рабочего стола на рабочем столе:

```azurecli

cd ~/Desktop

```

Затем запустите этот скрипт, чтобы создать приложение логики. 

```azurecli-interactive

#!/bin/bash

# Create a resource group

az group create --name testResourceGroup --location westus

# Create your logic app

az logic workflow create --resource-group "testResourceGroup" --location "westus" --name "testLogicApp" --definition "testDefinition.json"

```

### <a name="clean-up-deployment"></a>Очистка развертывания

Завершив использование примера сценария, выполните следующую команду, чтобы удалить группу ресурсов и все ее вложенные ресурсы, включая приложение логики.

```azurecli-interactive

az group delete --name testResourceGroup --yes

```

## <a name="script-explanation"></a>Описание скрипта

Этот пример скрипта использует следующие команды для создания новой группы ресурсов и приложения логики.

| Get-Help | Примечания |
| ------- | ----- |
| [`az group create`](/cli/azure/group#az-group-create) | Создает группу ресурсов, в которой хранятся ресурсы приложения логики. |
| [`az logic workflow create`](/cli/azure/ext/logic/logic/workflow#ext-logic-az-logic-workflow-create) | Создает приложение логики на основе рабочего процесса, определенного в параметре `--definition` . |
| [`az group delete`](/cli/azure/vm/extension) | Удаляет группу ресурсов и все ее вложенные ресурсы. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure CLI см. в [документации по Azure CLI](/cli/azure/).

Дополнительные примеры скриптов CLI для Logic Apps можно найти в [браузере примеров кода Майкрософт](/samples/browse/?products=azure-logic-apps).
