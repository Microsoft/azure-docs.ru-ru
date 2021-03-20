---
title: Использование Azure CLI для создания SAS для делегирования пользователя для контейнера или большого двоичного объекта
titleSuffix: Azure Storage
description: Узнайте, как создать SAS для делегирования пользователей с учетными данными Azure Active Directory с помощью Azure CLI.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.custom: devx-track-azurecli
ms.openlocfilehash: 536cd01fbcf2c5d18a8c12030b709427d9bb91b1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98703612"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-the-azure-cli"></a>Создание SAS делегирования пользователя для контейнера или большого двоичного объекта с помощью Azure CLI

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

В этой статье показано, как использовать учетные данные Azure Active Directory (Azure AD) для создания SAS делегирования пользователя для контейнера или большого двоичного объекта с Azure CLI.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="install-the-latest-version-of-the-azure-cli"></a>Установите последнюю версию Azure CLI

Чтобы использовать Azure CLI для защиты SAS с помощью учетных данных Azure AD, сначала убедитесь, что установлена последняя версия Azure CLI. Дополнительные сведения об установке Azure CLI см. в разделе [установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы создать SAS для делегирования пользователей с помощью Azure CLI, убедитесь, что установлена версия 2.0.78 или более поздняя. Чтобы проверить установленную версию, используйте `az --version` команду.

## <a name="sign-in-with-azure-ad-credentials"></a>Вход с использованием учетных данных Azure AD

Войдите в Azure CLI с учетными данными Azure AD. Дополнительные сведения см. в разделе [Вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

## <a name="assign-permissions-with-azure-rbac"></a>Назначение разрешений с помощью Azure RBAC

Чтобы создать SAS для делегирования пользователей из Azure PowerShell, учетной записи Azure AD, используемой для входа в Azure CLI, должна быть назначена роль, включающая действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** . Это разрешение позволяет этой учетной записи Azure AD запрашивать *ключ делегирования пользователя*. Ключ делегирования пользователя используется для подписи SAS делегирования пользователя. Роль, предоставляющая действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** , должна быть назначена на уровне учетной записи хранения, группы ресурсов или подписки.

Если у вас нет достаточных разрешений для назначения ролей Azure субъекту безопасности Azure AD, может потребоваться попросить владельца или администратора учетной записи назначить необходимые разрешения.

В следующем примере назначается роль **участника данных BLOB-объекта хранилища** , которая включает действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** . Роль ограничивается уровнем учетной записи хранения.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli-interactive
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <email> \
    --scope "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

Дополнительные сведения о встроенных ролях, включающих действие **Microsoft. Storage/storageAccounts/блобсервицес/женератеусерделегатионкэй** , см. в статье [встроенные роли Azure](../../role-based-access-control/built-in-roles.md).

## <a name="use-azure-ad-credentials-to-secure-a-sas"></a>Использование учетных данных Azure AD для защиты SAS

При создании SAS делегирования пользователя с Azure CLI ключ делегирования пользователя, используемый для подписи SAS, создается неявно. Время начала и окончания срока действия, заданное для SAS, также используются в качестве времени начала и срока действия для ключа делегирования пользователя.

Так как максимальный интервал, по истечении которого ключ делегирования пользователя является допустимым, — 7 дней с даты начала, необходимо указать время окончания срока действия для SAS в течение 7 дней с момента запуска. После истечения срока действия ключа делегирования пользователя SAS является недействительным, поэтому срок действия SAS с временем окончания срока хранения более 7 дней будет действительным только в течение 7 дней.

При создании SAS для делегирования пользователей `--auth-mode login` требуются и `--as-user parameters` . Укажите *имя входа* для `--auth-mode` параметра, чтобы запросы, выполненные в службе хранилища Azure, были разрешены с учетными данными Azure AD. Укажите `--as-user` параметр, чтобы указать, что возвращаемые SAS должны быть подписаны на делегирование пользователя.

### <a name="create-a-user-delegation-sas-for-a-container"></a>Создание SAS для делегирования пользователя для контейнера

Чтобы создать SAS для делегирования пользователя для контейнера с Azure CLI, вызовите команду [AZ Storage Container Generate-SAS](/cli/azure/storage/container#az-storage-container-generate-sas) .

Поддерживаются следующие разрешения для сопоставления безопасности делегирования пользователей в контейнере: Добавление, создание, удаление, перечисление, чтение и запись. Разрешения можно указать отдельно или в сочетании. Дополнительные сведения об этих разрешениях см. [в разделе Создание SAS для делегирования пользователей](/rest/api/storageservices/create-user-delegation-sas).

В следующем примере возвращается маркер SAS для делегирования пользователя для контейнера. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями:

```azurecli-interactive
az storage container generate-sas \
    --account-name <storage-account> \
    --name <container> \
    --permissions acdlrw \
    --expiry <date-time> \
    --auth-mode login \
    --as-user
```

Возвращаемый токен SAS для делегирования пользователя будет выглядеть следующим образом:

```
se=2019-07-27&sp=r&sv=2018-11-09&sr=c&skoid=<skoid>&sktid=<sktid>&skt=2019-07-26T18%3A01%3A22Z&ske=2019-07-27T00%3A00%3A00Z&sks=b&skv=2018-11-09&sig=<signature>
```

### <a name="create-a-user-delegation-sas-for-a-blob"></a>Создание SAS делегирования пользователя для большого двоичного объекта

Чтобы создать SAS делегирования пользователя для большого двоичного объекта с Azure CLI, вызовите команду [AZ Storage BLOB Generate-SAS](/cli/azure/storage/blob#az-storage-blob-generate-sas) .

Поддерживаются следующие разрешения для сопоставления безопасности делегирования пользователей в большом двоичном объекте: Добавление, создание, удаление, чтение и запись. Разрешения можно указать отдельно или в сочетании. Дополнительные сведения об этих разрешениях см. [в разделе Создание SAS для делегирования пользователей](/rest/api/storageservices/create-user-delegation-sas).

Следующий синтаксис возвращает SAS делегирования пользователя для большого двоичного объекта. В примере указывается `--full-uri` параметр, который ВОЗВРАЩАЕТ URI большого двоичного объекта с добавленным маркером SAS. Не забудьте заменить значения заполнителей в квадратных скобках собственными значениями:

```azurecli-interactive
az storage blob generate-sas \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob> \
    --permissions acdrw \
    --expiry <date-time> \
    --auth-mode login \
    --as-user \
    --full-uri
```

Возвращаемый URI SAS для делегирования пользователя будет выглядеть примерно так:

```
https://storagesamples.blob.core.windows.net/sample-container/blob1.txt?se=2019-08-03&sp=rw&sv=2018-11-09&sr=b&skoid=<skoid>&sktid=<sktid>&skt=2019-08-02T2
2%3A32%3A01Z&ske=2019-08-03T00%3A00%3A00Z&sks=b&skv=2018-11-09&sig=<signature>
```

> [!NOTE]
> SAS пользователя, поддерживающий делегирование, не поддерживает определение разрешений с помощью хранимой политики доступа.

## <a name="revoke-a-user-delegation-sas"></a>Отзыв SAS для делегирования пользователя

Чтобы отозвать SAS для делегирования пользователя из Azure CLI, вызовите команду [AZ Storage Account REVOKE-делегирование Keys](/cli/azure/storage/account#az-storage-account-revoke-delegation-keys) . Эта команда отменяет все ключи делегирования пользователя, связанные с указанной учетной записью хранения. Все подписанные URL, связанные с этими ключами, становятся недействительными.

Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

```azurecli-interactive
az storage account revoke-delegation-keys \
    --name <storage-account> \
    --resource-group <resource-group>
```

> [!IMPORTANT]
> Как ключ делегирования пользователя, так и назначение ролей Azure кэшируются службой хранилища Azure, поэтому при инициации процесса отзыва может возникнуть задержка, и если существующее сопоставление безопасности делегирования пользователя станет недействительным.

## <a name="next-steps"></a>Дальнейшие действия

- [Создание SAS для делегирования пользователей (REST API)](/rest/api/storageservices/create-user-delegation-sas)
- [Операция получения ключа делегирования пользователя](/rest/api/storageservices/get-user-delegation-key)
