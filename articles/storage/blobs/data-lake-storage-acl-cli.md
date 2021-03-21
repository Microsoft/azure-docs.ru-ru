---
title: Использование Azure CLI для задания списков управления доступом в Azure Data Lake Storage 2-го поколения
description: Используйте Azure CLI для управления списками управления доступом (ACL) в учетных записях хранения, имеющих иерархическое пространство имен.
services: storage
author: normesta
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.topic: how-to
ms.date: 02/17/2021
ms.author: normesta
ms.reviewer: prishet
ms.custom: devx-track-azurecli
ms.openlocfilehash: 5ec7d2b243a5eadab2d22dea14ebeac8eabb1722
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103563170"
---
# <a name="use-azure-cli-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Использование Azure CLI для управления списками ACL в Azure Data Lake Storage 2-го поколения

В этой статье показано, как использовать [интерфейс Command-Line Azure (CLI)](/cli/azure/) для получения, установки и обновления списков управления доступом к каталогам и файлам.

Наследование ACL уже доступно для новых дочерних элементов, созданных в родительском каталоге. Но можно также добавлять, обновлять и удалять списки управления доступом для существующих дочерних элементов родительского каталога без необходимости вносить эти изменения отдельно для каждого дочернего элемента.

[Справочные материалы](/cli/azure/storage/fs/access)  |  [Примеры](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)  |  [Отправить отзыв](https://github.com/Azure/azure-cli-extensions/issues)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

- Учетная запись хранения с включенным иерархическим пространством имен. Выполните [эти](create-data-lake-storage-account.md) инструкции, чтобы создать учетную запись.

- Azure CLI версии`2.14.0` или выше.

- Одно из следующих разрешений безопасности:

  - Подготовленный [субъект безопасности](../../role-based-access-control/overview.md#security-principal) Azure Active Directory (Azure AD), которому назначена роль [владельца данных BLOB-объекта хранилища](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) в области целевого контейнера, родительской группы ресурсов или подписки.  

  - Пользователь-владелец целевого контейнера или каталога, к которому планируется применять параметры ACL. Чтобы настроить списки управления доступом рекурсивно, сюда входят все дочерние элементы в целевом контейнере или каталоге.
  
  - Ключ учетной записи хранения.

## <a name="ensure-that-you-have-the-correct-version-of-azure-cli-installed"></a>Убедитесь, что у вас установлена правильная версия Azure CLI

1. Откройте [Azure Cloud Shell](../../cloud-shell/overview.md) или, если вы [установили](/cli/azure/install-azure-cli) Azure CLI локально, командное консольное приложение (например, Windows PowerShell).

2. Убедитесь, что установлена версия Azure CLI`2.14.0` или выше, выполнив следующую команду.

   ```azurecli
    az --version
   ```

   Если ваша версия Azure CLI ниже чем `2.14.0`, установите более позднюю версию. Подробнее см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="connect-to-the-account"></a>Подключение к учетной записи

1. Если вы используете Azure CLI локально, выполните команду для входа.

   ```azurecli
   az login
   ```

   Если в CLI можно запустить браузер по умолчанию, откроется браузер со страницей входа.

   В противном случае самостоятельно откройте в браузере страницу [https://aka.ms/devicelogin](https://aka.ms/devicelogin) и введите код авторизации, отображаемый в терминале. Затем выполните вход в браузере с помощью учетных данных.

   Дополнительные сведения о различных методах аутентификации см. на странице [Авторизация доступа к данным BLOB-объектов или очередей с помощью Azure CLI](./authorize-data-operations-cli.md).

2. Если удостоверение связано с несколькими подписками, установите активную подписку в качестве подписки учетной записи хранения, в которой будет размещен статический веб-сайт.

   ```azurecli
   az account set --subscription <subscription-id>
   ```

   Замените значение заполнителя `<subscription-id>` идентификатором своей подписки.

> [!NOTE]
> В примере, приведенном в этой статье, показана авторизация Azure Active Directory (Azure AD). Дополнительные сведения о методах аутентификации см. на странице [Авторизация доступа к данным BLOB-объектов или очередей с помощью Azure CLI](./authorize-data-operations-cli.md).

## <a name="get-acls"></a>Получение списков управления доступом

Получите список ACL для **каталога** с помощью команды `az storage fs access show`.

В этом примере список ACL сначала извлекается для каталога, а затем выводится на консоль.

```azurecli
az storage fs access show -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

Получите права доступа к **файлу** с помощью команды `az storage fs access show`. 

В этом примере список ACL сначала извлекается для файла, а затем выводится на консоль.

```azurecli
az storage fs access show -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

На следующем изображении показаны выходные данные после получения списка ACL для каталога.

![Получение выходных данных ACL](./media/data-lake-storage-directory-file-acl-cli/get-acl.png)

В этом примере владелец имеет разрешения на чтение, запись и выполнение. Группа-владелец имеет разрешения только на чтение и выполнение. Дополнительные сведения о списках управления доступом в Azure Data Lake Storage 2-го поколения см. на [этой странице](data-lake-storage-access-control.md).

## <a name="set-acls"></a>Настройка списков управления доступом

При *задании* ACL **заменяется** весь список ACL, включая все его записи. Если вы хотите изменить уровень разрешений субъекта безопасности или добавить новый субъект безопасности в список управления доступом, не затрагивая другие существующие записи, следует *Обновить* список управления доступом. Чтобы обновить список управления доступом вместо его замены, см. раздел [Обновление списков управления доступом](#update-acls) этой статьи.  

Если вы решили *задать* список управления доступом, необходимо добавить запись для пользователя-владельца, запись для группы владельцев и запись для всех других пользователей. Дополнительные сведения о пользователях-владельце, группе владельцев и других пользователях см. в разделе [Пользователи и удостоверения](data-lake-storage-access-control.md#users-and-identities).

В этом разделе показано, как:

- Настройка списка ACL
- Рекурсивная настройка ACL

### <a name="set-an-acl"></a>Настройка списка ACL

Используйте команду `az storage fs access set`, чтобы задать список ACL для **каталога**. 

В этом примере список ACL задается для каталога владельца, группы-владельца или других пользователей, а затем выводится на консоль.

```azurecli
az storage fs access set --acl "user::rw-,group::rw-,other::-wx" -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

В этом примере список ACL *по умолчанию* задается в каталоге для владельца, группы-владельца или других пользователей, а затем выводится на консоль.

```azurecli
az storage fs access set --acl "default:user::rw-,group::rw-,other::-wx" -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

Используйте команду `az storage fs access set`, чтобы задать список ACL для **файла**. 

В этом примере список ACL задается для файла владельца, группы-владельца или других пользователей, а затем выводится на консоль.

```azurecli
az storage fs access set --acl "user::rw-,group::rw-,other::-wx" -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Чтобы задать список управления доступом для определенной группы или пользователя, используйте соответствующие идентификаторы объектов. Например, `group:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` или `user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

На следующем изображении показаны выходные данные после установки списка ACL для файла.

![Получить выходные данные ACL 2](./media/data-lake-storage-directory-file-acl-cli/set-acl-file.png)

В этом примере владелец и группа-владелец имеют разрешения только на чтение и запись. У всех остальных пользователей есть разрешения на запись и выполнение. Дополнительные сведения о списках управления доступом в Azure Data Lake Storage 2-го поколения см. на [этой странице](data-lake-storage-access-control.md).

### <a name="set-acls-recursively"></a>Рекурсивная настройка ACL

Настройте списки управления доступом рекурсивно с помощью команды [AZ Storage FS Set-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_set_recursive) .

В этом примере задается список ACL для каталога с именем `my-parent-directory` . Эти записи предоставляют владельцам разрешения на чтение, запись и выполнение, предоставляя группе-владельцу только разрешения на чтение и выполнение, а также предоставляет всем остальным доступ. Последняя запись ACL в этом примере предоставляет пользователю с ИДЕНТИФИКАТОРом "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" разрешения на чтение и выполнение.

```azurecli
az storage fs access set-recursive --acl "user::rwx,group::r-x,other::---,user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Если вы хотите задать запись ACL **по умолчанию** , добавьте префикс `default:` в каждую запись. Например, `default:user::rwx` или `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x`.

## <a name="update-acls"></a>Обновление списков управления доступом

При *обновлении* списка управления доступом необходимо изменить список управления доступом вместо замены списка управления доступом. Например, можно добавить нового субъекта безопасности в список управления доступом, не затрагивая другие субъекты безопасности, перечисленные в списке ACL.  Чтобы заменить список управления доступом вместо обновления, см. раздел [Настройка списков управления доступом](#set-acls) этой статьи.

Чтобы обновить список управления доступом, создайте новый объект ACL с записью ACL, которую требуется обновить, а затем используйте этот объект в операции Update ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для обновления.

В этом разделе показано, как:

- Обновление списка ACL
- Рекурсивно обновлять списки ACL

### <a name="update-an-acl"></a>Обновление списка ACL

Другим способом установки этого разрешения является использование команды `az storage fs access set`. 

Обновите список ACL для каталога или файла, установив параметр `-permissions` в краткой форме ACL.

В этом примере список ACL обновляется для **каталога**.

```azurecli
az storage fs access set --permissions rwxrwxrwx -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

В этом примере список ACL обновляется для **файла**.

```azurecli
az storage fs access set --permissions rwxrwxrwx -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Чтобы обновить список управления доступом для определенной группы или пользователя, используйте соответствующие идентификаторы объектов. Например, `group:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` или `user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

Вы также можете сделать обновление для владельца и группы каталога или файла, установив параметры `--owner` или `group` для пользовательского имени субъекта-пользователя или идентификатора сущности.

В этом примере изменяется владелец каталога.

```azurecli
az storage fs access set --owner xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p my-directory -f my-file-system --account-name mystorageaccount --auth-mode login
```

В этом примере изменяется владелец файла. 

```azurecli
az storage fs access set --owner xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p my-directory/upload.txt -f my-file-system --account-name mystorageaccount --auth-mode login

```

### <a name="update-acls-recursively"></a>Рекурсивно обновлять списки ACL

Рекурсивно обновите списки управления доступом с помощью команды  [AZ Storage FS доступ Update-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_update_recursive) .

В этом примере обновляется запись ACL с разрешением на запись.

```azurecli
az storage fs access update-recursive --acl "user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:rwx" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Если вы хотите обновить запись ACL **по умолчанию** , добавьте префикс `default:` в каждую запись. Например, `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:r-x`.

## <a name="remove-acl-entries-recursively"></a>Рекурсивное удаление записей ACL

Можно рекурсивно удалить одну или несколько записей списка ACL. Чтобы удалить запись ACL, создайте новый объект ACL для удаляемой записи ACL, а затем используйте этот объект в операции удаления списка ACL. Не изменяйте существующий список ACL, просто предоставьте записи ACL для удаления.

Удалите записи списков ACL с помощью команды [AZ Storage FS доступ recurs-recursive](/cli/azure/storage/fs/access#az_storage_fs_access_remove_recursive) .

В этом примере удаляется запись ACL из корневого каталога контейнера.  

```azurecli
az storage fs access remove-recursive --acl "user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login
```

> [!NOTE]
> Если вы хотите удалить запись ACL **по умолчанию** , добавьте префикс `default:` в каждую запись. Например, `default:user:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

## <a name="recover-from-failures"></a>Восстановление после сбоев

При рекурсивном изменении списков ACL могут возникать ошибки времени выполнения или разрешения. Для ошибок времени выполнения перезапустите процесс с самого начала. Ошибки разрешений могут возникать, если субъект безопасности не имеет достаточных разрешений для изменения списка управления доступом к каталогу или файлу в изменяемой иерархии каталогов. Устраните проблему с разрешениями, а затем выберите либо возобновление процесса с точки сбоя с помощью токена продолжения, либо перезапуск процесса с начала. Если вы предпочитаете перезапустить с самого начала, вам не нужно использовать токен продолжения. Вы можете повторно применить записи ACL без отрицательного воздействия.

В случае сбоя можно вернуть токен продолжения, задав `--continue-on-failure` для параметра значение `false` . После устранения ошибок можно возобновить процесс с точки сбоя, выполнив команду еще раз, а затем присвоив `--continuation` параметру маркер продолжения.

```azurecli
az storage fs access set-recursive --acl "user::rw-,group::r-x,other::---" --continue-on-failure false --continuation xxxxxxx -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login  
```

Если вы хотите, чтобы процесс завершился без прерывания с ошибками разрешений, можно указать это.

Чтобы убедиться, что процесс завершается без прерывания, присвойте `--continue-on-failure` параметру значение `true` .

```azurecli
az storage fs access set-recursive --acl "user::rw-,group::r-x,other::---" --continue-on-failure true --continuation xxxxxxx -p my-parent-directory/ -f my-container --account-name mystorageaccount --auth-mode login  
```

[!INCLUDE [updated-for-az](../../../includes/recursive-acl-best-practices.md)]

## <a name="see-also"></a>См. также раздел

- [Примеры](https://github.com/Azure/azure-cli/blob/dev/src/azure-cli/azure/cli/command_modules/storage/docs/ADLS%20Gen2.md)
- [Отправка отзывов](https://github.com/Azure/azure-cli-extensions/issues)
- [Известные проблемы](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
- [Модель контроля доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
- [Списки управления доступом (ACL) в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)