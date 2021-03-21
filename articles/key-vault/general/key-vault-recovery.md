---
title: Общие сведения о восстановлении Azure Key Vault | Документация Майкрософт
description: Функции восстановления Key Vault предназначены для предотвращения случайного или вредоносного удаления хранилища ключей, секретов, ключей и сертификатов, хранящихся в хранилище ключей.
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.author: mbaldwin
author: msmbaldwin
manager: rkarlin
ms.date: 09/30/2020
ms.openlocfilehash: a8e8e791f0dbe18322ad43364ae4ffd09b430caf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98790390"
---
# <a name="azure-key-vault-recovery-management-with-soft-delete-and-purge-protection"></a>Azure Key Vault Управление восстановлением с обратимым удалением и защитой от вирусов

В этой статье рассматриваются две функции восстановления Azure Key Vault, обратимого удаления и защиты от вирусов. В этом документе представлен обзор этих функций и показано, как управлять ими с помощью портал Azure, Azure CLI и Azure PowerShell.

Дополнительные сведения о Key Vault см. в разделе.
- [Обзор Key Vault](overview.md)
- [Общие сведения о ключах, секретах и сертификатах Azure Key Vault](about-keys-secrets-certificates.md)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/dotnet).
* [Модуль PowerShell](/powershell/azure/install-az-ps).
* [Azure CLI](/cli/azure/install-azure-cli)
* Хранилище Key Vault можно создать с помощью [портала Azure](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md) или [Azure PowerShell](../general/quick-create-powershell.md).
* Для выполнения операций с обратимо удаленными хранилищами пользователь должен иметь следующие разрешения (на уровне подписки):

  | Разрешение | Описание |
  |---|---|
  |Microsoft.KeyVault/locations/deletedVaults/read|Отображает свойства обратимо удаленного Key Vault.|
  |Microsoft.KeyVault/locations/deletedVaults/purge/action|Очищает обратимо удаленное хранилище Key Vault.|


## <a name="what-are-soft-delete-and-purge-protection"></a>Что такое обратимое удаление и защита от очистки

Защита от [обратимого удаления](soft-delete-overview.md) и очистки — это две различные функции восстановления хранилища ключей.

> [!IMPORTANT]
> Включение обратимого удаления крайне важно для обеспечения защиты хранилища ключей и учетных данных от случайного удаления. Однако включение обратимого удаления считается критическим изменением, так как может потребовать изменения логики приложения или предоставления дополнительных разрешений для субъектов-служб. Перед включением обратимого удаления с помощью приведенных ниже инструкций убедитесь, что приложение совместимо с изменениями, использующими этот документ [ .](soft-delete-change.md)

**Обратимое удаление** предназначено для предотвращения случайного удаления хранилища ключей и ключей, секретов и сертификатов, хранящихся в хранилище ключей. Обратимое удаление подобно корзине. При удалении хранилища ключей или объекта хранилища ключей оно остается восстановленным для настраиваемого пользователем периода хранения или по умолчанию 90 дней. Хранилища ключей в состоянии обратимого удаления также могут быть **очищены** . Это означает, что они окончательно удаляются. Это позволяет повторно создавать хранилища ключей и объекты хранилища ключей с тем же именем. Для восстановления и удаления хранилищ ключей и объектов требуются разрешения политики повышенного доступа. **После включения обратимого удаления его нельзя отключить.**

Важно отметить, что **имена хранилища ключей являются глобально уникальными**, поэтому вы не сможете создать хранилище ключей с тем же именем, что и хранилище ключей, в состоянии обратимого удаления. Аналогично имена ключей, секретов и сертификатов уникальны в пределах хранилища ключей. Вы не сможете создать секрет, ключ или сертификат с тем же именем, что и у другого в состоянии обратимого удаления.

**Защита от очистки** предназначена для предотвращения удаления вашего хранилища ключей, ключей, секретов и сертификатов вредоносной программой-участником. Подумайте об этом в качестве корзины с блокировкой на основе времени. Вы можете восстановить элементы в любой момент времени в течение настраиваемого срока хранения. **Вы не сможете окончательно удалить или очистить хранилище ключей до истечения срока хранения.** По истечении срока хранения ключа хранилище ключей или объект хранилища ключей будет очищено автоматически.

> [!NOTE]
> Защита от удаления разработана таким образом, что роль администратора или разрешение не могут переопределять, отключать или обходить защиту от очистки. **После включения защиты от удаления она не может быть отключена или переопределена любым пользователем, включая Майкрософт.** Это означает, что необходимо восстановить удаленное хранилище ключей или подождать окончания срока хранения, прежде чем повторно использовать имя хранилища ключей.

Дополнительные сведения об обратимом удалении см. в статье [обзор Azure Key Vault обратимого удаления](soft-delete-overview.md) .

# <a name="azure-portal"></a>[Портал Azure](#tab/azure-portal)

## <a name="verify-if-soft-delete-is-enabled-on-a-key-vault-and-enable-soft-delete"></a>Проверка включения обратимого удаления в хранилище ключей и включение обратимого удаления

1. Войдите на портал Azure.
1. Выберите нужное хранилище ключей.
1. Щелкните колонку "Свойства".
1. Убедитесь, что переключатель рядом с параметром обратимого удаления имеет значение "включить восстановление".
1. Если обратимое удаление не включено в хранилище ключей, щелкните переключатель, чтобы включить обратимое удаление, и нажмите кнопку "Сохранить".

:::image type="content" source="../media/key-vault-recovery-1.png" alt-text="Для свойства обратимое удаление выделяется, как и значение, чтобы включить его.":::

## <a name="grant-access-to-a-service-principal-to-purge-and-recover-deleted-secrets"></a>Предоставление доступа к субъекту-службе для очистки и восстановления удаленных секретов

1. Войдите на портал Azure.
1. Выберите нужное хранилище ключей.
1. Щелкните колонку "Политика доступа".
1. В таблице найдите строку субъекта безопасности, к которому требуется предоставить доступ (или добавьте нового субъекта безопасности).
1. Щелкните раскрывающийся список ключи, сертификаты и секреты.
1. Прокрутите раскрывающийся список вниз и щелкните "восстановить" и "очистить"
1. Субъектам безопасности также требуются функции получения и перечисления для выполнения большинства операций.

:::image type="content" source="../media/key-vault-recovery-2.png" alt-text="В левой области навигации выделены политики доступа. В политиках доступа отображается раскрывающийся список должностей, в котором выделены четыре элемента: Get, List, Recover и очистить.":::

## <a name="list-recover-or-purge-a-soft-deleted-key-vault"></a>Перечисление, восстановление или очистка обратимо удаленного хранилища ключей

1. Войдите на портал Azure.
1. Щелкните строку поиска в верхней части страницы.
1. В разделе "Недавние службы" щелкните "Key Vault". Не щелкайте отдельные хранилища ключей.
1. В верхней части экрана щелкните параметр "Управление удаленными хранилищами".
1. В правой части экрана откроется область контекста.
1. Выберите свою подписку.
1. Если хранилище ключей было обратимо удалено, оно появится в области контекста справа.
1. Если количество хранилищ слишком велико, можно либо щелкнуть "загрузить еще" в нижней части области контекста, либо воспользоваться интерфейсом командной строки или PowerShell, чтобы получить результаты.
1. Найдя хранилище, которое вы хотите восстановить или очистить, установите флажок рядом с ним.
1. Если вы хотите восстановить хранилище ключей, выберите параметр восстановить в нижней части области контекста.
1. Выберите параметр очистить, если вы хотите безвозвратно удалить хранилище ключей.

:::image type="content" source="../media/key-vault-recovery-3.png" alt-text="В хранилищах ключей выделен параметр Управление удаленными хранилищами.":::

:::image type="content" source="../media/key-vault-recovery-4.png" alt-text="В разделе Управление удаленными хранилищами ключей выделено и выделяется только указанное хранилище ключей, а кнопка восстановить выделена.":::

## <a name="list-recover-or-purge-soft-deleted-secrets-keys-and-certificates"></a>Перечисление, восстановление или очистка обратимо удаленных секретов, ключей и сертификатов

1. Войдите на портал Azure.
1. Выберите нужное хранилище ключей.
1. Выберите колонку, соответствующую типу секрета, которым вы хотите управлять (ключи, секреты или сертификаты).
1. В верхней части экрана щелкните "Управление удаленными (ключи, секреты или сертификаты)".
1. В правой части экрана появится область контекста.
1. Если секрет, ключ или сертификат не отображаются в списке, он не находится в состоянии обратимого удаления.
1. Выберите секрет, ключ или сертификат, которым вы хотите управлять.
1. Выберите параметр для восстановления или очистки в нижней части области контекста.

:::image type="content" source="../media/key-vault-recovery-5.png" alt-text="Для ключей параметр Управление удаленными ключами выделен.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

## <a name="key-vault-cli"></a>Key Vault (CLI)

* Проверьте, включено ли обратимое удаление в хранилище ключей.

    ```azurecli
    az keyvault show --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}
    ```

* Включение обратимого удаления в хранилище ключей

    По умолчанию для всех новых хранилищ ключей включено обратимое удаление. Если в настоящее время имеется хранилище ключей, в котором не включено обратимое удаление, используйте следующую команду, чтобы включить обратимое удаление.

    ```azurecli
    az keyvault update --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME} --enable-soft-delete true
    ```

* Удалить хранилище ключей (восстанавливаемое, если обратимое удаление включено)

    ```azurecli
    az keyvault delete --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}
    ```

* Список всех обратимо удаленных хранилищ ключей

    ```azurecli
    az keyvault list-deleted --subscription {SUBSCRIPTION ID} --resource-type vault
    ```

* Восстановление обратимо удаленных ключей — хранилище

    ```azurecli
    az keyvault recover --subscription {SUBSCRIPTION ID} -n {VAULT NAME}
    ```

* Очистить обратимо удаленное хранилище ключей **(предупреждение! Это действие приведет к безвозвратному УДАЛЕНию хранилища ключей** .

    ```azurecli
    az keyvault purge --subscription {SUBSCRIPTION ID} -n {VAULT NAME}
    ```

* Включение очистки с защитой в хранилище ключей

    ```azurecli
    az keyvault update --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME} --enable-purge-protection true
    ```

## <a name="certificates-cli"></a>Сертификаты (CLI)

* Предоставление доступа для очистки и восстановления сертификатов

    ```azurecli
    az keyvault set-policy --upn user@contoso.com --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}  --certificate-permissions recover purge
    ```

* Удаление сертификата

    ```azurecli
    az keyvault certificate delete --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {CERTIFICATE NAME}
    ```

* Список удаленных сертификатов

    ```azurecli
    az keyvault certificate list-deleted --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME}
    ```

* Восстановить удаленный сертификат

    ```azurecli
    az keyvault certificate recover --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {CERTIFICATE NAME}
    ```

* Очистить Обратимо удаленный сертификат **(предупреждение! Эта операция приведет к ОКОНЧАТЕЛЬНОму УДАЛЕНию сертификата)**

    ```azurecli
    az keyvault certificate purge --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {CERTIFICATE NAME}
    ```

## <a name="keys-cli"></a>Ключи (CLI)

* Предоставление доступа для очистки и восстановления ключей

    ```azurecli
    az keyvault set-policy --upn user@contoso.com --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}  --key-permissions recover purge
    ```

* DELETE

    ```azurecli
    az keyvault key delete --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {KEY NAME}
    ```

* Вывод списка удаленных ключей

    ```azurecli
    az keyvault key list-deleted --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME}
    ```

* Восстановить удаленный ключ

    ```azurecli
    az keyvault key recover --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {KEY NAME}
    ```

* Очистить Обратимо удаленный ключ **(предупреждение! Эта операция удалит ключ без возможности восстановления** .

    ```azurecli
    az keyvault key purge --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {KEY NAME}
    ```

## <a name="secrets-cli"></a>Секреты (CLI)

* Предоставление доступа для очистки и восстановления секретов

    ```azurecli
    az keyvault set-policy --upn user@contoso.com --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}  --secret-permissions recover purge
    ```

* Удалить секрет

    ```azurecli
    az keyvault secret delete --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {SECRET NAME}
    ```

* Список удаленных секретов

    ```azurecli
    az keyvault secret list-deleted --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME}
    ```

* Восстановить удаленный секрет

    ```azurecli
    az keyvault secret recover --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {SECRET NAME}
    ```

* Очистить Обратимо удаленный секрет **(предупреждение! Эта операция приведет к ОКОНЧАТЕЛЬНОму УДАЛЕНию СЕКРЕТа** .

    ```azurecli
    az keyvault secret purge --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {SECRET NAME}
    ```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

## <a name="key-vault-powershell"></a>Key Vault (PowerShell)

* Проверьте, включено ли обратимое удаление в хранилище ключей.

    ```powershell
    Get-AzKeyVault -VaultName "ContosoVault"
    ```

* Удаление хранилища ключей

    ```powershell
    Remove-AzKeyVault -VaultName 'ContosoVault'
    ```

* Список всех обратимо удаленных хранилищ ключей

    ```powershell
    Get-AzKeyVault -InRemovedState
    ```

* Восстановление обратимо удаленных ключей — хранилище

    ```powershell
    Undo-AzKeyVaultRemoval -VaultName ContosoVault -ResourceGroupName ContosoRG -Location westus
    ```

* Очистить обратимо удаленное хранилище ключей **(предупреждение) Это действие приведет к безвозвратному УДАЛЕНию хранилища ключей** .

    ```powershell
    Remove-AzKeyVault -VaultName ContosoVault -InRemovedState -Location westus
    ```

* Включение очистки с защитой в хранилище ключей

    ```powershell
    ($resource = Get-AzResource -ResourceId (Get-AzKeyVault -VaultName "ContosoVault").ResourceId).Properties | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true"

    Set-AzResource -resourceid $resource.ResourceId -Properties $resource.Properties
    ```

## <a name="certificates-powershell"></a>Сертификаты (PowerShell)

* Предоставление разрешений на восстановление и очистку сертификатов

    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToCertificates recover,purge
    ```

* Удаление сертификата

  ```powershell
  Remove-AzKeyVaultCertificate -VaultName ContosoVault -Name 'MyCert'
  ```

* Список всех удаленных сертификатов в хранилище ключей

  ```powershell
  Get-AzKeyVaultCertificate -VaultName ContosoVault -InRemovedState
  ```

* Восстановление сертификата в удаленном состоянии

  ```powershell
  Undo-AzKeyVaultCertificateRemoval -VaultName ContosoVault -Name 'MyCert'
  ```

* Очистка обратимо удаленного сертификата **(предупреждение! Эта операция приведет к ОКОНЧАТЕЛЬНОму УДАЛЕНию сертификата)**

  ```powershell
  Remove-AzKeyVaultcertificate -VaultName ContosoVault -Name 'MyCert' -InRemovedState
  ```

## <a name="keys-powershell"></a>Ключи (PowerShell)

* Предоставление разрешений на восстановление и очистку ключей

    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToKeys recover,purge
    ```

* Удаление ключа

  ```powershell
  Remove-AzKeyVaultKey -VaultName ContosoVault -Name 'MyKey'
  ```

* Список всех удаленных сертификатов в хранилище ключей

  ```powershell
  Get-AzKeyVaultKey -VaultName ContosoVault -InRemovedState
  ```

* Восстановление обратимо удаленного ключа

    ```powershell
    Undo-AzKeyVaultKeyRemoval -VaultName ContosoVault -Name ContosoFirstKey
    ```

* Очистка ключа обратимого удаления **(предупреждение! Эта операция удалит ключ без возможности восстановления** .

    ```powershell
    Remove-AzKeyVaultKey -VaultName ContosoVault -Name ContosoFirstKey -InRemovedState
    ```

## <a name="secrets-powershell"></a>Секреты (PowerShell)

* Предоставление разрешений на восстановление и очистку секретов

    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToSecrets recover,purge
    ```

* Удаление секрета с именем SQLPassword

  ```powershell
  Remove-AzKeyVaultSecret -VaultName ContosoVault -name SQLPassword
  ```

* Вывод списка всех удаленных секретов в хранилище ключей

  ```powershell
  Get-AzKeyVaultSecret -VaultName ContosoVault -InRemovedState
  ```

* Восстановление секрета в удаленном состоянии

  ```powershell
  Undo-AzKeyVaultSecretRemoval -VaultName ContosoVault -Name SQLPAssword
  ```

* Очистка секрета в удаленном состоянии **(предупреждение! Эта операция удалит ключ без возможности восстановления** .

  ```powershell
  Remove-AzKeyVaultSecret -VaultName ContosoVault -InRemovedState -name SQLPassword
  ```
---

## <a name="next-steps"></a>Дальнейшие действия

- [Azure Key Vault командлетов PowerShell](/powershell/module/az.keyvault)
- [Key Vault команды Azure CLI](/cli/azure/keyvault)
- [Резервное копирование в Azure Key Vault](backup.md)
- [Включение ведения журнала Key Vault](howto-logging.md)
- [Безопасный доступ к хранилищу ключей](secure-your-key-vault.md)
- [Руководство разработчика Azure Key Vault](developers-guide.md)
- [Рекомендации по использованию хранилища ключей](security-overview.md)