---
title: Перемещение регионов для ресурсов в Microsoft. Resources
description: Показывает, как перемещать ресурсы, которые находятся в пространстве имен Microsoft. Resources, в новые регионы.
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 098e5c36969da12966d793b6e1eddd4f79701ead
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88951058"
---
# <a name="move-microsoftresources-resources-to-new-region"></a>Перемещение ресурсов Microsoft. Resources в новый регион

Может потребоваться переместить существующий ресурс в новый регион. В этой статье показано, как переместить два типа ресурсов — Темплатеспекс и deploymentScripts, которые находятся в пространстве имен Microsoft. Resources.

## <a name="move-template-specs-to-new-region"></a>Переместить спецификации шаблона в новый регион

Если у вас есть [Спецификация шаблона](../templates/template-specs.md) в одном регионе и вы хотите переместить ее в новый регион, можно экспортировать спецификацию шаблона и повторно развернуть ее.

1. Используйте команду, чтобы экспортировать существующую спецификацию шаблона. Для значений параметров укажите значения, соответствующие спецификации шаблона, который необходимо экспортировать.

   Для Azure PowerShell:

   ```azurepowershell
   Export-AzTemplateSpec `
     -ResourceGroupName demoRG `
     -Name demoTemplateSpec `
     -Version 1.0 `
     -OutputFolder c:\export
   ```

   Для интерфейса командной строки Azure:

   ```azurecli
   az template-specs export \
     --resource-group demoRG \
     --name demoTemplateSpec \
     --version 1.0 \
     --output-folder c:\export
   ```

1. Используйте экспортированную спецификацию шаблона, чтобы создать новую спецификацию шаблона. В следующих примерах показан `westus` новый регион, но можно указать нужный регион.

   Для Azure PowerShell:

   ```azurepowershell
   New-AzTemplateSpec `
     -Name movedTemplateSpec `
     -Version 1.0 `
     -ResourceGroupName newRG `
     -Location westus `
     -TemplateJsonFile c:\export\1.0.json
   ```

   Для интерфейса командной строки Azure:

   ```azurecli
   az template-specs create \
     --name movedTemplateSpec \
     --version "1.0" \
     --resource-group newRG \
     --location "westus" \
     --template-file "c:\export\demoTemplateSpec.json"
   ```

## <a name="move-deployment-scripts-to-new-region"></a>Переместить скрипты развертывания в новый регион

1. Выберите группу ресурсов, содержащую скрипт развертывания, который необходимо переместить в новый регион.

1. [Экспортируйте шаблон](../templates/export-template-portal.md). При экспорте выберите скрипт развертывания и все необходимые ресурсы.

1. В экспортированном шаблоне удалите следующие свойства:

   * tenantId
   * principalId
   * clientid

1. Экспортированный шаблон имеет жестко заданное значение для региона скрипта развертывания.

   ```json
   "location": "westus2",
   ```

   Измените шаблон, чтобы разрешить параметр для задания расположения. Дополнительные сведения см. [в разделе Задание расположения ресурса в шаблоне ARM](../templates/resource-location.md) .

   ```json
   "location": "[parameters('location')]",
   ```

1. [Разверните экспортированный шаблон](../templates/deploy-powershell.md) и укажите новый регион для скрипта развертывания.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о перемещении ресурсов в новую группу ресурсов или подписку см. в статье [Перемещение ресурсов в новую группу ресурсов или](move-resource-group-and-subscription.md)подписку.
* Дополнительные сведения о перемещении ресурсов в новый регион см. в статье [Перемещение ресурсов Azure в разных регионах](move-region.md).
