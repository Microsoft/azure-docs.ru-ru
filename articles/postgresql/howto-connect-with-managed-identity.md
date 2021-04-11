---
title: Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью управляемого удостоверения
description: Сведения о подключении и проверке подлинности с помощью управляемого удостоверения для прохождения проверки подлинности в базе данных Azure для PostgreSQL
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: how-to
ms.date: 05/19/2020
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: d59b449a2dbf2ed3b1177db6e543de4c34a8ecb1
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105604164"
---
# <a name="connect-with-managed-identity-to-azure-database-for-postgresql"></a>Подключение к базе данных Azure для PostgreSQL с помощью управляемого удостоверения

В этой статье показано, как использовать назначаемое пользователем удостоверение для виртуальной машины Azure в целях доступа к серверу базы данных Azure для PostgreSQL. Управляемыми удостоверениями службы автоматически управляет Azure. Они позволяют проходить проверку подлинности в службах, поддерживающих аутентификацию Azure AD, без указания учетных данных в коде. 

Вы узнаете, как выполнять следующие задачи:
- Предоставление виртуальной машине доступа к серверу базы данных Azure для PostgreSQL
- Создание в базе данных пользователя, который представляет назначаемое пользователем удостоверение виртуальной машины
- Получение маркера доступа с использованием идентификатора виртуальной машины и отправка запроса на сервер базы данных Azure для PostgreSQL с его помощью
- Реализация получения маркера в примере приложения C#

## <a name="prerequisites"></a>Предварительные требования

- См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](../../articles/active-directory/managed-identities-azure-resources/overview.md). Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжить.
- Для выполнения требуемых операций создания ресурсов и управления ролями учетной записи нужно предоставить разрешения роли "Владелец" в соответствующей области (подписка или группа ресурсов). Если нуждаетесь в помощи с назначением ролей, прочитайте статью [Назначение ролей Azure с помощью портала Azure](../../articles/role-based-access-control/role-assignments-portal.md).
- Вам потребуется виртуальная машина Azure (например, на базе Ubuntu Linux), которую вы хотите использовать для доступа к базе данных с помощью управляемого удостоверения.
- Вам нужен сервер базы данных Azure для PostgreSQL, на котором настроена [проверка подлинности Azure AD](howto-configure-sign-in-aad-authentication.md)
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

## <a name="creating-a-postgresql-user-for-your-managed-identity"></a>Создание пользователя PostgreSQL для управляемого удостоверения

Теперь подключитесь к базе данных PostgreSQL в качестве пользователя с правами администратора Azure AD и выполните следующие инструкции SQL:

```sql
SET aad_validate_oids_in_tenant = off;
CREATE ROLE myuser WITH LOGIN PASSWORD 'CLIENT_ID' IN ROLE azure_ad_user;
```

Теперь управляемое удостоверение имеет доступ при проверке подлинности с помощью имени пользователя `myuser` (замените другим именем по своему усмотрению).

## <a name="retrieving-the-access-token-from-azure-instance-metadata-service"></a>Получение маркера доступа из Службы метаданных экземпляров Azure

Теперь приложение может получить маркер доступа из Службы метаданных экземпляров Azure и использовать его для проверки подлинности в базе данных.

Получение маркера осуществляется посредством отправки HTTP-запроса на адрес `http://169.254.169.254/metadata/identity/oauth2/token` и передачи следующих параметров:

* `api-version` = `2018-02-01`
* `resource` = `https://ossrdbms-aad.database.windows.net`
* `client_id` = `CLIENT_ID` (получено ранее)

В результате вы получите JSON с полем `access_token` — это длинное текстовое значение является маркером доступа управляемого удостоверения, который следует использовать в качестве пароля при подключении к базе данных.

В целях проверки можно выполнить в оболочке приведенные ниже команды. Обратите внимание, что требуется `curl`, `jq` и установленный клиент `psql`.

```bash
# Retrieve the access token
export PGPASSWORD=`curl -s 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fossrdbms-aad.database.windows.net&client_id=CLIENT_ID' -H Metadata:true | jq -r .access_token`

# Connect to the database
psql -h SERVER --user USER@SERVER DBNAME
```

Теперь вы подключены к настроенной ранее базе данных.

## <a name="connecting-using-managed-identity-in-c"></a>Подключение с использованием управляемого удостоверения в C#

В этом разделе показано, как получить маркер доступа с помощью назначаемого пользователем управляемого удостоверения виртуальной машины и использовать этот маркер для обращения к базе данных Azure для PostgreSQL. В базе данных Azure для PostgreSQL изначально реализована поддержка проверки подлинности Azure AD, поэтому она может напрямую принимать маркеры доступа, полученные с помощью управляемых удостоверений для ресурсов Azure. При создании подключения к PostgreSQL вы передаете маркер доступа в поле пароля.

Ниже приведен пример кода .NET для установки подключения к PostgreSQL с помощью маркера доступа. Этот код должен выполняться на виртуальной машине, чтобы получить доступ к конечной точке назначаемого пользователем управляемого удостоверения виртуальной машины. Для использования метода с маркером доступа требуется .NET Framework 4.6 или более поздней версии либо .NET Core 2.2 или более поздней версии. Замените значения HOST, USER, DATABASE и CLIENT_ID.

```csharp
using System;
using System.Net;
using System.IO;
using System.Collections;
using System.Collections.Generic;
using System.Text.Json;
using System.Text.Json.Serialization;
using Npgsql;

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

        static void Main(string[] args)
        {
            //
            // Get an access token for PostgreSQL.
            //
            Console.Out.WriteLine("Getting access token from Azure Instance Metadata service...");

            // Azure AD resource ID for Azure Database for PostgreSQL is https://ossrdbms-aad.database.windows.net/
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
            // Open a connection to the PostgreSQL server using the access token.
            //
            string connString =
                String.Format(
                    "Server={0}; User Id={1}; Database={2}; Port={3}; Password={4};SSLMode=Prefer",
                    Host,
                    User,
                    Database,
                    5432,
                    accessToken);

            using (var conn = new NpgsqlConnection(connString))
            {
                Console.Out.WriteLine("Opening connection using access token...");
                conn.Open();

                using (var command = new NpgsqlCommand("SELECT version()", conn))
                {

                    var reader = command.ExecuteReader();
                    while (reader.Read())
                    {
                        Console.WriteLine("\nConnected!\n\nPostgres version: {0}", reader.GetString(0));
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

Postgres version: PostgreSQL 11.6, compiled by Visual C++ build 1800, 64-bit
```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомление с общими концепциями [проверки подлинности Azure Active Directory с помощью базы данных Azure для PostgreSQL.](concepts-aad-authentication.md)
