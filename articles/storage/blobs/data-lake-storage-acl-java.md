---
title: Использование Java для задания списков управления доступом в Azure Data Lake Storage 2-го поколения
description: Используйте библиотеки службы хранилища Azure для Java, чтобы управлять списками управления доступом (ACL) в учетных записях хранения с включенным иерархическое пространством имен (HNS).
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.custom: devx-track-java
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.openlocfilehash: e7d6156fe5cd8ab32ff159bda64e0c06cfbac406
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100654477"
---
# <a name="use-java-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Использование Java для управления списками ACL в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать Java для получения, установки и обновления списков управления доступом к каталогам и файлам. 

Наследование ACL уже доступно для новых дочерних элементов, созданных в родительском каталоге. Но можно также добавлять, обновлять и удалять списки управления доступом для существующих дочерних элементов родительского каталога без необходимости вносить эти изменения отдельно для каждого дочернего элемента. 

[Пакет (Maven)](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake)  |  [Примеры](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake)  |  [Справочник](/java/api/overview/azure/storage-file-datalake-readme)  |  по API Сопоставление Gen1 с [Gen2](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)  |  [Отправить отзыв](https://github.com/Azure/azure-sdk-for-java/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- В учетной записи хранения включено иерархическое пространство имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.6.0` или выше.

- Одно из следующих разрешений безопасности:

  - [Участник безопасности](../../role-based-access-control/overview.md#security-principal) подготовленного Azure Active Directory (AD), которому назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) в области целевого контейнера, родительской группы ресурсов или подписки.  

  - Пользователь-владелец целевого контейнера или каталога, к которому планируется применять параметры ACL. Чтобы настроить списки управления доступом рекурсивно, сюда входят все дочерние элементы в целевом контейнере или каталоге.
  
  - Ключ учетной записи хранения.

## <a name="set-up-your-project"></a>Настройка проекта

Чтобы приступить к работе, откройте [эту страницу](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake) и найдите последнюю версию библиотеки Java. Затем откройте файл *pom.xml* в текстовом редакторе. Добавьте элемент зависимости, который ссылается на эту версию.

Если вы планируете выполнять проверку подлинности клиентского приложения с помощью Azure Active Directory (AD), добавьте зависимость в клиентскую библиотеку секретного кода Azure. См. раздел  [Добавление пакета секретной клиентской библиотеки в проект](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity#adding-the-package-to-your-project).

Затем добавьте эти операторы Imports в файл кода.

```java
import com.azure.storage.common.StorageSharedKeyCredential;
import com.azure.storage.file.datalake.DataLakeDirectoryClient;
import com.azure.storage.file.datalake.DataLakeFileClient;
import com.azure.storage.file.datalake.DataLakeFileSystemClient;
import com.azure.storage.file.datalake.DataLakeServiceClient;
import com.azure.storage.file.datalake.DataLakeServiceClientBuilder;
import com.azure.storage.file.datalake.models.ListPathsOptions;
import com.azure.storage.file.datalake.models.PathItem;
import com.azure.storage.file.datalake.models.AccessControlChangeCounters;
import com.azure.storage.file.datalake.models.AccessControlChangeResult;
import com.azure.storage.file.datalake.models.AccessControlType;
import com.azure.storage.file.datalake.models.PathAccessControl;
import com.azure.storage.file.datalake.models.PathAccessControlEntry;
import com.azure.storage.file.datalake.models.PathPermissions;
import com.azure.storage.file.datalake.models.PathRemoveAccessControlEntry;
import com.azure.storage.file.datalake.models.RolePermissions;
import com.azure.storage.file.datalake.options.PathSetAccessControlRecursiveOptions;
```

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Чтобы использовать фрагменты кода в этой статье, необходимо создать экземпляр **даталакесервицеклиент** , представляющий учетную запись хранения. 

### <a name="connect-by-using-azure-active-directory-azure-ad"></a>Подключение с помощью Azure Active Directory (Azure AD)

Для проверки подлинности приложения в Azure AD можно использовать [клиентскую библиотеку удостоверений Azure для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity) .

Получите идентификатор клиента, секрет клиента и идентификатор клиента. Для этого см. статью [Получение маркера из Azure AD для авторизации запросов из клиентского приложения](../common/storage-auth-aad-app.md). В рамках этого процесса вам потребуется назначить одну из следующих ролей Azure [RBAC (Управление доступом на основе ролей)](../../role-based-access-control/overview.md) для субъекта безопасности. 

|Роль|Возможность настройки ACL|
|--|--|
|[владелец данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner);|Все каталоги и файлы в учетной записи.|
|[участник данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor);|Только каталоги и файлы, принадлежащие субъекту безопасности.|

В этом примере создается экземпляр **даталакесервицеклиент** с использованием идентификатора клиента, секрета клиента и идентификатора клиента.  

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/Authorize_DataLake.java" id="Snippet_AuthorizeWithAzureAD":::

> [!NOTE]
> Дополнительные примеры см. в документации по [клиентской библиотеке удостоверений Azure для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity) .

### <a name="connect-by-using-an-account-key"></a>Подключение с помощью ключа учетной записи

Это самый простой способ подключения к учетной записи. 

В этом примере создается экземпляр **даталакесервицеклиент** с помощью ключа учетной записи.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/Authorize_DataLake.java" id="Snippet_AuthorizeWithKey":::

## <a name="set-acls"></a>Настройка списков управления доступом

При *задании* ACL **заменяется** весь список ACL, включая все его записи. Если вы хотите изменить уровень разрешений субъекта безопасности или добавить новый субъект безопасности в список управления доступом, не затрагивая другие существующие записи, следует *Обновить* список управления доступом. Чтобы обновить список управления доступом вместо его замены, см. раздел [Обновление списков управления доступом](#update-acls) этой статьи.  

Если вы решили *задать* список управления доступом, необходимо добавить запись для пользователя-владельца, запись для группы владельцев и запись для всех других пользователей. Дополнительные сведения о пользователях-владельце, группе владельцев и других пользователях см. в разделе [Пользователи и удостоверения](data-lake-storage-access-control.md#users-and-identities).

В этом разделе показано, как:

- Задание ACL для каталога
- Задание списка управления доступом для файла
- Рекурсивная настройка ACL 

### <a name="set-the-acl-of-a-directory"></a>Задание ACL для каталога

В этом примере получается и затем задается список ACL для каталога с именем `my-directory` . В этом примере предоставляются права владельца «чтение», «запись» и «выполнение», которая предоставляет группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ для чтения.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ManageDirectoryACLs":::

Также можно получить и задать список управления доступом для корневого каталога контейнера. Чтобы получить корневой каталог, передайте в `""` метод **Даталакефилесистемклиент. жетдиректориклиент** пустую строку ().

### <a name="set-the-acl-of-a-file"></a>Задание списка управления доступом для файла

Этот пример получает и затем задает список управления доступом для файла с именем `upload-file.txt` . В этом примере предоставляются права владельца «чтение», «запись» и «выполнение», которая предоставляет группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ для чтения.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ManageFileACLs":::

### <a name="set-acls-recursively"></a>Рекурсивная настройка ACL

Настройте списки управления доступом рекурсивно, вызвав метод **даталакедиректориклиент. сетакцессконтролрекурсиве** . Передайте этот метод в [список](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) объектов [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) . Каждый [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) определяет запись ACL.

Если вы хотите задать запись ACL **по умолчанию** , то можно вызвать метод **сетдефаултскопе** объекта [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) и передать значение **true**. 

В этом примере задается список ACL для каталога с именем `my-parent-directory` . Этот метод принимает логический параметр с именем `isDefaultScope` , указывающий, следует ли ЗАДАВАТЬ ACL по умолчанию. Этот параметр используется в каждом вызове метода **сетдефаултскопе** объекта [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). Записи списка управления доступом предоставляют владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ. Последняя запись ACL в этом примере предоставляет пользователю с ИДЕНТИФИКАТОРом "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" разрешения на чтение и выполнение.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_SetACLRecursively":::

## <a name="update-acls"></a>Обновление списков управления доступом

При *обновлении* списка управления доступом необходимо изменить список управления доступом вместо замены списка управления доступом. Например, можно добавить нового субъекта безопасности в список управления доступом, не затрагивая другие субъекты безопасности, перечисленные в списке ACL.  Чтобы заменить список управления доступом вместо обновления, см. раздел [Настройка списков управления доступом](#set-acls) этой статьи.

В этом разделе показано, как:

- Обновление списка ACL
- Рекурсивно обновлять списки ACL

### <a name="update-an-acl"></a>Обновление списка ACL

Сначала получите список ACL для каталога, вызвав метод [пасакцессконтрол. жетакцессконтроллист](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrol.getaccesscontrollist) . Скопируйте список записей ACL в новый объект **List** типа [пасакцессконтроллистентри](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrolentry). Затем найдите запись, которую требуется обновить, и замените ее в списке. Задайте список ACL, вызвав метод [даталакедиректориклиент. сетакцессконтроллист](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist) .

В этом примере обновляется список управления доступом для каталога `my-parent-directory` с именем, заменяющий запись для всех остальных пользователей. 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_UpdateACL":::

Также можно получить и задать список управления доступом для корневого каталога контейнера. Чтобы получить корневой каталог, передайте в `""` метод **Даталакефилесистемклиент. жетдиректориклиент** пустую строку ().

### <a name="update-acls-recursively"></a>Рекурсивно обновлять списки ACL

Чтобы рекурсивно обновить список ACL, создайте новый объект ACL с записью ACL, которую требуется обновить, а затем используйте этот объект в операции Update ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для обновления.

Рекурсивно обновите списки управления доступом, вызвав метод **даталакедиректориклиент. упдатеакцессконтролрекурсиве** .  Передайте этот метод в [список](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) объектов [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) . Каждый [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) определяет запись ACL. 

Если вы хотите обновить запись ACL **по умолчанию** , то можете использовать метод **сетдефаултскопе** [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) и передать значение **true**. 

В этом примере обновляется запись ACL с разрешением на запись. Этот метод принимает логический параметр с именем `isDefaultScope` , указывающий, следует ли обновлять ACL по умолчанию. Этот параметр используется в вызове метода **сетдефаултскопе** объекта [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html). 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_UpdateACLRecursively":::

## <a name="remove-acl-entries"></a>Удалить записи ACL

Можно удалить одну или несколько записей ACL. В этом разделе показано, как:

- Удаление записи ACL
- Рекурсивное удаление записей ACL

### <a name="remove-an-acl-entry"></a>Удаление записи ACL

Сначала получите список ACL для каталога, вызвав метод [пасакцессконтрол. жетакцессконтроллист](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrol.getaccesscontrollist) . Скопируйте список записей ACL в новый объект **List** типа [пасакцессконтроллистентри](/java/api/com.azure.storage.file.datalake.models.pathaccesscontrolentry). Затем найдите запись, которую необходимо удалить, и вызовите метод **Remove** объекта **List** . Установите обновленный список ACL, вызвав метод [даталакедиректориклиент. сетакцессконтроллист](/dotnet/api/azure.storage.files.datalake.datalakedirectoryclient.setaccesscontrollist) .

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_RemoveACLEntry":::

### <a name="remove-acl-entries-recursively"></a>Рекурсивное удаление записей ACL

Чтобы рекурсивно удалить записи ACL, создайте новый объект ACL для удаляемой записи ACL, а затем используйте этот объект в операции удаления списка ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для удаления.

Удалите записи ACL, вызвав метод **даталакедиректориклиент. ремовеакцессконтролрекурсиве** . Передайте этот метод в [список](https://docs.oracle.com/javase/8/docs/api/java/util/List.html) объектов [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) . Каждый [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) определяет запись ACL. 

Если вы хотите удалить запись ACL **по умолчанию** , то можете использовать метод **сетдефаултскопе** [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) и передать значение **true**.  

В этом примере удаляется запись ACL из списка ACL каталога с именем `my-parent-directory` . Этот метод принимает логический параметр с именем `isDefaultScope` , который указывает, следует ли удалить запись из ACL по умолчанию. Этот параметр используется в вызове метода **сетдефаултскопе** объекта [пасакцессконтролентри](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html).

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_RemoveACLRecursively":::

## <a name="recover-from-failures"></a>Восстановление после сбоев

Могут возникать ошибки времени выполнения или разрешения. Для ошибок времени выполнения перезапустите процесс с самого начала. Ошибки разрешений могут возникать, если субъект безопасности не имеет достаточных разрешений для изменения списка управления доступом к каталогу или файлу в изменяемой иерархии каталогов. Устраните проблему с разрешениями, а затем выберите либо возобновление процесса с точки сбоя с помощью токена продолжения, либо перезапуск процесса с начала. Если вы предпочитаете перезапустить с самого начала, вам не нужно использовать токен продолжения. Вы можете повторно применить записи ACL без отрицательного воздействия.

Этот пример возвращает токен продолжения в случае сбоя. Приложение может снова вызвать этот пример метода после устранения ошибки и передать маркер продолжения. Если этот метод примера вызывается впервые, приложение может передать значение `null` для параметра токен продолжения.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ResumeSetACLRecursively":::

Если вы хотите, чтобы процесс завершился без прерывания с ошибками разрешений, можно указать это.

Чтобы убедиться в том, что процесс завершается без прерывания, вызовите метод **сетконтинуеонфаилуре** объекта [пассетакцессконтролрекурсивеоптионс](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-storage-file-datalake/12.3.0-beta.1/index.html) и передайте значение **true**.

В этом примере записи ACL задаются рекурсивно. Если этот код вызывает ошибку разрешения, он записывает этот сбой и возобновляет выполнение. В этом примере на консоль выводится число сбоев.

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ContinueOnFailure":::

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>См. также раздел

- [Справочная документация по API](/java/api/overview/azure/storage-file-datalake-readme)
- [Пакет (Maven)](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake)
- [Примеры](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake)
- [Сопоставление 1-го и 2-го поколения](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Оставить отзыв](https://github.com/Azure/azure-sdk-for-java/issues)
- [Модель контроля доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
- [Списки управления доступом (ACL) в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)