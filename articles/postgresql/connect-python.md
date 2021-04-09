---
title: Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Python
description: В этом кратком руководстве представлены примеры кода Python, которые можно использовать для подключения к службе "База данных Azure для PostgreSQL — отдельный сервер" и запроса данных из нее.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc, devcenter, devx-track-python
ms.devlang: python
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: 6c589f3f0913609640ac14534c170c741c16a084
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105048176"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" и выполнение запроса данных с помощью Python

Из этого краткого руководства вы узнаете, как подключиться к базе данных в службе "База данных Azure для PostgreSQL — отдельный сервер" и выполнить инструкции SQL для создания запросов с помощью Python в macOS, Ubuntu Linux или Windows.

> [!TIP]
> Если вы хотите создать приложение Django с помощью PostgreSQL, изучите учебник [Развертывание веб-приложения Django с помощью PostgreSQL](../app-service/tutorial-python-postgresql-app.md).


## <a name="prerequisites"></a>Предварительные требования
Для целей этого краткого руководства понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free) бесплатно.
- Создать отдельный сервер Базы данных Azure для PostgreSQL на [портале Azure](./quickstart-create-server-database-portal.md) <br/> или с помощью [Azure CLI](./quickstart-create-server-database-azure-cli.md), если он еще не создан.
- Выполнить **ОДНО** из действий (в зависимости от того, пользуетесь вы общим или частным доступом), чтобы настроить возможность подключения.

  |Действие| Метод подключения|Практическое руководство|
  |:--------- |:--------- |:--------- |
  | **Настройка правил брандмауэра** | Общие | [Портал](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
  | **Настройка конечной точки службы** | Общие | [Портал](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)|
  | **Настройка приватного канала** | Private | [Портал](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) |

- [Python](https://www.python.org/downloads/) 2.7, 3.6 или более поздней версии.

- Установщик пакета [pip](https://pip.pypa.io/en/stable/installing/) последней версии.
- Установить [psycopg2](https://pypi.python.org/pypi/psycopg2/), используя `pip install psycopg2` в терминале или окне командной строки. Дополнительные сведения см. в статье об [установке`psycopg2`](http://initd.org/psycopg/docs/install.html).

## <a name="get-database-connection-information"></a>Получение сведений о подключении к базе данных
Чтобы подключиться к базе данных службы "База данных Azure для PostgreSQL", требуется полное имя сервера и учетные данные для входа. Эти сведения можно получить на портале Azure.

1. На [портале Azure](https://portal.azure.com/) выполните поиск по имени сервера и выберите сервер службы "База данных Azure для PostgreSQL".
1. На странице **Обзор** сервера скопируйте **полное имя сервера** и **имя администратора**. Полное **имя сервера** всегда имеет формат *\<my-server-name>.postgres.database.azure.com*, а **имя администратора** — формат *\<my-admin-username>@\<my-server-name>* .

   Кроме того, потребуется пароль администратора. Если вы не помните этот пароль, вы можете сбросить его на этой странице.

   :::image type="content" source="./media/connect-python/1-connection-string.png" alt-text="Имя сервера службы &quot;База данных Azure для PostgreSQL&quot;":::

> [!IMPORTANT]
>  Измените следующие значения:
>   - `<server-name>` и `<admin-username>` значениями, скопированными на портале Azure.
>   - `<admin-password>` паролем сервера.
>   - `<database-name>` база данных по умолчанию с именем *postgres* была автоматически создана при создании сервера. Можно переименовать эту базу данных или [создать новую базу данных](https://www.postgresql.org/docs/9.0/sql-createdatabase.html) с помощью команд SQL.

## <a name="step-1-connect-and-insert-data"></a>Шаг 1. Подключение и введение данных
Следующий пример кода подключается к Базе данных Azure для PostgreSQL с помощью
-  функции [psycopg2.connect](http://initd.org/psycopg/docs/connection.html) и загружает данные с помощью инструкции **SQL INSERT**.
- Функция [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) выполняет SQL-запрос к базе данных.

```Python
import psycopg2

# Update connection string information
host = "<server-name>"
dbname = "<database-name>"
user = "<admin-username>"
password = "<admin-password>"
sslmode = "require"

# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string)
print("Connection established")

cursor = conn.cursor()

# Drop previous table of same name if one exists
cursor.execute("DROP TABLE IF EXISTS inventory;")
print("Finished dropping table (if existed)")

# Create a table
cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
print("Finished creating table")

# Insert some data into the table
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
print("Inserted 3 rows of data")

# Clean up
conn.commit()
cursor.close()
conn.close()
```

При успешном выполнении кода возвращаются следующие данные:

:::image type="content" source="media/connect-python/2-example-python-output.png" alt-text="Вывод командной строки":::


[Возникли проблемы? Сообщите нам об этом](https://aka.ms/postgres-doc-feedback)

## <a name="step-2-read-data"></a>Шаг 2. Чтение данных
Следующий пример кода подключается к базе данных службы "База данных Azure для PostgreSQL" и использует
- [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) с инструкцией SQL **SELECT** для чтения данных.
- Функция [сursor.fetchall()](http://initd.org/psycopg/docs/cursor.html#cursor.fetchall) принимает запрос и возвращает результирующий набор для итерации с помощью

```Python

# Fetch all rows from table
cursor.execute("SELECT * FROM inventory;")
rows = cursor.fetchall()

# Print all rows
for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))


```
[Возникли проблемы? Сообщите нам об этом](https://aka.ms/postgres-doc-feedback)

## <a name="step-3-update-data"></a>Шаг 3. Обновление данных
В следующем примере кода для обновления данных используется [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) с инструкцией SQL **UPDATE**.

```Python

# Update a data row in the table
cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
print("Updated 1 row of data")

```
[Возникли проблемы? Сообщите нам об этом](https://aka.ms/postgres-doc-feedback)

## <a name="step-5-delete-data"></a>Шаг 5. Удаление данных
В следующем примере кода [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) выполняется с инструкцией SQL **DELETE**, чтобы удалить ранее вставленный элемент inventory.

```Python

# Delete data row from table
cursor.execute("DELETE FROM inventory WHERE name = %s;", ("orange",))
print("Deleted 1 row of data")

```

[Возникли проблемы? Сообщите нам об этом](https://aka.ms/postgres-doc-feedback)

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить все ресурсы, используемые во время этого краткого руководства, удалите группу ресурсов с помощью следующей команды:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для MySQL" через портал](./howto-create-manage-server-portal.md)<br/>

> [!div class="nextstepaction"]
> [Управление сервером службы "База данных Azure для MySQL" с помощью CLI](./how-to-manage-server-cli.md)<br/>

[Не можете найти нужную информацию? Сообщите нам!](https://aka.ms/postgres-doc-feedback)
