---
title: Управление сервером Azure CLI. база данных Azure для PostgreSQL — гибкий сервер
description: Узнайте, как управлять базой данных Azure для PostgreSQL-гибкого сервера из Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 278f8f816909a7e365d7e45d04c5169950e79a65
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96493684"
---
# <a name="manage-an-azure-database-for-postgresql---flexible-server-by-using-the-azure-cli"></a>Управление базой данных Azure для PostgreSQL-гибкого сервера с помощью Azure CLI

> [!IMPORTANT]
> База данных Azure для PostgreSQL — гибкий сервер в предварительной версии.

В этой статье показано, как управлять гибким сервером, развернутым в Azure. Задачи управления включают в себя вычисление и масштабирование хранилища, сброс пароля администратора и Просмотр сведений о сервере.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу. 

Необходимо запустить Azure CLI версии 2,0 или более поздней локально. Чтобы узнать, какая установлена версия, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

Войдите в свою учетную запись с помощью команды [AZ login](/cli/azure/reference-index#az-login) . 

```azurecli-interactive
az login
```

Выберите подписку с помощью команды [AZ Account Set](/cli/azure/account) . Запишите значение **идентификатора** из выходных данных **AZ login** , чтобы использовать его в качестве значения аргумента **Subscription** в следующей команде. Если у вас несколько подписок, выберите подписку, к которой должен быть выставлен счет за ресурс. Чтобы найти все подписки, используйте команду [AZ Account List](/cli/azure/account#az-account-list) .

```azurecli
az account set --subscription <subscription id>
```

> [!Important]
> Если вы еще не создали гибкий сервер, вам потребуется сделать это для выполнения этого руководства.

## <a name="scale-compute-and-storage"></a>Масштабирование вычислений и хранилища

Вы можете легко масштабировать свой уровень вычислений, виртуальных ядер и хранилище с помощью следующей команды. Список всех операций сервера, которые можно запустить, см. в обзоре команды [AZ postgres гибкого сервера](/cli/azure/postgres/flexible-server) .

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --sku-name Standard_D4ds_v3 --storage-size 6144
```

Ниже приведены сведения о аргументах в приведенном выше коде.

**Параметр** | **Пример значения** | **Описание**
---|---|---
name | mydemoserver | Введите уникальное имя для сервера. Имя сервера может содержать только строчные буквы, цифры и знак дефиса (-). Длина должна составлять от 3 до 63 символов.
resource-group | myresourcegroup | Укажите имя группы ресурсов Azure.
sku-name|Standard_D4ds_v3|Введите имя уровня и размера вычислений. Значение соответствует соглашению *Standard_ {размер виртуальной машины}* в сокращенном виде. Дополнительные сведения см. на странице с [ценовыми категориями](../concepts-pricing-tiers.md).
storage-size | 6144 | Введите емкость хранилища сервера в мегабайтах. Минимальное значение — 5120, увеличивающееся с шагом в 1024.

> [!IMPORTANT]
> Нельзя масштабировать хранилище. 

## <a name="manage-postgresql-databases-on-a-server"></a>Управление базами данных PostgreSQL на сервере

К базе данных Azure для сервера PostgreSQL можно подключиться с помощью нескольких приложений. Если на клиентском компьютере установлен PostgreSQL, можно использовать локальный экземпляр [psql](https://www.postgresql.org/docs/current/static/app-psql.html). Теперь выполним использование программы командной строки psql для подключения к серверу базы данных Azure для PostgreSQL.

1. Выполните следующую команду **psql** :

   ```bash
   psql --host=<servername> --port=<port> --username=<user> --dbname=<dbname>
   ```

   Например, следующая команда подключается к базе данных по умолчанию с именем **postgres** на сервере PostgreSQL **mydemoserver.postgres.Database.Azure.com** с помощью учетных данных доступа. При появлении запроса введите выбранный вами элемент `<server_admin_password>` .
  
   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin --dbname=postgres
   ```

   После подключения средство psql выведет запрос **postgres** , где можно ввести команды SQL. Если используемая версия psql отличается от версии на сервере базы данных Azure для PostgreSQL, в первоначальном выходных данных подключения появится предупреждение.

   Пример выходных данных psql:

   ```bash
   psql (11.3, server 12.1)
   WARNING: psql major version 11, server major version 12.
            Some psql features might not work.
   SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
   Type "help" for help.

   postgres=>
   ```

   > [!TIP]
   > Если в брандмауэре не настроено разрешение IP-адресов клиента, возникает следующая ошибка:
   >
   > "psql: FATAL: no pg_hba.conf entry for host `<IP address>`, user "myadmin", database "postgres", SSL on FATAL: необходимо SSL-подключение". Укажите параметры SSL и повторите попытку. "
   >
   > Убедитесь, что IP-адрес клиента разрешен в правилах брандмауэра.

2. Создайте пустую базу данных с именем **постгресдб** , введя следующую команду в командной строке:

    ```bash
    CREATE DATABASE postgresdb;
    ```

3. В командной строке выполните следующую команду, чтобы переключить подключения к только что созданной базе данных **постгресдб**:

    ```bash
    \c postgresdb
    ```

4. Введите  `\q` и нажмите клавишу ВВОД, чтобы выйти из psql.

В этом разделе вы подключились к серверу базы данных Azure для PostgreSQL с помощью psql и создали пустую пользовательскую базу данных.

## <a name="reset-the-admin-password"></a>Сброс пароля администратора

Пароль роли администратора можно изменить с помощью следующей команды:

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!IMPORTANT]
> Выберите пароль длиной не менее 8 символов и не более 128 символов. Пароль должен содержать символы трех из следующих категорий: 
> - прописные латинские буквы;
> - строчные латинские буквы;
> - Числа
> - Символы, отличные от буквенно-цифровых

## <a name="delete-a-server"></a>Удаление сервера

Чтобы удалить гибкий сервер базы данных Azure для PostgreSQL, выполните команду [AZ postgres гибкий-Server DELETE](/cli/azure/postgres/flexible-server#az-PostgreSQL-flexible-server-delete) .

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Дальнейшие действия

- [Основные сведения о резервном копировании и восстановлении](concepts-backup-restore.md)
- [Настройка и мониторинг сервера](concepts-monitoring.md)