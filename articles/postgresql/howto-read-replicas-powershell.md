---
title: Управление репликами чтения — Azure PowerShell — база данных Azure для PostgreSQL
description: Узнайте, как настраивать реплики чтения и управлять ими в базе данных Azure для PostgreSQL с помощью PowerShell.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b0a5547928bd7d19343c50e40ab9fcb2c335e893
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97674537"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-postgresql-using-powershell"></a>Создание реплик чтения и управление ими в базе данных Azure для PostgreSQL с помощью PowerShell

Из этой статьи вы узнаете, как создавать реплики чтения и управлять ими в службе "база данных Azure для PostgreSQL" с помощью PowerShell. Дополнительные сведения о репликах чтения см. в [этой статье](concepts-read-replicas.md).

## <a name="azure-powershell"></a>Azure PowerShell

Вы можете создавать реплики чтения и управлять ими с помощью PowerShell.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль Az.PostgreSql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az с помощью команды `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Как только модуль Az.PostgreSql PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) .

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Функция чтения реплики доступна только для серверов базы данных Azure для PostgreSQL в общего назначения или ценовой категории, оптимизированные для памяти. Убедитесь, что сервер-источник находится в одной из этих ценовых категорий.

### <a name="create-a-read-replica"></a>Создание реплики чтения

Чтобы создать сервер реплики чтения, выполните следующую команду:

```azurepowershell-interactive
Get-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  New-AzPostgreSqlServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

Для команды `New-AzPostgreSqlServerReplica` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Группа ресурсов, в которой создается сервер реплики.  |
| Имя | mydemoreplicaserver | Имя нового сервера реплики, который создается. |

Чтобы создать реплику чтения между регионами, используйте параметр **Location** . В следующем примере создается реплика в регионе " **Западная часть США** ".

```azurepowershell-interactive
Get-AzPostgreSqlServer -Name mrdemoserver -ResourceGroupName myresourcegroup |
  New-AzPostgreSQLServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup -Location westus
```

Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md).

По умолчанию реплики чтения создаются с той же конфигурацией сервера, что и у источника, если не указан параметр **SKU** .

> [!NOTE]
> Рекомендуется, чтобы конфигурация сервера реплики оставалась равна или больше, чем база данных-источник, чтобы гарантировать, что реплика сможет поддерживать базу данных master.

### <a name="list-replicas-for-a-primary-server"></a>Вывод списка реплик для сервера-источника

Чтобы просмотреть все реплики для данного сервера-источника, выполните следующую команду:

```azurepowershell-interactive
Get-AzPostgreSQLReplica -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Для команды `Get-AzPostgreSQLReplica` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Группа ресурсов, в которой будет создан сервер реплики.  |
| ServerName | mydemoserver | Имя или идентификатор сервера источника. |

### <a name="stop-a-replica-server"></a>Останавливает сервер реплики

Остановка сервера реплики чтения повышает, что реплика чтения становится независимым сервером. Это можно сделать, запустив `Update-AzPostgreSqlServer` командлет и задав для параметра репликатионроле значение `None` .

```azurepowershell-interactive
Update-AzPostgreSqlServer -Name mydemoreplicaserver -ResourceGroupName myresourcegroup -ReplicationRole None
```

### <a name="delete-a-replica-server"></a>Удаление сервера-реплики

Для удаления сервера реплики чтения можно выполнить `Remove-AzPostgreSqlServer` командлет.

```azurepowershell-interactive
Remove-AzPostgreSqlServer -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

### <a name="delete-a-primary-server"></a>Удаление сервера-источника

> [!IMPORTANT]
> Удаление сервера-источника останавливает репликацию на все серверы-реплики и удаляет сам основной сервер. Серверы-реплики становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

Чтобы удалить сервер-источник, можно выполнить `Remove-AzPostgreSqlServer` командлет.

```azurepowershell-interactive
Remove-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Перезапуск базы данных Azure для сервера PostgreSQL с помощью PowerShell](howto-restart-server-powershell.md)