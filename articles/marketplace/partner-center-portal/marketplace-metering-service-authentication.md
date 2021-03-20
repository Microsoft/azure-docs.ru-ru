---
title: Стратегии проверки подлинности службы контроля Marketplace | Azure Marketplace
description: Стратегии проверки подлинности службы контроля, поддерживаемые в Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/21/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: b418a9cae6f6d58dbe82babcfe6fe1e1a5027d43
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97657079"
---
# <a name="marketplace-metering-service-authentication-strategies"></a>Стратегии проверки подлинности службы контроля Marketplace

Служба контроля Marketplace поддерживает две стратегии проверки подлинности:

* [Маркер безопасности Azure AD](../../active-directory/develop/access-tokens.md).
* [Управляемые удостоверения](../../active-directory/managed-identities-azure-resources/overview.md) 

Мы объясним, когда и как использовать различные стратегии проверки подлинности для безопасной отправки настраиваемых счетчиков с помощью службы контроля Marketplace.

## <a name="using-the-azure-ad-security-token"></a>Использование маркера безопасности Azure AD

Применимые типы предложений — это SaaS и приложения Azure с типом плана управляемого приложения.  

Отправьте пользовательские счетчики, используя предопределенный фиксированный идентификатор приложения Azure AD для проверки подлинности.

Для предложений SaaS это единственный доступный вариант. Это обязательный шаг для публикации любого предложения SaaS, как описано в разделе [Регистрация приложения SaaS](./pc-saas-registration.md).

Для приложений Azure с планом управляемого приложения эту стратегию следует использовать в следующих случаях.

* У вас уже есть механизм взаимодействия с серверными службами и вы хотите расширить этот механизм для выдачи настраиваемых счетчиков из центральной службы.
* У вас есть сложная логика настраиваемых счетчиков.  Запустите эту логику в центральном расположении, а не на ресурсах управляемого приложения.

После регистрации приложения можно программным способом запросить маркер безопасности Azure AD. Ожидается, что издатель будет использовать этот токен и отправит запрос на его разрешение.

Дополнительные сведения об этих маркерах см. в статье [Маркеры доступа в Azure Active Directory](../../active-directory/develop/access-tokens.md).

### <a name="get-a-token-based-on-the-azure-ad-app"></a>Получение токена для приложения Azure AD

#### <a name="http-method"></a>Метод HTTP

**POST**

#### <a name="request-url"></a>*Request URL* (URL-адрес запроса)

**`https://login.microsoftonline.com/*{tenantId}*/oauth2/token`**

#### <a name="uri-parameter"></a>*Параметр универсального кода ресурса*

|  **Имя параметра** |  **Обязательно**  |  **Описание**          |
|  ------------------ |--------------- | ------------------------  |
|  `tenantId`         |   True         | Идентификатор клиента для зарегистрированного приложения Azure AD.   |
| | | |

#### <a name="request-header"></a>*Заголовок запроса*

|  **Имя заголовка**    |  **Обязательно**  |  **Описание**          |
|  ------------------ |--------------- | ------------------------  |
|  `Content-Type`     |   True         | Связанный с этим запросом тип содержимого. Значение по умолчанию — `application/x-www-form-urlencoded`.  |
| | | |

#### <a name="request-body"></a>*Текст запроса*

|  **Имя свойства**  |  **Обязательно**  |  **Описание**          |
|  ------------------ |--------------- | ------------------------  |
|  `Grant_type`       |   True         | Тип предоставления разрешения. Используйте команду `client_credentials`. |
|  `Client_id`        |   True         | Идентификатор клиента или приложения, связанный с приложением Azure AD.|
|  `client_secret`    |   True         | Секрет, связанный с приложением Azure AD.  |
|  `Resource`         |   True         | Целевой ресурс, для которого запрашивается токен. Используйте команду `20e940b3-4c77-4b0b-9a53-9e16a1b010a7`. |
| | | |

#### <a name="response"></a>*Ответ*

|  **имя**;    |  **Тип**  |  **Описание**          |
|  ------------------ |--------------- | ----------------------  |
|  `200 OK`     |   `TokenResponse`    | Запрос успешно выполнен.  |
| | | |

#### <a name="tokenresponse"></a>*TokenResponse*

Пример токена ответа.

```JSON
  {
      "token_type": "Bearer",
      "expires_in": "3600",
      "ext_expires_in": "0",
      "expires_on": "15251…",
      "not_before": "15251…",
      "resource": "20e940b3-4c77-4b0b-9a53-9e16a1b010a7",
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayIsImtpZCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayJ9…"
  }
```

## <a name="using-the-azure-managed-identities-token"></a>Использование маркера удостоверений, управляемых Azure

Применимый тип предложений: приложения Azure с планом управляемого приложения.

Это позволит использовать удостоверение развернутых ресурсов для проверки подлинности для отправки событий использования настраиваемых счетчиков.  Вы можете внедрить код, который порождает использование в пределах развертывания.

>[!Note]
>Издателю следует убедиться в том, что ресурсы, порождаемые использованием, заблокированы, чтобы их нельзя было изменить.

Управляемое приложение может содержать ресурсы различных типов — от виртуальных машин до Функций Azure.  Дополнительные сведения о проверке подлинности с помощью управляемых удостоверений для различных служб см. в статье [Использование управляемых удостоверений для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md#how-can-i-use-managed-identities-for-azure-resources).

Например, выполните указанные ниже действия для проверки подлинности с помощью виртуальной машины Windows.

1. Убедитесь, что управляемое удостоверение настроено с помощью одного из следующих методов:
    * [Пользовательский интерфейс портала Azure](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
    * [CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
    * [PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
    * [Шаблон Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
    * [Остальные](../../active-directory/managed-identities-azure-resources/qs-configure-rest-vm.md#system-assigned-managed-identity))
    * [Пакеты SDK для Azure](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

1. Получите маркер доступа для идентификатора приложения службы контроля Marketplace (`20e940b3-4c77-4b0b-9a53-9e16a1b010a7`), используя системное удостоверение, RDP для виртуальной машины, откройте консоль PowerShell и выполните следующую команду.

    ```powershell
    # curl is an alias to Web-Invoke PowerShell command
    # Get system identity access tokenn
    $MetadataUrl = "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F"
    $Token = curl -H @{"Metadata" = "true"} $MetadataUrl | Select-Object -Expand Content | ConvertFrom-Json
    $Headers = @{}
    $Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
    ```

1. Получить идентификатор управляемого приложения из свойства "ManagedBy" текущей группы ресурсов.

    ```powershell
    # Get subscription and resource group
    $metadata = curl -H @{'Metadata'='true'} http://169.254.169.254/metadata/instance?api-version=2019-06-01 | select -ExpandProperty Content | ConvertFrom-Json 

    # Make sure the system identity has at least reader permission on the resource group
    $managementUrl = "https://management.azure.com/subscriptions/" + $metadata.compute.subscriptionId + "/resourceGroups/" + $metadata.compute.resourceGroupName + "?api-version=2019-10-01"
    $resourceGroupInfo = curl -Headers $Headers $managementUrl | select -ExpandProperty Content | ConvertFrom-Json
    $managedappId = $resourceGroupInfo.managedBy 
    ```

1. Служба контроля Marketplace должна сообщать об использовании на `resourceID` и `resourceUsageId` в случае управляемого приложения.

    ```powershell
    # Get resourceUsageId from the managed app
    $managedAppUrl = "https://management.azure.com/subscriptions/" + $metadata.compute.subscriptionId + "/resourceGroups/" + $metadata.compute.resourceGroupName + "/providers/Microsoft.Solutions/applications/" + $managedappId + "\?api-version=2019-07-01"
    $ManagedApp = curl $managedAppUrl -H $Headers | Select-Object -Expand Content | ConvertFrom-Json
    # Use this resource ID to emit usage 
    $resourceUsageId = $ManagedApp.properties.billingDetails.resourceUsageId
    ```

1. Используйте [API службы контроля Marketplace](./marketplace-metering-service-apis.md) для порождения использования.

## <a name="next-steps"></a>Дальнейшие действия

* [Создание предложения приложения Azure](../create-new-azure-apps-offer.md)
* [Планирование предложения SaaS](../plan-saas-offer.md)
