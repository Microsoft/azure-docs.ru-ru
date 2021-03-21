---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: 6195b949cc71043dfa7a12bdece7a311dbde5e21
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95560750"
---
## <a name="next-steps"></a>Дальнейшие действия

После включения интеграции хранилища ключей Azure вы сможете включить шифрование SQL Server на своей виртуальной машине с SQL. Во-первых, необходимо создать асимметричный ключ в вашем хранилище ключей и симметричный ключ в SQL Server на виртуальной машине. После этого вы сможете выполнять инструкции T-SQL для включения шифрования базы данных и резервных копий.

Существует несколько способов шифрования, преимуществами которых вы можете воспользоваться.

* [Прозрачное шифрование данных (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption)
* [Зашифрованные резервные копии](/sql/relational-databases/backup-restore/backup-encryption)
* [Шифрование на уровне столбцов (CLE)](/sql/t-sql/functions/cryptographic-functions-transact-sql)

Следующие сценарии Transact-SQL содержат примеры для каждого из этих вариантов.

### <a name="prerequisites-for-examples"></a>Предварительные требования для примеров

Каждый пример основан на двух предварительных требованиях: асимметричном ключе из хранилища ключей, именуемом **CONTOSO_KEY** , и учетные данные, созданные функцией интеграции AKV, именуемой **Azure_EKM_cred**. Следующие команды Transact-SQL настраивают эти предварительные требования для запуска примеров.

``` sql
USE master;
GO

--create credential
--The <<SECRET>> here requires the <Application ID> (without hyphens) and <Secret> to be passed together without a space between them.
CREATE CREDENTIAL Azure_EKM_cred
    WITH IDENTITY = 'keytestvault', --keyvault
    SECRET = '<<SECRET>>'
FOR CRYPTOGRAPHIC PROVIDER AzureKeyVault_EKM_Prov;


--Map the credential to a SQL login that has sysadmin permissions. This allows the SQL login to access the key vault when creating the asymmetric key in the next step.
ALTER LOGIN [SQL_Login]
ADD CREDENTIAL Azure_EKM_cred;


CREATE ASYMMETRIC KEY CONTOSO_KEY
FROM PROVIDER [AzureKeyVault_EKM_Prov]
WITH PROVIDER_KEY_NAME = 'KeyName_in_KeyVault',  --The key name here requires the key we created in the key vault
CREATION_DISPOSITION = OPEN_EXISTING;
```

### <a name="transparent-data-encryption-tde"></a>Прозрачное шифрование данных (TDE)

1. Создайте имя входа SQL Server для использования компонентом Database Engine для прозрачного шифрования данных, а затем добавьте учетные данные.

   ``` sql
   USE master;
   -- Create a SQL Server login associated with the asymmetric key
   -- for the Database engine to use when it loads a database
   -- encrypted by TDE.
   CREATE LOGIN EKM_Login
   FROM ASYMMETRIC KEY CONTOSO_KEY;
   GO

   -- Alter the TDE Login to add the credential for use by the
   -- Database Engine to access the key vault
   ALTER LOGIN EKM_Login
   ADD CREDENTIAL Azure_EKM_cred;
   GO
   ```

1. Создайте ключ шифрования базы данных, который будет использоваться для прозрачного шифрования данных.

   ``` sql
   USE ContosoDatabase;
   GO

   CREATE DATABASE ENCRYPTION KEY 
   WITH ALGORITHM = AES_128 
   ENCRYPTION BY SERVER ASYMMETRIC KEY CONTOSO_KEY;
   GO

   -- Alter the database to enable transparent data encryption.
   ALTER DATABASE ContosoDatabase
   SET ENCRYPTION ON;
   GO
   ```

### <a name="encrypted-backups"></a>Зашифрованные резервные копии

1. Создайте имя входа SQL Server для использования компонентом Database Engine для шифрования резервных копий, а затем добавьте учетные данные.

   ``` sql
   USE master;
   -- Create a SQL Server login associated with the asymmetric key
   -- for the Database engine to use when it is encrypting the backup.
   CREATE LOGIN EKM_Login
   FROM ASYMMETRIC KEY CONTOSO_KEY;
   GO

   -- Alter the Encrypted Backup Login to add the credential for use by
   -- the Database Engine to access the key vault
   ALTER LOGIN EKM_Login
   ADD CREDENTIAL Azure_EKM_cred ;
   GO
   ```

1. Зашифруйте архивированную базу данных асимметричным ключом, сохраненным в хранилище ключей.

   ``` sql
   USE master;
   BACKUP DATABASE [DATABASE_TO_BACKUP]
   TO DISK = N'[PATH TO BACKUP FILE]'
   WITH FORMAT, INIT, SKIP, NOREWIND, NOUNLOAD,
   ENCRYPTION(ALGORITHM = AES_256, SERVER ASYMMETRIC KEY = [CONTOSO_KEY]);
   GO
   ```

### <a name="column-level-encryption-cle"></a>Шифрование на уровне столбцов (CLE)

Этот скрипт создает симметричный ключ, защищенный асимметричным ключом в хранилище ключей, и затем использует симметричный ключ для шифрования данных в базе данных.

``` sql
CREATE SYMMETRIC KEY DATA_ENCRYPTION_KEY
WITH ALGORITHM=AES_256
ENCRYPTION BY ASYMMETRIC KEY CONTOSO_KEY;

DECLARE @DATA VARBINARY(MAX);

--Open the symmetric key for use in this session
OPEN SYMMETRIC KEY DATA_ENCRYPTION_KEY
DECRYPTION BY ASYMMETRIC KEY CONTOSO_KEY;

--Encrypt syntax
SELECT @DATA = ENCRYPTBYKEY(KEY_GUID('DATA_ENCRYPTION_KEY'), CONVERT(VARBINARY,'Plain text data to encrypt'));

-- Decrypt syntax
SELECT CONVERT(VARCHAR, DECRYPTBYKEY(@DATA));

--Close the symmetric key
CLOSE SYMMETRIC KEY DATA_ENCRYPTION_KEY;
```

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения об использовании этих возможностей шифрования см. в статье [Использование расширенного управления ключами с функциями шифрования SQL Server](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server#UsesOfEKM).

Обратите внимание, что действия, описанные в этой статье, предполагают, что у вас уже есть SQL Server на виртуальной машине Azure. Если нет, см. статью [Подготовка виртуальной машины с SQL Server в Azure](../articles/azure-sql/virtual-machines/windows/create-sql-vm-portal.md). Другие темы, связанные с запуском SQL Server на виртуальных машинах Azure, см. в статье [Общие сведения об SQL Server на виртуальных машинах Azure](../articles/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md).