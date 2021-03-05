---
title: Связывание шаблонов для развертывания
description: Описывает, как использовать связанные шаблоны в шаблоне Azure Resource Manager (шаблон ARM) для создания решения модульного шаблона. Показывает, как передавать значения параметров, указывать файл параметров и динамически создаваемые URL-адреса.
ms.topic: conceptual
ms.date: 01/26/2021
ms.openlocfilehash: 3636ea64227a7c013134d96647144d4f1e2ae31e
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102211317"
---
# <a name="using-linked-and-nested-templates-when-deploying-azure-resources"></a>Использование связанных и вложенных шаблонов при развертывании ресурсов Azure

Для развертывания сложных решений можно разбить шаблон Azure Resource Manager (шаблон ARM) на множество связанных шаблонов, а затем развернуть их вместе с помощью основного шаблона. Связанные шаблоны могут быть отдельными файлами или синтаксисом шаблонов, внедренными в основной шаблон. В этой статье используется термин **связанный шаблон** для ссылки на отдельный файл шаблона, на который есть ссылка через ссылку из основного шаблона. Он использует термин **вложенный шаблон** для ссылки на синтаксис внедренного шаблона в основном шаблоне.

Для небольших и средних решений отдельный шаблон проще в понимании и обслуживании. Все ресурсы и значения можно увидеть в отдельном файле. Для более сложных сценариев связанные шаблоны позволяют разбить решение на целевые компоненты. Эти шаблоны можно многократно использовать в других сценариях.

Руководство см. в разделе [учебник. Развертывание связанного шаблона](./deployment-tutorial-linked-template.md).

> [!NOTE]
> Для связанных или вложенных шаблонов можно задать только [добавочный](deployment-modes.md)режим развертывания. Однако основной шаблон можно развернуть в полном режиме. Если главный шаблон развертывается в режиме полного выполнения, а связанный или вложенный шаблон предназначен для одной и той же группы ресурсов, то ресурсы, развернутые в связанном или вложенном шаблоне, включаются в оценку развертывания полного режима. Объединенная коллекция ресурсов, развернутых в основном шаблоне, а также связанные или вложенные шаблоны, сравниваются с существующими ресурсами в группе ресурсов. Все ресурсы, не входящие в эту объединенную коллекцию, будут удалены.
>
> Если связанный или вложенный шаблон предназначен для другой группы ресурсов, это развертывание использует добавочный режим.
>

## <a name="nested-template"></a>Вложенный шаблон

Чтобы вложить шаблон, добавьте [ресурс развертывания](/azure/templates/microsoft.resources/deployments) в основной шаблон. В `template` свойстве укажите синтаксис шаблона.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "nestedTemplate1",
      "properties": {
        "mode": "Incremental",
        "template": {
          <nested-template-syntax>
        }
      }
    }
  ],
  "outputs": {
  }
}
```

В следующем примере учетная запись хранения развертывается с помощью вложенного шаблона.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "nestedTemplate1",
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2019-04-01",
              "name": "[parameters('storageAccountName')]",
              "location": "West US",
              "sku": {
                "name": "Standard_LRS"
              },
              "kind": "StorageV2"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
  }
}
```

### <a name="expression-evaluation-scope-in-nested-templates"></a>Область вычисления выражения во вложенных шаблонах

При использовании вложенного шаблона можно указать, должны ли его выражения вычисляться в области родительского шаблона или вложенного шаблона. Область определяет, как разрешаются параметры, переменные и функции, такие как [resourceGroup](template-functions-resource.md#resourcegroup) и [Подписка](template-functions-resource.md#subscription) .

Область задается через `expressionEvaluationOptions` свойство. По умолчанию `expressionEvaluationOptions` свойство имеет значение, то `outer` есть использует область родительского шаблона. Установите значение, чтобы `inner` выражения выводились в области вложенного шаблона.

```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2019-10-01",
  "name": "nestedTemplate1",
  "properties": {
    "expressionEvaluationOptions": {
      "scope": "inner"
    },
  ...
```

> [!NOTE]
>
> Если параметр scope имеет значение `outer` , нельзя использовать `reference` функцию в разделе Outputs вложенного шаблона для ресурса, развернутого во вложенном шаблоне. Чтобы получить значения для развернутого ресурса во вложенном шаблоне, используйте `inner` область или преобразуйте вложенный шаблон в связанный шаблон.

В следующем шаблоне показано, как выражения шаблона разрешаются в соответствии с областью. Он содержит переменную с именем `exampleVar` , которая определена как в родительском, так и во вложенном шаблоне. Он возвращает значение переменной.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  },
  "variables": {
    "exampleVar": "from parent template"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "nestedTemplate1",
      "properties": {
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "variables": {
            "exampleVar": "from nested template"
          },
          "resources": [
          ],
          "outputs": {
            "testVar": {
              "type": "string",
              "value": "[variables('exampleVar')]"
            }
          }
        }
      }
    }
  ],
  "outputs": {
    "messageFromLinkedTemplate": {
      "type": "string",
      "value": "[reference('nestedTemplate1').outputs.testVar.value]"
    }
  }
}
```

Значение изменяется в `exampleVar` зависимости от значения `scope` свойства в `expressionEvaluationOptions` . В следующей таблице показаны результаты для обеих областей.

| Область оценки | Выходные данные |
| ----- | ------ |
| Внутреннее | из вложенного шаблона |
| внешний (или по умолчанию) | из родительского шаблона |

В следующем примере выполняется развертывание SQL Server и получение секрета хранилища ключей, используемого для пароля. Область имеет значение, `inner` так как она динамически создает идентификатор хранилища ключей (см `adminPassword.reference.keyVault` . во внешних шаблонах `parameters` ) и передает его в качестве параметра вложенному шаблону.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "The location where the resources will be deployed."
      }
    },
    "vaultName": {
      "type": "string",
      "metadata": {
        "description": "The name of the keyvault that contains the secret."
      }
    },
    "secretName": {
      "type": "string",
      "metadata": {
        "description": "The name of the secret."
      }
    },
    "vaultResourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "The name of the resource group that contains the keyvault."
      }
    },
    "vaultSubscription": {
      "type": "string",
      "defaultValue": "[subscription().subscriptionId]",
      "metadata": {
        "description": "The name of the subscription that contains the keyvault."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "dynamicSecret",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "parameters": {
          "location": {
            "value": "[parameters('location')]"
          },
          "adminLogin": {
            "value": "ghuser"
          },
          "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "[resourceId(parameters('vaultSubscription'), parameters('vaultResourceGroupName'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
              },
              "secretName": "[parameters('secretName')]"
            }
          }
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "adminLogin": {
              "type": "string"
            },
            "adminPassword": {
              "type": "securestring"
            },
            "location": {
              "type": "string"
            }
          },
          "variables": {
            "sqlServerName": "[concat('sql-', uniqueString(resourceGroup().id, 'sql'))]"
          },
          "resources": [
            {
              "type": "Microsoft.Sql/servers",
              "apiVersion": "2018-06-01-preview",
              "name": "[variables('sqlServerName')]",
              "location": "[parameters('location')]",
              "properties": {
                "administratorLogin": "[parameters('adminLogin')]",
                "administratorLoginPassword": "[parameters('adminPassword')]"
              }
            }
          ],
          "outputs": {
            "sqlFQDN": {
              "type": "string",
              "value": "[reference(variables('sqlServerName')).fullyQualifiedDomainName]"
            }
          }
        }
      }
    }
  ],
  "outputs": {
  }
}
```

Будьте внимательны при использовании значений защищенных параметров во вложенном шаблоне. Если задать для области значение OUTER, защищенные значения будут храниться в виде обычного текста в журнале развертывания. Пользователь, просматривающий шаблон в журнале развертывания, может увидеть защищенные значения. Вместо этого используйте внутреннюю область или добавьте к родительскому шаблону ресурсы, которым требуются безопасные значения.

В следующем фрагменте показано, какие значения являются безопасными и не являются безопасными.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Username for the Virtual Machine."
      }
    },
    "adminPasswordOrKey": {
      "type": "securestring",
      "metadata": {
        "description": "SSH Key or password for the Virtual Machine. SSH key is recommended."
      }
    }
  },
  ...
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2020-06-01",
      "name": "mainTemplate",
      "properties": {
        ...
        "osProfile": {
          "computerName": "mainTemplate",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPasswordOrKey')]" // Yes, secure because resource is in parent template
        }
      }
    },
    {
      "name": "outer",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "properties": {
        "expressionEvaluationOptions": {
          "scope": "outer"
        },
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.Compute/virtualMachines",
              "apiVersion": "2020-06-01",
              "name": "outer",
              "properties": {
                ...
                "osProfile": {
                  "computerName": "outer",
                  "adminUsername": "[parameters('adminUsername')]",
                  "adminPassword": "[parameters('adminPasswordOrKey')]" // No, not secure because resource is in nested template with outer scope
                }
              }
            }
          ]
        }
      }
    },
    {
      "name": "inner",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "properties": {
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "mode": "Incremental",
        "parameters": {
          "adminPasswordOrKey": {
              "value": "[parameters('adminPasswordOrKey')]"
          },
          "adminUsername": {
              "value": "[parameters('adminUsername')]"
          }
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "adminUsername": {
              "type": "string",
              "metadata": {
                "description": "Username for the Virtual Machine."
              }
            },
            "adminPasswordOrKey": {
              "type": "securestring",
              "metadata": {
                "description": "SSH Key or password for the Virtual Machine. SSH key is recommended."
              }
            }
          },
          "resources": [
            {
              "type": "Microsoft.Compute/virtualMachines",
              "apiVersion": "2020-06-01",
              "name": "inner",
              "properties": {
                ...
                "osProfile": {
                  "computerName": "inner",
                  "adminUsername": "[parameters('adminUsername')]",
                  "adminPassword": "[parameters('adminPasswordOrKey')]" // Yes, secure because resource is in nested template and scope is inner
                }
              }
            }
          ]
        }
      }
    }
  ]
}
```

## <a name="linked-template"></a>Связанный шаблон

Чтобы связать шаблон, добавьте [ресурс развертывания](/azure/templates/microsoft.resources/deployments) в основной шаблон. В `templateLink` свойстве укажите универсальный код ресурса (URI) шаблона для включения. В следующем примере выполняется ссылка на шаблон, который находится в учетной записи хранения.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "linkedTemplate",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri":"https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
          "contentVersion":"1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

При ссылке на связанный шаблон значение `uri` не может быть локальным файлом или файлом, доступным только в локальной сети. Azure Resource Manager должен иметь доступ к шаблону. Укажите значение URI, которое скачивается как HTTP или HTTPS.

Вы можете ссылаться на шаблоны с помощью параметров, включающих HTTP или HTTPS. Например, распространенным шаблоном является использование `_artifactsLocation` параметра. Связанный шаблон можно задать с помощью выражения, например:

```json
"uri": "[concat(parameters('_artifactsLocation'), '/shared/os-disk-parts-md.json', parameters('_artifactsLocationSasToken'))]"
```

Если вы создаете ссылку на шаблон в GitHub, используйте необработанный URL-адрес. Ссылка имеет формат: `https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-with-templates/quickstart-template/azuredeploy.json` . Чтобы получить необработанную ссылку, выберите **необработанный**.

:::image type="content" source="./media/linked-templates/select-raw.png" alt-text="Выберите необработанный URL-адрес":::

### <a name="parameters-for-linked-template"></a>Параметры для связанного шаблона

Параметры для связанного шаблона можно указать во внешнем файле или в встроенном виде. При предоставлении файла внешнего параметра используйте `parametersLink` свойство:

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2019-10-01",
    "name": "linkedTemplate",
    "properties": {
      "mode": "Incremental",
      "templateLink": {
        "uri": "https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.parameters.json",
        "contentVersion": "1.0.0.0"
      }
    }
  }
]
```

Чтобы передать значения параметров в строке, используйте `parameters` свойство.

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2019-10-01",
    "name": "linkedTemplate",
    "properties": {
      "mode": "Incremental",
      "templateLink": {
        "uri": "https://mystorageaccount.blob.core.windows.net/AzureTemplates/newStorageAccount.json",
        "contentVersion": "1.0.0.0"
      },
      "parameters": {
        "storageAccountName": {
          "value": "[parameters('storageAccountName')]"
        }
      }
    }
  }
]
```

Нельзя использовать одновременно встроенные параметры и ссылку на их файл. Развертывание завершается сбоем с ошибкой когда указываются `parametersLink` и `parameters`.

### <a name="use-relative-path-for-linked-templates"></a>Использовать относительный путь для связанных шаблонов

`relativePath`Свойство класса `Microsoft.Resources/deployments` упрощает создание связанных шаблонов. Это свойство можно использовать для развертывания удаленного связанного шаблона в расположении относительно родительского элемента. Эта функция требует, чтобы все файлы шаблонов были промежуточными и доступны на удаленном URI, таком как GitHub или учетная запись хранения Azure. При вызове основного шаблона с помощью URI из Azure PowerShell или Azure CLI URI дочернего развертывания является сочетанием родительского и relativePath.

> [!NOTE]
> При создании Темплатеспек все шаблоны, на которые ссылается `relativePath` свойство, упаковываются в ресурс темплатеспек с Azure PowerShell или Azure CLI. Файлы для промежуточного размещения не требуются. Дополнительные сведения см. в статье [Создание шаблона спецификации со связанными шаблонами](./template-specs.md#create-a-template-spec-with-linked-templates).

Предположим, что такая структура папок выглядит следующим образом:

![относительный путь связанного шаблона диспетчера ресурсов](./media/linked-templates/resource-manager-linked-templates-relative-path.png)

В следующем шаблоне показано, как *mainTemplate.jsдля* развертываний *nestedChild.js* , как показано на предыдущем рисунке.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "functions": [],
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-10-01",
      "name": "childLinked",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "children/nestedChild.json"
        }
      }
    }
  ],
  "outputs": {}
}
```

В следующем развертывании URI связанного шаблона в предыдущем шаблоне имеет значение **https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/linked-template-relpath/children/nestedChild.json** .

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name linkedTemplateWithRelativePath `
  -ResourceGroupName "myResourceGroup" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/linked-template-relpath/mainTemplate.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name linkedTemplateWithRelativePath \
  --resource-group myResourceGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/linked-template-relpath/mainTemplate.json"
```

---

Чтобы развернуть связанные шаблоны с относительным путем, хранящимся в учетной записи хранения Azure, используйте `QueryString` / `query-string` параметр, чтобы указать маркер SAS, который будет использоваться с параметром TemplateUri. Этот параметр поддерживается только Azure CLI версии 2,18 или более поздней, а также Azure PowerShell версии 5,4 или более поздней.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Name linkedTemplateWithRelativePath `
  -ResourceGroupName "myResourceGroup" `
  -TemplateUri "https://stage20210126.blob.core.windows.net/template-staging/mainTemplate.json" `
  -QueryString $sasToken
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --name linkedTemplateWithRelativePath \
  --resource-group myResourceGroup \
  --template-uri "https://stage20210126.blob.core.windows.net/template-staging/mainTemplate.json" \
  --query-string $sasToken
```

---

Убедитесь, что в строке запроса отсутствует начальный знак "?". Развертывание добавляет его при сборке URI для развертываний.

## <a name="template-specs"></a>Спецификации шаблонов

Вместо поддержки связанных шаблонов на доступной конечной точке можно создать [шаблонную спецификацию](template-specs.md) , которая упаковывает основной шаблон и связанные с ним шаблоны в одну сущность, которую можно развернуть. Спецификация шаблона — это ресурс в подписке Azure. Это позволяет легко обеспечить безопасный общий доступ к шаблону для пользователей в вашей организации. Используйте управление доступом на основе ролей Azure (Azure RBAC), чтобы предоставить доступ к спецификации шаблона. Сейчас эта функция доступна в предварительной версии.

Дополнительные сведения см. в разделе:

- [Учебник. Создание спецификации шаблона со связанными шаблонами](./template-specs-create-linked.md).
- [Учебник. Развертывание спецификации шаблона в качестве связанного шаблона](./template-specs-deploy-linked-template.md).

## <a name="dependencies"></a>Зависимости

Как и в случае с другими типами ресурсов, можно задать зависимости между связанными шаблонами. Если ресурсы в одном связанном шаблоне должны быть развернуты до ресурсов во втором связанном шаблоне, установите второй шаблон, зависящий от первого.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/linkedtemplates/linked-dependency.json" highlight="10,22,24":::

## <a name="contentversion"></a>contentVersion

Вам не нужно указывать `contentVersion` свойство для `templateLink` `parametersLink` свойства или. Если не указать `contentVersion` , то будет развернута Текущая версия шаблона. Если предоставить значение для версии содержимого, оно должно совпадать с версией связанного шаблона, в противном случае развертывание завершится ошибкой.

## <a name="using-variables-to-link-templates"></a>Использование переменных для связывания шаблонов

В предыдущих примерах были показаны жестко запрограммированные значения URL-адреса для ссылок на шаблоны. Этот подход может работать для простого шаблона, но он не подходит для большого набора модульных шаблонов. Вместо этого можно создать статическую переменную, которая содержит базовый URL-адрес для основного шаблона, а затем динамически создавать URL-адреса для связанных шаблонов на основе этого адреса. Преимущество такого подхода заключается в том, что вы можете легко перемещать или разветвление шаблона, так как необходимо изменить только статическую переменную в основном шаблоне. Основной шаблон передает правильные URI через разделенный шаблон.

В следующем примере показано, как использовать базовый URL-адрес для создания двух URL-адресов для связанных шаблонов ( `sharedTemplateUrl` и `vmTemplateUrl` ).

```json
"variables": {
  "templateBaseUrl": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/postgresql-on-ubuntu/",
  "sharedTemplateUrl": "[uri(variables('templateBaseUrl'), 'shared-resources.json')]",
  "vmTemplateUrl": "[uri(variables('templateBaseUrl'), 'database-2disk-resources.json')]"
}
```

Вы также можете получить базовый URL-адрес текущего шаблона с помощью функции [deployment()](template-functions-deployment.md#deployment) , а затем использовать его для получения URL-адресов других шаблонов в том же расположении. Это полезно, если расположение шаблонов меняется или если вы не хотите указывать URL-адреса непосредственно в файле шаблона. `templateLink`Свойство возвращается только при связывании с удаленным шаблоном с URL-адресом. Если вы используете локальный шаблон, свойство недоступно.

```json
"variables": {
  "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"
}
```

В конечном счете, вы бы использовали переменную в `uri` свойстве `templateLink` Свойства.

```json
"templateLink": {
 "uri": "[variables('sharedTemplateUrl')]",
 "contentVersion":"1.0.0.0"
}
```

## <a name="using-copy"></a>Использование инструкции Copy

Чтобы создать несколько экземпляров ресурса с вложенным шаблоном, добавьте `copy` элемент на уровне `Microsoft.Resources/deployments` ресурса. Или, если область имеет значение `inner` , можно добавить копию внутри вложенного шаблона.

В следующем примере шаблона показано, как использовать `copy` с вложенным шаблоном.

```json
"resources": [
  {
    "type": "Microsoft.Resources/deployments",
    "apiVersion": "2019-10-01",
    "name": "[concat('nestedTemplate', copyIndex())]",
    // yes, copy works here
    "copy": {
      "name": "storagecopy",
      "count": 2
    },
    "properties": {
      "mode": "Incremental",
      "expressionEvaluationOptions": {
        "scope": "inner"
      },
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[concat(variables('storageName'), copyIndex())]",
            "location": "West US",
            "sku": {
              "name": "Standard_LRS"
            },
            "kind": "StorageV2"
            // Copy works here when scope is inner
            // But, when scope is default or outer, you get an error
            // "copy": {
            //   "name": "storagecopy",
            //   "count": 2
            // }
          }
        ]
      }
    }
  }
]
```

## <a name="get-values-from-linked-template"></a>Получение значений из связанного шаблона

Чтобы получить выходные значения из связанного шаблона, извлеките значение свойства с синтаксисом, например: `"[reference('deploymentName').outputs.propertyName.value]"`.

При получении выходного свойства из связанного шаблона имя свойства не должно включать тире.

В следующих примерах показано, как ссылаться на связанный шаблон и извлекать выходные значения. Связанный шаблон возвращает простое сообщение. Сначала связанный шаблон:

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/linkedtemplates/helloworld.json":::

Основной шаблон развертывает связанный шаблон и возвращает значение. Обратите внимание, что он ссылается на ресурс развертывания по имени и использует имя свойства, возвращенное связанным шаблоном.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/linkedtemplates/helloworldparent.json" highlight="10,23":::

В следующем примере показан шаблон, который развертывает общедоступный IP-адрес и возвращает идентификатор ресурса Azure для этого общедоступного IP-адреса:

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/linkedtemplates/public-ip.json" highlight="27":::

Чтобы использовать общедоступный IP-адрес из предыдущего шаблона при развертывании балансировщика нагрузки, свяжите его с шаблоном и объявите зависимость от `Microsoft.Resources/deployments` ресурса. Для общедоступного IP-адреса в подсистеме балансировки нагрузки задается выходное значение из связанного шаблона.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json" highlight="28,41":::

## <a name="deployment-history"></a>Журнал развертывания

Resource Manager обрабатывает каждый шаблон как отдельное развертывание в журнале развертывания. Основной шаблон с тремя связанными или вложенными шаблонами отображается в журнале развертывания как:

![Журнал развертывания](./media/linked-templates/deployment-history.png)

С помощью этих отдельных записей в журнале можно извлечь выходные значения после развертывания. Следующий шаблон создает общедоступный IP-адрес и выводит IP-адрес:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "publicIPAddresses_name": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2018-11-01",
      "name": "[parameters('publicIPAddresses_name')]",
      "location": "southcentralus",
      "properties": {
        "publicIPAddressVersion": "IPv4",
        "publicIPAllocationMethod": "Static",
        "idleTimeoutInMinutes": 4,
        "dnsSettings": {
          "domainNameLabel": "[concat(parameters('publicIPAddresses_name'), uniqueString(resourceGroup().id))]"
        }
      },
      "dependsOn": []
    }
  ],
  "outputs": {
    "returnedIPAddress": {
      "type": "string",
      "value": "[reference(parameters('publicIPAddresses_name')).ipAddress]"
    }
  }
}
```

Указанный ниже шаблон ссылается на предыдущий шаблон. Он создает три общедоступных IP-адреса.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "[concat('linkedTemplate', copyIndex())]",
      "copy": {
        "count": 3,
        "name": "ip-loop"
      },
      "properties": {
        "mode": "Incremental",
        "templateLink": {
        "uri": "[uri(deployment().properties.templateLink.uri, 'static-public-ip.json')]",
        "contentVersion": "1.0.0.0"
        },
        "parameters":{
          "publicIPAddresses_name":{"value": "[concat('myip-', copyIndex())]"}
        }
      }
    }
  ]
}
```

После развертывания можно извлечь выходные значения, выполнив следующий сценарий PowerShell.

```azurepowershell-interactive
$loopCount = 3
for ($i = 0; $i -lt $loopCount; $i++)
{
  $name = 'linkedTemplate' + $i;
  $deployment = Get-AzResourceGroupDeployment -ResourceGroupName examplegroup -Name $name
  Write-Output "deployment $($deployment.DeploymentName) returned $($deployment.Outputs.returnedIPAddress.value)"
}
```

Или сценарий Azure CLI в оболочке Bash:

```azurecli-interactive
#!/bin/bash

for i in 0 1 2;
do
  name="linkedTemplate$i";
  deployment=$(az deployment group show -g examplegroup -n $name);
  ip=$(echo $deployment | jq .properties.outputs.returnedIPAddress.value);
  echo "deployment $name returned $ip";
done
```

## <a name="securing-an-external-template"></a>Защита внешнего шаблона

Хотя связанный шаблон должен быть доступен извне, он не должен быть общедоступным. Шаблон можно добавить к личной учетной записи хранения, доступ к которой есть только у владельца учетной записи хранения. Затем создайте маркер подписанного URL-адреса для использования при развертывании. Этот маркер SAS добавляется в универсальный код ресурса (URI) связанного шаблона. Несмотря на то, что маркер передается в защищенной строке, универсальный код ресурса (URI) связанного шаблона, включающий в себя маркер SAS, добавляется в журнал операций развертывания. Чтобы снизить риск раскрытия, задайте срок действия маркера.

Доступ к файлу параметров также можно ограничить с помощью маркера SAS.

Сейчас невозможно создать ссылку на шаблон в учетной записи хранения, которая находится за [брандмауэром службы хранилища Azure](../../storage/common/storage-network-security.md).

> [!IMPORTANT]
> Вместо защиты связанного шаблона с помощью маркера SAS можно создать [спецификацию шаблона](template-specs.md). Спецификация шаблона безопасно сохраняет основной шаблон и связанные с ним шаблоны в качестве ресурса в подписке Azure. Для предоставления доступа пользователям, которым требуется развернуть шаблон, используется Azure RBAC.

В следующем примере показано, как передать маркер SAS при создании связи с шаблоном.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerSasToken": { "type": "securestring" }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-10-01",
      "name": "linkedTemplate",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "uri": "[concat(uri(deployment().properties.templateLink.uri, 'helloworld.json'), parameters('containerSasToken'))]",
          "contentVersion": "1.0.0.0"
        }
      }
    }
  ],
  "outputs": {
  }
}
```

В PowerShell получение маркера для контейнера и развертывание шаблонов выполняется с помощью следующих команд. Обратите внимание, что `containerSasToken` параметр определен в шаблоне. Он не является параметром в `New-AzResourceGroupDeployment` команде.

```azurepowershell-interactive
Set-AzCurrentStorageAccount -ResourceGroupName ManageGroup -Name storagecontosotemplates
$token = New-AzStorageContainerSASToken -Name templates -Permission r -ExpiryTime (Get-Date).AddMinutes(30.0)
$url = (Get-AzStorageBlob -Container templates -Blob parent.json).ICloudBlob.uri.AbsoluteUri
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateUri ($url + $token) -containerSasToken $token
```

Для Azure CLI в оболочке Bash происходит получение маркера для контейнера и развертывание шаблонов с помощью следующего кода.

```azurecli-interactive
#!/bin/bash

expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
  --resource-group ManageGroup \
  --name storagecontosotemplates \
  --query connectionString)
token=$(az storage container generate-sas \
  --name templates \
  --expiry $expiretime \
  --permissions r \
  --output tsv \
  --connection-string $connection)
url=$(az storage blob url \
  --container-name templates \
  --name parent.json \
  --output tsv \
  --connection-string $connection)
parameter='{"containerSasToken":{"value":"?'$token'"}}'
az deployment group create --resource-group ExampleGroup --template-uri $url?$token --parameters $parameter
```

## <a name="example-templates"></a>Образцы шаблонов

В следующих примерах показаны наиболее частые способы использования связанных шаблонов.

|Основной шаблон  |Связанный шаблон |Описание  |
|---------|---------| ---------|
|[Hello World](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworldparent.json) |[связанный шаблон](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/helloworld.json) | Возвращает строку из связанного шаблона. |
|[Подсистема балансировки нагрузки с общедоступным IP-адресом](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip-parentloadbalancer.json) |[связанный шаблон](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/public-ip.json) |Возвращает общедоступный IP-адрес из связанного шаблона и задает это значение в подсистеме балансировки нагрузки. |
|[Несколько IP-адресов](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip-parent.json) | [связанный шаблон](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/linkedtemplates/static-public-ip.json) |Создает несколько общедоступных IP-адресов в связанном шаблоне.  |

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы пройти обучение, см. раздел [учебник. Развертывание связанного шаблона](./deployment-tutorial-linked-template.md).
* Дополнительные сведения о том, как определить порядок развертывания ресурсов, см. в разделе [Определение заказа для развертывания ресурсов в шаблонах ARM](define-resource-dependency.md).
* Чтобы узнать, как определить один ресурс, но создать несколько его экземпляров, см. статью [Итерация ресурсов в шаблонах ARM](copy-resources.md).
* Инструкции по настройке шаблона в учетной записи хранения и формированию маркера SAS см. в статьях [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](deploy-powershell.md) или [развертывание ресурсов с помощью шаблонов ARM и Azure CLI](deploy-cli.md).
