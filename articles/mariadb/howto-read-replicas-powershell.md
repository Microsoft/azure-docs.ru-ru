---
title: Управление репликами чтения — Azure PowerShell — база данных Azure для MariaDB
description: Узнайте, как настраивать реплики чтения и управлять ими в базе данных Azure для MariaDB с помощью PowerShell.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 6/10/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 713f99f67706caded0694d88ece220373920e555
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98664993"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mariadb-using-powershell"></a>Создание реплик чтения и управление ими в базе данных Azure для MariaDB с помощью PowerShell

Из этой статьи вы узнаете, как создавать реплики чтения и управлять ими в службе "база данных Azure для MariaDB" с помощью PowerShell. Дополнительные сведения о репликах чтения см. в [этой статье](concepts-read-replicas.md).

## <a name="azure-powershell"></a>Azure PowerShell

Вы можете создавать реплики чтения и управлять ими с помощью PowerShell.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для MariaDB](quickstart-create-mariadb-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль PowerShell Az.MariaDb предоставляется в предварительной версии, его нужно установить отдельно от модуля Az PowerShell с помощью команды `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Как только модуль PowerShell Az.MariaDb станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) .

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

> [!IMPORTANT]
> Функция чтения реплики доступна только для серверов базы данных Azure для MariaDB в общего назначения или ценовой категории, оптимизированные для памяти. Убедитесь, что исходный сервер находится в одной из этих ценовых категорий.

### <a name="create-a-read-replica"></a>Создание реплики чтения

> [!IMPORTANT]
> При создании реплики для источника, не имеющего существующих реплик, источник сначала перезапускается для подготовки к репликации. Это необходимо учесть, то есть выполнять такие операции в период низкой нагрузки.

Чтобы создать сервер реплики чтения, выполните следующую команду:

```azurepowershell-interactive
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  New-AzMariaDbServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

Для команды `New-AzMariaDbServerReplica` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Группа ресурсов, в которой создается сервер реплики.  |
| Имя | mydemoreplicaserver | Имя нового сервера реплики, который создается. |

Чтобы создать реплику чтения между регионами, используйте параметр **Location** . В следующем примере создается реплика в регионе " **Западная часть США** ".

```azurepowershell-interactive
Get-AzMariaDbServer -Name mrdemoserver -ResourceGroupName myresourcegroup |
  New-AzMariaDServerReplica -Name mydemoreplicaserver -ResourceGroupName myresourcegroup -Location westus
```

Дополнительные сведения о том, в каких регионах можно создать реплику, см. в статье [об основных понятиях реплики чтения](concepts-read-replicas.md).

По умолчанию реплики чтения создаются с той же конфигурацией сервера, что и у источника, если не указан параметр **SKU** .

> [!NOTE]
> Рекомендуется, чтобы конфигурация сервера реплики хранилась в значении, превышающем значение источника, чтобы реплика могла поддерживать базу данных master.

### <a name="list-replicas-for-a-source-server"></a>Вывод списка реплик для исходного сервера

Чтобы просмотреть все реплики для данного исходного сервера, выполните следующую команду:

```azurepowershell-interactive
Get-AzMariaDReplica -ResourceGroupName myresourcegroup -ServerName mydemoserver
```

Для команды `Get-AzMariaDReplica` обязательны указанные ниже параметры.

| Параметр | Пример значения | Описание  |
| --- | --- | --- |
| ResourceGroupName |  myresourcegroup |  Группа ресурсов, в которой будет создан сервер реплики.  |
| ServerName | mydemoserver | Имя или идентификатор исходного сервера. |

### <a name="delete-a-replica-server"></a>Удаление сервера-реплики

Для удаления сервера реплики чтения можно выполнить `Remove-AzMariaDbServer` командлет.

```azurepowershell-interactive
Remove-AzMariaDbServer -Name mydemoreplicaserver -ResourceGroupName myresourcegroup
```

### <a name="delete-a-source-server"></a>Удаление исходного сервера

> [!IMPORTANT]
> Удаление исходного сервера приводит к остановке репликации на все серверы-реплики и удалению самого исходного сервера. Серверы-реплики становятся автономными серверами, которые начинают поддерживать операции чтения и записи.

Чтобы удалить исходный сервер, можно выполнить `Remove-AzMariaDbServer` командлет.

```azurepowershell-interactive
Remove-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Перезапуск базы данных Azure для сервера MariaDB с помощью PowerShell](howto-restart-server-powershell.md)