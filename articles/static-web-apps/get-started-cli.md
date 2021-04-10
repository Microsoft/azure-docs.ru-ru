---
title: Краткое руководство. Создание первого статического сайта с помощью службы "Статические веб-приложения Azure" в CLI
description: Узнайте, как развернуть статический сайт в службе "Статические веб-приложения Azure" с помощью Azure CLI.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: cshoe
ms.openlocfilehash: a95e1658c3633f4ae8d09b71e9d3b0c82446754a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105727592"
---
# <a name="quickstart-building-your-first-static-site-using-the-azure-cli"></a>Краткое руководство. Создание первого статического сайта с помощью Azure CLI

Служба "Статические веб-приложения Azure" публикует веб-сайты в рабочей среде, создавая приложения из репозитория GitHub. В этом кратком руководстве рассказывается о том, как развернуть веб-приложение в службе "Статические веб-приложения Azure" с помощью Azure CLI.

Если у вас еще нет подписки Azure, создайте [бесплатную пробную учетную запись](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Предварительные требования

- [GitHub](https://github.com) .
- [Личный маркер доступа GitHub](https://docs.github.com/github/authenticating-to-github/creating-a-personal-access-token)
- Учетная запись [Azure](https://portal.azure.com)
- Установленное средство [Azure CLI](/cli/azure/install-azure-cli) (версии 2.8.0 или выше)

[!INCLUDE [create repository from template](../../includes/static-web-apps-get-started-create-repo.md)]

[!INCLUDE [clone the repository](../../includes/static-web-apps-get-started-clone-repo.md)]

Затем перейдите в новую папку с помощью следующей команды.

```bash
cd my-first-static-web-app
```

## <a name="create-a-static-web-app"></a>Создание статического веб-приложения

Теперь, когда репозиторий создан, можно создать статическое веб-приложение в Azure CLI.

> [!IMPORTANT]
> Убедитесь, что текущая папка в терминале — _my-first-static-web-app_.

1. Войти в Azure CLI с помощью следующей команды.

    ```azurecli
    az login
    ```

1. Создайте новое статическое веб-приложение из репозитория.

    # <a name="no-framework"></a>[Без платформы](#tab/vanilla-javascript)

    ```azurecli
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b main \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    # <a name="angular"></a>[Angular](#tab/angular)

    ```azurecli
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b main \
        --app-artifact-location "dist/angular-basic" \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    # <a name="react"></a>[React](#tab/react)

    ```azurecli
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b main \
        --app-artifact-location "build" \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    # <a name="vue"></a>[Vue](#tab/vue)

    ```azurecli
    az staticwebapp create \
        -n my-first-static-web-app \
        -g <RESOURCE_GROUP_NAME> \
        -s https://github.com/<YOUR_GITHUB_ACCOUNT_NAME>/my-first-static-web-app \
        -l <LOCATION> \
        -b main \
        --app-artifact-location "dist" \
        --token <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
    ```

    ---
    
    > [!IMPORTANT]
    > URL-адрес, переданный в параметр `s`, не должен включать суффикс `.git`.

    - `<RESOURCE_GROUP_NAME>`: замените это значение именем существующей [группы ресурсов Azure](../azure-resource-manager/management/manage-resources-cli.md).

      - Дополнительные сведения о получении списка групп ресурсов см. в документации по [az group](/cli/azure/group#az_group_list).

    - `<YOUR_GITHUB_ACCOUNT_NAME>`: Замените это значение именем пользователя GitHub.

    - `<LOCATION>`: Замените это значение ближайшим к вам расположением. Доступные параметры:  _CentralUS_, _EastAsia_, _EastUS2_, _WestEurope_ и _WestUS2_.

    - `<YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>`: Замените это значение [личным маркером доступа GitHub](https://docs.github.com/github/authenticating-to-github/creating-a-personal-access-token), который вы создали ранее.

    Теперь вы можете просмотреть созданное приложение в Azure.

1. Откройте [портал Azure](https://portal.azure.com).

1. Выполните поиск **my-first-web-static-app** с помощью верхней строки поиска.

1. Выберите **my-first-web-static-app**.

[!INCLUDE [view website](../../includes/static-web-apps-get-started-view-website.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, вы можете удалить экземпляр службы "Статические веб-приложения Azure" с помощью следующей команды.

```azurecli
az staticwebapp delete \
    --name my-first-static-web-app \
    --resource-group my-first-static-web-app
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление API](add-api.md)