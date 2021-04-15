---
title: Создание резервной копии секрета, ключа или сертификата, сохраненного в Azure Key Vault | Документация Майкрософт
description: Этот документ поможет при резервном копировании секрета, ключа или сертификата, хранимых в Azure Key Vault.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 3/18/2021
ms.author: sudbalas
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3b148ac83b89850cad66bcd7254d385e655cc2fb
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105968753"
---
# <a name="azure-key-vault-backup"></a>Резервное копирование в Azure Key Vault

В этом документе показано, как создать резервную копию секретов, ключей и сертификатов, хранящихся в хранилище ключей. Резервное копирование позволяет создать автономную копию всех секретов на тот маловероятный случай, если доступ к хранилищу ключей будет утерян.

## <a name="overview"></a>Обзор

Azure Key Vault автоматически предоставляет несколько функций для поддержания доступности и предотвращения потери данных. Создавайте резервные копии секретов, только если у вас есть для этого серьезное бизнес-обоснование. Создание резервной копии секретов в хранилище ключей может добавить значительные операционные трудности, как, например, обслуживание нескольких наборов журналов, разрешений и резервных копий по истечении срока действия секретов или при их смене.

Key Vault сохраняет доступность в аварийных сценариях и автоматически выполняет отработку отказа запросов в парный регион без вмешательства пользователя. Дополнительные сведения см. в статье [Доступность и избыточность хранилища ключей Azure](./disaster-recovery-guidance.md).

Если вам требуется защита от случайного или злонамеренного удаления секретов, настройте в хранилище ключей функции обратимого удаления и защиты от очистки. Дополнительные сведения см. в статье [Общие сведения об обратимом удалении в Azure Key Vault](./soft-delete-overview.md).

## <a name="limitations"></a>Ограничения

> [!IMPORTANT]
> Key Vault не поддерживает резервное копирование более 500 предыдущих версий ключа, секрета или объекта сертификата. Попытка резервного копирования ключа, секрета или объекта сертификата может привести к ошибке. Удалить предыдущие версии ключа, секрета или сертификата нельзя.

Сейчас Key Vault не поддерживает резервное копирование всего хранилища ключей в одной операции. Корпорация Майкрософт и команда Azure Key Vault не поддерживает попытки применить команды, перечисленные в этом документе, для автоматического создания резервных копий хранилища ключей, и предупреждает о возможном возникновении ошибок. 

Учтите также приведенные ниже последствия.

* Создание резервной копии секретов, для которых существует несколько версий, может приводить к ошибкам времени ожидания.
* Резервная копия создается как моментальный снимок на определенный момент времени. Если секреты обновятся во время создания резервной копии, это приведет к несоответствию ключей шифрования.
* Если вы превысите ограничения службы хранилища ключей на количество запросов в секунду, к хранилищу ключей будет применяться регулирование, что приведет к сбою процесса резервного копирования.

## <a name="design-considerations"></a>Рекомендации по проектированию

При создании резервной копии объекта, хранимого в хранилище ключей (секрета, ключа или сертификата), он скачивается как зашифрованный большой двоичный объект. Этот большой двоичный объект невозможно расшифровать за пределами Azure. Чтобы получить пригодные для использования данные из этого большого двоичного объекта, необходимо восстановить его в хранилище ключей в той же подписке и [географической области Azure](https://azure.microsoft.com/global-infrastructure/geographies/).

## <a name="prerequisites"></a>Предварительные требования

Чтобы создать резервную копию объекта, сохраненного в хранилище ключей, вам потребуется следующее. 

* Разрешения уровня участника или более высокого уровня в подписке Azure.
* Первичное хранилище ключей с теми секретами, резервные копии которых нужно создать.
* Вторичное хранилище ключей, в котором будут восстановлены секреты.

## <a name="back-up-and-restore-from-the-azure-portal"></a>Создание резервной копии и восстановление с помощью портала Azure

Чтобы создать резервную копию объекта или восстановить его с помощью портала Azure, выполните перечисленные в этом разделе шаги.

### <a name="back-up"></a>Резервное копирование

1. Перейдите на портал Azure.
2. Выберите нужное хранилище ключей.
3. Перейдите к объекту (секрету, ключу или сертификату), резервную копию которого вы намерены создать.

    ![Снимок экрана для процесса выбора параметров ключа и объекта в хранилище ключей.](../media/backup-1.png)

4. Выберите объект.
5. Выберите **Скачать резервную копию**.

    ![Снимок экрана с кнопкой "Скачать резервную копию" в хранилище ключей.](../media/backup-2.png)
    
6. Выберите **Скачать**.

    ![Снимок экрана с кнопкой "Скачать" в хранилище ключей.](../media/backup-3.png)
    
7. Храните зашифрованный большой двоичный объект в безопасном расположении.

### <a name="restore"></a>Восстановить

1. Перейдите на портал Azure.
2. Выберите нужное хранилище ключей.
3. Перейдите к типу объекта (секрет, ключ или сертификат), который нужно восстановить.
4. Выберите **Восстановить резервную копию**.

    ![Снимок экрана с кнопкой "Восстановить резервную копию" в хранилище ключей.](../media/backup-4.png)
    
5. Перейдите к расположению, где сохранен зашифрованный большой двоичный объект.
6. Щелкните **ОК**.

## <a name="back-up-and-restore-from-the-azure-cli-or-azure-powershell"></a>Создание резервной копии и восстановление с помощью Azure CLI или Azure PowerShell

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
```azurecli
## Log in to Azure
az login

## Set your subscription
az account set --subscription {AZURE SUBSCRIPTION ID}

## Register Key Vault as a provider
az provider register -n Microsoft.KeyVault

## Back up a certificate in Key Vault
az keyvault certificate backup --file {File Path} --name {Certificate Name} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Back up a key in Key Vault
az keyvault key backup --file {File Path} --name {Key Name} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Back up a secret in Key Vault
az keyvault secret backup --file {File Path} --name {Secret Name} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Restore a certificate in Key Vault
az keyvault certificate restore --file {File Path} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Restore a key in Key Vault
az keyvault key restore --file {File Path} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}

## Restore a secret in Key Vault
az keyvault secret restore --file {File Path} --vault-name {Key Vault Name} --subscription {SUBSCRIPTION ID}
```
# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

```azurepowershell
## Log in to Azure
Connect-AzAccount

## Set your subscription
Set-AzContext -Subscription '{AZURE SUBSCRIPTION ID}'

## Back up a certificate in Key Vault
Backup-AzKeyVaultCertificate -VaultName '{Certificate Name}' -Name '{Key Vault Name}'

## Back up a key in Key Vault
Backup-AzKeyVaultKey -VaultName '{Key Name}' -Name '{Key Vault Name}'

## Back up a secret in Key Vault
Backup-AzKeyVaultSecret -VaultName '{Key Vault Name}' -Name '{Secret Name}'

## Restore a certificate in Key Vault
Restore-AzKeyVaultCertificate -VaultName '{Key Vault Name}' -InputFile '{File Path}'

## Restore a key in Key Vault
Restore-AzKeyVaultKey -VaultName '{Key Vault Name}' -InputFile '{File Path}'

## Restore a secret in Key Vault
Restore-AzKeyVaultSecret -VaultName '{Key Vault Name}' -InputFile '{File Path}'
```
---

## <a name="next-steps"></a>Дальнейшие действия

Включите [ведение журнала и мониторинг](./logging.md) для Key Vault.