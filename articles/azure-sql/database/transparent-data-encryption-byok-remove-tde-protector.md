---
title: Удалить средство защиты TDE (PowerShell & Azure CLI)
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Узнайте, как реагировать на потенциально скомпрометированный предохранитель TDE для базы данных SQL Azure или Azure синапсе Analytics с помощью TDE с поддержкой собственного ключа (BYOK).
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 02/24/2020
ms.openlocfilehash: 31298be4d50c7f562e2e2b9adbf889b165b197e5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96461863"
---
# <a name="remove-a-transparent-data-encryption-tde-protector-using-powershell"></a>Удаление предохранителя TDE с помощью PowerShell
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


В этом разделе описывается, как реагировать на потенциально скомпрометированный TDE для защиты базы данных SQL Azure или Azure синапсе Analytics, использующей TDE с ключами, управляемыми клиентами, в службе поддержки Azure Key Vault-создание собственных ключей (BYOK). См. дополнительные сведения о [поддержке BYOK для TDE](transparent-data-encryption-byok-overview.md).

> [!CAUTION]
> Процедуры, описанные в этой статье, следует выполнять только в экстремальных случаях или в тестовых средах. Внимательно изучите шаги, так как удаление TDEных предохранителей из Azure Key Vault приведет к тому, что **база данных станет недоступна**.

Если во всех случаях возникает угроза компрометации ключа, например, что служба или пользователь имели несанкционированный доступ к ключу, то лучше удалить ключ.

Помните, что после удаления предохранителя TDE в Key Vault, в течение 10 минут все зашифрованные базы данных начинают блокировать все подключения с соответствующим сообщением об ошибке и меняют состояние на [недоступное](./transparent-data-encryption-byok-overview.md#inaccessible-tde-protector).

В этом пошаговом руководство рассматриваются два подхода в зависимости от желаемого результата после скомпрометированного реагирования на инцидент:

- Для внесения баз данных в базу данных SQL Azure или Azure синапсе Analytics в **недоступном** виде.
- Для внесения баз данных в базу данных SQL Azure или Azure Azure синапсе Analytics **недоступна**.

## <a name="prerequisites"></a>Предварительные условия

- Подписка Azure и права администратора для нее.
- Необходимо установить и запустить Azure PowerShell.
- В этом пошаговом руководство предполагается, что вы уже используете ключ из Azure Key Vault в качестве предохранителя TDE для базы данных SQL Azure или Azure синапсе. См. дополнительные сведения о [прозрачном шифровании данных с поддержкой создания собственных ключей](transparent-data-encryption-byok-overview.md).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

 Инструкции по установке модуля Az см. в статье об [установке Azure PowerShell](/powershell/azure/install-az-ps). Конкретные командлеты см. в разделе [AzureRM. SQL](/powershell/module/AzureRM.Sql/).

> [!IMPORTANT]
> Модуль PowerShell Azure Resource Manager (RM) по-прежнему поддерживается, но вся будущая разработка предназначена для модуля AZ. SQL. Исправления ошибок для модуля AzureRM будут продолжать выпускаться как минимум до декабря 2020 г.  Аргументы команд в модулях Az и AzureRm практически идентичны. Дополнительные сведения о совместимости см. в статье [Знакомство с новым модулем Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Сведения об установке см. [в разделе установка Azure CLI](/cli/azure/install-azure-cli).

* * *

## <a name="check-tde-protector-thumbprints"></a>Проверка отпечатков предохранителя TDE

В следующих шагах показано, как проверить, какие отпечатки предохранителя TDE все еще используются виртуальными файлами журнала (VLF) заданной базы данных.
Отпечаток текущего предохранителя TDE базы данных и идентификатор базы данных можно найти, выполнив команду:

```sql
SELECT [database_id],
       [encryption_state],
       [encryptor_type], /*asymmetric key means AKV, certificate means service-managed keys*/
       [encryptor_thumbprint]
 FROM [sys].[dm_database_encryption_keys]
```

Следующий запрос возвращает VLF и предохранители TDE, соответствующие используемым отпечаткам. Каждый отпечаток относится к разным ключам в Azure Key Vault (AKV):

```sql
SELECT * FROM sys.dm_db_log_info (database_id)
```

Кроме того, можно использовать PowerShell или Azure CLI:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Команда PowerShell **Get-AzureRmSqlServerKeyVaultKey**   предоставляет отпечаток предохранителя TDE, используемого в запросе, чтобы можно было видеть, какие ключи следует удерживать, а какие — удалять в AKV. Из Azure Key Vault можно безопасно удалить только те ключи, которые больше не используются базой данных.

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Команда PowerShell **AZ SQL Server Key Показать**   предоставляет отпечаток предохранителя TDE, используемого в запросе, чтобы можно было видеть, какие ключи следует учитывать, а какие — удалять в AKV. Из Azure Key Vault можно безопасно удалить только те ключи, которые больше не используются базой данных.

* * *

## <a name="keep-encrypted-resources-accessible"></a>Обеспечение доступности зашифрованных ресурсов

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Создайте [ключ в Key Vault](/powershell/module/az.keyvault/add-azkeyvaultkey). Убедитесь, что новый ключ успешно создан в другом хранилище. Нельзя использовать то же хранилище, где размещен скомпрометированный предохранитель TDE, так как управление доступом предоставляется на уровне хранилища.

2. Добавьте новый ключ на сервер с помощью командлетов [Add-азсклсерверкэйваулткэй](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) и [Set-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) и обновите его в качестве нового предохранителя TDE для сервера.

   ```powershell
   # add the key from Key Vault to the server  
   Add-AzSqlServerKeyVaultKey -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -KeyId <KeyVaultKeyId>

   # set the key as the TDE protector for all resources under the server
   Set-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> `
       -ServerName <LogicalServerName> -Type AzureKeyVault -KeyId <KeyVaultKeyId>
   ```

3. Убедитесь, что сервер и все реплики обновлены до нового предохранителя TDE с помощью командлета [Get-азсклсервертранспарентдатаенкриптионпротектор](/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) .

   > [!NOTE]
   > Распространение нового предохранителя TDE на все базы данных-источники и базы данных-получатели для обновляемого сервера может занять несколько минут.

   ```powershell
   Get-AzSqlServerTransparentDataEncryptionProtector -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

4. Создайте [резервную копию нового ключа](/powershell/module/az.keyvault/backup-azkeyvaultkey) в Key Vault.

   ```powershell
   # -OutputFile parameter is optional; if removed, a file name is automatically generated.
   Backup-AzKeyVaultKey -VaultName <KeyVaultName> -Name <KeyVaultKeyName> -OutputFile <DesiredBackupFilePath>
   ```

5. Удалите скомпрометированный ключ из Key Vault с помощью командлета [Remove-азкэйваулткэй](/powershell/module/az.keyvault/remove-azkeyvaultkey) .

   ```powershell
   Remove-AzKeyVaultKey -VaultName <KeyVaultName> -Name <KeyVaultKeyName>
   ```

6. Чтобы восстановить ключ для Key Vault в будущем с помощью командлета [RESTORE-азкэйваулткэй](/powershell/module/az.keyvault/restore-azkeyvaultkey) :

   ```powershell
   Restore-AzKeyVaultKey -VaultName <KeyVaultName> -InputFile <BackupFilePath>
   ```

# <a name="the-azure-cli"></a>[CLI Azure.](#tab/azure-cli)

Справочник по командам см. в [Azure CLI keyvault](/cli/azure/keyvault/key).

1. Создайте [ключ в Key Vault](/cli/azure/keyvault/key#az-keyvault-key-create). Убедитесь, что новый ключ успешно создан в другом хранилище. Нельзя использовать то же хранилище, где размещен скомпрометированный предохранитель TDE, так как управление доступом предоставляется на уровне хранилища.

2. Добавьте новый ключ на сервер и обновите его как новый компонент защиты TDE сервера.

   ```azurecli
   # add the key from Key Vault to the server  
   az sql server key create --kid <KeyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <LogicalServerName>

   # set the key as the TDE protector for all resources under the server
   az sql server tde-key set --server-key-type AzureKeyVault --kid <KeyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <LogicalServerName>
   ```

3. Убедитесь, что сервер и все реплики обновлены до нового предохранителя TDE.

   > [!NOTE]
   > Распространение нового предохранителя TDE на все базы данных-источники и базы данных-получатели для обновляемого сервера может занять несколько минут.

   ```azurecli
   az sql server tde-key show --resource-group <SQLDatabaseResourceGroupName> --server <LogicalServerName>
   ```

4. Создайте резервную копию нового ключа в Key Vault.

   ```azurecli
   # --file parameter is optional; if removed, a file name is automatically generated.
   az keyvault key backup --file <DesiredBackupFilePath> --name <KeyVaultKeyName> --vault-name <KeyVaultName>
   ```

5. Удалите скомпрометированный ключ из Key Vault.

   ```azurecli
   az keyvault key delete --name <KeyVaultKeyName> --vault-name <KeyVaultName>
   ```

6. Восстановление ключа для Key Vault в будущем.

   ```azurecli
   az keyvault key restore --file <BackupFilePath> --vault-name <KeyVaultName>
   ```

* * *

## <a name="make-encrypted-resources-inaccessible"></a>Сделать зашифрованные ресурсы недоступными

1. Удалите все базы данных, которые зашифрованы с помощью потенциально скомпрометированного ключа.

   Для файлов базы данных и журналов резервные копии создаются автоматически, поэтому вы всегда сможете восстановить базу данных на определенный момент времени, предоставив нужный ключ. Удаление баз данных перед удалением действующего предохранителя TDE нужно для того, чтобы предотвратить потерю данных из последних транзакций (до 10 минут).

2. Создайте в Key Vault резервную копию материала ключа для предохранителя TDE.
3. Удалите потенциально скомпрометированный ключ из Key Vault.

[!INCLUDE [sql-database-akv-permission-delay](../includes/sql-database-akv-permission-delay.md)]

## <a name="next-steps"></a>Дальнейшие действия

- См. дополнительные сведения о том, как [заменить предохранитель TDE для сервера в соответствии с требованиями к безопасности с помощью PowerShell](transparent-data-encryption-byok-key-rotation.md).
- См. дополнительные сведения о [включении прозрачного шифрования данных с помощью собственного ключа из Key Vault с помощью PowerShell](transparent-data-encryption-byok-configure.md).