---
title: Использовать Key Vault при развертывании управляемого приложения
description: Демонстрация использования секретов доступа в Azure Key Vault при развертывании управляемых приложений
author: tfitzmac
ms.topic: conceptual
ms.date: 01/30/2019
ms.author: tomfitz
ms.openlocfilehash: 5dc219747b9cc74b6c6aac8ab190ebfbe5ae9b32
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81458289"
---
# <a name="access-key-vault-secret-when-deploying-azure-managed-applications"></a>Доступ к секрету Key Vault при развертывании Управляемых приложений Azure

Если необходимо передать защищенное значение (например, пароль) в качестве параметра во время развертывания, можно извлечь его из [Azure Key Vault](../../key-vault/general/overview.md). Чтобы получить доступ к Key Vault при развертывании управляемых приложений, нужно предоставить доступ к субъекту-службе **Поставщик ресурсов устройств**. Служба управляемых приложений использует это удостоверение для выполнения операций. Для успешного извлечения значения из Key Vault во время развертывания у субъекта-службы должен быть доступ к Key Vault.

Эта статья описывает настройку Key Vault для работы с управляемыми приложениями.

## <a name="enable-template-deployment"></a>Включение развертывания шаблона

1. На портале выберите Key Vault.

1. Выберите **Политики доступа**.   

   ![Выбор политик доступа](./media/key-vault-access/select-access-policies.png)

1. Выберите **Щелкните, чтобы показать политики расширенного доступа**.

   ![Отображение политик расширенного доступа](./media/key-vault-access/advanced.png)

1. Выберите **Включить доступ к Azure Resource Manager для развертывания шаблонов**. Затем нажмите кнопку **Сохранить**.

   ![Включение развертывания шаблона](./media/key-vault-access/enable-template.png)

## <a name="add-service-as-contributor"></a>Добавление службы в качестве участника

1. Выберите **Управление доступом (IAM)** .

   ![Выбор управления доступом](./media/key-vault-access/access-control.png)

1. Выберите **Добавить назначение ролей**.

   ![Выбор элемента "Добавить"](./media/key-vault-access/add-access-control.png)

1. Выберите **Участник** для роли. Найдите **Поставщик ресурсов устройств** и выберите его в списке доступных вариантов.

   ![Поиск поставщика](./media/key-vault-access/search-provider.png)

1. Щелкните **Сохранить**.

## <a name="reference-key-vault-secret"></a>Ссылка на секрет Key Vault

Чтобы передать секрет из Key Vault в шаблон в управляемом приложении, необходимо использовать [связанный или вложенный шаблон](../templates/linked-templates.md) и ссылаться на Key Vault в параметрах для связанного или вложенного шаблона. Укажите идентификатор ресурса Key Vault и имя секрета.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
      "apiVersion": "2018-05-01",
      "name": "dynamicSecret",
      "properties": {
        "mode": "Incremental",
        "expressionEvaluationOptions": {
          "scope": "inner"
        },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
        }
      }
    }
  ],
  "outputs": {
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Вы настроили доступ к Key Vault во время развертывания управляемого приложения.

* Сведения о передаче значения из Key Vault в виде параметра шаблона см. в статье [Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания](../templates/key-vault-parameter.md).
* См. дополнительные сведения о [примерах проектов для управляемых приложений Azure](sample-projects.md).
* Чтобы узнать, как создать файл определения пользовательского интерфейса для управляемого приложения, изучите статью [Начало работы с CreateUiDefinition](create-uidefinition-overview.md).
