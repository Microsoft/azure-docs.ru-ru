---
title: Перезапуск сервера-Azure PowerShell — база данных Azure для PostgreSQL
description: В этой статье описывается, как можно перезапустить сервер базы данных Azure для PostgreSQL с помощью PowerShell.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 2eadde4e3158c60685ba30542845f28954651bde
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105609332"
---
# <a name="restart-azure-database-for-postgresql-server-using-powershell"></a>Перезапуск базы данных Azure для сервера PostgreSQL с помощью PowerShell

В этой статье объясняется, как перезапустить сервер в Базе данных Azure для PostgreSQL. Может потребоваться перезапустить сервер для обслуживания, что приводит к кратковременному отключению во время выполнения операции.

Если служба занята, перезагрузка сервера блокируется. Например, служба может обрабатывать запрошенную ранее операцию, такую как масштабирование виртуальных ядер.

> [!NOTE] 
> Время, необходимое для завершения перезапуска, зависит от процесса восстановления PostgreSQL. Чтобы уменьшить время перезапуска, рекомендуем свести к минимуму объем действий, выполняемых на сервере перед перезапуском. Также может потребоваться увеличить частоту контрольных точек. Можно также настроить значения параметров, связанных с контрольной точкой `max_wal_size` , включая. Также рекомендуется выполнить `CHECKPOINT` команду перед перезапуском сервера.

## <a name="prerequisites"></a>Предварительные требования

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

- [Модуль AZ PowerShell](/powershell/azure/install-az-ps) , установленный локально или [Azure Cloud Shell](https://shell.azure.com/) в браузере
- [Сервер базы данных Azure для PostgreSQL](quickstart-create-postgresql-server-database-using-azure-powershell.md)

> [!IMPORTANT]
> Так как модуль Az.PostgreSql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az с помощью команды `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Как только модуль Az.PostgreSql PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы решили использовать PowerShell локально, подключитесь к учетной записи Azure с помощью командлета [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) .

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="restart-the-server"></a>Перезапустите сервер.

Перезапустите сервер с помощью следующей команды:

```azurepowershell-interactive
Restart-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание сервера базы данных Azure для PostgreSQL с помощью PowerShell](quickstart-create-postgresql-server-database-using-azure-powershell.md)