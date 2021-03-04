---
author: erhopf
ms.author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 05/11/2020
ms.openlocfilehash: fcb4113a4dab1e3de17eb022b1ad386cbc6a9583
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102109041"
---
## <a name="authenticate-with-azure-active-directory"></a>Проверка подлинности с помощью Azure Active Directory

> [!IMPORTANT]
> 1. Сейчас **только** API компьютерного зрения, API распознавания лиц, API анализа текста, иммерсивное средство чтения, распознаватель форм, детектор аномалий, QnA Maker и все службы Bing, кроме пользовательский поиск Bing, поддерживают проверку подлинности с помощью Azure Active Directory (AAD).
> 2. Проверку подлинности AAD необходимо всегда использовать вместе с именем пользовательского поддомена ресурса Azure. [Региональные конечные точки](../articles/cognitive-services/cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) не поддерживают проверку подлинности AAD.

В предыдущих разделах мы показали, как выполнять аутентификацию в Azure Cognitive Services с помощью ключа подписки с одной или несколькими службами. Хотя эти ключи предоставляют быстрый и простой путь для начала разработки, они немного сложны в более сложных сценариях, требующих управления доступом на основе ролей Azure (Azure RBAC). Давайте посмотрим, что требуется для проверки подлинности с помощью Azure Active Directory (AAD).

В следующих разделах вы будете использовать Azure Cloud Shell среду или Azure CLI для создания поддомена, назначения ролей и получения токена носителя для вызова Cognitive Services Azure. В случае зависания в каждом разделе содержатся ссылки со всеми доступными параметрами для каждой команды в Azure Cloud Shell и Azure CLI.

### <a name="create-a-resource-with-a-custom-subdomain"></a>Создание ресурса с пользовательским поддоменом

Первым шагом является создание пользовательского поддомена. Если вы хотите использовать существующий ресурс Cognitive Services без имени пользовательского поддомена, следуйте инструкциям в разделе [Cognitive Services пользовательские](../articles/cognitive-services/cognitive-services-custom-subdomains.md#how-does-this-impact-existing-resources) поддомены, чтобы включить пользовательский поддомен для ресурса.

1. Для начала откройте Azure Cloud Shell. Затем [выберите подписку](/powershell/module/az.accounts/set-azcontext?view=azps-3.3.0):

   ```powershell-interactive
   Set-AzContext -SubscriptionName <SubscriptionName>
   ```

2. Затем [создайте Cognitive Services ресурс](/powershell/module/az.cognitiveservices/new-azcognitiveservicesaccount?view=azps-1.8.0) с пользовательским поддоменом. Имя поддомена должно быть глобально уникальным и не может содержать специальные символы, например: ".", "!", ",".

   ```powershell-interactive
   $account = New-AzCognitiveServicesAccount -ResourceGroupName <RESOURCE_GROUP_NAME> -name <ACCOUNT_NAME> -Type <ACCOUNT_TYPE> -SkuName <SUBSCRIPTION_TYPE> -Location <REGION> -CustomSubdomainName <UNIQUE_SUBDOMAIN>
   ```

3. В случае успеха **Конечная точка** должна отображать имя поддомена, уникальное для ресурса.


### <a name="assign-a-role-to-a-service-principal"></a>Назначение роли субъекту-службе.

Теперь, когда у вас есть пользовательский поддомен, связанный с ресурсом, необходимо назначить роль субъекту-службе.

> [!NOTE]
> Помните, что для распространения назначений ролей Azure может потребоваться до пяти минут.

1. Сначала выполним регистрацию [приложения AAD](/powershell/module/Az.Resources/New-AzADApplication?view=azps-1.8.0).

   ```powershell-interactive
   $SecureStringPassword = ConvertTo-SecureString -String <YOUR_PASSWORD> -AsPlainText -Force

   $app = New-AzADApplication -DisplayName <APP_DISPLAY_NAME> -IdentifierUris <APP_URIS> -Password $SecureStringPassword
   ```

   Вам потребуется идентификатор **приложения на следующем** шаге.

2. Далее необходимо [создать субъект-службу](/powershell/module/az.resources/new-azadserviceprincipal?view=azps-1.8.0) для приложения AAD.

   ```powershell-interactive
   New-AzADServicePrincipal -ApplicationId <APPLICATION_ID>
   ```

   >[!NOTE]
   > При регистрации приложения в портал Azure этот шаг завершается.

3. Последним шагом является [назначение роли "Cognitive Services пользователя"](/powershell/module/az.Resources/New-azRoleAssignment?view=azps-1.8.0) субъекту-службе (областью действия ресурса). Назначая роль, вы предоставляете субъекту-службе доступ к этому ресурсу. Вы можете предоставить одному субъекту-службе доступ к нескольким ресурсам в подписке.
   >[!NOTE]
   > Используется ObjectId субъекта-службы, а не ObjectId для приложения.
   > ACCOUNT_ID будет идентификатором ресурса Azure созданной учетной записи Cognitive Services. Идентификатор ресурса Azure можно найти в "свойствах" ресурса в портал Azure.

   ```azurecli-interactive
   New-AzRoleAssignment -ObjectId <SERVICE_PRINCIPAL_OBJECTID> -Scope <ACCOUNT_ID> -RoleDefinitionName "Cognitive Services User"
   ```

### <a name="sample-request"></a>Пример запроса

В этом примере для проверки подлинности субъекта-службы используется пароль. Затем предоставленный токен используется для вызова API компьютерного зрения.

1. Получите **ИД** клиента:
   ```powershell-interactive
   $context=Get-AzContext
   $context.Tenant.Id
   ```

2. Получение маркера:
   > [!NOTE]
   > Если вы используете Azure Cloud Shell, `SecureClientSecret` класс недоступен. 

   #### <a name="powershell"></a>[PowerShell](#tab/powershell)
   ```powershell-interactive
   $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList "https://login.windows.net/<TENANT_ID>"
   $secureSecretObject = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.SecureClientSecret" -ArgumentList $SecureStringPassword   
   $clientCredential = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $app.ApplicationId, $secureSecretObject
   $token=$authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", $clientCredential).Result
   $token
   ```
   
   #### <a name="azure-cloud-shell"></a>[Azure Cloud Shell](#tab/azure-cloud-shell);
   ```Azure Cloud Shell
   $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList "https://login.windows.net/<TENANT_ID>"
   $clientCredential = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential" -ArgumentList $app.ApplicationId, <YOUR_PASSWORD>
   $token=$authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", $clientCredential).Result
   $token
   ``` 

   ---

3. Вызовите API компьютерного зрения:
   ```powershell-interactive
   $url = $account.Endpoint+"vision/v1.0/models"
   $result = Invoke-RestMethod -Uri $url  -Method Get -Headers @{"Authorization"=$token.CreateAuthorizationHeader()} -Verbose
   $result | ConvertTo-Json
   ```

Кроме того, можно проверить подлинность субъекта-службы с помощью сертификата. Кроме субъекта-службы, субъект пользователя также поддерживается путем делегирования разрешений через другое приложение AAD. В этом случае вместо паролей или сертификатов пользователям будет предложено использовать двухфакторную проверку подлинности при получении маркера.

## <a name="authorize-access-to-managed-identities"></a>Авторизация доступа к управляемым удостоверениям
 
Cognitive Services поддерживает проверку подлинности Azure Active Directory (Azure AD) с помощью [управляемых удостоверений для ресурсов Azure](../articles/active-directory/managed-identities-azure-resources/overview.md). Управляемые удостоверения для ресурсов Azure могут авторизовать доступ к ресурсам Cognitive Services с помощью учетных данных Azure AD из приложений, работающих на виртуальных машинах Azure, приложений-функций, масштабируемых наборов виртуальных машин и других служб. Используя управляемые удостоверения для ресурсов Azure вместе с проверкой подлинности Azure AD, можно избежать хранения учетных данных в приложениях, выполняемых в облаке.  

### <a name="enable-managed-identities-on-a-vm"></a>Включение управляемых удостоверений на виртуальной машине

Прежде чем использовать управляемые удостоверения для ресурсов Azure для авторизации доступа к ресурсам Cognitive Services из виртуальной машины, необходимо включить управляемые удостоверения для ресурсов Azure на виртуальной машине. Сведения о том, как включить управляемые удостоверения для ресурсов Azure, см. в следующих статьях:

- [Портал Azure](../articles/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../articles/active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../articles/active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Шаблон Azure Resource Manager](../articles/active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Клиентские библиотеки Azure Resource Manager](../articles/active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

Дополнительные сведения об управляемых удостоверениях см. в статье [управляемые удостоверения для ресурсов Azure](../articles/active-directory/managed-identities-azure-resources/overview.md).