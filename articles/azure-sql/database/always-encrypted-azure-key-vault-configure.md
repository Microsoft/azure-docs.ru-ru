---
title: Настройка Always Encrypted с помощью Azure Key Vault
description: В этом учебнике показано, как защитить конфиденциальные данные в базе данных SQL Azure с помощью шифрования данных при помощи мастера Always Encrypted в SQL Server Management Studio.
keywords: шифрование данных, ключ шифрования, шифрование в облаке
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: VanMSFT
ms.author: vanto
ms.reviewer: ''
ms.date: 11/02/2020
ms.openlocfilehash: 257abf03994c7006b1c3789174f550515dcd309a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94841467"
---
# <a name="configure-always-encrypted-by-using-azure-key-vault"></a>Настройка Always Encrypted с помощью Azure Key Vault 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb-sqlmi.md)]

В этой статье показано, как защитить конфиденциальные данные в базе данных SQL Azure с помощью шифрования данных при помощи [мастера Always encrypted](/sql/relational-databases/security/encryption/always-encrypted-wizard) в [SQL Server Management Studio (SSMS)](/sql/ssms/sql-server-management-studio-ssms). Она также содержит инструкции, с помощью которых вы узнаете, как сохранить каждый ключ шифрования в хранилище ключей Azure.

Always Encrypted — это технология шифрования данных, которая помогает защитить конфиденциальные данные на сервере во время перемещения между клиентом и сервером, а также во время использования данных. Always Encrypted гарантирует, что конфиденциальные данные никогда не отобразятся как открытый текст внутри системы баз данных. После настройки шифрования данных получить доступ к данным в виде открытого текста смогут только клиентские приложения и серверы приложений, у которых есть доступ к ключам. Дополнительные сведения см. в статье [Always Encrypted (ядро СУБД)](/sql/relational-databases/security/encryption/always-encrypted-database-engine).

После настройки функции Always Encrypted для базы данных вы создадите клиентское приложение на языке C# для работы с зашифрованными данными, используя Visual Studio.

Выполните действия, описанные в этой статье, и Узнайте, как настроить Always Encrypted для базы данных в базе данных SQL Azure или в SQL Управляемый экземпляр. Здесь вы научитесь:

- Использовать мастер настройки Always Encrypted в SSMS для создания [ключей Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine#Anchor_3).
  - Создавать [главный ключ столбца (CMK)](/sql/t-sql/statements/create-column-master-key-transact-sql).
  - Создавать [ключ шифрования столбца (CEK)](/sql/t-sql/statements/create-column-encryption-key-transact-sql).
- Создавать таблицу базы данных и шифровать столбцы.
- Создавать приложение, которое вставляет, выбирает и отображает данные зашифрованных столбцов.

## <a name="prerequisites"></a>Предварительные требования


- Учетная запись и подписка Azure. Если у вас ее нет, зарегистрируйтесь для получения [бесплатной пробной версии](https://azure.microsoft.com/pricing/free-trial/).
- База данных в [базе данных SQL Azure](single-database-create-quickstart.md) или [Azure SQL управляемый экземпляр](../managed-instance/instance-create-quickstart.md).
- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) версии 13.0.700.242 или более поздней версии.
- [NET Framework версии 4.6](/dotnet/framework/) или более поздней версии (на клиентском компьютере).
- [Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx).
- [Azure PowerShell](/powershell/azure/) или [Azure CLI](/cli/azure/install-azure-cli)

## <a name="enable-client-application-access"></a>Разрешить доступ к клиентскому приложению

Необходимо разрешить клиентскому приложению доступ к базе данных в базе данных SQL, настроив приложение Azure Active Directory (Azure AD) и скопировав *идентификатор приложения* и *ключ* , которые понадобятся для проверки подлинности приложения.

Чтобы получить *идентификатор приложения* и *ключ*, ознакомьтесь с процедурой в разделе [Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="create-a-key-vault-to-store-your-keys"></a>Создание хранилища ключей для хранения ваших ключей

Теперь, когда клиентское приложение настроено и у вас есть идентификатор приложения, пора создать хранилище ключей и настроить его политику доступа таким образом, чтобы ваше приложение могло обращаться к секретам в хранилище (ключам Always Encrypted). Для создания главного ключа столбца и настройки шифрования с помощью SQL Server Management Studio необходимы разрешения *create*, *get*, *list*, *sign*, *verify*, *wrapKey* и *unwrapKey*.

Можно быстро создать хранилище ключей, выполнив следующий сценарий. Подробное описание этих команд и дополнительные сведения о создании и настройке хранилища ключей см. в разделе [что такое Azure Key Vault?](../../key-vault/general/overview.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager (RM) по-прежнему поддерживается базой данных SQL Azure, но вся будущая разработка предназначена для модуля AZ. SQL. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

```powershell
$subscriptionName = '<subscriptionName>'
$userPrincipalName = '<username@domain.com>'
$applicationId = '<applicationId from AAD application>'
$resourceGroupName = '<resourceGroupName>' # use the same resource group name when creating your SQL Database below
$location = '<datacenterLocation>'
$vaultName = '<vaultName>'

Connect-AzAccount
$subscriptionId = (Get-AzSubscription -SubscriptionName $subscriptionName).Id
Set-AzContext -SubscriptionId $subscriptionId

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzKeyVault -VaultName $vaultName -ResourceGroupName $resourceGroupName -Location $location

Set-AzKeyVaultAccessPolicy -VaultName $vaultName -ResourceGroupName $resourceGroupName -PermissionsToKeys create,get,wrapKey,unwrapKey,sign,verify,list -UserPrincipalName $userPrincipalName
Set-AzKeyVaultAccessPolicy  -VaultName $vaultName  -ResourceGroupName $resourceGroupName -ServicePrincipalName $applicationId -PermissionsToKeys get,wrapKey,unwrapKey,sign,verify,list
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
$subscriptionName = '<subscriptionName>'
$userPrincipalName = '<username@domain.com>'
$applicationId = '<applicationId from AAD application>'
$resourceGroupName = '<resourceGroupName>' # use the same resource group name when creating your database in Azure SQL Database below
$location = '<datacenterLocation>'
$vaultName = '<vaultName>'

az login
az account set --subscription $subscriptionName

az group create --location $location --name $resourceGroupName

az keyvault create --name $vaultName --resource-group $resourceGroupName --location $location

az keyvault set-policy --name $vaultName --key-permissions create get list sign unwrapKey verify wrapKey --resource-group $resourceGroupName --upn $userPrincipalName
az keyvault set-policy --name $vaultName --key-permissions get list sign unwrapKey verify wrapKey --resource-group $resourceGroupName --spn $applicationId
```

---

## <a name="connect-with-ssms"></a>Подключение к SSMS

Откройте SQL Server Management Studio (SSMS) и подключитесь к серверу или под управлением базы данных.

1. Откройте SSMS. (Перейдите к разделу **Подключение**  >  **Ядро СУБД** , чтобы открыть окно **Подключение к серверу** , если оно не открыто.)

2. Введите имя сервера или имя экземпляра и учетные данные. 

    ![Копирование строки подключения](./media/always-encrypted-azure-key-vault-configure/ssms-connect.png)

Если откроется окно **Новое правило брандмауэра** , войдите в Azure, и SSMS создаст новое правило брандмауэра для вас.

## <a name="create-a-table"></a>Создание таблицы

В этом разделе вы создадите таблицу для хранения данных пациентов. Изначально в ней будет отсутствовать шифрование — его вы настроите в следующем разделе.

1. Разверните узел **Базы данных**.
2. Щелкните правой кнопкой мыши базу данных и выберите команду **создать запрос**.
3. Вставьте следующий сценарий Transact-SQL в окно нового запроса и нажмите кнопку **Выполнить** .

```sql
CREATE TABLE [dbo].[Patients](
         [PatientId] [int] IDENTITY(1,1),
         [SSN] [char](11) NOT NULL,
         [FirstName] [nvarchar](50) NULL,
         [LastName] [nvarchar](50) NULL,
         [MiddleName] [nvarchar](50) NULL,
         [StreetAddress] [nvarchar](50) NULL,
         [City] [nvarchar](50) NULL,
         [ZipCode] [char](5) NULL,
         [State] [char](2) NULL,
         [BirthDate] [date] NOT NULL
         PRIMARY KEY CLUSTERED ([PatientId] ASC) ON [PRIMARY] );
GO
```

## <a name="encrypt-columns-configure-always-encrypted"></a>Шифрование столбцов (настройка Always Encrypted)

В SSMS есть мастер, с помощью которого можно легко настроить функцию Always Encrypted. Он позволяет настроить главный ключ столбца, ключ шифрования столбца и зашифрованные столбцы.

1. Разверните узел **базы данных**  >    >  **таблицы** курса.
2. Щелкните правой кнопкой мыши таблицу **Patients** и выберите пункт **Зашифровать столбцы**, чтобы открыть мастер настройки Always Encrypted.

    ![Снимок экрана, на котором выделены столбцы шифрования... пункт меню.](./media/always-encrypted-azure-key-vault-configure/encrypt-columns.png)

Мастер настройки Always Encrypted содержит следующие разделы: **Выбор столбца**, **Настройка главного ключа**, **Проверка** и **Сводка**.

### <a name="column-selection"></a>Выполните действия на странице Выбор столбцов.

На странице **Введение** нажмите кнопку **Далее**, чтобы открыть страницу **Выбор столбца**. На этой странице можно будет выбрать столбцы для шифрования, [тип шифрования и используемый ключ шифрования столбца (CEK)](/sql/relational-databases/security/encryption/always-encrypted-wizard#Anchor_2) .

Для каждого пациента необходимо зашифровать данные в столбцах **SSN** и **BirthDate**. Для столбца SSN будет использоваться детерминированное шифрование, которое поддерживает уточняющие запросы на соответствие условию, операции объединения и группировки, а для столбца BirthDate — случайное шифрование, которое не поддерживает какие-либо операции.

Задайте для параметра **Тип шифрования** столбца SSN значение **Детерминированное**, а для столбца BirthDate — **Случайное**. Щелкните **Далее**.

![Шифрование столбцов…](./media/always-encrypted-azure-key-vault-configure/column-selection.png)

### <a name="master-key-configuration"></a>Настройка главного ключа

На странице **Настройка главного ключа** можно настроить CMK и выбрать поставщик хранилища ключей, в котором будет находиться CMK. В настоящее время CMK можно хранить в хранилище сертификатов Windows, хранилище ключей Azure или аппаратном модуле безопасности (HSM).

В этом руководстве показано, как сохранить ключи в хранилище ключей Azure.

1. Выберите **Хранилище ключей Azure**.
2. Из раскрывающегося списка выберите нужное хранилище ключей.
3. Щелкните **Далее**.

![Настройка главного ключа](./media/always-encrypted-azure-key-vault-configure/master-key-configuration.png)

### <a name="validation"></a>Проверка

Можно зашифровать столбцы сейчас или сохранить сценарий PowerShell и выполнить его позже. Для целей этого руководства выберите **Перейти к завершению** и нажмите кнопку **Далее**.

### <a name="summary"></a>Итоги

Убедитесь, что все параметры настроены правильно, и нажмите кнопку **Готово** , чтобы завершить настройку Always Encrypted.

![На снимке экрана показана страница результатов с задачами, помеченными как пройденные.](./media/always-encrypted-azure-key-vault-configure/summary.png)

### <a name="verify-the-wizards-actions"></a>Проверка действий мастера

После завершения работы мастера для базы данных будет настроена функция Always Encrypted. Мастер выполнил следующие действия:

- Создал главный ключ столбца и сохранил его в хранилище ключей Azure.
- Создал ключ шифрования столбца и сохранил его в хранилище ключей Azure.
- Настройка шифрования для выбранных столбцов. Сейчас в таблице Patients нет данных, но как только они появятся, они будут зашифрованы в выбранных столбцах.

Чтобы проверить создание ключей в SSMS **, разверните узел**  >  **Безопасность**  >  **Always encrypted ключи**.

## <a name="create-a-client-application-that-works-with-the-encrypted-data"></a>Создание клиентского приложения, которое работает с зашифрованными данными

Теперь, когда функция Always Encrypted настроена, мы можем создать приложение, которое выполняет c зашифрованными столбцами операции *вставки* и *выборки*.  

> [!IMPORTANT]
> При передаче открытого текста на сервер со столбцами с постоянным шифрованием приложение должно использовать объекты [SqlParameter](/dotnet/api/system.data.sqlclient.sqlparameter) . Передача значений литералов без использования объектов SqlParameter приведет к возникновению исключения.

1. Откройте Visual Studio и создайте **консольное приложение** C# (Visual Studio 2015 и более ранней версии) или **консольное приложение (.NET Framework)** (Visual Studio 2017 и более поздней версии). Убедитесь, что для проекта используется платформа **.NET Framework** версии 4.6 или более поздней.
2. Укажите имя проекта **AlwaysEncryptedConsoleApp** и нажмите кнопку **ОК**.
3. Установите следующие пакеты NuGet, выбрав **инструменты**  >  **Диспетчер пакетов NuGet**  >  **консоль диспетчера пакетов**.

Выполните следующие две строки кода в консоли диспетчера пакетов:

   ```powershell
   Install-Package Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```

## <a name="modify-your-connection-string-to-enable-always-encrypted"></a>Изменение строки подключения для активации функции постоянного шифрования

В этом разделе объясняется, как включить функцию Always Encrypted в строке подключения к базе данных.

Чтобы включить функцию Always Encrypted, необходимо добавить ключевое слово **Column Encryption Setting** в строку подключения и задать для него значение **Enabled**.

Это можно задать непосредственно в строке подключения или с помощью [SqlConnectionStringBuilder](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder). В разделе ниже показано, как использовать **SqlConnectionStringBuilder** для примера приложения.

### <a name="enable-always-encrypted-in-the-connection-string"></a>Активация функции постоянного шифрования в строке подключения

Добавьте в строку подключения следующее ключевое слово.

   `Column Encryption Setting=Enabled`

### <a name="enable-always-encrypted-with-sqlconnectionstringbuilder"></a>Включение функции Always Encrypted с помощью SqlConnectionStringBuilder

В коде ниже показано, как активировать функцию Always Encrypted, указав параметр [SqlConnectionStringBuilder.ColumnEncryptionSetting](/dotnet/api/system.data.sqlclient.sqlconnectionstringbuilder.columnencryptionsetting) со значением [Enabled](/dotnet/api/system.data.sqlclient.sqlconnectioncolumnencryptionsetting).

```csharp
// Instantiate a SqlConnectionStringBuilder.
SqlConnectionStringBuilder connStringBuilder = new SqlConnectionStringBuilder("replace with your connection string");

// Enable Always Encrypted.
connStringBuilder.ColumnEncryptionSetting = SqlConnectionColumnEncryptionSetting.Enabled;
```

## <a name="register-the-azure-key-vault-provider"></a>Регистрация поставщика хранилища ключей Azure
В следующем коде показано, как зарегистрировать поставщик хранилища ключей Azure с помощью драйвера ADO.NET.

```csharp
private static ClientCredential _clientCredential;

static void InitializeAzureKeyVaultProvider() {
    _clientCredential = new ClientCredential(applicationId, clientKey);

    SqlColumnEncryptionAzureKeyVaultProvider azureKeyVaultProvider = new SqlColumnEncryptionAzureKeyVaultProvider(GetToken);

    Dictionary<string, SqlColumnEncryptionKeyStoreProvider> providers = new Dictionary<string, SqlColumnEncryptionKeyStoreProvider>();

    providers.Add(SqlColumnEncryptionAzureKeyVaultProvider.ProviderName, azureKeyVaultProvider);
    SqlConnection.RegisterColumnEncryptionKeyStoreProviders(providers);
}
```

## <a name="always-encrypted-sample-console-application"></a>Пример консольного приложения с функцией постоянного шифрования

В этом примере показано, как:

- Изменить строку подключения для активации функции постоянного шифрования.
- Зарегистрируйте хранилище ключей Azure в качестве поставщика хранилища ключей приложения.  
- Вставить данные в зашифрованные столбцы.
- Выбрать запись, выполнив фильтрацию по конкретному значению в зашифрованном столбце.

Замените содержимое *Program.cs* приведенным ниже кодом. Замените строку подключения для глобальной переменной connectionString, расположенную в строке прямо над методом Main, действительной строкой подключения с портала Azure. Это единственное изменение, которое необходимо внести в этот код.

Запустите приложение, чтобы увидеть функцию Always Encrypted в действии.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Data;
using System.Data.SqlClient;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.SqlServer.Management.AlwaysEncrypted.AzureKeyVaultProvider;

namespace AlwaysEncryptedConsoleAKVApp {
    class Program {
        // Update this line with your Clinic database connection string from the Azure portal.
        static string connectionString = @"<connection string from the portal>";
        static string applicationId = @"<application ID from your AAD application>";
        static string clientKey = "<key from your AAD application>";

        static void Main(string[] args) {
            InitializeAzureKeyVaultProvider();

            Console.WriteLine("Signed in as: " + _clientCredential.ClientId);

            Console.WriteLine("Original connection string copied from the Azure portal:");
            Console.WriteLine(connectionString);

            // Create a SqlConnectionStringBuilder.
            SqlConnectionStringBuilder connStringBuilder =
                new SqlConnectionStringBuilder(connectionString);

            // Enable Always Encrypted for the connection.
            // This is the only change specific to Always Encrypted
            connStringBuilder.ColumnEncryptionSetting =
                SqlConnectionColumnEncryptionSetting.Enabled;

            Console.WriteLine(Environment.NewLine + "Updated connection string with Always Encrypted enabled:");
            Console.WriteLine(connStringBuilder.ConnectionString);

            // Update the connection string with a password supplied at runtime.
            Console.WriteLine(Environment.NewLine + "Enter server password:");
            connStringBuilder.Password = Console.ReadLine();

            // Assign the updated connection string to our global variable.
            connectionString = connStringBuilder.ConnectionString;

            // Delete all records to restart this demo app.
            ResetPatientsTable();

            // Add sample data to the Patients table.
            Console.Write(Environment.NewLine + "Adding sample patient data to the database...");

            InsertPatient(new Patient() {
                SSN = "999-99-0001",
                FirstName = "Orlando",
                LastName = "Gee",
                BirthDate = DateTime.Parse("01/04/1964")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0002",
                FirstName = "Keith",
                LastName = "Harris",
                BirthDate = DateTime.Parse("06/20/1977")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0003",
                FirstName = "Donna",
                LastName = "Carreras",
                BirthDate = DateTime.Parse("02/09/1973")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0004",
                FirstName = "Janet",
                LastName = "Gates",
                BirthDate = DateTime.Parse("08/31/1985")
            });
            InsertPatient(new Patient() {
                SSN = "999-99-0005",
                FirstName = "Lucy",
                LastName = "Harrington",
                BirthDate = DateTime.Parse("05/06/1993")
            });

            // Fetch and display all patients.
            Console.WriteLine(Environment.NewLine + "All the records currently in the Patients table:");

            foreach (Patient patient in SelectAllPatients()) {
                Console.WriteLine(patient.FirstName + " " + patient.LastName + "\tSSN: " + patient.SSN + "\tBirthdate: " + patient.BirthDate);
            }

            // Get patients by SSN.
            Console.WriteLine(Environment.NewLine + "Now lets locate records by searching the encrypted SSN column.");

            string ssn;

            // This very simple validation only checks that the user entered 11 characters.
            // In production be sure to check all user input and use the best validation for your specific application.
            do {
                Console.WriteLine("Please enter a valid SSN (ex. 999-99-0003):");
                ssn = Console.ReadLine();
            } while (ssn.Length != 11);

            // The example allows duplicate SSN entries so we will return all records
            // that match the provided value and store the results in selectedPatients.
            Patient selectedPatient = SelectPatientBySSN(ssn);

            // Check if any records were returned and display our query results.
            if (selectedPatient != null) {
                Console.WriteLine("Patient found with SSN = " + ssn);
                Console.WriteLine(selectedPatient.FirstName + " " + selectedPatient.LastName + "\tSSN: "
                    + selectedPatient.SSN + "\tBirthdate: " + selectedPatient.BirthDate);
            }
            else {
                Console.WriteLine("No patients found with SSN = " + ssn);
            }

            Console.WriteLine("Press Enter to exit...");
            Console.ReadLine();
        }

        private static ClientCredential _clientCredential;

        static void InitializeAzureKeyVaultProvider() {
            _clientCredential = new ClientCredential(applicationId, clientKey);

            SqlColumnEncryptionAzureKeyVaultProvider azureKeyVaultProvider =
              new SqlColumnEncryptionAzureKeyVaultProvider(GetToken);

            Dictionary<string, SqlColumnEncryptionKeyStoreProvider> providers =
              new Dictionary<string, SqlColumnEncryptionKeyStoreProvider>();

            providers.Add(SqlColumnEncryptionAzureKeyVaultProvider.ProviderName, azureKeyVaultProvider);
            SqlConnection.RegisterColumnEncryptionKeyStoreProviders(providers);
        }

        public async static Task<string> GetToken(string authority, string resource, string scope) {
            var authContext = new AuthenticationContext(authority);
            AuthenticationResult result = await authContext.AcquireTokenAsync(resource, _clientCredential);

            if (result == null)
                throw new InvalidOperationException("Failed to obtain the access token");
            return result.AccessToken;
        }

        static int InsertPatient(Patient newPatient) {
            int returnValue = 0;

            string sqlCmdText = @"INSERT INTO [dbo].[Patients] ([SSN], [FirstName], [LastName], [BirthDate])
     VALUES (@SSN, @FirstName, @LastName, @BirthDate);";

            SqlCommand sqlCmd = new SqlCommand(sqlCmdText);

            SqlParameter paramSSN = new SqlParameter(@"@SSN", newPatient.SSN);
            paramSSN.DbType = DbType.AnsiStringFixedLength;
            paramSSN.Direction = ParameterDirection.Input;
            paramSSN.Size = 11;

            SqlParameter paramFirstName = new SqlParameter(@"@FirstName", newPatient.FirstName);
            paramFirstName.DbType = DbType.String;
            paramFirstName.Direction = ParameterDirection.Input;

            SqlParameter paramLastName = new SqlParameter(@"@LastName", newPatient.LastName);
            paramLastName.DbType = DbType.String;
            paramLastName.Direction = ParameterDirection.Input;

            SqlParameter paramBirthDate = new SqlParameter(@"@BirthDate", newPatient.BirthDate);
            paramBirthDate.SqlDbType = SqlDbType.Date;
            paramBirthDate.Direction = ParameterDirection.Input;

            sqlCmd.Parameters.Add(paramSSN);
            sqlCmd.Parameters.Add(paramFirstName);
            sqlCmd.Parameters.Add(paramLastName);
            sqlCmd.Parameters.Add(paramBirthDate);

            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    sqlCmd.ExecuteNonQuery();
                }
                catch (Exception ex) {
                    returnValue = 1;
                    Console.WriteLine("The following error was encountered: ");
                    Console.WriteLine(ex.Message);
                    Console.WriteLine(Environment.NewLine + "Press Enter key to exit");
                    Console.ReadLine();
                    Environment.Exit(0);
                }
            }
            return returnValue;
        }


        static List<Patient> SelectAllPatients() {
            List<Patient> patients = new List<Patient>();

            SqlCommand sqlCmd = new SqlCommand(
              "SELECT [SSN], [FirstName], [LastName], [BirthDate] FROM [dbo].[Patients]",
                new SqlConnection(connectionString));

            using (sqlCmd.Connection = new SqlConnection(connectionString))

            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    SqlDataReader reader = sqlCmd.ExecuteReader();

                    if (reader.HasRows) {
                        while (reader.Read()) {
                            patients.Add(new Patient() {
                                SSN = reader[0].ToString(),
                                FirstName = reader[1].ToString(),
                                LastName = reader["LastName"].ToString(),
                                BirthDate = (DateTime)reader["BirthDate"]
                            });
                        }
                    }
                }
                catch (Exception ex) {
                    throw;
                }
            }

            return patients;
        }

        static Patient SelectPatientBySSN(string ssn) {
            Patient patient = new Patient();

            SqlCommand sqlCmd = new SqlCommand(
                "SELECT [SSN], [FirstName], [LastName], [BirthDate] FROM [dbo].[Patients] WHERE [SSN]=@SSN",
                new SqlConnection(connectionString));

            SqlParameter paramSSN = new SqlParameter(@"@SSN", ssn);
            paramSSN.DbType = DbType.AnsiStringFixedLength;
            paramSSN.Direction = ParameterDirection.Input;
            paramSSN.Size = 11;

            sqlCmd.Parameters.Add(paramSSN);

            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    SqlDataReader reader = sqlCmd.ExecuteReader();

                    if (reader.HasRows) {
                        while (reader.Read()) {
                            patient = new Patient() {
                                SSN = reader[0].ToString(),
                                FirstName = reader[1].ToString(),
                                LastName = reader["LastName"].ToString(),
                                BirthDate = (DateTime)reader["BirthDate"]
                            };
                        }
                    }
                    else {
                        patient = null;
                    }
                }
                catch (Exception ex) {
                    throw;
                }
            }
            return patient;
        }

        // This method simply deletes all records in the Patients table to reset our demo.
        static int ResetPatientsTable() {
            int returnValue = 0;

            SqlCommand sqlCmd = new SqlCommand("DELETE FROM Patients");
            using (sqlCmd.Connection = new SqlConnection(connectionString)) {
                try {
                    sqlCmd.Connection.Open();
                    sqlCmd.ExecuteNonQuery();

                }
                catch (Exception ex) {
                    returnValue = 1;
                }
            }
            return returnValue;
        }
    }

    class Patient {
        public string SSN { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public DateTime BirthDate { get; set; }
    }
}
```

## <a name="verify-that-the-data-is-encrypted"></a>Как проверить, что данные шифруются

Вы можете быстро проверить, действительно ли данные на сервере зашифрованы. Для этого можно запросить данные таблицы Patients с помощью SSMS (используя текущее подключение, где не задан параметр **Column Encryption Setting**).

Выполните следующий запрос к базе данных Clinic.

```sql
SELECT FirstName, LastName, SSN, BirthDate FROM Patients;
```

Вы увидите, что в зашифрованных столбцах не содержатся данные в виде открытого текста.

   ![Снимок экрана, на котором показано, что зашифрованные столбцы не содержат никаких данных в виде открытого текста.](./media/always-encrypted-azure-key-vault-configure/ssms-encrypted.png)

Чтобы получить доступ к данным в виде открытого текста, сначала убедитесь, что пользователь имеет необходимые разрешения в Azure Key Vault: *get*, *unwrapKey* и *verify*. Дополнительные сведения см. в статье [Создание и хранение главных ключей столбцов (постоянное шифрование)](/sql/relational-databases/security/encryption/create-and-store-column-master-keys-always-encrypted).

Затем во время подключения добавьте параметр *Column Encryption Setting=enabled*.

1. В **обозревателе объектов** SSMS щелкните сервер правой кнопкой мыши и выберите пункт **Отключить**.
2. Нажмите кнопку **подключить**  >  **ядро СУБД** , чтобы открыть окно **Подключение к серверу** , и выберите пункт **Параметры**.
3. Щелкните **Дополнительные параметры соединения** и введите **Column Encryption Setting=enabled**.

    ![Снимок экрана, на котором показана вкладка дополнительных параметров исправления.](./media/always-encrypted-azure-key-vault-configure/ssms-connection-parameter.png)

4. Выполните следующий запрос к базе данных Clinic.

   ```sql
   SELECT FirstName, LastName, SSN, BirthDate FROM Patients;
   ```

   Теперь в зашифрованных столбцах отображаются незашифрованные данные.
   
   ![Новое консольное приложение](./media/always-encrypted-azure-key-vault-configure/ssms-plaintext.png)

## <a name="next-steps"></a>Дальнейшие действия

После настройки базы данных для использования Always Encrypted может потребоваться выполнить следующие действия.

- [Сменить и очистить ключи](/sql/relational-databases/security/encryption/configure-always-encrypted-using-sql-server-management-studio).
- [Перенести данные, которые уже зашифрованы с использованием функции Always Encrypted.](/sql/relational-databases/security/encryption/migrate-sensitive-data-protected-by-always-encrypted)

## <a name="related-information"></a>Дополнительные сведения

- [Постоянное шифрование (разработка клиентских приложений)](/sql/relational-databases/security/encryption/always-encrypted-client-development)
- [Прозрачное шифрование данных](/sql/relational-databases/security/encryption/transparent-data-encryption)
- [Шифрование SQL Server](/sql/relational-databases/security/encryption/sql-server-encryption)
- [Мастер Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-wizard)
- [Блог о постоянном шифровании](/archive/blogs/sqlsecurity/always-encrypted-key-metadata)
