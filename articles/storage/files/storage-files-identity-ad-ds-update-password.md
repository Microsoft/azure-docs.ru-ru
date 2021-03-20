---
title: Изменение пароля учетной записи хранения AD DS
description: Узнайте, как обновить пароль учетной записи служб домен Active Directory Services, которая представляет вашу учетную запись хранения. Это предотвращает очистку учетной записи хранения при истечении срока действия пароля, предотвращая сбои проверки подлинности.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: d16f11a85c6b370b0187975cce965bf3e1b5ba3f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85510260"
---
# <a name="update-the-password-of-your-storage-account-identity-in-ad-ds"></a>Обновите пароль для удостоверения учетной записи хранения в AD DS

Если вы зарегистрировали удостоверение или учетную запись домен Active Directory Services (AD DS), представляющую вашу учетную запись хранения в подразделении или домене, которые применяют время истечения срока действия пароля, необходимо изменить пароль до максимального срока. В организации могут выполняться автоматизированные сценарии очистки, которые удаляют учетные записи по истечении срока их действия. Поэтому, если вы не измените пароль до истечения срока его действия, ваша учетная запись может быть удалена, что приведет к потере доступа к общим файловым ресурсам Azure.

Для активации смены пароля можно выполнить `Update-AzStorageAccountADObjectPassword` команду из [модуля азфилешибрид](https://github.com/Azure-Samples/azure-files-samples/releases). Эта команда должна выполняться в локальной среде, присоединенной к AD DS, с помощью гибридного пользователя с разрешением владельца учетной записи хранения и разрешений AD DS для изменения пароля удостоверения, представляющего учетную запись хранения. Команда выполняет такие действия, как смена ключа учетной записи хранения. В частности, он получает второй ключ Kerberos учетной записи хранения и использует его для обновления пароля зарегистрированной учетной записи в AD DS. Затем он повторно создает целевой ключ Kerberos учетной записи хранения и обновляет пароль зарегистрированной учетной записи в AD DS. Эту команду необходимо выполнить в локальной среде, присоединенной к AD DS.

```PowerShell
# Update the password of the AD DS account registered for the storage account
# You may use either kerb1 or kerb2
Update-AzStorageAccountADObjectPassword `
        -RotateToKerbKey kerb2 `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -StorageAccountName "<your-storage-account-name-here>"
```
