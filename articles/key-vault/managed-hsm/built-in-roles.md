---
title: Встроенные роли локальной системы RBAC управляемого устройства HSM — Azure Key Vault | Документация Майкрософт
description: Обзор встроенных ролей управляемого устройства HSM, которые можно назначить пользователям, субъект-службам, группам и управляемым удостоверениям
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 01e96922d9c0c47eaf4d430e92eafcd9d0964e13
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105557230"
---
# <a name="managed-hsm-local-rbac-built-in-roles"></a>Встроенные роли локальной системы RBAC управляемого устройства HSM

Локальная система RBAC управляемого устройства HSM имеет несколько встроенных ролей. Эти роли можно назначить пользователям, субъектам-службам, группам и управляемым удостоверениям. Чтобы разрешить субъекту выполнять операции, ему необходимо назначить роль, которая предоставляет разрешение на выполнение этих операций. Все эти роли и операции позволяют управлять только разрешениями на операции в плоскости данных. Чтобы управлять разрешениями уровня управления для ресурса управляемого устройства HSM, необходимо использовать [управление доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md). Примеры операций уровня управления: создание управляемого устройства HSM или обновление, перемещение, удаление существующего.

## <a name="built-in-roles"></a>Встроенные роли

|Имя роли|Описание|ID|
|---|---|---|
|Администратор управляемого устройства HSM| Предоставляет разрешения на выполнение всех операций, относящихся к домену безопасности, полному резервному копированию и восстановлению, а также управлению ролями. Выполнять какие-либо важные операции управления запрещено.|a290e904-7015-4bba-90c8-60543313cdb4|
|Специалист по системе шифрования управляемого устройства HSM|Предоставляет разрешения на выполнение всех операций по управлению ролями, очистки или восстановления удаленных ключей, а также экспорта ключей. Выполнять какие-либо другие важные операции управления запрещено.|515eb02d-2335-4d2d-92f2-b1cbdf9c3778|
|Пользователь шифрования управляемого устройства HSM|Предоставляет разрешения на выполнение всех важных операций по управлению ключами, за исключением очистки или восстановления удаленных ключей и экспорта ключей.|21dbd100-6940-42c2-9190-5d6cb909625b|
|Администратор политики управляемого устройства HSM| Предоставляет разрешение на создание и удаление назначений ролей.|4bd23610-cdcf-4971-bdee-bdc562cc28e4|
|Аудитор системы шифрования управляемого устройства HSM|Предоставляет разрешения на чтение (но не использование) атрибутов ключей.|2c18b078-7c48-4d3a-af88-5a3a1b3f82b3|
|Шифрование с помощью службы шифрования управляемого устройства HSM| Предоставляет разрешение на использование ключа для шифрования службы. |33413926-3206-4cdd-b39a-83574fe37a17|
|Резервная копия управляемого устройства HSM| Предоставляет разрешение на выполнение резервного копирования одного ключа или всего устройства HSM.|7b127d3c-77bd-4e3e-bbe0-dbb8971fa7f8|

## <a name="permitted-operations"></a>Разрешенные операции
> [!NOTE]  
> - "X" указывает, что в роли есть разрешение на выполнение действий с данными. Пустая ячейка указывает, что у роли нет разрешения на выполнение этой операции с данными.
> - Все имена действий с данными имеют префикс "Microsoft.KeyVault/managedHsm". В следующих таблицах этот префикс опускается для краткости.
> - Все имена ролей имеют префикс "Managed HSM". В следующей таблице этот префикс опускается для краткости.

|Действие с данными | Администратор | Администратор системы шифрования | Пользователь шифрования | администратор политики; | Шифрование с помощью службы шифрования | Backup | Аудитор шифрования|
|---|---|---|---|---|---|---|---|
|**Управление доменом безопасности**|
/securitydomain/download/action|<center>X</center>||||||
/securitydomain/upload/action|<center>X</center>||||||
/securitydomain/upload/read|<center>X</center>||||||
/securitydomain/transferkey/read|<center>X</center>||||||
|**Управление ключами**|
|/keys/read/action|||<center>X</center>||<center>X</center>||<center>X</center>|
|/keys/write/action|||<center>X</center>||||
|/keys/create|||<center>X</center>||||
|/keys/delete|||<center>X</center>||||
|/keys/deletedKeys/read/action||<center>X</center>|||||
|/keys/deletedKeys/recover/action||<center>X</center>|||||
|/keys/deletedKeys/delete||<center>X</center>|||||<center>X</center>|
|/keys/backup/action|||<center>X</center>|||<center>X</center>|
|/keys/restore/action|||<center>X</center>||||
|/keys/export/action||<center>X</center>|||||
|/keys/release/action|||<center>X</center>||||
|/keys/import/action|||<center>X</center>||||
|**Криптографические операции на основе ключей**|
|/keys/encrypt/action|||<center>X</center>||||
|/keys/decrypt/action|||<center>X</center>||||
|/keys/wrap/action|||<center>X</center>||<center>X</center>||
|/keys/unwrap/action|||<center>X</center>||<center>X</center>||
|/keys/sign/action|||<center>X</center>||||
|/keys/verify/action|||<center>X</center>||||
|**Управление ролями**|
|/roleAssignments/read/action|<center>X</center>|<center>X</center>|<center>X</center>|<center>X</center>|||<center>X</center>
|/roleAssignments/write/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|/roleAssignments/delete/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|/roleDefinitions/read/action|<center>X</center>|<center>X</center>|<center>X</center>|<center>X</center>|||<center>X</center>
|/roleDefinitions/write/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|/roleDefinitions/delete/action|<center>X</center>|<center>X</center>||<center>X</center>|||
|**Управление резервным копированием и восстановлением**|
|/backup/start/action|<center>X</center>|||||<center>X</center>|
|/backup/status/action|<center>X</center>|||||<center>X</center>|
|/restore/start/action|<center>X</center>||||||
|/restore/status/action|<center>X</center>||||||
||||||||

## <a name="next-steps"></a>Дальнейшие действия

- См. общие сведения об [управлении доступом на основе ролей в Azure (Azure RBAC)](../../role-based-access-control/overview.md).
- См. руководство [Управление ролями в службе "Управляемое устройство HSM"](role-management.md)
