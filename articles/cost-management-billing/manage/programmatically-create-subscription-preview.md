---
title: Программное создание подписок Azure с помощью предварительных версий API
description: Узнайте, как создавать дополнительные подписки Azure программным способом с помощью предварительных версий REST API, Azure CLI и Azure PowerShell.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 11/17/2020
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: b524869998dd2464ed359ec61ce655a807899aaa
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102565722"
---
# <a name="programmatically-create-azure-subscriptions-with-preview-apis"></a>Программное создание подписок Azure с помощью предварительных версий API

Эта статья поможет программно создавать подписки Azure с помощью уже устаревшей предварительной версии нашего API. В этой статье Вы узнаете как создавать подписки программно с помощью Azure Resource Manager.

Наши новые статьи о последних версиях API для использования с разными типами подписок для соглашений Azure:

- [Создание подписок EA программными средствами с помощью API последней версии](programmatically-create-subscription-enterprise-agreement.md)
- [Создание подписок MCA программными средствами с помощью API последней версии](programmatically-create-subscription-microsoft-customer-agreement.md)
- [Создание подписок MPA программными средствами с помощью API последней версии](Programmatically-create-subscription-microsoft-customer-agreement.md)

Но эта статья все равно будет полезна, если вы не хотите использовать последнюю версию API.

Создавать подписки программным способом могут клиенты Azure с учетной записью выставления счетов для следующих типов соглашений:

- Соглашение Enterprise
- Клиентское соглашение Майкрософт (MCA);
- Соглашение с партнером Майкрософт (ПСМ)

Когда вы создаете подписки Azure программными средствами, они регулируются соглашением, в соответствии с которым вы получили службы Azure от корпорации Майкрософт или уполномоченного торгового посредника. Дополнительные сведения можно найти на странице [Юридическая информация Службы Microsoft Azure](https://azure.microsoft.com/support/legal/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="create-subscriptions-for-an-ea-billing-account"></a>Создание подписок для учетной записи выставления счетов по EA

Из следующих разделов вы узнаете, как создать подписки с EA.

### <a name="prerequisites"></a>Предварительные требования

Для создания подписки необходимо иметь роль владельца учетной записи регистрации. Ее можно получить двумя способами.

* Администратор регистрации, отвечающий за вашу регистрацию, [может назначить роль владельца учетной записи](https://ea.azure.com/helpdocs/addNewAccount) (требуется вход в систему), что сделает вас владельцем учетной записи регистрации.
* Текущий владелец учетной записи регистрации может [предоставить вам права доступа](grant-access-to-create-subscription.md). Аналогично, чтобы использовать субъект-службу для создания подписки с EA, необходимо [предоставить ему возможность создания подписки](grant-access-to-create-subscription.md).

### <a name="find-accounts-you-have-access-to"></a>Поиск учетных записей, к которым у вас есть доступ

После того как вы будете добавлены в учетную запись регистрации, связанную с владельцем учетной записи, платформа Azure использует связь между регистрацией и учетной записью, чтобы определить целевой объект для выставления счетов за подписки. Все подписки, созданные с учетной записью, оплачиваются в рамках регистрации EA, к которой относится эта учетная запись. Чтобы создать подписки, необходимо в значениях передать о регистрации учетной записи и субъектов-пользователя — владельца подписки.

Чтобы выполнить следующие команды, войдите в *домашний каталог* владельца учетной записи, в котором по умолчанию создаются подписки.

### <a name="rest"></a>[REST](#tab/rest)

Запросите вывод списка всех учетных записей регистрации, к которым у вас есть доступ.

```json
GET https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts?api-version=2018-03-01-preview
```

В ответе API будет выведен список всех учетных записей регистрации, к которым у вас есть доступ.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "SignUpEngineering@contoso.com"
      }
    },
    {
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Billing/enrollmentAccounts",
      "properties": {
        "principalName": "BillingPlatformTeam@contoso.com"
      }
    }
  ]
}
```

Используйте свойство `principalName`, чтобы указать учетную запись для оплаты подписок. Скопируйте значение `name` этой учетной записи. Например, чтобы создать подписки в учетной записи регистрации SignUpEngineering@contoso.com, скопируйте ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Этот идентификатор — идентификатор объекта учетной записи регистрации. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге как `enrollmentAccountObjectId`.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Откройте [Azure Cloud Shell](https://shell.azure.com/) и выберите PowerShell.

Используйте командлет [Get-AzEnrollmentAccount](/powershell/module/az.billing/get-azenrollmentaccount), чтобы вывести список всех учетных записей регистрации, к которым у вас есть доступ.

```azurepowershell-interactive
Get-AzEnrollmentAccount
```

Платформа Azure выведет список учетных записей регистрации, к которым у вас есть доступ.

```azurepowershell
ObjectId                               | PrincipalName
747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | SignUpEngineering@contoso.com
4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | BillingPlatformTeam@contoso.com
```
Используйте свойство `principalName`, чтобы указать учетную запись для оплаты подписок. Скопируйте значение `ObjectId` этой учетной записи. Например, чтобы создать подписки в учетной записи регистрации SignUpEngineering@contoso.com, скопируйте ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Вставьте этот идентификатор объекта в любое место, чтобы его можно было использовать на следующем шаге в качестве `enrollmentAccountObjectId`.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Используйте [команду Get-EnrollmentAccount](/cli/azure/billing), чтобы вывести список всех учетных записей регистрации, к которым у вас есть доступ.

```azurecli-interactive
az billing enrollment-account list
```

Платформа Azure выведет список учетных записей регистрации, к которым у вас есть доступ.

```json
[
  {
    "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "principalName": "SignUpEngineering@contoso.com",
    "type": "Microsoft.Billing/enrollmentAccounts",
  },
  {
    "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "principalName": "BillingPlatformTeam@contoso.com",
    "type": "Microsoft.Billing/enrollmentAccounts",
  }
]
```

Используйте свойство `principalName`, чтобы указать учетную запись для оплаты подписок. Скопируйте значение `name` этой учетной записи. Например, чтобы создать подписки в учетной записи регистрации SignUpEngineering@contoso.com, скопируйте ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Этот идентификатор — идентификатор объекта учетной записи регистрации. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге как `enrollmentAccountObjectId`.

---

### <a name="create-subscriptions-under-a-specific-enrollment-account"></a>Создание подписок с помощью определенной учетной записи регистрации

В следующем примере в учетной записи регистрации, выбранной на предыдущем шаге, создается подписка с именем *Dev Team Subscription*. Предложение подписки — *MS-AZR-0017P* (обычное Соглашение Enterprise с корпорацией Майкрософт). Этот пример также при необходимости добавляет двух пользователей в качестве владельцев Azure RBAC для подписки.

### <a name="rest"></a>[REST](#tab/rest)

Выполните следующий запрос, заменив `<enrollmentAccountObjectId>` на `name`, скопированный на первом шаге (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Сведения об указании владельцев см. в разделе [Получение идентификаторов объектов пользователей](grant-access-to-create-subscription.md#userObjectId).

```json
POST https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>/providers/Microsoft.Subscription/createSubscription?api-version=2018-03-01-preview

{
  "displayName": "Dev Team Subscription",
  "offerType": "MS-AZR-0017P",
  "owners": [
    {
      "objectId": "<userObjectId>"
    },
    {
      "objectId": "<servicePrincipalObjectId>"
    }
  ]
}
```

| Имя элемента  | Обязательно | Тип   | Описание                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | нет      | Строка | Отображаемое имя подписки. Если нет — ему присваивается имя предложения, например "Microsoft Azure Enterprise".                                 |
| `offerType`   | Да      | Строка | Предложение подписки. Доступны два варианта предложения EA: [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (для использования в рабочей среде) и [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (для разработки и тестирования, должен быть [включен с помощью портала EA](https://ea.azure.com/helpdocs/DevOrTestOffer)).                |
| `owners`      | нет       | Строка | Идентификатор объекта любого пользователя, который добавляется в качестве владельца Azure RBAC для подписки при ее создании.  |

В ответе в составе заголовка `Location` вы получаете URL-адрес, по которому можно запросить состояние операции создания подписки. После завершения создания подписки в ответ на запрос GET для URL-адреса `Location` вы получите объект `subscriptionLink` с идентификатором подписки. Дополнительные сведения см. в [документации по API подписки](/rest/api/subscription/).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы установить последнюю версию модуля, содержащего командлет `New-AzSubscription`, запустите `Install-Module Az.Subscription`. Сведения о том, как установить последнюю версию PowerShellGet, см. в статье о [получении модуля PowerShellGet](/powershell/scripting/gallery/installing-psget).

Воспользуйтесь приведенной ниже командой [New-AzSubscription](/powershell/module/az.subscription), заменив `<enrollmentAccountObjectId>` на значение `ObjectId`, полученное на первом шаге (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Сведения об указании владельцев см. в разделе [Получение идентификаторов объектов пользователей](grant-access-to-create-subscription.md#userObjectId).

```azurepowershell-interactive
New-AzSubscription -OfferType MS-AZR-0017P -Name "Dev Team Subscription" -EnrollmentAccountObjectId <enrollmentAccountObjectId> -OwnerObjectId <userObjectId1>,<servicePrincipalObjectId>
```

| Имя элемента  | Обязательно | Тип   | Описание |
|---------------|----------|--------|----|
| `Name` | нет      | Строка | Отображаемое имя подписки. Если нет — ему присваивается имя предложения, например *Microsoft Azure Enterprise*. |
| `OfferType`   | Да      | Строка | Предложение подписки. Доступны два варианта предложения EA: [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (для использования в рабочей среде) и [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (для разработки и тестирования, должен быть [включен с помощью портала EA](https://ea.azure.com/helpdocs/DevOrTestOffer)).                |
| `EnrollmentAccountObjectId`      | Да       | Строка | Идентификатор объекта учетной записи для регистрации, в которой создается подписка и для которой выставляются счета. Это значение GUID, полученное из `Get-AzEnrollmentAccount`. |
| `OwnerObjectId`      | нет       | Строка | Идентификатор объекта любого пользователя, который добавляется в качестве владельца Azure RBAC для подписки при ее создании.  |
| `OwnerSignInName`    | нет       | Строка | Адрес электронной почты любого пользователя, который добавляется в качестве владельца Azure RBAC для подписки при ее создании. Можно использовать этот параметр вместо `OwnerObjectId`.|
| `OwnerApplicationId` | нет       | Строка | Идентификатор приложения любого субъекта-службы, который добавляется в качестве владельца Azure RBAC для подписки при ее создании. Можно использовать этот параметр вместо `OwnerObjectId`. При использовании этого параметра служба должна иметь [доступ на чтение к каталогу](/powershell/azure/active-directory/signing-in-service-principal#give-the-service-principal-reader-access-to-the-current-tenant-get-azureaddirectoryrole).|

Полный список параметров приведен в разделе о [New-AzSubscription](/powershell/module/az.subscription/New-AzSubscription).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала установите эту предварительную версию расширения, запустив `az extension add --name subscription`.

Воспользуйтесь приведенной ниже командой [az account create](/cli/azure/ext/subscription/account#-ext-subscription-az-account-create), заменив `<enrollmentAccountObjectId>` на значение `name`, скопированное на первом шаге (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Сведения об указании владельцев см. в разделе [Получение идентификаторов объектов пользователей](grant-access-to-create-subscription.md#userObjectId).

```azurecli-interactive
az account create --offer-type "MS-AZR-0017P" --display-name "Dev Team Subscription" --enrollment-account-object-id "<enrollmentAccountObjectId>" --owner-object-id "<userObjectId>","<servicePrincipalObjectId>"
```

| Имя элемента  | Обязательно | Тип   | Описание |
|---------------|----------|--------|------------|
| `display-name` | нет      | Строка | Отображаемое имя подписки. Если нет — ему присваивается имя предложения, например *Microsoft Azure Enterprise*.|
| `offer-type`   | Да      | Строка | Предложение подписки. Доступны два варианта предложения EA: [MS-AZR-0017P](https://azure.microsoft.com/pricing/enterprise-agreement/) (для использования в рабочей среде) и [MS-AZR-0148P](https://azure.microsoft.com/offers/ms-azr-0148p/) (для разработки и тестирования, должен быть [включен с помощью портала EA](https://ea.azure.com/helpdocs/DevOrTestOffer)).                |
| `enrollment-account-object-id`      | Да       | Строка | Идентификатор объекта учетной записи для регистрации, в которой создается подписка и для которой выставляются счета. Это значение GUID, полученное из `az billing enrollment-account list`. |
| `owner-object-id`      | нет       | Строка | Идентификатор объекта любого пользователя, который добавляется в качестве владельца Azure RBAC для подписки при ее создании.  |
| `owner-upn`    | нет       | Строка | Адрес электронной почты любого пользователя, который добавляется в качестве владельца Azure RBAC для подписки при ее создании. Можно использовать этот параметр вместо `owner-object-id`.|
| `owner-spn` | нет       | Строка | Идентификатор приложения любого субъекта-службы, который добавляется в качестве владельца Azure RBAC для подписки при ее создании. Можно использовать этот параметр вместо `owner-object-id`. При использовании этого параметра служба должна иметь [доступ на чтение к каталогу](/powershell/azure/active-directory/signing-in-service-principal#give-the-service-principal-reader-access-to-the-current-tenant-get-azureaddirectoryrole).|

Полный список параметров приведен в разделе о команде [az account create](/cli/azure/ext/subscription/account#-ext-subscription-az-account-create).

---

### <a name="limitations-of-azure-enterprise-subscription-creation-api"></a>Ограничения API создания подписок Azure Enterprise

- С помощью этого API могут быть созданы только подписки Azure Enterprise.
- Для учетной записи регистрации действует ограничение в 2000 подписок. После этого дополнительные подписки для учетной записи можно создать только на портале Azure. Если вы хотите создать больше подписок с помощью API, создайте другую учетную запись регистрации. Отмененные, удаленные и переданные подписки учитываются в ограничение в 2000 подписок.
- Пользователи, которые не являются владельцами учетной записи, но были добавлены в учетную запись регистрации с помощью Azure RBAC, не могут создавать подписки на портале Azure.
- Невозможно выбрать клиент для создания подписки. Подписка всегда создается в домашнем клиенте владельца учетной записи. Чтобы переместить подписку в другой клиент, ознакомьтесь с [изменением подписки клиента](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).


## <a name="create-subscriptions-for-an-mca-account"></a>Создание подписок для учетной записи MCA

Используйте сведения в следующих разделах для создания подписок для учетной записи MCA.

### <a name="prerequisites"></a>Предварительные требования

Для создания подписок необходимо иметь роль владельца, участника или создателя подписок Azure в разделе счета либо роль владельца или участника в профиле выставления счетов или учетной записи выставления счетов. Дополнительные сведения см. в статье [Роли и задачи выставления счетов в подписке](understand-mca-roles.md#subscription-billing-roles-and-tasks).

В следующих примерах используются интерфейсы REST API. В настоящее время PowerShell и Azure CLI не поддерживаются.

### <a name="find-billing-accounts-that-you-have-access-to"></a>Поиск учетных записей выставления счетов, к которым у вас есть доступ

Отправьте следующий запрос, чтобы получить список всех учетных записей выставления счетов.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?api-version=2019-10-01-preview
```
В ответе API будет выведен список учетных записей выставления счетов, к которым у вас есть доступ.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "displayName": "Contoso",
        "hasReadAccess": true,
        "organizationId": "41b29574-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "displayName": "Fabrikam",
        "hasReadAccess": true,
        "organizationId": "41b29574-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}

```
Используйте свойство `displayName`, чтобы определить учетную запись выставления счетов, для которой нужно создать подписки. Убедитесь, что значение agreementType для учетной записи равно *MicrosoftCustomerAgreement*. Скопируйте значение `name` учетной записи.  Например, чтобы создать подписку для учетной записи выставления счетов `Contoso`, скопируйте `5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx`. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

### <a name="find-invoice-sections-to-create-subscriptions"></a>Поиск разделов счетов для создания подписок

Плата за подписку отображается в этом разделе счета профиля выставления счетов. Используйте следующий API для получения списка разделов счетов и профилей выставления счетов, для которых у вас есть разрешения на создание подписок Azure.

Выполните следующий запрос, заменив `<billingAccountName>` на `name`, скопированный на первом шаге (```5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx```).

```json
POST https://management.azure.com/providers/Microsoft.Billing/billingAccounts/<billingAccountName>/listInvoiceSectionsWithCreateSubscriptionPermission?api-version=2019-10-01-preview
```

В ответе API будут перечислены все разделы счетов и их профили выставления счетов, которые доступны вам для создания подписок.

```json
{
    "value": [{
        "billingProfileDisplayName": "Contoso finance",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx",
        "enabledAzurePlans": [{
            "productId": "DZH318Z0BPS6",
            "skuId": "0001",
            "skuDescription": "Microsoft Azure Plan"
        }, {
            "productId": "DZH318Z0BPS6",
            "skuId": "0002",
            "skuDescription": "Microsoft Azure Plan for DevTest"
        }],
        "invoiceSectionDisplayName": "Development",
        "invoiceSectionId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/invoiceSections/GJ77-xxxx-xxx-xxx"
    }, {
        "billingProfileDisplayName": "Contoso finance",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx",
        "enabledAzurePlans": [{
            "productId": "DZH318Z0BPS6",
            "skuId": "0001",
            "skuDescription": "Microsoft Azure Plan"
        }, {
            "productId": "DZH318Z0BPS6",
            "skuId": "0002",
            "skuDescription": "Microsoft Azure Plan for DevTest"
        }],
        "invoiceSectionDisplayName": "Testing",
        "invoiceSectionId": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-XXXX-XXX-XXX/invoiceSections/GJGR-XXXX-XXX-XXX"
  }]
}

```

Используйте свойство `invoiceSectionDisplayName`, чтобы определить раздел счета, для которого нужно создать подписки. Скопируйте `invoiceSectionId`, `billingProfileId` и одно из значений `skuId` для раздела счета. Например, чтобы создать подписку типа `Microsoft Azure plan` для раздела `Development` счетов, скопируйте `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_2019-05-31/billingProfiles/PBFV-XXXX-XXX-XXX/invoiceSections/GJGR-XXXX-XXX-XXX`, `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_2019-05-31/billingProfiles/PBFV-xxxx-xxx-xxx` и `0001`. Вставьте эти значения в любое место, чтобы их можно было использовать на следующем шаге.

### <a name="create-a-subscription-for-an-invoice-section"></a>Создание подписки для раздела счета

В следующем примере создается подписка с именем *Dev Team subscription* типа *План Microsoft Azure* для раздела счета *Development*. Счет за подписку будет выставлен для профиля выставления счетов *Contoso finance's*, а сама подписка будет указана в разделе *Development* этого счета.

Выполните следующий запрос, заменив идентификатор `<invoiceSectionId>` на `invoiceSectionId`, скопированный на втором шаге (```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_2019-05-31/billingProfiles/PBFV-XXXX-XXX-XXX/invoiceSections/GJGR-XXXX-XXX-XXX```). В параметрах запроса API необходимо передать `billingProfileId` и `skuId`, скопированные на втором шаге. Сведения об указании владельцев см. в разделе [Получение идентификаторов объектов пользователей](grant-access-to-create-subscription.md#userObjectId).

```json
POST https://management.azure.com<invoiceSectionId>/providers/Microsoft.Subscription/createSubscription?api-version=2018-11-01-preview
```

```json
'{"displayName": "Dev Team subscription",
  "billingProfileId": "<billingProfileId>",
  "skuId": "<skuId>",
  "owners": [
      {
        "objectId": "<userObjectId>"
      },
      {
        "objectId": "<servicePrincipalObjectId>"
      }
    ],
  "costCenter": "35683",
  "managementGroupId": "/providers/Microsoft.Management/managementGroups/xxxxxxx",",
}'

```

| Имя элемента  | Обязательно | Тип   | Описание                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | Да      | Строка | Отображаемое имя подписки.|
| `billingProfileId`   | Да      | Строка | Идентификатор профиля выставления счетов, для которого будет взиматься плата за использование подписки.  |
| `skuId` | Да      | Строка | Идентификатор SKU, определяющий тип плана Azure. |
| `owners`      | нет       | Строка | Идентификатор объекта любого пользователя или субъекта-службы, который добавляется в качестве владельца Azure RBAC для подписки при ее создании.  |
| `costCenter` | нет      | Строка | Место возникновения затрат, связанное с подпиской. Он отображается в CSV-файле потребления. |
| `managementGroupId` | нет      | Строка | Идентификатор группы управления, в которую будет добавлена подписка. Сведения о том, как получить список групп управления, см. в статье [Группы управления: вывод списка API](/rest/api/resources/managementgroups/list). Используйте идентификатор группы управления из API. |

В ответе возвращается объект `subscriptionCreationResult` для отслеживания. После завершения создания подписки объект `subscriptionCreationResult` вернет объект `subscriptionLink` с идентификатором подписки.

## <a name="create-subscriptions-for-an-mpa-billing-account"></a>Создание подписок для учетной записи выставления счетов по MPA

Используйте сведения в следующих разделах для создания подписок в учетной записи выставления счетов MPA.

### <a name="prerequisites"></a>Предварительные требования

Чтобы создать подписку для учетной записи выставления счетов, необходимо иметь роль глобального администратора или агента администратора в учетной записи поставщика облачных решений вашей организации. Дополнительные сведения см. в статье [Назначение ролей и разрешений пользователям компании для работы в Центре партнеров](/partner-center/permissions-overview).

В следующих примерах используются интерфейсы REST API. В настоящее время PowerShell и Azure CLI не поддерживаются.

### <a name="find-the-billing-accounts-that-you-have-access-to"></a>Поиск учетных записей выставления счетов, к которым у вас есть доступ

Отправьте указанный ниже запрос, чтобы получить список всех учетных записей выставления счетов, к которым у вас есть доступ.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?api-version=2019-10-01-preview
```

В ответе API будут перечислены учетные записи выставления счетов.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftPartnerAgreement",
        "displayName": "Contoso",
        "hasReadAccess": true,
        "organizationId": "1d100e69-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "4f89e155-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "displayName": "Fabrikam",
        "hasReadAccess": true,
        "organizationId": "1d100e69-xxxx-xxxx-xxxx-xxxxxxxxxxxxx_xxxx-xx-xx"
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}

```
Используйте свойство `displayName`, чтобы определить учетную запись выставления счетов, для которой нужно создать подписки. Убедитесь, что значение agreementType для учетной записи равно *MicrosoftPartnerAgreement*. Скопируйте `name` для учетной записи. Например, чтобы создать подписку для учетной записи выставления счетов `Contoso`, скопируйте `99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx`. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

### <a name="find-customers-that-have-azure-plans"></a>Поиск клиентов, имеющих планы Azure

Выполните следующий запрос, заменив `<billingAccountName>` на значение `name`, скопированное на первом шаге (```5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx```), чтобы получить список всех клиентов в учетной записи выставления счетов, для которых вы можете создавать подписки Azure.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/<billingAccountName>/customers?api-version=2019-10-01-preview
```
В ответе API будут перечислены клиенты в учетной записи выставления счетов с планами Azure. Вы можете создать подписки для этих клиентов.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "properties": {
        "billingProfileDisplayName": "Contoso USD",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/JUT6-xxxx-xxxx-xxxx",
        "displayName": "Fabrikam toys"
      },
      "type": "Microsoft.Billing/billingAccounts/customers"
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/97c3fac4-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "97c3fac4-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "properties": {
        "billingProfileDisplayName": "Fabrikam sports",
        "billingProfileId": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/JUT6-xxxx-xxxx-xxxx",
        "displayName": "Fabrikam bakery"
      },
      "type": "Microsoft.Billing/billingAccounts/customers"
    }]
}

```

Используйте свойство `displayName`, чтобы определить клиентов, для которых нужно создать подписки. Скопируйте значение `id` для клиента. Например, чтобы создать подписку для `Fabrikam toys`, скопируйте `/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx`. Вставьте это значение в любое место, чтобы использовать его для последующих шагов.

### <a name="optional-for-indirect-providers-get-the-resellers-for-a-customer"></a>Необязательный этап для косвенных поставщиков: получение списка торговых посредников для клиента

Непрямой поставщик в двухуровневой модели CSP может указать торгового посредника при создании подписок для клиентов.

Отправьте следующий запрос, заменив `<customerId>` на значение `id`, скопированное на втором шаге (```/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx```), чтобы получить список всех торговых посредников, доступных для клиента.

```json
GET https://management.azure.com<customerId>?$expand=resellers&api-version=2019-10-01-preview
```
В ответе API будут перечислены торговые посредники для клиента.

```json
{
  "value": [{
  "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "2ed2c490-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Billing/billingAccounts/customers",
  "properties": {
    "displayName": "Fabrikam toys",
    "resellers": [
      {
        "resellerId": "3xxxxx",
        "description": "Wingtip"
      }
    ]
  }
},
{
  "id": "/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/4ed2c793-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "4ed2c793-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Billing/billingAccounts/customers",
  "properties": {
    "displayName": "Fabrikam toys",
    "resellers": [
      {
        "resellerId": "5xxxxx",
        "description": "Tailspin"
      }
    ]
  }
}]
}
```

Используйте свойство `description`, чтобы определить торгового посредника, который связан с подпиской. Скопируйте значение `resellerId` для торгового посредника. Например, чтобы связать `Wingtip`, скопируйте `3xxxxx`. Вставьте это значение в любое место, чтобы его можно было использовать на следующем шаге.

### <a name="create-a-subscription-for-a-customer"></a>Создание подписки для клиента

В следующем примере создается подписка с именем *Dev Team subscription* для *Fabrikam toys*, которая затем связывается с торговым посредником *Wingtip*. 

Выполните следующий запрос, заменив идентификатор `<customerId>` на `id`, скопированный на втором шаге (```/providers/Microsoft.Billing/billingAccounts/99a13315-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/customers/2281f543-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Передайте в параметрах запроса API необязательное значение *resellerId*, скопированное на втором шаге.

```json
POST https://management.azure.com<customerId>/providers/Microsoft.Subscription/createSubscription?api-version=2018-11-01-preview
```

```json
'{"displayName": "Dev Team subscription",
  "skuId": "0001",
  "resellerId": "<resellerId>",
}'
```

| Имя элемента  | Обязательно | Тип   | Описание                                                                                               |
|---------------|----------|--------|-----------------------------------------------------------------------------------------------------------|
| `displayName` | Да      | Строка | Отображаемое имя подписки.|
| `skuId` | Да      | Строка | Идентификатор SKU плана Azure. Используйте *0001* для подписок типа "План Microsoft Azure". |
| `resellerId`      | нет       | Строка | Идентификатор MPN торгового посредника, который будет связан с подпиской.  |

В ответе возвращается объект `subscriptionCreationResult` для отслеживания. По завершении создания подписки объект `subscriptionCreationResult` возвращает объект `subscriptionLink`. Он содержит идентификатор подписки.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [примером кода на сайте GitHub](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core), демонстрирующим создание подписок с Соглашением Enterprise (EA) с помощью .NET.
* После создания подписки можно предоставить эту возможность для других пользователей и субъектов-служб. Дополнительные сведения см. статье [Предоставление доступа к созданию подписок Azure Enterprise (предварительная версия)](grant-access-to-create-subscription.md).
* Дополнительные сведения о переносе большого числа подписок с помощью групп управления см. в руководстве по [упорядочиванию ресурсов с помощью групп управления Azure](../../governance/management-groups/overview.md).
