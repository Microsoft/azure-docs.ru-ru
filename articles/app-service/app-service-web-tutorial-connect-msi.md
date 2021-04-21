---
title: Руководство по получению доступа к данным с помощью управляемого удостоверения
description: Сведения об обеспечении безопасного подключения данных с использованием управляемого удостоверения, а также о применении этого к другим службам Azure.
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/27/2020
ms.custom: devx-track-csharp, mvc, cli-validate, devx-track-azurecli
ms.openlocfilehash: 0012c3d208998786ef5aa34320f3bccc4e51ebe6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107782789"
---
# <a name="tutorial-secure-azure-sql-database-connection-from-app-service-using-a-managed-identity"></a>Руководство по Безопасное подключение к Базе данных SQL Azure из службы приложений с использованием управляемого удостоверения

[Служба приложений](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости в Azure. Она также предоставляет [управляемое удостоверение](overview-managed-identity.md) для вашего приложения, которое является готовым решением для защиты доступа к [Базе данных SQL Azure](/azure/sql-database/) и другим службам Azure. Управляемые удостоверения в службе приложений делают ваше приложение более безопасным, устраняя из него секреты, такие как учетные данные в строках подключения. В этом руководстве вы добавите управляемое удостоверение в пример веб-приложения, которое вы создали при работе с одним из следующих руководств: 

- [Руководство. Создание приложения ASP.NET в Azure с подключением к Базе данных SQL Azure](app-service-web-tutorial-dotnet-sqldatabase.md)
- [Руководство. Создание приложения ASP.NET Core с подключением к Базе данных SQL Azure в Службе приложений Azure](tutorial-dotnetcore-sqldb-app.md)

Когда вы закончите, ваш пример приложения будет безопасно подключаться к базе данных SQL без необходимости использования имен пользователей и паролей.

> [!NOTE]
> Шаги, описанные в этом руководстве, поддерживаются в следующих версиях:
> 
> - .NET Framework 4.7.2 и выше;
> - .NET Core 2.2 и выше.
>

Освещаются следующие темы:

> [!div class="checklist"]
> * Включение управляемых удостоверений
> * Предоставление управляемому удостоверению доступа к Базе данных SQL
> * Настройте Entity Framework для использования аутентификации Azure AD с базой данных SQL
> * Подключитесь к базе данных SQL из Visual Studio с использованием аутентификации Azure AD

> [!NOTE]
>Аутентификация Azure AD _отличается_ от [интегрированной аутентификации Windows](/previous-versions/windows/it-pro/windows-server-2003/cc758557(v=ws.10)) в локальном экземпляре Active Directory (AD DS). Доменные службы Active Directory и Azure AD используют разные протоколы аутентификации. Дополнительные сведения см. на странице [Документация по доменным службам Azure AD](../active-directory-domain-services/index.yml).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Эта статья является продолжением статьи [Руководство. Создание приложения ASP.NET в Azure с подключением к Базе данных SQL](app-service-web-tutorial-dotnet-sqldatabase.md) или [Руководство по созданию приложения ASP.NET Core и Базы данных SQL в Службе приложений Azure](tutorial-dotnetcore-sqldb-app.md). Если вы еще не ознакомились с этими руководствами, сначала изучите одно из них. Кроме того, вы можете адаптировать шаги для своего собственного приложения .NET с Базой данных SQL.

Чтобы отладить приложение, используя Базу данных SQL в качестве серверного компонента, убедитесь, что вы разрешили клиентские подключения со своего компьютера. В противном случае добавьте IP-адрес клиента, выполнив шаги, указанные в разделе [Управление правилами брандмауэра для IP-адресов на уровне сервера с помощью портала Azure](../azure-sql/database/firewall-configure.md#use-the-azure-portal-to-manage-server-level-ip-firewall-rules).

Подготовьте среду к работе с Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="grant-database-access-to-azure-ad-user"></a>Предоставление доступа к базе данных для пользователя Azure AD

Сначала включите аутентификацию Azure AD в базе данных SQL, назначив пользователя Azure AD администратором Active Directory сервера. Этот пользователь отличается от учетной записи Майкрософт, которую вы использовали для регистрации на подписку Azure. Это должен быть пользователь, которого вы создали, импортировали, синхронизировали или пригласили в Azure AD. Дополнительные сведения о разрешенных пользователях Azure AD см. в разделе [Возможности и ограничения Azure AD в базе данных SQL](../azure-sql/database/authentication-aad-overview.md#azure-ad-features-and-limitations).

Если в вашем клиенте Azure AD отсутствует пользователь, создайте его, выполнив действия, описанные в статье [Add or delete users using Azure Active Directory](../active-directory/fundamentals/add-users-azure-active-directory.md) (Добавление или удаление пользователей с помощью Azure Active Directory).

Найдите идентификатор объекта пользователя Azure AD с помощью [`az ad user list`](/cli/azure/ad/user#az_ad_user_list) и замените *\<user-principal-name>* . Результат сохраняется в переменной.

```azurecli-interactive
azureaduser=$(az ad user list --filter "userPrincipalName eq '<user-principal-name>'" --query [].objectId --output tsv)
```
> [!TIP]
> Чтобы просмотреть список всех имен участников-пользователей в Azure AD, запустите `az ad user list --query [].userPrincipalName`.
>

Добавьте этого пользователя Azure AD в качестве администратора Active Directory с помощью команды [`az sql server ad-admin create`](/cli/azure/sql/server/ad-admin#az_sql_server_ad_admin_create) в Cloud Shell. В следующей команде замените *\<server-name>* именем сервера (без суффикса `.database.windows.net`).

```azurecli-interactive
az sql server ad-admin create --resource-group myResourceGroup --server-name <server-name> --display-name ADMIN --object-id $azureaduser
```

Для получения дополнительных сведений о добавлении администратора Active Directory см. раздел [Подготовка администратора Azure Active Directory для сервера](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance)

## <a name="set-up-visual-studio"></a>Настройка Visual Studio

### <a name="windows-client"></a>Клиент Windows
Версия Visual Studio для Windows интегрирована с проверкой подлинности Azure AD. Чтобы включить разработку и отладку в Visual Studio, добавьте своего пользователя Azure AD в Visual Studio, выбрав в меню **Файл** > **Параметры учетной записи** и нажмите **Добавить учетную запись**.

Чтобы настроить пользователя Azure AD для службы аутентификации Azure, выберите в меню **Инструменты** > **Параметры**, а затем выберите **Служба аутентификация Azure** > **Выбор учетной записи**. Выберите добавленного пользователя Azure AD и нажмите **ОК**.

Теперь вы готовы разрабатывать и отлаживать свое приложение с базой данных SQL в качестве серверной части, используя аутентификацию Azure AD.

### <a name="macos-client"></a>Клиент macOS

Версия Visual Studio для Mac интегрирована с проверкой подлинности Azure AD. Но библиотека [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication), которая будет использоваться позже, может использовать маркеры из Azure CLI. Чтобы включить разработку и отладку в Visual Studio, сначала необходимо [установить Azure CLI](/cli/azure/install-azure-cli) на локальном компьютере.

После установки Azure CLI на локальном компьютере войдите в Azure CLI с помощью следующей команды, используя учетные данные пользователя Azure AD:

```azurecli
az login --allow-no-subscriptions
```
Теперь вы готовы разрабатывать и отлаживать свое приложение с базой данных SQL в качестве серверной части, используя аутентификацию Azure AD.

## <a name="modify-your-project"></a>Изменение проекта

Действия, которые необходимо выполнить для настройки проекта, зависят от типа самого проекта: ASP.NET или ASP.NET Core.

- [Изменение ASP.NET](#modify-aspnet)
- [Изменение ASP.NET Core](#modify-aspnet-core)

### <a name="modify-aspnet"></a>Изменение ASP.NET

В Visual Studio откройте консоль диспетчера пакетов и добавьте пакет NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication):

```powershell
Install-Package Microsoft.Azure.Services.AppAuthentication -Version 1.4.0
```

Работая в верхней части файла в *Web.config*, внесите следующие изменения:

- В `<configSections>` добавьте следующее объявление разделов в

    ```xml
    <section name="SqlAuthenticationProviders" type="System.Data.SqlClient.SqlAuthenticationProviderConfigurationSection, System.Data, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
    ```

- ниже закрывающийся тег `</configSections>`, а также добавьте следующий код XML для `<SqlAuthenticationProviders>`.

    ```xml
    <SqlAuthenticationProviders>
      <providers>
        <add name="Active Directory Interactive" type="Microsoft.Azure.Services.AppAuthentication.SqlAppAuthenticationProvider, Microsoft.Azure.Services.AppAuthentication" />
      </providers>
    </SqlAuthenticationProviders>
    ```    

- Найдите строку подключения с именем `MyDbConnection` и замените ее значение `connectionString` на `"server=tcp:<server-name>.database.windows.net;database=<db-name>;UID=AnyString;Authentication=Active Directory Interactive"`. Замените _\<server-name>_ и _\<db-name>_ именем вашего сервера и именем базы данных.

> [!NOTE]
> Только что зарегистрированный класс SqlAuthenticationProvider основывается на библиотеке AppAuthentication, установленной ранее. По умолчанию он использует назначенное системой удостоверение. Чтобы использовать назначенное пользователем удостоверение, необходимо предоставить дополнительную конфигурацию. Сведения о библиотеке AppAuthentication см. в разделе [Поддержка строки подключения](/dotnet/api/overview/azure/service-to-service-authentication#connection-string-support).

Это все, что необходимо для подключения к Базе данных SQL. При отладке в Visual Studio ваш код использует имя пользователя Azure AD, указанное в разделе [Настройка Visual Studio](#set-up-visual-studio). Позже вы настроите Базу данных SQL, чтобы разрешить установку подключения с использованием управляемого удостоверения вашего приложения Службы приложений.

Введите `Ctrl+F5`, чтобы снова запустить приложение. То же приложение CRUD в вашем браузере теперь подключается к базе данных SQL Azure напрямую, используя аутентификацию Azure AD. Эта настройка позволяет запускать перенос базы данных из Visual Studio.

### <a name="modify-aspnet-core"></a>Изменение ASP.NET Core

В Visual Studio откройте консоль диспетчера пакетов и добавьте пакет NuGet [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication):

```powershell
Install-Package Microsoft.Azure.Services.AppAuthentication -Version 1.4.0
```

В [руководстве по созданию приложения ASP.NET Core и Базы данных SQL](tutorial-dotnetcore-sqldb-app.md) строка подключения `MyDbConnection` вообще не применяется, так как локальная среда разработки использует файл базы данных Sqlite, а производственная среда Azure — строку подключения из Службы приложений. При проверке подлинности Active Directory обе среды должны использовать одну и ту же строку подключения. В файле *appsettings.json* замените значение строки подключения `MyDbConnection` на следующий фрагмент кода:

```json
"Server=tcp:<server-name>.database.windows.net,1433;Database=<database-name>;"
```

Затем предоставьте контекст базы данных Entity Framework с маркером доступа для Базы данных SQL. В файле *Data \ MyDatabaseContext.cs* добавьте следующий код в фигурные скобки пустого конструктора `MyDatabaseContext (DbContextOptions<MyDatabaseContext> options)`:

```csharp
var connection = (SqlConnection)Database.GetDbConnection();
connection.AccessToken = (new Microsoft.Azure.Services.AppAuthentication.AzureServiceTokenProvider()).GetAccessTokenAsync("https://database.windows.net/").Result;
```

> [!NOTE]
> Этот демонстрационный код является синхронным для ясности и простоты.

Это все, что необходимо для подключения к Базе данных SQL. При отладке в Visual Studio ваш код использует имя пользователя Azure AD, указанное в разделе [Настройка Visual Studio](#set-up-visual-studio). Позже вы настроите Базу данных SQL, чтобы разрешить установку подключения с использованием управляемого удостоверения вашего приложения Службы приложений. Класс `AzureServiceTokenProvider` кэширует токен в памяти и извлекает его из Azure AD прямо перед истечением срока действия. Для обновления токена не требуется пользовательский код.

> [!TIP]
> Если настроенный вами пользователь Azure AD имеет доступ к нескольким клиентам, вызовите `GetAccessTokenAsync("https://database.windows.net/", tenantid)` с идентификатором нужного клиента, чтобы получить соответствующий маркер доступа.

Введите `Ctrl+F5`, чтобы снова запустить приложение. То же приложение CRUD в вашем браузере теперь подключается к базе данных SQL Azure напрямую, используя аутентификацию Azure AD. Эта настройка позволяет запускать перенос базы данных из Visual Studio.

## <a name="use-managed-identity-connectivity"></a>Использование соединения управляемого удостоверения

Затем настройте приложение службы приложений для подключения к базе данных SQL с назначенным системой управляемым удостоверением.

> [!NOTE]
> Хотя инструкции в этом разделе касаются назначенного системой удостоверения, можно так же легко использовать удостоверение, назначенное пользователем. Для этого потребуется изменить `az webapp identity assign command`, чтобы назначить нужное удостоверение, назначенное пользователем. Затем при создании пользователя SQL вместо имени сайта следует использовать имя ресурса назначенного пользователем удостоверения.

### <a name="enable-managed-identity-on-app"></a>Включение управляемого удостоверения в приложении

Чтобы включить управляемое удостоверение для приложения Azure, используйте команду [az webapp identity assign](/cli/azure/webapp/identity#az_webapp_identity_assign) в Cloud Shell. В следующей команде замените *\<app-name>* .

```azurecli-interactive
az webapp identity assign --resource-group myResourceGroup --name <app-name>
```

Ниже приведен пример выхода.

<pre>
{
  "additionalProperties": {},
  "principalId": "21dfa71c-9e6f-4d17-9e90-1d28801c9735",
  "tenantId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
  "type": "SystemAssigned"
}
</pre>

### <a name="grant-permissions-to-managed-identity"></a>Предоставление разрешений управляемому удостоверению

> [!NOTE]
> При необходимости можно добавить удостоверение в [группу Azure AD](../active-directory/fundamentals/active-directory-manage-groups.md), а затем предоставить доступ к Базе данных SQL группе Azure AD, а не удостоверению. Например, следующие команды добавляют управляемое удостоверение из предыдущего шага в новую группу с именем _myAzureSQLDBAccessGroup_:
> 
> ```azurecli-interactive
> groupid=$(az ad group create --display-name myAzureSQLDBAccessGroup --mail-nickname myAzureSQLDBAccessGroup --query objectId --output tsv)
> msiobjectid=$(az webapp identity show --resource-group myResourceGroup --name <app-name> --query principalId --output tsv)
> az ad group member add --group $groupid --member-id $msiobjectid
> az ad group member list -g $groupid
> ```
>

В Cloud Shell войдите в базу данных SQL с помощью команды SQLCMD. Замените _\<server-name>_ именем сервера, _\<db-name>_ именем базы данных, которое использует приложение, а _\<aad-user-name>_ и _\<aad-password>_ учетными данными пользователя Azure AD.

```bash
sqlcmd -S <server-name>.database.windows.net -d <db-name> -U <aad-user-name> -P "<aad-password>" -G -l 30
```

В командной строке SQL нужной базы данных выполните следующие команды, чтобы предоставить приложению требуемые разрешения. Например, 

```sql
CREATE USER [<identity-name>] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [<identity-name>];
ALTER ROLE db_datawriter ADD MEMBER [<identity-name>];
ALTER ROLE db_ddladmin ADD MEMBER [<identity-name>];
GO
```

*\<identity-name>*  — имя управляемого удостоверения в Azure AD. Если удостоверение назначено системой, имя всегда совпадает с именем приложения Службы приложений. Чтобы предоставить разрешения для группы Azure AD, используйте отображаемое имя группы (например, *myAzureSQLDBAccessGroup*).

Введите `EXIT`, чтобы вернуться в командную строку Cloud Shell.

> [!NOTE]
> Серверные службы управляемых удостоверений также [поддерживают кэш маркеров](overview-managed-identity.md#obtain-tokens-for-azure-resources), который обновляет маркер целевого ресурса только по истечении срока его действия. Если вы допустите ошибку при настройке разрешений Базы данных SQL и попытаетесь изменить разрешения *после* попытки получить маркер в приложении, вы получите новый маркер с обновленными разрешениями только после того, как истечет срок действия кэшированного маркера.

> [!NOTE]
> AAD не поддерживается для локального экземпляра SQL Server, в том числе для файлов MSI. 

### <a name="modify-connection-string"></a>Изменение строки подключения

Помните, что те же изменения, которые вы внесли в файл *Web.config* или *appsettings.json*, работают с управляемым удостоверением, поэтому единственное, что нужно сделать, — это удалить существующую строку подключения в Службе приложений, которую было создано Visual Studio во время первого развертывая вашего приложения. Используйте в следующих командах, но замените *\<app-name>* именем вашего приложения.

```azurecli-interactive
az webapp config connection-string delete --resource-group myResourceGroup --name <app-name> --setting-names MyDbConnection
```

## <a name="publish-your-changes"></a>Публикация изменений

Теперь требуется опубликовать изменения в Azure.

**Если ранее вы изучали [руководство по созданию приложения ASP.NET в Azure с подключением к Базе данных SQL](app-service-web-tutorial-dotnet-sqldatabase.md)** , опубликуйте изменения в Visual Studio. В **обозревателе решений** щелкните правой кнопкой мыши проект **DotNetAppSqlDb** и выберите **Опубликовать**.

![Публикация в обозревателе решений](./media/app-service-web-tutorial-dotnet-sqldatabase/solution-explorer-publish.png)

На странице публикации щелкните **Опубликовать**. 

**Если ранее вы изучали [руководство по созданию приложения ASP.NET Core и Базы данных SQL в Службе приложений Azure](tutorial-dotnetcore-sqldb-app.md)** , опубликуйте свои изменения с помощью Git, выполнив следующие команды:

```bash
git commit -am "configure managed identity"
git push azure main
```

Когда на новой веб-странице отображается список дел, ваше приложение подключается к базе данных с управляемым удостоверением.

![Приложение Azure после включения Code First Migration](./media/app-service-web-tutorial-dotnet-sqldatabase/this-one-is-done.png)

Теперь вы должны иметь возможность редактировать список дел по-прежнему.

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * Включение управляемых удостоверений
> * Предоставление управляемому удостоверению доступа к Базе данных SQL
> * Настройте Entity Framework для использования аутентификации Azure AD с базой данных SQL
> * Подключитесь к базе данных SQL из Visual Studio с использованием аутентификации Azure AD

Перейдите к следующему руководству, чтобы научиться сопоставлять пользовательские DNS-имена с веб-приложением.

> [!div class="nextstepaction"]
> [Сопоставление существующего настраиваемого DNS-имени со Службой приложений Azure](app-service-web-tutorial-custom-domain.md)