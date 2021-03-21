---
title: Шифрование данных — портал Azure — база данных Azure для MySQL
description: Узнайте, как настроить шифрование данных для базы данных Azure для MySQL и управлять им с помощью портал Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 01/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 00670746c1686bca354adc989ddce6c9dd336491
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96519065"
---
# <a name="data-encryption-for-azure-database-for-mysql-by-using-the-azure-portal"></a>Шифрование данных для базы данных Azure для MySQL с помощью портал Azure

Узнайте, как использовать портал Azure для настройки и управления шифрованием данных для базы данных Azure для MySQL.

## <a name="prerequisites-for-azure-cli"></a>Необходимые условия для Azure CLI

* Подписка Azure и права администратора для нее.
* В Azure Key Vault создайте хранилище ключей и ключ, который будет использоваться для ключа, управляемого клиентом.
* Для использования в качестве ключа, управляемого клиентом, хранилище ключей должно иметь следующие свойства:
  * [Обратимое удаление](../key-vault/general/soft-delete-overview.md)

    ```azurecli-interactive
    az resource update --id $(az keyvault show --name \ <key_vault_name> -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
    ```

  * [Очистить защищенные](../key-vault/general/soft-delete-overview.md#purge-protection)

    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
    ```
  * Дни хранения заданы как 90 дней
  
    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --retention-days 90
    ```

* Чтобы использовать ключ в качестве управляемого клиентом, он должен иметь следующие атрибуты:
  * без даты окончания срока действия;
  * не отключено;
  * Выполнение операций **получения**, **переноса** и **распаковки**
  * для атрибута рековерилевел задано значение " **восстанавливаемый** " (для этого требуется включить обратимое удаление с периодом хранения, равным 90 дням)
  * Защита от очистки включена

Проверить указанные выше атрибуты ключа можно с помощью следующей команды:

```azurecli-interactive
az keyvault key show --vault-name <key_vault_name> -n <key_name>
```

## <a name="set-the-right-permissions-for-key-operations"></a>Задайте правильные разрешения для операций с ключами.

1. В Key Vault выберите **политики доступа**  >  **Добавить политику доступа**.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/show-access-policy-overview.png" alt-text="Снимок экрана Key Vault с выделенными политиками доступа и добавлением политики доступа":::

2. Выберите **ключевые разрешения** и выберите **получить**, **обернуть**, **распереносить**, а **основной**— имя сервера MySQL. Если участник сервера не найден в списке существующих участников, его необходимо зарегистрировать. Вам будет предложено зарегистрировать сервер-участник при первой попытке настроить шифрование данных и выполнить его не удастся.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/access-policy-wrap-unwrap.png" alt-text="Общие сведения о политике доступа":::

3. Щелкните **Сохранить**.

## <a name="set-data-encryption-for-azure-database-for-mysql"></a>Настройка шифрования данных для базы данных Azure для MySQL

1. В базе данных Azure для MySQL выберите **Шифрование данных** , чтобы настроить ключ, управляемый клиентом.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/data-encryption-overview.png" alt-text="Снимок экрана базы данных Azure для MySQL с выделенным шифрованием данных":::

2. Можно выбрать хранилище ключей и пару ключей или ввести идентификатор ключа.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/setting-data-encryption.png" alt-text="Снимок экрана базы данных Azure для MySQL с выделенными параметрами шифрования данных":::

3. Щелкните **Сохранить**.

4. Чтобы убедиться, что все файлы (включая временные файлы) полностью зашифрованы, перезапустите сервер.

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>Использование шифрования данных для серверов восстановления или реплик

После шифрования Базы данных Azure для MySQL с помощью управляемого ключа клиента, хранящегося в Key Vault, все создаваемые копии сервера также шифруются. Эту новую копию можно создать с помощью локальной или географической операции либо с помощью операции реплики (локальной или кросс-региона). Для зашифрованного сервера MySQL можно выполнить следующие действия, чтобы создать зашифрованный восстановленный сервер.

1. На сервере выберите **Обзор**  >  **восстановить**.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/show-restore.png" alt-text="Снимок экрана базы данных Azure для MySQL с выделенным обзором и восстановлением":::

   Или для сервера с поддержкой репликации в разделе **Параметры** выберите **репликация**.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/mysql-replica.png" alt-text="Снимок экрана базы данных Azure для MySQL с выделенной репликацией":::

2. После завершения операции восстановления новый созданный сервер шифруется с помощью ключа основного сервера. Однако функции и параметры на сервере отключены, и сервер недоступен. Это предотвращает обработку любых данных, так как удостоверению нового сервера еще не было предоставлено разрешение на доступ к хранилищу ключей.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/show-restore-data-encryption.png" alt-text="Снимок экрана базы данных Azure для MySQL с выделенным состоянием &quot;недоступно&quot;":::

3. Чтобы сделать сервер доступным, повторно проверьте ключ на восстановленном сервере. Выберите ключ повторной проверки **шифрования данных**  >  .

   > [!NOTE]
   > Первая попытка повторной проверки завершится ошибкой, так как субъекту-службе нового сервера необходимо предоставить доступ к хранилищу ключей. Чтобы создать субъект-службу, выберите повторно **проверить ключ**, в котором будет отображаться сообщение об ошибке, но создается субъект-служба. После этого ознакомьтесь с [этими действиями](#set-the-right-permissions-for-key-operations) ранее в этой статье.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/show-revalidate-data-encryption.png" alt-text="Снимок экрана базы данных Azure для MySQL с выделенным шагом повторной проверки":::

   Необходимо предоставить хранилищу ключей доступ к новому серверу.

4. После регистрации субъекта-службы повторно проверьте ключ, и сервер возобновит нормальную работу.

   :::image type="content" source="media/concepts-data-access-and-security-data-encryption/restore-successful.png" alt-text="Снимок экрана базы данных Azure для MySQL с восстановленными функциями":::

## <a name="next-steps"></a>Дальнейшие действия

 Дополнительные сведения о шифровании данных см. в статье [Шифрование данных в базе данных Azure для MySQL с помощью ключа, управляемого клиентом](concepts-data-encryption-mysql.md).
