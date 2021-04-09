---
title: Классификация данных Azure SQL с помощью меток Azure Purview
description: Импорт настроек классификации из Azure Purview в Базу данных SQL Azure и Azure Synpase Analytics
services: sql-database
ms.service: sql-database
ms.custom: ''
ms.devlang: azurepowershell
ms.topic: sample
author: davidtrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 02/17/2021
ms.openlocfilehash: 2eab7c535ff0c68da772e8a45ead12420734279c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101714907"
---
# <a name="classify-your-azure-sql-data-using-azure-purview-labels"></a>Классификация данных Azure SQL с помощью меток Azure Purview
[!INCLUDE[appliesto-sqldb-asa](../../azure-sql/includes/appliesto-sqldb-asa.md)]

В этом документе объясняется, как добавить метки Azure Purview в Базу данных SQL Azure и Azure Synapse Analytics (ранее — Хранилище данных SQL).

## <a name="create-an-application"></a>Создание приложения

1. На портале Azure откройте **Azure Active Directory**.
2. В разделе **Управление** выберите **Регистрация приложения**.
3. Создайте приложение Azure Active Directory, выбрав **Новое приложение**.
4. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
5. Создав приложение, перейдите на вкладку **Сертификаты и секреты** в разделе **Управление**.
6. Создайте секрет клиента, выбрав элемент **Новый секрет клиента** в разделе **Секреты клиента**.
7. Добавьте описание для секрета клиента, выберите срок действия и щелкните **Добавить**.
8. Сохраните значение секрета из столбца **Значение** для дальнейшего использования.

   > [!NOTE]
   > После закрытия страницы значение будет скрыто. Если вы вернетесь на страницу, вы не сможете получить это значение секрета клиента. Вам придется создать новый секрет клиента.

9. Вернитесь на страницу обзора созданного приложения и скопируйте следующие значения для дальнейшего использования:
    1. Идентификатор приложения (клиента)
    1. Идентификатор каталога (клиента)

## <a name="provide-permissions-to-the-application"></a>Предоставление разрешений для приложения

1. На портале Azure выполните поиск по запросу **учетные записи Purview**.
2. Выберите учетную запись Purview, в которой классифицированы базы данных SQL и Synapse.
3. Откройте вкладку **Управление доступом (IAM)** и выберите **Добавить**.

4. Выберите **Добавить назначение ролей**.
5. В разделе **Роль** найдите роль **Purview Data Reader** (Читатель данных Purview) и выберите ее.
6. В разделе **Выбор** найдите приложение, которое вы создали ранее, выберите его и щелкните **Сохранить**.

## <a name="extract-the-classification-from-azure-purview"></a>Извлечение классификации из Azure Purview

1. Откройте учетную запись Purview и на домашней странице найдите Базу данных SQL Azure или Azure Synapse Analytics, куда нужно скопировать метки.
2. В разделе **Свойства** скопируйте значение qualifiedName и сохраните его для дальнейшего использования.
3. Откройте оболочку PowerShell.

4. Скопируйте один из приведенных ниже скриптов в соответствии с типом ресурса SQL (База данных SQL Azure или Azure Synapse).
5. Укажите для параметров значения, скопированные выше:

   а. $TenantID: раздел 1, шаг 9.
   
   b. $ClientID: раздел 1, шаг 9.
   
   c. $SecretID: раздел 1, шаг 8.
   
   d. $purviewAccountName: раздел 2, шаг 2.
   
   д) $sqlDatabaseName: раздел 3, шаг 2.

6. Скопируйте выходные данные скрипта для дальнейшего использования.

### <a name="for-azure-sql-database"></a>Для Базы данных SQL Azure

```powershell
# Replace the values below with the relevant values for your environment
$TenantID = 'your_tenant_id'
$ClientID = 'your_client_id'
$SecretID = 'your_secret_id'
$purviewAccountName='purview_account_name'
$sqlDatabaseName="mssql://sql_server_name.database.windows.net/db_name"

###############################################################################
# Get an access accessToken, and build REST Request Header.
###############################################################################
$cmdletParameters = @{
  Method  = "POST"
  URI     = "https://login.microsoftonline.com/$TenantID/oauth2/token"
  Headers = @{
    "Content-Type" = 'application/x-www-form-urlencoded'
  }
  Body    = "client_id=$ClientID&client_secret=$SecretID&resource=https://purview.azure.net&grant_type=client_credentials"
}

$invokeResult = Invoke-RestMethod @cmdletParameters;
$accessToken = $invokeResult.access_token; 

$restRequestHeader = @{
  "authorization" = "Bearer $accessToken"
};

###############################################################################
# Get database entity.
###############################################################################
$getDatabaseEntityEntryEndpoint = "https://" + $purviewAccountName + ".catalog.purview.azure.com/api/atlas/v2/entity/uniqueAttribute/type/azure_sql_db?attr:qualifiedName=" + $sqlDatabaseName;
$cmdletParameters = @{
  Method      = "Get"
  Uri         = $getDatabaseEntityEntryEndpoint
  Headers     = $restRequestHeader
  ContentType = "application/json"        
};

$invokeResult =  Invoke-RestMethod @cmdletParameters;
$referredEntities = $invokeResult.referredEntities;
if ($null -eq $referredEntities) {
  Write-Output "No referred entities found under database entity!";
  exit;
}

###############################################################################
# Iterate database referred entities, find classified columns, and build T-SQL.
###############################################################################
foreach ($referredEntity in $referredEntities.psobject.Properties.GetEnumerator()) {
  $typeName = $referredEntity.Value.typeName;
  if ($null -eq $typeName -or $typeName -ne 'azure_sql_column') {
    continue;
  }
  
  $classifications = $referredEntity.Value.classifications;
  if ($null -eq $classifications) {
    continue;
  }
  
  foreach ($classification in $classifications.GetEnumerator())
  {
    if ($classification.typeName -notmatch 'Microsoft\.Label\.(?<labelId>.+)') { 
        continue;
    }

      $labelId = $Matches.labelId -replace "_", "-";

      $attributes = $referredEntity.Value.attributes;
      if ($null -eq $attributes) {
        continue;
      }
    
      $qualifiedName = $attributes.qualifiedName;
      if ($qualifiedName -notmatch '.+\.database\.windows\.net\/.+\/(?<schemaName>.+)\/(?<tableName>.+)#(?<columnName>.+)') {
        continue;
      }

      $schemaName = $Matches.schemaName;
      $tableName = $Matches.tableName;
      $columnName = $Matches.columnName;
      
      Write-Output "ADD SENSITIVITY CLASSIFICATION TO ${schemaName}.${tableName}.${columnName} WITH (LABEL='Azure Purview Label', LABEL_ID='${labelId}');";
  }
}
```

### <a name="for-azure-synapse-analytics"></a>Для Azure Synapse Analytics

```powershell
# Replace the values below with the relevant values for your environment
$TenantID = 'your_tenant_id'
$ClientID = 'your_client_id'
$SecretID = 'your_secret_id'
$purviewAccountName='purview_account_name'
$dwDatabaseName="mssql://dw_server_name.database.windows.net/dw_name"

###############################################################################
# Get an access accessToken, and build REST Request Header.
###############################################################################
$cmdletParameters = @{
  Method  = "POST"
  URI     = "https://login.microsoftonline.com/$TenantID/oauth2/token"
  Headers = @{
    "Content-Type" = 'application/x-www-form-urlencoded'
  }
  Body    = "client_id=$ClientID&client_secret=$SecretID&resource=https://purview.azure.net&grant_type=client_credentials"
}

$invokeResult = Invoke-RestMethod @cmdletParameters;
$accessToken = $invokeResult.access_token; 

$restRequestHeader = @{
  "authorization" = "Bearer $accessToken"
};

###############################################################################
# Get database entity.
###############################################################################
$getDatabaseEntityEntryEndpoint = "https://" + $purviewAccountName + ".catalog.purview.azure.com/api/atlas/v2/entity/uniqueAttribute/type/azure_sql_dw?attr:qualifiedName=" + $dwDatabaseName;
$cmdletParameters = @{
  Method      = "Get"
  Uri         = $getDatabaseEntityEntryEndpoint
  Headers     = $restRequestHeader
  ContentType = "application/json"        
};

$invokeResult =  Invoke-RestMethod @cmdletParameters;
$referredEntities = $invokeResult.referredEntities;
if ($null -eq $referredEntities) {
  Write-Output "No referred entities found under database entity!";
  exit;
}

###############################################################################
# Iterate database referred entities, find classified columns, and build T-SQL.
###############################################################################
foreach ($referredEntity in $referredEntities.psobject.Properties.GetEnumerator()) {
  $typeName = $referredEntity.Value.typeName;
  if ($null -eq $typeName -or $typeName -ne 'azure_sql_dw_column') {
    continue;
  }
  
  $classifications = $referredEntity.Value.classifications;
  if ($null -eq $classifications) {
    continue;
  }
  
  foreach ($classification in $classifications.GetEnumerator())
  {
    if ($classification.typeName -notmatch 'Microsoft\.Label\.(?<labelId>.+)') { 
        continue;
    }

      $labelId = $Matches.labelId -replace "_", "-";

      $attributes = $referredEntity.Value.attributes;
      if ($null -eq $attributes) {
        continue;
      }
    
      $qualifiedName = $attributes.qualifiedName;
      if ($qualifiedName -notmatch '.+\.database\.windows\.net\/.+\/(?<schemaName>.+)\/(?<tableName>.+)#(?<columnName>.+)') {
        continue;
      }

      $schemaName = $Matches.schemaName;
      $tableName = $Matches.tableName;
      $columnName = $Matches.columnName;
      
      Write-Output "ADD SENSITIVITY CLASSIFICATION TO ${schemaName}.${tableName}.${columnName} WITH (LABEL='Azure Purview Label', LABEL_ID='${labelId}');";
  }
}
```

## <a name="run-the-t-sql-command-on-your-sql-asset"></a>Выполнение команды T-SQL для ресурса SQL

1. Подключитесь к Базе данных SQL Azure или Azure Synapse с помощью используемого вами средства.
2. Выполните команду T-SQL, скопированную в предыдущем разделе.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure PowerShell см. в [документации по Azure PowerShell](/powershell/).

Дополнительные сведения об Azure Purview см. в [документации по Azure Purview](../../purview/index.yml).