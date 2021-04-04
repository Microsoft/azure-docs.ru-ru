---
title: Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" с помощью Go
description: В этом кратком руководстве представлен пример кода на языке программирования Go, который можно использовать для подключения к службе "База данных Azure для PostgreSQL — отдельный сервер" и запроса данных из нее.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.custom: mvc
ms.devlang: go
ms.topic: quickstart
ms.date: 5/6/2019
ms.openlocfilehash: eb844790ac0ae97c281f6d2d7022bee559f545cd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93331936"
---
# <a name="quickstart-use-go-language-to-connect-and-query-data-in-azure-database-for-postgresql---single-server"></a>Краткое руководство. Подключение к службе "База данных Azure для PostgreSQL — отдельный сервер" и выполнение запроса данных с помощью Go

В этом кратком руководстве описывается, как подключиться к базе данных Azure для PostgreSQL с помощью кода на языке [Go](https://golang.org/) (golang). Здесь также показано, как использовать инструкции SQL для запроса, вставки, обновления и удаления данных в базе данных. В этой статье предполагается, что у вас уже есть опыт разработки на Go, и вы только начали работу с базой данных Azure для PostgreSQL.

## <a name="prerequisites"></a>Предварительные требования
В качестве отправной точки в этом кратком руководстве используются ресурсы, созданные в соответствии со следующими материалами:
- [Создание базы данных — портал](quickstart-create-server-database-portal.md)
- [Создание базы данных с помощью Azure CLI](quickstart-create-server-database-azure-cli.md)

## <a name="install-go-and-pq-connector"></a>Установка Go и соединителя pq
Установите язык [Go](https://golang.org/doc/install) и [драйвер Pure Go Postgres](https://github.com/lib/pq) на своем компьютере. В зависимости от используемой платформы выполните соответствующие действия.

### <a name="windows"></a>Windows
1. [Скачайте](https://golang.org/dl/) и установите Go для Microsoft Windows [согласно инструкциям по установке](https://golang.org/doc/install).
2. Запустите командную строку из меню "Пуск".
3. Создайте папку для проекта, например `mkdir  %USERPROFILE%\go\src\postgresqlgo`.
4. Перейдите в папку проекта, например `cd %USERPROFILE%\go\src\postgresqlgo`.
5. В качестве значения для переменной среды GOPATH укажите путь к каталогу с исходным кодом. `set GOPATH=%USERPROFILE%\go`.
6. Установите [драйвер Pure Go Postgres (pq)](https://github.com/lib/pq) с помощью команды `go get github.com/lib/pq`.

   Установите Go, а затем выполните следующие команды в командной строке:
   ```cmd
   mkdir  %USERPROFILE%\go\src\postgresqlgo
   cd %USERPROFILE%\go\src\postgresqlgo
   set GOPATH=%USERPROFILE%\go
   go get github.com/lib/pq
   ```

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
1. Запустите оболочку Bash. 
2. Установите Go, выполнив команду `sudo apt-get install golang-go`.
3. Создайте папку для проекта в корневом каталоге, например `mkdir -p ~/go/src/postgresqlgo/`.
4. Перейдите в другую папку, например `cd ~/go/src/postgresqlgo/`.
5. В качестве значения для переменной среды GOPATH укажите путь к действительному исходному каталогу, например к текущей папке Go в корневом каталоге. В оболочке Bash выполните команду `export GOPATH=~/go`, чтобы добавить каталог Go в качестве значения переменной GOPATH для текущего сеанса оболочки.
6. Установите [драйвер Pure Go Postgres (pq)](https://github.com/lib/pq) с помощью команды `go get github.com/lib/pq`.

   Выполните следующие команды Bash:
   ```bash
   sudo apt-get install golang-go
   mkdir -p ~/go/src/postgresqlgo/
   cd ~/go/src/postgresqlgo/
   export GOPATH=~/go/
   go get github.com/lib/pq
   ```

### <a name="apple-macos"></a>Apple macOS
1. Скачайте и установите Go согласно [инструкциям по установке](https://golang.org/doc/install) с учетом вашей платформы. 
2. Запустите оболочку Bash. 
3. Создайте папку для проекта в корневом каталоге, например `mkdir -p ~/go/src/postgresqlgo/`.
4. Перейдите в другую папку, например `cd ~/go/src/postgresqlgo/`.
5. В качестве значения для переменной среды GOPATH укажите путь к действительному исходному каталогу, например к текущей папке Go в корневом каталоге. В оболочке Bash выполните команду `export GOPATH=~/go`, чтобы добавить каталог Go в качестве значения переменной GOPATH для текущего сеанса оболочки.
6. Установите [драйвер Pure Go Postgres (pq)](https://github.com/lib/pq) с помощью команды `go get github.com/lib/pq`.

   Установите Go, а затем выполните следующие команды Bash:
   ```bash
   mkdir -p ~/go/src/postgresqlgo/
   cd ~/go/src/postgresqlgo/
   export GOPATH=~/go/
   go get github.com/lib/pq
   ```

## <a name="get-connection-information"></a>Получение сведений о подключении
Получите сведения, необходимые для подключения к базе данных Azure.для PostgreSQL. Вам потребуется полное имя сервера и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В меню слева на портале Azure щелкните **Все ресурсы** и выполните поиск по имени созданного сервера (например, **mydemoserver**).
3. Щелкните имя сервера.
4. Запишите **имя сервера** и **имя для входа администратора сервера** с панели сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой панели.
 :::image type="content" source="./media/connect-go/1-connection-string.png" alt-text="Имя сервера службы &quot;База данных Azure для PostgreSQL&quot;":::

## <a name="build-and-run-go-code"></a>Сборка и выполнение кода Go 
1. Для написания кода на Golang можно использовать обычный текстовый редактор, например Блокнот в Microsoft Windows, [vi](https://manpages.ubuntu.com/manpages/xenial/man1/nvi.1.html#contenttoc5) или [Nano](https://www.nano-editor.org/) в Ubuntu, а также TextEdit в macOS. Если вы предпочитаете использовать полнофункциональную интегрированную среду разработки, попробуйте [GoLand](https://www.jetbrains.com/go/) от JetBrains, [Visual Studio Code](https://code.visualstudio.com/) от корпорации Майкрософт или [Atom](https://atom.io/).
2. Вставьте код Golang из приведенных ниже разделов в текстовые файлы и сохраните их с расширением \*.go в папке проекта, например `%USERPROFILE%\go\src\postgresqlgo\createtable.go` в Windows или `~/go/src/postgresqlgo/createtable.go` в Linux.
3. Найдите константы `HOST`, `DATABASE`, `USER` и `PASSWORD` в коде и замените приведенные для примера значения своими собственными.  
4. Запустите командную строку или оболочку Bash. Перейдите в папку проекта. В Windows это будет команда `cd %USERPROFILE%\go\src\postgresqlgo\`, а в Linux — `cd ~/go/src/postgresqlgo/`. Некоторые из упомянутых сред IDE позволяют выполнять отладку и использовать среду выполнения без применения команд оболочки.
5. Выполните код с помощью команды `go run createtable.go`, чтобы скомпилировать и запустить приложение. 
6. Вы также можете создать код в собственном приложении. Для этого введите `go build createtable.go`, а затем выполните `createtable.exe`, чтобы запустить приложение.

## <a name="connect-and-create-a-table"></a>Подключение и создание таблицы
Используйте приведенный ниже код для подключения и создайте таблицу с помощью инструкции SQL **CREATE TABLE**. Добавьте строки в таблицу, применив инструкцию SQL **INSERT INTO**.

Код импортирует три пакета: [пакет sql](https://golang.org/pkg/database/sql/), [пакет pq](https://godoc.org/github.com/lib/pq) как драйвер для обмена данными с сервером PostgreSQL и [пакет fmt](https://golang.org/pkg/fmt/) для вывода входных и выходных данных в командной строке.

Код вызывает метод [sql.Open()](https://godoc.org/github.com/lib/pq#Open) для подключения к базе данных Azure для PostgreSQL, а затем проверяет подключение с помощью метода [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). [Дескриптор базы данных](https://golang.org/pkg/database/sql/#DB) используется для всех компонентов. Он содержит пул подключений к серверу базы данных. Этот код несколько раз вызывает метод [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) для выполнения нескольких команд SQL. При каждом запуске пользовательский метод checkError() проверяет наличие ошибок и инициирует аварийный выход в случае их обнаружения.

Замените значения параметров `HOST`, `DATABASE`, `USER` и `PASSWORD` своими значениями. 

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    // Initialize connection string.
    var connectionString string = fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Drop previous table of same name if one exists.
    _, err = db.Exec("DROP TABLE IF EXISTS inventory;")
    checkError(err)
    fmt.Println("Finished dropping table (if existed)")

    // Create table.
    _, err = db.Exec("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
    checkError(err)
    fmt.Println("Finished creating table")

    // Insert some data into table.
    sql_statement := "INSERT INTO inventory (name, quantity) VALUES ($1, $2);"
    _, err = db.Exec(sql_statement, "banana", 150)
    checkError(err)
    _, err = db.Exec(sql_statement, "orange", 154)
    checkError(err)
    _, err = db.Exec(sql_statement, "apple", 100)
    checkError(err)
    fmt.Println("Inserted 3 rows of data")
}
```

## <a name="read-data"></a>Чтение данных
Используйте указанный ниже код с инструкцией SQL **SELECT** для подключения и чтения данных. 

Код импортирует три пакета: [пакет sql](https://golang.org/pkg/database/sql/), [пакет pq](https://godoc.org/github.com/lib/pq) как драйвер для обмена данными с сервером PostgreSQL и [пакет fmt](https://golang.org/pkg/fmt/) для вывода входных и выходных данных в командной строке.

Код вызывает метод [sql.Open()](https://godoc.org/github.com/lib/pq#Open) для подключения к базе данных Azure для PostgreSQL, а затем проверяет подключение с помощью метода [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). [Дескриптор базы данных](https://golang.org/pkg/database/sql/#DB) используется для всех компонентов. Он содержит пул подключений к серверу базы данных. Запрос выбора выполняется с помощью метода [db.Query()](https://golang.org/pkg/database/sql/#DB.Query), а итоговые строки сохраняются в переменной типа [rows](https://golang.org/pkg/database/sql/#Rows). Код считывает из столбцов значения данных в текущей строке с помощью метода [rows.Scan()](https://golang.org/pkg/database/sql/#Rows.Scan), а затем запускает цикл по строкам, используя итератор [сrows.Next()](https://golang.org/pkg/database/sql/#Rows.Next), пока строк больше не останется. Значения столбцов для каждой строки выводятся на консоль. При каждом запуске пользовательский метод checkError() используется, чтобы проверить наличие ошибок и инициировать аварийный выход в случае их обнаружения.

Замените значения параметров `HOST`, `DATABASE`, `USER` и `PASSWORD` своими значениями. 

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString string = fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Read rows from table.
    var id int
    var name string
    var quantity int

    sql_statement := "SELECT * from inventory;"
    rows, err := db.Query(sql_statement)
    checkError(err)
    defer rows.Close()

    for rows.Next() {
        switch err := rows.Scan(&id, &name, &quantity); err {
        case sql.ErrNoRows:
            fmt.Println("No rows were returned")
        case nil:
            fmt.Printf("Data row = (%d, %s, %d)\n", id, name, quantity)
        default:
            checkError(err)
        }
    }
}
```

## <a name="update-data"></a>Обновление данных
Используйте указанный ниже код с инструкцией SQL **UPDATE** для подключения и обновления данных.

Код импортирует три пакета: [пакет sql](https://golang.org/pkg/database/sql/), [пакет pq](https://godoc.org/github.com/lib/pq) как драйвер для обмена данными с сервером Postgres и [пакет fmt](https://golang.org/pkg/fmt/) для вывода входных и выходных данных в командной строке.

Код вызывает метод [sql.Open()](https://godoc.org/github.com/lib/pq#Open) для подключения к базе данных Azure для PostgreSQL, а затем проверяет подключение с помощью метода [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). [Дескриптор базы данных](https://golang.org/pkg/database/sql/#DB) используется для всех компонентов. Он содержит пул подключений к серверу базы данных. Этот код вызывает метод [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) для выполнения инструкции SQL, которая обновляет таблицу. Пользовательский метод checkError() используется, чтобы проверить наличие ошибок и инициировать аварийный выход в случае их обнаружения.

Замените значения параметров `HOST`, `DATABASE`, `USER` и `PASSWORD` своими значениями. 
```go
package main

import (
  "database/sql"
  _ "github.com/lib/pq"
  "fmt"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    
    // Initialize connection string.
    var connectionString string = 
        fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Modify some data in table.
    sql_statement := "UPDATE inventory SET quantity = $2 WHERE name = $1;"
    _, err = db.Exec(sql_statement, "banana", 200)
    checkError(err)
    fmt.Println("Updated 1 row of data")
}
```

## <a name="delete-data"></a>Удаление данных
Используйте указанный ниже код с инструкцией SQL **DELETE** для подключения и удаления данных. 

Код импортирует три пакета: [пакет sql](https://golang.org/pkg/database/sql/), [пакет pq](https://godoc.org/github.com/lib/pq) как драйвер для обмена данными с сервером Postgres и [пакет fmt](https://golang.org/pkg/fmt/) для вывода входных и выходных данных в командной строке.

Код вызывает метод [sql.Open()](https://godoc.org/github.com/lib/pq#Open) для подключения к базе данных Azure для PostgreSQL, а затем проверяет подключение с помощью метода [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping). [Дескриптор базы данных](https://golang.org/pkg/database/sql/#DB) используется для всех компонентов. Он содержит пул подключений к серверу базы данных. Этот код вызывает метод [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) для выполнения инструкции SQL, которая удаляет строку из таблицы. Пользовательский метод checkError() используется, чтобы проверить наличие ошибок и инициировать аварийный выход в случае их обнаружения.

Замените значения параметров `HOST`, `DATABASE`, `USER` и `PASSWORD` своими значениями. 
```go
package main

import (
  "database/sql"
  _ "github.com/lib/pq"
  "fmt"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    
    // Initialize connection string.
    var connectionString string = 
        fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Delete some data from table.
    sql_statement := "DELETE FROM inventory WHERE name = $1;"
    _, err = db.Exec(sql_statement, "orange")
    checkError(err)
    fmt.Println("Deleted 1 row of data")
}
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
> [Перенос базы данных с помощью экспорта и импорта](./howto-migrate-using-export-and-import.md)
