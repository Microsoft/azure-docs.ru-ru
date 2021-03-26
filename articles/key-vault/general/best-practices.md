---
title: Рекомендации по использованию Key Vault-Azure Key Vault | Документация Майкрософт
description: Узнайте о рекомендациях по Azure Key Vault, включая управление доступом, использование отдельных хранилищ ключей, резервное копирование, ведение журнала и параметры восстановления.
services: key-vault
author: msmbaldwin
tags: azure-key-vault
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 01/29/2021
ms.author: mbaldwin
ms.openlocfilehash: e81cbd7e6584f4a280ab9507a989b52d3b188f2d
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105566614"
---
# <a name="best-practices-to-use-key-vault"></a>Рекомендации по использованию Key Vault

## <a name="use-separate-key-vaults"></a>Использование отдельных хранилищ ключей

Мы рекомендуем использовать хранилище для каждого приложения в каждой среде (разработка, подготовка и Рабочая среда). Это позволяет не обмениваться секретами в разных средах, а также снизить угрозу в случае нарушения.

## <a name="control-access-to-your-vault"></a>Управление доступом к хранилищу

Azure Key Vault — это облачная служба, которая обеспечивает защиту ключей шифрования и секретов (например, сертификатов, строк подключения и паролей). Так как это критически важные для бизнеса конфиденциальные данные, следует обеспечить защиту доступа к хранилищу ключей, чтобы доступ могли получить только авторизованные приложения и пользователи. В этой [статье](secure-your-key-vault.md) представлен обзор модели доступа Key Vault. Здесь приводится описание процессов аутентификации и авторизации, а также содержатся сведения о том, как защитить доступ к вашему хранилищу ключей.

Ниже приведены рекомендации по управлению доступом к хранилищу.
1. Блокировка доступа к подписке, группе ресурсов и хранилищам ключей (Azure RBAC)
2. Создание политик доступа для каждого хранилища
3. Использование субъекта доступа с минимальными правами для предоставления доступа
4. Включить [конечные точки службы брандмауэра и виртуальной сети](overview-vnet-service-endpoints.md)

## <a name="backup"></a>Backup

Убедитесь, что вы регулярно создаете резервные копии хранилища на стороне обновления, удаления или создания объектов в хранилище.

### <a name="azure-powershell-backup-commands"></a>Azure PowerShell команды резервного копирования

* [Сертификат резервной копии](/powershell/module/azurerm.keyvault/Backup-AzureKeyVaultCertificate)
* [Создание резервной копии ключа](/powershell/module/azurerm.keyvault/Backup-AzureKeyVaultKey)
* [Секрет резервного копирования](/powershell/module/azurerm.keyvault/Backup-AzureKeyVaultSecret)

### <a name="azure-cli-backup-commands"></a>Azure CLI команды резервного копирования

* [Сертификат резервной копии](/cli/azure/keyvault/certificate#az-keyvault-certificate-backup)
* [Создание резервной копии ключа](/cli/azure/keyvault/key#az-keyvault-key-backup)
* [Секрет резервного копирования](/cli/azure/keyvault/secret#az-keyvault-secret-backup)


## <a name="turn-on-logging"></a>Включить ведение журнала

[Включите ведение журнала](logging.md) для хранилища. Также Настройте оповещения.

## <a name="turn-on-recovery-options"></a>Включение параметров восстановления

1. Включите [обратимое удаление](soft-delete-overview.md).
2. Включите защиту от очистки, если хотите защититься от принудительного удаления секрета или хранилища даже после включения обратимого удаления.