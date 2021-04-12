---
title: Руководство по использованию управляемого удостоверения для доступа к Azure Cosmos DB для Windows в Azure AD
description: В этом руководстве объясняется, как получить доступ к Azure Cosmos DB с помощью назначаемого системой управляемого удостоверения на виртуальной машине Windows.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/10/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 04508f1aa8ee9d6b4f730f57c60d959fab209122
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101093803"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>Руководство по Использование назначаемого системой управляемого удостоверения виртуальной машины Windows для доступа к Azure Cosmos DB

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве описано, как получить доступ к Cosmos DB с помощью назначаемого системой управляемого удостоверения для виртуальной машины Windows. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание учетной записи Cosmos DB
> * предоставление доступа назначаемому системой управляемому удостоверению виртуальной машины Windows к ключам доступа для учетной записи Cosmos DB;
> * получение маркера доступа с помощью назначаемого системой управляемого удостоверения виртуальной машины Windows и вызов Azure Resource Manager;
> * Получение ключей доступа из Azure Resource Manager для создания вызовов Cosmos DB

## <a name="prerequisites"></a>Предварительные требования

- См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](overview.md). 
- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжить.
- Для выполнения требуемых операций создания ресурсов и управления ролями учетной записи нужно предоставить разрешения роли "Владелец" в соответствующей области (подписка или группа ресурсов). Если нуждаетесь в помощи с назначением ролей, прочитайте статью [Назначение ролей Azure с помощью портала Azure](../../role-based-access-control/role-assignments-portal.md).
- Установка последней версии [Azure PowerShell](/powershell/azure/install-az-ps)
- Кроме того, вам потребуется виртуальная машина Windows с включенными управляемыми удостоверениями, назначенными системой.
  - Если вам нужно создать виртуальную машину для работы с этим руководством, см. раздел [Управляемое удостоверение, назначаемое системой](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity).

## <a name="create-a-cosmos-db-account"></a>Создание учетной записи Cosmos DB 

Если у вас еще нет учетной записи Cosmos DB, создайте ее. Этот шаг можно пропустить и использовать имеющуюся учетную запись Cosmos DB. 

1. Нажмите кнопку **Создать ресурс** в верхнем левом углу окна портала Azure.
2. Выберите **Databases** (Базы данных),а затем — **Azure Cosmos DB**, после чего отобразится панель New account (Новая учетная запись).
3. Введите **идентификатор** учетной записи Cosmos DB, которая будет использоваться далее.  
4. **API** должно иметь значение SQL. Подход, описанный в этом руководстве, можно использовать с другими доступными типами API, но указанные в этом руководстве действия подходят только для API SQL.
5. Убедитесь, что значения **подписки** и **группы ресурсов** соответствуют указанным при создании виртуальной машины на предыдущем шаге.  Выберите **расположение**, в котором доступен Cosmos DB.
6. Нажмите кнопку **Создать**.

### <a name="create-a-collection"></a>Создание коллекции 

Затем добавьте коллекцию данных в учетную запись Cosmos DB, с помощью которой можно создавать запросы на последующих шагах.

1. Перейдите к созданной учетной записи Cosmos DB.
2. На вкладке **Обзор** нажмите кнопку **Добавить коллекцию**. Откроется панель "Добавить коллекцию".
3. Присвойте коллекции идентификатор базы данных, идентификатор коллекции, выберите емкость хранилища, введите ключ секции и значение пропускной способности, а затем нажмите кнопку **ОК**.  Для этого руководства в качестве идентификатора базы данных и коллекции достаточно будет использовать значение Test. Выберите фиксированное значение емкости хранилища и самую низкую пропускную способность (400 ЕЗ/с).  


## <a name="grant-access"></a>Предоставление доступа

В этом разделе описано, как предоставлять назначаемому системой управляемому удостоверению виртуальной машины Windows доступ к ключам доступа для учетной записи Cosmos DB. В Cosmos DB не встроена поддержка аутентификации Azure AD. Но можно использовать назначаемое системой управляемое удостоверение, чтобы извлечь ключ доступа к Cosmos DB из Resource Manager и с помощью этого ключа получить доступ к Cosmos DB. На этом шаге назначаемому системой управляемому удостоверению предоставляется доступ к ключам учетной записи Cosmos DB.

Чтобы с помощью PowerShell предоставить виртуальной машине Windows доступ на основе управляемого удостоверения, назначаемого системой, к учетной записи Cosmos DB в Azure Resource Manager, укажите следующие значения:

- `<SUBSCRIPTION ID>`
- `<RESOURCE GROUP>`
- `<COSMOS DB ACCOUNT NAME>`

Cosmos DB поддерживает два уровня детализации при использовании ключей доступа: доступ на чтение и запись для учетной записи и доступ только для чтения для учетной записи.  Назначьте роль `DocumentDB Account Contributor`, если вы хотите получить для учетной записи ключи для записи и чтения, или роль `Cosmos DB Account Reader Role`, чтобы получить ключи только для чтения.  Для этого руководства назначьте `Cosmos DB Account Reader Role`:

```azurepowershell
$spID = (Get-AzVM -ResourceGroupName myRG -Name myVM).identity.principalid
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Cosmos DB Account Reader Role" -Scope "/subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>"
```

>[!NOTE]
> Помните, что, если вы не можете выполнить операцию, возможно, у вас нет нужных разрешений. Если требуется доступ на запись к ключам, необходимо использовать роль Azure (например, участника учетной записи DocumentDB) или создать настраиваемую роль. Дополнительные сведения см. в статье [Управление доступом на основе ролей Azure в Azure Cosmos DB](../../cosmos-db/role-based-access-control.md).

## <a name="access-data"></a>Доступ к данным

В этом разделе показано, как вызывать Azure Resource Manager с помощью маркера доступа для назначенного системой управляемого удостоверения ВМ Windows. Далее в этом руководстве мы будем работать с виртуальной машиной, которую только что создали. 

Вам нужно установить последнюю версию [Azure CLI](/cli/azure/install-azure-cli) на виртуальной машине Windows.

### <a name="get-an-access-token"></a>Получение маркера доступа

1. На портале Azure перейдите к разделу **Виртуальные машины**, выберите свою виртуальную машину Windows и вверху в разделе **Обзор** щелкните **Подключить**. 
2. Введите **имя пользователя** и **пароль**, добавленные при создании виртуальной машины Windows. 
3. Теперь, когда создано **подключение к удаленному рабочему столу** с виртуальной машиной, откройте PowerShell в удаленном сеансе.
4. С помощью командлета Powershell Invoke-WebRequest выполните запрос к локальным управляемым удостоверениям для конечной точки ресурсов Azure, чтобы получить маркер доступа для Azure Resource Manager.

   ```powershell
   $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -Method GET -Headers @{Metadata="true"}
   ```

   > [!NOTE]
   > Значение параметра resource должно точно совпадать со значением, которое будет использоваться в Azure AD. Если используется идентификатор ресурса Resource Manager, добавьте косую черту после универсального кода ресурса (URI).
    
   Затем извлеките элемент content, который хранится в виде форматированной строки JSON в объекте $response. 
    
   ```powershell
   $content = $response.Content | ConvertFrom-Json
   ```
   Затем извлеките маркер доступа из ответа.
    
   ```powershell
   $ArmToken = $content.access_token
   ```

### <a name="get-access-keys"></a>Получение ключей доступа 

В этом разделе описано, как получить ключи доступа из Azure Resource Manager для создания вызовов Cosmos DB С помощью PowerShell мы обратимся к Resource Manager, используя полученный ранее маркер доступа, и получим ключ доступа к учетной записи Cosmos DB. Получив ключ доступа, можно создать запрос к Cosmos DB. Используйте собственные значения, чтобы заменить приведенные ниже:

- `<SUBSCRIPTION ID>`
- `<RESOURCE GROUP>`
- `<COSMOS DB ACCOUNT NAME>` 
- Замените значение `<ACCESS TOKEN>` маркером доступа, который вы получили ранее. 

>[!NOTE]
>Если вы хотите получить ключи для чтения и записи, используйте тип операции ключа `listKeys`.  Если вы хотите получить ключи только для чтения, используйте тип операции ключа `readonlykeys`. Если вы не можете использовать listkeys, убедитесь, что управляемому удостоверению назначена [соответствующая роль](../../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role).

```powershell
Invoke-WebRequest -Uri 'https://management.azure.com/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RESOURCE-GROUP>/providers/Microsoft.DocumentDb/databaseAccounts/<COSMOS DB ACCOUNT NAME>/readonlykeys/?api-version=2016-03-31' -Method POST -Headers @{Authorization="Bearer $ARMToken"}
```
В ответе будет содержаться список ключей.  Например, при получении ключей только для чтения вы увидите следующее:

```powershell
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```
Теперь, когда у вас есть ключ доступа для учетной записи Cosmos DB, вы можете передать его в пакет SDK Cosmos DB и выполнять вызовы для получения доступа к учетной записи.  Чтобы получить краткий пример, вы можете передать ключ доступа в Azure CLI.  Вы можете получить `<COSMOS DB CONNECTION URL>` на вкладке **Обзор** в колонке учетной записи Cosmos DB на портале Azure.  Замените `<ACCESS KEY>` полученным выше значением.

```azurecli
az cosmosdb collection show -c <COLLECTION ID> -d <DATABASE ID> --url-connection "<COSMOS DB CONNECTION URL>" --key <ACCESS KEY>
```

Эта команда CLI возвращает сведения о коллекции.

```output
{
  "collection": {
    "_conflicts": "conflicts/",
    "_docs": "docs/",
    "_etag": "\"00006700-0000-0000-0000-5a8271e90000\"",
    "_rid": "Es5SAM2FDwA=",
    "_self": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/",
    "_sprocs": "sprocs/",
    "_triggers": "triggers/",
    "_ts": 1518498281,
    "_udfs": "udfs/",
    "id": "Test",
    "indexingPolicy": {
      "automatic": true,
      "excludedPaths": [],
      "includedPaths": [
        {
          "indexes": [
            {
              "dataType": "Number",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "String",
              "kind": "Range",
              "precision": -1
            },
            {
              "dataType": "Point",
              "kind": "Spatial"
            }
          ],
          "path": "/*"
        }
      ],
      "indexingMode": "consistent"
    }
  },
  "offer": {
    "_etag": "\"00006800-0000-0000-0000-5a8271ea0000\"",
    "_rid": "f4V+",
    "_self": "offers/f4V+/",
    "_ts": 1518498282,
    "content": {
      "offerIsRUPerMinuteThroughputEnabled": false,
      "offerThroughput": 400
    },
    "id": "f4V+",
    "offerResourceId": "Es5SAM2FDwA=",
    "offerType": "Invalid",
    "offerVersion": "V2",
    "resource": "dbs/Es5SAA==/colls/Es5SAM2FDwA=/"
  }
}
```


## <a name="disable"></a>Отключить

[!INCLUDE [msi-tut-disable](../../../includes/active-directory-msi-tut-disable.md)]



## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как получить доступ к Cosmos DB с помощью назначаемого системой управляемого удостоверения на виртуальной машине Windows.  Дополнительные сведения о Cosmos DB см. здесь:

> [!div class="nextstepaction"]
>[Добро пожаловать в базу данных Azure Cosmos DB](../../cosmos-db/introduction.md)