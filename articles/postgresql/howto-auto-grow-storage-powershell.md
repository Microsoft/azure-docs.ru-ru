---
title: Автоматическое расширение хранилища — Azure PowerShell — база данных Azure для PostgreSQL
description: В этой статье описывается, как включить автоматическое расширение хранилища с помощью PowerShell в базе данных Azure для PostgreSQL.
author: ambhatna
ms.author: ambhatna
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: bc4655ce6cd572183cd92e1c8b2ac10e613ebd8f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92489971"
---
# <a name="auto-grow-storage-in-azure-database-for-postgresql-server-using-powershell"></a>Автоматическое увеличение объема хранилища в базе данных Azure для сервера PostgreSQL с помощью PowerShell

В этой статье описывается, как можно настроить расширение хранилища базы данных Azure для PostgreSQL, не влияя на рабочую нагрузку.

Автоматическое увеличение хранилища предотвращает [достижение сервером ограничения хранилища](./concepts-pricing-tiers.md#reaching-the-storage-limit) и становится только чтением. Для серверов с 100 ГБ или меньше подготовленного хранилища размер увеличивается на 5 ГБ, когда объем свободного пространства ниже 10%. Для серверов с более чем 100 ГБ подготовленного хранилища размер увеличивается на 5%, если объем свободного пространства меньше 10 ГБ. Максимальные ограничения хранилища применяются, как указано в разделе Storage ценовых категорий [базы данных Azure для PostgreSQL](./concepts-pricing-tiers.md#storage).

> [!IMPORTANT]
> Помните, что хранилище можно масштабировать только вверх, а не вниз.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль Az.PostgreSql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az с помощью команды `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Как только модуль Az.PostgreSql PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) .

## <a name="enable-postgresql-server-storage-auto-grow"></a>Включить автоматическое увеличение размера хранилища сервера PostgreSQL

Включите автоматическое расширение хранилища на сервере на существующем сервере с помощью следующей команды:

```azurepowershell-interactive
Update-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -StorageAutogrow Enabled
```

Включите автоматическое расширение хранилища сервера при создании нового сервера с помощью следующей команды:

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -StorageAutogrow Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Как создавать реплики чтения и управлять ими в базе данных Azure для PostgreSQL с помощью PowerShell](howto-read-replicas-powershell.md).