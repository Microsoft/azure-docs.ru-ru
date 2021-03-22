---
title: Программное создание подписок Azure для Соглашения с партнером Майкрософт с помощью новейших интерфейсов API
description: Узнайте, как создавать подписки Azure для Соглашения с партнером Майкрософт программными средствами с помощью последних версий REST API, Azure CLI и Azure PowerShell.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 11/17/2020
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: de1183c1364fcb7e5483559899c2939df15d26b6
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102215788"
---
# <a name="programmatically-create-azure-subscriptions-for-a-microsoft-partner-agreement-with-the-latest-apis"></a>Программное создание подписок Azure для Соглашения с партнером Майкрософт с помощью новейших интерфейсов API

Эта статья поможет вам создать подписки Azure для Соглашения с партнером Майкрософт программными средствами с помощью новейших версий API. Если вы все еще используете более раннюю предварительную версию, см. статью [Программное создание подписок Azure с помощью предварительных версий API](programmatically-create-subscription-preview.md). 

В этой статье Вы узнаете как создавать подписки программно с помощью Azure Resource Manager.

Когда вы создаете подписки Azure программными средствами, они регулируются соглашением, в соответствии с которым вы получили службы Azure от корпорации Майкрософт или уполномоченного торгового посредника. Дополнительные сведения можно найти на странице [Юридическая информация Службы Microsoft Azure](https://azure.microsoft.com/support/legal/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Чтобы создать подписку для учетной записи выставления счетов, необходимо иметь роль глобального администратора или агента администратора в учетной записи поставщика облачных решений вашей организации. Дополнительные сведения см. в статье [Назначение ролей и разрешений пользователям компании для работы в Центре партнеров](/partner-center/permissions-overview).

Если вы не знаете точно, есть ли у вас доступ к учетной записи Соглашения с партнером Майкрософт, ознакомьтесь с разделом [Проверка доступа к Соглашению с партнером Майкрософт](mpa-request-ownership.md#check-access-to-a-microsoft-partner-agreement).

В следующих примерах используются интерфейсы REST API. В настоящее время PowerShell и Azure CLI не поддерживаются.

## <a name="find-the-billing-accounts-that-you-have-access-to"></a>Поиск учетных записей выставления счетов, к которым у вас есть доступ

Отправьте следующий запрос, чтобы получить список всех учетных записей выставления счетов, к которым у вас есть доступ.

### <a name="rest"></a>[REST](#tab/rest-getBillingAccount-MPA)

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingaccounts/?api-version=2020-05-01
```

В ответе API будут перечислены учетные записи выставления счетов.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountStatus": "Active",
        "accountType": "Partner",
        "agreementType": "MicrosoftPartnerAgreement",
        "billingProfiles": {
          "hasMoreResults": false
        },
        "displayName": "Contoso",
        "hasReadAccess": true
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}
```

Используйте свойство `displayName`, чтобы определить учетную запись выставления счетов, для которой нужно создать подписки. Убедитесь, что значение agreementType для учетной записи равно *MicrosoftPartnerAgreement*. Скопируйте `name` для учетной записи. Например, чтобы создать подписку для учетной записи выставления счетов `Contoso`, скопируйте `99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx`. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

<!--
### [PowerShell](#tab/azure-powershell-getBillingAccounts-MPA)

we're still working on enabling PowerShell SDK for billing APIs. Check back soon.
-->

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli-getBillingAccounts-MPA)

```azurecli
> az billing account list
```
Вы получите список всех учетных записей выставления счетов, к которым у вас есть доступ. 

```json
[
  {
    "accountStatus": "Active",
    "accountType": "Partner",
    "agreementType": "MicrosoftPartnerAgreement",
    "billingProfiles": {
      "hasMoreResults": false,
      "value": null
    },
    "departments": null,
    "displayName": "Contoso",
    "enrollmentAccounts": null,
    "enrollmentDetails": null,
    "hasReadAccess": true,
    "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
    "name": "99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
    "soldTo": null,
    "type": "Microsoft.Billing/billingAccounts"
  }
]
```

Используйте свойство displayName, чтобы определить учетную запись выставления счетов, для которой нужно создать подписки. Убедитесь, что значение agreementType для учетной записи равно MicrosoftPartnerAgreement. Скопируйте имя для учетной записи. Например, чтобы создать подписку для учетной записи выставления счетов Contoso, скопируйте 99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

---

## <a name="find-customers-that-have-azure-plans"></a>Поиск клиентов, имеющих планы Azure

Отправьте следующий запрос со значением `name`, скопированным на первом шаге (```99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx```), чтобы получить список всех клиентов в учетной записи выставления счетов, для которых вы можете создавать подписки Azure.

### <a name="rest"></a>[REST](#tab/rest-getCustomers)

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers?api-version=2020-05-01
```

В ответе API будут перечислены клиенты в учетной записи выставления счетов с планами Azure. Вы можете создать подписки для этих клиентов.

```json
{
  "totalCount": 2,
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/7d15644f-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "7d15644f-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "properties": {
        "billingProfileDisplayName": "Fabrikam toys Billing Profile",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/YL4M-xxxx-xxx-xxx",
        "displayName": "Fabrikam toys"
      },
      "type": "Microsoft.Billing/billingAccounts/customers"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "properties": {
        "billingProfileDisplayName": "Contoso toys Billing Profile",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/YL4M-xxxx-xxx-xxx",
        "displayName": "Contoso toys"
      },
      "type": "Microsoft.Billing/billingAccounts/customers"
    }
  ]
}

```

Используйте свойство `displayName`, чтобы определить клиентов, для которых нужно создать подписки. Скопируйте значение `id` для клиента. Например, чтобы создать подписку для `Fabrikam toys`, скопируйте `/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/7d15644f-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Вставьте это значение в любое место, чтобы использовать его для последующих шагов.

<!--
### [PowerShell](#tab/azure-powershell-getCustomers)

we're still working on enabling PowerShell SDK for billing APIs. Check back soon.
-->


### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli-getCustomers)

```json
> az billing customer list --account-name 99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx
```

В ответе API будут перечислены клиенты в учетной записи выставления счетов с планами Azure. Вы можете создать подписки для этих клиентов.

```json
[
  {
    "billingProfileDisplayName": "Fabrikam toys Billing Profile",
    "billingProfileId": "providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/7d15644f-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "displayName": "Fabrikam toys",
    "id": "providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/7d15644f-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "resellers": null,
    "type": "Microsoft.Billing/billingAccounts/customers"
  },
  {
    "billingProfileDisplayName": "Contoso toys Billing Profile",
    "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "displayName": "Contoso toys",
    "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "d49c364c-f866-4cc2-a284-d89f369b7951",
    "resellers": null,
    "type": "Microsoft.Billing/billingAccounts/customers"
  }
]

```

Используйте свойство `displayName`, чтобы определить клиентов, для которых нужно создать подписки. Скопируйте значение `id` для клиента. Например, чтобы создать подписку для `Fabrikam toys`, скопируйте `/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/7d15644f-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Вставьте это значение в любое место, чтобы использовать его для последующих шагов.

---

## <a name="get-the-resellers-for-a-customer"></a>получение списка торговых посредников для клиента

Этот раздел является необязательным и предназначен только для косвенных поставщиков.

Непрямой поставщик в двухуровневой модели CSP может указать торгового посредника при создании подписок для клиентов.

### <a name="rest"></a>[REST](#tab/rest-getIndirectResellers)

Отправьте следующий запрос со значением `id`, скопированным на втором шаге (```/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx```), чтобы получить список всех торговых посредников, доступных для клиента.

```json
GET "https://management.azure.com/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx?$expand=resellers&api-version=2020-05-01"
```
В ответе API будут перечислены торговые посредники для клиента.

```json
{
  "value": [{
  "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Billing/billingAccounts/customers",
  "properties": {
    "billingProfileDisplayName": "Fabrikam toys Billing Profile",
    "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/YL4M-xxxx-xxx-xxx",
    "displayName": "Fabrikam toys",
    "resellers": [
      {
        "resellerId": "3xxxxx",
        "description": "Wingtip"
      }
    ]
  }
}]
}
```

Используйте свойство `description`, чтобы определить торгового посредника, который связан с подпиской. Скопируйте значение `resellerId` для торгового посредника. Например, чтобы связать `Wingtip`, скопируйте `3xxxxx`. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

<!--
### [PowerShell](#tab/azure-powershell-getIndirectResellers)

we're still working on enabling PowerShell SDK for billing APIs. Check back soon.
-->

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli-getIndirectResellers)

Выполните следующий запрос со значением `name`, скопированным на первом шаге (```99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx```) и именем клиента (`name`), скопированным на предыдущем шаге (```acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx```).

```azurecli-interactive
 > az billing customer show --expand "enabledAzurePlans,resellers" --account-name "99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx" --name "acba85c9-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

В ответе API будут перечислены торговые посредники для клиента.

```json
{
  "billingProfileDisplayName": "Fabrikam toys Billing Profile",
  "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/YL4M-xxxx-xxx-xxx",
  "displayName": "Fabrikam toys",
  "enabledAzurePlans": [
    {
      "skuDescription": "Microsoft Azure Plan",
      "skuId": "0001"
    }
  ],
  "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resellers": [
    {
      "description": "Wingtip",
      "resellerId": "3xxxxx"
    }
  ],
  "type": "Microsoft.Billing/billingAccounts/customers"
}

```

Используйте свойство `description`, чтобы определить торгового посредника, который связан с подпиской. Скопируйте значение `resellerId` для торгового посредника. Например, чтобы связать `Wingtip`, скопируйте `3xxxxx`. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

---

## <a name="create-a-subscription-for-a-customer"></a>Создание подписки для клиента

В следующем примере создается подписка с именем *Dev Team subscription* для *Fabrikam toys*, которая затем связывается с торговым посредником *Wingtip*. Используйте область выставления счетов, скопированную из предыдущего шага: `/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. 

### <a name="rest"></a>[REST](#tab/rest-MPA)

```json
PUT  https://management.azure.com/providers/Microsoft.Subscription/aliases/sampleAlias?api-version=2020-09-01
```

### <a name="request-body"></a>Тело запроса

```json
{
  "properties":
    {
        "billingScope": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "DisplayName": "Dev Team subscription",
        "Workload": "Production"
    }
}
```
### <a name="response"></a>Ответ

```json
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "type": "Microsoft.Subscription/aliases",
  "properties": {
    "subscriptionId": "b5bab918-e8a9-4c34-a2e2-ebc1b75b9d74",
    "provisioningState": "Accepted"
  }
}
```

Чтобы получить состояние запроса, можно отправить запрос GET для того же URL-адреса.

### <a name="request"></a>Запрос

```json
GET https://management.azure.com/providers/Microsoft.Subscription/aliases/sampleAlias?api-version=2020-09-01
```

### <a name="response"></a>Ответ

```json
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "type": "Microsoft.Subscription/aliases",
  "properties": {
    "subscriptionId": "b5bab918-e8a9-4c34-a2e2-ebc1b75b9d74",
    "provisioningState": "Succeeded"
  }
}
```

Состояние "Выполняется" возвращается в виде состояния `Accepted` в разделе `provisioningState`. 

Передайте в тексте запроса API необязательное значение *resellerId*, скопированное на втором шаге.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell-MPA)

Чтобы установить последнюю версию модуля, содержащего командлет `New-AzSubscriptionAlias`, запустите `Install-Module Az.Subscription`. Сведения о том, как установить последнюю версию PowerShellGet, см. в статье о [получении модуля PowerShellGet](/powershell/scripting/gallery/installing-psget).

Воспользуйтесь следующей командой [New-AzSubscriptionAlias](/powershell/module/az.subscription/new-azsubscription), указав область выставления счетов `"/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`. 

```azurepowershell-interactive
New-AzSubscriptionAlias -AliasName "sampleAlias" -SubscriptionName "Dev Team Subscription" -BillingScope "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -Workload 'Production"
```

Вы получите subscriptionId в ответе команды.

```azurepowershell
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "properties": {
    "provisioningState": "Succeeded",
    "subscriptionId": "4921139b-ef1e-4370-a331-dd2229f4f510"
  },
  "type": "Microsoft.Subscription/aliases"
}
```

При вызове `New-AzSubscriptionAlias` передайте необязательное значение *resellerId*, скопированное на втором шаге.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli-MPA)

Сначала установите это расширение с помощью команд `az extension add --name account` и `az extension add --name alias`.

Воспользуйтесь следующей командой [az account alias create](/cli/azure/ext/account/account/alias#ext_account_az_account_alias_create). 

```azurecli-interactive
az account alias create --name "sampleAlias" --billing-scope "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx" --display-name "Dev Team Subscription" --workload "Production"
```

Вы получите subscriptionId в ответе команды.

```azurecli
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "properties": {
    "provisioningState": "Succeeded",
    "subscriptionId": "4921139b-ef1e-4370-a331-dd2229f4f510"
  },
  "type": "Microsoft.Subscription/aliases"
}
```

При вызове `az account alias create` передайте необязательное значение *resellerId*, скопированное на втором шаге.

---

## <a name="next-steps"></a>Дальнейшие действия

* После создания подписки можно предоставить эту возможность для других пользователей и субъектов-служб. Дополнительные сведения см. статье [Предоставление доступа к созданию подписок Azure Enterprise (предварительная версия)](grant-access-to-create-subscription.md).
* Дополнительные сведения о переносе большого числа подписок с помощью групп управления см. в руководстве по [упорядочиванию ресурсов с помощью групп управления Azure](../../governance/management-groups/overview.md).
