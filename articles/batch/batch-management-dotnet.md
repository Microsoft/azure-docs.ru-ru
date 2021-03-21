---
title: Управление ресурсами учетной записи с помощью библиотеки .NET для управления пакетной службой
description: Создание, удаление и изменение ресурсов учетных записей пакетной службы Azure с помощью библиотеки .NET для управления пакетной службой.
ms.topic: how-to
ms.date: 01/26/2021
ms.custom: seodec18, has-adal-ref, devx-track-csharp
ms.openlocfilehash: f6acf23742b8d4c5c31a5ea952aba5c76b64cae0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98896737"
---
# <a name="manage-batch-accounts-and-quotas-with-the-batch-management-client-library-for-net"></a>Управление учетными записями и квотами пакетной службы с помощью клиентской библиотеки .NET для управления пакетной службой

С помощью [библиотеки .NET для управления пакетной службой](/dotnet/api/overview/azure/batch) можно снизить издержки на обслуживание приложений пакетной службы Azure. Эта библиотека позволяет автоматизировать создание и удаление учетных записей пакетной службы, управление ключами и определение квот.

- **Создание и удаление учетных записей пакетной службы** в любом регионе. Например, вы являетесь независимым поставщиком программного обеспечения и оказываете услуги клиентам, каждому из которых назначена соответствующая учетная запись пакетной службы для выставления счетов. Для повышения удобства вы можете добавить на портал для пользователей возможность создания и удаления учетных записей.
- **Получение и повторное создание ключей учетных записей** для всех учетных записей пакетной службы программным образом. Это особенно удобно для обеспечения соответствия политикам безопасности, которые могут требовать периодической смены ключей и определять сроки действия ключей учетных записей. Если у вас есть несколько учетных записей пакетной службы в разных регионах Azure, автоматизация смены ключей повысит эффективность вашего решения.
- **Проверка квот учетных записей** и исключение метода проб и ошибок из процедуры определения ограничений учетных записей пакетной службы. Проверка квот учетной записи до запуска заданий, создание пулов или добавление вычислительных узлов позволит вам заранее выбирать время и место создания вычислительных ресурсов. Вы можете определить учетные записи, требующие повышения квот, прежде чем выделить дополнительные ресурсы в этих учетных записях.
- **Объедините функции других служб Azure** для полнофункционального управления с помощью .net, [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)и [Azure Resource Manager](../azure-resource-manager/management/overview.md) вместе в одном приложении. С помощью этих функций и соответствующих API вы можете предоставлять клиентам удобные возможности проверки подлинности, создания и удаления групп ресурсов, а также доступа к описанным выше функциям.

> [!NOTE]
> Хотя эта статья посвящена программному управлению учетными записями пакетной службы, ключами и квотами, многие из этих действий можно также выполнить с [помощью портал Azure](batch-account-create-portal.md).

## <a name="create-and-delete-batch-accounts"></a>Создание и удаление учетных записей пакетной службы

Одной из основных возможностей API управления пакетной службой является создание и удаление [учетных записей пакетной](accounts.md) службы в регионе Azure. Для этого используйте методы [BatchManagementClient.Account.CreateAsync](/dotnet/api/microsoft.azure.management.batch.batchaccountoperationsextensions.createasync) и [DeleteAsync](/dotnet/api/microsoft.azure.management.batch.batchaccountoperationsextensions.deleteasync), а также их синхронные аналоги.

В следующем фрагменте кода создается учетная запись, выполняется получение созданной учетной записи из пакетной службы, а затем она удаляется. В этом и других фрагментах кода, приведенных в этой статье, `batchManagementClient` представляет собой полностью инициализированный экземпляр [BatchManagementClient](/dotnet/api/microsoft.azure.management.batch.batchmanagementclient).

```csharp
// Create a new Batch account
await batchManagementClient.Account.CreateAsync("MyResourceGroup",
    "mynewaccount",
    new BatchAccountCreateParameters() { Location = "West US" });

// Get the new account from the Batch service
AccountResource account = await batchManagementClient.Account.GetAsync(
    "MyResourceGroup",
    "mynewaccount");

// Delete the account
await batchManagementClient.Account.DeleteAsync("MyResourceGroup", account.Name);
```

> [!NOTE]
> Приложениям, использующим библиотеку .NET для управления пакетной службой и класс BatchManagementClient, необходимы права администратора службы или соадминистратора для доступа к подписке, которой принадлежит управляемая учетная запись пакетной службы. Дополнительные сведения можно получить, ознакомившись с разделом Azure Active Directory ниже и примером кода [AccountManagement](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/AccountManagement).

## <a name="retrieve-and-regenerate-account-keys"></a>Получение и повторное создание ключей учетных записей

Получите первичные и вторичные ключи учетной записи из любой учетной записи пакетной службы в подписке с помощью [жеткэйсасинк](/dotnet/api/microsoft.azure.management.batch.batchaccountoperationsextensions.getkeysasync). Для повторного создания этих ключей используется метод [RegenerateKeyAsync](/dotnet/api/microsoft.azure.management.batch.batchaccountoperationsextensions.regeneratekeyasync).

```csharp
// Get and print the primary and secondary keys
BatchAccountGetKeyResult accountKeys =
    await batchManagementClient.Account.GetKeysAsync(
        "MyResourceGroup",
        "mybatchaccount");
Console.WriteLine("Primary key:   {0}", accountKeys.Primary);
Console.WriteLine("Secondary key: {0}", accountKeys.Secondary);

// Regenerate the primary key
BatchAccountRegenerateKeyResponse newKeys =
    await batchManagementClient.Account.RegenerateKeyAsync(
        "MyResourceGroup",
        "mybatchaccount",
        new BatchAccountRegenerateKeyParameters() {
            KeyName = AccountKeyType.Primary
            });
```

> [!TIP]
> Вы можете упростить процедуру подключения в своем приложении для управления. Сначала получите ключ учетной записи пакетной службы, которой вы хотите управлять с помощью [жеткэйсасинк](/dotnet/api/microsoft.azure.management.batch.batchaccountoperationsextensions.getkeysasync). Затем используйте этот ключ при инициализации класса [BatchSharedKeyCredentials](/dotnet/api/microsoft.azure.batch.auth.batchsharedkeycredentials) библиотеки .NET пакетной службы, который применяется при инициализации [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient).

## <a name="check-azure-subscription-and-batch-account-quotas"></a>Проверка подписки Azure и квот учетной записи пакетной службы

Подписки Azure и отдельные службы Azure, такие как пакетная служба, имеют стандартные квоты для ограничения количества определенных в них сущностей. Квоты по умолчанию для подписок Azure см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md). Квоты пакетной службы по умолчанию см. в статье [Квоты и ограничения пакетной службы Azure](batch-quota-limit.md). С помощью библиотеки .NET для управления пакетной службой можно проверять эти квоты в приложениях. Это позволяет принимать решения о выделении ресурсов перед добавлением учетных записей или вычислительных ресурсов, таких как пулы и вычислительные узлы.

### <a name="check-an-azure-subscription-for-batch-account-quotas"></a>Определение квот для учетной записи пакетной службы в подписке Azure

Прежде чем создавать учетную запись пакетной службы в определенном регионе, вы можете проверить данные подписки Azure, чтобы узнать о возможности создания учетной записи в этом регионе.

В следующем фрагменте кода мы сначала используем **ListAsync** для получения коллекции всех учетных записей пакетной службы, которые находятся в подписке. После получения этой коллекции мы определяем количество учетных записей в целевом регионе. Затем мы используем **жеткуотасасинк** , чтобы получить квоту учетной записи пакетной службы и определить, сколько учетных записей может быть создано в этом регионе.

```csharp
// Get a collection of all Batch accounts within the subscription
BatchAccountListResponse listResponse =
        await batchManagementClient.BatchAccount.ListAsync(new AccountListParameters());
IList<AccountResource> accounts = listResponse.Accounts;
Console.WriteLine("Total number of Batch accounts under subscription id {0}:  {1}",
    creds.SubscriptionId,
    accounts.Count);

// Get a count of all accounts within the target region
string region = "westus";
int accountsInRegion = accounts.Count(o => o.Location == region);

// Get the account quota for the specified region
SubscriptionQuotasGetResponse quotaResponse = await batchManagementClient.Location.GetQuotasAsync(region);
Console.WriteLine("Account quota for {0} region: {1}", region, quotaResponse.AccountQuota);

// Determine how many accounts can be created in the target region
Console.WriteLine("Accounts in {0}: {1}", region, accountsInRegion);
Console.WriteLine("You can create {0} accounts in the {1} region.", quotaResponse.AccountQuota - accountsInRegion, region);
```

В приведенном выше фрагменте `creds` является экземпляром **TokenCredentials**. Пример создания этого объекта см. в примере [AccountManagement](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/AccountManagement) на GitHub.

### <a name="check-a-batch-account-for-compute-resource-quotas"></a>Определение квоты вычислительных ресурсов для учетной записи пакетной службы

Прежде чем увеличивать количество вычислительных ресурсов в решении пакетной службы, убедитесь, что выделяемые ресурсы не превысят квоты для учетной записи. В следующем фрагменте кода мы выводим сведения о квотах для учетной записи пакетной службы с именем `mybatchaccount`. В своем приложении с помощью этих сведений можно определить, способна ли учетная запись обрабатывать дополнительные ресурсы, которые вы хотите создать.

```csharp
// First obtain the Batch account
BatchAccountGetResponse getResponse =
    await batchManagementClient.Account.GetAsync("MyResourceGroup", "mybatchaccount");
AccountResource account = getResponse.Resource;

// Now print the compute resource quotas for the account
Console.WriteLine("Core quota: {0}", account.Properties.CoreQuota);
Console.WriteLine("Pool quota: {0}", account.Properties.PoolQuota);
Console.WriteLine("Active job and job schedule quota: {0}", account.Properties.ActiveJobAndJobScheduleQuota);
```

> [!IMPORTANT]
> Хотя для подписок и служб Azure существуют квоты по умолчанию, многие из этих ограничений могут быть вызваны [запросом увеличения квоты в портал Azure](batch-quota-limit.md#increase-a-quota).

## <a name="use-azure-ad-with-batch-management-net"></a>Использование Azure AD с библиотекой .NET для управления пакетной службой

Данная библиотека является клиентом поставщика ресурсов Azure и используется совместно с [Azure Resource Manager](../azure-resource-manager/management/overview.md) для программного управления ресурсами учетных записей. Служба Azure AD необходима для аутентификации запросов, выполненных с помощью любого клиента поставщика ресурсов Azure, а также библиотеки .NET для управления пакетной службой и Azure Resource Manager. Сведения об использовании Azure AD с библиотекой .NET для управления пакетной службой см. в разделе [Authenticate from Batch solutions with Active Directory](batch-aad-auth.md) (Аутентификация решений пакетной службы в Active Directory).

## <a name="sample-project-on-github"></a>Пример проекта на сайте GitHub

Работу библиотеки .NET для управления пакетной службой можно посмотреть на примере проекта [AccountManagement](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement) на сайте GitHub. В примере приложения AccountManagement демонстрируются следующие операции.

1. Получение маркера безопасности из Active Directory Azure с помощью библиотеки [ADAL](../active-directory/azuread-dev/active-directory-authentication-libraries.md). Если пользователь не выполнил вход, ему будет предложено ввести учетные данные Azure.
2. Создание [SubscriptionClient](/dotnet/api/microsoft.azure.management.resourcemanager.subscriptionclient) для отправки запроса к Azure на получение списка подписок, связанных с учетной записью, с помощью маркера безопасности, полученного из Azure AD. Пользователь может выбрать подписку из списка, если он содержит более одной подписки.
3. Получение учетных данных, связанных с выбранной подпиской.
4. Создание объекта [ResourceManagementClient](/dotnet/api/microsoft.azure.management.resourcemanager.resourcemanagementclient) с использованием учетных данных.
5. Создание группы ресурсов с помощью объекта [ResourceManagementClient](/dotnet/api/microsoft.azure.management.resourcemanager.resourcemanagementclient).
6. Выполнение нескольких операций с учетной записью пакетной службы с помощью объекта [BatchManagementClient](/dotnet/api/microsoft.azure.management.batch.batchmanagementclient).
   - Создание учетной записи пакетной службы в новой группе ресурсов.
   - Получение созданной учетной записи из пакетной службы.
   - Вывод ключей учетной записи для новой учетной записи.
   - Повторное создание первичного ключа для учетной записи.
   - Вывод сведений о квотах для учетной записи.
   - Вывод сведений о квотах для подписки.
   - Вывод всех учетных записей в подписке.
   - Удалите созданную учетную запись.
7. Удалите ее.

Для успешного выполнения примера приложения необходимо зарегистрировать его в клиенте Azure AD на портале Azure и предоставить разрешения для API Azure Resource Manager. Выполните инструкции, описанные в статье [Аутентификация решений по управлению пакетной службой с помощью Active Directory](batch-aad-auth-management.md).

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте подробнее о [рабочем процессе и основных ресурсах пакетной службы](batch-service-workflow-features.md), таких как пулы, узлы, задания и задачи.
- Здесь приведены основные сведения о разработке приложений с поддержкой пакетной службы с помощью [клиентской библиотеки .NET для пакетной службы](quick-run-dotnet.md) или [Python](quick-run-python.md). В этих кратких руководствах описывается пример приложения, которое использует пакетную службу для выполнения рабочей нагрузки на нескольких процессорных узлах с использованием службы хранилища Azure для промежуточного хранения и извлечения файлов рабочей нагрузки. git пус
