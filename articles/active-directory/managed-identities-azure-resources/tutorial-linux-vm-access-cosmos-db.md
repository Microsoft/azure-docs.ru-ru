---
title: Учебник. Получение доступа к Azure Cosmos DB для Linux в Azure AD с помощью управляемого удостоверения
description: Из этого руководства вы узнаете, как получить доступ к Azure Cosmos DB с помощью назначаемого системой управляемого удостоверения на виртуальной машине Linux.
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
ms.openlocfilehash: b81f80af69b47152f7111066070e173bb1ede5f4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101093951"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-cosmos-db"></a>Руководство по Использование назначаемого системой управляемого удостоверения на виртуальной машине Linux для доступа к Azure Cosmos DB 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

В этом руководстве описывается, как получить доступ к Azure Cosmos DB с помощью управляемого удостоверения для виртуальной машины Linux, назначаемого системой. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание учетной записи Cosmos DB
> * Создание коллекции в учетной записи Cosmos DB
> * предоставление доступа к экземпляру Azure Cosmos DB назначаемому системой управляемому удостоверению;
> * получение `principalID` назначаемого системой управляемого удостоверения виртуальной машины Linux;
> * Получение маркера доступа и вызов Azure Resource Manager с его помощью.
> * Получение ключей доступа из Azure Resource Manager для создания вызовов Cosmos DB

## <a name="prerequisites"></a>Предварительные требования

- См. дополнительные сведения об [управляемых удостоверениях для ресурсов Azure](overview.md). 
- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжить.
- Для выполнения требуемых операций создания ресурсов и управления ролями учетной записи нужно предоставить разрешения роли "Владелец" в соответствующей области (подписка или группа ресурсов). Если нуждаетесь в помощи с назначением ролей, прочитайте статью [Назначение ролей Azure с помощью портала Azure](../../role-based-access-control/role-assignments-portal.md).
- Выполнить примеры скриптов можно двумя способами:
    - используйте службу [Azure Cloud Shell](../../cloud-shell/overview.md), которую можно открыть с помощью кнопки **Попробовать** в правом верхнем углу блоков кода.
    - Выполните скрипты локально, установив последнюю версию [Azure CLI](/cli/azure/install-azure-cli), а затем войдите в Azure с помощью команды [az login](/cli/azure/reference-index#az-login). Используйте учетную запись, связанную с подпиской Azure, в рамках которой нужно создать ресурсы.

## <a name="create-a-cosmos-db-account"></a>Создание учетной записи Cosmos DB 

Если у вас еще нет учетной записи Cosmos DB, создайте ее. Этот шаг можно пропустить и использовать имеющуюся учетную запись Cosmos DB. 

1. Нажмите кнопку **Создать ресурс** в верхнем левом углу окна портала Azure.
2. Выберите **Databases** (Базы данных),а затем — **Azure Cosmos DB**, после чего отобразится панель New account (Новая учетная запись).
3. Введите **идентификатор** учетной записи Cosmos DB, которая будет использоваться далее.  
4. **API** должно иметь значение SQL. Подход, описанный в этом руководстве, можно использовать с другими доступными типами API, но указанные в этом руководстве действия подходят только для API SQL.
5. Убедитесь, что значения **подписки** и **группы ресурсов** соответствуют указанным при создании виртуальной машины на предыдущем шаге.  Выберите **расположение**, в котором доступен Cosmos DB.
6. Щелкните **Создать**.

### <a name="create-a-collection-in-the-cosmos-db-account"></a>Создание коллекции в учетной записи Cosmos DB

Затем добавьте коллекцию данных в учетную запись Cosmos DB, с помощью которой можно создавать запросы на последующих шагах.

1. Перейдите к созданной учетной записи Cosmos DB.
2. На вкладке **Обзор** нажмите кнопку **Добавить коллекцию**. Откроется панель "Добавить коллекцию".
3. Присвойте коллекции идентификатор базы данных, идентификатор коллекции, выберите емкость хранилища, введите ключ секции и значение пропускной способности, а затем нажмите кнопку **ОК**.  Для этого руководства в качестве идентификатора базы данных и коллекции достаточно будет использовать значение Test. Выберите фиксированное значение емкости хранилища и самую низкую пропускную способность (400 ЕЗ/с).  

## <a name="grant-access"></a>Предоставление доступа

Чтобы получить доступ к ключам доступа учетной записи Cosmos DB из Resource Manager в следующем разделе, необходимо получить `principalID` назначаемого системой управляемого удостоверения виртуальной машины Linux.  Обязательно замените значения параметров `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` (группа ресурсов виртуальной машины) и `<VM NAME>` собственными значениями.

```azurecli-interactive
az resource show --id /subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAMe> --api-version 2017-12-01
```
Ответ будет содержать сведения о назначаемом системой управляемом удостоверении (обратите внимание на principalID, так как это значение используется в следующем разделе):

```output  
{
    "id": "/subscriptions/<SUBSCRIPTION ID>/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAMe>",
  "identity": {
    "principalId": "6891c322-314a-4e85-b129-52cf2daf47bd",
    "tenantId": "733a8f0e-ec41-4e69-8ad8-971fc4b533f8",
    "type": "SystemAssigned"
 }
```

### <a name="grant-your-linux-vms-system-assigned-identity-access-to-the-cosmos-db-account-access-keys"></a>Предоставление доступа к ключам доступа учетной записи Cosmos DB назначаемому системой управляемому удостоверению виртуальной машины Linux

В Cosmos DB не встроена поддержка аутентификации Azure AD. Но можно использовать управляемое удостоверение для извлечения ключей доступа к Cosmos DB из Resource Manager, а затем применить эти ключи для получения доступа к Cosmos DB. На этом шаге управляемому удостоверению предоставляется доступ к ключам учетной записи Cosmos DB.

Чтобы предоставить назначаемому системой управляемому удостоверению доступ к учетной записи Cosmos DB в Azure Resource Manager с помощью Azure CLI, обновите значения `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` и `<COSMOS DB ACCOUNT NAME>` для своей среды. Замените `<MI PRINCIPALID>` свойством `principalId`, возвращенным командой `az resource show` при получении principalID управляемого удостоверения виртуальной машины Linux.  Cosmos DB поддерживает два уровня детализации при использовании ключей доступа: доступ на чтение и запись для учетной записи и доступ только для чтения для учетной записи.  Назначьте роль `DocumentDB Account Contributor`, если вы хотите получить ключи для записи и чтения для учетной записи, или назначьте роль `Cosmos DB Account Reader Role`, чтобы получить ключи только для чтения для учетной записи.

```azurecli-interactive
az role assignment create --assignee <MI PRINCIPALID> --role '<ROLE NAME>' --scope "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMODS DB ACCOUNT NAME>"
```

Ответ включает сведения о созданных назначениях ролей:

```output
{
  "id": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>/providers/Microsoft.Authorization/roleAssignments/5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "name": "5b44e628-394e-4e7b-bbc3-d6cd4f28f15b",
  "properties": {
    "principalId": "c0833082-6cc3-4a26-a1b1-c4b5f90a981f",
    "roleDefinitionId": "/subscriptions/<SUBSCRIPTION ID>/providers/Microsoft.Authorization/roleDefinitions/fbdf93bf-df7d-467e-a4d2-9458aa1360c8",
    "scope": "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT>"
  },
  "resourceGroup": "<RESOURCE GROUP>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## <a name="access-data"></a>Доступ к данным

Далее в этом руководстве мы будем работать с виртуальной машиной.

Для выполнения этих действий вам потребуется клиент SSH. Если вы используете Windows, можно использовать клиент SSH в [подсистеме Windows для Linux](/windows/wsl/install-win10). Если вам нужна помощь в настройке ключей SSH-клиента, ознакомьтесь с разделом [Использование ключей SSH с Windows в Azure](../../virtual-machines/linux/ssh-from-windows.md) или [Как создать и использовать пару из открытого и закрытого ключей SSH для виртуальных машин Linux в Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. На портале Azure перейдите к разделу **Виртуальные машины**, выберите свою виртуальную машину Linux и вверху в разделе **Обзор** щелкните **Подключить**. Скопируйте строку подключения к виртуальной машине. 
2. Подключитесь к виртуальной машине c помощью клиента SSH.  
3. После этого вам предложат ввести **пароль**, добавленный при создании **виртуальной машины Linux**. Вы должны без проблем войти в систему.  
4. Используйте cURL для получения маркера доступа для Azure Resource Manager. 
     
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true   
    ```
 
    > [!NOTE]
    > В предыдущем запросе значение параметра resource должно точно совпадать со значением, которое ожидает Azure AD. Если используется идентификатор ресурса Resource Manager, добавьте косую черту после универсального кода ресурса (URI).
    > В следующем ответе элемент access_token сокращен для удобства восприятия.
    
    ```bash
    {"access_token":"eyJ0eXAiOi...",
     "expires_in":"3599",
     "expires_on":"1518503375",
     "not_before":"1518499475",
     "resource":"https://management.azure.com/",
     "token_type":"Bearer",
     "client_id":"1ef89848-e14b-465f-8780-bf541d325cd5"}
     ```
    
### <a name="get-access-keys-from-azure-resource-manager-to-make-cosmos-db-calls"></a>Получение ключей доступа из Azure Resource Manager для создания вызовов Cosmos DB  

Теперь с помощью CURL обратимся к Resource Manager, используя полученный на предыдущем этапе маркер доступа, и получим ключ доступа к учетной записи Cosmos DB. Получив ключ доступа, можно создать запрос к Cosmos DB. Не забудьте заменить значения параметров `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>` и `<COSMOS DB ACCOUNT NAME>` своими значениями. Замените значение `<ACCESS TOKEN>` маркером доступа, который вы получили ранее.  Если вы хотите получить ключи для чтения и записи, используйте тип операции ключа `listKeys`.  Если вы хотите получить ключи только для чтения, используйте тип операции ключа `readonlykeys`.

```bash 
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.DocumentDB/databaseAccounts/<COSMOS DB ACCOUNT NAME>/<KEY OPERATION TYPE>?api-version=2016-03-31' -X POST -d "" -H "Authorization: Bearer <ACCESS TOKEN>" 
```

> [!NOTE]
> Для текста в предыдущем URL-адресе учитывается регистр, поэтому используйте тот же регистр, в котором указано имя группы ресурсов. Кроме того, убедитесь, что используется запрос POST, а не GET, а также, что вы передали значение для регистрации максимальной длины с помощью параметра -d, который может иметь значение NULL.  

В ответе CURL будет содержаться список ключей.  Например, при получении ключей только для чтения вы увидите следующее:  

```bash 
{"primaryReadonlyMasterKey":"bWpDxS...dzQ==",
"secondaryReadonlyMasterKey":"38v5ns...7bA=="}
```

Теперь, когда у вас есть ключ доступа для учетной записи Cosmos DB, вы можете передать его в пакет SDK Cosmos DB и выполнять вызовы для получения доступа к учетной записи.  Чтобы получить краткий пример, вы можете передать ключ доступа в Azure CLI.  Вы можете получить `<COSMOS DB CONNECTION URL>` на вкладке **Обзор** в колонке учетной записи Cosmos DB на портале Azure.  Замените `<ACCESS KEY>` полученным выше значением.

```azurecli-interactive
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

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как получить доступ к базе данных Cosmos DB с помощью назначаемого системой управляемого удостоверения виртуальной машины Linux.  Дополнительные сведения о Cosmos DB см. здесь:

> [!div class="nextstepaction"]
>[Добро пожаловать в базу данных Azure Cosmos DB](../../cosmos-db/introduction.md)
