---
title: Краткое руководство. Подключение к Базе данных Azure для MySQL с помощью Node.js
description: В этом кратком руководстве представлены примеры кода Node.js, которые можно использовать для подключения к базе данных Azure для MySQL и запроса данных из нее.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-js
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 12/11/2020
ms.openlocfilehash: 6a9134e13e3145daea1eed81c4aa8795a0a49950
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97588239"
---
# <a name="quickstart-use-nodejs-to-connect-and-query-data-in-azure-database-for-mysql"></a>Краткое руководство. Подключение и запрос данных с помощью Node.js в Базе данных Azure для MySQL

Из этого краткого руководства вы узнаете, как подключиться к Базе данных Azure для MySQL с использованием Node.js. Также вы узнаете, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных на платформах Windows, Mac и Ubuntu Linux. 

В этой статье предполагается, что у вас уже есть опыт разработки на Node.js и вы только начали работу со службой "База данных Azure для MySQL".

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- Сервер Базы данных Azure для MySQL. Создайте сервер Базы данных Azure для MySQL с помощью [портала Azure](quickstart-create-mysql-server-database-using-azure-portal.md) или [Azure CLI](quickstart-create-mysql-server-database-using-azure-cli.md).

> [!IMPORTANT] 
> Убедитесь, что IP-адрес, с которого вы подключаетесь, добавлен в правила брандмауэра на сервере через [портал Azure](./howto-manage-firewall-using-portal.md) или [Azure CLI](./howto-manage-firewall-using-cli.md).

## <a name="install-nodejs-and-the-mysql-connector"></a>Установка Node.js и соединителя MySQL

В зависимости от используемой платформы выполните инструкции из соответствующего раздела, чтобы установить [Node.js](https://nodejs.org). Используйте npm, чтобы установить пакет [mysql](https://www.npmjs.com/package/mysql) и его зависимости в папку проекта.

### <a name="windows"></a>Windows

1. Войдите на [страницу скачиваемых файлов Node.js](https://nodejs.org/en/download/) и выберите нужный установщик Windows.
2. Создайте папку локального проекта, например `nodejsmysql`. 
3. Откройте командную строку и перейдите в папку проекта, например в `cd c:\nodejsmysql\`.
4. Запустите средство NPM, чтобы установить библиотеку mysql в папку проекта.

   ```cmd
   cd c:\nodejsmysql\
   "C:\Program Files\nodejs\npm" install mysql
   "C:\Program Files\nodejs\npm" list
   ```

5. Проверьте установку, просмотрев текст вывода `npm list`. Номера версии могут отличаться, когда будут выпущены новые обновления.

### <a name="linux-ubuntu"></a>Linux (Ubuntu)

1. Чтобы установить **Node.js** и **NPM** (диспетчер пакетов для Node.js), выполните следующую команду:

   ```bash
    # Using Ubuntu
    curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
    sudo apt-get install -y nodejs

    # Using Debian, as root
    curl -sL https://deb.nodesource.com/setup_14.x | bash -
    apt-get install -y nodejs
   ```

2. Выполните следующие команды, чтобы создать папку проекта `mysqlnodejs` и установить в эту папку пакет mysql.

   ```bash
   mkdir nodejsmysql
   cd nodejsmysql
   npm install --save mysql
   npm list
   ```
3. Проверьте установку, просмотрев текст вывода npm list. Номера версии могут отличаться, когда будут выпущены новые обновления.

### <a name="macos"></a>macOS

1. Войдите на [страницу скачиваемых файлов Node.js](https://nodejs.org/en/download/) и выберите установщик для macOS.

2. Выполните следующие команды, чтобы создать папку проекта `mysqlnodejs` и установить в эту папку пакет mysql.

   ```bash
   mkdir nodejsmysql
   cd nodejsmysql
   npm install --save mysql
   npm list
   ```

3. Проверьте установку, просмотрев текст вывода `npm list`. Номера версии могут отличаться, когда будут выпущены новые обновления.

## <a name="get-connection-information"></a>Получение сведений о подключении

Получите сведения о подключении, необходимые для подключения к базе данных Azure.для MySQL. Вам потребуется полное имя сервера и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В меню слева на портале Azure выберите **Все ресурсы** и выполните поиск по имени созданного сервера (например, **mydemoserver**).
3. Выберите имя сервера.
4. Запишите **имя сервера** и **имя для входа администратора сервера** с панели сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой панели.
 :::image type="content" source="./media/connect-nodejs/server-name-azure-database-mysql.png" alt-text="Имя сервера базы данных Azure для MySQL":::

## <a name="running-the-javascript-code-in-nodejs"></a>Выполнение кода JavaScript в Node.js

1. Вставьте код JavaScript в текстовые файлы и сохраните их в папку проекта с расширением файла JS, например C:\nodejsmysql\createtable.js или /home/username/nodejsmysql/createtable.js.
2. Откройте командную строку или оболочку Bash и перейдите в папку проекта `cd nodejsmysql`.
3. Чтобы запустить приложение, введите команду Node, указав после нее имя файла, например `node createtable.js`.
4. В Windows, если приложение Node не указано в переменной среды PATH, может потребоваться указать полный путь, чтобы запустить приложение Node, например `"C:\Program Files\nodejs\node.exe" createtable.js`.

## <a name="connect-create-table-and-insert-data"></a>Подключение, создание таблицы и вставка данных

Используйте приведенный ниже код для подключения и загрузки данных с помощью инструкций SQL **CREATE TABLE** и **INSERT INTO**.

Метод [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) используется для обмена данными с сервером MySQL. Функция [connect()](https://github.com/mysqljs/mysql#establishing-connections) используется для подключения к серверу. Функция [query()](https://github.com/mysqljs/mysql#performing-queries) используется для выполнения SQL-запроса к базе данных MySQL. 

Замените параметры `host`, `user`, `password` и `database` значениями, указанными при создании сервера и базы данных.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
    if (err) { 
        console.log("!!! Cannot connect !!! Error:");
        throw err;
    }
    else
    {
       console.log("Connection established.");
           queryDatabase();
    }
});

function queryDatabase(){
    conn.query('DROP TABLE IF EXISTS inventory;', function (err, results, fields) { 
        if (err) throw err; 
        console.log('Dropped inventory table if existed.');
    })
        conn.query('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);', 
            function (err, results, fields) {
                if (err) throw err;
        console.log('Created inventory table.');
    })
    conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['banana', 150], 
            function (err, results, fields) {
                if (err) throw err;
        else console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
    conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['orange', 154], 
            function (err, results, fields) {
                if (err) throw err;
        console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
    conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['apple', 100], 
    function (err, results, fields) {
                if (err) throw err;
        console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
    conn.end(function (err) { 
    if (err) throw err;
    else  console.log('Done.') 
    });
};
```

## <a name="read-data"></a>Чтение данных

Используйте указанный ниже код с инструкцией SQL **SELECT** для подключения и чтения данных. 

Метод [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) используется для обмена данными с сервером MySQL. Метод [connect()](https://github.com/mysqljs/mysql#establishing-connections) используется для подключения к серверу. Метод [query()](https://github.com/mysqljs/mysql#performing-queries) используется для выполнения SQL-запроса к базе данных MySQL. Массив результатов используется для хранения результатов запроса.

Замените параметры `host`, `user`, `password` и `database` значениями, указанными при создании сервера и базы данных.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            readData();
        }
    });

function readData(){
    conn.query('SELECT * FROM inventory', 
        function (err, results, fields) {
            if (err) throw err;
            else console.log('Selected ' + results.length + ' row(s).');
            for (i = 0; i < results.length; i++) {
                console.log('Row: ' + JSON.stringify(results[i]));
            }
            console.log('Done.');
        })
    conn.end(
        function (err) { 
            if (err) throw err;
            else  console.log('Closing connection.') 
    });
};
```

## <a name="update-data"></a>Обновление данных

Используйте указанный ниже код с инструкцией SQL **UPDATE** для подключения и чтения данных. 

Метод [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) используется для обмена данными с сервером MySQL. Метод [connect()](https://github.com/mysqljs/mysql#establishing-connections) используется для подключения к серверу. Метод [query()](https://github.com/mysqljs/mysql#performing-queries) используется для выполнения SQL-запроса к базе данных MySQL. 

Замените параметры `host`, `user`, `password` и `database` значениями, указанными при создании сервера и базы данных.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            updateData();
        }
    });

function updateData(){
       conn.query('UPDATE inventory SET quantity = ? WHERE name = ?', [200, 'banana'], 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Updated ' + results.affectedRows + ' row(s).');
           })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Done.') 
        });
};
```

## <a name="delete-data"></a>Удаление данных

Используйте следующий код с инструкцией SQL **DELETE** для подключения и чтения данных. 

Метод [mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) используется для обмена данными с сервером MySQL. Метод [connect()](https://github.com/mysqljs/mysql#establishing-connections) используется для подключения к серверу. Метод [query()](https://github.com/mysqljs/mysql#performing-queries) используется для выполнения SQL-запроса к базе данных MySQL. 

Замените параметры `host`, `user`, `password` и `database` значениями, указанными при создании сервера и базы данных.

```javascript
const mysql = require('mysql');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            deleteData();
        }
    });

function deleteData(){
       conn.query('DELETE FROM inventory WHERE name = ?', ['orange'], 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Deleted ' + results.affectedRows + ' row(s).');
           })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Done.') 
        });
};
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
> [Перенос базы данных с помощью экспорта и импорта](./concepts-migrate-import-export.md)
