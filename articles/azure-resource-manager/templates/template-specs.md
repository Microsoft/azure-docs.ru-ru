---
title: Создание и развертывание спецификации шаблона
description: Описание способов создания спецификаций шаблонов и предоставления к ним общего доступа другим пользователям в Организации.
ms.topic: conceptual
ms.date: 03/02/2021
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: 76573e4415dffb2212dd025ed486d834446d3851
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102043904"
---
# <a name="azure-resource-manager-template-specs-preview"></a>Спецификации шаблонов Azure Resource Manager (Предварительная версия)

Спецификация шаблона — это тип ресурса для хранения шаблона Azure Resource Manager (шаблон ARM) в Azure для последующего развертывания. Этот тип ресурсов позволяет предоставлять общий доступ к шаблонам ARM другим пользователям в вашей организации. Как и любой другой ресурс Azure, вы можете использовать управление доступом на основе ролей Azure (Azure RBAC) для совместного использования спецификации шаблона.

**Microsoft. Resources/темплатеспекс** — это тип ресурса для спецификаций шаблонов. Он состоит из основного шаблона и любого количества связанных шаблонов. Azure безопасно сохраняет спецификации шаблонов в группах ресурсов. Спецификации шаблонов поддерживают [Управление версиями](#versioning).

Для развертывания спецификации шаблона используются стандартные инструменты Azure, такие как PowerShell, Azure CLI, портал Azure, остальные и другие поддерживаемые пакеты SDK и клиенты. Вы используете те же команды, что и для шаблона.

> [!NOTE]
> Сейчас спецификации шаблонов доступны в предварительной версии. Чтобы использовать их с Azure PowerShell, необходимо установить [версию 5.0.0 или более новую](/powershell/azure/install-az-ps). Чтобы использовать их с Azure CLI, воспользуйтесь [версией 2.14.2 или более новой](/cli/azure/install-azure-cli).

## <a name="why-use-template-specs"></a>Зачем использовать спецификации шаблонов?

Если у вас есть шаблоны в репозитории GitHub или учетной записи хранения, при попытке совместного использования шаблонов и их применения возникают некоторые трудности. Чтобы пользователь был развернут, шаблон должен быть локальным, либо URL-адрес шаблона должен быть общедоступным. Чтобы обойти это ограничение, вы можете использовать копии шаблона совместно с пользователями, которым требуется развернуть его, или открыть доступ к репозиторию или учетной записи хранения. Когда пользователи владеют локальными копиями шаблона, эти копии могут в конечном итоге рассходится от исходного шаблона. Когда вы сделаете репозиторий или учетную запись хранения общедоступной, вы можете разрешить непреднамеренному пользователю доступ к шаблону.

Преимущество использования спецификаций шаблонов заключается в том, что вы можете создавать канонические шаблоны и совместно использовать их с командами в Организации. Спецификации шаблонов безопасны, так как они доступны для Azure Resource Manager развертывания, но недоступны пользователям без разрешения Azure RBAC. Пользователям требуется только доступ для чтения к спецификации шаблона, чтобы развернуть его шаблон, чтобы можно было предоставить общий доступ к шаблону, не позволяя другим пользователям изменять его.

Шаблоны, включаемые в спецификацию шаблона, должны быть проверены администраторами в вашей организации в соответствии с требованиями и рекомендациями Организации.

## <a name="create-template-spec"></a>Создание спецификации шаблона

В следующем примере показан простой шаблон для создания учетной записи хранения в Azure.

```json
{
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
      ]
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[concat('store', uniquestring(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "kind": "StorageV2",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      }
    }
  ]
}
```

При создании спецификации шаблона команды PowerShell или CLI передаются в основной файл шаблона. Если основной шаблон ссылается на связанные шаблоны, команды найдут их и упаковывают для создания спецификации шаблона. Дополнительные сведения см. в статье [Создание шаблона спецификации со связанными шаблонами](#create-a-template-spec-with-linked-templates).

Создайте шаблонную спецификацию с помощью:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzTemplateSpec -Name storageSpec -Version 1.0a -ResourceGroupName templateSpecsRg -Location westus2 -TemplateFile ./mainTemplate.json
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json"
```

---

Вы можете просмотреть все спецификации шаблонов в подписке с помощью:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts list
```

---

Сведения о спецификации шаблона, включая ее версии, можно просмотреть с помощью:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec -ResourceGroupName templateSpecsRG -Name storageSpec
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts show \
    --name storageSpec \
    --resource-group templateSpecRG \
    --version "1.0a"
```

---

## <a name="deploy-template-spec"></a>Развертывание спецификации шаблона

После создания спецификации шаблона пользователи с доступом на **Чтение** спецификации шаблона могут развернуть его. Сведения о предоставлении доступа см. [в разделе Учебник. Предоставление группе доступа к ресурсам Azure с помощью Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

Спецификации шаблонов можно развернуть с помощью портала, PowerShell, Azure CLI или в виде связанного шаблона в более крупном развертывании шаблона. Пользователи в организации могут развернуть шаблон спецификации в любой области в Azure (Группа ресурсов, подписка, Группа управления или клиент).

Вместо передачи пути или URI для шаблона вы развертываете спецификацию шаблона, предоставляя идентификатор ресурса. Идентификатор ресурса имеет следующий формат:

**/субскриптионс/{субскриптион-ид}/ресаурцеграупс/{ресаурце-грауп}/провидерс/микрософт.ресаурцес/темплатеспекс/{темплате-спек-наме}/версионс/{темплате-спек-версион}**

Обратите внимание, что идентификатор ресурса включает имя версии для спецификации шаблона.

Например, вы развертываете спецификацию шаблона с помощью следующей команды.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0a"

New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0a"

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

На практике обычно выполняется `Get-AzTemplateSpec` или используется `az ts show` для получения идентификатора спецификации шаблона, которую требуется развернуть.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = (Get-AzTemplateSpec -Name storageSpec -ResourceGroupName templateSpecsRg -Version 1.0a).Versions.Id

New-AzResourceGroupDeployment `
  -ResourceGroupName demoRG `
  -TemplateSpecId $id
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
id = $(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0a" --query "id")

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

Чтобы развернуть спецификацию шаблона, можно также открыть URL-адрес в следующем формате:

```url
https://portal.azure.com/#create/Microsoft.Template/templateSpecVersionId/%2fsubscriptions%2f{subscription-id}%2fresourceGroups%2f{resource-group-name}%2fproviders%2fMicrosoft.Resources%2ftemplateSpecs%2f{template-spec-name}%2fversions%2f{template-spec-version}
```

## <a name="parameters"></a>Параметры

Передача параметров в спецификацию шаблона в точности аналогична передаче параметров в шаблон ARM. Добавьте значения параметров как встроенные, так и в файле параметров.

Чтобы передать параметр встроенным, используйте:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -StorageAccountType Standard_GRS
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters storageAccountType='Standard_GRS'
```

---

Чтобы создать локальный файл параметров, используйте:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageAccountType": {
      "value": "Standard_GRS"
    }
  }
}
```

И передайте этот файл параметров с помощью:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -TemplateParameterFile ./mainTemplate.parameters.json
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters "./mainTemplate.parameters.json"
```

---

## <a name="versioning"></a>Управление версиями

При создании спецификации шаблона необходимо указать для нее имя версии. По мере выполнения итерации по коду шаблона можно либо обновить существующую версию (для исправлений), либо опубликовать новую версию. Версия является текстовой строкой. Можно выбрать любую систему управления версиями, включая семантическую версию. Пользователи спецификации шаблона могут предоставить имя версии, которое они хотят использовать при развертывании.

## <a name="use-tags"></a>Использование тегов

[Теги](../management/tag-resources.md) помогают логически упорядочивать ресурсы. Добавить теги в спецификации шаблона можно с помощью Azure PowerShell и Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzTemplateSpec `
  -Name storageSpec `
  -Version 1.0a `
  -ResourceGroupName templateSpecsRg `
  -Location westus2 `
  -TemplateFile ./mainTemplate.json `
  -Tag @{Dept="Finance";Environment="Production"}
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json" \
  --tags Dept=Finance Environment=Production
```

---

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Set-AzTemplateSpec `
  -Name storageSpec `
  -Version 1.0a `
  -ResourceGroupName templateSpecsRg `
  -Location westus2 `
  -TemplateFile ./mainTemplate.json `
  -Tag @{Dept="Finance";Environment="Production"}
```

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli
az ts update \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json" \
  --tags Dept=Finance Environment=Production
```

---

При создании или изменении спецификации шаблона с указанным параметром версии, но без параметра TAG или Tags:

- Если спецификация шаблона существует и содержит теги, но версия не существует, Новая версия наследует те же теги, что и существующая спецификация шаблона.

При создании или изменении спецификации шаблона с параметром TAG или Tags и указанным параметром version:

- Если и спецификация шаблона, и версия не существуют, теги добавляются как в новую спецификацию шаблона, так и в новую версию.
- Если спецификация шаблона существует, но ее версия не существует, теги добавляются только в новую версию.
- Если существует и спецификация шаблона, и версия, то теги применяются только к версии.

При изменении шаблона с заданным параметром TAG или Tags, но без указания параметра версии теги добавляются только в спецификацию шаблона.

## <a name="create-a-template-spec-with-linked-templates"></a>Создание спецификации шаблона со связанными шаблонами

Если основной шаблон для спецификации шаблона ссылается на связанные шаблоны, команды PowerShell и CLI могут автоматически находить и упаковывать связанные шаблоны с локального диска. Вам не нужно вручную настраивать учетные записи хранения или репозитории для размещения спецификаций шаблонов. все содержится в ресурсе спецификации шаблона.

Следующий пример состоит из основного шаблона с двумя связанными шаблонами. Пример является только выдержкой из шаблона. Обратите внимание, что для `relativePath` связи с другими шаблонами используется свойство с именем. `apiVersion` `2020-06-01` Для ресурса развертывания необходимо использовать или более поздней версии.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/webapp.json"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/database.json"
        }
      }
    }
  ],
  "outputs": {}
}
```

При выполнении команды PowerShell или CLI для создания спецификации шаблона в предыдущем примере команда находит три файла: основной шаблон, шаблон веб-приложения ( `webapp.json` ) и шаблон базы данных ( `database.json` ), а затем упаковывает их в спецификацию шаблона.

Дополнительные сведения см. в разделе [учебник. Создание шаблона спецификации со связанными шаблонами](template-specs-create-linked.md).

## <a name="deploy-template-spec-as-a-linked-template"></a>Развертывание спецификации шаблона в качестве связанного шаблона

После создания спецификации шаблона ее легко использовать из шаблона ARM или другой спецификации шаблона. Чтобы создать ссылку на спецификацию шаблона, добавьте ее идентификатор ресурса в шаблон. Спецификация связанного шаблона автоматически развертывается при развертывании основного шаблона. Такое поведение позволяет разрабатывать спецификации модульных шаблонов и использовать их по мере необходимости.

Например, можно создать шаблон спецификации, который развертывает сетевые ресурсы, и другую спецификацию шаблона, которая развертывает ресурсы хранилища. В шаблонах ARM вы можете ссылаться на эти две спецификации шаблонов в любое время, когда вам понадобится настроить сетевые ресурсы или хранилища.

Следующий пример похож на предыдущий пример, но вы используете `id` свойство для связи с спецификацией шаблона, а не со `relativePath` свойством для связи с локальным шаблоном. Используйте `2020-06-01` для ресурса deployments в качестве версии API. В этом примере спецификации шаблонов находятся в группе ресурсов с именем **темплатеспексрг**.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "networkingDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'networkingSpec', '1.0a')]"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "storageDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0a')]"
        }
      }
    }
  ],
  "outputs": {}
}
```

Дополнительные сведения о связывании спецификаций шаблонов см. в разделе [учебник. Развертывание шаблона спецификации в качестве связанного шаблона](template-specs-deploy-linked-template.md).

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о создании и развертывании шаблона см. в разделе [Краткое руководство. Создание и развертывание спецификации шаблона](quickstart-create-template-specs.md).

* Дополнительные сведения о связывании шаблонов в спецификациях шаблонов см. в разделе [учебник. Создание шаблона спецификации со связанными шаблонами](template-specs-create-linked.md).

* Дополнительные сведения о развертывании спецификации шаблона в качестве связанного шаблона см. [в разделе Учебник. Развертывание шаблона спецификации в качестве связанного шаблона](template-specs-deploy-linked-template.md).
