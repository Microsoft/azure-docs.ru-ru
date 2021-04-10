---
title: Руководство по проектированию сервера Базы данных Azure для MariaDB с помощью Azure PowerShell
description: Этот учебник содержит сведения о создании базы данных и сервера Базы данных Azure для MariaDB и управлении ими с помощью PowerShell.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurepowershell
ms.topic: tutorial
ms.date: 05/26/2020
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 9b4500df459e4d4ef67f97dc4fa923988f30401b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98664507"
---
# <a name="tutorial-design-an-azure-database-for-mariadb-using-powershell"></a>Руководство по проектированию Базы данных Azure для MariaDB с помощью PowerShell

База данных Azure для MariaDB — это служба реляционной базы данных в среде Microsoft Cloud на основе ядра СУБД MariaDB Community Edition. При работе с этим учебником вы будете использовать PowerShell и другие средства, чтобы выполнить следующие операции:

> [!div class="checklist"]
> - создание Базы данных Azure для MariaDB;
> - настройка брандмауэра сервера;
> - использование [программы командной строки MySQL](https://dev.mysql.com/doc/refman/5.6/en/mysql.html) для создания базы данных.
> - Загрузка примера данных
> - Данные запросов
> - Обновление данных
> - восстановление данных.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Так как модуль PowerShell Az.MariaDb предоставляется в предварительной версии, его нужно установить отдельно от модуля Az PowerShell с помощью команды `Install-Module -Name Az.MariaDb -AllowPrerelease`.
> Как только модуль PowerShell Az.MariaDb станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и встроен в Azure Cloud Shell.

Если вы впервые используете службу "База данных Azure для MariaDB", необходимо зарегистрировать поставщик ресурсов **Microsoft.DBforMariaDB**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.DBforMariaDB
```

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Если вы используете несколько подписок Azure, выберите ту, за ресурсы в которой будут выставляться счета. Выберите идентификатор требуемой подписки с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов Azure](../azure-resource-manager/management/overview.md) с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа.

В следующем примере создается группа ресурсов с именем **myresourcegroup** в регионе **Западная часть США**.

```azurepowershell-interactive
New-AzResourceGroup -Name myresourcegroup -Location westus
```

## <a name="create-an-azure-database-for-mariadb-server"></a>Создание сервера Базы данных Azure для MariaDB

Создайте сервер Базы данных Azure для MariaDB с помощью командлета `New-AzMariaDbServer`. Сервер может управлять несколькими базами данных. Как правило, для каждого проекта и для каждого пользователя используется отдельная база данных.

В следующем примере в регионе **Западная часть США** создается сервер MariaDB с именем **mydemoserver** в группе ресурсов **myresourcegroup** с администратором сервера с именем **myadmin**. Это сервер 5-го поколения ценовой категории "Общего назначения" с двумя виртуальными ядрами и геоизбыточным резервным копированием. Запишите пароль, указанный в первой строке примера, так как это пароль для учетной записи администратора сервера MariaDB.

> [!TIP]
> Имя сервера сопоставляется с DNS-именем и должно быть глобально уникальным в Azure.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -GeoRedundantBackup Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

Значение параметра **SKU** соответствует формату **ценовая-категория\_поколение-вычислительных-ресурсов\_количество-виртуальных-ядер**, как показано в примерах ниже.

- `-Sku B_Gen5_1` — "Базовый", поколение 5, 1 виртуальное ядро; Это номер SKU наименьший по размеру из доступных.
- `-Sku GP_Gen5_32` — "Общего назначения", поколение 5, 32 виртуальных ядра;
- `-Sku MO_Gen5_2` — "Оптимизированная для операций в памяти", поколение 5, 2 виртуальных ядра.

См. сведения о допустимых значениях **SKU** по регионам и ценовым категориям в описании [ценовых категорий Базы данных Azure для MariaDB](./concepts-pricing-tiers.md).

Используйте ценовую категорию "Базовый", если для вашей рабочей нагрузки не требуется большое количество вычислительных ресурсов и операций ввода-вывода.

> [!IMPORTANT]
> Серверы, созданные в ценовой категории "Базовый", не удастся позже масштабировать до ценовых категорий "Общего назначения" или "Оптимизировано для памяти" и к ним нельзя применить георепликацию.

## <a name="configure-a-firewall-rule"></a>Настройка правила брандмауэра

Создайте правило брандмауэра для сервера Базы данных Azure для MariaDB, используя командлет `New-AzMariaDbFirewallRule`. Правило брандмауэра на уровне сервера позволяет внешним приложениям, таким как программа командной строки `mysql` или MariaDB Workbench, подключаться к серверу через брандмауэр службы "База данных Azure для MariaDB".

В приведенном ниже примере создается правило брандмауэра с именем **AllowMyIP**, которое разрешает подключения с определенного IP-адреса (192.168.0.1). Замените его IP-адресом или диапазоном IP-адресов, которые применяются для того расположения, из которого вы подключаетесь.

```azurepowershell-interactive
New-AzMariaDbFirewallRule -Name AllowMyIP -ResourceGroupName myresourcegroup -ServerName mydemoserver -StartIPAddress 192.168.0.1 -EndIPAddress 192.168.0.1
```

> [!NOTE]
> Подключитесь к Базе данных Azure для MariaDB через порт 3306. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 3306 может быть запрещен. В этом сценарии вы сможете подключиться к серверу, только если ИТ-отдел откроет для вас порт 3306.

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа. Чтобы получить сведения о подключении, используйте следующий пример. Запишите значения **FullyQualifiedDomainName** и **AdministratorLogin**.

```azurepowershell-interactive
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Select-Object -Property FullyQualifiedDomainName, AdministratorLogin
```

```Output
FullyQualifiedDomainName                    AdministratorLogin
------------------------                    ------------------
mydemoserver.mariadb.database.azure.com       myadmin
```

## <a name="connect-to-the-server-using-the-mysql-command-line-tool"></a>Подключение к серверу с помощью средства командной строки mysql

Подключитесь к серверу с помощью средства командной строки `mysql`. Скачать и установить его можно [здесь](https://dev.mysql.com/downloads/shell/). Вы можете также получить доступ к предустановленной версии средства командной строки `mysql` из Azure Cloud Shell, нажав кнопку **Попробовать** в примере кода в этой статье. Azure Cloud Shell также можно открыть, нажав кнопку **>_** на панели инструментов вверху справа на портале Azure или перейдя по адресу [shell.azure.com](https://shell.azure.com/).

```azurepowershell-interactive
mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="create-a-database"></a>Создание базы данных

Подключившись к серверу, создайте пустую базу данных.

```sql
mysql> CREATE DATABASE mysampledb;
```

Выполните следующую команду в командной строке, чтобы подключиться к созданной базе данных:

```sql
mysql> USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>Создание таблиц в базе данных

Теперь, когда вы знаете, как подключиться к базе данных Azure для MariaDB, выполните основные задачи.

Сначала создайте таблицу и заполните ее некоторыми данными. Давайте создадим таблицу, в которой хранятся данные инвентаризации.

```sql
CREATE TABLE inventory (
  id serial PRIMARY KEY,
  name VARCHAR(50),
  quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>Загрузка данных в таблицу

Теперь, когда таблица создана, мы можем вставить в нее данные. Чтобы вставить некоторые строки данных, в открытом окне командной строки выполните следующий запрос:

```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150);
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Итак, в созданной ранее таблице содержится две строки данных.

## <a name="query-and-update-the-data-in-the-tables"></a>Запрос и обновление данных в таблицах

Чтобы извлечь сведения из таблицы базы данных, выполните приведенный ниже запрос.

```sql
SELECT * FROM inventory;
```

Вы можете также обновить данные в таблицах, выполнив следующую команду:

```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

При извлечении данных строка будет обновляться соответствующим образом.

```sql
SELECT * FROM inventory;
```

## <a name="restore-a-database-to-a-previous-point-in-time"></a>Восстановление базы данных до предыдущей точки во времени

Вы можете восстановить сервер до предыдущей точки во времени. Восстановленные данные копируются на новый сервер, а исходный сервер остается неизменным. Например, если таблица была случайно удалена, ее можно восстановить на момент сразу перед удалением. Затем вы можете извлечь отсутствующие таблицы и данные из восстановленной копии сервера.

Чтобы восстановить сервер, используйте командлет PowerShell `Restore-AzMariaDbServer`.

### <a name="run-the-restore-command"></a>Выполнение команды restore

Чтобы восстановить сервер, выполните следующий пример в PowerShell.

```azurepowershell-interactive
$restorePointInTime = (Get-Date).AddMinutes(-10)
Get-AzMariaDbServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Restore-AzMariaDbServer -Name mydemoserver-restored -ResourceGroupName myresourcegroup -RestorePointInTime $restorePointInTime -UsePointInTimeRestore
```

При восстановлении сервера до более ранней точки во времени будет создан новый сервер. Исходный сервер и его базы данных, начиная с указанного момента во времени, копируются на новый сервер.

Значения расположения и ценовой категории для восстановленного сервера совпадают со значениями исходного сервера.

После завершения восстановления найдите новый сервер, чтобы убедиться, что данные были восстановлены, как и ожидалось. Имя и пароль администратора сервера для входа на него будут теми же, что и для исходного сервера в момент, когда было запущен процесс восстановления. Пароль можно изменить на странице **Обзор** для нового сервера.

На новом сервере, созданном во время восстановления, нет конечных точек службы виртуальной сети, которые настроены на исходном сервере. Для нового сервера правила нужно настроить отдельно. Правила брандмауэра восстанавливаются с исходного сервера.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Как создать резервную копию сервера Базы данных Azure для MariaDB и восстановить сервер с помощью PowerShell](howto-restore-server-powershell.md)