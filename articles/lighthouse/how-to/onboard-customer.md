---
title: Подключение клиента к Azure Lighthouse
description: Узнайте, как подключить клиента к Azure Лигхсаусе, предоставив доступ к ресурсам и управление ими с помощью собственного клиента, используя делегированное управление ресурсами Azure.
ms.date: 02/16/2021
ms.topic: how-to
ms.openlocfilehash: 4487dd82b30e14f9db2001dc10f7437a53e745f3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100556101"
---
# <a name="onboard-a-customer-to-azure-lighthouse"></a>Подключение клиента к Azure Lighthouse

В этой статье объясняется, как поставщик услуг может подключить клиента к Azure Лигхсаусе. При этом делегированные ресурсы (подписки и группы ресурсов) в клиенте Azure Active Directory Azure AD можно управлять через свой клиент с помощью [управления делегированными ресурсами Azure](../concepts/azure-delegated-resource-management.md).

> [!TIP]
> Хотя в этом разделе мы будем называть поставщиков услуг и клиентов, [предприятия, управляющие несколькими клиентами](../concepts/enterprise.md) , могут использовать один и тот же процесс для настройки Azure лигхсаусе и консолидации их возможностей управления.

Процесс адаптации можно повторить для нескольких клиентов. Когда пользователь с соответствующими разрешениями выполняет вход в управляющий клиент, этот пользователь может быть разрешен в областях клиентской аренды для выполнения операций управления без необходимости входа в каждый клиент клиента.

Чтобы отслеживать свое влияние на разных клиентов и получать дополнительное признание, свяжите свой идентификатор Microsoft Partner Network (MPN) хотя бы с одной учетной записью пользователя, которая имеет доступ к каждой из подключенных подписок. Вам потребуется выполнить эту связь в клиенте поставщика услуг. Рекомендуется создать в клиенте учетную запись субъекта-службы, связанную с ИДЕНТИФИКАТОРом MPN, а затем включать этот субъект-службу при каждом подключении клиента. Дополнительные сведения см. [в статье связывание идентификатора партнера, чтобы включить получение партнерского кредита на делегированные ресурсы](partner-earned-credit.md).

> [!NOTE]
> Клиенты могут также подключаться к Azure Лигхсаусе при покупке предложения управляемой службы (открытого или частного), которое [публикуется в Azure Marketplace](publish-managed-services-offers.md). Вы также можете использовать процесс адаптации, описанный здесь, вместе с предложениями, опубликованными в Azure Marketplace.

Для процесса подключения требуется, чтобы действия выполнялись в клиенте поставщика услуг и клиенте пользователя. Все эти шаги описаны в данной статье.

## <a name="gather-tenant-and-subscription-details"></a>Сбор сведений о клиентах и подписках

Чтобы подключить клиент пользователя, у него должна быть активная подписка Azure. Для этого потребуется следующее:

- идентификатор клиента поставщика услуг (где вы будете управлять ресурсами клиента);
- идентификатор пользовательского клиента (в котором будут ресурсы, управляемые поставщиком услуг);
- идентификаторы подписки для каждой конкретной подписки в арендаторе клиента, которой будет управлять поставщик услуг (или которая содержит группы ресурсов, управляемые поставщиком услуг).

Если вы не знаете значения этого идентификатора, их можно получить любым из следующих способов. Обязательно укажите точные значения для развертывания.

### <a name="azure-portal"></a>Портал Azure

Идентификатор клиента можно увидеть, наведя указатель мыши на имя учетной записи в правом верхнем углу портала Azure либо выбрав **Переключение каталога**. Чтобы выбрать и скопировать идентификатор клиента, выполните поиск по запросу "Azure Active Directory" на портале, затем выберите параметр **Свойства** и скопируйте значение, отображаемое в поле **Идентификатор каталога**. Чтобы найти идентификатор подписки в арендаторе клиента, выполните поиск по запросу "Подписки", а затем выберите идентификатор соответствующей подписки.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

Select-AzSubscription <subscriptionId>
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az account set --subscription <subscriptionId/name>
az account show
```

> [!NOTE]
> При подключении подписки (или одной или нескольких групп ресурсов в подписке) с помощью описанного здесь способа поставщик ресурсов **Microsoft. ManagedServices** будет зарегистрирован для этой подписки.

## <a name="define-roles-and-permissions"></a>Определение ролей и разрешений

Как поставщик услуг вы можете выполнять для одного клиента несколько задач, требующих разных прав доступа для нескольких областей. Вы можете определить столько авторизаций, сколько требуется для назначения соответствующих [встроенных ролей Azure](../../role-based-access-control/built-in-roles.md). Каждая авторизация включает в себя **principalId** , который ссылается на пользователя, группу или субъект-службу Azure AD в управляющем клиенте.

> [!NOTE]
> Если явно не указано иное, ссылки на "пользователь" в документации по Azure Лигхсаусе могут применяться к пользователю, группе или субъекту-службе Azure AD в авторизации.

Чтобы упростить управление, рекомендуется использовать группы пользователей Azure AD для каждой роли, если это возможно, а не для отдельных пользователей. Это дает возможность добавлять или удалять отдельных пользователей в группе, имеющей доступ, чтобы не повторять процесс адаптации для внесения изменений в пользователя. Можно также назначить роли субъекту-службе, который может быть полезен для сценариев автоматизации.

> [!IMPORTANT]
> Чтобы добавить разрешения для группы Azure AD, **Тип группы** должен быть установлен в значение **Безопасность**. Этот вариант автоматически выбирается при создании группы. Дополнительные сведения см. в статье [Создание простой группы и добавление в нее участников с помощью Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

При определении авторизаций обязательно следуйте принципу минимальных привилегий, чтобы пользователи имели только необходимые разрешения для выполнения их работы. Сведения о поддерживаемых ролях и рекомендациях см. в статье о [клиентах, пользователях и ролях в сценариях Azure лигхсаусе](../concepts/tenants-users-roles.md).

Чтобы определить разрешения, необходимо знать значения ИДЕНТИФИКАТОРов для каждого пользователя, группы пользователей или субъекта-службы в клиенте поставщика услуг, которому требуется предоставить доступ. Вам также потребуется идентификатор определения роли для каждой встроенной роли, которую вы хотите назначить. Если у вас их еще нет, выполните следующие команды в арендаторе поставщика услуг.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# To retrieve the objectId for an Azure AD group
(Get-AzADGroup -DisplayName '<yourGroupName>').id

# To retrieve the objectId for an Azure AD user
(Get-AzADUser -UserPrincipalName '<yourUPN>').id

# To retrieve the objectId for an SPN
(Get-AzADApplication -DisplayName '<appDisplayName>' | Get-AzADServicePrincipal).Id

# To retrieve role definition IDs
(Get-AzRoleDefinition -Name '<roleName>').id
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# To retrieve the objectId for an Azure AD group
az ad group list --query "[?displayName == '<yourGroupName>'].objectId" --output tsv

# To retrieve the objectId for an Azure AD user
az ad user show --id "<yourUPN>" --query "objectId" --output tsv

# To retrieve the objectId for an SPN
az ad sp list --query "[?displayName == '<spDisplayName>'].objectId" --output tsv

# To retrieve role definition IDs
az role definition list --name "<roleName>" | grep name
```

> [!TIP]
> Рекомендуется назначать роль [Managed Services Registration Assignment Delete](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) (Удаление назначенных регистраций управляемых служб) при подключении клиента, чтобы пользователи в арендаторе позже могли при необходимости [удалить доступ к делегированию](remove-delegation.md). Если эта роль не назначена, делегированные ресурсы могут быть удалены только пользователем в арендаторе клиента.

## <a name="create-an-azure-resource-manager-template"></a>Создание шаблона Azure Resource Manager

Чтобы подключить клиента, необходимо создать шаблон [Azure Resource Manager](../../azure-resource-manager/index.yml) вашего предложения, который содержит приведенные ниже сведения. Значения **мспоффернаме** и **мспоффердескриптион** будут отображаться для клиента на [странице поставщики услуг](view-manage-service-providers.md) портал Azure.

|Поле  |Определение  |
|---------|---------|
|**mspOfferName**     |Имя, описывающее это определение. Это значение отображается для клиента в качестве заголовка предложения и должно быть уникальным значением.        |
|**mspOfferDescription**     |Краткое описание предложения (например, "Предложение для управления виртуальной машиной Contoso").      |
|**managedByTenantId**     |Идентификатор клиента.          |
|**Авторизации**     |Значения **principalId** для пользователей, групп и имен субъектов-служб из арендатора с параметром **principalIdDisplayName**, чтобы клиент мог понять назначение авторизации, сопоставленные со встроенным значением **roleDefinitionId**, чтобы определить уровень доступа      |

Для подключения требуется шаблон Azure Resource Manager (доступен в [репозитории примеров](https://github.com/Azure/Azure-Lighthouse-samples/)) и соответствующий файл параметров, который вы изменяете в соответствии с конфигурацией и в котором определяете параметры авторизации.

> [!IMPORTANT]
> Описанный здесь процесс требует отдельного развертывания для каждой подписки, даже если вы планируете подписку в одном клиенте клиента. Требуются также отдельные развертывания, если вы подключаете несколько групп ресурсов в разных подписках в одном арендаторе клиента. Тем не менее подключение нескольких групп ресурсов в рамках одной подписки может быть выполнено за одно развертывание.
>
> Для нескольких предложений, применяемых к одной подписке (или к группам ресурсов в рамках подписки), также необходимы отдельные развертывания. Каждое применимое предложение должно использовать разные значения **mspOfferName**.

Выбора шаблона будет зависеть от того, подключаете ли вы всю подписку, группу ресурсов или несколько групп ресурсов в пределах подписки. Мы также предоставляем шаблон для клиентов, которые приобрели предложение управляемых служб, опубликованное в Azure Marketplace, если вы предпочитаете таким образом подключать свои подписки.

|Чтобы подключить  |Используйте этот шаблон Azure Resource Manager  |И измените этот файл параметров |
|---------|---------|---------|
|Подписка   |[delegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management/delegatedResourceManagement.json)  |[delegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/delegated-resource-management/delegatedResourceManagement.parameters.json)    |
|Группа ресурсов   |[rgDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/rgDelegatedResourceManagement.json)  |[rgDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/rgDelegatedResourceManagement.parameters.json)    |
|Несколько групп ресурсов в рамках подписки   |[multipleRgDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/multipleRgDelegatedResourceManagement.json)  |[multipleRgDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/rg-delegated-resource-management/multipleRgDelegatedResourceManagement.parameters.json)    |
|Подписка (при использовании предложения, опубликованного в Azure Marketplace)   |[marketplaceDelegatedResourceManagement.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/marketplace-delegated-resource-management/marketplaceDelegatedResourceManagement.json)  |[marketplaceDelegatedResourceManagement.parameters.json](https://github.com/Azure/Azure-Lighthouse-samples/blob/master/templates/marketplace-delegated-resource-management/marketplaceDelegatedResourceManagement.parameters.json)    |

> [!TIP]
> Хотя вы не можете подключить всю группу управления в одном развертывании, можно [развернуть политику на уровне группы управления](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-delegate-management-groups). Политика проверяет, делегирована ли каждая подписка в группе управления указанному управляющему клиенту, и, если нет, создаст назначение на основе предоставленных значений.

В следующем примере показан измененный файл **delegatedResourceManagement.parameters.json**, который можно использовать для подключения подписки. Файлы параметров группы ресурсов (расположенные в папке [rg-delegated-resource-management](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/rg-delegated-resource-management)) аналогичны, но также содержат параметр **rgName** для определения конкретных групп ресурсов, которые должны быть подключены.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "mspOfferName": {
            "value": "Fabrikam Managed Services - Interstellar"
        },
        "mspOfferDescription": {
            "value": "Fabrikam Managed Services - Interstellar"
        },
        "managedByTenantId": {
            "value": "df4602a3-920c-435f-98c4-49ff031b9ef6"
        },
        "authorizations": {
            "value": [
                {
                    "principalId": "0019bcfb-6d35-48c1-a491-a701cf73b419",
                    "principalIdDisplayName": "Tier 1 Support",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "0019bcfb-6d35-48c1-a491-a701cf73b419",
                    "principalIdDisplayName": "Tier 1 Support",
                    "roleDefinitionId": "36243c78-bf99-498c-9df9-86d9f8d28608"
                },
                {
                    "principalId": "0afd8497-7bff-4873-a7ff-b19a6b7b332c",
                    "principalIdDisplayName": "Tier 2 Support",
                    "roleDefinitionId": "acdd72a7-3385-48ef-bd42-f606fba81ae7"
                },
                {
                    "principalId": "9fe47fff-5655-4779-b726-2cf02b07c7c7",
                    "principalIdDisplayName": "Service Automation Account",
                    "roleDefinitionId": "b24988ac-6180-42a0-ab88-20f7382dd24c"
                },
                {
                    "principalId": "3kl47fff-5655-4779-b726-2cf02b05c7c4",
                    "principalIdDisplayName": "Policy Automation Account",
                    "roleDefinitionId": "18d7d88d-d35e-4fb5-a5c3-7773c20a72d9",
                    "delegatedRoleDefinitionIds": [
                        "b24988ac-6180-42a0-ab88-20f7382dd24c",
                        "92aaf0da-9dab-42b6-94a3-d43ce8d16293"
                    ]
                }
            ]
        }
    }
}
```

Последняя авторизация в приведенном выше примере добавляет параметр **principalId** с ролью "Администратор доступа пользователя" (18d7d88d-d35e-4fb5-a5c3-7773c20a72d9). При назначении этой роли необходимо включить свойство **делегатедроледефинитионидс** и одну или несколько поддерживаемых встроенных ролей Azure. Пользователь, созданный в этой авторизации, сможет назначить эти роли [управляемым удостоверениям](../../active-directory/managed-identities-azure-resources/overview.md) в клиенте клиента, который необходим для [развертывания политик, которые можно исправить](deploy-policy-remediation.md).  Пользователь также может создавать обращения в службу поддержки. Никакие другие разрешения, обычно связанные с ролью администратора доступа пользователей, будут применены к этому **principalId**.

## <a name="deploy-the-azure-resource-manager-templates"></a>Развертывание шаблонов Azure Resource Manager

После обновления файла параметров пользователь в клиенте клиента должен развернуть шаблон Azure Resource Manager в своем клиенте. Для каждой подписки, которую необходимо подключить (или для каждой подписки, содержащей группы ресурсов, которые необходимо подключить), требуется отдельное развертывание.

> [!IMPORTANT]
> Это развертывание должно выполняться учетной записью, отличной от гостевой, в клиенте клиента, имеющей роль с `Microsoft.Authorization/roleAssignments/write` разрешением (например, [владелец](../../role-based-access-control/built-in-roles.md#owner)) для подключенной подписки (или содержащей группы ресурсов, для которых выполняется подключение). Чтобы найти пользователей, которые могут делегировать подписку, пользователь в клиенте клиента может выбрать подписку в портал Azure, открыть **Управление доступом (IAM)** и [Просмотреть всех пользователей с ролью владельца](../../role-based-access-control/role-assignments-list-portal.md#list-owners-of-a-subscription). 
>
> Если подписка была создана в рамках [программы "Поставщик облачных решений" (CSP)](../concepts/cloud-solution-provider.md), развертывание может выполнить любой пользователь с ролью [агента администратора](/partner-center/permissions-overview#manage-commercial-transactions-in-partner-center-azure-ad-and-csp-roles) в арендаторе поставщика услуг.

Развертывание можно выполнить в портал Azure, с помощью PowerShell или с помощью Azure CLI, как показано ниже.

### <a name="azure-portal"></a>Портал Azure

1. В нашем [репозитории GitHub](https://github.com/Azure/Azure-Lighthouse-samples/)нажмите кнопку **развернуть в Azure** , показанную рядом с шаблоном, который вы хотите использовать. Шаблон откроется на портале Azure.
1. Введите значения в параметре **имя предложения MSP**, **Описание предложения MSP**, **Управление по идентификатору клиента** и **авторизацию**. При желании можно выбрать параметр **изменить параметры** , чтобы ввести значения для `mspOfferName` , `mspOfferDescription` , `managedbyTenantId` и `authorizations` непосредственно в файл параметров. Не забудьте обновить эти значения, а не использовать значения по умолчанию из шаблона.
1. Выберите **проверить и создать**, а затем щелкните **создать**.

Через несколько минут вы увидите уведомление о завершении развертывания.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# Deploy Azure Resource Manager template using template and parameter file locally
New-AzSubscriptionDeployment -Name <deploymentName> `
                 -Location <AzureRegion> `
                 -TemplateFile <pathToTemplateFile> `
                 -TemplateParameterFile <pathToParameterFile> `
                 -Verbose

# Deploy Azure Resource Manager template that is located externally
New-AzSubscriptionDeployment -Name <deploymentName> `
                 -Location <AzureRegion> `
                 -TemplateUri <templateUri> `
                 -TemplateParameterUri <parameterUri> `
                 -Verbose
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# Deploy Azure Resource Manager template using template and parameter file locally
az deployment sub create --name <deploymentName> \
                         --location <AzureRegion> \
                         --template-file <pathToTemplateFile> \
                         --parameters <parameters/parameterFile> \
                         --verbose

# Deploy external Azure Resource Manager template, with local parameter file
az deployment sub create --name <deploymentName> \
                         --location <AzureRegion> \
                         --template-uri <templateUri> \
                         --parameters <parameterFile> \
                         --verbose
```

## <a name="confirm-successful-onboarding"></a>Подтверждение успешного подключения

Если подписка клиента успешно подключена к Azure Лигхсаусе, пользователи в клиенте поставщика услуг смогут просматривать подписку и ее ресурсы (если им предоставлен доступ к ней через описанный выше процесс, по отдельности или как член группы Azure AD с соответствующими разрешениями). Чтобы подтвердить это, убедитесь, что подписка отображается одним из следующих способов.  

### <a name="azure-portal"></a>Портал Azure

В клиенте поставщика услуг выполните приведенные ниже действия.

1. Перейдите на страницу [Мои клиенты](view-manage-customers.md).
2. Выберите **Клиенты**.
3. Убедитесь, что вы видите подписки с именем предложения, которое вы указали в шаблоне Resource Manager.

> [!IMPORTANT]
> Чтобы просмотреть делегированную подписку в разделе " [Мои клиенты](view-manage-customers.md)", пользователям в клиенте поставщика услуг должна быть предоставлена роль [читателя](../../role-based-access-control/built-in-roles.md#reader) (или другая встроенная роль, которая включает доступ для чтения) при подключении подписки.

В клиенте пользователя выполните следующее.

1. Перейдите на страницу [Поставщики услуг](view-manage-service-providers.md).
2. Выберите **Предложения поставщика услуг**.
3. Убедитесь, что вы видите подписки с именем предложения, которое вы указали в шаблоне Resource Manager.

> [!NOTE]
> После завершения развертывания может пройти до 15 минут, прежде чем обновления будут отражены в портал Azure. Вы можете увидеть обновления быстрее, если обновить токен Azure Resource Manager путем обновления браузера, входа и выхода или запроса нового маркера.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

Get-AzContext

# Confirm successful onboarding for Azure Lighthouse

Get-AzManagedServicesDefinition
Get-AzManagedServicesAssignment
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az account list
```

Если необходимо внести изменения после подключения клиента, можно [Обновить делегирование](update-delegation.md). Вы также можете полностью [отменить доступ к делегированию](remove-delegation.md) .

## <a name="troubleshooting"></a>Устранение неполадок

Если вы не можете успешно подключить клиента или у пользователей возникли проблемы с делегированными ресурсами, ознакомьтесь со следующими советами и требованиями и повторите попытку.

- `managedbyTenantId`Значение не должно совпадать с идентификатором клиента для подписки.
- Нельзя использовать несколько назначений в одной и той же области `mspOfferName` .
- Поставщик ресурсов **Microsoft. ManagedServices** должен быть зарегистрирован для делегированной подписки. Это должно происходить автоматически во время развертывания, но если нет, [его можно зарегистрировать вручную](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).
- Авторизация не должна включать пользователей с встроенной ролью [владельца](../../role-based-access-control/built-in-roles.md#owner) или встроенными ролями с [действиями](../../role-based-access-control/role-definitions.md#dataactions)с данными.
- Группы должны быть созданы с [**типом группы**](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md#group-types) " **Безопасность** ", а не **Microsoft 365**.
- Перед включением доступа для [вложенных групп](../..//active-directory/fundamentals/active-directory-groups-membership-azure-portal.md)может возникнуть дополнительная задержка.
- Пользователи, которым требуется просматривать ресурсы в портал Azure, должны иметь роль [читателя](../../role-based-access-control/built-in-roles.md#reader) (или другую встроенную роль, которая включает доступ для чтения).
- [Встроенные роли Azure](../../role-based-access-control/built-in-roles.md) , включаемые в разрешения, не должны включать устаревшие роли. Если встроенная роль Azure становится устаревшей, все пользователи, подключенные к этой роли, будут потеряют доступ, и вы не сможете подключить дополнительные делегирования. Чтобы устранить эту проблему, обновите шаблон, чтобы использовать только поддерживаемые встроенные роли, а затем выполните новое развертывание.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об [интерфейсах управления для различных клиентов](../concepts/cross-tenant-management-experience.md).
- [Просматривайте клиентов и управляйте ими](view-manage-customers.md), перейдя в раздел **Мои клиенты** на портале Azure.
- Узнайте, как [Обновить](update-delegation.md) или [Удалить](remove-delegation.md) делегирование.
