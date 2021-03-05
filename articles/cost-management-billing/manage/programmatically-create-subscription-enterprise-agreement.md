---
title: Программное создание подписок Azure для Соглашения Enterprise с помощью новейших API
description: Сведения о том, как создавать подписки Azure для Соглашения Enterprise программным способом с помощью последних версий REST API, Azure CLI и Azure PowerShell.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 01/13/2021
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 69cc0998be6079b7d3f2ecf209e5a709771ae293
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102200603"
---
# <a name="programmatically-create-azure-enterprise-agreement-subscriptions-with-the-latest-apis"></a>Программное создание подписок Azure для Соглашения Enterprise с помощью новейших API

Эта статья поможет вам создать подписки Azure для учетной записи выставления счетов по Соглашению Enterprise программными средствами с помощью новейших версий API. Если вы все еще используете более раннюю предварительную версию, см. статью [Программное создание подписок Azure с помощью предварительных версий API](programmatically-create-subscription-preview.md). 

В этой статье Вы узнаете как создавать подписки программно с помощью Azure Resource Manager.

Когда вы создаете подписки Azure программными средствами, они регулируются соглашением, в соответствии с которым вы получили службы Azure от корпорации Майкрософт или уполномоченного торгового посредника. Дополнительные сведения можно найти на странице [Юридическая информация Службы Microsoft Azure](https://azure.microsoft.com/support/legal/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Для создания подписки необходимо иметь роль владельца учетной записи регистрации. Ее можно получить двумя способами.

* Администратор регистрации, отвечающий за вашу регистрацию, [может назначить роль владельца учетной записи](https://ea.azure.com/helpdocs/addNewAccount) (требуется вход в систему), что сделает вас владельцем учетной записи регистрации.
* Текущий владелец учетной записи регистрации может [предоставить вам права доступа](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). Аналогично, чтобы использовать субъект-службу для создания подписки с EA, необходимо [предоставить ему возможность создания подписки](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). 
  > [!NOTE]
  > Убедитесь, что вы используете правильную версию API, чтобы предоставить учетной записи регистрации права владельца. При изучении этой статьи и для интерфейсов API, приведенных в ней, используйте API версии [2019-10-01-preview](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). Если вы выполняете миграцию для использования новых версий API, необходимо предоставить разрешение владельца для использования [2019-10-01-preview](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put). Предыдущая конфигурация, выполненная с помощью [версии 2015-07-01](grant-access-to-create-subscription.md), не преобразуется автоматически для использования новых версий API.

## <a name="find-accounts-you-have-access-to"></a>Поиск учетных записей, к которым у вас есть доступ

После того как вы будете добавлены в учетную запись регистрации, связанную с владельцем учетной записи, платформа Azure использует связь между регистрацией и учетной записью, чтобы определить целевой объект для выставления счетов за подписки. Все подписки, созданные с учетной записью, оплачиваются в рамках регистрации EA, к которой относится эта учетная запись. Чтобы создать подписки, необходимо в значениях передать о регистрации учетной записи и субъектов-пользователя — владельца подписки.

Чтобы выполнить следующие команды, войдите в *домашний каталог* владельца учетной записи, в котором по умолчанию создаются подписки.

### <a name="rest"></a>[REST](#tab/rest-getEnrollments)

Запросите вывод списка всех учетных записей регистрации, к которым у вас есть доступ.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingaccounts/?api-version=2020-05-01
```

В ответе API будет выведен список всех учетных записей регистрации, к которым у вас есть доступ.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/1234567",
      "name": "1234567",
      "properties": {
        "accountStatus": "Unknown",
        "accountType": "Enterprise",
        "agreementType": "EnterpriseAgreement",
        "soldTo": {
          "companyName": "Contoso",
          "country": "US "
        },
        "billingProfiles": {
          "hasMoreResults": false
        },
        "displayName": "Contoso",
        "enrollmentAccounts": [
          {
            "id": "/providers/Microsoft.Billing/billingAccounts/1234567/enrollmentAccounts/7654321",
            "name": "7654321",
            "type": "Microsoft.Billing/enrollmentAccounts",
            "properties": {
              "accountName": "Contoso",
              "accountOwnerEmail": "kenny@contoso.onmicrosoft.com",
              "costCenter": "Test",
              "isDevTest": false
            }
          }
        ],
        "hasReadAccess": false
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}

```

Значение для области выставления счетов и `id` — это одно и то же. `id` для учетной записи регистрации — это область выставления счетов, в рамках которой инициируется запрос на подписку. Знать идентификатор важно, так как это обязательный параметр, который будет использоваться далее при работе со статьей для создания подписки.

<!-- 
### [PowerShell](#tab/azure-powershell-getEnrollments)

we're still working on enabling PowerShell SDK for billing APIs. Check back soon.

-->


<!--
### [Azure CLI](#tab/azure-cli-getEnrollments)

we're still working on enabling CLI SDK for billing APIs. Check back soon.
-->

---

## <a name="create-subscriptions-under-a-specific-enrollment-account"></a>Создание подписок с помощью определенной учетной записи регистрации

В следующем примере в учетной записи регистрации, выбранной на предыдущем шаге, создается подписка с именем *Dev Team Subscription*. 

### <a name="rest"></a>[REST](#tab/rest-EA)

Вызовите API PUT, чтобы создать запрос или псевдоним для создания подписки.

```json
PUT  https://management.azure.com/providers/Microsoft.Subscription/aliases/sampleAlias?api-version=2020-09-01 
```

В тексте запроса в качестве `billingScope` укажите `id` одной из учетных записей `enrollmentAccounts`.

```json 
{
  "properties": {
        "billingScope": "/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321",
        "DisplayName": "Dev Team Subscription", //Subscription Display Name
        "Workload": "Production"
  }
}
```

Разрешены следующие значения для `Workload`: `Production` и `DevTest`.

#### <a name="response"></a>Ответ

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

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell-EA)

Чтобы установить последнюю версию модуля, содержащего командлет `New-AzSubscriptionAlias`, запустите `Install-Module Az.Subscription`. Сведения о том, как установить последнюю версию PowerShellGet, см. в статье о [получении модуля PowerShellGet](/powershell/scripting/gallery/installing-psget).

Воспользуйтесь следующей командой [New-AzSubscriptionAlias](/powershell/module/az.subscription/new-azsubscription), указав область выставления счетов `"/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321"`. 

```azurepowershell-interactive
New-AzSubscriptionAlias -AliasName "sampleAlias" -SubscriptionName "Dev Team Subscription" -BillingScope "/providers/Microsoft.Billing/BillingAccounts/1234567/enrollmentAccounts/7654321" -Workload 'Production"
```

Вы получите subscriptionId в ответе команды.

```azurepowershell
{
  "id": "/providers/Microsoft.Subscription/aliases/sampleAlias",
  "name": "sampleAlias",
  "type": "Microsoft.Subscription/aliases",
  "properties": {
    "provisioningState": "Succeeded",
    "subscriptionId": "4921139b-ef1e-4370-a331-dd2229f4f510"
  }
}
```

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli-EA)

Сначала установите это расширение с помощью команд `az extension add --name account` и `az extension add --name alias`.

Воспользуйтесь следующей командой [az account alias create](/cli/azure/ext/account/account/alias#ext_account_az_account_alias_create) и укажите `billing-scope` и `id` одной из учетных записей `enrollmentAccounts`. 

```azurecli-interactive
az account alias create --name "sampleAlias" --billing-scope "/providers/Microsoft.Billing/billingAccounts/1234567/enrollmentAccounts/654321" --display-name "Dev Team Subscription" --workload "Production"
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

---

## <a name="limitations-of-azure-enterprise-subscription-creation-api"></a>Ограничения API создания подписок Azure Enterprise

- С помощью этого API создаются только подписки Azure Enterprise.
- Для учетной записи регистрации действует ограничение в 2000 подписок. После этого дополнительные подписки для учетной записи можно создать только на портале Azure. Чтобы создать дополнительные подписки с помощью API, создайте другую учетную запись регистрации. Отмененные, удаленные и переданные подписки учитываются в ограничение в 2000 подписок.
- Пользователи, которые не являются владельцами учетной записи, но были добавлены в учетную запись регистрации с помощью Azure RBAC, не могут создавать подписки на портале Azure.
- Невозможно выбрать клиент для создания подписки. Подписка всегда создается в домашнем клиенте владельца учетной записи. Чтобы переместить подписку в другой клиент, ознакомьтесь с [изменением подписки клиента](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).


## <a name="next-steps"></a>Дальнейшие действия

* После создания подписки можно предоставить эту возможность для других пользователей и субъектов-служб. Дополнительные сведения см. статье [Предоставление доступа к созданию подписок Azure Enterprise (предварительная версия)](grant-access-to-create-subscription.md).
* Дополнительные сведения о переносе большого числа подписок с помощью групп управления см. в руководстве по [упорядочиванию ресурсов с помощью групп управления Azure](../../governance/management-groups/overview.md).
