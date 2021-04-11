---
title: Краткое руководство. Создание схемы с помощью Azure CLI
description: В рамках этого краткого руководства вы будете использовать Azure Blueprints для создания, определения и развертывания артефактов с помощью Azure CLI.
ms.date: 01/27/2021
ms.topic: quickstart
ms.openlocfilehash: fbe5c12f1c94d4b59dbdc2a97b6a4cb9af5a2328
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105563673"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-azure-cli"></a>Краткое руководство. Определение и назначение схемы Azure с помощью Azure CLI

Зная, как создавать и назначать схемы, вы сможете определять типичные шаблоны для разработки многоразовых и быстро развертываемых конфигураций на основе шаблонов Azure Resource Manager (шаблонов ARM), политик, правил безопасности и т. п. В этом руководстве вы научитесь использовать службу Azure Blueprint для выполнения некоторых общих задач, связанных с созданием, публикацией и назначением схемы в вашей организации, таких как:

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free), прежде чем начинать работу.
- Если вы ранее не использовали Azure Blueprints, зарегистрируйте поставщик ресурсов, выполнив команду `az provider register --namespace Microsoft.Blueprint` в Azure CLI.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-blueprint-extension"></a>Добавление расширения схемы

Чтобы разрешить Azure CLI управлять определениями и назначениями схем, необходимо добавить расширение.
Это расширение работает везде, где может использоваться оболочка Azure CLI, включая [bash на операционной системе Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (как автономно, так и внутри портала), [образ Azure CLI Docker](https://hub.docker.com/_/microsoft-azure-cli) или установленную локально.

1. Убедитесь, что установлена последняя версия Azure CLI (не ниже **2.0.76**). Если она еще не установлена, выполните [эти инструкции](/cli/azure/install-azure-cli-windows).

1. В выбранной вами среде Azure CLI импортируйте ее с помощью следующей команды:

   ```azurecli-interactive
   # Add the Blueprint extension to the Azure CLI environment
   az extension add --name blueprint
   ```

1. Проверьте установку расширения и номер версии (не ниже **0.1.0**):

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for extension options
   az blueprint -h
   ```

## <a name="create-a-blueprint"></a>Создание схемы

Первым этапом при определении стандартной модели соответствия требованиям является формирование схемы на основе доступных ресурсов. Мы создадим схему с именем MyBlueprint для настройки роли и назначения политик для подписки. Затем мы добавим группу ресурсов, шаблон ARM и назначение ролей группе ресурсов.

> [!NOTE]
> При использовании Azure CLI сначала создается объект _схемы_. Для каждого _артефакта_ с параметрами, который необходимо добавить, необходимо заранее определить параметры в исходной _схеме_.

1. Создайте исходный объект _схемы_. Параметр **parameters** принимает JSON-файл, содержащий все параметры уровня схемы. Эти параметры задаются при назначении и используются артефактами, которые будут добавлены в дальнейшем.

   - JSON-файл — blueprintparms.json

     ```json
     {
        "storageAccountType": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "allowedValues": [
                "Standard_LRS",
                "Standard_GRS",
                "Standard_ZRS",
                "Premium_LRS"
            ],
            "metadata": {
                "displayName": "storage account type.",
                "description": null
            }
        },
        "tagName": {
            "type": "string",
            "metadata": {
                "displayName": "The name of the tag to provide the policy assignment.",
                "description": null
            }
        },
        "tagValue": {
            "type": "string",
            "metadata": {
                "displayName": "The value of the tag to provide the policy assignment.",
                "description": null
            }
        },
        "contributors": {
            "type": "array",
            "metadata": {
                "description": "List of AAD object IDs that is assigned Contributor role at the subscription",
                "strongType": "PrincipalId"
            }
        },
        "owners": {
            "type": "array",
            "metadata": {
                "description": "List of AAD object IDs that is assigned Owner role at the resource group",
                "strongType": "PrincipalId"
            }
        }
     }
     ```

   - Команда Azure CLI

     ```azurecli-interactive
     # Login first with az login if not using Cloud Shell

     # Create the blueprint object
     az blueprint create \
        --name 'MyBlueprint' \
        --description 'This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.' \
        --parameters blueprintparms.json
     ```

     > [!NOTE]
     > При импорте определений схем используйте имя файла _blueprint.json_.
     > Это имя файла используется при вызове [az blueprint import](/cli/azure/ext/blueprint/blueprint#ext_blueprint_az_blueprint_import).

     По умолчанию объект схемы создается в подписке, заданной по умолчанию. Чтобы указать группу управления, используйте параметр **managementgroup**. Чтобы указать подписку, используйте параметр **subscription**.

1. Добавьте в определение группу ресурсов для артефактов хранилища.

   ```azurecli-interactive
   az blueprint resource-group add \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'storageRG' \
      --description 'Contains the resource template deployment and a role assignment.'
   ```

1. Добавьте назначение ролей в подписку. В приведенном ниже примере удостоверения субъектов, которым предоставляется указанная роль, передаются в параметре, задаваемом во время назначения схемы. В этом примере используется встроенная роль _Участник_ с глобальным уникальным идентификатором `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   ```azurecli-interactive
   az blueprint artifact role create \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'roleContributor' \
      --role-definition-id '/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c' \
      --principal-ids "[parameters('contributors')]"
   ```

1. Добавьте назначение политики в подписку. В этом примере используется встроенная политика "_Задать тег и его значение по умолчанию для групп ресурсов_" с глобальным уникальным идентификатором `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - JSON-файл — artifacts\policyTags.json

     ```json
     {
        "tagName": {
           "value": "[parameters('tagName')]"
        },
        "tagValue": {
           "value": "[parameters('tagValue')]"
        }
     }
     ```

   - Команда Azure CLI

     ```azurecli-interactive
     az blueprint artifact policy create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'policyTags' \
        --policy-definition-id '/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71' \
        --display-name 'Apply tag and its default value to resource groups' \
        --description 'Apply tag and its default value to resource groups' \
        --parameters artifacts\policyTags.json
     ```

     > [!NOTE]
     > При использовании `az blueprint` на компьютерах Mac замените `\` на `/` в значениях параметров, которые включают путь. В этом случае значение **parameters** примет вид `artifacts/policyTags.json`.

1. Добавьте в подписку еще одно назначение политики для тега хранилища (повторно используя параметр _storageAccountType_). Этот дополнительный артефакт назначения политики демонстрирует, что определенный в схеме параметр может использоваться несколькими артефактами. В этом примере **storageAccountType** используется для настройки тега группы ресурсов. Это значение предоставляет сведения об учетной записи хранения, которая будет создана на следующем шаге. В этом примере используется встроенная политика "_Задать тег и его значение по умолчанию для групп ресурсов_" с глобальным уникальным идентификатором `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - JSON-файл — artifacts\policyStorageTags.json

     ```json
     {
        "tagName": {
           "value": "StorageType"
        },
        "tagValue": {
           "value": "[parameters('storageAccountType')]"
        }
     }
     ```

   - Команда Azure CLI

     ```azurecli-interactive
     az blueprint artifact policy create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'policyStorageTags' \
        --policy-definition-id '/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71' \
        --display-name 'Apply storage tag to resource group' \
        --description 'Apply storage tag and the parameter also used by the template to resource groups' \
        --parameters artifacts\policyStorageTags.json
     ```

     > [!NOTE]
     > При использовании `az blueprint` на компьютерах Mac замените `\` на `/` в значениях параметров, которые включают путь. В этом случае значение **parameters** примет вид `artifacts/policyStorageTags.json`.

1. Добавьте шаблон в группу ресурсов. Параметр **template** для шаблона ARM содержит обычные компоненты JSON шаблона. Этот шаблон также повторно использует параметры схемы **storageAccountType**, **tagName** и **tagValue**, передавая каждый из них в шаблон. Параметры схемы доступны шаблону за счет использования параметра **parameters**. В коде JSON-шаблона эта пара "ключ-значение" используется для внедрения значения. Имена параметров схемы и шаблона могут быть одинаковыми.

   - Файл JSON шаблона ARM — \artifacts\templateStorage.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "type": "string",
                 "metadata": {
                     "description": "Storage Account type"
                 }
             },
             "tagNameFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag name from blueprint"
                 }
             },
             "tagValueFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag value from blueprint"
                 }
             }
         },
         "variables": {
             "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
         },
         "resources": [{
             "type": "Microsoft.Storage/storageAccounts",
             "name": "[variables('storageAccountName')]",
             "apiVersion": "2016-01-01",
             "tags": {
                 "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
             },
             "location": "[resourceGroup().location]",
             "sku": {
                 "name": "[parameters('storageAccountTypeFromBP')]"
             },
             "kind": "Storage",
             "properties": {}
         }],
         "outputs": {
             "storageAccountSku": {
                 "type": "string",
                 "value": "[variables('storageAccountName')]"
             }
         }
     }
     ```

   - Файл JSON параметров шаблона ARM — \artifacts\templateStorageParams.json

     ```json
     {
        "storageAccountTypeFromBP": {
           "value": "[parameters('storageAccountType')]"
        },
        "tagNameFromBP": {
           "value": "[parameters('tagName')]"
        },
        "tagValueFromBP": {
           "value": "[parameters('tagValue')]"
        }
     }
     ```

   - Команда Azure CLI

     ```azurecli-interactive
     az blueprint artifact template create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'templateStorage' \
        --template artifacts\templateStorage.json \
        --parameters artifacts\templateStorageParams.json \
        --resource-group-art 'storageRG'
     ```

     > [!NOTE]
     > При использовании `az blueprint` на компьютерах Mac замените `\` на `/` в значениях параметров, которые включают путь. В этом случае значение **template** примет виде `artifacts/templateStorage.json`, а **parameters** — `artifacts/templateStorageParams.json`.

1. Добавьте назначение ролей в группу ресурсов. Так же как и в предыдущем фрагменте назначения ролей, в приведенном ниже примере используется идентификатор для роли **Владелец**, но из схемы для него передается другой параметр. В этом примере используется встроенная роль _Владелец_ с глобальным уникальным идентификатором `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   ```azurecli-interactive
   az blueprint artifact role create \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'roleOwner' \
      --role-definition-id '/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635' \
      --principal-ids "[parameters('owners')]" \
      --resource-group-art 'storageRG'
   ```

## <a name="publish-a-blueprint"></a>Публикация схемы

Теперь, когда артефакты добавлены в схему, пора ее опубликовать. Публикация схемы позволяет назначать ее подписке.

```azurecli-interactive
az blueprint publish --blueprint-name 'MyBlueprint' --version '{BlueprintVersion}'
```

Значение `{BlueprintVersion}` представляет собой строку, состоящую из букв, цифр и дефисов (без пробелов или других специальных символов), содержащую не более 20 символов. Оно должно быть уникальным и информативным, например **v20200605-135541**.

## <a name="assign-a-blueprint"></a>Назначение схемы

После публикации схемы с помощью Azure CLI ее можно назначить подписке. Назначьте созданную схему одной из подписок в иерархии группы управления. Если схема сохранена в подписке, схему можно будет назначить только этой подписке. Параметр **blueprint-name** определяет схему для назначения. Чтобы указать параметры имени, расположения, идентификатора, блокировки и схемы, используйте соответствующие параметры Azure CLI в команде `az blueprint assignment create` или укажите их в параметре JSON-файла **parameters**.

1. Запустите развертывание схемы, назначив ее подписке. Так как в параметрах **contributors** и **owners** должны быть переданы массивы идентификаторов objectId субъектов, которым должно быть предоставлено назначение ролей, используйте [API Graph Azure Active Directory](/graph/migrate-azure-ad-graph-planning-checklist) для сбора идентификаторов objectId, которые вы затем будете использовать в параметре **parameters** для собственных пользователей, групп или субъектов-служб.

   - JSON-файл blueprintAssignment.json

     ```json
     {
        "storageAccountType": {
            "value": "Standard_GRS"
        },
        "tagName": {
            "value": "CostCenter"
        },
        "tagValue": {
            "value": "ContosoIT"
        },
        "contributors": {
            "value": [
                "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                "38833b56-194d-420b-90ce-cff578296714"
            ]
        },
        "owners": {
            "value": [
                "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                "316deb5f-7187-4512-9dd4-21e7798b0ef9"
            ]
        }
     }
     ```

   - Команда Azure CLI

     ```azurecli-interactive
     az blueprint assignment create \
        --name 'assignMyBlueprint' \
        --location 'westus' \
        --resource-group-value artifact_name=storageRG name=StorageAccount location=eastus \
        --parameters blueprintAssignment.json
     ```

   - Управляемое удостоверение, назначаемое пользователем

     Для назначения схемы можно также использовать [управляемое удостоверение, назначаемое пользователем](../../active-directory/managed-identities-azure-resources/overview.md).
     В этом случае для параметра **identity-type** задается значение _UserAssigned_, а параметр **user-assigned-identitie** задает удостоверение. Замените `{userIdentity}` именем вашего управляемого удостоверения, назначаемого пользователем.

     ```azurecli-interactive
     az blueprint assignment create \
        --name 'assignMyBlueprint' \
        --location 'westus' \
        --identity-type UserAssigned \
        --user-assigned-identities {userIdentity} \
        --resource-group-value artifact_name=storageRG name=StorageAccount location=eastus \
        --parameters blueprintAssignment.json
     ```

     **Управляемое удостоверение, назначаемое пользователем**, может быть в любой подписке и группе ресурсов, к которым у пользователя, назначающего схему, есть разрешения на доступ.

     > [!IMPORTANT]
     > Azure Blueprints не администрирует управляемое удостоверение, назначаемое пользователем. Назначение соответствующих ролей и разрешений выполняют пользователи. В противном случае назначение схемы завершится ошибкой.

## <a name="clean-up-resources"></a>Очистка ресурсов

### <a name="unassign-a-blueprint"></a>Отмена назначения схемы

Вы можете удалить схему из подписки. Удаление часто выполняется, когда ресурсы артефакта больше не требуются. При удалении схемы назначенные артефакты оставляются. Чтобы удалить назначение схемы, используйте команду `az blueprint assignment delete`:

```azurecli-interactive
az blueprint assignment delete --name 'assignMyBlueprint'
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы создали, назначили и удалили схему с помощью Azure CLI. Чтобы узнать больше о схемах Azure, перейдите к статье о жизненном цикле схемы.

> [!div class="nextstepaction"]
> [Ознакомьтесь со сведениями о жизненном цикле схем](./concepts/lifecycle.md)