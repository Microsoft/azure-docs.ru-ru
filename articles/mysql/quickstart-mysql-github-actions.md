---
title: Краткое руководство. Подключение к Azure MySQL с помощью GitHub Actions
description: Использование Azure MySQL из рабочего процесса GitHub Actions
author: juliakm
ms.service: mysql
ms.topic: quickstart
ms.author: jukullam
ms.date: 10/12/2020
ms.custom: github-actions-azure
ms.openlocfilehash: c4e21267c5eee9d86c05c51bc57bebfee699ef2c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103200594"
---
# <a name="quickstart-use-github-actions-to-connect-to-azure-mysql"></a>Краткое руководство. Использование GitHub Actions для подключения к Azure MySQL

**Область применения**: :::image type="icon" source="./media/applies-to/yes.png" border="false":::Отдельный сервер Базы данных Azure для MySQL:::image type="icon" source="./media/applies-to/yes.png" border="false":::Гибкий сервер Базы данных Azure для MySQL

Начните работу с [GitHub Actions](https://docs.github.com/en/actions), используя рабочий процесс для развертывания обновлений базы данных в [Базе данных Azure для MySQL](https://azure.microsoft.com/services/mysql/).


## <a name="prerequisites"></a>Обязательные условия

Вам потребуется: 
- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Репозиторий GitHub с примером данных (`data.sql`). Если у вас нет учетной записи GitHub, [зарегистрируйтесь бесплатно](https://github.com/join).  
- Сервер Базы данных Azure для MySQL.
    - [Краткое руководство. Создание базы данных Azure для сервера MySQL с помощью портала Azure](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="workflow-file-overview"></a>Общие сведения о файле рабочего процесса

Рабочий процесс в GitHub Actions определяется файлом .yml, который размещается в папке репозитория `/.github/workflows/`. Это определение содержит разные шаги и параметры рабочего процесса.

Этот файл содержит два раздела:

|Section  |Задания  |
|---------|---------|
|**Аутентификация** | 1. Определение субъекта-службы. <br /> 2. Создание секрета GitHub. |
|**Развертывание** | 1. Развертывание базы данных. |

## <a name="generate-deployment-credentials"></a>Создание учетных данных для развертывания.

Вы можете создать [субъект-службу](../active-directory/develop/app-objects-and-service-principals.md) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac&preserve-view=true) в [Azure CLI](/cli/azure/). Чтобы выполнить эту команду, откройте [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или нажмите кнопку **Попробовать**.

Замените заполнители `server-name` именем сервера MySQL, размещенного в Azure. Замените `subscription-id` и `resource-group` идентификатором подписки и группой ресурсов, связанных с вашим сервером MySQL.  

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

## <a name="copy-the-mysql-connection-string"></a>Скопируйте строку подключения MySQL 

На портале Azure перейдите к серверу Базы данных Azure для MySQL и откройте раздел **Параметры** > **Строки подключения**. Скопируйте строку подключения по протоколу **ADO.NET**. Замените заполнители значениями для `your_database` и `your_password`. Теперь строка подключения будет выглядеть примерно так. 

> [!IMPORTANT]
> - Для отдельного сервера используйте **Uid=adminusername@servername** . Обратите внимание, что часть **@servername** является обязательной.
> - Для гибкого сервера используйте **Uid=adminusername** без @servername. Обратите внимание, что гибкий сервер MySQL предоставляется в предварительной версии. 


```output
   Server=my-mysql-server.mysql.database.azure.com; Port=3306; Database={your_database}; Uid=adminname@my-mysql-server; Pwd={your_password}; SslMode=Preferred;
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

1. Вставьте значение строки подключения в поле значения секрета. Присвойте секрету имя `AZURE_MYSQL_CONNECTION_STRING`.


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

1. Присвойте рабочему процессу имя `MySQL for GitHub Actions` и добавьте действия для извлечения и входа. Эти действия извлекают код сайта из репозитория и выполняют проверку подлинности в Azure с помощью ранее созданного секрета GitHub `AZURE_CREDENTIALS`. 

    ```yaml
    name: MySQL for GitHub Actions

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

1. Используйте действие развертывания Azure MySQL для подключения к экземпляру MySQL. Замените `MYSQL_SERVER_NAME` именем сервера. На корневом уровне репозитория должен размещаться файл данных MySQL с именем `data.sql`. 

    ```yaml
    - uses: azure/mysql@v1
      with:
        server-name: MYSQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_MYSQL_CONNECTION_STRING }}
        sql-file: './data.sql'
    ``` 

1. Завершите создание рабочего процесса, добавив действие для выхода из Azure. Готовый рабочий процесс выглядит так. В папке `.github/workflows` репозитория появится новый файл.

    ```yaml
    name: MySQL for GitHub Actions

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

            - uses: azure/mysql@v1
                with:
                    server-name: MYSQL_SERVER_NAME
                    connection-string: ${{ secrets.AZURE_MYSQL_CONNECTION_STRING }}
                    sql-file: './data.sql'

            # Azure logout 
            - name: logout
                run: |
                    az logout
    ```

## <a name="review-your-deployment"></a>Проверка развертывания

1. Перейдите в раздел **Actions** (Действия) для репозитория GitHub. 

1. Откройте первый результат, чтобы проверить подробные журналы выполнения рабочего процесса. 
 
    :::image type="content" source="media/quickstart-mysql-github-actions/github-actions-run-mysql.png" alt-text="Журнал выполнения действий GitHub Actions":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда вы закончите работу с базой данных и репозиторием Azure MySQL, очистите ненужные развернутые ресурсы, удалив группу ресурсов и репозиторий GitHub. 

## <a name="next-steps"></a>Дальнейшие шаги

> [!div class="nextstepaction"]
> [Сведения об интеграции Azure с GitHub](/azure/developer/github/)
