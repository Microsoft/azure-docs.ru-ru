---
title: Подключение клиента к Azure Lighthouse
description: Узнайте, как подключить клиента к Azure Lighthouse, предоставив доступ к ресурсам и управление ими с помощью собственного клиента, используя делегированное управление ресурсами Azure.
ms.date: 09/24/2020
ms.topic: how-to
ms.openlocfilehash: 43f28073c996167c82e241476020bdc341486b26
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024302"
---
# <a name="onboard-a-customer-to-azure-lighthouse"></a>Подключение клиента к Azure Lighthouse

В этой статье объясняется, как поставщик услуг может подключить клиента к Azure Лигхсаусе. При этом можно получить доступ к делегированным ресурсам клиента (подпискам и группам ресурсов) и управлять ими с помощью собственного клиента Azure Active Directory (Azure AD), используя [делегированное управление ресурсами Azure](../concepts/azure-delegated-resource-management.md).

> [!TIP]
> Хотя в этом разделе мы будем называть поставщиков услуг и клиентов, [предприятия, управляющие несколькими клиентами](../concepts/enterprise.md) , могут использовать один и тот же процесс для настройки Azure лигхсаусе и консолидации их возможностей управления.

Процесс адаптации можно повторить для нескольких клиентов. Когда пользователь с соответствующими разрешениями выполняет вход в управляющий клиент, этот пользователь может быть разрешен в областях клиентской аренды для выполнения операций управления без необходимости входа в каждый клиент клиента.

Чтобы отслеживать свое влияние на разных клиентов и получать дополнительное признание, свяжите свой идентификатор Microsoft Partner Network (MPN) хотя бы с одной учетной записью пользователя, которая имеет доступ к каждой из подключенных подписок. Вам потребуется выполнить эту связь в клиенте поставщика услуг. Рекомендуется создать в клиенте учетную запись субъекта-службы, связанную с ИДЕНТИФИКАТОРом MPN, а затем включать этот субъект-службу при каждом подключении клиента. Дополнительные сведения см. [в статье связывание идентификатора партнера, чтобы включить получение партнерского кредита на делегированные ресурсы](partner-earned-credit.md).

> [!NOTE]
> Клиенты также могут быть подключены к Azure Лигхсаусе при покупке предложения управляемой службы (открытого или частного), которое [публикуется в Azure Marketplace](publish-managed-services-offers.md). Вы также можете использовать процесс адаптации, описанный здесь, наряду с предложениями, опубликованными в Azure Marketplace.

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

Как поставщик услуг вы можете выполнять для одного клиента несколько задач, требующих разных прав доступа для нескольких областей. Вы можете определить столько авторизаций, сколько требуется, чтобы назначить соответствующие [встроенные роли Azure](../../role-based-access-control/built-in-roles.md) пользователям в клиенте.

Чтобы упростить управление, мы рекомендуем использовать группы пользователей Azure AD для каждой роли. Это дает возможность добавлять или удалять отдельных пользователей в группе, имеющей доступ, чтобы не повторять процесс адаптации для внесения изменений в пользователя. Можно назначить роли субъекту-службе, который может быть полезен для сценариев автоматизации.

> [!IMPORTANT]
> Чтобы добавить разрешения для группы Azure AD, **Тип группы** должен быть установлен в значение **Безопасность**. Этот вариант автоматически выбирается при создании группы. Дополнительные сведения см. в статье [Создание простой группы и добавление в нее участников с помощью Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

При определении авторизаций обязательно следуйте принципу минимальных привилегий, чтобы пользователи имели только необходимые разрешения для выполнения их работы. Инструкции и сведения о поддерживаемых ролях см. в статье о [клиентах, пользователях и ролях в сценариях Azure лигхсаусе](../concepts/tenants-users-roles.md).

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

Последняя авторизация в приведенном выше примере добавляет параметр **principalId** с ролью "Администратор доступа пользователя" (18d7d88d-d35e-4fb5-a5c3-7773c20a72d9). При назначении этой роли вы должны включить свойство **delegatedRoleDefinitionIds** и одну или несколько встроенных ролей. Пользователь, созданный в этой авторизации, сможет назначить эти встроенные роли [управляемым удостоверениям](../../active-directory/managed-identities-azure-resources/overview.md) в клиенте клиента, который необходим для [развертывания политик, которые можно исправить](deploy-policy-remediation.md).  Пользователь также может создавать обращения в службу поддержки.  К этому пользователю не будут применяться никакие другие разрешения, обычно связанные с ролью "Администратор доступа пользователей".

## <a name="deploy-the-azure-resource-manager-templates"></a>Развертывание шаблонов Azure Resource Manager

После обновления файла параметров пользователь в клиенте клиента должен развернуть шаблон Azure Resource Manager в своем клиенте. Для каждой подписки, которую необходимо подключить (или для каждой подписки, содержащей группы ресурсов, которые необходимо подключить), требуется отдельное развертывание.

> [!IMPORTANT]
> Это развертывание должно выполняться учетной записью, отличной от гостевой, в клиенте клиента, имеющей [встроенную роль владельца](../../role-based-access-control/built-in-roles.md#owner) подписки (или содержащей группы ресурсов, которые будут подключены). Чтобы найти пользователей, которые могут делегировать подписку, пользователь клиента может выбрать подписку на портале Azure, открыть **управление доступом (IAM)** и [просмотреть всех пользователей с ролью владельца](../../role-based-access-control/role-assignments-list-portal.md#list-owners-of-a-subscription). 
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
> После завершения развертывания может пройти несколько минут, прежде чем обновления отобразятся на портале Azure.

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

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об [интерфейсах управления для различных клиентов](../concepts/cross-tenant-management-experience.md).
- [Просматривайте клиентов и управляйте ими](view-manage-customers.md), перейдя в раздел **Мои клиенты** на портале Azure.
- Узнайте, как [удалить доступ к делегированию](remove-delegation.md), подключенному ранее.
