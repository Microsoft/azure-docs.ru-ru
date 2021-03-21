---
title: Устранение неполадок подключений в Azure зрения
description: В этой статье описаны действия по устранению неполадок подключений в Azure зрения.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/23/2020
ms.openlocfilehash: c176fcafe13749ba89c04b34854f036aa5aea516
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101677650"
---
# <a name="troubleshoot-your-connections-in-azure-purview"></a>Устранение неполадок подключений в Azure зрения

В этой статье описывается, как устранять ошибки подключения при настройке проверок в источниках данных в Azure зрения.

## <a name="permission-the-credential-on-the-data-source"></a>Разрешение учетных данных в источнике данных

Если вы используете управляемое удостоверение или субъект-службу в качестве метода проверки подлинности для проверок, необходимо разрешить этим удостоверениям доступ к источнику данных.

Существуют специальные инструкции для каждого типа источника:

- [Хранилище BLOB-объектов Azure](register-scan-azure-blob-storage-source.md#setting-up-authentication-for-a-scan)
- [Azure Cosmos DB](register-scan-azure-cosmos-database.md#setting-up-authentication-for-a-scan)
- [Azure Data Explorer](register-scan-azure-data-explorer.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage 1-го поколения](register-scan-adls-gen1.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage 2-го поколения](register-scan-adls-gen2.md#setting-up-authentication-for-a-scan)
- [База данных SQL Azure](register-scan-azure-sql-database.md)
- [Управляемый экземпляр Базы данных SQL Azure](register-scan-azure-sql-database-managed-instance.md#setting-up-authentication-for-a-scan)
- [Azure Synapse Analytics](register-scan-azure-synapse-analytics.md#setting-up-authentication-for-a-scan)
- [SQL Server](register-scan-on-premises-sql-server.md#setting-up-authentication-for-a-scan)
- [Power BI](register-scan-power-bi-tenant.md)
- [Amazon S3](register-scan-amazon-s3.md#create-a-purview-credential-for-your-aws-bucket-scan)
## <a name="storing-your-credential-in-your-key-vault-and-using-the-right-secret-name-and-version"></a>Хранение учетных данных в хранилище ключей и использование правильного имени и версии секрета

Необходимо также сохранить учетные данные в экземпляре Azure Key Vault и использовать правильные имя и версию секрета.

Проверьте это, выполнив следующие действия:

1. Перейдите к Key Vault.
1. Выберите **Параметры** > **Секреты**.
1. Выберите секрет, который вы используете для проверки подлинности в источнике данных для проверок.
1. Выберите версию, которую предполагается использовать, и убедитесь, что пароль или ключ учетной записи указаны правильно, щелкнув **Показывать значение секрета**. 

## <a name="verify-permissions-for-the-purview-managed-identity-on-your-azure-key-vault"></a>Проверьте разрешения для управляемого удостоверения зрения на вашем Azure Key Vault

Убедитесь, что для доступа к Azure Key Vaultу настроены правильные разрешения для управляемого удостоверения зрения.

Чтобы проверить это, выполните следующие действия.

1. Перейдите к хранилищу ключей и к разделу **политики доступа** .

1. Убедитесь, что управляемое удостоверение зрения отображается в разделе *текущие политики доступа* по крайней мере с разрешениями **Get** и **List** для секретов.

   :::image type="content" source="./media/troubleshoot-connections/verify-minimum-permissions.png" alt-text="Изображение, показывающее выбор из списка параметров разрешений Get и List":::

Если управляемое удостоверение зрения отсутствует в списке, выполните действия, описанные в разделе [Создание учетных данных для сканирования и управление ими](manage-credentials.md) для добавления. 

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор каталога данных Azure Purview](how-to-browse-catalog.md)
- [Поиск по каталогу данных Azure Purview](how-to-search-catalog.md)
