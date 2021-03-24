---
title: Развертывание ресурсов с помощью REST API и шаблона
description: Сведения о том, как применить Azure Resource Manager и REST API Resource Manager для развертывания ресурсов в Azure. Эти ресурсы определяются в шаблоне Resource Manager.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: e688d7abfaca442c3de395d25961b4e81e6c7b24
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889201"
---
# <a name="deploy-resources-with-arm-templates-and-azure-resource-manager-rest-api"></a>Развертывание ресурсов с помощью шаблонов ARM и Azure Resource Manager REST API

В этой статье объясняется, как использовать Azure Resource Manager REST API с шаблонами Azure Resource Manager (шаблоны ARM) для развертывания ресурсов в Azure.

Вы можете включить шаблон в текст запроса или связать с файлом. Файл может быть локальным или внешним, доступным по универсальному коду ресурса (URI). Если шаблон находится в учетной записи хранения, то во время развертывания можно ограничить доступ к шаблону и предоставить маркер подписанного URL-адреса (SAS).

## <a name="deployment-scope"></a>Область развертывания

Вы можете выбрать для развертывания группу ресурсов, подписку Azure, группу управления или клиент. В зависимости от области развертывания используются разные команды.

- Чтобы выполнить развертывание в **группе ресурсов**, используйте инструкции из статьи [Развертывания: создание](/rest/api/resources/deployments/createorupdate). Назначение для такого запроса:

  ```HTTP
  PUT https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-10-01
  ```

- Чтобы выполнить развертывание в **подписке**, используйте инструкции из статьи [Развертывания: создание в области подписки](/rest/api/resources/deployments/createorupdateatsubscriptionscope). Назначение для такого запроса:

  ```HTTP
  PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-10-01
  ```

  Дополнительные сведения о развертываниях на уровне подписки см. в статье [Создание групп ресурсов и ресурсов на уровне подписки](deploy-to-subscription.md).

- Чтобы выполнить развертывание в **группе управления**, используйте инструкции из статьи [Развертывания: создание в области группы управления](/rest/api/resources/deployments/createorupdateatmanagementgroupscope). Назначение для такого запроса:

  ```HTTP
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{groupId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-10-01
  ```

  Дополнительные сведения о развертываниях на уровне групп управления см. в статье [Создание ресурсов на уровне группы управления](deploy-to-management-group.md).

- Чтобы выполнить развертывание в **клиенте**, используйте инструкции из статьи [Развертывания: создание или обновление в области клиента](/rest/api/resources/deployments/createorupdateattenantscope). Назначение для такого запроса:

  ```HTTP
  PUT https://management.azure.com/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-10-01
  ```

  Дополнительные сведения о развертываниях на уровне клиента см. в статье [Создание ресурсов на уровне клиента](deploy-to-tenant.md).

В примерах, приведенных в этой статье, мы используем развертывание в группе ресурсов.

## <a name="deploy-with-the-rest-api"></a>Развертывание с помощью REST API

1. Задайте [общие параметры и заголовки](/rest/api/azure/), включая маркеры аутентификации.

1. Если развертывание выполняется в несуществующей группе ресурсов, создайте группу ресурсов. Укажите идентификатор подписки, имя новой группы ресурсов и расположение, необходимое для решения. Дополнительную информацию см. в разделе [Создание группы ресурсов](/rest/api/resources/resourcegroups/createorupdate).

   ```HTTP
   PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2020-06-01
   ```

   С текстом запроса:

   ```json
   {
    "location": "West US",
    "tags": {
      "tagname1": "tagvalue1"
    }
   }
   ```

1. Перед развертыванием шаблона можно просмотреть изменения, которые шаблон будет вносить в вашу среду. Используйте [операцию "что если](template-deploy-what-if.md) ", чтобы убедиться, что шаблон вносит необходимые изменения. Что если также проверяет шаблон на наличие ошибок.

1. Чтобы развернуть шаблон, укажите идентификатор подписки, имя группы ресурсов, имя развертывания и URI запроса.

   ```HTTP
   PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2020-10-01
   ```

   В тексте запроса укажите ссылку на шаблон и файл параметров. Дополнительные сведения о файле параметров см. в статье [Создание файла параметров Resource Manager](parameter-files.md).

   Обратите внимание, что `mode` задано **добавочное**. Чтобы выполнить полное развертывание, установите для параметра значение `mode` **завершено**. Будьте внимательны при использовании полного режима, так как вы можете случайно удалить ресурсы, которые находятся не в шаблоне.

   ```json
   {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental"
    }
   }
   ```

    Если требуется регистрировать содержимое ответа, запросить содержимое или и то, и другое, включите `debugSetting` в запрос.

   ```json
   {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental",
      "debugSetting": {
        "detailLevel": "requestContent, responseContent"
      }
    }
   }
   ```

    Можно настроить учетную запись хранения для использования маркера подписанного URL-адреса (SAS). Дополнительные сведения см. [в разделе Делегирование доступа с помощью подписанного URL-доступа](/rest/api/storageservices/delegate-access-with-shared-access-signature).

    Если требуется предоставить конфиденциальное значение для параметра (например, пароль), добавьте это значение в хранилище ключей. Получите хранилище ключей во время развертывания, как показано в предыдущем примере. Дополнительные сведения см. в статье [Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания](key-vault-parameter.md).

1. Вместо создания ссылки на файлы для шаблона и параметров их можно включить в тексте запроса. В следующем примере приведен текст запроса, в котором напрямую указаны шаблон и параметр.

   ```json
   {
      "properties": {
      "mode": "Incremental",
      "template": {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
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
              "description": "Storage Account type"
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          }
        },
        "variables": {
          "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
        },
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2018-02-01",
            "name": "[variables('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
              "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "properties": {}
          }
        ],
        "outputs": {
          "storageAccountName": {
            "type": "string",
            "value": "[variables('storageAccountName')]"
          }
        }
      },
      "parameters": {
        "location": {
          "value": "eastus2"
        }
      }
    }
   }
   ```

1. Чтобы получить состояние развертывания шаблона, используйте инструкции из статьи [Развертывания: получение](/rest/api/resources/deployments/get).

   ```HTTP
   GET https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-10-01
   ```

## <a name="deployment-name"></a>Deployment name (Имя развертывания)

Можно присвоить развертыванию имя, например `ExampleDeployment` .

Каждый раз при выполнении развертывания в журнал развертывания группы ресурсов добавляется запись с именем развертывания. Если запустить другое развертывание и присвоить ему такое же имя, то Предыдущая запись будет заменена текущим развертыванием. Если требуется хранить уникальные записи в журнале развертывания, присвойте каждому развертыванию уникальное имя.

Чтобы создать уникальное имя, можно назначить случайное число. Или добавьте значение даты.

При выполнении параллельных развертываний в одной группе ресурсов с тем же именем развертывания завершается только Последнее развертывание. Все развертывания с тем же именем, которые не были завершены, заменяются последним развертыванием. Например, если вы запускаете развертывание с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage1` и в то же время запускает другое развертывание с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage2` , развертывается только одна учетная запись хранения. Результирующая учетная запись хранения называется `storage2` .

Однако при запуске развертывания с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage1` , и сразу после завершения работы вы запускаете другое развертывание с именем, `newStorage` которое развертывает учетную запись хранения с именем `storage2` , а затем у вас есть две учетные записи хранения. Одна из них называется `storage1` , а другая называется `storage2` . Но в журнале развертывания имеется только одна запись.

При указании уникального имени для каждого развертывания их можно запускать параллельно без конфликтов. Если вы запускаете развертывание с именем, `newStorage1` которое развертывает учетную запись хранения с именем `storage1` и в то же время запускает другое развертывание с именем, `newStorage2` которое развертывает учетную запись хранения с именем `storage2` , то в журнале развертывания есть две учетные записи хранения и две записи.

Чтобы избежать конфликтов с параллельными развертываниями и обеспечить уникальность записей в журнале развертывания, присвойте каждому развертыванию уникальное имя.

## <a name="next-steps"></a>Дальнейшие действия

- Если возникнет ошибка при развертывании, выполните откат по инструкциям из статьи [Откат к работоспособному развертыванию в случае ошибки](rollback-on-error.md).
- Сведения о том, как указать способ обработки ресурсов, которые существуют в группе ресурсов, но не определены в шаблоне, см. в [описании режимов развертывания с помощью Azure Resource Manager](deployment-modes.md).
- Сведения об обработке асинхронных операций REST см. в статье [Track asynchronous Azure operations](../management/async-operations.md) (Отслеживание асинхронных операций Azure).
- Дополнительные сведения о шаблонах см. в статье [Общие сведения о структуре и синтаксисе шаблонов ARM](template-syntax.md).
