---
title: Поставщики ресурсов и типы ресурсов
description: Описывает поставщики ресурсов, которые поддерживают Azure Resource Manager. Здесь описываются их схемы, доступные версии API и регионы, в которых могут размещаться ресурсы.
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: devx-track-azurecli
ms.openlocfilehash: 584f3065d0e696f2ee379a8cf6c048994a1e68d5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103493141"
---
# <a name="azure-resource-providers-and-types"></a>Поставщики и типы ресурсов Azure

При развертывании ресурсов часто бывает необходимо получить сведения о типах и поставщиках ресурсов. Например, чтобы хранить ключи и секреты, вам придется работать с поставщиком ресурсов Microsoft.KeyVault. Этот поставщик ресурсов предлагает тип ресурса с именем vaults для создания хранилища ключей.

Формат имени типа ресурса: **{поставщик_ресурса}/{тип_ресурса}**. Тип ресурса для хранилища ключей — **Microsoft.KeyVault/vaults**.

Вы узнаете, как выполнять следующие задачи:

* просмотр всех поставщиков ресурсов в Azure;
* проверка состояния регистрации поставщика ресурсов;
* регистрация поставщика ресурсов;
* просмотр типов ресурсов для поставщика ресурсов;
* просмотр допустимых расположений для типа ресурса;
* просмотр допустимых версий API для типа ресурса.

Эти действия можно выполнить с помощью портал Azure, Azure PowerShell или Azure CLI.

Список поставщиков ресурсов для служб Azure см. в статье [Поставщики ресурсов для служб Azure](azure-services-resource-providers.md).

## <a name="register-resource-provider"></a>Регистрация поставщика ресурсов

Перед использованием поставщика ресурсов необходимо зарегистрировать подписку Azure для поставщика ресурсов. Регистрация настраивает подписку для работы с поставщиком ресурсов. Некоторые поставщики ресурсов регистрируются по умолчанию. Список поставщиков ресурсов, зарегистрированных по умолчанию, см. в статье [поставщики ресурсов для служб Azure](azure-services-resource-providers.md).

Другие поставщики ресурсов регистрируются автоматически при выполнении определенных действий. При развертывании шаблона Azure Resource Manager все необходимые поставщики ресурсов регистрируются автоматически. При создании ресурса с помощью портала поставщик ресурсов обычно регистрируется для вас. Для других сценариев может потребоваться вручную зарегистрировать поставщик ресурсов. 

В этой статье показано, как проверить состояние регистрации поставщика ресурсов и зарегистрировать его при необходимости. Необходимо иметь разрешение на выполнение `/register/action` операции для поставщика ресурсов. Разрешение включается в роли участника и владельца.

> [!IMPORTANT]
> Зарегистрируйте поставщик ресурсов только тогда, когда будете готовы использовать его. Этап регистрации позволяет поддерживать минимальные привилегии в рамках подписки. Пользователь-злоумышленник не может использовать незарегистрированные поставщики ресурсов.

Код приложения не должен блокировать создание ресурсов для поставщика ресурсов, который находится в состоянии **регистрации** . При регистрации поставщика ресурсов операция выполняется отдельно для каждого поддерживаемого региона. Для создания ресурсов в регионе необходимо выполнить регистрацию только в этом регионе. Если поставщик ресурсов не блокируется в состоянии регистрации, приложение может продолжить работу намного быстрее, чем ждать завершения всех регионов.

Вы не можете отменить регистрацию поставщика ресурсов, если у вас по-прежнему есть типы ресурсов от этого поставщика ресурсов в вашей подписке.

## <a name="azure-portal"></a>Портал Azure

### <a name="register-resource-provider"></a>Регистрация поставщика ресурсов

Чтобы просмотреть всех поставщиков ресурсов, а также состояние регистрации для подписки, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com).
1. В меню портал Azure найдите пункт **подписки**. Выберите подходящий вариант из предложенных.

   :::image type="content" source="./media/resource-providers-and-types/search-subscriptions.png" alt-text="Поиск по подпискам":::

1. Выберите подписку, которую нужно просмотреть.

   :::image type="content" source="./media/resource-providers-and-types/select-subscription.png" alt-text="Выбор подписок":::

1. В разделе **Параметры** выберите **Поставщики ресурсов**.

   :::image type="content" source="./media/resource-providers-and-types/select-resource-providers.png" alt-text="Выбор поставщиков ресурсов":::

6. Найдите поставщик ресурсов, который нужно зарегистрировать, и нажмите кнопку **зарегистрировать**. Для поддержания минимальных привилегий в подписке следует регистрировать только те поставщики ресурсов, которые готовы к использованию.

   :::image type="content" source="./media/resource-providers-and-types/register-resource-provider.png" alt-text="Регистрация поставщиков ресурсов":::

### <a name="view-resource-provider"></a>Просмотр поставщика ресурсов

Чтобы просмотреть сведения для конкретного поставщика ресурсов, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com).
2. В меню портала Azure выберите **Все службы**.
3. В поле **Все службы** введите **обозреватель ресурсов** и выберите **Обозреватель ресурсов**.

    ![Пункт "Все службы"](./media/resource-providers-and-types/select-resource-explorer.png)

4. Разверните список **Поставщики**, щелкнув стрелку вправо.

    ![выбор поставщиков](./media/resource-providers-and-types/select-providers.png)

5. Разверните поставщик ресурсов и тип ресурса, который вы хотите просмотреть.

    ![Выбор типа ресурсов](./media/resource-providers-and-types/select-resource-type.png)

6. Диспетчер ресурсов поддерживается во всех регионах, однако развертываемые вами ресурсы могут поддерживаться не во всех регионах. Кроме того, в подписке могут быть ограничения, которые не позволяют использовать некоторые регионы, поддерживающие этот ресурс. Обозреватель ресурсов отображает допустимые расположения для типа ресурса.

    ![Отображение расположений](./media/resource-providers-and-types/show-locations.png)

7. Версия API соответствует версии операций API REST, выполняемых поставщиком ресурсов. При появлении в поставщике ресурсов новых возможностей выпускается новая версия REST API. Обозреватель ресурсов отображает допустимые версии API для типа ресурса.

    ![Отображение версий API](./media/resource-providers-and-types/show-api-versions.png)

## <a name="azure-powershell"></a>Azure PowerShell

Чтобы просмотреть всех поставщиков ресурсов в Azure, а также состояние регистрации для подписки, используйте команду ниже:

```azurepowershell-interactive
Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

Она возвращает результаты, подобные следующим:

```output
ProviderNamespace                RegistrationState
-------------------------------- ------------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

Чтобы просмотреть все зарегистрированные поставщики ресурсов для подписки, используйте:

```azurepowershell-interactive
 Get-AzResourceProvider -ListAvailable | Where-Object RegistrationState -eq "Registered" | Select-Object ProviderNamespace, RegistrationState | Sort-Object ProviderNamespace
```

Для поддержания минимальных привилегий в подписке следует регистрировать только те поставщики ресурсов, которые готовы к использованию. Чтобы зарегистрировать поставщик ресурсов, воспользуйтесь командой:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
```

Она возвращает результаты, подобные следующим:

```output
ProviderNamespace : Microsoft.Batch
RegistrationState : Registering
ResourceTypes     : {batchAccounts, operations, locations, locations/quotas}
Locations         : {West Europe, East US, East US 2, West US...}
```

Чтобы просмотреть сведения для конкретного поставщика ресурсов, используйте команду ниже:

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Batch
```

Она возвращает результаты, подобные следующим:

```output
{ProviderNamespace : Microsoft.Batch
RegistrationState : Registered
ResourceTypes     : {batchAccounts}
Locations         : {West Europe, East US, East US 2, West US...}

...
```

Чтобы просмотреть типы ресурсов для поставщика ресурсов, используйте команду ниже:

```azurepowershell-interactive
(Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes.ResourceTypeName
```

Возвращаемые данные:

```output
batchAccounts
operations
locations
locations/quotas
```

Версия API соответствует версии операций API REST, выполняемых поставщиком ресурсов. При появлении в поставщике ресурсов новых возможностей выпускается новая версия REST API.

Чтобы получить список доступных версий API для типа ресурса, используйте следующую команду:

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).ApiVersions
```

Возвращаемые данные:

```output
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

Диспетчер ресурсов поддерживается во всех регионах, однако развертываемые вами ресурсы могут поддерживаться не во всех регионах. Кроме того, в подписке могут быть ограничения, которые не позволяют использовать некоторые регионы, поддерживающие этот ресурс.

Чтобы получить список поддерживаемых расположений для типа ресурса, используйте следующую команду:

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

Возвращаемые данные:

```output
West Europe
East US
East US 2
West US
...
```

## <a name="azure-cli"></a>Azure CLI

Чтобы просмотреть всех поставщиков ресурсов в Azure, а также состояние регистрации для подписки, используйте команду ниже:

```azurecli-interactive
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

Она возвращает результаты, подобные следующим:

```output
Provider                         Status
-------------------------------- ----------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

Чтобы просмотреть все зарегистрированные поставщики ресурсов для подписки, используйте:

```azurecli-interactive
az provider list --query "sort_by([?registrationState=='Registered'].{Provider:namespace, Status:registrationState}, &Provider)" --out table
```

Для поддержания минимальных привилегий в подписке следует регистрировать только те поставщики ресурсов, которые готовы к использованию. Чтобы зарегистрировать поставщик ресурсов, воспользуйтесь командой:

```azurecli-interactive
az provider register --namespace Microsoft.Batch
```

Оно возвращает сообщение о выполнении регистрации.

Чтобы просмотреть сведения для конкретного поставщика ресурсов, используйте команду ниже:

```azurecli-interactive
az provider show --namespace Microsoft.Batch
```

Она возвращает результаты, подобные следующим:

```output
{
    "id": "/subscriptions/####-####/providers/Microsoft.Batch",
    "namespace": "Microsoft.Batch",
    "registrationsState": "Registering",
    "resourceTypes:" [
        ...
    ]
}
```

Чтобы просмотреть типы ресурсов для поставщика ресурсов, используйте команду ниже:

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[*].resourceType" --out table
```

Возвращаемые данные:

```output
Result
---------------
batchAccounts
operations
locations
locations/quotas
```

Версия API соответствует версии операций API REST, выполняемых поставщиком ресурсов. При появлении в поставщике ресурсов новых возможностей выпускается новая версия REST API.

Чтобы получить список доступных версий API для типа ресурса, используйте следующую команду:

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].apiVersions | [0]" --out table
```

Возвращаемые данные:

```output
Result
---------------
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

Диспетчер ресурсов поддерживается во всех регионах, однако развертываемые вами ресурсы могут поддерживаться не во всех регионах. Кроме того, в подписке могут быть ограничения, которые не позволяют использовать некоторые регионы, поддерживающие этот ресурс.

Чтобы получить список поддерживаемых расположений для типа ресурса, используйте следующую команду:

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" --out table
```

Возвращаемые данные:

```output
Result
---------------
West Europe
East US
East US 2
West US
...
```

## <a name="next-steps"></a>Следующие шаги

* Дополнительные сведения о создании шаблонов диспетчер ресурсов см. в разделе [Создание шаблонов Azure Resource Manager](../templates/template-syntax.md). 
* Схемы шаблонов поставщиков ресурсов см. в статье [Define resources in Azure Resource Manager templates](/azure/templates/) (Определение ресурсов в шаблонах Azure Resource Manager).
* Список поставщиков ресурсов для служб Azure см. в статье [Поставщики ресурсов для служб Azure](azure-services-resource-providers.md).
* Чтобы просмотреть операции для поставщика ресурсов, ознакомьтесь с [Azure REST API](/rest/api/).
