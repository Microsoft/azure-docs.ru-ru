---
title: Использование Кэша Azure для Redis с приложениями Go
description: Из этого краткого руководства вы узнаете, как создать приложение Go, в котором используется Кэш Azure для Redis.
author: abhirockzz
ms.author: abhishgu
ms.service: cache
ms.devlang: go
ms.topic: quickstart
ms.date: 01/08/2021
ms.openlocfilehash: 04b582b5ef31e61039c5513ea2a4aa60f1c638e7
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102121343"
---
# <a name="quickstart-use-azure-cache-for-redis-with-go"></a>Краткое руководство. Использование Кэша Azure для Redis с приложениями Go

Из этой статьи вы узнаете, как создать в Go REST API, который будет хранить и получать сведения о пользователях с помощью структуры данных [HASH](https://redis.io/topics/data-types-intro#redis-hashes) в [Кэше Azure для Redis](./cache-overview.md). 

## <a name="skip-to-the-code-on-github"></a>Переход к коду на GitHub

Если вы хотите сразу перейти к коду, см. [краткое руководство по Go](https://github.com/Azure-Samples/azure-redis-cache-go-quickstart/) на сайте GitHub.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- [Go](https://golang.org/doc/install) (предпочтительная версия — 1.13 или более поздние версии);
- [Git](https://git-scm.com/downloads)
- HTTP-клиент, например [curl](https://curl.se/);

## <a name="create-an-azure-cache-for-redis-instance"></a>Создание экземпляра кэша Azure для Redis
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

## <a name="review-the-code-optional"></a>Просмотр кода (необязательно)

Если вы хотите узнать, как работает код, изучите приведенные ниже фрагменты. Если нет, вы можете сразу перейти к разделу [Выполнение приложения](#run-the-application).

Библиотека [go-redis](https://github.com/go-redis/redis) с открытым кодом используется для взаимодействия с Кэшем Azure для Redis.

Сначала с помощью функции `main` считывается имя узла и пароль (ключ доступа) для экземпляра Кэша Azure для Redis.

```go
func main() {
    redisHost := os.Getenv("REDIS_HOST")
    redisPassword := os.Getenv("REDIS_PASSWORD")
...
```

Затем мы устанавливаем подключение к Кэшу Azure для Redis. Обратите внимание, что используется [tls.Config](https://golang.org/pkg/crypto/tls/#Config). Кэш Azure для Redis принимает только безопасные подключения с использованием [TLS 1.2 в качестве минимальной требуемой версии](cache-remove-tls-10-11.md).

```go
...
op := &redis.Options{Addr: redisHost, Password: redisPassword, TLSConfig: &tls.Config{MinVersion: tls.VersionTLS12}}
client := redis.NewClient(op)

ctx := context.Background()
err := client.Ping(ctx).Err()
if err != nil {
    log.Fatalf("failed to connect with redis instance at %s - %v", redisHost, err)
}
...
```

Если подключение установлено успешно, [обработчики HTTP-данных](https://golang.org/pkg/net/http/#HandleFunc) настраиваются для обработки операций `POST` и `GET`, а также запуска HTTP-сервера. 

> [!NOTE] 
> Для маршрутизации используется [библиотека gorilla mux](https://github.com/gorilla/mux) (хотя это не является строгим требованием, и для этого примера приложения можно было бы использовать стандартную библиотеку).
>

```go
uh := userHandler{client: client}

router := mux.NewRouter()
router.HandleFunc("/users/", uh.createUser).Methods(http.MethodPost)
router.HandleFunc("/users/{userid}", uh.getUser).Methods(http.MethodGet)

log.Fatal(http.ListenAndServe(":8080", router))
```

Структура `userHandler` инкапсулирует [redis.Client](https://pkg.go.dev/github.com/go-redis/redis/v8#Client), который используется методами `createUser` и `getUser`. Код для этих методов опущен для краткости. 

- `createUser` — принимает полезные данные JSON (содержащие сведения о пользователе) и сохраняет их в качестве `HASH` в Кэше Azure для Redis.
- `getUser` — извлекает сведения о пользователе из `HASH` или возвращает ответ HTTP `404`, если они не найдены.

```go
type userHandler struct {
    client *redis.Client
}
...

func (uh userHandler) createUser(rw http.ResponseWriter, r *http.Request) {
    // details omitted
}
...

func (uh userHandler) getUser(rw http.ResponseWriter, r *http.Request) {
    // details omitted
}
```

## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Прежде всего, клонируйте приложение из репозитория GitHub.

1. Откройте командную строку и создайте новую папку с именем `git-samples`.

    ```bash
    md "C:\git-samples"
    ```

1. Откройте окно терминала Git, например Git Bash. С помощью команды `cd` перейдите к новой папке, куда будете клонировать пример приложения.

    ```bash
    cd "C:\git-samples"
    ```

1. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере.

    ```bash
    git clone https://github.com/Azure-Samples/azure-redis-cache-go-quickstart.git
    ```

## <a name="run-the-application"></a>Выполнение приложения

Приложение принимает сведения о подключении и учетные данные в виде переменных среды. 

1. Извлеките **имя узла** и **ключи доступа** (доступны в разделе "Ключи доступа") для экземпляра Кэша Azure для Redis на [портале Azure](https://portal.azure.com/).

1. Задайте их для соответствующих переменных среды:

    ```shell
    set REDIS_HOST=<Host name>:<port> (e.g. <name of cache>.redis.cache.windows.net:6380)
    set REDIS_PASSWORD=<Primary Access Key>
    ```

1. В окне терминала перейдите в нужную папку. Пример:

    ```shell
    cd "C:\git-samples\azure-redis-cache-go-quickstart"
    ```

1. В сеансе терминала выполните следующую команду, чтобы запустить приложение.

    ```shell
    go run main.go
    ```

HTTP-сервер будет запущен на порте `8080`.

## <a name="test-the-application"></a>Тестирование приложения

1. Создайте несколько записей пользователей. В следующем примере используется curl:

    ```bash
    curl -i -X POST -d '{"id":"1","name":"foo1", "email":"foo1@baz.com"}' localhost:8080/users/
    curl -i -X POST -d '{"id":"2","name":"foo2", "email":"foo2@baz.com"}' localhost:8080/users/
    curl -i -X POST -d '{"id":"3","name":"foo3", "email":"foo3@baz.com"}' localhost:8080/users/
    ```

1. Получите сведения о существующем пользователе, используя его `id`.

    ```bash
    curl -i localhost:8080/users/1
    ```

    Должен отобразиться примерно такой ответ JSON:
    
    ```json
    {
        "email": "foo1@bar",
        "id": "1",
        "name": "foo1"
    }
    ```

1. При попытке получить сведения о пользователе, которого не существует, поступит HTTP-ответ `404`. Пример:

    ```bash
    curl -i localhost:8080/users/100
    
    #response

    HTTP/1.1 404 Not Found
    Date: Fri, 08 Jan 2021 13:43:39 GMT
    Content-Length: 0
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы закончили работу с группой ресурсов и ресурсами Azure, созданными в рамках этого краткого руководства, вы можете удалить их, чтобы избежать ненужных расходов.

> [!IMPORTANT]
> Удаление группы ресурсов — необратимая операция, и все соответствующие ресурсы удаляются окончательно. Если вы создали экземпляр Кэша Azure для Redis в существующей группе ресурсов, которую вы хотите сохранить, можно удалить только кэш, выбрав **Удалить** на странице **Обзор** кэша. 

Чтобы удалить группу ресурсов и ее экземпляра Кэша Azure для Redis, выполните следующие действия.

1. На [портале Azure](https://portal.azure.com) найдите и выберите **Группы ресурсов**.
1. В текстовом поле **Фильтровать по имени** введите имя группы ресурсов, содержащей экземпляр кэша, а затем выберите его в результатах поиска. 
1. На странице группы ресурсов щелкните **Удалить группу ресурсов**.
1. Введите имя группы ресурсов, а затем щелкните **Удалить**.
   
   ![Удаление группы ресурсов для Кэша Azure для Redis](./media/cache-python-get-started/delete-your-resource-group-for-azure-cache-for-redis.png)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как начать использование Go с Кэшем Azure для Redis. Вы настроили и запустили простое приложение на основе REST API для создания и получения сведений о пользователях с помощью структуры данных Redis `HASH`.

> [!div class="nextstepaction"]
> [Создание простого веб-приложения ASP.NET, в котором используется кэш Redis для Azure](./cache-web-app-howto.md)
