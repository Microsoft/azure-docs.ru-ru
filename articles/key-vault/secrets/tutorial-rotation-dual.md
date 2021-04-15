---
title: Учебник по смене учетных данных для ресурсов, которые используют два набора учетных данных
description: В этом учебнике описывается, как автоматизировать смену секретов для ресурсов, которые используют два набора учетных данных для аутентификации.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: secrets
ms.topic: tutorial
ms.date: 06/22/2020
ms.author: jalichwa
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d75ba091ff634bf613722e3a194407beeeda68fb
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105967240"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-that-have-two-sets-of-authentication-credentials"></a>Автоматизация смены секретов для ресурсов с двумя наборами учетных данных для аутентификации

Лучшим способом проверки подлинности в службах Azure является использование [управляемого удостоверения](../general/authentication.md), но иногда его использование невозможно. В этих случаях используются ключи доступа или пароли. Ключи доступа и пароли следует часто сменять.

В этом учебнике описано, как автоматизировать периодическую смену секретов для баз данных и служб, которые используют два набора учетных данных для аутентификации. В частности, в этом учебнике показано, как выполнять смену ключей учетной записи службы хранилища Azure, сохраненных в Azure Key Vault в виде секретов. Вы будете использовать функцию, активируемую уведомлением Сетки событий Azure. 

> [!NOTE]
> Ключами учетной записи хранения можно автоматически управлять в Key Vault, предоставляя маркеры подписанного URL-адреса для делегированного доступа к учетной записи хранения. Существуют службы, которым требуются строки подключения учетной записи хранения с ключами доступа. Для такого сценария рекомендуется использовать именно это решение.

На следующей схеме показано решение для смены, описанное в этом учебнике: 

![Схема: решение для смены.](../media/secrets/rotation-dual/rotation-diagram.png)

В этом решении Azure Key Vault хранит отдельные ключи доступа учетной записи хранения как версии одного секрета, меняя местами первичный и вторичный ключи в каждой следующей версии. Когда один из ключей доступа сохраняется в последней версии секрета, другой ключ создается повторно и добавляется в Key Vault как новая последняя версия секрета. Такое решение обеспечивает полный цикл смены в приложении для обновления последнего созданного ключа. 

1. За 30 дней до истечения срока действия секрета Key Vault публикует в Сетке событий соответствующее событие.
1. Сетка событий проверяет подписки на события и с помощью HTTP-запроса методом POST вызывает конечную точку приложения-функции, подписанную на это событие.
1. Приложение-функция идентифицирует другой ключ (не назначенный последней версией) и вызывает учетную запись хранения для его повторного создания.
1. Приложение-функция добавляет повторно созданный ключ в Azure Key Vault в качестве новой версии этого секрета.

## <a name="prerequisites"></a>Предварительные требования
* Подписка Azure. [Создайте ее бесплатно.](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Azure [Cloud Shell](https://shell.azure.com/). В этом учебнике используется портал Cloud Shell и оболочка PowerShell
* Azure Key Vault.
* Две учетные записи хранения Azure.

Если у вас нет хранилища ключей и учетных записей хранения, можно использовать следующую ссылку на развертывание:

[![Ссылка "Развернуть в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2FARM-Templates%2FInitial-Setup%2Fazuredeploy.json)

1. Для параметра **Группа ресурсов** выберите **Создать**. Присвойте группе имя **vaultrotation** и нажмите кнопку **ОК**.
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

    ![Снимок экрана: создание группы ресурсов.](../media/secrets/rotation-dual/dual-rotation-1.png)

Теперь у вас есть хранилище ключей и две учетные записи хранения. Вы можете проверить эти ресурсы в Azure CLI или Azure PowerShell, выполнив следующую команду:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az resource list -o table -g vaultrotation
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzResource -Name 'vaultrotation*' | Format-Table
```
---

Должно отобразиться примерно следующее:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
vaultrotation-kv         vaultrotation      westus      Microsoft.KeyVault/vaults
vaultrotationstorage     vaultrotation      westus      Microsoft.Storage/storageAccounts
vaultrotationstorage2    vaultrotation      westus      Microsoft.Storage/storageAccounts
```

## <a name="create-and-deploy-the-key-rotation-function"></a>Создание и развертывание функции смены ключей

Теперь вам нужно создать приложение-функцию с управляемым системой удостоверением и другими обязательными компонентами. Вы также развернете функцию смены для ключей учетной записи хранения.

Для функции смены в приложении-функции требуются следующие компоненты и настройки:
- План службы приложений Azure.
- учетная запись хранения для управления триггерами приложения-функции;
- Политика доступа для доступа к секретам в Key Vault.
- роль службы оператора ключа учетной записи хранения, назначенная приложению-функции, чтобы оно могло обращаться к ключам доступа учетной записи хранения;
- функция смены ключей с активацией по событию и HTTP-триггеру (смена по запросу);
- подписка на событие Сетки событий для события **SecretNearExpiry**.

1. Щелкните ссылку для развертывания шаблона в Azure: 

   [![Ссылка "Развертывание шаблона в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2FARM-Templates%2FFunction%2Fazuredeploy.json)

1. В списке **Группа ресурсов** выберите **vaultrotation**.
1. В поле **Группа ресурсов учетной записи хранения** введите имя группы ресурсов, в которой расположена ваша учетная запись хранения. Оставьте значение по умолчанию **[resourceGroup().name]** , если ваша учетная запись хранения уже расположена в группе ресурсов, в которой будет развернута функция смены ключа.
1. В поле **Имя учетной записи хранения** введите имя учетной записи хранения, которая содержит сменяемые ключи. Сохраните значение по умолчанию **[concat(resourceGroup().name, 'storage')]** , если вы используете учетную запись хранения, созданную в разделе [Предварительные требования](#prerequisites).
1. В поле **Группа ресурсов Key Vault** введите имя группы ресурсов, в которой расположено ваше хранилище ключей. Оставьте значение по умолчанию **[resourceGroup().name]** , если ваше хранилище ключей уже существует в группе ресурсов, в которой будет развернута функция смены ключа.
1. В поле **Имя Key Vault** введите имя хранилища ключей. Сохраните значение по умолчанию **[concat(resourceGroup().name, '-kv')]** , если вы используете хранилище ключей, созданное в разделе [Предварительные требования](#prerequisites).
1. В поле **App Service Plan Type** (Тип плана службы приложений) выберите план размещения. **План Premium** требуется только в том случае, если хранилище ключей находится за брандмауэром.
1. В поле **Имя приложения-функции** введите соответствующее имя.
1. В поле **Имя секрета** введите имя секрета, в котором будут храниться ключи доступа.
1. В поле **URL-адрес репозитория** введите расположение кода функции в GitHub. В этом учебнике вы можете использовать **https://github.com/Azure-Samples/KeyVault-Rotation-StorageAccountKey-PowerShell.git** .
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

   ![Снимок экрана, на котором создается и развертывается функция.](../media/secrets/rotation-dual/dual-rotation-2.png)

Выполнив описанные выше действия, вы создадите учетную запись хранения, ферму серверов, приложение-функцию и экземпляр Application Insights. Когда развертывание будет завершено, отобразится следующая страница:

   ![Снимок экрана: страница "Развертывание выполнено".](../media/secrets/rotation-dual/dual-rotation-3.png)
> [!NOTE]
> При возникновении сбоя можно щелкнуть **Повторить развертывание**, чтобы завершить развертывание компонентов.


Шаблоны развертывания и код функции смены можно найти в [Azure Samples](https://github.com/Azure-Samples/KeyVault-Rotation-StorageAccountKey-PowerShell).

## <a name="add-the-storage-account-access-keys-to-key-vault"></a>Добавление ключей доступа учетной записи хранения в Key Vault

Для начала настройте политику доступа, которая будет предоставлять субъекту-пользователю разрешение на **управление секретами**:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az keyvault set-policy --upn <email-address-of-user> --name vaultrotation-kv --secret-permissions set delete get list
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Set-AzKeyVaultAccessPolicy -UserPrincipalName <email-address-of-user> --name vaultrotation-kv -PermissionsToSecrets set,delete,get,list
```
---

Теперь вы можете создать новый секрет, указав в качестве его значения ключ доступа учетной записи хранения. Вам также понадобятся идентификатор ресурса учетной записи хранения, значение срока действия секрета и идентификатор ключа для добавления в секрет, чтобы функция смены смогла повторно создать ключ в учетной записи хранения.

Определите идентификатор ресурса учетной записи хранения. Это значение можно найти в свойстве `id`.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az storage account show -n vaultrotationstorage
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccount -Name vaultrotationstorage -ResourceGroupName vaultrotation | Select-Object -Property *
```
---

Получите список ключей доступа к учетной записи хранения, чтобы получить значения ключей:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage -ResourceGroupName vaultrotation
```
---

Добавьте секрет в хранилище ключей с датой окончания срока действия на завтра, сроком действия на 60 дней и идентификатором ресурса учетной записи хранения. Выполните эту команду, используя полученные значения для `key1Value` и `storageAccountResourceId`:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddTHH:mm:ssZ")
az keyvault secret set --name storageKey --vault-name vaultrotation-kv --value <key1Value> --tags "CredentialId=key1" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
$tomorrowDate = (Get-Date).AddDays(+1).ToString('yyy-MM-ddTHH:mm:ssZ')
$secretVaule = ConvertTo-SecureString -String '<key1Value>' -AsPlainText -Force
$tags = @{
    CredentialId='key1'
    ProviderAddress='<storageAccountResourceId>'
    ValidityPeriodDays='60'
}
Set-AzKeyVaultSecret -Name storageKey -VaultName vaultrotation-kv -SecretValue $secretVaule -Tag $tags -Expires $tomorrowDate
```
---

Приведенный выше секрет будет активировать событие `SecretNearExpiry` в течение нескольких минут. Это событие, в свою очередь, активирует функцию для смены секрета с истечением срока действия через 60 дней. В такой конфигурации событие "SecretNearExpiry" будет запускаться каждые 30 дней (30 дней до истечения срока действия), а функция вращения будет чередовать ключи key1 и key2.

Чтобы убедиться, что ключи доступа созданы повторно, вы можете получить ключ учетной записи хранения и секрет Key Vault и сравнить их.

Используйте следующую команду, чтобы получить данные о секрете:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az keyvault secret show --vault-name vaultrotation-kv --name storageKey
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzKeyVaultSecret -VaultName vaultrotation-kv -Name storageKey -AsPlainText
```
---

Обратите внимание, что `CredentialId` получает значение другого `keyName`, а `value` создается повторно:

![Снимок экрана: выходные данные команды "az keyvault secret show" для первой учетной записи хранения.](../media/secrets/rotation-dual/dual-rotation-4.png)

Получите ключи доступа для сравнения значений:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage 
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage -ResourceGroupName vaultrotation
```
---

Обратите внимание, что `value` ключа совпадает с секретом в хранилище ключей.

![Снимок экрана: выходные данные команды az storage account keys list для первой учетной записи хранения.](../media/secrets/rotation-dual/dual-rotation-5.png)

## <a name="add-storage-accounts-for-rotation"></a>Добавление учетных записей хранения для смены

Вы можете повторно использовать одно и то же приложение-функцию для смены ключей для нескольких учетных записей хранения. 

Чтобы добавить ключи учетной записи хранения в существующую функцию для смены, вам потребуется:
- роль службы оператора ключа учетной записи хранения, назначенная приложению-функции, чтобы оно могло обращаться к ключам доступа учетной записи хранения;
- подписка на событие Сетки событий для события **SecretNearExpiry**.

1. Щелкните ссылку для развертывания шаблона в Azure: 

   [![Ссылка "Развертывание шаблона в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2FARM-Templates%2FAdd-Event-Subscriptions%2Fazuredeploy.json)

1. В списке **Группа ресурсов** выберите **vaultrotation**.
1. В поле **Группа ресурсов учетной записи хранения** введите имя группы ресурсов, в которой расположена ваша учетная запись хранения. Оставьте значение по умолчанию **[resourceGroup().name]** , если ваша учетная запись хранения уже расположена в группе ресурсов, в которой будет развернута функция смены ключа.
1. В поле **Имя учетной записи хранения** введите имя учетной записи хранения, которая содержит сменяемые ключи.
1. В поле **Группа ресурсов Key Vault** введите имя группы ресурсов, в которой расположено ваше хранилище ключей. Оставьте значение по умолчанию **[resourceGroup().name]** , если ваше хранилище ключей уже существует в группе ресурсов, в которой будет развернута функция смены ключа.
1. В поле **Имя Key Vault** введите имя хранилища ключей.
1. В поле **Имя приложения-функции** введите соответствующее имя.
1. В поле **Имя секрета** введите имя секрета, в котором будут храниться ключи доступа.
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.

   ![Снимок экрана: создание дополнительной учетной записи хранения.](../media/secrets/rotation-dual/dual-rotation-7.png)

### <a name="add-another-storage-account-access-key-to-key-vault"></a>Добавление дополнительного ключа доступа учетной записи хранения в Key Vault

Определите идентификатор ресурса учетной записи хранения. Это значение можно найти в свойстве `id`.
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az storage account show -n vaultrotationstorage2
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccount -Name vaultrotationstorage -ResourceGroupName vaultrotation | Select-Object -Property *
```
---

Получите список ключей доступа к учетной записи хранения, чтобы получить значения второго ключа:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage2
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage2 -ResourceGroupName vaultrotation
```
---

Добавьте секрет в хранилище ключей с датой окончания срока действия на завтра, сроком действия на 60 дней и идентификатором ресурса учетной записи хранения. Выполните эту команду, используя полученные значения для `key2Value` и `storageAccountResourceId`:

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
$tomorrowDate = (Get-Date).AddDays(+1).ToString('yyy-MM-ddTHH:mm:ssZ')
az keyvault secret set --name storageKey2 --vault-name vaultrotation-kv --value <key2Value> --tags "CredentialId=key2" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddTHH:mm:ssZ")
$secretVaule = ConvertTo-SecureString -String '<key1Value>' -AsPlainText -Force
$tags = @{
    CredentialId='key2';
    ProviderAddress='<storageAccountResourceId>';
    ValidityPeriodDays='60'
}
Set-AzKeyVaultSecret -Name storageKey2 -VaultName vaultrotation-kv -SecretValue $secretVaule -Tag $tags -Expires $tomorrowDate
```
---

Используйте следующую команду, чтобы получить данные о секрете:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az keyvault secret show --vault-name vaultrotation-kv --name storageKey2
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzKeyVaultSecret -VaultName vaultrotation-kv -Name storageKey2 -AsPlainText
```
---

Обратите внимание, что `CredentialId` получает значение другого `keyName`, а `value` создается повторно:

![Снимок экрана: выходные данные команды "az keyvault secret show" для второй учетной записи хранения.](../media/secrets/rotation-dual/dual-rotation-8.png)

Получите ключи доступа для сравнения значений:
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
az storage account keys list -n vaultrotationstorage 
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azurepowershell)

```azurepowershell
Get-AzStorageAccountKey -Name vaultrotationstorage -ResourceGroupName vaultrotation
```
---

Обратите внимание, что `value` ключа совпадает с секретом в хранилище ключей.

![Снимок экрана: выходные данные команды az storage account keys list для второй учетной записи хранения.](../media/secrets/rotation-dual/dual-rotation-9.png)

## <a name="key-vault-rotation-functions-for-two-sets-of-credentials"></a>Функции ротации хранилища ключей для двух наборов учетных данных

Шаблон функций ротации для двух наборов учетных данных и нескольких готовых к использованию функций:

- [шаблон проекта](https://serverlesslibrary.net/sample/bc72c6c3-bd8f-4b08-89fb-c5720c1f997f);
- [Кэш Redis](https://serverlesslibrary.net/sample/0d42ac45-3db2-4383-86d7-3b92d09bc978)
- [Учетная запись хранения](https://serverlesslibrary.net/sample/0e4e6618-a96e-4026-9e3a-74b8412213a4)
- [База данных Cosmos](https://serverlesslibrary.net/sample/bcfaee79-4ced-4a5c-969b-0cc3997f47cc)

> [!NOTE]
> Описанные выше функции ротации создают члены сообщества, а не корпорация Майкрософт. Функции Azure, созданные сообществом, не поддерживаются ни в соответствии с какой-либо программой поддержки Майкрософт, ни какими-либо службами поддержки Майкрософт и предоставляются КАК ЕСТЬ без каких-либо гарантий.

## <a name="next-steps"></a>Дальнейшие действия

- Руководство по [Чередование секретов для одного набора учетных данных](./tutorial-rotation.md)
- Общие сведения. [Мониторинг Key Vault с помощью Сетки событий Azure](../general/event-grid-overview.md)
- Руководство. [Создание первой функции на портале Azure](../../azure-functions/functions-get-started.md)
- Практическое руководство. [Получение сообщения электронной почты при смене секрета в Key Vault](../general/event-grid-logicapps.md)
- Справочные материалы. [Схема событий Сетки событий Azure для Azure Key Vault](../../event-grid/event-schema-key-vault.md)