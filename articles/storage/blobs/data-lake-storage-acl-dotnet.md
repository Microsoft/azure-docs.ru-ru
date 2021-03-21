---
title: Использование .NET для установки списков управления доступом в Azure Data Lake Storage 2-го поколения
description: Используйте .NET для управления списками управления доступом (ACL) в учетных записях хранения с включенным иерархическое пространством имен (HNS).
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-csharp
ms.openlocfilehash: 626e89d8d758d5fe31ef6c913076a9154274bb61
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100654482"
---
# <a name="use-net-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Использование .NET для управления списками ACL в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать .NET для получения, установки и обновления списков управления доступом к каталогам и файлам.

Наследование ACL уже доступно для новых дочерних элементов, созданных в родительском каталоге. Но можно также добавлять, обновлять и удалять списки управления доступом для существующих дочерних элементов родительского каталога без необходимости вносить эти изменения отдельно для каждого дочернего элемента.

[Пакет (NuGet)](https://www.nuget.org/packages/Azure.Storage.Files.DataLake)  |  [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake)  |  Пример рекурсивного [ACL](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0)  |  [Справочник](/dotnet/api/azure.storage.files.datalake)  |  по API Сопоставление Gen1 с [Gen2](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake/GEN1_GEN2_MAPPING.md)  |  [Отправить отзыв](https://github.com/Azure/azure-sdk-for-net/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- В учетной записи хранения включено иерархическое пространство имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.6.0` или выше.

- Одно из следующих разрешений безопасности:

  - [Участник безопасности](../../role-based-access-control/overview.md#security-principal) подготовленного Azure Active Directory (AD), которому назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) в области целевого контейнера, родительской группы ресурсов или подписки.  

  - Пользователь-владелец целевого контейнера или каталога, к которому планируется применять параметры ACL. Чтобы настроить списки управления доступом рекурсивно, сюда входят все дочерние элементы в целевом контейнере или каталоге.
  
  - Ключ учетной записи хранения.

## <a name="set-up-your-project"></a>Настройка проекта

Чтобы приступить к работе, установите пакет NuGet [Azure. Storage. Files. Data Lake](https://www.nuget.org/packages/Azure.Storage.Files.DataLake/) .

1. Откройте командное окно (например, Windows PowerShell).

2. В каталоге проекта установите пакет Azure. Storage. Files. Data Lake Preview с помощью `dotnet add package` команды.

   ```console
   dotnet add package Azure.Storage.Files.DataLake -v 12.6.0 -s https://pkgs.dev.azure.com/azure-sdk/public/_packaging/azure-sdk-for-net/nuget/v3/index.json
   ```

   Затем добавьте эти операторы using в начало файла кода.

   ```csharp
   using Azure;
   using Azure.Core;
   using Azure.Storage;
   using Azure.Storage.Files.DataLake;
   using Azure.Storage.Files.DataLake.Models;
   using System.Collections.Generic;
   using System.Threading.Tasks;
   ```

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Чтобы использовать фрагменты кода в этой статье, необходимо создать экземпляр [даталакесервицеклиент](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) , представляющий учетную запись хранения. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Подключение с помощью Azure Active Directory (AD)

> [!NOTE]
> Если вы используете Azure Active Directory (Azure AD) для авторизации доступа, убедитесь, что участнику безопасности назначена [роль владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Дополнительные сведения о применении разрешений ACL и их влиянии на их изменение см. в разделе  [модель контроля доступа в Azure Data Lake Storage 2-го поколения](./data-lake-storage-access-control-model.md).

Для проверки подлинности приложения в Azure AD можно использовать [клиентскую библиотеку удостоверений Azure для .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) .

После установки пакета добавьте этот оператор using в начало файла кода.

```csharp
using Azure.Identity;
```

Получите идентификатор клиента, секрет клиента и идентификатор клиента. Для этого см. статью [Получение маркера из Azure AD для авторизации запросов из клиентского приложения](../common/storage-auth-aad-app.md). В рамках этого процесса вам потребуется назначить одну из следующих ролей Azure [RBAC (Управление доступом на основе ролей)](../../role-based-access-control/overview.md) для субъекта безопасности. 

|Роль|Возможность настройки ACL|
|--|--|
|[владелец данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner);|Все каталоги и файлы в учетной записи.|
|[участник данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor);|Только каталоги и файлы, принадлежащие субъекту безопасности.|

В этом примере создается экземпляр [даталакесервицеклиент](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) с использованием идентификатора клиента, секрета клиента и идентификатора клиента.  

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Authorize_DataLake.cs" id="Snippet_AuthorizeWithAAD":::

> [!NOTE]
> Дополнительные примеры см. в документации по [клиентской библиотеке удостоверений Azure для .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/identity/Azure.Identity) .

### <a name="connect-by-using-an-account-key"></a>Подключение с помощью ключа учетной записи

Это самый простой способ подключения к учетной записи.

В этом примере создается экземпляр [даталакесервицеклиент](/dotnet/api/azure.storage.files.datalake.datalakeserviceclient) с помощью ключа учетной записи.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/Authorize_DataLake.cs" id="Snippet_AuthorizeWithKey":::

## <a name="set-acls"></a>Настройка списков управления доступом

При *задании* ACL **заменяется** весь список ACL, включая все его записи. Если вы хотите изменить уровень разрешений субъекта безопасности или добавить новый субъект безопасности в список управления доступом, не затрагивая другие существующие записи, следует *Обновить* список управления доступом. Чтобы обновить список управления доступом вместо его замены, см. раздел [Обновление списков управления доступом](#update-acls) этой статьи.  

Если вы решили *задать* список управления доступом, необходимо добавить запись для пользователя-владельца, запись для группы владельцев и запись для всех других пользователей. Дополнительные сведения о пользователях-владельце, группе владельцев и других пользователях см. в разделе [Пользователи и удостоверения](data-lake-storage-access-control.md#users-and-identities).

В этом разделе показано, как:

- Задание ACL для каталога
- Задание списка управления доступом для файла
- Рекурсивная настройка ACL

### <a name="set-the-acl-of-a-directory"></a>Задание ACL для каталога

Получите список управления доступом (ACL) каталога, вызвав метод [даталакедиректориклиент. жетакцессконтроласинк](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.getaccesscontrolasync) и ЗАДАВ список ACL, вызвав метод [даталакедиректориклиент. сетакцессконтроллист](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist) .

Этот пример возвращает и задает список ACL для каталога с именем `my-directory` . Эта строка `user::rwx,group::r-x,other::rw-` предоставляет владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным разрешение на чтение и запись.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_ACLDirectory":::

Также можно получить и задать список управления доступом для корневого каталога контейнера. Чтобы получить корневой каталог, передайте в `""` метод [Даталакефилесистемклиент. жетдиректориклиент](/dotnet/api/azure.storage.files.datalake.datalakefilesystemclient.getdirectoryclient) пустую строку ().

### <a name="set-the-acl-of-a-file"></a>Задание списка управления доступом для файла

Получите список управления доступом (ACL) файла, вызвав метод [даталакефилеклиент. жетакцессконтроласинк](/dotnet/api/azure.storage.files.datalake.datalakefileclient.getaccesscontrolasync) и ЗАДАВ список ACL, вызвав метод [даталакефилеклиент. сетакцессконтроллист](/dotnet/api/azure.storage.files.datalake.datalakefileclient.setaccesscontrollist) .

Этот пример возвращает и задает список управления доступом для файла с именем `my-file.txt` . Эта строка `user::rwx,group::r-x,other::rw-` предоставляет владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным разрешение на чтение и запись.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_FileACL":::

### <a name="set-acls-recursively"></a>Рекурсивная настройка ACL

Настройте списки управления доступом рекурсивно, вызвав метод **даталакедиректориклиент. сетакцессконтролрекурсивеасинк** . Передайте этот метод [](/dotnet/api/system.collections.generic.list-1) в список [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Каждый [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) определяет запись ACL.

Если вы хотите задать запись ACL **по умолчанию** , можно установить свойство [пасакцессконтролитем. DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) для [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) в **значение true**. 

В этом примере задается список ACL для каталога с именем `my-parent-directory` . Этот метод принимает логический параметр с именем `isDefaultScope` , указывающий, следует ли ЗАДАВАТЬ ACL по умолчанию. Этот параметр используется в конструкторе объекта [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Записи списка управления доступом предоставляют владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ. Последняя запись списка управления доступом в этом примере предоставляет конкретному пользователю с ИДЕНТИФИКАТОРом объекта "" XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX "разрешения на чтение и выполнение.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_SetACLRecursively":::

Пример рекурсивного задания списков ACL в пакетах путем указания размера пакета [см. в примере .NET](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0).

## <a name="update-acls"></a>Обновление списков управления доступом

При *обновлении* списка управления доступом необходимо изменить список управления доступом вместо замены списка управления доступом. Например, можно добавить нового субъекта безопасности в список управления доступом, не затрагивая другие субъекты безопасности, перечисленные в списке ACL.  Чтобы заменить список управления доступом вместо обновления, см. раздел [Настройка списков управления доступом](#set-acls) этой статьи.

В этом разделе показано, как:

- Обновление списка ACL
- Рекурсивно обновлять списки ACL

### <a name="update-an-acl"></a>Обновление списка ACL

Сначала получите список ACL для каталога, вызвав метод [даталакедиректориклиент. жетакцессконтроласинк](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.getaccesscontrolasync) . Скопируйте список записей ACL в новый [список](/dotnet/api/system.collections.generic.list-1) объектов [пасакцессконтрол](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrol) . Затем найдите запись, которую требуется обновить, и замените ее в списке. Задайте список ACL, вызвав метод [даталакедиректориклиент. сетакцессконтроллист](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist) .

В этом примере обновляется корневой список ACL контейнера путем замены записи ACL для всех остальных пользователей. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_UpdateACL":::

### <a name="update-acls-recursively"></a>Рекурсивно обновлять списки ACL

Чтобы рекурсивно обновить список ACL, создайте новый объект ACL с записью ACL, которую требуется обновить, а затем используйте этот объект в операции Update ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для обновления.

Рекурсивно обновите список ACL, вызвав метод **даталакедиректориклиент. упдатеакцессконтролрекурсивеасинк** .  Передайте этот метод [](/dotnet/api/system.collections.generic.list-1) в список [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Каждый [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) определяет запись ACL.

Если требуется обновить запись ACL **по умолчанию** , можно установить свойство [пасакцессконтролитем. DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) для [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) в **значение true**.

В этом примере обновляется запись ACL с разрешением на запись. Этот метод принимает логический параметр с именем `isDefaultScope` , указывающий, следует ли обновлять ACL по умолчанию. Этот параметр используется в конструкторе объекта [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_UpdateACLsRecursively":::

Пример, который рекурсивно обновляет списки ACL в пакетах путем указания размера пакета, см. в [примере](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0).NET.

## <a name="remove-acl-entries"></a>Удалить записи ACL

Можно удалить одну или несколько записей ACL. В этом разделе показано, как:

- Удаление записи ACL
- Рекурсивное удаление записей ACL

### <a name="remove-an-acl-entry"></a>Удаление записи ACL

Сначала получите список ACL для каталога, вызвав метод [даталакедиректориклиент. жетакцессконтроласинк](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.getaccesscontrolasync) . Скопируйте список записей ACL в новый [список](/dotnet/api/system.collections.generic.list-1) объектов [пасакцессконтрол](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrol) . Затем найдите запись, которую необходимо удалить, и вызовите метод [Remove](/dotnet/api/system.collections.ilist.remove) коллекции. Установите обновленный список ACL, вызвав метод [даталакедиректориклиент. сетакцессконтроллист](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist) .

В этом примере обновляется корневой список ACL контейнера путем замены записи ACL для всех остальных пользователей. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_RemoveACLEntry":::

### <a name="remove-acl-entries-recursively"></a>Рекурсивное удаление записей ACL

Чтобы рекурсивно удалить записи ACL, создайте новый объект ACL для удаляемой записи ACL, а затем используйте этот объект в операции удаления списка ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для удаления. 

Удалите записи ACL, вызвав метод **даталакедиректориклиент. ремовеакцессконтролрекурсивеасинк** . Передайте этот метод [](/dotnet/api/system.collections.generic.list-1) в список [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem). Каждый [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) определяет запись ACL. 

Если требуется удалить запись ACL **по умолчанию** , можно установить свойство [пасакцессконтролитем. DefaultScope](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem.defaultscope#Azure_Storage_Files_DataLake_Models_PathAccessControlItem_DefaultScope) для [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem) в **значение true**. 

В этом примере удаляется запись ACL из списка ACL каталога с именем `my-parent-directory` . Этот метод принимает логический параметр с именем `isDefaultScope` , который указывает, следует ли удалить запись из ACL по умолчанию. Этот параметр используется в конструкторе объекта [пасакцессконтролитем](/dotnet/api/azure.storage.files.datalake.models.pathaccesscontrolitem).

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_RemoveACLRecursively":::

Пример, в котором рекурсивно удаляются списки ACL в пакетах путем указания размера пакета, [см. в примере .NET](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0).

## <a name="recover-from-failures"></a>Восстановление после сбоев

При рекурсивном изменении списков ACL могут возникать ошибки времени выполнения или разрешения. Для ошибок времени выполнения перезапустите процесс с самого начала. Ошибки разрешений могут возникать, если субъект безопасности не имеет достаточных разрешений для изменения списка управления доступом к каталогу или файлу в изменяемой иерархии каталогов. Устраните проблему с разрешениями, а затем выберите либо возобновление процесса с точки сбоя с помощью токена продолжения, либо перезапуск процесса с начала. Если вы предпочитаете перезапустить с самого начала, вам не нужно использовать токен продолжения. Вы можете повторно применить записи ACL без отрицательного воздействия.

Этот пример возвращает токен продолжения в случае сбоя. Приложение может снова вызвать этот пример метода после устранения ошибки и передать маркер продолжения. Если этот метод примера вызывается впервые, приложение может передать значение `null` для параметра токен продолжения. 

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_ResumeContinuationToken":::

Пример рекурсивного задания списков ACL в пакетах путем указания размера пакета [см. в примере .NET](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0).

Если вы хотите, чтобы процесс завершился без прерывания с ошибками разрешений, можно указать это.

Чтобы убедиться в том, что процесс завершается без прерывания, передайте объект **акцессконтролчанжедоптионс** и задайте для свойства **континуеонфаилуре** этого объекта значение ``true`` .

В этом примере записи ACL задаются рекурсивно. Если этот код вызывает ошибку разрешения, он записывает этот сбой и возобновляет выполнение. В этом примере на консоль выводится число сбоев.

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/ACL_DataLake.cs" id="Snippet_ContinueOnFailure":::

Пример рекурсивного задания списков ACL в пакетах путем указания размера пакета [см. в примере .NET](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Frecursiveaclpr.blob.core.windows.net%2Fprivatedrop%2FRecursive-Acl-Sample-Net.zip%3Fsv%3D2019-02-02%26st%3D2020-08-24T07%253A45%253A28Z%26se%3D2021-09-25T07%253A45%253A00Z%26sr%3Db%26sp%3Dr%26sig%3D2GI3f0KaKMZbTi89AgtyGg%252BJePgNSsHKCL68V6I5W3s%253D&data=02%7C01%7Cnormesta%40microsoft.com%7C6eae76c57d224fb6de8908d848525330%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637338865714571853&sdata=%2FWom8iI3DSDMSw%2FfYvAaQ69zbAoqXNTQ39Q9yVMnASA%3D&reserved=0).

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>См. также раздел

- [Справочная документация по API](/dotnet/api/azure.storage.files.datalake)
- [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Storage.Files.DataLake)
- [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake)
- [Сопоставление 1-го и 2-го поколения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Files.DataLake/GEN1_GEN2_MAPPING.md)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Оставить отзыв](https://github.com/Azure/azure-sdk-for-net/issues)
- [Модель контроля доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
- [Списки управления доступом (ACL) в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)