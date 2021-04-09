---
title: Полное резервное копирование, а также полное и выборочное восстановление для Управляемого устройства HSM в Azure
description: Этот документ описывает процессы полного резервного копирования, полного и выборочного восстановления
services: key-vault
author: amitbapat
tags: azure-key-vault
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: e926dcd4b05d137c7927bdfe5221923d25d4670c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100093494"
---
# <a name="full-backup-and-restore"></a>Полное резервное копирование и восстановление

> [!NOTE]
> Эта возможность доступна только для управляемых устройств HSM в качестве типа ресурса.

Управляемое устройство HSM позволяет создавать полную резервную копию всего содержимого HSM, включая все ключи, версии, атрибуты, теги и назначения ролей. Эта резервная копия шифруется с помощью криптографических ключей, связанных с доменом безопасности HSM.

Резервное копирование выполняется в плоскости данных. Сторона, инициирующая операцию резервного копирования, должна иметь разрешение на действие dataAction **Microsoft.KeyVault/managedHsm/backup/start/action**.

Только следующие встроенные роли имеют разрешение на выполнение полного резервного копирования:
- администратор Управляемого устройства HSM;
- резервное копирование Управляемого устройства HSM.

Чтобы выполнить полное резервное копирование, необходимо предоставить следующие сведения:
- имя или URL-адрес HSM;
- Имя учетной записи хранения
- контейнер хранения больших двоичных объектов в учетной записи хранения;
- маркер SAS для контейнера хранилища с разрешениями `crdw`.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="full-backup"></a>Полное резервное копирование

Хотя резервное копирование — это длительная операция, при ее запуске немедленно возвращается идентификатор задания. По этому идентификатору задания вы можете проверить состояние процесса резервного копирования. Процесс резервного копирования создает в указанном контейнере папку с именем, созданным по шаблону **`mhsm-{HSM_NAME}-{YYYY}{MM}{DD}{HH}{mm}{SS}`** , где HSM_NAME обозначает это имя управляемого устройства HSM, а YYYY, MM, DD, HH, MM, mm, SS — соответственно год, месяц, день, час, минуту и секунду получения команды резервного копирования в формате UTC.

Пока выполняется резервное копирование, HSM не сможет обеспечивать полную пропускную способность, так как некоторые секции устройства заняты выполнением операции резервного копирования.

```azurecli-interactive
# time for 500 minutes later for SAS token expiry

end=$(date -u -d "500 minutes" '+%Y-%m-%dT%H:%MZ')

# Get storage account key

skey=$(az storage account keys list --query '[0].value' -o tsv --account-name mhsmdemobackup --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)

# Create a container

az storage container create --account-name  mhsmdemobackup --name mhsmdemobackupcontainer  --account-key $skey

# Generate a container sas token

sas=$(az storage container generate-sas -n mhsmdemobackupcontainer --account-name mhsmdemobackup --permissions crdw --expiry $end --account-key $skey -o tsv --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)

# Backup HSM

az keyvault backup start --hsm-name mhsmdemo2 --storage-account-name mhsmdemobackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas --subscription 361da5d4-a47a-4c79-afdd-d66f684f4070
```

## <a name="full-restore"></a>Полное восстановление

Полное восстановление позволяет полностью восстановить содержимое HSM из ранее созданной резервной копии, включая все ключи, версии, атрибуты, теги и назначения ролей. Все данные, которые до этого хранились в HSM, будут полностью очищены, и устройство вернется к тому состоянию, в котором находилось на момент создания этой резервной копии.

> [!IMPORTANT]
> Полное восстановление — это достаточно разрушительный процесс. Поэтому вам нужно обязательно создать полную резервную копию не более чем за 30 минут до начала операции `restore`.

Восстановление выполняется в плоскости данных. Сторона, инициирующая операцию восстановления, должна иметь разрешение на действие dataAction **Microsoft.KeyVault/managedHsm/restore/start/action**. Исходное устройство HSM, из которого создавалась резервная копия, и целевое устройство HSM, на которое выполняется восстановление, **обязательно** должны иметь одинаковый домен безопасности. Сведения о домене безопасности для Управляемого устройства HSM см. [здесь](security-domain.md).

Чтобы выполнить полное восстановление, необходимо предоставить следующие сведения:
- имя или URL-адрес HSM;
- Имя учетной записи хранения
- контейнер больших двоичных объектов в учетной записи хранения;
- маркер SAS для контейнера хранилища с разрешениями `rl`;
- имя папки в контейнере хранилища, где хранится исходная резервная копия.

Хотя восстановление — это длительная операция, при ее запуске немедленно возвращается идентификатор задания. По этому идентификатору задания вы можете проверить состояние процесса восстановления. Пока идет процесс восстановления, HSM переходит в особый режим восстановления и не принимает команды плоскости данных (за исключением проверки состояния восстановления).

```azurecli-interactive
#### time for 500 minutes later for SAS token expiry

end=$(date -u -d "500 minutes" '+%Y-%m-%dT%H:%MZ')

# Get storage account key

skey=$(az storage account keys list --query '[0].value' -o tsv --account-name mhsmdemobackup --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)

# Generate a container sas token

sas=$(az storage container generate-sas -n mhsmdemobackupcontainer --account-name mhsmdemobackup --permissions rl --expiry $end --account-key $skey -o tsv --subscription a1ba9aaa-b7f6-4a33-b038-6e64553a6c7b)
```

## <a name="restore-hsm"></a>Восстановление HSM

```
az keyvault restore start --hsm-name mhsmdemo2 --storage-account-name mhsmdemobackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas --backup-folder mhsm-mhsmdemo-2020083120161860
```

## <a name="next-steps"></a>Next Steps
- См. статью [Управление Управляемым устройством HSM с помощью Azure CLI](key-management.md).
- Узнайте больше о [домене безопасности для Управляемого устройства HSM](security-domain.md).
