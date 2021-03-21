---
title: Использование JavaScript (Node.js) для задания списков управления доступом в Azure Data Lake Storage 2-го поколения
description: Используйте клиентскую библиотеку службы хранилища Azure Data Lake для JavaScript, чтобы управлять списками управления доступом (ACL) в учетных записях хранения с включенным иерархическое пространством имен (HNS).
author: normesta
ms.service: storage
ms.date: 03/19/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-js
ms.openlocfilehash: 21b4977102a484d8a3a680450a9cb6f77c7e3fbd
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104722758"
---
# <a name="use-javascript-sdk-in-nodejs-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Использование пакета SDK для JavaScript в Node.js для управления списками управления доступом в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать Node.js для получения, установки и обновления списков управления доступом к каталогам и файлам. 

[Пакет (диспетчер пакетов node)](https://www.npmjs.com/package/@azure/storage-file-datalake)  |  [Примеры](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples)  |  [Отправить отзыв](https://github.com/Azure/azure-sdk-for-java/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- В учетной записи хранения включено иерархическое пространство имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.6.0` или выше.

- Одно из следующих разрешений безопасности:

  - [Участник безопасности](../../role-based-access-control/overview.md#security-principal) подготовленного Azure Active Directory (AD), которому назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) в области целевого контейнера, родительской группы ресурсов или подписки.  

  - Пользователь-владелец целевого контейнера или каталога, к которому планируется применять параметры ACL. Чтобы настроить списки управления доступом рекурсивно, сюда входят все дочерние элементы в целевом контейнере или каталоге.
  
  - Ключ учетной записи хранения..

## <a name="set-up-your-project"></a>Настройка проекта

Установите Data Lake клиентскую библиотеку для JavaScript, открыв окно терминала, а затем введя следующую команду.

```javascript
npm install @azure/storage-file-datalake
```

Импортируйте `storage-file-datalake` пакет, поместив этот оператор в начало файла кода. 

```javascript
const {
AzureStorageDataLake,
DataLakeServiceClient,
StorageSharedKeyCredential
} = require("@azure/storage-file-datalake");
```

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Чтобы использовать фрагменты кода в этой статье, необходимо создать экземпляр **даталакесервицеклиент** , представляющий учетную запись хранения. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Подключение с помощью Azure Active Directory (AD)

> [!NOTE]
> Если вы используете Azure Active Directory (Azure AD) для авторизации доступа, убедитесь, что участнику безопасности назначена [роль владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Дополнительные сведения о применении разрешений ACL и их влиянии на их изменение см. в разделе  [модель контроля доступа в Azure Data Lake Storage 2-го поколения](./data-lake-storage-access-control-model.md).

Вы можете использовать [клиентскую библиотеку Azure Identity для JS](https://www.npmjs.com/package/@azure/identity) для проверки подлинности приложения в Azure AD.

Получите идентификатор клиента, секрет клиента и идентификатор клиента. Для этого см. статью [Получение маркера из Azure AD для авторизации запросов из клиентского приложения](../common/storage-auth-aad-app.md). В рамках этого процесса вам потребуется назначить одну из следующих ролей Azure [RBAC (Управление доступом на основе ролей)](../../role-based-access-control/overview.md) для субъекта безопасности. 

|Роль|Возможность настройки ACL|
|--|--|
|[владелец данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner);|Все каталоги и файлы в учетной записи.|
|[участник данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor);|Только каталоги и файлы, принадлежащие субъекту безопасности.|

В этом примере создается экземпляр **даталакесервицеклиент** с использованием идентификатора клиента, секрета клиента и идентификатора клиента.  

```javascript
function GetDataLakeServiceClientAD(accountName, clientID, clientSecret, tenantID) {

  const credential = new ClientSecretCredential(tenantID, clientID, clientSecret);
  
  const datalakeServiceClient = new DataLakeServiceClient(
      `https://${accountName}.dfs.core.windows.net`, credential);

  return datalakeServiceClient;             
}
```

> [!NOTE]
> Дополнительные примеры см. в документации по [клиентской библиотеке удостоверений Azure для JS](https://www.npmjs.com/package/@azure/identity) .

### <a name="connect-by-using-an-account-key"></a>Подключение с помощью ключа учетной записи

Это самый простой способ подключения к учетной записи. 

В этом примере создается экземпляр **даталакесервицеклиент** с помощью ключа учетной записи.

```javascript

function GetDataLakeServiceClient(accountName, accountKey) {

  const sharedKeyCredential = 
     new StorageSharedKeyCredential(accountName, accountKey);
  
  const datalakeServiceClient = new DataLakeServiceClient(
      `https://${accountName}.dfs.core.windows.net`, sharedKeyCredential);

  return datalakeServiceClient;             
}      

```

> [!NOTE]
> Этот метод авторизации работает только для приложений Node.js. Если вы планируете выполнять код в браузере, можно авторизовать с помощью Azure Active Directory (AD).

## <a name="get-and-set-a-directory-acl"></a>Получение и задание ACL каталога

В этом примере получается и затем задается список ACL для каталога с именем `my-directory` . В этом примере предоставляются права владельца «чтение», «запись» и «выполнение», которая предоставляет группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ для чтения.

> [!NOTE]
> Если приложение разрешает доступ с помощью Azure Active Directory (Azure AD), убедитесь, что участнику безопасности, используемому приложением для авторизации доступа, назначена [роль владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Дополнительные сведения о применении разрешений ACL и последствиях их изменения см. на странице [Контроль доступа в Azure Data Lake Storage 2-го поколения](./data-lake-storage-access-control.md).

```javascript
async function ManageDirectoryACLs(fileSystemClient) {

    const directoryClient = fileSystemClient.getDirectoryClient("my-directory"); 
    const permissions = await directoryClient.getAccessControl();

    console.log(permissions.acl);

    const acl = [
    {
      accessControlType: "user",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: true,
        execute: true
      }
    },
    {
      accessControlType: "group",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: false,
        execute: true
      }
    },
    {
      accessControlType: "other",
      entityId: "",
      defaultScope: false,
      permissions: {
        read: true,
        write: true,
        execute: false
      }

    }

  ];

  await directoryClient.setAccessControl(acl);
}
```

Также можно получить и задать список управления доступом для корневого каталога контейнера. Чтобы получить корневой каталог, передайте в `/` метод **Даталакефилесистемклиент. жетдиректориклиент** пустую строку ().

### <a name="get-and-set-a-file-acl"></a>Получение и задание ACL файла

Этот пример получает и затем задает список управления доступом для файла с именем `upload-file.txt` . В этом примере предоставляются права владельца «чтение», «запись» и «выполнение», которая предоставляет группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ для чтения.

> [!NOTE]
> Если приложение разрешает доступ с помощью Azure Active Directory (Azure AD), убедитесь, что участнику безопасности, используемому приложением для авторизации доступа, назначена [роль владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Дополнительные сведения о применении разрешений ACL и последствиях их изменения см. на странице [Контроль доступа в Azure Data Lake Storage 2-го поколения](./data-lake-storage-access-control.md).

```javascript
async function ManageFileACLs(fileSystemClient) {

  const fileClient = fileSystemClient.getFileClient("my-directory/uploaded-file.txt"); 
  const permissions = await fileClient.getAccessControl();

  console.log(permissions.acl);

  const acl = [
  {
    accessControlType: "user",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: true,
      execute: true
    }
  },
  {
    accessControlType: "group",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: false,
      execute: true
    }
  },
  {
    accessControlType: "other",
    entityId: "",
    defaultScope: false,
    permissions: {
      read: true,
      write: true,
      execute: false
    }

  }

];

await fileClient.setAccessControl(acl);        
}
```

## <a name="see-also"></a>См. также раздел

- [Пакет (диспетчер пакетов узла)](https://www.npmjs.com/package/@azure/storage-file-datalake)
- [Примеры](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-file-datalake/samples)
- [Оставить отзыв](https://github.com/Azure/azure-sdk-for-java/issues)
- [Модель контроля доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
- [Списки управления доступом (ACL) в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)