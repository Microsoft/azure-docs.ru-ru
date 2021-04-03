---
title: Краткое руководство. Создание сервера Базы данных Azure для MySQL с помощью Azure PowerShell
description: В этом кратком руководстве показано, как создать сервер Базы данных Azure для MySQL в группе ресурсов Azure с помощью Azure PowerShell.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurepowershell
ms.topic: quickstart
ms.date: 04/28/2020
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: d12d447acb3b6bf2b6f84e9768e9f063a9a36b03
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94542326"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-using-powershell"></a>Краткое руководство. Создание сервера Базы данных Azure для MySQL с помощью Azure PowerShell

В этом кратком руководстве показано, как создать сервер Базы данных Azure для MySQL в группе ресурсов Azure с помощью Azure PowerShell. С помощью Azure PowerShell можно создавать и администрировать ресурсы Azure интерактивно или с помощью скриптов.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Так как модуль Az.MySql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az PowerShell с помощью следующей команды: `Install-Module -Name Az.MySql -AllowPrerelease`.
> Как только модуль PowerShell Az.MySql станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы впервые используете Базу данных Azure для MySQL, зарегистрируйте поставщик ресурсов **Microsoft.DBforMySQL**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.DBforMySQL
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

## <a name="create-an-azure-database-for-mysql-server"></a>Создайте сервер базы данных Azure для MySQL.

Создайте сервер Базы данных Azure для MySQL с помощью командлета `New-AzMySqlServer`. Сервер может управлять несколькими базами данных. Как правило, для каждого проекта и для каждого пользователя используется отдельная база данных.

В следующей таблице приводится список часто используемых параметров и примеры значений для командлета `New-AzMySqlServer`.

|        **Параметр**         | **Пример значения** |                                                                                                                                                             **Описание**                                                                                                                                                              |
| -------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Имя                       | mydemoserver     | Выберите глобально уникальное имя для сервера Базы данных Azure для MySQL. Имя сервера может содержать только буквы, цифры и дефис (-). Все символы в верхнем регистре при создании будут автоматически преобразованы в нижний регистр. Его длина должна составлять от 3 до 63 символов. |
| ResourceGroupName          | myresourcegroup  | Укажите имя группы ресурсов Azure.                                                                                                                                                                                                                                                                                            |
| Sku                        | GP_Gen5_2        | Имя номера SKU. Сокращенная запись соответствует соглашению: **ценовая-категория\_поколение-вычислительных-ресурсов\_число-виртуальных-ядер**. Под этой таблицей приведены дополнительные сведения о параметре SKU.                                                                                                                                           |
| BackupRetentionDay         | 7                | Срок хранения резервной копии. Указывается в днях. Можно указать от 7 до 35 дней.                                                                                                                                                                                                                                                                       |
| GeoRedundantBackup         | Активировано          | Позволяет включить или отключить создание геоизбыточных резервных копий для этого сервера. Это значение нельзя использовать для серверов ценовой категории "Базовый", а также изменять после создания сервера. Допустимые значения: Enabled, Disabled.                                                                                                      |
| Расположение                   | westus           | Регион Azure для сервера.                                                                                                                                                                                                                                                                                                         |
| SslEnforcement             | Активировано          | Позволяет включить или отключить SSL для этого сервера. Допустимые значения: Enabled, Disabled.                                                                                                                                                                                                                                                 |
| StorageInMb                | 51 200            | Объем хранилища сервера (в мегабайтах). StorageInMb имеет минимальное значение 5120 МБ, которое увеличивается на 1024 МБ. См. сведения об ограничениях размера хранилища в описании [ценовых категорий Базы данных Azure для MySQL](./concepts-pricing-tiers.md).                                                                               |
| Версия                    | 5.7              | Основной номер версии MySQL.                                                                                                                                                                                                                                                                                                                 |
| AdministratorUserName      | myadmin          | Имя для входа администратора. Не может иметь значение **azure_superuser**, **admin**, **administrator**, **root**, **guest** или **public**.                                                                                                                                                                                            |
| AdministratorLoginPassword | `<securestring>` | Пароль администратора в виде защищенной строки. Пароль должен содержать от 8 до 128 символов. Пароль должен содержать символы из таких трех категорий: прописные латинские буквы, строчные латинские буквы, цифры и небуквенно-цифровые знаки.                                       |

Значение параметра **SKU** соответствует формату **ценовая-категория\_поколение-вычислительных-ресурсов\_количество-виртуальных-ядер**, как показано в примерах ниже.

- `-Sku B_Gen5_1` — "Базовый", поколение 5, 1 виртуальное ядро; Это номер SKU наименьший по размеру из доступных.
- `-Sku GP_Gen5_32` — "Общего назначения", поколение 5, 32 виртуальных ядра;
- `-Sku MO_Gen5_2` — "Оптимизированная для операций в памяти", поколение 5, 2 виртуальных ядра.

См. сведения о допустимых значениях **SKU** по регионам и ценовым категориям в описании [ценовых категорий Базы данных Azure для MySQL](./concepts-pricing-tiers.md).

Следующий пример создает в регионе **Западная часть США** сервер MySQL с именем **mydemoserver** в группе ресурсов **myresourcegroup** с администратором сервера с именем **myadmin**. Это сервер 5-го поколения ценовой категории "Общего назначения" с двумя виртуальными ядрами и геоизбыточным резервным копированием. Запишите пароль, указанный в первой строке примера, так как это пароль для учетной записи администратора сервера MySQL.

> [!TIP]
> Имя сервера сопоставляется с DNS-именем и должно быть глобально уникальным в Azure.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -GeoRedundantBackup Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

Используйте ценовую категорию "Базовый", если для вашей рабочей нагрузки не требуется большое количество вычислительных ресурсов и операций ввода-вывода.

> [!IMPORTANT]
> Серверы, созданные в ценовой категории "Базовый", не удастся позже масштабировать до ценовых категорий "Общего назначения" или "Оптимизировано для памяти" и к ним нельзя применить георепликацию.

## <a name="configure-a-firewall-rule"></a>Настройка правила брандмауэра

Создайте правило брандмауэра для сервера Базы данных Azure для MySQL, используя командлет `New-AzMySqlFirewallRule`. Правило брандмауэра на уровне сервера позволяет внешним приложениям, таким как средство командной строки `mysql` или MySQL Workbench, подключаться к серверу через брандмауэр Базы данных Azure для MySQL.

В приведенном ниже примере создается правило брандмауэра с именем **AllowMyIP**, которое разрешает подключения с определенного IP-адреса (192.168.0.1). Замените его IP-адресом или диапазоном IP-адресов, которые применяются для того расположения, из которого вы подключаетесь.

```azurepowershell-interactive
New-AzMySqlFirewallRule -Name AllowMyIP -ResourceGroupName myresourcegroup -ServerName mydemoserver -StartIPAddress 192.168.0.1 -EndIPAddress 192.168.0.1
```

> [!NOTE]
> Подключитесь к базе данных Azure для MySQL через порт 3306. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 3306 может быть запрещен. В этом сценарии вы сможете подключиться к серверу, только если ИТ-отдел откроет для вас порт 3306.

## <a name="configure-ssl-settings"></a>Настройка параметров SSL

По умолчанию между сервером и клиентскими приложениями применяется SSL-соединение. Это обеспечивает защиту данных _при перемещении_ за счет шифрования потока данных через Интернет. В этом руководстве отключите SSL-соединения для вашего сервера. Дополнительные сведения см. в статье [Настройка SSL-подключений в приложении для безопасного подключения к базе данных Azure для MySQL](./howto-configure-ssl.md).

> [!WARNING]
> Мы не рекомендуем так делать для рабочих серверов.

Следующий пример отключает SSL на сервере Базы данных Azure для MySQL.

```azurepowershell-interactive
Update-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -SslEnforcement Disabled
```

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа. Чтобы получить сведения о подключении, используйте следующий пример. Запишите значения **FullyQualifiedDomainName** и **AdministratorLogin**.

```azurepowershell-interactive
Get-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Select-Object -Property FullyQualifiedDomainName, AdministratorLogin
```

```Output
FullyQualifiedDomainName                    AdministratorLogin
------------------------                    ------------------
mydemoserver.mysql.database.azure.com       myadmin
```

## <a name="connect-to-the-server-using-the-mysql-command-line-tool"></a>Подключение к серверу с помощью средства командной строки mysql

Подключитесь к серверу с помощью средства командной строки `mysql`. Скачать его можно [здесь](https://dev.mysql.com/downloads/shell/). Вы можете также получить доступ к предустановленной версии средства командной строки `mysql` из Azure Cloud Shell, нажав кнопку **Попробовать** в примере кода в этой статье. Azure Cloud Shell также можно открыть, нажав кнопку **>_** на панели инструментов вверху справа на портале Azure или перейдя по адресу [shell.azure.com](https://shell.azure.com/).

1. Подключитесь к серверу с помощью средства командной строки `mysql`.

   ```azurepowershell-interactive
   mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
   ```

1. Просмотрите состояние сервера.

   ```sql
   mysql> status
   ```

    ```Output
    C:\Users\>mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
    Enter password: *************
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 65512
    Server version: 5.6.42.0 MySQL Community Server (GPL)

    Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> status
    --------------
    mysql  Ver 14.14 Distrib 5.7.29, for Win64 (x86_64)

    Connection id:          65512
    Current database:
    Current user:           myadmin@myipaddress
    SSL:                    Not in use
    Using delimiter:        ;
    Server version:         5.6.42.0 MySQL Community Server (GPL)
    Protocol version:       10
    Connection:             mydemoserver.mysql.database.azure.com via TCP/IP
    Server characterset:    latin1
    Db     characterset:    latin1
    Client characterset:    utf8
    Conn.  characterset:    utf8
    TCP port:               3306
    Uptime:                 1 hour 2 min 12 sec

    Threads: 7  Questions: 952  Slow queries: 0  Opens: 66  Flush tables: 3  Open tables: 16  Queries per second avg: 0.255
    --------------

    mysql>
    ```

Дополнительные команды см. в [разделе 4.5.1 справочного руководства по MySQL 5.7](https://dev.mysql.com/doc/refman/5.7/en/mysql.html).

## <a name="connect-to-the-server-using-mysql-workbench"></a>Подключение к серверу с помощью MySQL Workbench

1. Запустите приложение MySQL Workbench на клиентском компьютере. Скачать MySQL Workbench для установки вы можете [отсюда](https://dev.mysql.com/downloads/workbench/).

1. В диалоговом окне **Настройка нового подключения** на вкладке **Параметры** введите следующие сведения:

   :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-powershell/setup-new-connection.png" alt-text="Настройка нового подключения":::

    |    **Параметр**    |           **Рекомендуемое значение**           |                      **Описание**                       |
    | ----------------- | --------------------------------------- | ---------------------------------------------------------- |
    | Имя подключения   | Мое подключение                           | Определение метки для этого подключения                        |
    | Способ подключения | Стандартный способ (по протоколу TCP/IP)                       | Применение протокола TCP/IP для подключения к Базе данных Azure для MySQL |
    | Имя узла          | `mydemoserver.mysql.database.azure.com` | Имя сервера, которое вы записали ранее                           |
    | Порт              | 3306                                    | Порт MySQL по умолчанию                                 |
    | Имя пользователя          | myadmin@mydemoserver                    | Имя для входа администратора сервера, которое вы записали ранее                |
    | Пароль          | *************                           | Используйте пароль учетной записи администратора, настроенный ранее      |

1. Щелкните **Проверить подключение**, чтобы проверить, все ли параметры настроены правильно.

1. Выберите подключение для соединения с сервером.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные в этом кратком руководстве ресурсы не нужны вам для другого руководства, их можно удалить с помощью команды из следующего примера.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этого краткого руководства, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myresourcegroup
```

Чтобы удалить только тот сервер, который вы создали с помощью этого краткого руководства, но сохранить группу ресурсов, используйте командлет `Remove-AzMySqlServer`.

```azurepowershell-interactive
Remove-AzMySqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Проектирование Базы данных Azure для MySQL с помощью PowerShell](tutorial-design-database-using-powershell.md)