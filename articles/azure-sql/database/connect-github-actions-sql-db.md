---
title: Краткое руководство. Подключение к Базе данных SQL Azure с помощью GitHub Actions
description: Использование Azure SQL из рабочего процесса GitHub Actions
author: juliakm
services: sql-database
ms.service: sql-database
ms.topic: quickstart
ms.author: jukullam
ms.date: 10/12/2020
ms.custom: github-actions-azure
ms.openlocfilehash: ceb9f0f9ef2a88532d5af16a03fcfd0282da84f8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107787293"
---
# <a name="use-github-actions-to-connect-to-azure-sql-database"></a>Применение GitHub Actions для подключения к Базе данных SQL Azure

Начните работу с [GitHub Actions](https://docs.github.com/en/actions), настроив и применив рабочий процесс для развертывания обновлений базы данных в [Базе данных SQL](../azure-sql-iaas-vs-paas-what-is-overview.md). 

## <a name="prerequisites"></a>Обязательные условия

Вам потребуется: 
- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Репозиторий GitHub с пакетом DACPAC (`Database.dacpac`). Если у вас нет учетной записи GitHub, [зарегистрируйтесь бесплатно](https://github.com/join).  
- База данных Azure SQL.
    - [Краткое руководство. Создание отдельной базы данных в Базе данных SQL Azure](single-database-create-quickstart.md)
    - [Создание пакета DACPAC из существующей Базы данных SQL Server](/sql/relational-databases/data-tier-applications/export-a-data-tier-application)

## <a name="workflow-file-overview"></a>Общие сведения о файле рабочего процесса

Рабочий процесс в GitHub Actions определяется файлом .yml, который размещается в папке репозитория `/.github/workflows/`. Это определение содержит разные шаги и параметры рабочего процесса.

Этот файл содержит два раздела:

|Section  |Задания  |
|---------|---------|
|**Аутентификация** | 1. Определение субъекта-службы. <br /> 2. Создание секрета GitHub. |
|**Развертывание** | 1. Развертывание базы данных. |

## <a name="generate-deployment-credentials"></a>Создание учетных данных для развертывания.

Вы можете создать [субъект-службу](../../active-directory/develop/app-objects-and-service-principals.md) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) в [Azure CLI](/cli/azure/). Чтобы выполнить эту команду, откройте [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или нажмите кнопку **Попробовать**.

Замените заполнители `server-name` именем сервера SQL, размещенного в Azure. Замените `subscription-id` и `resource-group` идентификатором подписки и группой ресурсов, которые сопоставлены с этим сервером SQL.  

```azurecli-interactive
   az ad sp create-for-rbac --name {server-name} --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
```

Выходные данные содержат объект JSON с учетными данными назначения роли, которые предоставляют доступ к этой базе данных, как показано в этом примере. Скопируйте выходной объект JSON для последующего использования.

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

## <a name="copy-the-sql-connection-string"></a>Копирование строки подключения SQL 

На портале Azure перейдите к серверу Базы данных SQL и последовательно откройте **Параметры** > **Строки подключения**. Скопируйте строку подключения по протоколу **ADO.NET**. Замените заполнители значениями для `your_database` и `your_password`. Теперь строка подключения будет выглядеть как в этих выходных данных. 

```output
    Server=tcp:my-sql-server.database.windows.net,1433;Initial Catalog={your-database};Persist Security Info=False;User ID={admin-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
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

1. Вставьте значение строки подключения в поле значения секрета. Присвойте секрету имя `AZURE_SQL_CONNECTION_STRING`.


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

1. Присвойте рабочему процессу имя `SQL for GitHub Actions` и добавьте действия для извлечения и входа. Эти действия извлекают код сайта из репозитория и выполняют проверку подлинности в Azure с помощью ранее созданного секрета GitHub `AZURE_CREDENTIALS`. 

    ```yaml
    name: SQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    build:
        runs-on: windows-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

1. Используйте действие развертывания Azure SQL для подключения к экземпляру SQL. Замените `SQL_SERVER_NAME` именем таблицы. На корневом уровне репозитория должен размещаться пакет DACPAC с именем `Database.dacpac`. 

    ```yaml
    - uses: azure/sql-action@v1
      with:
        server-name: SQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_SQL_CONNECTION_STRING }}
        sql-file: './Database.dacpac'
    ``` 

1. Завершите создание рабочего процесса, добавив действие для выхода из Azure. Готовый рабочий процесс выглядит так. В папке `.github/workflows` репозитория появится новый файл.

    ```yaml
   name: SQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]


    jobs:
    build:
        runs-on: windows-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

    - uses: azure/sql-action@v1
      with:
        server-name: SQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_SQL_CONNECTION_STRING }}
        sql-file: './Database.dacpac'

        # Azure logout 
    - name: logout
      run: |
         az logout
    ```

## <a name="review-your-deployment"></a>Проверка развертывания

1. Перейдите в раздел **Actions** (Действия) для репозитория GitHub. 

1. Откройте первый результат, чтобы проверить подробные журналы выполнения рабочего процесса. 
 
   :::image type="content" source="media/quickstart-sql-github-actions/github-actions-run-sql.png" alt-text="Журнал выполнения действий GitHub Actions":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда вы закончите работу с базой данных и репозиторием Azure SQL, очистите развернутые ресурсы, удалив группу ресурсов и репозиторий GitHub. 

## <a name="next-steps"></a>Дальнейшие шаги

> [!div class="nextstepaction"]
> [Сведения об интеграции Azure с GitHub](/azure/developer/github/)