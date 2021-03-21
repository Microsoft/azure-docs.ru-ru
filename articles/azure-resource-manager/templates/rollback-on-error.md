---
title: Откат к ошибке при успешном развертывании
description: Укажите, что неудачное развертывание должно выполнить откат к успешному развертыванию.
ms.topic: conceptual
ms.date: 02/02/2021
ms.openlocfilehash: 742a8f16a2dce3204b48085759091540586a4522
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99492218"
---
# <a name="rollback-on-error-to-successful-deployment"></a>Откат к ошибке при успешном развертывании

Если развертывание завершается ошибкой, вы можете автоматически повторно развернуть ресурс, успешно развернутый ранее, из журнала развертывания. Эта функция полезна, если вы получили известное хорошее состояние развертывания инфраструктуры и хотите вернуться к этому состоянию. Можно указать определенное более раннее развертывание или Последнее успешное развертывание.

> [!IMPORTANT]
> Эта функция выполняет откат неудачного развертывания, повторно развертывая предыдущее развертывание. Результат может отличаться от того, который вы хотели бы отменить при неудачном развертывании. Убедитесь, что вы понимаете, как развертывается предыдущее развертывание.

## <a name="considerations-for-redeploying"></a>Рекомендации по повторному развертыванию

Перед использованием этой функции рассмотрите следующие сведения об обработке повторного развертывания.

- Предыдущее развертывание выполняется с помощью [полного режима](./deployment-modes.md#complete-mode), даже если во время предыдущего развертывания использовался [добавочный режим](./deployment-modes.md#incremental-mode) . Повторное развертывание в полном режиме может привести к непредвиденным результатам, если предыдущее развертывание использовалось добавочно. Полный режим означает, что все ресурсы, не входящие в предыдущее развертывание, будут удалены. Укажите более раннее развертывание, представляющее все ресурсы и их состояния, которые должны существовать в группе ресурсов. Дополнительные сведения см. в разделе [режимы развертывания](./deployment-modes.md).
- Повторное развертывание выполняется точно так же, как было запущено ранее с теми же параметрами. Изменить параметры невозможно.
- Повторное развертывание влияет только на ресурсы, любые изменения данных не затрагиваются.
- Эту функцию можно использовать только при развертывании группы ресурсов. Он не поддерживает развертывание на уровне подписки, группы управления или клиента. Дополнительные сведения о развертывании уровня подписки см. в статье [Создание групп ресурсов и ресурсов на уровне подписки](./deploy-to-subscription.md).
- Этот параметр можно использовать только с развертываниями корневого уровня. Повторное развертывание из вложенных шаблонов не поддерживается.

Чтобы использовать этот параметр, в журнале развертывания должны быть уникальные имена развертываний. Это только уникальные имена, которые может идентифицировать конкретное развертывание. Если у вас нет уникальных имен, неудачное развертывание может перезаписать успешное развертывание в журнале.

Если указано более раннее развертывание, которое не существует в журнале развертывания, то откат возвращает ошибку.

## <a name="powershell"></a>PowerShell

Чтобы выполнить повторно последнее успешное развертывание, добавьте параметр `-RollbackToLastDeployment` в качестве флага.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollbackToLastDeployment
```

Чтобы выполнить повторно конкретное развертывание, используйте параметр `-RollBackDeploymentName` и укажите имя развертывания. Указанное развертывание должно быть успешным.

```azurepowershell-interactive
New-AzResourceGroupDeployment -Name ExampleDeployment02 `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -RollBackDeploymentName ExampleDeployment01
```

## <a name="azure-cli"></a>Azure CLI

Чтобы выполнить повторно последнее успешное развертывание, добавьте параметр `--rollback-on-error` в качестве флага.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error
```

Чтобы выполнить повторно конкретное развертывание, используйте параметр `--rollback-on-error` и укажите имя развертывания. Указанное развертывание должно быть успешным.

```azurecli-interactive
az deployment group create \
  --name ExampleDeployment02 \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error ExampleDeployment01
```

## <a name="rest-api"></a>REST API

Чтобы повторно выполнить последнее успешное развертывание, если текущее развертывание завершается сбоем, используйте следующий код:

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "LastSuccessful",
    }
  }
}
```

Чтобы повторно выполнить определенное развертывание, если текущее развертывание завершается сбоем, используйте следующий код:

```json
{
  "properties": {
    "templateLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
      "contentVersion": "1.0.0.0"
    },
    "mode": "Incremental",
    "parametersLink": {
      "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
      "contentVersion": "1.0.0.0"
    },
    "onErrorDeployment": {
      "type": "SpecificDeployment",
      "deploymentName": "<deploymentname>"
    }
  }
}
```

Указанное развертывание должно быть успешным.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о полном и инкрементном режимах см. в разделе [Azure Resource Manager режимы развертывания](deployment-modes.md).
- Сведения об определении параметров в шаблоне см. в статье [Описание структуры и синтаксиса шаблонов Azure Resource Manager](template-syntax.md).
