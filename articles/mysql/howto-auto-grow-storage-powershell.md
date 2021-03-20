---
title: Автоматическое увеличение хранилища Azure PowerShell — база данных Azure для MySQL
description: В этой статье описывается, как включить автоматическое расширение хранилища с помощью PowerShell в базе данных Azure для MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 4/28/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e8c9f6f66e484fbd9ebe5c15934936d6e5c59073
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94542037"
---
# <a name="auto-grow-storage-in-azure-database-for-mysql-server-using-powershell"></a>Автоматическое увеличение объема хранилища в базе данных Azure для сервера MySQL с помощью PowerShell

В этой статье описывается, как можно настроить расширение хранилища базы данных Azure для MySQL, не влияя на рабочую нагрузку.

Автоматическое увеличение хранилища предотвращает [достижение сервером ограничения хранилища](./concepts-pricing-tiers.md#reaching-the-storage-limit) и становится только чтением. Для серверов с 100 ГБ или меньше подготовленного хранилища размер увеличивается на 5 ГБ, когда объем свободного пространства ниже 10%. Для серверов с более чем 100 ГБ подготовленного хранилища размер увеличивается на 5%, если объем свободного пространства меньше 10 ГБ. Максимальные ограничения хранилища применяются, как указано в разделе "хранилище" [для ценовых категорий базы данных Azure для MySQL](./concepts-pricing-tiers.md#storage).

> [!IMPORTANT]
> Помните, что хранилище можно масштабировать только вверх, а не вниз.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль Az.MySql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az PowerShell с помощью следующей команды: `Install-Module -Name Az.MySql -AllowPrerelease`.
> Как только модуль PowerShell Az.MySql станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/Connect-AzAccount) .

## <a name="enable-mysql-server-storage-auto-grow"></a>Включение автоматического увеличения хранилища сервера MySQL

Включите автоматическое расширение хранилища на сервере на существующем сервере с помощью следующей команды:

```azurepowershell-interactive
Update-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -StorageAutogrow Enabled
```

Включите автоматическое расширение хранилища сервера при создании нового сервера с помощью следующей команды:

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -StorageAutogrow Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Как создавать реплики чтения и управлять ими в базе данных Azure для MySQL с помощью PowerShell](howto-read-replicas-powershell.md).