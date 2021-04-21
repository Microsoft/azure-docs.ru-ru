---
title: Краткое руководство. Подключение к Azure PostgreSQL с помощью GitHub Actions
description: Использование Azure PostgreSQL из рабочего процесса GitHub Actions
author: mksuni
ms.service: postgresql
ms.topic: quickstart
ms.author: sumuth
ms.date: 10/12/2020
ms.custom: github-actions-azure
ms.openlocfilehash: 7fc59c0d9036a2e83c742f51fc17750d40e057fe
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107791433"
---
# <a name="quickstart-use-github-actions-to-connect-to-azure-postgresql"></a>Краткое руководство. Использование GitHub Actions для подключения к Azure PostgreSQL

<Token>**ОБЛАСТЬ ПРИМЕНЕНИЯ:** :::image type="icon" source="./media/applies-to/yes.png" border="false":::База данных Azure для PostgreSQL — отдельный сервер :::image type="icon" source="./media/applies-to/yes.png" border="false":::База данных Azure для PostgreSQL — гибкий сервер</Token>

Начните работу с [GitHub Actions](https://docs.github.com/en/actions), используя рабочий процесс для развертывания обновлений базы данных в [Базе данных Azure для PostgreSQL](https://azure.microsoft.com/services/postgresql/).

## <a name="prerequisites"></a>Обязательные условия

Вам потребуется:
- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Репозиторий GitHub с примером данных (`data.sql`). Если у вас нет учетной записи GitHub, [зарегистрируйтесь бесплатно](https://github.com/join).
- Сервер Базы данных Azure для PostgreSQL.
    - [Краткое руководство. Создание базы данных Azure для сервера PostgreSQL на портале Azure](quickstart-create-server-database-portal.md)

## <a name="workflow-file-overview"></a>Общие сведения о файле рабочего процесса

Рабочий процесс в GitHub Actions определяется файлом .yml, который размещается в папке репозитория `/.github/workflows/`. Это определение содержит разные шаги и параметры рабочего процесса.

Этот файл содержит два раздела:

|Section  |Задания  |
|---------|---------|
|**Аутентификация** | 1. Определение субъекта-службы. <br /> 2. Создание секрета GitHub. |
|**Развертывание** | 1. Развертывание базы данных. |

## <a name="generate-deployment-credentials"></a>Создание учетных данных для развертывания.

Вы можете создать [субъект-службу](../active-directory/develop/app-objects-and-service-principals.md) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac&preserve-view=true) в [Azure CLI](/cli/azure/). Чтобы выполнить эту команду, откройте [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или нажмите кнопку **Попробовать**.

Замените заполнители `server-name` именем сервера PostgreSQL, размещенного в Azure. Замените `subscription-id` и `resource-group` идентификатором подписки и группой ресурсов, связанными с вашим сервером PostgreSQL.

```azurecli-interactive
   az ad sp create-for-rbac --name {server-name} --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
```

Выходные данные содержат объект JSON с учетными данными назначения роли, которые предоставляют доступ к этой базе данных, как показано ниже. Скопируйте этот выходной объект JSON для последующего использования.

```output
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Рекомендуется всегда предоставлять минимальные разрешения доступа. В предыдущем примере область действия ограничена конкретным сервером и не распространяется на группу ресурсов.

## <a name="copy-the-postgresql-connection-string"></a>Копирование строки подключения PostgreSQL

На портале Azure перейдите к серверу Базы данных Azure для PostgreSQL и откройте раздел **Параметры** > **Строки подключения**. Скопируйте строку подключения по протоколу **ADO.NET**. Замените заполнители значениями для `your_database` и `your_password`. Теперь строка подключения будет выглядеть примерно так.

> [!IMPORTANT]
> - Для отдельного сервера используйте ```user=adminusername@servername```. Обратите внимание, что часть ```@servername``` является обязательной.
> - Для гибкого сервера используйте ```user= adminusername``` без ```@servername```.

```output
psql host={servername.postgres.database.azure.com} port=5432 dbname={your_database} user={adminusername} password={your_database_password} sslmode=require
```

Эту строку подключения вы примените в качестве секрета GitHub.

## <a name="configure-the-github-secrets"></a>Настройка секретов GitHub

1. В [GitHub](https://github.com/) найдите нужный репозиторий.

1. Выберите **Settings > Secrets > New secret** (Параметры > Секреты > Новый секрет).

1. Вставьте все выходные данные JSON, полученные из команды Azure CLI, в поле значения секрета. Присвойте секрету имя `AZURE_CREDENTIALS`.

    Когда вы позже настроите файл рабочего процесса, этот секрет будет передан в действие входа в Azure как параметр `creds`. Пример:

    ```yaml
    - uses: azure/login@v1
    with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
   ```

1. Снова выберите **New secret** (Новый секрет).

1. Вставьте значение строки подключения в поле значения секрета. Присвойте секрету имя `AZURE_POSTGRESQL_CONNECTION_STRING`.


## <a name="add-your-workflow"></a>Добавление рабочего процесса

1. Перейдите в раздел **Actions** (Действия) для репозитория GitHub.

2. Выберите **Set up your workflow yourself** (Настроить рабочий процесс самостоятельно).

2. Удалите все содержимое в файле рабочего процесса после раздела `on:`. После этого рабочий процесс должен выглядеть примерно так.

    ```yaml
    name: CI

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]
    ```

1. Присвойте рабочему процессу имя `PostgreSQL for GitHub Actions` и добавьте действия для извлечения и входа. Эти действия извлекают код сайта из репозитория и выполняют проверку подлинности в Azure с помощью ранее созданного секрета GitHub `AZURE_CREDENTIALS`.

    ```yaml
    name: PostgreSQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

2. Используйте действие развертывания Azure PostgreSQL для подключения к экземпляру PostgreSQL. Замените `POSTGRESQL_SERVER_NAME` именем сервера. На корневом уровне репозитория должен размещаться файл данных PostgreSQL с именем `data.sql`.

    ```yaml
     - uses: azure/postgresql@v1
       with:
        connection-string: ${{ secrets.AZURE_POSTGRESQL_CONNECTION_STRING }}
        server-name: POSTGRESQL_SERVER_NAME
        sql-file: './data.sql'
    ```

3. Завершите создание рабочего процесса, добавив действие для выхода из Azure. Готовый рабочий процесс выглядит так. В папке `.github/workflows` репозитория появится новый файл.

    ```yaml
   name: PostgreSQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]


    jobs:
    build:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

    - uses: azure/postgresql@v1
      with:
        server-name: POSTGRESQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_POSTGRESQL_CONNECTION_STRING }}
        sql-file: './data.sql'

        # Azure logout
    - name: logout
      run: |
         az logout
    ```

## <a name="review-your-deployment"></a>Проверка развертывания

1. Перейдите в раздел **Actions** (Действия) для репозитория GitHub.

1. Откройте первый результат, чтобы проверить подробные журналы выполнения рабочего процесса.

    :::image type="content" source="media/how-to-deploy-github-action/gitbub-action-postgres-success.png" alt-text="Журнал выполнения действий GitHub Actions":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда вы закончите работу с базой данных и репозиторием Azure PostgreSQL, очистите ненужные развернутые ресурсы, удалив группу ресурсов и репозиторий GitHub.

## <a name="next-steps"></a>Дальнейшие шаги

> [!div class="nextstepaction"]
> [Сведения об интеграции Azure с GitHub](/azure/developer/github/)
<br/>
> [!div class="nextstepaction"]
> [Сведения о подключении к серверу](how-to-connect-query-guide.md)
