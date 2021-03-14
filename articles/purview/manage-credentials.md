---
title: Создание учетных данных для проверок и управление ими
description: Узнайте, как создавать учетные данные в Azure зрения и управлять ими.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 02/11/2021
ms.openlocfilehash: 3802d25ebd8f21ab5b8991a66ceb6650f2f276a9
ms.sourcegitcommit: afb9e9d0b0c7e37166b9d1de6b71cd0e2fb9abf5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/14/2021
ms.locfileid: "103461714"
---
# <a name="credentials-for-source-authentication-in-azure-purview"></a>Учетные данные для исходной проверки подлинности в Azure зрения

В этой статье описывается, как можно создать учетные данные в Azure зрения. Эти сохраненные учетные данные позволяют быстро повторно использовать и применять сохраненные сведения проверки подлинности к проверкам источника данных.

## <a name="prerequisites"></a>Предварительные требования

- Хранилище ключей Azure. Сведения о том, как ее создать, см. в разделе [Краткое руководство. Создание хранилища ключей с помощью портал Azure](../key-vault/general/quick-create-portal.md).

## <a name="introduction"></a>Введение

Учетные данные — это данные проверки подлинности, которые Azure зрения может использовать для проверки подлинности в зарегистрированных источниках данных. Объект учетных данных может быть создан для различных типов сценариев проверки подлинности, например для обычной проверки подлинности, требующей имени пользователя и пароля. Для регистрации учетных данных требуются определенные сведения, необходимые для проверки подлинности, в зависимости от выбранного типа метода проверки подлинности. Учетные данные используют существующие секреты хранилища ключей Azure для получения конфиденциальных сведений о проверке подлинности в процессе создания учетных данных.

## <a name="use-purview-managed-identity-to-set-up-scans"></a>Использование управляемого удостоверения зрения для настройки проверок

Если для настройки проверок используется управляемое удостоверение зрения, вам не потребуется явно создавать учетные данные и связывать хранилище ключей с зрения для их хранения. Подробные инструкции по добавлению управляемого удостоверения зрения для получения доступа к просмотру источников данных см. в следующих разделах проверки подлинности для конкретного источника данных:

- [Хранилище BLOB-объектов Azure](register-scan-azure-blob-storage-source.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage 1-го поколения](register-scan-adls-gen1.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage 2-го поколения](register-scan-adls-gen2.md#setting-up-authentication-for-a-scan)
- [База данных SQL Azure](register-scan-azure-sql-database.md)
- [Управляемый экземпляр Базы данных SQL Azure](register-scan-azure-sql-database-managed-instance.md#setting-up-authentication-for-a-scan)
- [Azure Synapse Analytics](register-scan-azure-synapse-analytics.md#setting-up-authentication-for-a-scan)

## <a name="create-azure-key-vaults-connections-in-your-azure-purview-account"></a>Создание подключений Azure Key Vault в учетной записи Azure зрения

Прежде чем можно будет создать учетные данные, сначала Свяжите один или несколько существующих экземпляров Azure Key Vault с учетной записью Azure зрения.

1. В [портал Azure](https://portal.azure.com)выберите учетную запись Azure зрения и откройте Azure зрения Studio. Перейдите в **центр управления** в Azure зрения Studio и перейдите к **учетным данным**.

2. На странице **учетные данные** выберите **управление подключениями Key Vault**.

   :::image type="content" source="media/manage-credentials/manage-kv-connections.png" alt-text="Управление подключениями Azure Key Vault":::

3. На странице Управление подключениями Key Vault выберите **+ создать** .

4. Укажите необходимые сведения, а затем щелкните **создать**.

5. Убедитесь, что ваша Key Vault успешно связана с вашей учетной записью Azure зрения, как показано в следующем примере:

   :::image type="content" source="media/manage-credentials/view-kv-connections.png" alt-text="Просмотрите Azure Key Vault подключения для подтверждения.":::

## <a name="grant-the-purview-managed-identity-access-to-your-azure-key-vault"></a>Предоставьте управляемому удостоверению зрения доступ к вашей Azure Key Vault

1. Перейдите к Azure Key Vault.

2. Перейдите на страницу **политики доступа** .

3. Нажмите **Добавить политику доступа**.

   :::image type="content" source="media/manage-credentials/add-msi-to-akv.png" alt-text="Добавление зрения MSI в AKV":::

4. В раскрывающемся списке **разрешения секреты** выберите **получить** и **вывести список** разрешений.

5. В поле **Выбор субъекта** выберите управляемое удостоверение зрения. Можно выполнить поиск MSI зрения с помощью имени экземпляра зрения **или** идентификатора приложения управляемого удостоверения. В настоящее время не поддерживаются составные удостоверения (имя управляемого удостоверения + идентификатор приложения).

   :::image type="content" source="media/manage-credentials/add-access-policy.png" alt-text="Добавление политики доступа":::

6. Щелкните **Добавить**.

7. Нажмите кнопку **сохранить** , чтобы сохранить политику доступа.

   :::image type="content" source="media/manage-credentials/save-access-policy.png" alt-text="Сохранить политику доступа":::

## <a name="create-a-new-credential"></a>Создание новых учетных данных

Эти типы учетных данных поддерживаются в зрения:

- Обычная проверка подлинности. Вы добавляете **пароль** в качестве секрета в хранилище ключей.
- Субъект-служба. Вы добавляете **ключ субъекта-службы** в качестве секрета в хранилище ключей.
- Проверка подлинности SQL. Вы добавляете **пароль** в качестве секрета в хранилище ключей.
- Ключ учетной записи. Вы добавляете **ключ учетной записи** в качестве секрета в хранилище ключей.
- Роль ARN. для источника данных Amazon S3 добавьте свою **роль ARN** в AWS. 

Дополнительные сведения см. [в разделе Добавление секрета в Key Vault](../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault) и [Создание новой роли AWS для зрения](register-scan-amazon-s3.md#create-a-new-aws-role-for-purview).

После сохранения секретов в хранилище ключей:

1. В Azure зрения перейдите на страницу учетные данные.

2. Создайте новые учетные данные, выбрав **+ создать**.

3. Укажите необходимые сведения. Выберите **метод проверки подлинности** и **Key Vault подключение** , из которого следует выбрать секрет.

4. После заполнения всех сведений выберите **создать**.

   :::image type="content" source="media/manage-credentials/new-credential.png" alt-text="Новые учетные данные":::

5. Убедитесь, что новые учетные данные отображаются в представлении списка и готовы к использованию.

   :::image type="content" source="media/manage-credentials/view-credentials.png" alt-text="Просмотр учетных данных":::

## <a name="manage-your-key-vault-connections"></a>Управление подключениями к хранилищу ключей

1. Поиск и поиск Key Vault подключений по имени

   :::image type="content" source="media/manage-credentials/search-kv.png" alt-text="Поиск хранилища ключей":::

2. Удаление одного или нескольких подключений Key Vault

   :::image type="content" source="media/manage-credentials/delete-kv.png" alt-text="Удаление хранилища ключей":::

## <a name="manage-your-credentials"></a>Управление учетными данными

1. Поиск и поиск учетных данных по имени.
  
2. Выберите и обновите существующие учетные данные.

3. Удалите одну или несколько учетных данных.

## <a name="next-steps"></a>Дальнейшие действия

[Создание набора правил проверки](create-a-scan-rule-set.md)
