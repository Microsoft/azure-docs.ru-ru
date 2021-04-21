---
title: Автоматическое расширение хранилища — Azure PowerShell — база данных Azure для PostgreSQL
description: В этой статье описывается, как включить автоматическое расширение хранилища с помощью PowerShell в базе данных Azure для PostgreSQL.
author: rothja
ms.author: jroth
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 05333aa4a42b821366ea7ad0a564781422fda66a
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106551056"
---
# <a name="auto-grow-storage-in-azure-database-for-postgresql-server-using-powershell"></a>Автоматическое увеличение объема хранилища в базе данных Azure для сервера PostgreSQL с помощью PowerShell

В этой статье описывается, как можно настроить расширение хранилища базы данных Azure для PostgreSQL, не влияя на рабочую нагрузку.

Автоматическое увеличение хранилища предотвращает [достижение сервером ограничения хранилища](./concepts-pricing-tiers.md#reaching-the-storage-limit), когда запись становится невозможной. Для серверов с величиной подготовленного хранилища 100 ГБ или меньше размер увеличивается на 5 ГБ, когда объем свободного пространства ниже 10 %. Для серверов с величиной подготовленного хранилища 100 ГБ или больше размер увеличивается на 5 %, когда объем свободного пространства ниже 10 ГБ. Максимальные ограничения хранилища применяются, как указано в разделе, посвященном хранилищу, документа [Ценовые категории базы данных Azure для PostgreSQL](./concepts-pricing-tiers.md#storage).

> [!IMPORTANT]
> Помните, что масштаб хранилища можно только увеличить, но не уменьшить.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps), установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [сервер базы данных Azure для PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md);

> [!IMPORTANT]
> Так как модуль Az.PostgreSql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az с помощью команды `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Как только модуль Az.PostgreSql PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

## <a name="enable-postgresql-server-storage-auto-grow"></a>Включение автоматического увеличения размера хранилища сервера PostgreSQL

Включите автоматическое расширение хранилища на существующем сервере с помощью следующей команды.

```azurepowershell-interactive
Update-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -StorageAutogrow Enabled
```

Включите автоматическое расширение хранилища сервера при создании нового сервера с помощью следующей команды.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -StorageAutogrow Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание реплик чтения и управление ими в Базе данных Azure для MySQL с помощью PowerShell](howto-read-replicas-powershell.md).