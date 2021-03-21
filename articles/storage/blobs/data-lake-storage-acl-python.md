---
title: Использование Python для управления списками ACL в Azure Data Lake Storage 2-го поколения
description: Используйте Python управление списками управления доступом (ACL) в учетных записях хранения, где включено иерархическое пространство имен (HNS).
author: normesta
ms.service: storage
ms.date: 02/17/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-python
ms.openlocfilehash: ba864aa1aa2462f21e05ab5e779c8e715d6bb973
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100654476"
---
# <a name="use-python-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Использование Python для управления списками ACL в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать Python для получения, установки и обновления списков управления доступом к каталогам и файлам. 

Наследование ACL уже доступно для новых дочерних элементов, созданных в родительском каталоге. Но можно также добавлять, обновлять и удалять списки управления доступом для существующих дочерних элементов родительского каталога без необходимости вносить эти изменения отдельно для каждого дочернего элемента. 

[Пакет (индекс пакета Python)](https://pypi.org/project/azure-storage-file-datalake/)  |  [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)  |  [Рекурсивные примеры ACL](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py)  |  [Справочник](/python/api/azure-storage-file-datalake/azure.storage.filedatalake)  |  по API Сопоставление Gen1 с [Gen2](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)  |  [Отправить отзыв](https://github.com/Azure/azure-sdk-for-python/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- В учетной записи хранения включено иерархическое пространство имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.6.0` или выше.

- Одно из следующих разрешений безопасности:

  - [Участник безопасности](../../role-based-access-control/overview.md#security-principal) подготовленного Azure Active Directory (AD), которому назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) в области целевого контейнера, родительской группы ресурсов или подписки.  

  - Пользователь-владелец целевого контейнера или каталога, к которому планируется применять параметры ACL. Чтобы настроить списки управления доступом рекурсивно, сюда входят все дочерние элементы в целевом контейнере или каталоге.
  
  - Ключ учетной записи хранения.

## <a name="set-up-your-project"></a>Настройка проекта

Установите клиентскую библиотеку Azure Data Lake Storage для Python с помощью [PIP](https://pypi.org/project/pip/).

```
pip install azure-storage-file-datalake
```

Добавьте эти операторы импорта в начало файла кода.

```python
import os, uuid, sys
from azure.storage.filedatalake import DataLakeServiceClient
from azure.core._match_conditions import MatchConditions
from azure.storage.filedatalake._models import ContentSettings
```

## <a name="connect-to-the-account"></a>Подключение к учетной записи

Чтобы использовать фрагменты кода в этой статье, необходимо создать экземпляр **даталакесервицеклиент** , представляющий учетную запись хранения. 

### <a name="connect-by-using-azure-active-directory-ad"></a>Подключение с помощью Azure Active Directory (AD)

> [!NOTE]
> Если вы используете Azure Active Directory (Azure AD) для авторизации доступа, убедитесь, что участнику безопасности назначена [роль владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner). Дополнительные сведения о применении разрешений ACL и их влиянии на их изменение см. в разделе  [модель контроля доступа в Azure Data Lake Storage 2-го поколения](./data-lake-storage-access-control-model.md).

Для проверки подлинности приложения в Azure AD можно использовать [клиентскую библиотеку удостоверений Azure для Python](https://pypi.org/project/azure-identity/) .

Получите идентификатор клиента, секрет клиента и идентификатор клиента. Для этого см. статью [Получение маркера из Azure AD для авторизации запросов из клиентского приложения](../common/storage-auth-aad-app.md). В рамках этого процесса вам потребуется назначить одну из следующих ролей Azure [RBAC (Управление доступом на основе ролей)](../../role-based-access-control/overview.md) для субъекта безопасности. 

|Роль|Возможность настройки ACL|
|--|--|
|[владелец данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner);|Все каталоги и файлы в учетной записи.|
|[участник данных BLOB-объектов хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor);|Только каталоги и файлы, принадлежащие субъекту безопасности.|

В этом примере создается экземпляр **даталакесервицеклиент** с использованием идентификатора клиента, секрета клиента и идентификатора клиента.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_AuthorizeWithAAD":::

> [!NOTE]
> Дополнительные примеры см. в документации по [клиентской библиотеке Azure Identity для Python](https://pypi.org/project/azure-identity/) .

### <a name="connect-by-using-an-account-key"></a>Подключение с помощью ключа учетной записи

Это самый простой способ подключения к учетной записи.

В этом примере создается экземпляр **даталакесервицеклиент** с помощью ключа учетной записи.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_AuthorizeWithKey":::
 
- Замените значение заполнителя `storage_account_name` именем вашей учетной записи хранения.

- Замените `storage_account_key` значение заполнителя ключом доступа к учетной записи хранения.

## <a name="set-acls"></a>Настройка списков управления доступом

При *задании* ACL **заменяется** весь список ACL, включая все его записи. Если вы хотите изменить уровень разрешений субъекта безопасности или добавить новый субъект безопасности в список управления доступом, не затрагивая другие существующие записи, следует *Обновить* список управления доступом. Чтобы обновить список управления доступом вместо его замены, см. раздел [Обновление списков управления доступом](#update-acls-recursively) этой статьи.  

В этом разделе показано, как:

- Задание ACL для каталога
- Задание списка управления доступом для файла

### <a name="set-the-acl-of-a-directory"></a>Задание ACL для каталога

Получите список управления доступом (ACL) каталога, вызвав метод **DataLakeDirectoryClient.get_access_control** и ЗАДАВ список ACL, вызвав метод **DataLakeDirectoryClient.set_access_control** .

Этот пример возвращает и задает список ACL для каталога с именем `my-directory` . Эта строка `rwxr-xrw-` предоставляет владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным разрешение на чтение и запись.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_ACLDirectory":::

Также можно получить и задать список управления доступом для корневого каталога контейнера. Чтобы получить корневой каталог, вызовите метод **FileSystemClient._get_root_directory_client** .

### <a name="set-the-acl-of-a-file"></a>Задание списка управления доступом для файла

Получите список управления доступом (ACL) файла, вызвав метод **DataLakeFileClient.get_access_control** и ЗАДАВ список ACL, вызвав метод **DataLakeFileClient.set_access_control** .

Этот пример возвращает и задает список управления доступом для файла с именем `my-file.txt` . Эта строка `rwxr-xrw-` предоставляет владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным разрешение на чтение и запись.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_FileACL":::

## <a name="set-acls-recursively"></a>Рекурсивная настройка ACL

При *задании* ACL **заменяется** весь список ACL, включая все его записи. Если вы хотите изменить уровень разрешений субъекта безопасности или добавить новый субъект безопасности в список управления доступом, не затрагивая другие существующие записи, следует *Обновить* список управления доступом. Чтобы обновить список управления доступом вместо его замены, см. раздел [Обновление списков ACL](#update-acls-recursively) в этой статье рекурсивно.

Настройте списки управления доступом рекурсивно, вызвав метод **DataLakeDirectoryClient.set_access_control_recursive** .

Если вы хотите задать запись ACL **по умолчанию** , добавьте строку `default:` в начало каждой строки записи ACL.

В этом примере задается список ACL для каталога с именем `my-parent-directory` .

Этот метод принимает логический параметр с именем `is_default_scope` , указывающий, следует ли ЗАДАВАТЬ ACL по умолчанию. Если этот параметр имеет значение `True` , перед списком записей ACL добавляется строка `default:` .

Записи списка управления доступом предоставляют владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ. Последняя запись списка управления доступом в этом примере предоставляет конкретному пользователю с ИДЕНТИФИКАТОРом объекта "" XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX "разрешения на чтение и выполнение. Эти записи предоставляют владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ. Последняя запись списка управления доступом в этом примере предоставляет конкретному пользователю с ИДЕНТИФИКАТОРом объекта "" XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX "разрешения на чтение и выполнение.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_SetACLRecursively":::

Пример, который рекурсивно обрабатывает списки ACL в пакетах путем указания размера пакета, [см. в примере Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py).

## <a name="update-acls-recursively"></a>Рекурсивно обновлять списки ACL

При *обновлении* списка управления доступом необходимо изменить список управления доступом вместо замены списка управления доступом. Например, можно добавить нового субъекта безопасности в список управления доступом, не затрагивая другие субъекты безопасности, перечисленные в списке ACL.  Чтобы заменить список управления доступом вместо обновления, см. раздел [Настройка списков управления доступом](#set-acls) этой статьи.

Чтобы рекурсивно обновить список ACL, создайте новый объект ACL с записью ACL, которую требуется обновить, а затем используйте этот объект в операции Update ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для обновления. Рекурсивно обновите список ACL, вызвав метод **DataLakeDirectoryClient.update_access_control_recursive** . Если вы хотите обновить запись ACL **по умолчанию** , добавьте строку `default:` в начало каждой строки записи ACL.

В этом примере обновляется запись ACL с разрешением на запись.

В этом примере задается список ACL для каталога с именем `my-parent-directory` . Этот метод принимает логический параметр с именем `is_default_scope` , указывающий, следует ли обновлять ACL по умолчанию. Если этот параметр имеет значение `True` , то обновленной записи ACL предшествует строка `default:` .  

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_UpdateACLsRecursively":::

Пример, который рекурсивно обрабатывает списки ACL в пакетах путем указания размера пакета, [см. в примере Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py).

## <a name="remove-acl-entries-recursively"></a>Рекурсивное удаление записей ACL

Можно удалить одну или несколько записей ACL. Чтобы рекурсивно удалить записи ACL, создайте новый объект ACL для удаляемой записи ACL, а затем используйте этот объект в операции удаления списка ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для удаления. 

Удалите записи ACL, вызвав метод **DataLakeDirectoryClient.remove_access_control_recursive** . Если вы хотите удалить запись ACL **по умолчанию** , добавьте строку `default:` в начало строки записи ACL. 

В этом примере удаляется запись ACL из списка ACL каталога с именем `my-parent-directory` . Этот метод принимает логический параметр с именем `is_default_scope` , который указывает, следует ли удалить запись из ACL по умолчанию. Если этот параметр имеет значение `True` , то обновленной записи ACL предшествует строка `default:` . 

```python
def remove_permission_recursively(is_default_scope):
    
    try:
        file_system_client = service_client.get_file_system_client(file_system="my-container")

        directory_client = file_system_client.get_directory_client("my-parent-directory")
              
        acl = 'user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

        if is_default_scope:
           acl = 'default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'

        directory_client.remove_access_control_recursive(acl=acl)

    except Exception as e:
     print(e)
```

Пример, который рекурсивно обрабатывает списки ACL в пакетах путем указания размера пакета, [см. в примере Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py).

## <a name="recover-from-failures"></a>Восстановление после сбоев

Могут возникать ошибки времени выполнения или разрешения. Для ошибок времени выполнения перезапустите процесс с самого начала. Ошибки разрешений могут возникать, если субъект безопасности не имеет достаточных разрешений для изменения списка управления доступом к каталогу или файлу в изменяемой иерархии каталогов. Устраните проблему с разрешениями, а затем выберите либо возобновление процесса с точки сбоя с помощью токена продолжения, либо перезапуск процесса с начала. Если вы предпочитаете перезапустить с самого начала, вам не нужно использовать токен продолжения. Вы можете повторно применить записи ACL без отрицательного воздействия.

Этот пример возвращает токен продолжения в случае сбоя. Приложение может снова вызвать этот пример метода после устранения ошибки и передать маркер продолжения. Если этот метод примера вызывается впервые, приложение может передать значение ``None`` для параметра токен продолжения.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_ResumeContinuationToken":::

Пример, который рекурсивно обрабатывает списки ACL в пакетах путем указания размера пакета, [см. в примере Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py).

Если вы хотите, чтобы процесс завершился без прерывания с ошибками разрешений, можно указать это.

Чтобы гарантировать, что процесс завершается без прерывания, не передавайте токен продолжения в метод **DataLakeDirectoryClient.set_access_control_recursive** .

В этом примере записи ACL задаются рекурсивно. Если этот код вызывает ошибку разрешения, он записывает этот сбой и возобновляет выполнение. В этом примере на консоль выводится число сбоев.

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_ContinueOnFailure":::

Пример, который рекурсивно обрабатывает списки ACL в пакетах путем указания размера пакета, [см. в примере Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/storage/azure-storage-file-datalake/samples/datalake_samples_access_control_recursive.py).

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>См. также раздел

- [Справочная документация по API](/python/api/azure-storage-file-datalake/azure.storage.filedatalake)
- [Пакет (индекс пакета Python)](https://pypi.org/project/azure-storage-file-datalake/)
- [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)
- [Сопоставление 1-го и 2-го поколения](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Оставить отзыв](https://github.com/Azure/azure-sdk-for-python/issues)
- [Модель контроля доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
- [Списки управления доступом (ACL) в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)