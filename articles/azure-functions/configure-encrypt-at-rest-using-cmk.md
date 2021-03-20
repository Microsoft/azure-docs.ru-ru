---
title: Шифрование неактивных источников приложения
description: Зашифруйте данные приложения в службе хранилища Azure и развертывайте их как файл пакета.
ms.topic: article
ms.date: 03/06/2020
ms.openlocfilehash: 6e4cf16118e748e3ee1d1ff4899730463565ec70
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92108037"
---
# <a name="encrypt-your-application-data-at-rest-using-customer-managed-keys"></a>Шифрование неактивных данных приложения с помощью управляемых клиентом ключей

Для шифрования неактивных данных приложения функции требуется учетная запись хранения Azure и Azure Key Vault. Эти службы используются при запуске приложения из пакета развертывания.

  - Служба [хранилища Azure обеспечивает шифрование неактивных данных](../storage/common/storage-service-encryption.md). Вы можете использовать предоставленные системой ключи или собственные ключи, управляемые клиентом. Именно здесь данные приложения хранятся, если они не выполняются в приложении-функции в Azure.
  - [Запуск из пакета развертывания](run-functions-from-deployment-package.md) — это функция развертывания службы приложений. Она позволяет развертывать содержимое сайта из учетной записи хранения Azure, используя URL-адрес подписанного общего доступа (SAS).
  - [Key Vault ссылки](../app-service/app-service-key-vault-references.md) — это функция безопасности службы приложений. Он позволяет импортировать секреты во время выполнения в качестве параметров приложения. Используйте этот параметр, чтобы зашифровать URL-адрес SAS учетной записи хранения Azure.

## <a name="set-up-encryption-at-rest"></a>Настройка шифрования неактивных данных

### <a name="create-an-azure-storage-account"></a>Создание учетной записи хранения Azure

Сначала [Создайте учетную запись хранения Azure](../storage/common/storage-account-create.md) и [зашифруйте ее с помощью управляемых клиентом ключей](../storage/common/customer-managed-keys-overview.md). После создания учетной записи хранения используйте [Обозреватель службы хранилища Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) для отправки файлов пакетов.

Затем используйте Обозреватель службы хранилища, чтобы [создать SAS](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#generate-a-sas-in-storage-explorer). 

> [!NOTE]
> Сохраните этот URL-адрес SAS. он будет использоваться позже для обеспечения безопасного доступа пакета развертывания во время выполнения.

### <a name="configure-running-from-a-package-from-your-storage-account"></a>Настройка запуска из пакета из учетной записи хранения
  
Когда вы отправите файл в хранилище BLOB-объектов и получите для него URL SAS, задайте `WEBSITE_RUN_FROM_PACKAGE` для параметра приложения URL-адрес SAS. В следующем примере он выполняется с помощью Azure CLI:

```
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_RUN_FROM_PACKAGE="<your-SAS-URL>"
```

Добавление этого параметра приложения приводит к перезапуску приложения функции. После перезапуска приложения перейдите к нему и убедитесь, что приложение запущено правильно с помощью пакета развертывания. Если приложение не запустилось должным образом, см. [руководство по устранению неполадок при запуске пакета](run-functions-from-deployment-package.md#troubleshooting).

### <a name="encrypt-the-application-setting-using-key-vault-references"></a>Шифрование параметра приложения с помощью ссылок Key Vault

Теперь можно заменить значение `WEBSITE_RUN_FROM_PACKAGE` параметра приложения на Key Vault ссылку на URL-адрес в кодировке SAS. Это обеспечит шифрование URL-адреса SAS в Key Vault, что обеспечивает дополнительный уровень безопасности.

1. Используйте следующую [`az keyvault create`](/cli/azure/keyvault#az-keyvault-create) команду, чтобы создать экземпляр Key Vault.       

    ```azurecli    
    az keyvault create --name "Contoso-Vault" --resource-group <group-name> --location eastus    
    ```    

1. [Чтобы предоставить приложению доступ](../app-service/app-service-key-vault-references.md#granting-your-app-access-to-key-vault) к хранилищу ключей, выполните следующие инструкции:

1. Используйте следующую [`az keyvault secret set`](/cli/azure/keyvault/secret#az-keyvault-secret-set) команду, чтобы добавить внешний URL-адрес в качестве секрета в хранилище ключей:   

    ```azurecli    
    az keyvault secret set --vault-name "Contoso-Vault" --name "external-url" --value "<SAS-URL>"    
    ```    

1.  Используйте следующую [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) команду, чтобы создать `WEBSITE_RUN_FROM_PACKAGE` параметр приложения со значением в качестве Key Vault ссылки на внешний URL-адрес:

    ```azurecli    
    az webapp config appsettings set --settings WEBSITE_RUN_FROM_PACKAGE="@Microsoft.KeyVault(SecretUri=https://Contoso-Vault.vault.azure.net/secrets/external-url/<secret-version>"    
    ```

    `<secret-version>`Будет находиться в выходных данных предыдущей `az keyvault secret set` команды.

Обновление этого параметра приложения приводит к перезапуску приложения функции. После перезапуска приложения перейдите к нему, чтобы убедиться, что оно запущено правильно, используя ссылку на Key Vault.

## <a name="how-to-rotate-the-access-token"></a>Как повернуть маркер доступа

Рекомендуется периодически менять ключ SAS учетной записи хранения. Чтобы приложение-функция не было случайно слабо иметь доступ, необходимо также обновить URL SAS в Key Vault.

1. Чтобы повернуть ключ SAS, перейдите к своей учетной записи хранения в портал Azure. В разделе **Параметры**  >  **ключи доступа** щелкните значок, чтобы повернуть ключ SAS.

1. Скопируйте новый URL-адрес SAS и используйте следующую команду, чтобы задать обновленный URL-адрес SAS в хранилище ключей:

    ```azurecli    
    az keyvault secret set --vault-name "Contoso-Vault" --name "external-url" --value "<SAS-URL>"    
    ``` 

1. Обновите ссылку на хранилище ключей в параметре приложения, указав новую версию секрета:

    ```azurecli    
    az webapp config appsettings set --settings WEBSITE_RUN_FROM_PACKAGE="@Microsoft.KeyVault(SecretUri=https://Contoso-Vault.vault.azure.net/secrets/external-url/<secret-version>"    
    ```

    `<secret-version>`Будет находиться в выходных данных предыдущей `az keyvault secret set` команды.

## <a name="how-to-revoke-the-function-apps-data-access"></a>Как отозвать доступ к данным приложения функции

Существует два способа отмены доступа приложения функции к учетной записи хранения. 

### <a name="rotate-the-sas-key-for-the-azure-storage-account"></a>Смена ключа SAS для учетной записи хранения Azure

Если ключ SAS для учетной записи хранения поворачивается, приложение-функция больше не будет иметь доступа к учетной записи хранения, но будет продолжать работать с последней скачанной версией файла пакета. Перезапустите приложение функции, чтобы очистить последнюю скачанную версию.

### <a name="remove-the-function-apps-access-to-key-vault"></a>Удаление доступа приложения функции к Key Vault

Вы можете отозвать доступ приложения-функции к данным сайта, отключив к нему доступ к Key Vault. Для этого удалите политику доступа для удостоверения приложения функции. Это то же удостоверение, которое вы создали ранее при настройке ссылок на хранилище ключей.

## <a name="summary"></a>Итоги

Теперь файлы приложения шифруются в вашей учетной записи хранения. Когда приложение-функция запускается, оно получает URL-адрес SAS из хранилища ключей. Наконец, приложение функции загружает файлы приложения из учетной записи хранения. 

Если необходимо отозвать доступ приложения-функции к вашей учетной записи хранения, можно отозвать доступ к хранилищу ключей или сменить ключи учетной записи хранения, что сделает URL-адрес SAS недействительным.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="is-there-any-additional-charge-for-running-my-function-app-from-the-deployment-package"></a>Есть ли дополнительная плата за запуск моего приложения-функции из пакета развертывания?

Только затраты, связанные с учетной записью хранения Azure, и соответствующими исходящими расходами.

### <a name="how-does-running-from-the-deployment-package-affect-my-function-app"></a>Как работает из пакета развертывания, влияет на мое приложение функции?

- Запуск приложения из пакета развертывания делает его `wwwroot/` доступным только для чтения. Приложение получает сообщение об ошибке при попытке записи в этот каталог.
- Форматы TAR и GZIP не поддерживаются.
- Эта функция несовместима с локальным кэшем.

## <a name="next-steps"></a>Дальнейшие действия

- [Key Vault ссылки для службы приложений](../app-service/app-service-key-vault-references.md)
- [Шифрование службы хранилища Azure для неактивных данных](../storage/common/storage-service-encryption.md)