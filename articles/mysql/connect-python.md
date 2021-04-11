---
title: Краткое руководство. Подключение к Базе данных Azure для MySQL с помощью Python
description: В этом кратком руководстве представлены примеры кода Python, которые можно использовать для подключения к базе данных Azure для MySQL и запроса данных из нее.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom:
- mvc
- seo-python-october2019
- devx-track-python
ms.devlang: python
ms.topic: quickstart
ms.date: 10/28/2020
ms.openlocfilehash: a4391ecb7175b0e473b47cc3de43fd113795bc6b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104889031"
---
# <a name="quickstart-use-python-to-connect-and-query-data-in-azure-database-for-mysql"></a>Краткое руководство. Подключение к Базе данных Azure для MySQL и запрос данных с помощью Python

Из этого краткого руководства вы узнаете, как подключиться к Базе данных Azure для MySQL с использованием Python. Также вы узнаете, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных на платформах Windows, Mac и Ubuntu Linux. 

## <a name="prerequisites"></a>Предварительные требования
Для целей этого краткого руководства понадобится:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free) бесплатно.
- Создание отдельного сервера Базы данных Azure для MySQL с помощью [портала Azure](./quickstart-create-mysql-server-database-using-azure-portal.md) <br/> или с помощью [Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md), если он еще не создан.
- Выполнить **ОДНО** из действий (в зависимости от того, пользуетесь вы общим или частным доступом), чтобы настроить возможность подключения.

   |Действие| Метод подключения|Практическое руководство|
   |:--------- |:--------- |:--------- |
   | **Настройка правил брандмауэра** | Общие | [Портал](./howto-manage-firewall-using-portal.md) <br/> [CLI](./howto-manage-firewall-using-cli.md)|
   | **Настройка конечной точки службы** | Общие | [Портал](./howto-manage-vnet-using-portal.md) <br/> [CLI](./howto-manage-vnet-using-cli.md)| 
   | **Настройка приватного канала** | Private | [Портал](./howto-configure-privatelink-portal.md) <br/> [CLI](./howto-configure-privatelink-cli.md) | 

- [Создание базы данных и пользователя без прав администратора](./howto-create-users.md)

## <a name="install-python-and-the-mysql-connector"></a>Установка Python и соединителя MySQL

Установите Python и соединитель MySQL для Python на компьютере с помощью приведенных ниже шагов. 

> [!NOTE]
> В этом кратком руководстве используется [руководство для разработчика соединителя MySQL или Python](https://dev.mysql.com/doc/connector-python/en/).

1. Загрузите и установите [Python 3.7 или выше](https://www.python.org/downloads/) для вашей ОС. Не забудьте добавить Python в `PATH`, потому что это нужно для соединителя MySQL.
   
2. Откройте командную строку или оболочку `bash` и проверьте версию Python, запустив `python -V` с помощью коммутатора V прописными буквами.
   
3. Установщик пакетов `pip` включено в последние версии Python. Обновите `pip` до последней версии, запустив `pip install -U pip`. 
   
   Если `pip` не установлен, можно скачать и установить его с `get-pip.py`. Дополнительные сведения см. в разделе [Установка](https://pip.pypa.io/en/stable/installing/). 
   
4. Используйте `pip` чтоб установить соединитель MySQL для Python и его зависимости.
   
   ```bash
   pip install mysql-connector-python
   ```
   
[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="get-connection-information"></a>Получение сведений о подключении

Получите сведения, необходимые для подключения к базе данных Azure для MySQL с портала Azure. Вам потребуется имя сервера, имя базы данных и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).
   
1. На панели поиска портала найдите и выберите базу данных Azure для сервера MySQL, которую вы создали, например, **mydemoserver**.
   
   :::image type="content" source="./media/connect-python/1_server-overview-name-login.png" alt-text="Имя сервера базы данных Azure для MySQL":::
   
1. Запишите **имя сервера** и **имя для входа администратора сервера** со страницы сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой страницы.
   
   :::image type="content" source="./media/connect-python/azure-database-for-mysql-server-overview-name-login.png" alt-text="Имя сервера Базы данных Azure для MySQL (2)":::

## <a name="step-1-create-a-table-and-insert-data"></a>Шаг 1. Создание таблицы и вставка данных

Используйте следующий код для подключения к серверу и базе данных, создания таблицы и загрузки данных с помощью инструкции SQL **INSERT**. Код импортирует библиотеку mysql.connector и использует следующие методы:
- [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) используется для подключения Базы данных Azure для MySQL с помощью [аргументов](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html) из коллекции конфигураций; 
- [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) выполняет запрос SQL к базе данных MySQL; 
- [cursor.close()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-close.html) используется при завершении работы с курсором;
- [mysqli_close](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlconnection-close.html) используется, чтобы закрыть подключение.

> [!IMPORTANT]
> - Протокол SSL включен по умолчанию. Для подключения из локальной среды может потребоваться скачать [SSL-сертификат DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem).
> - В коде замените заполнители `<mydemoserver>`, `<myadmin>`, `<mypassword>` и `<mydatabase>` значениями для сервера MySQL и базы данных.

```python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'<mydemoserver>.mysql.database.azure.com',
  'user':'<myadmin>@<mydemoserver>',
  'password':'<mypassword>',
  'database':'<mydatabase>',
  'client_flags': [mysql.connector.ClientFlag.SSL],
  'ssl_ca': '/var/wwww/html/DigiCertGlobalRootG2.crt.pem'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Drop previous table of same name if one exists
  cursor.execute("DROP TABLE IF EXISTS inventory;")
  print("Finished dropping table (if existed).")

  # Create table
  cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
  print("Finished creating table.")

  # Insert some data into table
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
  print("Inserted",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="step-2-read-data"></a>Шаг 2. Чтение данных

Используйте указанный ниже код с инструкцией SQL **SELECT** для подключения и чтения данных. Код импортирует библиотеку mysql.connector и использует метод [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) для выполнения запроса SQL к базе данных MySQL. 

Код считывает строки данных с помощью метода [fetchall()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-fetchall.html), сохраняет набор результатов в строке коллекции и использует итератор `for` для циклического перемещения по строкам.

```python
  # Read data
  cursor.execute("SELECT * FROM inventory;")
  rows = cursor.fetchall()
  print("Read",cursor.rowcount,"row(s) of data.")

  # Print all rows
  for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))

```

## <a name="step-3-update-data"></a>Шаг 3. Обновление данных

Используйте указанный ниже код с инструкцией SQL **UPDATE** для подключения и обновления данных. Код импортирует библиотеку mysql.connector и использует метод [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) для выполнения запроса SQL к базе данных MySQL. 

```python
  # Update a data row in the table
  cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
  print("Updated",cursor.rowcount,"row(s) of data.")
```

## <a name="step-4-delete-data"></a>Шаг 4. Удаление данных

Используйте указанный ниже код с инструкцией SQL **DELETE** для подключения и удаления данных. Код импортирует библиотеку mysql.connector и использует метод [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) для выполнения запроса SQL к базе данных MySQL. 

```python

  # Delete a data row in the table
  cursor.execute("DELETE FROM inventory WHERE name=%(param1)s;", {'param1':"orange"})
  print("Deleted",cursor.rowcount,"row(s) of data.")
```

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
> [Управление сервером службы "База данных Azure для MySQL" с помощью CLI](./how-to-manage-single-server-cli.md)

[Не можете найти нужную информацию? Сообщите нам!](https://aka.ms/mysql-doc-feedback)
