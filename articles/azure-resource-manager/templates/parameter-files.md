---
title: Создание файла параметров
description: Создание файла параметров для передачи значений во время развертывания шаблона Azure Resource Manager
ms.topic: conceptual
ms.date: 09/01/2020
ms.openlocfilehash: 2b6d942b21594fa608127bb8f403e72295671005
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89276666"
---
# <a name="create-resource-manager-parameter-file"></a>Создание файла параметров Resource Manager

Вместо передачи параметров в виде встроенных значений в сценарии вам может быть проще использовать JSON-файл, содержащий значения параметров. В этой статье показано, как создать файл параметров.

## <a name="parameter-file"></a>Файл параметров

Файл параметров должен иметь указанный ниже формат.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "value": "<second-value>"
    }
  }
}
```

Обратите внимание, что значения параметров хранятся в файле параметров в виде обычного текста. Этот подход работает для неконфиденциальных значений, например при указания номера SKU для ресурса. Он не работает для конфиденциальных значений, таких как пароли. Если необходимо передать конфиденциальное значение в качестве параметра, сохраните значение в хранилище ключей и укажите ссылку на хранилище ключей в файле параметров. Конфиденциальное значение безопасно извлекается во время развертывания.

Следующий файл параметров включает в себя значение в простом тексте и значение, которое хранится в хранилище ключей.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "<first-parameter-name>": {
      "value": "<first-value>"
    },
    "<second-parameter-name>": {
      "reference": {
        "keyVault": {
          "id": "<resource-id-key-vault>"
        },
        "secretName": "<secret-name>"
      }
    }
  }
}
```

Дополнительные сведения об использовании значений из хранилища ключей см. в статье [Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания](key-vault-parameter.md).

## <a name="define-parameter-values"></a>Определение значений параметров

Чтобы выяснить, как определить значения параметров, откройте развертываемый шаблон. Взгляните на раздел параметров шаблона. В следующем примере приведены параметры из шаблона.

```json
"parameters": {
  "storagePrefix": {
    "type": "string",
    "maxLength": 11
  },
  "storageAccountType": {
    "type": "string",
    "defaultValue": "Standard_LRS",
    "allowedValues": [
    "Standard_LRS",
    "Standard_GRS",
    "Standard_ZRS",
    "Premium_LRS"
    ]
  }
}
```

Первой деталью, которую следует отметить, является имя каждого параметра. Значения в файле параметров должны совпадать с именами.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
    },
    "storageAccountType": {
    }
  }
}
```

Обратите внимание на тип параметра. Значения в файле параметров должны иметь те же типы. Для этого шаблона можно указать оба параметра как строки.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": ""
    },
    "storageAccountType": {
      "value": ""
    }
  }
}
```

Затем найдите значение по умолчанию. Если у параметра есть значение по умолчанию, можно указать значение, но это необязательно.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "" // This value must be provided.
    },
    "storageAccountType": {
      "value": "" // This value is optional. Template will use default value if not provided.
    }
  }
}
```

Наконец, просмотрите допустимые значения и любые ограничения, такие как максимальная длина. Они сообщают диапазон значений, которые можно указать для параметра.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storagePrefix": {
      "value": "storage"
    },
    "storageAccountType": {
      "value": "Standard_ZRS"
    }
  }
}
```

Файл параметров может содержать только значения параметров, определенных в шаблоне. Если файл параметров содержит дополнительные параметры, которые не соответствуют параметрам в шаблоне, возникает ошибка.

## <a name="parameter-type-formats"></a>Форматы типа параметров

В следующем примере показаны форматы различных типов параметров.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "exampleString": {
      "value": "test string"
    },
    "exampleInt": {
      "value": 4
    },
    "exampleBool": {
      "value": true
    },
    "exampleArray": {
      "value": [
        "value 1",
        "value 2"
      ]
    },
    "exampleObject": {
      "value": {
        "property1": "value1",
        "property2": "value2"
      }
    }
   }
}
```

## <a name="deploy-template-with-parameter-file"></a>Развертывание шаблона с помощью файла параметров

Чтобы передать локальный файл параметров с Azure CLI, укажите параметр @ и имя файла параметров.

```azurecli
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

Дополнительные сведения см. в статье [развертывание ресурсов с помощью шаблонов ARM и Azure CLI](./deploy-cli.md#parameters).

Чтобы передать локальный файл параметров с Azure PowerShell, используйте `TemplateParameterFile` параметр.

```azurepowershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

Дополнительные сведения см. в статье [развертывание ресурсов с помощью шаблонов ARM и Azure PowerShell](./deploy-powershell.md#pass-parameter-values)

> [!NOTE]
> Невозможно использовать файл параметров с колонкой настраиваемого шаблона на портале.

## <a name="file-name"></a>Имя файла

Общее соглашение об именовании файла параметров заключается в добавлении **.parameters** к имени шаблона. Например, если шаблон имеет имя **azuredeploy.json**, файл параметров будет называться **azuredeploy.parameters.json**. Это соглашение об именовании помогает увидеть связь между шаблоном и параметрами.

Для развертывания в разных средах создайте более одного файла параметров. При именовании файла параметров добавьте что-нибудь, указывающее на его предназначение. Например, используйте **azuredeploy.parameters-dev.json** и **azuredeploy.parameters-prod.json**

## <a name="parameter-precedence"></a>Приоритет параметров

Вы можете использовать в ходе одной операции развертывания как встроенные параметры, так и локальный файл параметров. Например, часть значений можно указать в локальном файле параметров, а другую часть — в команде развертывания. Если значения для одного параметра указаны одновременно и в локальном файле параметров, и в командной строке, более высокий приоритет имеет значение из командной строки.

Можно использовать внешний файл параметров, указав универсальный код ресурса (URI) для файла. При использовании внешнего файла параметров нельзя передавать другие значения как встроенные данные, так и из локального файла. Все встроенные параметры игнорируются. Все значения параметров следует указать во внешнем файле.

## <a name="parameter-name-conflicts"></a>Конфликты параметров

Если шаблон содержит параметр, имя которого совпадает с именем одного из параметров в команде PowerShell, параметр из шаблон отображается с постфиксом **FromTemplate**. Предположим, что параметр **ResourceGroupName** в шаблоне конфликтует с параметром **ResourceGroupName** в командлете [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment). Будет предложено указать значение для параметра **ResourceGroupNameFromTemplate**. Этой путаницы можно избежать, используя имена параметров, которые не используются для команд развертывания.


## <a name="next-steps"></a>Дальнейшие действия

- Сведения об определении параметров в шаблоне см. в статье [Параметры в шаблонах Azure Resource Manager](template-parameters.md).
- Дополнительные сведения об использовании значений из хранилища ключей см. в статье [Использование Azure Key Vault для передачи защищенного значения параметра во время развертывания](key-vault-parameter.md).
- Дополнительные сведения о параметрах см. в статье [Параметры в шаблонах Azure Resource Manager](template-parameters.md).
