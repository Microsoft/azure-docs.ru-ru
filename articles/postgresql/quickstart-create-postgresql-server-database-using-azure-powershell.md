---
title: Краткое руководство. Создание базы данных в службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Azure PowerShell
description: Краткое руководство по созданию базы данных в службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Azure PowerShell.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.devlang: azurepowershell
ms.topic: quickstart
ms.date: 06/08/2020
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 79a070dbd49ed6ee00dd790c8157057aebde30df
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105606745"
---
# <a name="quickstart-create-an-azure-database-for-postgresql---single-server-using-powershell"></a>Краткое руководство. Создание базы данных в службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Azure PowerShell

В этом кратком руководстве показано, как создать сервер Базы данных Azure для PostgreSQL в группе ресурсов Azure с помощью Azure PowerShell. С помощью Azure PowerShell можно создавать и администрировать ресурсы Azure интерактивно или с помощью скриптов.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Так как модуль Az.PostgreSql PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно от модуля Az с помощью команды `Install-Module -Name Az.PostgreSql -AllowPrerelease`.
> Как только модуль Az.PostgreSql PowerShell станет общедоступным, он будет включен в один из будущих выпусков Az PowerShell и встроен в Azure Cloud Shell.

Если вы впервые используете службу "База данных Azure для PostgreSQL", зарегистрируйте поставщик ресурсов **Microsoft.DBforPostgreSQL**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.DBforPostgreSQL
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

## <a name="create-an-azure-database-for-postgresql-server"></a>Создание сервера Базы данных Azure для PostgreSQL

Создайте сервер Базы данных Azure для PostgreSQL с помощью командлета `New-AzPostgreSqlServer`. Сервер может управлять несколькими базами данных. Как правило, для каждого проекта и для каждого пользователя используется отдельная база данных.

В следующей таблице приводится список часто используемых параметров и примеры значений для командлета `New-AzPostgreSqlServer`.

|        **Параметр**         | **Пример значения** |                                                                                                                                                             **Описание**                                                                                                                                                              |
| -------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Имя                       | mydemoserver     | Выберите глобально уникальное в Azure имя для сервера Базы данных Azure для PostgreSQL. Имя сервера может содержать только буквы, цифры и дефис (-). Все символы в верхнем регистре при создании будут автоматически преобразованы в нижний регистр. Его длина должна составлять от 3 до 63 символов. |
| ResourceGroupName          | myresourcegroup  | Укажите имя группы ресурсов Azure.                                                                                                                                                                                                                                                                                            |
| Sku                        | GP_Gen5_2        | Имя номера SKU. Сокращенная запись соответствует соглашению: **ценовая-категория\_поколение-вычислительных-ресурсов\_число-виртуальных-ядер**. Под этой таблицей приведены дополнительные сведения о параметре SKU.                                                                                                                                           |
| BackupRetentionDay         | 7                | Срок хранения резервной копии. Указывается в днях. Можно указать от 7 до 35 дней.                                                                                                                                                                                                                                                                       |
| GeoRedundantBackup         | Активировано          | Позволяет включить или отключить создание геоизбыточных резервных копий для этого сервера. Это значение нельзя использовать для серверов ценовой категории "Базовый", а также изменять после создания сервера. Допустимые значения: Enabled, Disabled.                                                                                                      |
| Расположение                   | westus           | Регион Azure для сервера.                                                                                                                                                                                                                                                                                                         |
| SslEnforcement             | Активировано          | Позволяет включить или отключить SSL для этого сервера. Допустимые значения: Enabled, Disabled.                                                                                                                                                                                                                                                 |
| StorageInMb                | 51 200            | Объем хранилища сервера (в мегабайтах). StorageInMb имеет минимальное значение 5120 МБ, которое увеличивается на 1024 МБ. См. сведения об ограничениях размера хранилища в описании [ценовых категорий Базы данных Azure для PostgreSQL](./concepts-pricing-tiers.md).                                                                               |
| Версия                    | 9,6              | Основной номер версии PostgreSQL.                                                                                                                                                                                                                                                                                                                 |
| AdministratorUserName      | myadmin          | Имя для входа администратора. Не может иметь значение **azure_superuser**, **admin**, **administrator**, **root**, **guest** или **public**.                                                                                                                                                                                            |
| AdministratorLoginPassword | `<securestring>` | Пароль администратора в виде защищенной строки. Пароль должен содержать от 8 до 128 символов. Пароль должен содержать символы из таких трех категорий: прописные латинские буквы, строчные латинские буквы, цифры и небуквенно-цифровые знаки.                                       |

Значение параметра **SKU** соответствует формату **ценовая-категория\_поколение-вычислительных-ресурсов\_количество-виртуальных-ядер**, как показано в примерах ниже.

- `-Sku B_Gen5_1` — "Базовый", поколение 5, 1 виртуальное ядро; Это номер SKU наименьший по размеру из доступных.
- `-Sku GP_Gen5_32` — "Общего назначения", поколение 5, 32 виртуальных ядра;
- `-Sku MO_Gen5_2` — "Оптимизированная для операций в памяти", поколение 5, 2 виртуальных ядра.

См. сведения о допустимых значениях **SKU** по регионам и ценовым категориям в описании [ценовых категорий Базы данных Azure для PostgreSQL](./concepts-pricing-tiers.md).

В следующем примере в регионе **Западная часть США** создается сервер PostgreSQL с именем **mydemoserver** в группе ресурсов **myresourcegroup** с администратором сервера с именем **myadmin**. Это сервер 5-го поколения ценовой категории "Общего назначения" с двумя виртуальными ядрами и геоизбыточным резервным копированием. Запишите пароль, указанный в первой строке примера, так как это пароль для учетной записи администратора сервера PostgreSQL.

> [!TIP]
> Имя сервера сопоставляется с DNS-именем и должно быть глобально уникальным в Azure.

```azurepowershell-interactive
$Password = Read-Host -Prompt 'Please enter your password' -AsSecureString
New-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup -Sku GP_Gen5_2 -GeoRedundantBackup Enabled -Location westus -AdministratorUsername myadmin -AdministratorLoginPassword $Password
```

Используйте ценовую категорию "Базовый", если для вашей рабочей нагрузки не требуется большое количество вычислительных ресурсов и операций ввода-вывода.

> [!IMPORTANT]
> Серверы, созданные в ценовой категории "Базовый", не удастся позже масштабировать до ценовых категорий "Общего назначения" или "Оптимизировано для памяти" и к ним нельзя применить георепликацию.

## <a name="configure-a-firewall-rule"></a>Настройка правила брандмауэра

Создайте правило брандмауэра для сервера Базы данных Azure для PostgreSQL, используя командлет `New-AzPostgreSqlFirewallRule`. Правило брандмауэра на уровне сервера позволяет внешним приложениям, таким как средство командной строки `psql` или PostgreSQL Workbench, подключаться к серверу через брандмауэр Базы данных Azure для PostgreSQL.

В приведенном ниже примере создается правило брандмауэра с именем **AllowMyIP**, которое разрешает подключения с определенного IP-адреса (192.168.0.1). Замените его IP-адресом или диапазоном IP-адресов, которые применяются для того расположения, из которого вы подключаетесь.

```azurepowershell-interactive
New-AzPostgreSqlFirewallRule -Name AllowMyIP -ResourceGroupName myresourcegroup -ServerName mydemoserver -StartIPAddress 192.168.0.1 -EndIPAddress 192.168.0.1
```

> [!NOTE]
> Подключения к Базе данных Azure для PostgreSQL используют порт 5432. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 5432 может быть запрещен. В таком случае вы сможете подключиться к серверу, только если ИТ-отдел откроет для вас порт 5432.

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа. Чтобы получить сведения о подключении, используйте следующий пример. Запишите значения **FullyQualifiedDomainName** и **AdministratorLogin**.

```azurepowershell-interactive
Get-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup |
  Select-Object -Property FullyQualifiedDomainName, AdministratorLogin
```

```Output
FullyQualifiedDomainName                    AdministratorLogin
------------------------                    ------------------
mydemoserver.postgres.database.azure.com       myadmin
```

## <a name="connect-to-postgresql-database-using-psql"></a>Подключение к базе данных PostgreSQL с помощью psql

Если на клиентском компьютере установлено PostgreSQL, вы можете использовать локальный экземпляр [psql](https://www.postgresql.org/docs/current/static/app-psql.html), чтобы подключиться к серверу Azure PostgreSQL. Вы можете также получить доступ к предустановленной версии средства командной строки `psql` из Azure Cloud Shell, нажав кнопку **Попробовать** в примере кода в этой статье. Azure Cloud Shell также можно открыть, нажав кнопку **>_** на панели инструментов вверху справа на портале Azure или перейдя по адресу [shell.azure.com](https://shell.azure.com/).

1. Подключитесь к серверу Azure PostgreSQL с помощью служебной программы командной строки `psql`.

   ```azurepowershell-interactive
   psql --host=<servername> --port=<port> --username=<user@servername> --dbname=<dbname>
   ```

   Например, следующая команда устанавливает подключение к базе данных по умолчанию **postgres** на сервере PostgreSQL `mydemoserver.postgres.database.azure.com`, используя учетные данные для доступа. Введите `<server_admin_password>`, указанный при появлении запроса на ввод пароля.

   ```azurepowershell-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   > [!TIP]
   > Если вы предпочитаете использовать URL-путь для подключения к Postgres, закодируйте с помощью URL-адреса знак @ в имени пользователя с использованием `%40`. Например, строка подключения для psql будет выглядеть так: `psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres`.

1. Подключившись к серверу, создайте пустую базу данных с помощью командной строки.

   ```sql
   CREATE DATABASE mypgsqldb;
   ```

1. Чтобы подключиться к созданной базе данных **mypgsqldb**, выполните следующую команду в командной строке:

   ```sql
   \c mypgsqldb
   ```

## <a name="connect-to-the-postgresql-server-using-pgadmin"></a>Подключение к серверу PostgreSQL с помощью pgAdmin

pgAdmin — это средство с открытым кодом, которое используется с PostgreSQL. Средство PgAdmin можно установить с [веб-сайта pgAdmin](https://www.pgadmin.org/). Ваша версия pgAdmin может отличаться от используемой в этом кратком руководстве. Если вам требуются дополнительные инструкции, ознакомьтесь с документацией по pgAdmin.

1. Откройте приложение pgAdmin на клиентском компьютере.

1. На панели инструментов выберите **Object** (Объект), наведите указатель мыши на пункт **Create** (Создать) и выберите **Server** (Сервер).

1. На вкладке **General** (Общее) в диалоговом окне **Create — Server** (Создание сервера) введите уникальное понятное имя сервера, например **mydemoserver**.

   :::image type="content" source="./media/quickstart-create-postgresql-server-database-using-azure-powershell/9-pgadmin-create-server.png" alt-text="Вкладка &quot;Общие&quot;":::

1. На вкладке **Connection** (Подключение) в диалоговом окне **Create — Server** (Создание сервера) заполните таблицу настроек.

   :::image type="content" source="./media/quickstart-create-postgresql-server-database-using-azure-powershell/10-pgadmin-create-server.png" alt-text="Вкладка &quot;Подключение&quot;":::

    параметр pgAdmin |Значение|Описание
    ---|---|---
    Имя узла и адрес | Имя сервера | Значение имени сервера, которое вы использовали раньше при создании базы данных Azure для сервера PostgreSQL. В нашем примере используется имя сервера **mydemoserver.postgres.database.azure.com**. Используйте полное доменное имя ( **\*.postgres.database.azure.com**), как показано в примере. Если вы не помните имя своего сервера, выполните действия из предыдущего раздела, чтобы получить сведения о подключении.
    Порт | 5432 | Порт, используемый при подключении к базе данных Azure для сервера PostgreSQL.
    База данных обслуживания | *postgres* | Имя базы данных по умолчанию, созданное системой.
    Имя пользователя | Имя для входа администратора сервера | Имя для входа администратора сервера, которое вы использовали раньше при создании базы данных Azure для сервера PostgreSQL. Если вы не помните имя пользователя, выполните действия из предыдущего раздела, чтобы получить сведения о подключении. Формат *имя пользователя\@имя сервера*.
    Пароль | Ваш пароль администратора | Пароль, выбранный при создании сервера во время работы с этим руководством.
    Роль | Не указывайте | Указывать роль на этом шаге не нужно. Оставьте поле пустым.
    Режим SSL | *Require* (Требовать) | На вкладке pgAdmin SSL можно настроить режим TLS или SSL. По умолчанию все серверы службы "База данных Azure для PostgreSQL" создаются с включенным применением TLS. Чтобы отключить применение TLS, см. раздел [Настройка принудительного применения TLS](./concepts-ssl-connection-security.md#configure-enforcement-of-tls).

1. Щелкните **Сохранить**.

1. В области **Браузер** слева разверните узел **Серверы**. Выберите свой сервер, например **mydemoserver**. Щелкните, чтобы подключиться к нему.

1. Разверните узел сервера, а затем разверните раздел **Базы данных** под ним. Список должен включать существующую базу данных *postgres* и еще одну базу данных, созданную вами. С помощью службы "База данных Azure для PostgreSQL" для сервера можно создать несколько баз данных.

1. Щелкните правой кнопкой мыши **Базы данных**, выберите меню **Создать**, а затем щелкните **База данных**.

1. Введите выбранное имя базы данных в поле **База данных**, например **mypgsqldb2**.

1. Выберите **владельца** базы данных из списка. Выберите имя администратора сервера для входа (например, **my admin**).

   :::image type="content" source="./media/quickstart-create-postgresql-server-database-using-azure-powershell/11-pgadmin-database.png" alt-text="Создание базы данных в pgAdmin":::

1. выберите **Сохранить**, чтобы создать пустую базу данных.

1. Созданная база данных отобразится в области **Browser** (Обозреватель) в списке баз данных под именем сервера.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные в этом кратком руководстве ресурсы не нужны вам для другого руководства, их можно удалить с помощью команды из следующего примера.

> [!CAUTION]
> Следующий пример удаляет указанную группу ресурсов и все содержащиеся в ней ресурсы.
> Если в указанной группе ресурсов существуют другие ресурсы, кроме созданных для этого краткого руководства, они также будут удалены.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myresourcegroup
```

Чтобы удалить только тот сервер, который вы создали с помощью этого краткого руководства, но сохранить группу ресурсов, используйте командлет `Remove-AzPostgreSqlServer`.

```azurepowershell-interactive
Remove-AzPostgreSqlServer -Name mydemoserver -ResourceGroupName myresourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Разработка Базы данных Azure для PostgreSQL с помощью PowerShell](tutorial-design-database-using-powershell.md)