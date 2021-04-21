---
title: Управление ключами учетной записи хранения с помощью Azure Key Vault и Azure CLI
description: Ключи учетной записи хранения обеспечивают простую интеграцию с Azure Key Vault с возможностью доступа на основе ключа к учетной записи службы хранилища Azure.
ms.topic: tutorial
services: key-vault
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/18/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 573e4c9d8db3f07f223826ab648f2ef57e1d9c58
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107766323"
---
# <a name="manage-storage-account-keys-with-key-vault-and-the-azure-cli"></a>Управление ключами учетной записи хранения с помощью Key Vault и Azure CLI
> [!IMPORTANT]
> Мы рекомендуем использовать интеграцию службы хранилища Azure с Azure Active Directory (Azure AD), облачной службой корпорации Майкрософт для управления удостоверениями и доступом. Интеграция с Azure AD доступна для [BLOB-объектов и очередей Azure](../../storage/common/storage-auth-aad.md). Она обеспечивает доступ к службе хранилища Azure на основе токенов OAuth2 (точно так же, как Azure Key Vault). Azure AD позволяет проверять подлинность клиентского приложения, используя удостоверение приложения или пользователя вместо учетных данных учетной записи хранения. Вы можете использовать [управляемое удостоверение Azure AD](../../active-directory/managed-identities-azure-resources/index.yml) при запуске в Azure. Управляемые удостоверения устраняют необходимость проверки подлинности клиента и хранения учетных данных в приложении или вместе с ним. Используйте приведенное ниже решение только в том случае, если аутентификация Azure AD невозможна.

Учетная запись хранения Azure использует учетные данные, состоящие из имени учетной записи и ключа. Ключ генерируется автоматически и служит скорее паролем, чем криптографическим ключом. Key Vault управляет ключами учетной записи хранения, периодически создавая их заново в учетной записи хранения, и предоставляет маркеры подписанного URL-адреса в случае делегированного доступа к ресурсам в учетной записи хранения.

С помощью функции ключа управляемой учетной записи хранения Key Vault можно перечислять (синхронизировать) ключи с учетными записями хранения Azure и периодически повторно создавать (заменять) ключи. Вы можете управлять ключами как для учетных записей хранения, так и для классических учетных записей хранения.

При использовании функции ключей для управляемой учетной записи хранения следует учитывать следующие моменты:

- Значения ключей никогда не возвращаются в ответе вызывающему объекту.
- Управлять ключами учетной записи хранения должно только решение Key Vault. Не управляйте ключами самостоятельно и избегайте конфликтов с процессами Key Vault.
- Управлять ключами учетной записи хранения должен только один объект Key Vault. Не разрешайте управление ключами из нескольких объектов.
- Создавайте ключи повторно только с помощью Key Vault. Не создавайте повторно ключи своей учетной записи хранения вручную.

## <a name="service-principal-application-id"></a>Идентификатор приложения субъекта-службы

Арендатор Azure AD предоставляет каждому зарегистрированному приложению [субъект-службу](../../active-directory/develop/developer-glossary.md#service-principal-object). Субъект-служба выступает в качестве идентификатора приложения, который используется во время настройки авторизации для получения доступа к другим ресурсам Azure с помощью Azure RBAC.

Key Vault — это приложение Майкрософт, которое предварительно зарегистрировано во всех клиентах Azure AD. Key Vault регистрируется с одним и тем же идентификатором приложения в каждом облаке Azure.

| Клиенты | Cloud | Идентификатор приложения |
| --- | --- | --- |
| Azure AD | Azure для государственных организаций | `7e7c393b-45d0-48b1-a35e-2905ddf8183c` |
| Azure AD | общедоступный пиринг Azure; | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |
| Другое  | Любой | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сначала вам следует выполнить перечисленные ниже действия.

- [Установка Azure CLI](/cli/azure/install-azure-cli).
- [Создайте хранилище ключей.](quick-create-cli.md)
- [Создайте учетную запись хранения Azure](../../storage/common/storage-account-create.md?tabs=azure-cli). В имени учетной записи хранения должны использоваться только строчные буквы и цифры. Имя должно содержать от 3 до 24 знаков.
      
## <a name="manage-storage-account-keys"></a>Управление ключами учетной записи хранения

### <a name="connect-to-your-azure-account"></a>Подключение к учетной записи Azure

Проверьте подлинность сеанса Azure CLI с помощью команд [az login](/powershell/module/az.accounts/connect-azaccount).

```azurecli-interactive
az login
``` 

### <a name="give-key-vault-access-to-your-storage-account"></a>Предоставление доступа Key Vault к учетной записи хранения

Используйте команду Azure CLI [az role assignment create](/cli/azure/role/assignment), чтобы предоставить Key Vault доступ к учетной записи хранения. Укажите в команде следующие значения параметров:

- `--role`: передает роль Azure "Роль службы оператора ключей учетных записей хранения". Эта роль ограничивает область доступа учетной записью хранения. Для классической учетной записи хранения вместо этого передается "Роль службы оператора ключей классических учетных записей хранения".
- `--assignee`: передает значение "https://vault.azure.net", которое является URL-адресом для Key Vault в общедоступном облаке Azure. (Для облака Azure для государственных организаций используйте вместо этого "--asingee-object-id". См. раздел [Идентификатор приложения субъекта-службы](#service-principal-application-id).)
- `--scope`: передает идентификатор ресурса учетной записи хранения в форме `/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>`. Чтобы найти идентификатор подписки, используйте команду Azure CLI [az account list](/cli/azure/account?#az_account_list). Чтобы найти имя учетной записи хранения и группу ресурсов учетной записи хранения, используйте команду Azure CLI [az storage account list](/cli/azure/storage/account?#az_storage_account_list).

```azurecli-interactive
az role assignment create --role "Storage Account Key Operator Service Role" --assignee 'https://vault.azure.net' --scope "/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>"
 ```
### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Предоставление разрешения учетной записи пользователя для управляемых учетных записей хранения

Используйте командлет Azure CLI [az keyvault-set-policy](/cli/azure/keyvault?#az_keyvault_set_policy), чтобы обновить политику доступа Key Vault и предоставить учетной записи пользователя разрешения учетной записи хранения.

```azurecli-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --storage-permissions get list delete set update regeneratekey getsas listsas deletesas setsas recover backup restore purge
```

Обратите внимание, что разрешения для учетных записей хранения не доступны на странице "Политики доступа" учетной записи хранения на портале Azure.
### <a name="create-a-key-vault-managed-storage-account"></a>Создание управляемой учетной записи хранения Key Vault

 Создайте управляемую учетную запись хранения Key Vault с помощью команды Azure CLI [az keyvault storage](/cli/azure/keyvault/storage?#az_keyvault_storage_add). Задайте период повторного создания длительностью 90 дней. Когда настанет время замены, Key Vault повторно создаст ключ. Сначала он будет неактивен, но затем Key Vault задаст его в качестве активного. В любой момент времени для выпусков маркеров SAS используется только один ключ, который и является активным. Укажите в команде следующие значения параметров:

- `--vault-name`: передает имя хранилища ключей. Чтобы найти имя хранилища ключей, используйте команду Azure CLI [az keyvault list](/cli/azure/keyvault?#az_keyvault_list).
- `-n`: передает имя учетной записи хранения. Чтобы найти имя учетной записи хранения, используйте команду Azure CLI [az storage account list](/cli/azure/storage/account?#az_storage_account_list).
- `--resource-id`: передает идентификатор ресурса учетной записи хранения в форме `/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>`. Чтобы найти идентификатор подписки, используйте команду Azure CLI [az account list](/cli/azure/account?#az_account_list). Чтобы найти имя учетной записи хранения и группу ресурсов учетной записи хранения, используйте команду Azure CLI [az storage account list](/cli/azure/storage/account?#az_storage_account_list).
   
 ```azurecli-interactive
az keyvault storage add --vault-name <YourKeyVaultName> -n <YourStorageAccountName> --active-key-name key1 --auto-regenerate-key --regeneration-period P90D --resource-id "/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>"
 ```

## <a name="shared-access-signature-tokens"></a>Маркер подписанного URL-адреса

Key Vault также может создавать маркеры подписанного URL-адреса. Подпись общего доступа обеспечивает делегированный доступ к ресурсам в вашей учетной записи хранения. Вы можете предоставить клиентам доступ к ресурсам в учетной записи хранения, не предоставляя общий доступ к ключам учетной записи. Подписанный URL-адрес обеспечивает возможность безопасно предоставлять общий доступ к ресурсам хранилища, не раскрывая ключи учетной записи.

С помощью команд в этом разделе можно выполнить следующие действия:

- Задать определение подписанного URL-адреса учетной записи `<YourSASDefinitionName>`. Определение задается для управляемой учетной записи хранения Key Vault `<YourStorageAccountName>` в хранилище ключей `<YourKeyVaultName>`.
- Создать маркер подписанного URL-адреса учетной записи для служб BLOB-объектов, файлов, таблиц и очередей. Токен создается для служб, контейнеров и объектов типов ресурсов по протоколу HTTPS со всеми разрешениями и заданными датами начала и окончания.
- Задать в хранилище определение подписанного URL-адреса управляемого хранилища Key Vault. Определение содержит шаблон универсального кода ресурса (URI) созданного маркера подписанного URL-адреса. Этот подписанный URL-адрес относится к типу `account` и действителен в течение N дней.
- Убедиться, что подписанный URL-адрес сохранен в хранилище ключей в качестве секрета.

### <a name="create-a-shared-access-signature-token"></a>Создание маркера подписанного URL-адреса

Определение маркера подписанного URL-адреса можно создать с помощью команды Azure CLI [az storage account generate-sas](/cli/azure/storage/account?#az_storage_account_generate_sas). Для этой операции требуются разрешения `storage` и `setsas`.


```azurecli-interactive
az storage account generate-sas --expiry 2020-01-01 --permissions rw --resource-types sco --services bfqt --https-only --account-name <YourStorageAccountName> --account-key 00000000
```
После успешного выполнения операции скопируйте выходные данные.

```console
"se=2020-01-01&sp=***"
```

Эти выходные данные будут переданы параметру `--template-uri` на следующем шаге.

### <a name="generate-a-shared-access-signature-definition"></a>Создание определения подписанного URL-адреса

Чтобы создать определение подписанного URL-адреса, используйте команду Azure CLI [az keyvault storage sas-definition create](/cli/azure/keyvault/storage/sas-definition?#az_keyvault_storage_sas_definition_create), передав выходные данные предыдущего шага в параметр `--template-uri`.  С помощью параметра `-n` можно задать выбранное имя.

```azurecli-interactive
az keyvault storage sas-definition create --vault-name <YourKeyVaultName> --account-name <YourStorageAccountName> -n <YourSASDefinitionName> --validity-period P2D --sas-type account --template-uri <OutputOfSasTokenCreationStep>
```

### <a name="verify-the-shared-access-signature-definition"></a>Проверка определения подписанного URL-адреса

Убедиться в том, что определение подписанного URL-адреса сохранено в хранилище ключей, можно с помощью команды Azure CLI [az keyvault storage sas-definition show](/cli/azure/keyvault/storage/sas-definition?#az_keyvault_storage_sas_definition_show).

Теперь для просмотра содержимого этого секрета можно воспользоваться командой [az keyvault storage sas-definition show](/cli/azure/keyvault/storage/sas-definition?#az_keyvault_storage_sas_definition_show) и свойством `id`.

```azurecli-interactive
az keyvault storage sas-definition show --id https://<YourKeyVaultName>.vault.azure.net/storage/<YourStorageAccountName>/sas/<YourSASDefinitionName>
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [ключах, секретах и сертификатах](/rest/api/keyvault/).
- Ознакомьтесь со статьями в [блоге команды разработчиков Azure Key Vault](/archive/blogs/kv/).
- Просмотрите справочную документацию по [az keyvault storage](/cli/azure/keyvault/storage).
