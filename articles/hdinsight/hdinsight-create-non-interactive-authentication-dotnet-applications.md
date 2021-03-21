---
title: Приложение .NET без интерактивной проверки подлинности — Azure HDInsight
description: Узнайте, как создавать приложения Microsoft .NET с неинтерактивной проверкой подлинности в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/23/2019
ms.openlocfilehash: 0fffe763f77fffcec6010d43deb92ffea9ece15d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101094916"
---
# <a name="create-a-non-interactive-authentication-net-hdinsight-application"></a>Создание приложения .NET HDInsight с неинтерактивной проверкой подлинности

Запустите приложение Microsoft .NET Azure HDInsight в собственном удостоверении приложения (не в интерактивном режиме) или в удостоверении пользователя, выполнившего вход в приложение (интерактивное). В этой статье описано, как создать приложение .NET с неинтерактивной проверкой подлинности для подключения к Azure и управления HDInsight. Пример интерактивного приложения см. в разделе [Подключение к Azure HDInsight](hdinsight-administer-use-dotnet-sdk.md#connect-to-azure-hdinsight).

Из неинтерактивного приложения .NET, вам потребуется следующее:

* Идентификатор клиента для подписки Azure (также называется *идентификатором каталога*). Дополнительные сведения см. в разделе [Получение идентификатора клиента](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
* Идентификатор клиента для приложения Azure Active Directory (Azure AD). См. статью [создание Azure Active Directory приложения](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) и [Получение идентификатора приложения](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
* Секретный ключ приложения Azure AD. Дополнительные сведения см. в разделе [Получение идентификатора приложения и ключа проверки подлинности](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).

## <a name="prerequisites"></a>Предварительные условия

Кластер HDInsight. Дополнительные сведения см. в [руководстве по началу работы](hadoop/apache-hadoop-linux-tutorial-get-started.md).

## <a name="assign-a-role-to-the-azure-ad-application"></a>Назначение роли приложению Azure AD

Назначьте приложению Azure AD определенную [роль](../role-based-access-control/built-in-roles.md), чтобы предоставить ему разрешения на выполнение действий. Вы можете задать область действия на уровне подписки, группы ресурсов или ресурса. Разрешения наследуют более низкие уровни области действия. Например, Добавление приложения в роль читатель для группы ресурсов означает, что приложение может считывать группу ресурсов и все ресурсы в ней. В этой статье вы настроите область на уровне группы ресурсов. Дополнительные сведения см. в статье [назначение ролей Azure для управления доступом к ресурсам подписки Azure](../role-based-access-control/role-assignments-portal.md).

**Добавление роли владельца в приложение Azure AD**

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к группе ресурсов с кластером HDInsight, в которой будет выполняться запрос Hive далее в этой статье. Если у вас много групп ресурсов, для поиска нужной можно использовать фильтр.
1. В меню группы ресурсов щелкните **Управление доступом (IAM)**.
1. Выберите вкладку **Назначения ролей**, чтобы просмотреть текущие назначения ролей.
1. В верхней части страницы выберите **+ Добавить**.
1. Выполните инструкции по добавлению роли владельца в приложение Azure AD. После успешного добавления роли приложение отобразится в списке роли владельца.

## <a name="develop-an-hdinsight-client-application"></a>Разработка клиентского приложения HDInsight

1. Создайте консольное приложение C#.
2. Добавьте следующие пакеты [NuGet](https://www.nuget.org/) :

    * `Install-Package Microsoft.Azure.Common.Authentication -Pre`
    * `Install-Package Microsoft.Azure.Management.HDInsight -Pre`
    * `Install-Package Microsoft.Azure.Management.Resources -Pre`

3. Выполните следующий код:

    ```csharp
    using System;
    using System.Security;
    using Microsoft.Azure;
    using Microsoft.Azure.Common.Authentication;
    using Microsoft.Azure.Common.Authentication.Factories;
    using Microsoft.Azure.Common.Authentication.Models;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.HDInsight;
    
    namespace CreateHDICluster
    {
        internal class Program
        {
            private static HDInsightManagementClient _hdiManagementClient;
    
            private static Guid SubscriptionId = new Guid("<Enter your Azure subscription ID>");
            private static string tenantID = "<Enter your tenant ID (also called directory ID)>";
            private static string applicationID = "<Enter your application ID>";
            private static string secretKey = "<Enter the application secret key>";
    
            private static void Main(string[] args)
            {
                var key = new SecureString();
                foreach (char c in secretKey) { key.AppendChar(c); }
    
                var tokenCreds = GetTokenCloudCredentials(tenantID, applicationID, key);
                var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
    
                var resourceManagementClient = new ResourceManagementClient(subCloudCredentials);
                resourceManagementClient.Providers.Register("Microsoft.HDInsight");
    
                _hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);
    
                var results = _hdiManagementClient.Clusters.List();
                foreach (var name in results.Clusters)
                {
                    Console.WriteLine("Cluster Name: " + name.Name);
                    Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
                    Console.WriteLine("\t Cluster location: " + name.Location);
                    Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
                }
                Console.WriteLine("Press Enter to continue");
                Console.ReadLine();
            }
    
            /// Get the access token for a service principal and provided key.          
            public static TokenCloudCredentials GetTokenCloudCredentials(string tenantId, string clientId, SecureString secretKey)
            {
                var authFactory = new AuthenticationFactory();
                var account = new AzureAccount { Type = AzureAccount.AccountType.ServicePrincipal, Id = clientId };
                var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
                var accessToken =
                    authFactory.Authenticate(account, env, tenantId, secretKey, ShowDialog.Never).AccessToken;
    
                return new TokenCloudCredentials(accessToken);
            }
    
            public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(SubscriptionCloudCredentials creds, Guid subId)
            {
                return new TokenCloudCredentials(subId.ToString(), ((TokenCloudCredentials)creds).Token);
            }
        }
    }
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Создание приложения Azure Active Directory и субъекта-службы на портале Azure](../active-directory/develop/howto-create-service-principal-portal.md).
* Узнайте, как [выполнять проверку подлинности субъекта-службы в Azure Resource Manager](../active-directory/develop/howto-authenticate-service-principal-powershell.md).
* Сведения об [управлении доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/role-assignments-portal.md).
