---
title: Управление ключами в управляемом устройстве HSM (Azure Key Vault) | Документация Майкрософт
description: Эта статья поможет вам в управлении ключами, сохраненными в управляемом устройстве HSM.
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 8d0cbd35b53bc8460ac8a19e5197d1f560657263
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102212048"
---
# <a name="manage-a-managed-hsm-using-the-azure-cli"></a>Управление Управляемым устройством HSM с помощью Azure CLI

> [!NOTE]
> Key Vault поддерживает два типа ресурсов: хранилища и управляемые устройства HSM. Эта статья посвящена **Управляемому устройству HSM**. Если вы хотите узнать, как управлять хранилищем, см. статью [Управление Key Vault с помощью Azure CLI](../general/manage-with-cli2.md).

Общие сведения об Управляемом устройстве HSM см. в статье [Что собой представляет служба "Управляемое устройство HSM"?](overview.md)

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, вам потребуется следующее:

* подписка на Microsoft Azure. Если у вас ее нет, зарегистрируйтесь, чтобы воспользоваться [бесплатной пробной версией](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI]( /cli/azure/install-azure-cli).
* Управляемое устройство HSM в подписке. См. [Краткое руководство. Подготовка и активация управляемого устройства HSM с помощью Azure CLI](quick-create-cli.md) для выполнения соответствующих действий.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы войти в Azure с помощью CLI, введите следующее:

```azurecli
az login
```

Дополнительные сведения о вариантах входа с помощью Azure CLI см. в [этой статье](/cli/azure/authenticate-azure-cli).

> [!NOTE]
> В каждой из представленных ниже команд демонстрируются два метода использования. В одном из них передаются параметры **--hsm-name** и **--name** (имя ключа), а в другом применяется параметр **--id**, в котором можно указать полный URL-адрес с именем ключа (если оно используется). Второй метод полезен в тех случаях, когда вызывающая сторона (пользователь или приложение) не имеет доступа на чтение к плоскости управления и имеет только ограниченный доступ к плоскости данных.

## <a name="create-an-hsm-key"></a>Создание ключа HSM

Для создания ключа используйте команду `az keyvault key create`.

### <a name="create-an-rsa-key"></a>Создание ключа RSA

В следующем примере показано, как создать 3072-битный ключ **RSA**, который будет использоваться только для операций **wrapKey и unwrapKey** (задаются параметром --ops). 


```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072

## OR
# Note the key name (myrsakey) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072
```

Обратите внимание, что операция `get` возвращает только открытый ключ и атрибуты ключа. Она не возвращает закрытый ключ (для асимметричного ключа) или материал ключа (для симметричного ключа).

### <a name="create-an-ec-key"></a>Создание ключа EC

В приведенном ниже примере показано, как создать ключ **EC** с кривой P-256, который будет использоваться только для операций **sign и verify** (задаются параметром --ops) и имеет два тега: **usage** и **appname**. Эти теги помогают добавлять в ключ дополнительные метаданные для отслеживания и управления.

```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myec256key --ops sign verify  --tags ‘usage=signing] appname=myapp’ --kty EC-HSM --curve P-256

## OR
# Note the key name (myec256key) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myec256key --ops sign verify  --tags ‘usage=signing] appname=myapp’ --kty EC-HSM --curve P-256
```

### <a name="create-a-256-bit-symmetric-key"></a>Создание 256-битного симметричного ключа

В следующем примере показано, как создать 256-битный **симметричный** ключ, который будет использоваться только для операций **encrypt и decrypt** (задаются параметром --ops).

```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myaeskey --ops encrypt decrypt  --tags --kty oct-HSM --size 256

## OR
# Note the key name (myaeskey) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myaeskey --ops encrypt decrypt  --tags ‘usage=signing] appname=myapp’ --kty oct-HSM --size 256
```

## <a name="view-key-attributes-and-tags"></a>Просмотр атрибутов и тегов ключа

Используйте команду `az keyvault key show`, чтобы просмотреть атрибуты, версии и теги для ключа.

```azurecli-interactive
az keyvault key show --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key show --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey

```

## <a name="list-keys"></a>получение списка ключей;

Команда `az keyvault key list` позволяет вывести список всех ключей в управляемом устройстве HSM.

```azurecli-interactive
az keyvault key list --hsm-name ContosoHSM

## OR
# use full URI

az keyvault key list --id https://ContosoMHSM.managedhsm.azure.net/

```

## <a name="delete-a-key"></a>Удаление ключа

Чтобы удалить ключ из управляемого устройства HSM, используйте команду `az keyvault key delete`. Обратите внимание, что обратимое удаление всегда включено. Это означает, что удаленный ключ будет сохраняться в удаленном состоянии и может быть восстановлен, пока не пройдет заданное количество дней хранения. После этого ключ окончательно удаляется без возможности восстановления.

```azurecli-interactive
az keyvault key delete --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key delete --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey
```

## <a name="list-deleted-keys"></a>Вывод списка удаленных ключей

Чтобы вывести список всех ключей в удаленном состоянии в управляемом устройстве HSM, используйте команду `az keyvault key list-deleted`.

```azurecli-interactive
az keyvault key list-deleted --hsm-name ContosoHSM

## OR
# use full URI

az keyvault key list-deleted --id https://ContosoMHSM.managedhsm.azure.net/
```

## <a name="recover-undelete-a-deleted-key"></a>Восстановление (отмена удаления) удаленного ключа

Чтобы вывести список всех ключей в удаленном состоянии в управляемом устройстве HSM, используйте команду `az keyvault key list-deleted`. Если вам нужно восстановить ключ по значению параметра --id, следует записать для удаленного ключа значение `recoveryId`, которое можно получить из команды `az keyvault key list-deleted`.

```azurecli-interactive
az keyvault key recover --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey
```

## <a name="purge-permanently-delete-a-key"></a>Окончательное удаление ключа.

Чтобы очистить (окончательно удалить) ключ, используйте команду `az keyvault key purge`.

> [!NOTE]
> Если для управляемого устройства HSM включена защита, операция очистки будет заблокирована. Такой ключ будет автоматически очищен по истечении периода хранения.

```azurecli-interactive
az keyvault key purge --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey

```

## <a name="create-a-single-key-backup"></a>Создание резервной копии одного ключа

Чтобы создать резервную копию ключа, используйте команду `az keyvault key backup`. Файл резервной копии — это зашифрованный большой двоичный объект, криптографически привязанный к домену безопасности исходного устройства HSM. Его можно восстановить только на тех устройствах HSM, которые подключены к тому же домену безопасности. Узнайте больше о [доменах безопасности](security-domain.md).

```azurecli-interactive
az keyvault key backup --hsm-name ContosoHSM --name myrsakey --file myrsakey.backup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key backup --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey  --file myrsakey.backup

```

## <a name="restore-a-single-key-from-backup"></a>Восстановление одного ключа из резервной копии

Чтобы восстановить один ключ, используйте команду `az keyvault key restore`. Исходное устройство HSM, в котором была создана резервная копия, должно быть подключено к тому же домену безопасности, что и целевое устройство HSM, в котором восстанавливается ключ.

> [!NOTE]
> Восстановление не сможет завершиться успешно, если уже существует ключ с таким же именем в активном или удаленном состоянии.

```azurecli-interactive
az keyvault key restore --hsm-name ContosoHSM --name myrsakey --file myrsakey.bakup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey --file myrsakey.bakup

```

## <a name="import-a-key-from-a-file"></a>Импорт ключа из файла

Чтобы импортировать ключ (поддерживаются только ключи RSA и EC) из файла, используйте команду `az keyvault key import`. Файл сертификата должен иметь закрытый ключ и использовать кодировку PEM (как определено в документах RFC [1421](https://tools.ietf.org/html/rfc1421), [1422](https://tools.ietf.org/html/rfc1422), [1423](https://tools.ietf.org/html/rfc1423), [1424](https://tools.ietf.org/html/rfc1424)).

```azurecli-interactive
az keyvault key import --hsm-name ContosoHSM --name myrsakey --pem-file mycert.key --password 'mypassword'

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey --pem-file mycert.key --password 'mypassword'
```

Сведения об импорте ключа из локального устройства HSM в управляемое устройство HSM см. в статье [Импорт защищенных модулем HSM ключей в службу "Управляемое устройство HSM" (BYOK)](hsm-protected-keys-byok.md).

## <a name="next-steps"></a>Дальнейшие действия

- Полное описание команд Azure CLI для хранилища ключей доступно в [справочнике по интерфейсу командной строки Key Vault](/cli/azure/keyvault).
- Справочные материалы по программированию см. в статье [Руководство разработчика хранилища ключей Azure](../general/developers-guide.md).
- Подробнее об [управлении ролями для Управляемого устройства HSM](role-management.md)
- Подробнее о [рекомендациях по использованию Управляемого устройства HSM](best-practices.md)
