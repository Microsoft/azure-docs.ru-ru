---
title: Подключение с помощью управляемого удостоверения — база данных для MySQL
description: Сведения о подключении и проверке подлинности с помощью управляемого удостоверения для прохождения проверки подлинности в базе данных Azure для MySQL
author: sunilagarwal
ms.author: sunila
ms.service: mysql
ms.topic: how-to
ms.date: 05/19/2020
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: f790e20c257c81418c6fcd5b14be957a6ef43b4a
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105612608"
---
# <a name="connect-with-managed-identity-to-azure-database-for-mysql"></a>Подключение к базе данных Azure для MySQL с помощью управляемого удостоверения

В этой статье показано, как использовать назначаемое пользователем удостоверение для виртуальной машины Azure в целях доступа к серверу базы данных Azure для MySQL. Управляемыми удостоверениями службы автоматически управляет Azure. Они позволяют проходить проверку подлинности в службах, поддерживающих аутентификацию Azure AD, без указания учетных данных в коде. 

Вы узнаете, как выполнять следующие задачи:

- Предоставление виртуальной машине доступа к серверу базы данных Azure для MySQL
- Создание в базе данных пользователя, который представляет назначаемое пользователем удостоверение виртуальной машины
- Получение маркера доступа с использованием идентификатора виртуальной машины и отправка запроса на сервер базы данных Azure для MySQL с его помощью.
- Реализация получения маркера в примере приложения C#

> [!IMPORTANT]
> Подключение с управляемым удостоверением доступно только для MySQL 5,7 и более новых версий.

## <a name="prerequisites"></a>Предварительные требования

- См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](../../articles/active-directory/managed-identities-azure-resources/overview.md). Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжить.
- Для выполнения требуемых операций создания ресурсов и управления ролями учетной записи нужно предоставить разрешения роли "Владелец" в соответствующей области (подписка или группа ресурсов). Если нуждаетесь в помощи с назначением ролей, прочитайте статью [Назначение ролей Azure с помощью портала Azure](../../articles/role-based-access-control/role-assignments-portal.md).
- Вам потребуется виртуальная машина Azure (например, на базе Ubuntu Linux), которую вы хотите использовать для доступа к базе данных с помощью управляемого удостоверения.
- Вам нужен сервер базы данных Azure для MySQL, на котором настроена [проверка подлинности Azure AD](howto-configure-sign-in-azure-ad-authentication.md)
- Для выполнения примера C# сначала завершите руководство по [подключению к C#](connect-csharp.md)

## <a name="creating-a-user-assigned-managed-identity-for-your-vm"></a>Создание управляемого удостоверения, назначаемого пользователем, для виртуальной машины

Создайте в подписке удостоверение, используя команду [az identity create](/cli/azure/identity#az-identity-create). Вы можете использовать ту же группу ресурсов, в которой работает ваша виртуальная машина, или другую.

```azurecli-interactive
az identity create --resource-group myResourceGroup --name myManagedIdentity
```

Чтобы настроить удостоверение на следующих шагах, выполните команду [az identity show](/cli/azure/identity#az-identity-show) для сохранения идентификаторов ресурса и клиента удостоверения в переменных.

```azurecli
# Get resource ID of the user-assigned identity
resourceID=$(az identity show --resource-group myResourceGroup --name myManagedIdentity --query id --output tsv)

# Get client ID of the user-assigned identity
clientID=$(az identity show --resource-group myResourceGroup --name myManagedIdentity --query clientId --output tsv)
```

Теперь можно назначить виртуальной машине удостоверение, назначаемое пользователем, с помощью команды [az vm identity assign](/cli/azure/vm/identity#az-vm-identity-assign):

```azurecli
az vm identity assign --resource-group myResourceGroup --name myVM --identities $resourceID
```

Чтобы завершить установку, отобразите значение идентификатора клиента, которое потребуется в следующих шагах:

```bash
echo $clientID
```

## <a name="creating-a-mysql-user-for-your-managed-identity"></a>Создание пользователя MySQL для управляемого удостоверения

Теперь подключитесь к базе данных MySQL в качестве пользователя с правами администратора Azure AD и выполните следующие инструкции SQL:

```sql
SET aad_auth_validate_oids_in_tenant = OFF;
CREATE AADUSER 'myuser' IDENTIFIED BY 'CLIENT_ID';
```

Теперь управляемое удостоверение имеет доступ при проверке подлинности с помощью имени пользователя `myuser` (замените другим именем по своему усмотрению).

## <a name="retrieving-the-access-token-from-azure-instance-metadata-service"></a>Получение маркера доступа из Службы метаданных экземпляров Azure

Теперь приложение может получить маркер доступа из Службы метаданных экземпляров Azure и использовать его для проверки подлинности в базе данных.

Получение маркера осуществляется посредством отправки HTTP-запроса на адрес `http://169.254.169.254/metadata/identity/oauth2/token` и передачи следующих параметров:

* `api-version` = `2018-02-01`
* `resource` = `https://ossrdbms-aad.database.windows.net`
* `client_id` = `CLIENT_ID` (получено ранее)

В результате вы получите JSON с полем `access_token` — это длинное текстовое значение является маркером доступа управляемого удостоверения, который следует использовать в качестве пароля при подключении к базе данных.

В целях проверки можно выполнить в оболочке приведенные ниже команды. Обратите внимание, что требуется `curl`, `jq` и установленный клиент `mysql`.

```bash
# Retrieve the access token
accessToken=$(curl -s 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fossrdbms-aad.database.windows.net&client_id=CLIENT_ID' -H Metadata:true | jq -r .access_token)

# Connect to the database
mysql -h SERVER --user USER@SERVER --enable-cleartext-plugin --password=$accessToken
```

Теперь вы подключены к настроенной ранее базе данных.

## <a name="connecting-using-managed-identity-in-c"></a>Подключение с использованием управляемого удостоверения в C#

В этом разделе показано, как получить маркер доступа с помощью назначаемого пользователем управляемого удостоверения виртуальной машины и использовать этот маркер для обращения к базе данных Azure для MySQL. В базе данных Azure для MySQL изначально реализована поддержка проверки подлинности Azure AD, поэтому она может напрямую принимать маркеры доступа, полученные с помощью управляемых удостоверений для ресурсов Azure. При создании подключения к MySQL вы передаете маркер доступа в поле пароля.

Ниже приведен пример кода .NET для установки подключения к MySQL с помощью маркера доступа. Этот код должен выполняться на виртуальной машине, чтобы получить доступ к конечной точке назначаемого пользователем управляемого удостоверения виртуальной машины. Для использования метода с маркером доступа требуется .NET Framework 4.6 или более поздней версии либо .NET Core 2.2 или более поздней версии. Замените значения HOST, USER, DATABASE и CLIENT_ID.

```csharp
using System;
using System.Net;
using System.IO;
using System.Collections;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.Json.Serialization;
using System.Threading.Tasks;
using MySql.Data.MySqlClient;

namespace Driver
{
    class Script
    {
        // Obtain connection string information from the portal
        //
        private static string Host = "HOST";
        private static string User = "USER";
        private static string Database = "DATABASE";
        private static string ClientId = "CLIENT_ID";

        static async Task Main(string[] args)
        {
            //
            // Get an access token for MySQL.
            //
            Console.Out.WriteLine("Getting access token from Azure Instance Metadata service...");

            // Azure AD resource ID for Azure Database for MySQL is https://ossrdbms-aad.database.windows.net/
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fossrdbms-aad.database.windows.net&client_id=" + ClientId);
            request.Headers["Metadata"] = "true";
            request.Method = "GET";
            string accessToken = null;

            try
            {
                // Call managed identities for Azure resources endpoint.
                HttpWebResponse response = (HttpWebResponse)request.GetResponse();

                // Pipe response Stream to a StreamReader and extract access token.
                StreamReader streamResponse = new StreamReader(response.GetResponseStream());
                string stringResponse = streamResponse.ReadToEnd();
                var list = JsonSerializer.Deserialize<Dictionary<string, string>>(stringResponse);
                accessToken = list["access_token"];
            }
            catch (Exception e)
            {
                Console.Out.WriteLine("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
                System.Environment.Exit(1);
            }

            //
            // Open a connection to the MySQL server using the access token.
            //
            var builder = new MySqlConnectionStringBuilder
            {
                Server = Host,
                Database = Database,
                UserID = User,
                Password = accessToken,
                SslMode = MySqlSslMode.Required,
            };

            using (var conn = new MySqlConnection(builder.ConnectionString))
            {
                Console.Out.WriteLine("Opening connection using access token...");
                await conn.OpenAsync();

                using (var command = conn.CreateCommand())
                {
                    command.CommandText = "SELECT VERSION()";

                    using (var reader = await command.ExecuteReaderAsync())
                    {
                        while (await reader.ReadAsync())
                        {
                            Console.WriteLine("\nConnected!\n\nMySQL version: {0}", reader.GetString(0));
                        }
                    }
                }
            }
        }
    }
}
```

При выполнении эта команда выдает следующий результат:

```
Getting access token from Azure Instance Metadata service...
Opening connection using access token...

Connected!

MySQL version: 5.7.27
```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомление с общими концепциями [проверки подлинности Azure Active Directory с помощью базы данных Azure для MySQL.](concepts-azure-ad-authentication.md)
