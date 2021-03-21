---
title: Управляемые удостоверения
description: Узнайте, как управляемые удостоверения работают в Службе приложений Azure и Функциях Azure, как настроить управляемое удостоверение и создать маркер для внутреннего ресурса.
author: mattchenderson
ms.topic: article
ms.date: 05/27/2020
ms.author: mahender
ms.reviewer: yevbronsh
ms.custom: devx-track-csharp, devx-track-python, devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 16cd4685f513eb628372802cc158195b81bce72a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98736177"
---
# <a name="how-to-use-managed-identities-for-app-service-and-azure-functions"></a>Использование управляемых удостоверений в Службе приложений и Функциях Azure

В этой статье показано, как создавать управляемое удостоверение для приложений Службы приложений и Функций Azure, а также как их использовать для получения доступа к другим ресурсам. 

> [!Important] 
> При переносе приложения между подписками или клиентами управляемые удостоверения для Службы приложений и Функций Azure не будут работать в надлежащем режиме. Для приложения нужно получить новое удостоверение, отключив службу и включив ее снова. См. раздел [Удаление удостоверения](#remove) ниже. Также нужно обновить политики доступа для всех зависимых ресурсов, чтобы они использовали новое удостоверение.

[!INCLUDE [app-service-managed-identities](../../includes/app-service-managed-identities.md)]

## <a name="add-a-system-assigned-identity"></a>Добавление назначаемого системой удостоверения

Чтобы создать приложение с назначаемым системой удостоверением, необходимо задать дополнительное свойство в приложении.

### <a name="using-the-azure-portal"></a>Использование портала Azure

Чтобы настроить управляемое удостоверение на портале, сначала необходимо создать обычное приложение, а затем активировать соответствующую функцию.

1. Создайте приложение на портале обычным образом. Перейдите к нему на портале.

2. Если используется приложение-функция, перейдите к **функциям платформы**. Для других типов приложений прокрутите вниз до группы **параметров** в левой области навигации.

3. Выберите **Удостоверение**.

4. На вкладке **Назначено системой** для параметра **Состояние** установите значение **Вкл**. Выберите команду **Сохранить**.

    ![Снимок экрана, на котором показано, куда переключается состояние, а затем нажмите кнопку Сохранить.](media/app-service-managed-service-identity/system-assigned-managed-identity-in-azure-portal.png)


> [!NOTE] 
> Чтобы найти управляемое удостоверение для веб-приложения или приложения слота в портал Azure, в разделе **корпоративные приложения** просмотрите раздел **Параметры пользователя** . Обычно имя слота аналогично `<app name>/slots/<slot name>` .


### <a name="using-the-azure-cli"></a>Использование Azure CLI

Чтобы настроить управляемое удостоверение с помощью Azure CLI, используйте команду `az webapp identity assign` для имеющегося приложения. Примеры из этого раздела можно выполнять тремя способами:

- использовать [Azure Cloud Shell](../cloud-shell/overview.md) с портала Azure;
- использовать внедренный компонент Azure Cloud Shell с помощью кнопки "Попробовать", расположенной в правом верхнем углу каждого блока кода ниже.
- [установить последнюю версию Azure CLI](/cli/azure/install-azure-cli) (2.0.31 или выше), если вы предпочитаете использовать локальную консоль CLI. 

Ниже описаны действия по созданию веб-приложения и присвоению удостоверения ему с помощью CLI:

1. Если вы используете Azure CLI в локальной консоли, сначала выполните вход в Azure с помощью команды [az login](/cli/azure/reference-index#az-login). Используйте учетную запись, связанную с подпиской Azure, с помощью которой нужно развернуть приложение:

    ```azurecli-interactive
    az login
    ```

2. Создайте веб-приложение с помощью CLI. Дополнительные примеры использования CLI со службой приложений см. в статье [Примеры Azure CLI](../app-service/samples-cli.md).

    ```azurecli-interactive
    az group create --name myResourceGroup --location westus
    az appservice plan create --name myPlan --resource-group myResourceGroup --sku S1
    az webapp create --name myApp --resource-group myResourceGroup --plan myPlan
    ```

3. Выполните команду `identity assign`, чтобы создать удостоверение для этого приложения.

    ```azurecli-interactive
    az webapp identity assign --name myApp --resource-group myResourceGroup
    ```

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Следующие шаги помогут вам создать приложение и назначить ему удостоверение с помощью Azure PowerShell. Инструкции по созданию веб-приложения и приложения функции различаются.

#### <a name="using-azure-powershell-for-a-web-app"></a>Использование Azure PowerShell для веб-приложения

1. При необходимости установите Azure PowerShell с помощью инструкций из [руководства по Azure PowerShell](/powershell/azure/), а затем выполните команду `Login-AzAccount`, чтобы создать подключение к Azure.

2. Создайте веб-приложение с помощью Azure PowerShell. Дополнительные примеры применения Azure PowerShell со службой приложений см. в статье [Примеры сценариев Azure PowerShell](../app-service/samples-powershell.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create an App Service plan in Free tier.
    New-AzAppServicePlan -Name $webappname -Location $location -ResourceGroupName $resourceGroupName -Tier Free

    # Create a web app.
    New-AzWebApp -Name $webappname -Location $location -AppServicePlan $webappname -ResourceGroupName $resourceGroupName
    ```

3. Выполните команду `Set-AzWebApp -AssignIdentity`, чтобы создать удостоверение для этого приложения.

    ```azurepowershell-interactive
    Set-AzWebApp -AssignIdentity $true -Name $webappname -ResourceGroupName $resourceGroupName 
    ```

#### <a name="using-azure-powershell-for-a-function-app"></a>Использование Azure PowerShell для приложения-функции

1. При необходимости установите Azure PowerShell с помощью инструкций из [руководства по Azure PowerShell](/powershell/azure/), а затем выполните команду `Login-AzAccount`, чтобы создать подключение к Azure.

2. Создание приложения-функции с помощью Azure PowerShell. Дополнительные примеры использования Azure PowerShell с функциями Azure см. в [справочнике по AZ. functions](/powershell/module/az.functions/#functions):

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account.
    New-AzStorageAccount -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName $sku

    # Create a function app with a system-assigned identity.
    New-AzFunctionApp -Name $functionAppName -ResourceGroupName $resourceGroupName -Location $location -StorageAccountName $storageAccountName -Runtime $runtime -IdentityType SystemAssigned
    ```

Можно также обновить существующее приложение функции `Update-AzFunctionApp` , используя вместо этого.

### <a name="using-an-azure-resource-manager-template"></a>Использование шаблона Azure Resource Manager

Шаблон Azure Resource Manager можно использовать для автоматизации развертывания ресурсов Azure. Дополнительные сведения о развертывании в службе приложений и Функциях см. в статьях [Предсказуемые подготовка и развертывание микрослужб в Azure](../app-service/deploy-complex-application-predictably.md) и [Автоматизация развертывания ресурсов приложения-функции для службы "Функции Azure"](../azure-functions/functions-infrastructure-as-code.md).

Любой ресурс типа `Microsoft.Web/sites` можно создать с помощью удостоверения, добавив следующее свойство в определение ресурса:

```json
"identity": {
    "type": "SystemAssigned"
}
```

> [!NOTE]
> Приложение может иметь назначаемое системой и назначаемое пользователем удостоверения одновременно. В этом случае для свойства `type` будет задано значение `SystemAssigned,UserAssigned`.

При добавлении назначаемого системой удостоверения Azure создает удостоверение для приложения и управляет им.

Например, веб-приложение может выглядеть следующим образом:

```json
{
    "apiVersion": "2016-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "SystemAssigned"
    },
    "properties": {
        "name": "[variables('appName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "hostingEnvironment": "",
        "clientAffinityEnabled": false,
        "alwaysOn": true
    },
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
    ]
}
```

Созданный сайт будет иметь следующие дополнительные свойства:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

Свойство tenantId определяет, к какому клиенту Azure AD относится это удостоверение. principalId — это уникальный идентификатор для нового удостоверения приложения. В Azure AD субъект-служба имеет то же имя, которое вы присвоили экземпляру Службы приложений или Функций Azure.

Если необходимо сослаться на эти свойства на более позднем этапе шаблона, это можно сделать с помощью [ `reference()` функции шаблона](../azure-resource-manager/templates/template-functions-resource.md#reference) с `'Full'` флагом, как показано в следующем примере:

```json
{
    "tenantId": "[reference(resourceId('Microsoft.Web/sites', variables('appName')), '2018-02-01', 'Full').identity.tenantId]",
    "objectId": "[reference(resourceId('Microsoft.Web/sites', variables('appName')), '2018-02-01', 'Full').identity.principalId]",
}
```

## <a name="add-a-user-assigned-identity"></a>Добавление назначаемого пользователем удостоверения

Для создания приложения с назначаемым пользователем удостоверением необходимо создать удостоверение, а затем добавить его идентификатор ресурса в конфигурацию приложения.

### <a name="using-the-azure-portal"></a>Использование портала Azure

Сначала необходимо создать ресурс назначаемого пользователем удостоверения.

1. Создайте ресурс назначаемого пользователем управляемого удостоверения в соответствии с [этими инструкциями](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity).

2. Создайте приложение на портале обычным образом. Перейдите к нему на портале.

3. Если используется приложение-функция, перейдите к **функциям платформы**. Для других типов приложений прокрутите вниз до группы **параметров** в левой области навигации.

4. Выберите **Удостоверение**.

5. На вкладке **Назначаемое пользователем** щелкните **Добавить**.

6. Найдите созданное ранее удостоверение и выберите его. Нажмите кнопку **Добавить**.

    ![Управляемое удостоверение в Службе приложений](media/app-service-managed-service-identity/user-assigned-managed-identity-in-azure-portal.png)

### <a name="using-azure-powershell"></a>Использование Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Следующие шаги помогут вам создать приложение и назначить ему удостоверение с помощью Azure PowerShell.

> [!NOTE]
> Текущая версия Azure PowerShell командлеты для службы приложений Azure не поддерживает назначенные пользователю удостоверения. Приведенные ниже инструкции предназначены для функций Azure.

1. При необходимости установите Azure PowerShell с помощью инструкций из [руководства по Azure PowerShell](/powershell/azure/), а затем выполните команду `Login-AzAccount`, чтобы создать подключение к Azure.

2. Создание приложения-функции с помощью Azure PowerShell. Дополнительные примеры использования Azure PowerShell с функциями Azure см. в [справочнике по AZ. functions](/powershell/module/az.functions/#functions). В приведенном ниже скрипте также используется, `New-AzUserAssignedIdentity` который должен быть установлен отдельно при [создании, перечислении или удалении назначенного пользователем управляемого удостоверения с помощью Azure PowerShell](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a storage account.
    New-AzStorageAccount -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName $sku

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Create a function app with a user-assigned identity.
    New-AzFunctionApp -Name $functionAppName -ResourceGroupName $resourceGroupName -Location $location -StorageAccountName $storageAccountName -Runtime $runtime -IdentityType UserAssigned -IdentityId $userAssignedIdentity.Id
    ```

Можно также обновить существующее приложение функции `Update-AzFunctionApp` , используя вместо этого.

### <a name="using-an-azure-resource-manager-template"></a>Использование шаблона Azure Resource Manager

Шаблон Azure Resource Manager можно использовать для автоматизации развертывания ресурсов Azure. Дополнительные сведения о развертывании в службе приложений и Функциях см. в статьях [Предсказуемые подготовка и развертывание микрослужб в Azure](../app-service/deploy-complex-application-predictably.md) и [Автоматизация развертывания ресурсов приложения-функции для службы "Функции Azure"](../azure-functions/functions-infrastructure-as-code.md).

Любой ресурс типа `Microsoft.Web/sites` можно создать с помощью удостоверения, добавив следующий блок в определение ресурса, заменив `<RESOURCEID>` идентификатором ресурса необходимого удостоверения:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

> [!NOTE]
> Приложение может иметь назначаемое системой и назначаемое пользователем удостоверения одновременно. В этом случае для свойства `type` будет задано значение `SystemAssigned,UserAssigned`.

При добавлении назначаемого пользователем типа служба Azure будет использовать назначаемое пользователем удостоверение, указанное для вашего приложения.

Например, веб-приложение может выглядеть следующим образом:

```json
{
    "apiVersion": "2016-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('appName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
        }
    },
    "properties": {
        "name": "[variables('appName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "hostingEnvironment": "",
        "clientAffinityEnabled": false,
        "alwaysOn": true
    },
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
    ]
}
```

Созданный сайт будет иметь следующие дополнительные свойства:

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

principalId — это уникальный идентификатор удостоверения, используемый для администрирования Azure AD. clientId — это уникальный идентификатор нового удостоверения приложения, который используется для того, чтобы указать, какое удостоверение следует использовать во время вызовов среды выполнения.

## <a name="obtain-tokens-for-azure-resources"></a>Получение маркеров для ресурсов Azure

Приложение может использовать управляемое удостоверение для получения маркеров доступа к другим ресурсам, защищенным Azure AD (например, Azure Key Vault). Эти маркеры представляют приложение, получающее доступ к ресурсам, а не конкретного пользователя приложения. 

Чтобы разрешить доступ из приложения, необходимо настроить целевой ресурс. Например, если вы запрашиваете маркер для получения доступа к Key Vault, необходимо добавить политику доступа, включающую в себя удостоверение приложения. В противном случае вызовы Key Vault будут отклонены, даже если они включают маркеры. Чтобы узнать больше о том, какие ресурсы поддерживают маркеры Azure Active Directory, см. сведения в разделе о [службах Azure, поддерживающих аутентификацию Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

> [!IMPORTANT]
> Серверные службы для управляемых удостоверений поддерживают кэш для URI каждого ресурса около 24 часов. Если вы обновляете политику доступа для определенного целевого ресурса и сразу получаете маркер для этого ресурса, вы можете постоянно получать кэшированный маркер с устаревшими разрешениями до тех пор, пока не истечет срок действия этого маркера. В настоящее время невозможно принудительно обновить маркер.

Существует простой протокол REST для получения маркера в службе приложений и Функциях Azure. Его можно использовать для всех приложений и языков. Для .NET и Java пакет Azure SDK предоставляет абстракцию для работы с этим протоколом и упрощает процесс разработки в локальной среде.

### <a name="using-the-rest-protocol"></a>Использование протокола REST

> [!NOTE]
> В более старой версии этого протокола, поддерживающей версию API "2017-09-01", использовался заголовок `secret` вместо `X-IDENTITY-HEADER` и принималось только свойство `clientid` для назначаемого пользователем удостоверения. В ней также возвращался параметр `expires_on` в формате метки времени. MSI_ENDPOINT можно использовать в качестве псевдонима для IDENTITY_ENDPOINT, а MSI_SECRET — в качестве псевдонима для IDENTITY_HEADER. Эта версия протокола в настоящее время требуется для планов размещения потребления Linux.

Приложение с управляемым удостоверением содержит две заданные переменные среды:

- IDENTITY_ENDPOINT — URL-адрес локальной службы токенов.
- IDENTITY_HEADER — заголовок, который используется для противостояния атакам с подделкой серверных запросов (SSRF). Это значение меняется платформой.

**IDENTITY_ENDPOINT** — это локальный URL-адрес, из которого приложение может запрашивать маркеры. Чтобы получить маркер для ресурса, отправьте запрос HTTP GET к этой конечной точке, задав следующие параметры:

> | Имя параметра    | В     | Описание                                                                                                                                                                                                                                                                                                                                |
> |-------------------|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
> | ресурс          | Запрос  | Универсальный код ресурса (URI) Azure AD, для которого нужно получить маркер. Это может быть URI одной из [служб Azure, которая поддерживает аутентификацию Azure AD,](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication) или любой другой URI ресурса.    |
> | api-version       | Запрос  | Версия API маркеров, которая будет использоваться. Используйте "2019-08-01" или более позднюю версию (если не используется Linux, в настоящее время предлагается только "2017-09-01" — см. примечание выше).                                                                                                                                                                                                                                                                 |
> | X-IDENTITY-HEADER | Заголовок | Значение переменной среды IDENTITY_HEADER. Заголовок, который используется при устранении атак с подделкой серверных запросов (SSRF).                                                                                                                                                                                                    |
> | client_id         | Запрос  | (Необязательно.) Идентификатор клиента назначаемого пользователем удостоверения, которое следует использовать. Не может использоваться для запроса, который включает в себя идентификатор `principal_id`, `mi_res_id`или `object_id`. Если все параметры ИД (`client_id`, `principal_id`, `object_id` и `mi_res_id`) опущены, используется назначаемое системой удостоверение.                                             |
> | principal_id      | Запрос  | (Необязательно.) Идентификатор субъекта назначаемого пользователем удостоверения, которое следует использовать. `object_id` — псевдоним, который можно использовать вместо этого. Не может использоваться для запроса, который включает в себя идентификатор client_id, mi_res_id или object_id. Если все параметры ИД (`client_id`, `principal_id`, `object_id` и `mi_res_id`) опущены, используется назначаемое системой удостоверение. |
> | mi_res_id         | Запрос  | (Необязательно.) Идентификатор ресурса Azure для назначаемого пользователем удостоверения, которое следует использовать. Не может использоваться для запроса, который включает в себя идентификатор `principal_id`, `client_id`или `object_id`. Если все параметры ИД (`client_id`, `principal_id`, `object_id` и `mi_res_id`) опущены, используется назначаемое системой удостоверение.                                      |

> [!IMPORTANT]
> Если вы хотите получить маркеры для назначаемых пользователем удостоверений, необходимо указать одно из дополнительных свойств. В противном случае служба токенов попытается получить маркер для назначаемого системой удостоверения, которое может и отсутствовать.

Успешный ответ 200 — OK включает текст JSON со следующими свойствами:

> | Имя свойства | Описание                                                                                                                                                                                                                                        |
> |---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
> | access_token  | Запрашиваемый маркер доступа. Вызывающая веб-служба может использовать этот маркер для проверки подлинности принимающей веб-службы.                                                                                                                               |
> | client_id     | Идентификатор клиента используемого удостоверения.                                                                                                                                                                                                       |
> | expires_on    | Период времени после истечения срока действия маркера доступа. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC (соответствует утверждению `exp` маркера).                                                                                |
> | not_before    | Период времени, когда маркер доступа вступает в силу и может быть принят. Дата представляется как количество секунд с 1970-01-01T0:0:0Z в формате UTC (соответствует утверждению `nbf` маркера).                                                      |
> | ресурс      | Ресурс, для которого был запрошен маркер доступа, соответствует параметру строки запроса `resource`.                                                                                                                               |
> | token_type    | Указывает значение типа маркера. Единственный тип, поддерживаемый Azure AD — носитель. Дополнительные сведения о маркерах носителей см. в спецификации [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (OAuth2.0 Authorization Framework: использование маркера носителя (RFC 6750)). |

Этот ответ совпадает с [ответом для запроса маркера взаимного доступа между службами Azure AD](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md#service-to-service-access-token-response).

### <a name="rest-protocol-examples"></a>Примеры протокола REST

Пример запроса может выглядеть следующим образом:

```http
GET /MSI/token?resource=https://vault.azure.net&api-version=2019-08-01 HTTP/1.1
Host: localhost:4141
X-IDENTITY-HEADER: 853b9a84-5bfa-4b22-a3f3-0b9a43d9ad8a
```

Пример ответа может выглядеть следующим образом:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "access_token": "eyJ0eXAi…",
    "expires_on": "1586984735",
    "resource": "https://vault.azure.net",
    "token_type": "Bearer",
    "client_id": "5E29463D-71DA-4FE0-8E69-999B57DB23B0"
}
```

### <a name="code-examples"></a>Примеры кода

# <a name="net"></a>[.NET](#tab/dotnet)

> [!TIP]
> Для языков .NET можно также использовать пакет [Microsoft.Azure.Services.AppAuthentication](#asal), вместо того чтобы создавать этот запрос самостоятельно.

```csharp
private readonly HttpClient _client;
// ...
public async Task<HttpResponseMessage> GetToken(string resource)  {
    var request = new HttpRequestMessage(HttpMethod.Get, 
        String.Format("{0}/?resource={1}&api-version=2019-08-01", Environment.GetEnvironmentVariable("IDENTITY_ENDPOINT"), resource));
    request.Headers.Add("X-IDENTITY-HEADER", Environment.GetEnvironmentVariable("IDENTITY_HEADER"));
    return await _client.SendAsync(request);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const rp = require('request-promise');
const getToken = function(resource, cb) {
    let options = {
        uri: `${process.env["IDENTITY_ENDPOINT"]}/?resource=${resource}&api-version=2019-08-01`,
        headers: {
            'X-IDENTITY-HEADER': process.env["IDENTITY_HEADER"]
        }
    };
    rp(options)
        .then(cb);
}
```

# <a name="python"></a>[Python](#tab/python)

```python
import os
import requests

identity_endpoint = os.environ["IDENTITY_ENDPOINT"]
identity_header = os.environ["IDENTITY_HEADER"]

def get_bearer_token(resource_uri):
    token_auth_uri = f"{identity_endpoint}?resource={resource_uri}&api-version=2019-08-01"
    head_msi = {'X-IDENTITY-HEADER':identity_header}

    resp = requests.get(token_auth_uri, headers=head_msi)
    access_token = resp.json()['access_token']

    return access_token
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
$resourceURI = "https://<AAD-resource-URI-for-resource-to-obtain-token>"
$tokenAuthURI = $env:IDENTITY_ENDPOINT + "?resource=$resourceURI&api-version=2019-08-01"
$tokenResponse = Invoke-RestMethod -Method Get -Headers @{"X-IDENTITY-HEADER"="$env:IDENTITY_HEADER"} -Uri $tokenAuthURI
$accessToken = $tokenResponse.access_token
```

---

### <a name="using-the-microsoftazureservicesappauthentication-library-for-net"></a><a name="asal"></a>Использование библиотеки Microsoft.Azure.Services.AppAuthentication для .NET

Самый простой способ для приложений и функций .NET работать с управляемыми удостоверениями заключается в использовании пакета Microsoft.Azure.Services.AppAuthentication. Эта библиотека также позволяет локально тестировать код на компьютере разработки с использованием учетной записи пользователя из Visual Studio, [Azure CLI](/cli/azure) или встроенной проверки подлинности Active Directory. При размещении в облаке по умолчанию будет использоваться назначенное системой удостоверение, но это поведение можно изменить с помощью переменной среды строки подключения, которая ссылается на идентификатор клиента для назначенного пользователю удостоверения. Дополнительные сведения о вариантах разработки с этой библиотекой см. в [справочнике по Microsoft. Azure. Services. AppAuthentication]. В этом разделе показано, как начать работу с библиотекой в коде.

1. Добавьте ссылки на пакет [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication) и другие пакеты NuGet в приложение. В примерах ниже также используется [Microsoft.Azure.KeyVault](https://www.nuget.org/packages/Microsoft.Azure.KeyVault).

2. Добавьте приведенный ниже код в приложение, указав необходимый целевой ресурс. В этом примере показано два способа работы с Azure Key Vault:

    ```csharp
    using Microsoft.Azure.Services.AppAuthentication;
    using Microsoft.Azure.KeyVault;
    // ...
    var azureServiceTokenProvider = new AzureServiceTokenProvider();
    string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://vault.azure.net");
    // OR
    var kv = new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));
    ```

Если вы хотите использовать управляемое пользователем удостоверение, можно задать `AzureServicesAuthConnectionString` для параметра приложения значение `RunAs=App;AppId=<clientId-guid>` . Замените `<clientId-guid>` идентификатором клиента идентификатора, который вы хотите использовать. Можно определить несколько таких строк подключения с помощью пользовательских параметров приложения и передать их значения в конструктор AzureServiceTokenProvider.

```csharp
    var identityConnectionString1 = Environment.GetEnvironmentVariable("UA1_ConnectionString");
    var azureServiceTokenProvider1 = new AzureServiceTokenProvider(identityConnectionString1);
    
    var identityConnectionString2 = Environment.GetEnvironmentVariable("UA2_ConnectionString");
    var azureServiceTokenProvider2 = new AzureServiceTokenProvider(identityConnectionString2);
```

Дополнительные сведения о настройке AzureServiceTokenProvider и предоставляемых ею операциях см. в [справочнике по Microsoft. Azure. Services. AppAuthentication] и [службе приложений и KEYVAULT с примером MSI .NET](https://github.com/Azure-Samples/app-service-msi-keyvault-dotnet).

### <a name="using-the-azure-sdk-for-java"></a>Использование пакета Azure SDK для Java

Для приложений и функций Java при работе с управляемым удостоверением проще всего использовать пакет [Azure SDK для Java](https://github.com/Azure/azure-sdk-for-java). В этом разделе показано, как начать работу с библиотекой в коде.

1. Добавьте ссылку на [библиотеку Azure SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure). В проектах Maven можно добавить в раздел `dependencies` файла POM вашего проекта следующий фрагмент кода:

    ```xml
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure</artifactId>
        <version>1.23.0</version>
    </dependency>
    ```

2. Для проверки подлинности используйте объект `AppServiceMSICredentials`. Следующий пример демонстрирует, как можно использовать этот механизм для работы с Azure Key Vault:

    ```java
    import com.microsoft.azure.AzureEnvironment;
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.keyvault.Vault
    //...
    Azure azure = Azure.authenticate(new AppServiceMSICredentials(AzureEnvironment.AZURE))
            .withSubscription(subscriptionId);
    Vault myKeyVault = azure.vaults().getByResourceGroup(resourceGroup, keyvaultName);

    ```


## <a name="remove-an-identity"></a><a name="remove"></a>Удаление удостоверения

Чтобы удалить назначаемое системой удостоверение, нужно отключить возможность с помощью портала, PowerShell или интерфейса командной строки точно так же, как и при его создании. Назначаемые пользователем удостоверения можно удалить по отдельности. Чтобы удалить все удостоверения, задайте для параметра Тип удостоверения значение "нет".

Если назначаемое системой удостоверение удаляется таким способом, то оно также удаляется и из Azure AD. Назначаемые системой удостоверения также автоматически удаляются из Azure AD, когда удаляется ресурс приложения.

Чтобы удалить все удостоверения в [шаблоне ARM](#using-an-azure-resource-manager-template), выполните следующие действия.

```json
"identity": {
    "type": "None"
}
```

Чтобы удалить все удостоверения в Azure PowerShell (только для функций Azure):

```azurepowershell-interactive
# Update an existing function app to have IdentityType "None".
Update-AzFunctionApp -Name $functionAppName -ResourceGroupName $resourceGroupName -IdentityType None
```

> [!NOTE]
> Также вы можете установить параметр приложения WEBSITE_DISABLE_MSI, который отключает локальную службу маркеров. Но при этом само удостоверение сохранится, а управляемое удостоверение будет отображаться как "включенное". По этой причине мы рекомендуем не использовать такой параметр.

## <a name="next-steps"></a>Дальнейшие действия

- [Безопасное подключение Базы данных Azure SQL с использованием управляемого удостоверения службы](app-service-web-tutorial-connect-msi.md)
- [Безопасное получение доступа к службе хранилища Azure с помощью управляемого удостоверения](scenario-secure-app-access-storage.md)
- [Безопасный вызов Microsoft Graph с помощью управляемого удостоверения](scenario-secure-app-access-microsoft-graph-as-app.md)

[Справочник Microsoft.Azure.Services.AppAuthentication]: /dotnet/api/overview/azure/service-to-service-authentication