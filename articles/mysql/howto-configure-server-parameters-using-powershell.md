---
title: Настройка параметров сервера — Azure PowerShell — база данных Azure для MySQL
description: В этой статье описывается, как настроить параметры службы в базе данных Azure для MySQL с помощью PowerShell.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurepowershell
ms.topic: how-to
ms.date: 10/1/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 973025dfd8c0141ed0884539fe5207cc64ec822c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94541867"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-powershell"></a>Настройка параметров сервера в базе данных Azure для MySQL с помощью PowerShell

Вы можете вывести список, отобразить и обновить параметры конфигурации для сервера базы данных Azure для MySQL с помощью PowerShell. Только подмножество конфигураций ядра предоставляется на уровне сервера и может быть изменено.

>[!Note]
> Параметры сервера можно обновлять глобально на уровне сервера с помощью [Azure CLI](./howto-configure-server-parameters-using-cli.md), [PowerShell](./howto-configure-server-parameters-using-powershell.md)или [портала Azure](./howto-server-parameters.md).

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль Az.MySql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az PowerShell с помощью следующей команды: `Install-Module -Name Az.MySql -AllowPrerelease`.
> Как только модуль PowerShell Az.MySql станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/Connect-AzAccount) .

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>Получение списка параметров конфигурации сервера для базы данных Azure для сервера MySQL

Чтобы перечислить все изменяемые параметры на сервере и их значения, выполните `Get-AzMySqlConfiguration` командлет.

В следующем примере перечисляются параметры конфигурации сервера **mydemoserver** в группе ресурсов **myresourcegroup**.

```azurepowershell-interactive
Get-AzMySqlConfiguration -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Определение каждого из перечисленных параметров см. в разделе ссылок MySQL на странице[Server System Variables](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) (Системные переменные сервера).

## <a name="show-server-configuration-parameter-details"></a>Отображение сведений о параметре конфигурации сервера

Чтобы отобразить сведения о конкретном параметре конфигурации для сервера, выполните `Get-AzMySqlConfiguration` командлет и укажите параметр **Name** .

В этом примере показаны подробные сведения о параметре конфигурации сервера **\_ \_ log** **mydemoserver** для сервера в разделе **myresourcegroup** группы ресурсов.

```azurepowershell-interactive
Get-AzMySqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

## <a name="modify-a-server-configuration-parameter-value"></a>Изменение значения параметра конфигурации сервера

Вы также можете изменить значение определенного параметра конфигурации сервера. При этом обновляется базовое значение конфигурации для ядра СУБД сервера MySQL. Чтобы обновить конфигурацию, используйте `Update-AzMySqlConfiguration` командлет.

Чтобы обновить параметр конфигурации сервера **\_ \_ log** **mydemoserver** для сервера в разделе **myresourcegroup** группы ресурсов.

```azurepowershell-interactive
Update-AzMySqlConfiguration -Name slow_query_log -ResourceGroupName myresourcegroup -ServerName mydemoserver -Value On
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Автоматическое увеличение объема хранилища в базе данных Azure для сервера MySQL с помощью PowerShell](howto-auto-grow-storage-powershell.md).
