---
title: Краткое руководство. Подключение к Базе данных Azure для PostgreSQL (Гибкий сервер) с помощью Python
description: В этом кратком руководстве представлено несколько примеров кода Python, которые можно использовать для подключения к Базе данных Azure для PostgreSQL (Гибкий сервер) и запроса данных из нее.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.custom: mvc
ms.devlang: python
ms.topic: quickstart
ms.date: 09/22/2020
ms.openlocfilehash: 823dd6fc64da2c4900f5a0a35a6cd27a9a2b43ff
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227490"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-postgresql---flexible-server"></a>Краткое руководство. Подключение к Базе данных Azure для PostgreSQL (Гибкий сервер) и запрашивание данных с помощью Python

> [!IMPORTANT]
> Гибкий сервер Базы данных Azure для PostgreSQL предоставляется в режиме предварительной версии

Из этого краткого руководства вы узнаете, как подключиться к Базе данных Azure для PostgreSQL (Гибкий сервер) с использованием Python. Также вы узнаете, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных на платформах Windows, Mac и Ubuntu Linux. 

В этой статье предполагается, что у вас уже есть опыт разработки на языке Python, но вы только начали работу с Базой данных Azure для PostgreSQL (Гибкий сервер).

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
* База данных Azure для PostgreSQL (Гибкий сервер). Чтобы создать гибкий сервер, воспользуйтесь статьей [Разработка базы данных в службе "База данных Azure для PostgreSQL (Гибкий сервер)" с помощью портала Azure](./quickstart-create-server-portal.md).
* [Python](https://www.python.org/downloads/) 2.7, 3.6 или более поздней версии.
* Установщик пакета [pip](https://pip.pypa.io/en/stable/installing/) последней версии.

## <a name="preparing-your-client-workstation"></a>Подготовка клиентской рабочей станции
- Если вы создали гибкий сервер в режиме *Закрытый доступ (интеграция с виртуальной сетью)* , к этому серверу придется подключаться с другого ресурса в той же виртуальной сети. Например, можно создать виртуальную машину и добавить ее в виртуальную сеть, созданную для гибкого сервера. Дополнительные сведения см. в статье [Создание виртуальной сети для Базы данных Azure для PostgreSQL (Гибкий сервер) и управление ею с помощью Azure CLI](./how-to-manage-virtual-network-cli.md).
- Если вы создали гибкий сервер в режиме *Открытый доступ (разрешенные IP-адреса)* , вы можете добавить локальный IP-адрес в список правил брандмауэра на этом сервере. Дополнительные сведения см. в статье [Создание правил брандмауэра для Базы данных Azure для PostgreSQL (Гибкий сервер) и управление ими с помощью Azure CLI](./how-to-manage-firewall-cli.md).

## <a name="install-the-python-libraries-for-postgresql"></a>Установка библиотек Python для PostgreSQL
Модуль [psycopg2](https://pypi.python.org/pypi/psycopg2/) позволяет подключаться к базе данных PostgreSQL и выполнять запросы к ней. Он доступен в виде пакета [Wheel](https://pythonwheels.com/) для Linux, macOS или Windows. Установите двоичную версию модуля, включая все зависимости. Дополнительные сведения об установке `psycopg2` и о предварительных требованиях см. на [этой странице](http://initd.org/psycopg/docs/install.html). 

Чтобы установить `psycopg2`, откройте окно терминала или командную строку и выполните команду `pip install psycopg2`.

## <a name="get-database-connection-information"></a>Получение сведений о подключении к базе данных
Чтобы подключиться к Базе данных Azure для PostgreSQL (Гибкий сервер), требуется полное имя сервера и учетные данные для входа. Эти сведения можно получить на портале Azure.

1. На [портале Azure](https://portal.azure.com/) найдите и выберите имя гибкого сервера. 
2. На странице **Обзор** сервера скопируйте **полное имя сервера** и **имя администратора**. Полное **имя сервера** всегда имеет формат *\<my-server-name>.postgres.database.azure.com*.

   Кроме того, потребуется пароль администратора. Если вы не помните этот пароль, вы можете сбросить его на странице обзора. 

   <!--![Azure Database for PostgreSQL server name](./media/connect-python/1-connection-string.png)-->

## <a name="how-to-run-the-python-examples"></a>Выполнение примеров кода Python

Для каждого примера кода в этой статье сделайте следующее:

1. Создайте файл в текстовом редакторе. 

1. Добавьте пример кода в файл. В коде замените:
   - `<server-name>` и `<admin-username>` значениями, скопированными на портале Azure.
   - `<admin-password>` паролем сервера.
   - `<database-name>` именем базы данных службы "База данных Azure для PostgreSQL (Гибкий сервер)". При создании сервера автоматически создается база данных по умолчанию с именем *postgres*. Вы можете переименовать эту базу данных или создать другую с помощью команд SQL. 

1. Сохраните файл в папке проекта с расширением *PY*, например *postgres-insert.py*. При сохранении файла в ОС Windows обязательно выберите кодировку UTF-8. 

1. Чтобы запустить этот файл, перейдите в папку проекта в интерфейсе командной строки и введите `python`, а затем — имя файла, например `python postgres-insert.py`.

## <a name="create-a-table-and-insert-data"></a>Создание таблицы и вставка данных
В приведенном ниже примере кода устанавливается подключение к базе данных в службе "База данных Azure для PostgreSQL (Гибкий сервер)" с помощью функции [psycopg2.connect](http://initd.org/psycopg/docs/connection.html) и загружаются данные с помощью инструкции SQL **INSERT**. Функция [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) выполняет SQL-запрос к базе данных. 

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

![Вывод командной строки](media/connect-python/2-example-python-output.png)

## <a name="read-data"></a>Чтение данных
В приведенном ниже примере кода устанавливается подключение к базе данных в службе "База данных Azure для PostgreSQL (Гибкий сервер)" и данные считываются с помощью функции [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) и инструкции SQL **SELECT**. Эта функция принимает запрос и возвращает результирующий набор для итерации с помощью [cursor.fetchall()](http://initd.org/psycopg/docs/cursor.html#cursor.fetchall). 

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
# Fetch all rows from table
cursor.execute("SELECT * FROM inventory;")
rows = cursor.fetchall()
# Print all rows
for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))
# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="update-data"></a>Обновление данных
В приведенном ниже примере кода устанавливается подключение к базе данных в службе "База данных Azure для PostgreSQL (Гибкий сервер)" и данные обновляются с помощью функции [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) и инструкции SQL **UPDATE**. 

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
# Update a data row in the table
cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
print("Updated 1 row of data")
# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="delete-data"></a>Удаление данных
В приведенном ниже примере кода устанавливается подключение к базе данных в службе "База данных Azure для PostgreSQL (Гибкий сервер)" и удаляется вставленный ранее элемент с помощью функции [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) и инструкции SQL **DELETE**. 

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
# Delete data row from table
cursor.execute("DELETE FROM inventory WHERE name = %s;", ("orange",))
print("Deleted 1 row of data")
# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="next-steps"></a>Следующие шаги
> [!div class="nextstepaction"]
> [Перенос базы данных с помощью метода "создание дампа и восстановление"](../howto-migrate-using-dump-and-restore.md)
