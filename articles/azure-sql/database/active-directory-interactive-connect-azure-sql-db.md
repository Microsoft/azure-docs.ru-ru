---
title: Активедиректоринтерактиве подключается к SQL
description: Пример кода C# с пояснениями для подключения к службе "База данных SQL Azure" с использованием режима SqlAuthenticationMethod.ActiveDirectoryInteractive.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: active directory, has-adal-ref, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: MirekS
ms.reviewer: vanto
ms.date: 04/23/2020
ms.openlocfilehash: 93831ec4c1dc3e34c2ea144e71b67dae711ee870
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94841654"
---
# <a name="connect-to-azure-sql-database-with-azure-ad-multi-factor-authentication"></a>Подключение к базе данных SQL Azure с помощью многофакторной идентификации Azure AD
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этой статье представлена программа на C#, которая подключается к базе данных SQL Azure. Программа использует интерактивный режим проверки подлинности, поддерживающий [многофакторную идентификацию Azure AD](../../active-directory/authentication/concept-mfa-howitworks.md).

Дополнительные сведения о поддержке многофакторной идентификации для средств SQL см. [в разделе поддержка Azure Active Directory в SQL Server Data Tools (SSDT)](/sql/ssdt/azure-active-directory).

## <a name="multi-factor-authentication-for-azure-sql-database"></a>Многофакторная идентификация для базы данных SQL Azure

Начиная с версии платформа .NET Framework 4.7.2, перечисление [`SqlAuthenticationMethod`](/dotnet/api/system.data.sqlclient.sqlauthenticationmethod) имеет новое значение: `ActiveDirectoryInteractive` . В клиентской программе C# значение перечисления указывает системе использовать интерактивный режим Azure Active Directory (Azure AD), который поддерживает многофакторную проверку подлинности для подключения к базе данных SQL Azure. Пользователь, который затем запускает программу, увидит следующие диалоговые окна:

* Диалоговое окно с отображением имени пользователя Azure AD, в котором запрашивается пароль пользователя.

   Если домен пользователя является федеративным с Azure AD, это диалоговое окно не отображается, так как пароль не требуется.

   Если политика Azure AD накладывает многофакторную проверку подлинности для пользователя, отображаются два следующих диалоговых окна.

* В первый раз, когда пользователь проходит через многофакторную проверку подлинности, система отображает диалоговое окно с запросом номера мобильного телефона для отправки текстовых сообщений. Каждое сообщение содержит *код проверки*, который пользователь должен ввести в следующем диалоговом окне.

* Диалоговое окно, запрашивающее код проверки многофакторной проверки подлинности, который система отправила на мобильный телефон.

Сведения о том, как настроить Azure AD таким образом, чтобы требовалась многофакторная проверка подлинности, см. в статье [Приступая к работе со службой Многофакторной идентификации Azure AD в облаке](../../active-directory/authentication/howto-mfa-getstarted.md).

Снимки экрана этих диалоговых окон см. в статье [Настройка Многофакторной идентификации для SQL Server Management Studio и Azure AD](authentication-mfa-ssms-configure.md).

> [!TIP]
> Вы можете выполнять поиск платформа .NET Framework API на [странице средства браузера .NET API](/dotnet/api/).
>
> Можно также выполнить поиск непосредственно с [необязательным &lt; &gt; параметром? Term = Поиск значения](/dotnet/api/?term=SqlAuthenticationMethod).

## <a name="configure-your-c-application-in-the-azure-portal"></a>Настройка приложения C# на портале Azure

Прежде чем начать, необходимо создать и получить [логический экземпляр SQL Server](logical-servers.md) .

### <a name="register-your-app-and-set-permissions"></a>Регистрация приложения и установка разрешений

Чтобы использовать проверку подлинности Azure AD, ваша программа C# должна быть зарегистрирована как приложение Azure AD. Чтобы зарегистрировать приложение, необходимо быть администратором Azure AD или пользователем с назначенной ролью *разработчика приложений* Azure AD. Дополнительные сведения о назначении ролей см. [в статье назначение ролей администратора и пользователей, не являющихся администраторами, с помощью Azure Active Directory](../../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md).

При выполнении регистрации приложения создается и отображается **идентификатор приложения**. Для возможности подключения программа должна включать этот идентификатор.

Чтобы зарегистрировать приложение и задать для него необходимые разрешения:

1. В портал Azure выберите **Azure Active Directory**  >  **Регистрация приложений**  >  **Новая регистрация**.

    ![Регистрация приложений](./media/active-directory-interactive-connect-azure-sql-db/image1.png)

    После создания регистрации приложения создается и отображается значение **идентификатора приложения** .

    ![Отображение идентификатора приложения](./media/active-directory-interactive-connect-azure-sql-db/image2.png)

2. Выберите **Разрешения API** > **Добавить разрешение**.

    ![Параметры разрешений для зарегистрированного приложения](./media/active-directory-interactive-connect-azure-sql-db/sshot-registered-app-settings-required-permissions-add-api-access-c32.png)

3. Выберите **интерфейсы API, используемые моей организацией** > введите **базу данных sql Azure** в > поиска и выберите **базу данных SQL Azure**.

    ![Добавление доступа через API для службы "База данных SQL Azure"](./media/active-directory-interactive-connect-azure-sql-db/sshot-registered-app-settings-required-permissions-add-api-access-Azure-sql-db-d11.png)

4. Выберите **делегированные разрешения**  >  **user_impersonation**  >  **Добавить разрешения**.

    ![Делегирование разрешений API для службы "База данных SQL Azure"](./media/active-directory-interactive-connect-azure-sql-db/sshot-add-api-access-azure-sql-db-delegated-permissions-checkbox-e14.png)

### <a name="set-an-azure-ad-admin-for-your-server"></a>Настройка администратора Azure AD для сервера

Для запуска программы C# логическому администратору [SQL Server](logical-servers.md) необходимо назначить администратора Azure AD для вашего сервера.

На странице **SQL Server** выберите **Active Directory администратор**  >  **задать администратора**.

Дополнительные сведения об администраторах и пользователях Azure AD для базы данных SQL Azure см. на снимках экрана в разделе [Настройка проверки подлинности Azure Active Directory и управление ею с помощью базы данных SQL](authentication-aad-configure.md#provision-azure-ad-admin-sql-database).

### <a name="add-a-non-admin-user-to-a-specific-database-optional"></a>Добавление пользователя без прав администратора для конкретной базы данных (необязательно)

Администратор Azure AD для [логического сервера SQL](logical-servers.md) Server может запустить пример программы на C#. Пользователь Azure AD может запустить программу, если она находится в базе данных. Добавить пользователя может администратор SQL Azure AD или пользователь Azure AD, который уже существует в базе данных и имеет на нее разрешение `ALTER ANY USER`.

Вы можете добавить пользователя в базу данных с помощью команды SQL [`Create User`](/sql/t-sql/statements/create-user-transact-sql) . Например, `CREATE USER [<username>] FROM EXTERNAL PROVIDER`.

Дополнительные сведения см. в статье [использование Azure Active Directory проверки подлинности для проверки подлинности с помощью базы данных SQL, управляемый экземпляр или Azure синапсе Analytics](authentication-aad-overview.md).

## <a name="new-authentication-enum-value"></a>Новое значение перечисления проверки подлинности

В примере на языке C# используется [`System.Data.SqlClient`](/dotnet/api/system.data.sqlclient) пространство имен. Особым интересом для многофакторной проверки подлинности является перечисление `SqlAuthenticationMethod` , которое имеет следующие значения:

* `SqlAuthenticationMethod.ActiveDirectoryInteractive`

   Используйте это значение с именем пользователя Azure AD для реализации многофакторной проверки подлинности. Значение подробно описано в этой статье. Он создает интерактивный интерфейс, отображая диалоговые окна для ввода пароля пользователя, а затем для проверки многофакторной проверки подлинности, если для этого пользователя установлено многофакторная идентификация. Это значение доступно начиная с .NET Framework версии 4.7.2.

* `SqlAuthenticationMethod.ActiveDirectoryIntegrated`

  Это значение следует использовать для *федеративной* учетной записи. При использовании федеративной учетной записи имя пользователя известно домену Windows. Этот метод проверки подлинности не поддерживает многофакторную проверку подлинности.

* `SqlAuthenticationMethod.ActiveDirectoryPassword`

  Используйте это значение для проверки подлинности, требующей имени пользователя и пароля Azure AD. Проверка подлинности выполняется в службе "База данных SQL Azure". Этот метод не поддерживает многофакторную проверку подлинности.

> [!NOTE]
> При использовании .NET Core необходимо использовать пространство имен [Microsoft. Data. SqlClient](/dotnet/api/microsoft.data.sqlclient?view=sqlclient-dotnet-core-1.1) . Дополнительные сведения см. в следующем [блоге](https://devblogs.microsoft.com/dotnet/introducing-the-new-microsoftdatasqlclient/).

## <a name="set-c-parameter-values-from-the-azure-portal"></a>Установка значений параметров C# с портала Azure

Для успешного выполнения программы C# необходимо назначить соответствующие значения статическим полям. Здесь отображаются поля с примерами значений. Также показаны портал Azure расположения, в которых можно получить необходимые значения.

| Имя статического поля | Пример значения | Расположение на портале Azure |
| :---------------- | :------------ | :-------------------- |
| Az_SQLDB_svrName | "my-sqldb-svr.database.windows.net" | **Серверы**  >  SQL Server **Фильтровать по имени** |
| AzureAD_UserID | "Аусер \@ abc.onmicrosoft.com" | **Azure Active Directory**  >  **Пользователь**  >  **Новый гостевой пользователь** |
| Initial_DatabaseName | "myDatabase" | **Серверы**  >  SQL Server **Базы данных SQL** |
| ClientApplicationID | "a94f9c62-97fe-4d19-b06d-111111111111" | **Azure Active Directory**  >  **Регистрация приложений**  >  **Поиск по имени**  >  **Идентификатор приложения** |
| RedirectUri | new Uri("https://mywebserver.com/") | **Azure Active Directory**  >  **Регистрация приложений**  >  **Поиск по имени**  >  *[Регистрация приложения]*  >  **Параметры**  >  **Редиректурис**<br /><br />В этой статье любое допустимое значение подходит для RedirectUri, так как оно не используется здесь. |
| &nbsp; | &nbsp; | &nbsp; |

## <a name="verify-with-sql-server-management-studio"></a>Проверка с помощью SQL Server Management Studio

Перед запуском программы на C# рекомендуется проверить правильность установки и конфигураций в SQL Server Management Studio (SSMS). Любой сбой программы C# можно затем свести к исходному коду.

### <a name="verify-server-level-firewall-ip-addresses"></a>Проверка IP-адресов брандмауэра на уровне сервера

Запустите SSMS с того же компьютера в том же здании, где вы планируете выполнять программу C#. Для этого теста любой режим **проверки подлинности** — ОК. Если вы указываете, что сервер не принимает IP-адрес, см. раздел [правила брандмауэра уровня сервера и базы данных](firewall-configure.md) .

### <a name="verify-azure-active-directory-multi-factor-authentication"></a>Проверка многофакторной проверки подлинности Azure Active Directory

Запустите SSMS еще раз, на этот раз для параметра **authentication** установлено значение **Azure Active Directory-Universal с MFA**. Этот параметр требует SSMS версии 17.5 или более поздней.

Дополнительные сведения см. в статье [Настройка многофакторной идентификации для SSMS и Azure AD](authentication-mfa-ssms-configure.md).

> [!NOTE]
> Если вы являетесь гостевым пользователем в базе данных, необходимо также указать доменное имя Azure AD для базы данных: выберите **Параметры**  >  **доменное имя AD или идентификатор клиента**. Чтобы найти имя домена в портал Azure, выберите **Azure Active Directory**  >  **пользовательские доменные имена**. В программе C# можно не предоставлять доменное имя.

## <a name="c-code-example"></a>Пример кода C#

> [!NOTE]
> При использовании .NET Core необходимо использовать пространство имен [Microsoft. Data. SqlClient](/dotnet/api/microsoft.data.sqlclient?view=sqlclient-dotnet-core-1.1) . Дополнительные сведения см. в следующем [блоге](https://devblogs.microsoft.com/dotnet/introducing-the-new-microsoftdatasqlclient/).

Пример программы C# основан на сборке библиотеки DLL [*Microsoft.IdentityModel.Clients.ActiveDirectory*](/dotnet/api/microsoft.identitymodel.clients.activedirectory).

Чтобы установить этот пакет, в Visual Studio выберите **проект**  >  **Управление пакетами NuGet**. Выполните поиск библиотеки **Microsoft.IdentityModel.Clients.ActiveDirectory** и установите ее.

Это пример исходного кода C#.

```csharp

using System;

// Reference to Azure AD authentication assembly
using Microsoft.IdentityModel.Clients.ActiveDirectory;

using DA = System.Data;
using SC = System.Data.SqlClient;
using AD = Microsoft.IdentityModel.Clients.ActiveDirectory;
using TX = System.Text;
using TT = System.Threading.Tasks;

namespace ADInteractive5
{
    class Program
    {
        // ASSIGN YOUR VALUES TO THESE STATIC FIELDS !!
        static public string Az_SQLDB_svrName = "<Your server>";
        static public string AzureAD_UserID = "<Your User ID>";
        static public string Initial_DatabaseName = "<Your Database>";
        // Some scenarios do not need values for the following two fields:
        static public readonly string ClientApplicationID = "<Your App ID>";
        static public readonly Uri RedirectUri = new Uri("<Your URI>");

        public static void Main(string[] args)
        {
            var provider = new ActiveDirectoryAuthProvider();

            SC.SqlAuthenticationProvider.SetProvider(
                SC.SqlAuthenticationMethod.ActiveDirectoryInteractive,
                //SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated,  // Alternatives.
                //SC.SqlAuthenticationMethod.ActiveDirectoryPassword,
                provider);

            Program.Connection();
        }

        public static void Connection()
        {
            SC.SqlConnectionStringBuilder builder = new SC.SqlConnectionStringBuilder();

            // Program._  static values that you set earlier.
            builder["Data Source"] = Program.Az_SQLDB_svrName;
            builder.UserID = Program.AzureAD_UserID;
            builder["Initial Catalog"] = Program.Initial_DatabaseName;

            // This "Password" is not used with .ActiveDirectoryInteractive.
            //builder["Password"] = "<YOUR PASSWORD HERE>";

            builder["Connect Timeout"] = 15;
            builder["TrustServerCertificate"] = true;
            builder.Pooling = false;

            // Assigned enum value must match the enum given to .SetProvider().
            builder.Authentication = SC.SqlAuthenticationMethod.ActiveDirectoryInteractive;
            SC.SqlConnection sqlConnection = new SC.SqlConnection(builder.ConnectionString);

            SC.SqlCommand cmd = new SC.SqlCommand(
                "SELECT '******** MY QUERY RAN SUCCESSFULLY!! ********';",
                sqlConnection);

            try
            {
                sqlConnection.Open();
                if (sqlConnection.State == DA.ConnectionState.Open)
                {
                    var rdr = cmd.ExecuteReader();
                    var msg = new TX.StringBuilder();
                    while (rdr.Read())
                    {
                        msg.AppendLine(rdr.GetString(0));
                    }
                    Console.WriteLine(msg.ToString());
                    Console.WriteLine(":Success");
                }
                else
                {
                    Console.WriteLine(":Failed");
                }
                sqlConnection.Close();
            }
            catch (Exception ex)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Connection failed with the following exception...");
                Console.WriteLine(ex.ToString());
                Console.ResetColor();
            }
        }
    } // EOClass Program.

    /// <summary>
    /// SqlAuthenticationProvider - Is a public class that defines 3 different Azure AD
    /// authentication methods.  The methods are supported in the new .NET 4.7.2.
    ///  .
    /// 1. Interactive,  2. Integrated,  3. Password
    ///  .
    /// All 3 authentication methods are based on the Azure
    /// Active Directory Authentication Library (ADAL) managed library.
    /// </summary>
    public class ActiveDirectoryAuthProvider : SC.SqlAuthenticationProvider
    {
        // Program._ more static values that you set!
        private readonly string _clientId = Program.ClientApplicationID;
        private readonly Uri _redirectUri = Program.RedirectUri;

        public override async TT.Task<SC.SqlAuthenticationToken>
            AcquireTokenAsync(SC.SqlAuthenticationParameters parameters)
        {
            AD.AuthenticationContext authContext =
                new AD.AuthenticationContext(parameters.Authority);
            authContext.CorrelationId = parameters.ConnectionId;
            AD.AuthenticationResult result;

            switch (parameters.AuthenticationMethod)
            {
                case SC.SqlAuthenticationMethod.ActiveDirectoryInteractive:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,  // "https://database.windows.net/"
                        _clientId,
                        _redirectUri,
                        new AD.PlatformParameters(AD.PromptBehavior.Auto),
                        new AD.UserIdentifier(
                            parameters.UserId,
                            AD.UserIdentifierType.RequiredDisplayableId));
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_1 == '.ActiveDirectoryIntegrated'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserCredential());
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryPassword:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_2 == '.ActiveDirectoryPassword'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserPasswordCredential(
                            parameters.UserId,
                            parameters.Password));
                    break;

                default: throw new InvalidOperationException();
            }
            return new SC.SqlAuthenticationToken(result.AccessToken, result.ExpiresOn);
        }

        public override bool IsSupported(SC.SqlAuthenticationMethod authenticationMethod)
        {
            return authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryInteractive
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryPassword;
        }
    } // EOClass ActiveDirectoryAuthProvider.
} // EONamespace.  End of entire program source code.

```

&nbsp;

Это пример выходных данных теста C#.

```C#
[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>> ADInteractive5.exe
In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.
******** MY QUERY RAN SUCCESSFULLY!! ********

:Success

[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>>
```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager по-прежнему поддерживается базой данных SQL Azure, но вся будущая разработка сосредоточена на модуле Az.Sql. Сведения об этих командлетах см. в разделе [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Аргументы команд в модулях Az и AzureRm практически идентичны.

& [Get-Азсклсерверактиведиректорядминистратор](/powershell/module/az.sql/get-azsqlserveractivedirectoryadministrator)