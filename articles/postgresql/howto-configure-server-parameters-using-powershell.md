---
title: Настройка параметров сервера — Azure PowerShell — база данных Azure для PostgreSQL
description: В этой статье описывается, как настроить параметры службы в базе данных Azure для PostgreSQL с помощью PowerShell.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 24106e40a78d4b5607c352ffd49a310533760a66
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92489954"
---
# <a name="customize-azure-database-for-postgresql-server-parameters-using-powershell"></a>Настройка параметров сервера базы данных Azure для PostgreSQL с помощью PowerShell

Вы можете вывести список, отобразить и обновить параметры конфигурации для сервера базы данных Azure для PostgreSQL с помощью PowerShell. Только подмножество конфигураций ядра предоставляется на уровне сервера и может быть изменено.

## <a name="prerequisites"></a>Предварительные требования

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль Az.PostgreSql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az с помощью команды `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Как только модуль Az.PostgreSql PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) .

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="list-server-configuration-parameters-for-azure-database-for-postgresql-server"></a>Получение списка параметров конфигурации сервера для базы данных Azure для сервера PostgreSQL

Чтобы перечислить все изменяемые параметры на сервере и их значения, выполните `Get-AzPostgreSqlConfiguration` командлет.

В следующем примере перечисляются параметры конфигурации сервера **mydemoserver** в группе ресурсов **myresourcegroup**.

```azurepowershell-interactive
Get-AzPostgreSqlConfiguration -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Определение каждого из перечисленных параметров см. в разделе PostgreSQL Reference ( [переменные среды](https://www.postgresql.org/docs/12/libpq-envars.html)).

## <a name="show-server-configuration-parameter-details"></a>Отображение сведений о параметре конфигурации сервера

Чтобы отобразить сведения о конкретном параметре конфигурации для сервера, выполните `Get-AzPostgreSqlConfiguration` командлет и укажите параметр **Name** .

В этом примере показаны подробные сведения о параметре конфигурации сервера **\_ \_ log** **mydemoserver** для сервера в разделе **myresourcegroup** группы ресурсов.

```azurepowershell-interactive
Get-AzPostgreSqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Изменение значения параметра конфигурации сервера

Вы также можете изменить значение определенного параметра конфигурации сервера. При этом обновляется базовое значение конфигурации для ядра сервера. Чтобы обновить конфигурацию, используйте `Update-AzPostgreSqlConfiguration` командлет.

Чтобы обновить параметр конфигурации сервера **\_ \_ log** **mydemoserver** для сервера в разделе **myresourcegroup** группы ресурсов.

```azurepowershell-interactive
Update-AzPostgreSqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver -Value On
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Автоматическое увеличение размера хранилища в базе данных Azure для PostgreSQL с помощью PowerShell](howto-auto-grow-storage-powershell.md).