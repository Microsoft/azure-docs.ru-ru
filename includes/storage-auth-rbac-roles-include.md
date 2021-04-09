---
title: включить файл
description: включить файл
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/06/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: f014ce55dc40723faf1b60f908814f9fa0428b8e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99213670"
---
Azure предоставляет приведенные ниже встроенные роли Azure для авторизации доступа к данным больших двоичных объектов и очередей с использованием Azure AD и OAuth.

- [Владелец данных BLOB-объектов хранилища](../articles/role-based-access-control/built-in-roles.md#storage-blob-data-owner). Предоставляет права владельца и позволяет управлять доступом POSIX для Azure Data Lake Storage 2-го поколения. Дополнительные сведения см. в статье [Контроль доступа в Azure Data Lake Storage 2-го поколения](../articles/storage/blobs/data-lake-storage-access-control.md).
- [Участник для данных BLOB-объектов хранилища](../articles/role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Предоставляет разрешения (чтение, запись и удаление) к ресурсам хранилища BLOB-объектов.
- [Модуль чтения данных BLOB-объектов хранилища](../articles/role-based-access-control/built-in-roles.md#storage-blob-data-reader). Предоставляет разрешения только для чтения к ресурсам хранилища BLOB-объектов.
- [Представитель BLOB-объектов хранилища](../articles/role-based-access-control/built-in-roles.md#storage-blob-delegator). Получение ключа делегирования пользователя, который можно использовать для создания подписи общего доступа, которую ставят при помощи учетных данных Azure AD для контейнера или большого двоичного объекта.
- [Участник для данных очереди хранилища](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-contributor). Предоставляет разрешения (чтение, запись и удаление) к очередям Azure.
- [Модуль чтения данных очереди хранилища](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-reader). Предоставляет разрешения только для чтения к очередям Azure.
- [Обработчик сообщений данных в очереди хранилища](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-message-processor). Предоставляет разрешения (просмотр, получение и удаление) к сообщениям в очередях службы хранилища Azure.
- [Отправитель сообщений данных в очередь хранилища](../articles/role-based-access-control/built-in-roles.md#storage-queue-data-message-sender). Предоставляет разрешения на добавление к сообщениям в очередях службы хранилища Azure.

Только роли, явно определенные для доступа к данным, позволяют субъекту безопасности получать доступ к данным большого двоичного объекта или очереди. Встроенные роли, такие как **Владелец**, **Участник** и **Участник учетных записей хранения**, позволяют субъекту безопасности управлять учетной записью хранения, но не предоставляют доступ к данным большого двоичного объекта или очереди в этой учетной записи через Azure AD. Однако если роль включает **Microsoft.Storage/storageAccounts/listKeys/action**, то пользователь, которому назначена эта роль, может получить доступ к данным в учетной записи хранения через авторизацию с общим ключом с использованием ключей доступа к учетной записи. Дополнительные сведения см. в статье [Использование портала Azure для доступа к данным большого двоичного объекта или очереди](../articles/storage/blobs/authorize-data-operations-portal.md).

Подробные сведения о встроенных ролях Azure для службы хранилища Azure, как для служб данных, так и для службы управления, см. в разделе **Хранилище** статьи [Встроенные роли Azure](../articles/role-based-access-control/built-in-roles.md#storage). Дополнительные сведения о различных типах ролей, которые предоставляют разрешения в Azure, см. в статье [Роли классического администратора подписки, роли Azure и роли Azure AD](../articles/role-based-access-control/rbac-and-directory-admin-roles.md).

> [!IMPORTANT]
> Назначение ролей Azure может занимать до 30 минут.
