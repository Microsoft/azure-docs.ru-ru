---
title: 'PowerShell: enable My-My-Key (BYOK) TDE'
titleSuffix: Azure SQL Managed Instance
description: Узнайте, как настроить Управляемый экземпляр Azure SQL, чтобы приступить к использованию прозрачное шифрование данных (TDE) для шифрования неактивных ключей с помощью PowerShell.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MladjoA
ms.author: mlandzic
ms.reviewer: vanto, sstein
ms.date: 08/25/2020
ms.openlocfilehash: 34a849fde315b45bdb1df577cf26c91f458abd72
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91323242"
---
# <a name="transparent-data-encryption-in-sql-managed-instance-using-your-own-key-from-azure-key-vault"></a>прозрачное шифрование данных в SQL Управляемый экземпляр с помощью собственного ключа из Azure Key Vault

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Этот пример сценария PowerShell настраивает прозрачное шифрование данных (TDE) с помощью ключа, управляемого клиентом, для Управляемый экземпляр SQL Azure, используя ключ из Azure Key Vault. Это часто называется сценарием "BYOK" с собственным ключом для TDE. Дополнительные сведения см. в статье [прозрачное шифрование данных Azure SQL с ключом, управляемым клиентом](../../database/transparent-data-encryption-byok-overview.md).

## <a name="prerequisites"></a>Предварительные условия

- управляемый экземпляр. См. раздел [Использование PowerShell для создания управляемого экземпляра](create-configure-managed-instance-powershell.md).

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Для использования PowerShell локально или Azure Cloud Shell требуется Azure PowerShell 2.3.2 или более поздней версии. Если необходимо выполнить обновление, обратитесь к разделу [установка Azure PowerShell Module](/powershell/azure/install-az-ps)или выполните приведенный ниже пример скрипта, чтобы установить модуль для текущего пользователя:

`Install-Module -Name Az -AllowClobber -Scope CurrentUser`

Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="sample-scripts"></a>Примеры сценариев 

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/transparent-data-encryption/setup-tde-byok-sqlmi.ps1 "Set up BYOK TDE for SQL Managed Instance")]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).

Дополнительные примеры сценариев PowerShell для Управляемого экземпляра SQL можно найти в статье [Сценарии PowerShell для Управляемого экземпляра SQL Azure](../../database/powershell-script-content-guide.md).
