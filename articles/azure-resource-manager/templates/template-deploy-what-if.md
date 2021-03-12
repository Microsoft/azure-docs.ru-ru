---
title: Шаблоны развертывания что если
description: Прежде чем развертывать шаблон Azure Resource Manager, определите, какие изменения будут выполнены для ресурсов.
author: tfitzmac
ms.topic: conceptual
ms.date: 03/09/2021
ms.author: tomfitz
ms.openlocfilehash: 3f35f00e5fcd26c7aa4bb685dc80f48eabb88b6d
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102547940"
---
# <a name="arm-template-deployment-what-if-operation"></a>Операция "что если" развертывания шаблона ARM

Перед развертыванием шаблона Azure Resource Manager (шаблон ARM) можно просмотреть изменения, которые будут выполняться. Azure Resource Manager предоставляет операцию "что если", которая позволяет увидеть, как ресурсы изменятся при развертывании шаблона. Операция "что если" не вносит изменений в существующие ресурсы. Вместо этого она прогнозирует изменения, которые произойдут при развертывании указанного шаблона.

Операцию "что если" можно использовать с операциями Azure PowerShell, Azure CLI или REST API. Что если поддерживается для групп ресурсов, подписок, групп управления и развертывания на уровне клиента.

## <a name="install-azure-powershell-module"></a>Установите модуль Azure PowerShell.

Чтобы использовать то, что в PowerShell, необходимо установить **модуль AZ версии 4,2 или более поздней**.

Чтобы установить модуль, используйте:

```powershell
Install-Module -Name Az -Force
```

Дополнительные сведения об установке модулей см. в разделе [Install Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="install-azure-cli-module"></a>Установка модуля Azure CLI

Чтобы использовать операцию "что если" в Azure CLI, необходимо иметь Azure CLI 2.5.0 или более поздней версии. При необходимости [установите последнюю версию Azure CLI](/cli/azure/install-azure-cli).

## <a name="see-results"></a>Просмотр результатов

При использовании функции что если в PowerShell или Azure CLI, выходные данные включают цветовые результаты, которые помогают увидеть различные типы изменений.

![диспетчер ресурсов развертывания шаблона что такое операция фуллресаурцепайлоад и изменение типов](./media/template-deploy-what-if/resource-manager-deployment-whatif-change-types.png)

Текстовые выходные данные:

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner: "Team A"
    ~ properties.addressSpace.addressPrefixes: [
      - 0: "10.0.0.0/16"
      + 0: "10.0.0.0/15"
      ]
    ~ properties.subnets: [
      - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

      ]

Resource changes: 1 to modify.
```

> [!NOTE]
> Операция "что если" не может разрешить [функцию ссылки](template-functions-resource.md#reference). Каждый раз, когда свойству задается выражение шаблона, включающее функцию Reference, то что если сообщает, что свойство изменится. Это происходит потому, что, если сравнивает текущее значение свойства (например, `true` или `false` для логического значения) с неразрешенным выражением шаблона. Очевидно, что эти значения не будут совпадать. При развертывании шаблона свойство изменится только в том случае, если выражение шаблона разрешается в другое значение.

## <a name="what-if-commands"></a>Команды "что если"

### <a name="azure-powershell"></a>Azure PowerShell

Чтобы просмотреть изменения перед развертыванием шаблона, используйте [New-азресаурцеграупдеплоймент](/powershell/module/az.resources/new-azresourcegroupdeployment) или [New-азсубскриптиондеплоймент](/powershell/module/az.resources/new-azdeployment). Добавьте `-Whatif` параметр Switch в команду развертывания.

* `New-AzResourceGroupDeployment -Whatif` для развертываний группы ресурсов
* `New-AzSubscriptionDeployment -Whatif` и `New-AzDeployment -Whatif` для развертываний на уровне подписки

Можно использовать `-Confirm` параметр Switch для предварительного просмотра изменений и получения запроса на продолжение развертывания.

* `New-AzResourceGroupDeployment -Confirm` для развертываний группы ресурсов
* `New-AzSubscriptionDeployment -Confirm` и `New-AzDeployment -Confirm` для развертываний на уровне подписки

Приведенные выше команды возвращают текстовую сводку, которую можно проверить вручную. Чтобы получить объект, который можно программно проверить на наличие изменений, используйте [Get-азресаурцеграупдеплойментвхатифресулт](/powershell/module/az.resources/get-azresourcegroupdeploymentwhatifresult) или [Get-азсубскриптиондеплойментвхатифресулт](/powershell/module/az.resources/get-azdeploymentwhatifresult).

* `$results = Get-AzResourceGroupDeploymentWhatIfResult` для развертываний группы ресурсов
* `$results = Get-AzSubscriptionDeploymentWhatIfResult` или `$results = Get-AzDeploymentWhatIfResult` для развертываний на уровне подписки

### <a name="azure-cli"></a>Azure CLI

Чтобы просмотреть изменения перед развертыванием шаблона, используйте:

* [AZ Deployment Group, что если](/cli/azure/deployment/group#az-deployment-group-what-if) для развертываний группы ресурсов
* [AZ Deployment подсистема "что если](/cli/azure/deployment/sub#az-deployment-sub-what-if) " для развертываний на уровне подписки
* [AZ Deployment mg что если](/cli/azure/deployment/mg#az-deployment-mg-what-if) для развертываний группы управления
* [AZ развертывание клиент что если](/cli/azure/deployment/tenant#az-deployment-tenant-what-if) для развертываний клиентов

Можно использовать `--confirm-with-what-if` параметр (или его краткую форму `-c` ) для предварительного просмотра изменений и получить запрос на продолжение развертывания. Добавьте этот переключатель в:

* [AZ развертывание группы развертывания](/cli/azure/deployment/group#az-deployment-group-create)
* [AZ Deployment подсоздание](/cli/azure/deployment/sub#az-deployment-sub-create).
* [AZ Deployment mg CREATE](/cli/azure/deployment/mg#az-deployment-mg-create)
* [AZ развертывание создание клиента](/cli/azure/deployment/tenant#az-deployment-tenant-create)

Например, используйте `az deployment group create --confirm-with-what-if` или `-c` для развертываний группы ресурсов.

Приведенные выше команды возвращают текстовую сводку, которую можно проверить вручную. Чтобы получить объект JSON, который можно программно проверить на наличие изменений, используйте `--no-pretty-print` параметр. Например, используйте `az deployment group what-if --no-pretty-print` для развертывания группы ресурсов.

Если вы хотите вернуть результаты без цветов, откройте файл [конфигурации Azure CLI](/cli/azure/azure-cli-configuration) . Задайте для параметра **no_color** значение **Да**.

### <a name="azure-rest-api"></a>Azure REST API

Для REST API используйте:

* [Развертывания — What If](/rest/api/resources/deployments/whatif) развертываний группы ресурсов
* [Развертывания — What If в области подписки](/rest/api/resources/deployments/whatifatsubscriptionscope) для развертываний подписки
* [Развертывания — What If в области группы управления](/rest/api/resources/deployments/whatifatmanagementgroupscope) для развертываний группы управления
* [Развертывания — What If в области клиента](/rest/api/resources/deployments/whatifattenantscope) для развертываний клиента.

## <a name="change-types"></a>Типы изменений

Операция "что если" перечисляет шесть различных типов изменений:

- **Создать**: ресурс в настоящее время не существует, но определен в шаблоне. Он будет создан.

- **Delete**: этот тип изменения применяется только при использовании [полного режима](deployment-modes.md) развертывания. Ресурс существует, но не определен в шаблоне. В полном режиме ресурс будет удален. В этот тип изменений включаются только ресурсы, [поддерживающие удаление полного режима](complete-mode-deletion.md) .

- **Ignore**: ресурс существует, но не определен в шаблоне. Он не будет развернут или изменен.

- **Без изменений**: ресурс существует и определен в шаблоне. Он будет развернут повторно, но его свойства не изменятся. Этот тип изменения возвращается, если [ресултформат](#result-format) имеет значение `FullResourcePayloads` , которое является значением по умолчанию.

- **Modify**: ресурс существует и определен в шаблоне. Он будет развернут повторно, и его свойства изменятся. Этот тип изменения возвращается, если [ресултформат](#result-format) имеет значение `FullResourcePayloads` , которое является значением по умолчанию.

- **Deploy**: ресурс существует и определен в шаблоне. Он будет развернут повторно. Свойства ресурса могут измениться или остаться без изменений. Операция возвращает этот тип изменения в случае, когда недостаточно сведений, чтобы определить, изменятся ли какие-либо свойства. Это условие отображается только в том случае, если для [ресултформат](#result-format) задано значение `ResourceIdOnly` .

## <a name="result-format"></a>Формат результата

Вы управляете уровнем детализации, который возвращается о прогнозируемых изменениях. Имеются две возможности.

* **Фуллресаурцепайлоадс** — возвращает список ресурсов, которые будут изменены, и сведения о свойствах, которые будут изменены.
* **Ресаурцеидонли** — возвращает список ресурсов, которые будут изменены

Значение по умолчанию — **фуллресаурцепайлоадс**.

Для команд развертывания PowerShell используйте `-WhatIfResultFormat` параметр. В командах программного объекта используйте `ResultFormat` параметр.

В Azure CLI используйте параметр `--result-format`.
 
Следующие результаты показывают два разных формата выходных данных:

- Полные полезные данные ресурсов

  ```powershell
  Resource and property changes are indicated with these symbols:
    - Delete
    + Create
    ~ Modify

  The deployment will update the following scope:

  Scope: /subscriptions/./resourceGroups/ExampleGroup

    ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
      - tags.Owner: "Team A"
      ~ properties.addressSpace.addressPrefixes: [
        - 0: "10.0.0.0/16"
        + 0: "10.0.0.0/15"
        ]
      ~ properties.subnets: [
        - 0:

          name:                     "subnet001"
          properties.addressPrefix: "10.0.0.0/24"

        ]

  Resource changes: 1 to modify.
  ```

- Только идентификатор ресурса

  ```powershell
  Resource and property changes are indicated with this symbol:
    ! Deploy

  The deployment will update the following scope:

  Scope: /subscriptions/./resourceGroups/ExampleGroup

    ! Microsoft.Network/virtualNetworks/vnet-001

  Resource changes: 1 to deploy.
  ```

## <a name="run-what-if-operation"></a>Выполнить операцию "что если"

### <a name="set-up-environment"></a>Настройка среды

Чтобы узнать, как работает, давайте выполним некоторые тесты. Сначала разверните [шаблон, который создает виртуальную сеть](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/what-if/what-if-before.json). Эта виртуальная сеть будет использоваться для тестирования того, как сообщения о внесении изменений сообщаются в.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name ExampleGroup `
  -Location centralus
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-before.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group create \
  --name ExampleGroup \
  --location "Central US"
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-before.json"
```

---

### <a name="test-modification"></a>Изменение теста

После завершения развертывания вы можете протестировать операцию "что если". На этот раз вы развертываете [шаблон, который изменяет виртуальную сеть](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/what-if/what-if-after.json). В ней отсутствует один исходный тег, подсеть удалена, а префикс адреса изменился.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -Whatif `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group what-if \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

---

Выходные данные операции что если выглядят примерно так:

![Результат операции развертывания шаблона диспетчер ресурсов что-if](./media/template-deploy-what-if/resource-manager-deployment-whatif-change-types.png)

Текстовые выходные данные:

```powershell
Resource and property changes are indicated with these symbols:
  - Delete
  + Create
  ~ Modify

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  ~ Microsoft.Network/virtualNetworks/vnet-001 [2018-10-01]
    - tags.Owner: "Team A"
    ~ properties.addressSpace.addressPrefixes: [
      - 0: "10.0.0.0/16"
      + 0: "10.0.0.0/15"
      ]
    ~ properties.subnets: [
      - 0:

        name:                     "subnet001"
        properties.addressPrefix: "10.0.0.0/24"

      ]

Resource changes: 1 to modify.
```

Обратите внимание, что в верхней части выходных данных определены цвета, указывающие тип изменений.

В нижней части выходных данных показано, что владелец тега был удален. Префикс адреса изменился с 10.0.0.0/16 на 10.0.0.0/15. Подсеть с именем subnet001 удалена. Помните, что эти изменения не были развернуты. Вы увидите предварительный просмотр изменений, которые будут происходить при развертывании шаблона.

Некоторые свойства, перечисленные как удаленные, фактически не меняются. Свойства могут быть ошибочно зарегистрированы как удаленные, если они не находятся в шаблоне, но автоматически устанавливаются во время развертывания в качестве значений по умолчанию. Этот результат считается "пропускаемым" в ответе "что если". Окончательный развернутый ресурс будет иметь значения, заданные для свойств. Как только операция "что если" завершилась, эти свойства будут отфильтрованы из результата.

## <a name="programmatically-evaluate-what-if-results"></a>Программное вычисление результатов поиска

Теперь давайте программно выберем результаты, если присвоить команде значение переменной.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$results = Get-AzResourceGroupDeploymentWhatIfResult `
  -ResourceGroupName ExampleGroup `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json"
```

Можно просмотреть сводку по каждому изменению.

```azurepowershell
foreach ($change in $results.Changes)
{
  $change.Delta
}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
results=$(az deployment group what-if --resource-group ExampleGroup --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/what-if/what-if-after.json" --no-pretty-print)
```

---

## <a name="confirm-deletion"></a>Подтверждение удаления

Операция "что если" поддерживает использование [режима развертывания](deployment-modes.md). Если задано значение полного режима, ресурсы, отсутствующие в шаблоне, удаляются. В следующем примере выполняется развертывание [шаблона, не имеющего ресурсов, определенных](https://github.com/Azure/azure-docs-json-samples/blob/master/empty-template/azuredeploy.json) в полном режиме.

Для предварительного просмотра изменений перед развертыванием шаблона используйте параметр подтверждения с командой развертывания. Если изменения ожидаемы, ответьте на то, что развертывание должно завершиться.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -Mode Complete `
  -Confirm `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/empty-template/azuredeploy.json"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group ExampleGroup \
  --mode Complete \
  --confirm-with-what-if \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/empty-template/azuredeploy.json"
```

---

Так как в шаблоне не определены ресурсы, а для режима развертывания задано значение "завершено", виртуальная сеть будет удалена.

![Диспетчер ресурсов развертывание шаблона "что если" выходное режим развертывания операции завершен](./media/template-deploy-what-if/resource-manager-deployment-whatif-output-mode-complete.png)

Текстовые выходные данные:

```powershell
Resource and property changes are indicated with this symbol:
  - Delete

The deployment will update the following scope:

Scope: /subscriptions/./resourceGroups/ExampleGroup

  - Microsoft.Network/virtualNetworks/vnet-001

      id:
"/subscriptions/./resourceGroups/ExampleGroup/providers/Microsoft.Network/virtualNet
works/vnet-001"
      location:        "centralus"
      name:            "vnet-001"
      tags.CostCenter: "12345"
      tags.Owner:      "Team A"
      type:            "Microsoft.Network/virtualNetworks"

Resource changes: 1 to delete.

Are you sure you want to execute the deployment?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "Y"):
```

Вы увидите ожидаемые изменения и можете подтвердить, что вы хотите выполнить развертывание.

## <a name="sdks"></a>Пакеты SDK

Вы можете использовать операцию "что если" с помощью пакетов SDK Azure.

* Для Python используйте параметр [что если](/python/api/azure-mgmt-resource/azure.mgmt.resource.resources.v2019_10_01.operations.deploymentsoperations#what-if-resource-group-name--deployment-name--properties--location-none--custom-headers-none--raw-false--polling-true----operation-config-).

* Для Java используйте [класс деплойментвхатиф](/java/api/com.microsoft.azure.management.resources.deploymentwhatif).

* Для .NET используйте [класс деплойментвхатиф](/dotnet/api/microsoft.azure.management.resourcemanager.models.deploymentwhatif).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения об использовании операции "что если" в конвейере см. [в разделе тестовые шаблоны ARM с What-If в конвейере](https://4bes.nl/2021/03/06/test-arm-templates-with-what-if/).
- Если вы заметили неправильные результаты операции "что если", сообщите о проблемах по адресу [https://aka.ms/whatifissues](https://aka.ms/whatifissues) .
- Сведения о модуле Microsoft Learn, который охватывает использование функции, см. в разделе [Предварительный просмотр изменений и проверка ресурсов Azure с помощью средств тестирования шаблонов ARM](/learn/modules/arm-template-test/).
