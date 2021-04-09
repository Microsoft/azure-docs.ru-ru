---
title: Создание функции Go или Rust с помощью Visual Studio Code — Функции Azure
description: Узнайте, как создать функцию Go в виде пользовательского обработчика Функций Azure, а затем опубликовать локальный проект в бессерверном размещении в Функциях Azure с помощью расширения Функций Azure в Visual Studio Code.
ms.topic: quickstart
ms.date: 12/4/2020
ms.openlocfilehash: 8b53031315cce3651a2de581d71da6ef6fe909c1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103470376"
---
# <a name="quickstart-create-a-go-or-rust-function-in-azure-using-visual-studio-code"></a>Краткое руководство. Создание функции Go или Rust в Azure с помощью Visual Studio Code

[!INCLUDE [functions-language-selector-quickstart-vs-code](../../includes/functions-language-selector-quickstart-vs-code.md)]

Из этой статьи вы узнаете, как с помощью Visual Studio Code создать [пользовательский обработчик](functions-custom-handlers.md), который отвечает на HTTP-запросы. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.

Пользовательские обработчики можно использовать для создания функций на любом языке и в любой среде выполнения, выполняя серверный процесс HTTP. В этой статье описываются процедуры для [Go](create-first-function-vs-code-other.md?tabs=go) и [Rust](create-first-function-vs-code-other.md?tabs=rust).

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

## <a name="configure-your-environment"></a>Настройка среды

Перед началом работы убедитесь, что выполнены следующие предварительные требования.

# <a name="go"></a>[GO](#tab/go)

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Visual Studio Code](https://code.visualstudio.com/) на одной из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

+ [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code.

+ [Azure Functions Core Tools](./functions-run-local.md#v2) версии 3.x. С помощью команды `func --version` убедитесь, что они установлены правильно.

+ [Go](https://golang.org/doc/install), рекомендуется использовать последнюю версию. Используйте команду `go version`, чтобы проверить установленную версию.

# <a name="rust"></a>[Rust](#tab/rust)

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

+ [Visual Studio Code](https://code.visualstudio.com/) на одной из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

+ [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code.

+ [Azure Functions Core Tools](./functions-run-local.md#v2) версии 3.x. С помощью команды `func --version` убедитесь, что они установлены правильно.

+ Цепочка инструментов Rust и использование [rustup](https://www.rust-lang.org/tools/install). Используйте команду `rustc --version`, чтобы проверить установленную версию.

---

## <a name="create-your-local-project"></a><a name="create-an-azure-functions-project"></a>Создание локального проекта

В этом разделе показано, как с помощью Visual Studio Code создать локальный проект Функций Azure для пользовательских обработчиков. Далее в этой статье вы опубликуете код функции в Azure.

1. Щелкните значок Azure на панели действий, а затем в области **Azure: Functions** (Azure: Функции) щелкните значок **Создать проект...** .

    ![Выбор варианта "Создать проект"](./media/functions-create-first-function-vs-code/create-new-project.png)

1. Выберите расположение для рабочей области проекта и нажмите кнопку **Выбрать**.

    > [!NOTE]
    > Рассматриваемые в этой статье шаги выполняются вне рабочей области. В этом случае не нужно указывать папку проекта, которая является частью рабочей области.

1. Введите следующие сведения по соответствующим запросам:

    + **Выберите язык для проекта приложения-функции**: Выберите `Custom`.

    + **Выберите шаблон для первой функции вашего проекта**. Выберите `HTTP trigger`.

    + **Укажите имя функции**. Введите `HttpExample`.

    + **Уровень авторизации**: выберите `Anonymous`, что позволит любому пользователю вызывать конечную точку функции. Дополнительные сведения об уровне авторизации см. в разделе [Authorization keys](functions-bindings-http-webhook-trigger.md#authorization-keys) (Ключи авторизации).

    + **Выберите, как вы хотели бы открыть свой проект**. Выберите `Add to workspace`.

1. Используя эти сведения, Visual Studio Code создает проект функций Azure с функцией для триггера HTTP. Файлы локального проекта можно просмотреть в Explorer. Дополнительные сведения см. в разделе [Generated project files](functions-develop-vs-code.md#generated-project-files) (Созданные файлы проекта). 

## <a name="create-and-build-your-function"></a>Создание и сборка функции

Файл *function.json* в папке *HttpExample* объявляет функцию для триггера HTTP. Чтобы реализовать эту функцию, вам нужно добавить к ней обработчик и скомпилировать исполняемый файл.

# <a name="go"></a>[GO](#tab/go)

1. Нажмите клавиши <kbd>CTRL+N</kbd> (или <kbd>CMD+N</kbd> в macOS), чтобы создать новый файл. Сохраните его с именем *handler.go* в корневой папке приложения-функции (где расположен файл *host.json*).

1. Добавьте в файл *handler.go* следующий код и сохраните его. Это и есть ваш пользовательский обработчик Go.

    ```go
    package main
    
    import (
        "fmt"
        "log"
        "net/http"
        "os"
    )
    
    func helloHandler(w http.ResponseWriter, r *http.Request) {
        message := "This HTTP triggered function executed successfully. Pass a name in the query string for a personalized response.\n"
        name := r.URL.Query().Get("name")
        if name != "" {
            message = fmt.Sprintf("Hello, %s. This HTTP triggered function executed successfully.\n", name)
        }
        fmt.Fprint(w, message)
    }
    
    func main() {
        listenAddr := ":8080"
        if val, ok := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT"); ok {
            listenAddr = ":" + val
        }
        http.HandleFunc("/api/HttpExample", helloHandler)
        log.Printf("About to listen on %s. Go to https://127.0.0.1%s/", listenAddr, listenAddr)
        log.Fatal(http.ListenAndServe(listenAddr, nil))
    }
    ```

1. Нажмите клавиши <kbd>CTRL+SHIFT+'</kbd> или выберите команду *New Terminal* (Создать терминал) в меню *Terminal* (Терминал), чтобы открыть новый встроенный терминал VS Code.

1. Скомпилируйте этот пользовательский обработчик с помощью следующей команды. Исполняемый файл с именем `handler` (`handler.exe` в Windows) выводится в корневую папку приложения-функции.

    ```bash
    go build handler.go
    ```

    ![VS Code — создание пользовательского обработчика Go](./media/functions-create-first-function-vs-code/functions-vscode-go.png)

# <a name="rust"></a>[Rust](#tab/rust)

1. Нажмите клавиши <kbd>CTRL+SHIFT+'</kbd> или выберите команду *New Terminal* (Создать терминал) в меню *Terminal* (Терминал), чтобы открыть новый встроенный терминал VS Code.

1. В корневой папке приложения-функции (где размещен файл *host.json*) инициализируйте проект Rust с именем `handler`.

    ```bash
    cargo init --name handler
    ```

1. В файл *Cargo.toml* добавьте следующие зависимости, требуемые для работы с этим кратким руководством. В нашем примере используется платформа веб-сервера [warp](https://docs.rs/warp/).

    ```toml
    [dependencies]
    warp = "0.3"
    tokio = { version = "1", features = ["rt", "macros", "rt-multi-thread"] }
    ```

1. Добавьте в файл *src/main.rs* следующий код и сохраните его. Это и есть ваш пользовательский обработчик Rust.

    ```rust
    use std::collections::HashMap;
    use std::env;
    use std::net::Ipv4Addr;
    use warp::{http::Response, Filter};

    #[tokio::main]
    async fn main() {
        let example1 = warp::get()
            .and(warp::path("api"))
            .and(warp::path("HttpExample"))
            .and(warp::query::<HashMap<String, String>>())
            .map(|p: HashMap<String, String>| match p.get("name") {
                Some(name) => Response::builder().body(format!("Hello, {}. This HTTP triggered function executed successfully.", name)),
                None => Response::builder().body(String::from("This HTTP triggered function executed successfully. Pass a name in the query string for a personalized response.")),
            });

        let port_key = "FUNCTIONS_CUSTOMHANDLER_PORT";
        let port: u16 = match env::var(port_key) {
            Ok(val) => val.parse().expect("Custom Handler port is not a number!"),
            Err(_) => 3000,
        };

        warp::serve(example1).run((Ipv4Addr::UNSPECIFIED, port)).await
    }
    ```

1. Скомпилируйте двоичный файл для этого обработчика. Исполняемый файл с именем `handler` (`handler.exe` в Windows) выводится в корневую папку приложения-функции.

    ```bash
    cargo build --release
    cp target/release/handler .
    ```

    ![VS Code — создание пользовательского обработчика Rust](./media/functions-create-first-function-vs-code/functions-vscode-rust.png)

---

## <a name="configure-your-function-app"></a>Настройка приложения-функции

Узел функции должен быть настроен так, чтобы при запуске выполнять двоичный файл пользовательского обработчика.

1. Откройте файл *host.json*.

1. В разделе `customHandler.description` задайте значение `defaultExecutablePath` для параметра `handler` (или `handler.exe` в среде Windows).

1. В разделе `customHandler` добавьте свойство с именем `enableForwardingHttpRequest` и задайте для него значение `true`. Для функций, которые содержат только триггер HTTP, этот параметр упрощает программирование. Он позволяет работать с обычным HTTP-запросом вместо [полезных данных](functions-custom-handlers.md#request-payload) пользовательского обработчика запросов.

1. Убедитесь, что раздел `customHandler` похож на представленный здесь пример. Сохраните файл.

    ```
    "customHandler": {
      "description": {
        "defaultExecutablePath": "handler",
        "workingDirectory": "",
        "arguments": []
      },
      "enableForwardingHttpRequest": true
    }
    ```

Теперь приложение-функция будет запускать исполняемый файл пользовательского обработчика.

## <a name="run-the-function-locally"></a>Локальное выполнение функции

Этот проект можно запустить на локальном компьютере разработки, прежде чем публиковать его в Azure.

1. В интегрированном терминале запустите приложение-функцию с помощью Azure Functions Core Tools.

    ```bash
    func start
    ```

1. Запустив Core Tools, перейдите по указанному далее URL-адресу, чтобы выполнить запрос GET, который включает строку запроса `?name=Functions`.

    `http://localhost:7071/api/HttpExample?name=Functions`

1. Возвращается ответ, который в браузере выглядит следующим образом:

    ![Браузер: пример выходных данных localhost](./media/create-first-function-vs-code-other/functions-test-local-browser.png)

1. Сведения о запросе отображаются на панели **Терминал**.

    ![Запуск узла задач: выходные данные терминала VS Code](../../includes/media/functions-run-function-test-local-vs-code/function-execution-terminal.png)

1. Чтобы остановить работу Core Tools, нажмите клавиши <kbd>CTRL+C</kbd>.

Убедившись, что функция выполняется правильно на локальном компьютере, опубликуйте проект в Azure с помощью Visual Studio Code.

[!INCLUDE [functions-sign-in-vs-code](../../includes/functions-sign-in-vs-code.md)]

## <a name="compile-the-custom-handler-for-azure"></a>Компиляция пользовательского обработчика для Azure

В этом разделе показано, как опубликовать проект в Azure в виде приложения-функции под управлением Linux. В большинстве случаев перед публикацией в Azure вам придется заново компилировать двоичный файл и настраивать конфигурацию для целевой платформы.

# <a name="go"></a>[GO](#tab/go)

1. Во встроенном терминале скомпилируйте обработчик для Linux/x64. В корневой папке приложения-функции будет создан двоичный файл с именем `handler`.

    # <a name="macos"></a>[macOS](#tab/macos)

    ```bash
    GOOS=linux GOARCH=amd64 go build handler.go
    ```

    # <a name="linux"></a>[Linux](#tab/linux)

    ```bash
    GOOS=linux GOARCH=amd64 go build handler.go
    ```

    # <a name="windows"></a>[Windows](#tab/windows)
    ```cmd
    set GOOS=linux
    set GOARCH=amd64
    go build handler.go
    ```

    Для параметра `defaultExecutablePath` в файле *host.json* измените значение `handler.exe` на `handler`. Теперь приложение-функция будет выполнять двоичный файл Linux.
    
    ---

# <a name="rust"></a>[Rust](#tab/rust)

1. Создайте файл по адресу *.cargo/config*. Добавьте в этот файл следующее содержимое и сохраните файл.

    ```
    [target.x86_64-unknown-linux-musl]
    linker = "rust-lld"
    ```

1. Во встроенном терминале скомпилируйте обработчик для Linux/x64. Будет создан двоичный файл с именем `handler`. Скопируйте его в корневую папку приложения-функции.

    ```bash
    rustup target add x86_64-unknown-linux-musl
    cargo build --release --target=x86_64-unknown-linux-musl
    cp target/x86_64-unknown-linux-musl/release/handler .
    ```

1. Если вы используете ОС Windows, для параметра `defaultExecutablePath` в файле *host.json* измените значение с `handler.exe` на `handler`. Теперь приложение-функция будет выполнять двоичный файл Linux.

1. Добавьте следующую строку в файл *.funcignore*.

    ```
    target
    ```

    Это предотвращает публикацию содержимого папки *target*.

---

## <a name="publish-the-project-to-azure"></a>Публикация проекта в Azure

В этом разделе показано создание приложения-функции и сопутствующих ресурсов в подписке Azure и последующее развертывание кода. 

> [!IMPORTANT]
> Публикация в существующее приложение-функцию перезаписывает содержимое этого приложения в Azure. 


1. Щелкните значок Azure на панели действий, а затем в области **Azure: Функции** выберите кнопку **Deploy to function app...** (Развертывание в приложение-функцию).

    ![Публикация проекта в Azure](../../includes/media/functions-publish-project-vscode/function-app-publish-project.png)

1. Введите следующие сведения по соответствующим запросам:

    + **Выбор папки**. Выберите папку из рабочей области или перейдите к папке, содержащей приложение-функцию. Этот элемент не отобразится, если вы уже открыли допустимое приложение-функцию.

    + **Выбрать подписку**. Выберите подписку, которую нужно использовать. Если у вас только одна подписка, вы не увидите этот параметр.

    + **Select Function App in Azure** (Выбор приложения-функции в Azure). Выберите `+ Create new Function App (advanced)`. 
    
        > [!IMPORTANT]
        > Параметр `advanced` позволяет выбрать операционную систему, в которой приложение-функция выполняется в Azure (в нашем примере это Linux).

        ![VS Code — выбор расширенных параметров создания приложения-функции](./media/functions-create-first-function-vs-code/functions-vscode-create-azure-advanced.png)

    + **Enter a globally unique name for the function app** (Ввод глобально уникального имени для приложения-функции). Введите имя, допустимое в пути URL-адреса. Имя, которое вы вводите, проверяется, чтобы убедиться, что оно уникально в функциях Azure.

    + **Select a runtime stack** (Выберите стек сред выполнения). Выберите `Custom Handler`.

    + **Select an OS** (Выберите ОС). Выберите `Linux`.

    + **Select a hosting plan** (Выберите план размещения). Выберите `Consumption`.

    + **Select a resource group** (Выберите группу ресурсов). Выберите `+ Create new resource group`. Введите имя группы ресурсов. Это имя должно быть уникальным в пределах подписки Azure. Можно использовать имя, предложенное в командной строке.

    + **Select a storage account** (Выберите учетную запись хранения). Выберите `+ Create new storage account`. Это имя должно быть глобально уникальным в Azure. Можно использовать имя, предложенное в командной строке.

    + **Select an Application Insights resource** (Выберите ресурс Application Insights). Выберите `+ Create Application Insights resource`. Это имя должно быть глобально уникальным в Azure. Можно использовать имя, предложенное в командной строке.

    + **Select a location for new resources** (Выбор расположения для новых ресурсов).  Для оптимальной производительности выберите [регион](https://azure.microsoft.com/regions/) рядом с вами. Расширение показывает в области уведомлений состояние отдельных ресурсов по мере их создания в Azure.

    :::image type="content" source="../../includes/media/functions-publish-project-vscode/resource-notification.png" alt-text="Уведомление о создании ресурса Azure":::

1. После завершения в Azure в вашей подписке создаются следующие ресурсы:

    [!INCLUDE [functions-vs-code-created-resources](../../includes/functions-vs-code-created-resources.md)]

    После создания приложения-функции и применения пакета развертывания отобразится уведомление. 

4. Выберите **View Output** (Просмотреть выходные данные) в уведомлении, чтобы просмотреть результаты создания и развертывания ресурсов Azure. Если вы пропустили уведомление, щелкните значок колокольчика в правом нижнем углу, чтобы снова просмотреть его.

    ![Создание уведомления о завершении](./media/functions-create-first-function-vs-code/function-create-notifications.png)

[!INCLUDE [functions-vs-code-run-remote](../../includes/functions-vs-code-run-remote.md)]

[!INCLUDE [functions-cleanup-resources-vs-code.md](../../includes/functions-cleanup-resources-vs-code.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сведения о пользовательских обработчиках в Функциях Azure](functions-custom-handlers.md)
