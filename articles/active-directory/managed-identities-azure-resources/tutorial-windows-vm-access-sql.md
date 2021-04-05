---
title: Руководство по получению доступа к Базе данных SQL Azure для Windows в Azure AD с помощью управляемого удостоверения
description: Из этого руководства вы узнаете, как получить доступ к Базе данных SQL Azure с помощью назначаемого системой управляемого удостоверения виртуальной машины Windows.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/14/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9f4f56ce9fa86dc27b77ad6b463479d13c8e4e7d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91856518"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-sql"></a>Руководство по Использование назначаемого системой управляемого удостоверения на виртуальной машине Windows для доступа к SQL Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве описывается, как получить доступ к Базе данных SQL Azure с помощью назначаемого системой удостоверения виртуальной машины Windows. Управляемыми удостоверениями службы автоматически управляет Azure. Они позволяют проходить проверку подлинности в службах, поддерживающих аутентификацию Azure AD, без указания учетных данных в коде. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * Предоставление виртуальной машине доступа к Базе данных SQL Azure
> * Включение аутентификации Azure AD
> * Создание в базе данных автономного пользователя, который представляет назначаемое системой удостоверение виртуальной машины
> * Получение маркера доступа с использованием удостоверения виртуальной машины и отправка запроса в Базу данных SQL Azure с его помощью

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="enable"></a>Включить

[!INCLUDE [msi-tut-enable](../../../includes/active-directory-msi-tut-enable.md)]

## <a name="grant-access"></a>Предоставление доступа

Чтобы предоставить виртуальной машине доступ к Базе данных SQL Azure, можно использовать существующий [логический сервер SQL Server](../../azure-sql/database/logical-servers.md) или создать новый. Чтобы создать сервер и базу данных с помощью портала Azure, следуйте указаниям в статье [Создание базы данных SQL Azure на портале Azure](../../azure-sql/database/single-database-create-quickstart.md). В [документации по SQL Azure](/azure/sql-database/) также есть краткие руководства, описывающие использование Azure CLI и Azure PowerShell.

Предоставление виртуальной машине доступа к базе данных выполняется в два этапа:

1. Включение аутентификации Azure AD для сервера.
2. Создание в базе данных **автономного пользователя**, который представляет назначаемое системой удостоверение виртуальной машины.

### <a name="enable-azure-ad-authentication"></a>Включение аутентификации Azure AD

**Для [настройки аутентификации Azure AD](../../azure-sql/database/authentication-aad-configure.md) выполните указанные ниже действия.**

1. На портале Azure выберите **SQL servers** (Серверы SQL Server) на левой панели навигации.
2. Щелкните сервер SQL, для которого нужно включить аутентификацию Azure AD.
3. В разделе колонки **Параметры** щелкните **Администратор Active Directory**.
4. В командной строке щелкните **Задать администратора**.
5. Выберите учетную запись пользователя Azure AD, которую необходимо сделать администратором сервера, а затем нажмите кнопку **Выбрать**.
6. На панели команд нажмите кнопку **Сохранить**.

### <a name="create-contained-user"></a>Создание автономного пользователя

В этом разделе показано, как в базе данных создать автономного пользователя, который представляет назначаемое системой удостоверение виртуальной машины. На этом шаге вам потребуется [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS). Прежде чем начать, советуем ознакомиться со следующими статьями, содержащими общие сведения об интеграции Azure AD:

- [Универсальная аутентификация для Базы данных SQL и Azure Synapse Analytics (поддержка SSMS для MFA)](../../azure-sql/database/authentication-mfa-ssms-overview.md)
- [Настройка аутентификации Azure Active Directory для Базы данных SQL или Azure Synapse Analytics](../../azure-sql/database/authentication-aad-configure.md)

Для базы данных SQL в AAD требуются уникальные отображаемые имена. Поэтому учетные записи AAD, например пользователей, групп и субъектов-служб (приложения), и имена виртуальных машин, которые включены для управляемого удостоверения, должны иметь уникальные отображаемые имена в AAD. База данных SQL проверяет отображаемое имя AAD при создании таких пользователей на языке T-SQL. Если имя не является уникальным, команда завершается ошибкой и отображается запрос на ввод уникального отображаемого имени AAD для этой учетной записи.

**Чтобы создать автономного пользователя, сделайте следующее:**

1. Запустите среду SQL Server Management Studio.
2. В диалоговом окне **Подключение к серверу** в поле **Имя сервера** введите имя сервера.
3. В поле **Authentication** (Аутентификация) выберите **Active Directory — универсальная с поддержкой MFA**.
4. В поле **Имя пользователя** введите имя учетной записи Azure AD, установленной в качестве администратора сервера, например helen@woodgroveonline.com.
5. Щелкните **Параметры**.
6. В поле **Подключение к базе данных** введите имя несистемной базы данных, которую требуется настроить.
7. Нажмите кнопку **Соединить**. Завершите процесс входа в систему.
8. В **обозревателе объектов** разверните папку **Базы данных**.
9. Щелкните правой кнопкой мыши пользовательскую базу данных и выберите **Создать запрос**.
10. В окне запроса введите следующую строку и на панели инструментов нажмите кнопку **Выполнить**.

    > [!NOTE]
    > В приведенной ниже команде `VMName` — это имя виртуальной машины, для которой вы ранее настроили назначаемое системой удостоверение.

    ```sql
    CREATE USER [VMName] FROM EXTERNAL PROVIDER
    ```

    В результате выполнения команды должен быть создан автономный пользователь для назначаемого системой удостоверения виртуальной машины.
11. Очистите окно запроса, введите следующую строку и на панели инструментов нажмите кнопку **Выполнить**.

    > [!NOTE]
    > В приведенной ниже команде `VMName` — это имя виртуальной машины, для которой вы ранее настроили назначаемое системой удостоверение.

    ```sql
    ALTER ROLE db_datareader ADD MEMBER [VMName]
    ```

    В результате выполнения команды автономному пользователю должна быть предоставлена возможность считывания всей базы данных.

Теперь код, запущенный на виртуальной машине, может с помощью назначаемого системой управляемого удостоверения получить маркер и использовать его для аутентификации на сервере.

## <a name="access-data"></a>Доступ к данным

В этом разделе показано, как получить маркер доступа с помощью назначаемого системой управляемого удостоверения виртуальной машины и использовать этот маркер для обращения к SQL Azure. В SQL Azure реализована поддержка аутентификации Azure AD, поэтому поддерживается непосредственный прием маркеров доступа, полученных с использованием управляемого удостоверения. Для создания подключения к SQL используется метод на основе **маркера доступа**. Он реализуется как часть интеграции SQL Azure с Azure AD и отличается от указания учетных данных в строке подключения.

Ниже приведен пример кода .NET для установки подключения к SQL с помощью маркера доступа. Этот код должен выполняться на виртуальной машине, чтобы обеспечить возможность доступа к конечной точке назначаемого системой управляемого удостоверения виртуальной машины. Для использования метода с маркером доступа требуется **.NET Framework 4.6** или более поздней версии либо **.NET Core 2.2** или более поздней версии. Замените значения AZURE-SQL-SERVERNAME и DATABASE соответствующим образом. Обратите внимание на то, что идентификатор ресурса для SQL Azure — `https://database.windows.net/`.

```csharp
using System.Net;
using System.IO;
using System.Data.SqlClient;
using System.Web.Script.Serialization;

//
// Get an access token for SQL.
//
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://database.windows.net/");
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
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

//
// Open a connection to the server using the access token.
//
if (accessToken != null) {
    string connectionString = "Data Source=<AZURE-SQL-SERVERNAME>; Initial Catalog=<DATABASE>;";
    SqlConnection conn = new SqlConnection(connectionString);
    conn.AccessToken = accessToken;
    conn.Open();
}
```

Кроме того, вы можете быстро протестировать сквозную настройку с помощью PowerShell без необходимости писать и развертывать приложение на виртуальной машине.

1. На портале перейдите к разделу **Виртуальные машины**, выберите свою виртуальную машину Windows и в разделе **Обзор** щелкните **Подключить**.
2. Введите **имя пользователя** и **пароль**, добавленные при создании виртуальной машины Windows.
3. Теперь, когда создано **подключение к удаленному рабочему столу** с виртуальной машиной, откройте **PowerShell** в удаленном сеансе.
4. С помощью команды PowerShell `Invoke-WebRequest` выполните запрос к локальной конечной точке управляемого удостоверения, чтобы получить маркер доступа к SQL Azure.

    ```powershell
        $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' -Method GET -Headers @{Metadata="true"}
    ```

    Преобразуйте ответ из объекта JSON в объект PowerShell.

    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```

    Извлеките маркер доступа из ответа.

    ```powershell
    $AccessToken = $content.access_token
    ```

5. Установите подключение к серверу. Не забудьте заменить значения AZURE-SQL-SERVERNAME и DATABASE.

    ```powershell
    $SqlConnection = New-Object System.Data.SqlClient.SqlConnection
    $SqlConnection.ConnectionString = "Data Source = <AZURE-SQL-SERVERNAME>; Initial Catalog = <DATABASE>"
    $SqlConnection.AccessToken = $AccessToken
    $SqlConnection.Open()
    ```

    Затем создайте и отправьте запрос на сервер. Не забудьте заменить значение для TABLE.

    ```powershell
    $SqlCmd = New-Object System.Data.SqlClient.SqlCommand
    $SqlCmd.CommandText = "SELECT * from <TABLE>;"
    $SqlCmd.Connection = $SqlConnection
    $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
    $SqlAdapter.SelectCommand = $SqlCmd
    $DataSet = New-Object System.Data.DataSet
    $SqlAdapter.Fill($DataSet)
    ```

Проверьте значение `$DataSet.Tables[0]`, чтобы просмотреть результаты запроса.

## <a name="disable"></a>Отключить

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать назначаемое системой управляемое удостоверение и получить доступ к Базе данных SQL Azure. См. дополнительные сведения о Базе данных SQL Azure:

> [!div class="nextstepaction"]
> [База данных SQL Azure](../../azure-sql/database/sql-database-paas-overview.md)
