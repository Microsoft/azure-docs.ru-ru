---
title: Настраиваемые контейнеры CI/CD из действий GitHub
description: Узнайте, как использовать действия GitHub для развертывания пользовательского контейнера Linux в службе приложений из конвейера CI/CD.
ms.devlang: na
ms.topic: article
ms.date: 12/04/2020
ms.author: jafreebe
ms.reviewer: ushan
ms.custom: github-actions-azure
ms.openlocfilehash: fec4ba8cba33a1d52d8f330308645fb616921ba4
ms.sourcegitcommit: 78ecfbc831405e8d0f932c9aafcdf59589f81978
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2021
ms.locfileid: "98726815"
---
# <a name="deploy-a-custom-container-to-app-service-using-github-actions"></a>Развертывание настраиваемого контейнера в Службе приложений с помощью GitHub Actions

[Действия GitHub](https://docs.github.com/en/free-pro-team@latest/actions) дают возможность создать автоматизированный рабочий процесс разработки программного обеспечения. С помощью [действия веб-развертывание Azure](https://github.com/Azure/webapps-deploy)можно автоматизировать рабочий процесс, чтобы развернуть пользовательские контейнеры в [службе приложений](overview.md) , используя действия GitHub.

Рабочий процесс определяется файлом YAML (.yml) по пути `/.github/workflows/` в вашем репозитории. Это определение содержит различные шаги и параметры, которые находятся в рабочем процессе.

Для рабочего процесса контейнера службы приложений Azure файл содержит три раздела:

|Section  |Задания  |
|---------|---------|
|**Аутентификация** | 1. получение субъекта-службы или профиля публикации. <br /> 2. Создание секрета GitHub. |
|**Сборка** | 1. Создайте среду. <br /> 2. Создайте образ контейнера. |
|**Развертывание** | 1. Разверните образ контейнера. |

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Учетная запись GitHub. Если у вас ее нет, зарегистрируйтесь [бесплатно](https://github.com/join). Для развертывания в службе приложений Azure необходимо иметь код в репозитории GitHub. 
- Рабочий реестр контейнеров и приложение службы приложений Azure для контейнеров. В этом примере используется реестр контейнеров Azure. Обязательно выполните полное развертывание в службе приложений Azure для контейнеров. В отличие от обычных веб-приложений, веб-приложения для контейнеров не имеют целевой страницы по умолчанию. Опубликуйте контейнер, чтобы получить рабочий пример.
    - [Узнайте, как создать контейнерное Node.jsное приложение с помощью DOCKER, отправить образ контейнера в реестр, а затем развернуть образ в службе приложений Azure.](/azure/developer/javascript/tutorial-vscode-docker-node-01)
        
## <a name="generate-deployment-credentials"></a>Создание учетных данных для развертывания.

Рекомендуемый способ проверки подлинности в службе приложений Azure для действий GitHub — профиль публикации. Вы также можете пройти проверку подлинности с помощью субъекта-службы, но процесс требует дополнительных действий. 

Сохраните учетные данные профиля публикации или субъекта-службы в качестве [секрета GitHub](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets) для аутентификации в Azure. Вы получите доступ к секрету в рабочем процессе. 

# <a name="publish-profile"></a>[Опубликовать профиль](#tab/publish-profile)

Профиль публикации — это учетные данные на уровне приложения. Настройте профиль публикации в качестве секрета GitHub. 

1. Перейдите к службе приложений в портал Azure. 

1. На странице **Обзор** выберите **получить профиль публикации**.

    > [!NOTE]
    > По состоянию на Октябрь 2020 для веб-приложений Linux потребуется параметр приложения, `WEBSITE_WEBDEPLOY_USE_SCM` установленный в значение `true` **перед скачиванием файла**. Это требование будет удалено в будущем. Сведения о настройке общих параметров веб-приложения см. в разделе [Настройка приложения службы приложений в портал Azure](./configure-common.md).  

1. Сохраните скачанный файл. Вы будете использовать содержимое файла для создания секрета GitHub.

# <a name="service-principal"></a>[Субъект-служба](#tab/service-principal)

Вы можете создать [субъект-службу](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) в [Azure CLI](/cli/azure/). Чтобы выполнить эту команду, откройте [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или нажмите кнопку **Попробовать**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

В примере Замените заполнители ИДЕНТИФИКАТОРом подписки, именем группы ресурсов и именем приложения. Выходные данные — это объект JSON с учетными данными назначения роли, которые предоставляют доступ к приложению службы приложений. Скопируйте этот объект JSON для последующего использования.

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
> Рекомендуется всегда предоставлять минимальные разрешения доступа. Область в предыдущем примере ограничена конкретным приложением службы приложений, а не всей группой ресурсов.

---
## <a name="configure-the-github-secret-for-authentication"></a>Настройка секрета GitHub для проверки подлинности

# <a name="publish-profile"></a>[Опубликовать профиль](#tab/publish-profile)

В [GitHub](https://github.com/)найдите репозиторий, выберите **параметры > секреты > добавить новый секрет**.

Чтобы использовать [учетные данные уровня приложения](#generate-deployment-credentials), вставьте содержимое скачанного файла профиля публикации в поле значение секрета. Назовите секрет `AZURE_WEBAPP_PUBLISH_PROFILE` .

При настройке рабочего процесса GitHub используйте `AZURE_WEBAPP_PUBLISH_PROFILE` в действии развертывание веб-приложения Azure. Например:
    
```yaml
- uses: azure/webapps-deploy@v2
  with:
    publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

# <a name="service-principal"></a>[Субъект-служба](#tab/service-principal)

В [GitHub](https://github.com/)найдите репозиторий, выберите **параметры > секреты > добавить новый секрет**.

Чтобы использовать [учетные данные на уровне пользователя](#generate-deployment-credentials), вставьте все выходные данные JSON из команды Azure CLI в поле значения секрета. Присвойте секрету имя, например `AZURE_CREDENTIALS` .

Когда вы позже настроите файл рабочего процесса, этот секрет будет передан в действие входа в Azure как параметр `creds`. Пример:

```yaml
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## <a name="configure-github-secrets-for-your-registry"></a>Настройка секретов GitHub для реестра

Определите секреты для использования с действием DOCKER Login. В примере в этом документе для реестра контейнеров используется реестр контейнеров Azure. 

1. Перейдите к контейнеру в портал Azure или DOCKER и скопируйте имя пользователя и пароль. Имя пользователя и пароль реестра контейнеров Azure можно найти в портал Azure в разделе **Параметры**  >  **ключи доступа** для реестра. 

2. Определите новый секрет для имени пользователя реестра с именем `REGISTRY_USERNAME` . 

3. Определите новый секрет для пароля реестра с именем `REGISTRY_PASSWORD` . 

## <a name="build-the-container-image"></a>Создание образа контейнера

В следующем примере показана часть рабочего процесса, который создает Node.JS образ DOCKER. Используйте [имя входа DOCKER](https://github.com/azure/docker-login) для входа в закрытый реестр контейнеров. В этом примере используется реестр контейнеров Azure, но одно и то же действие работает для других реестров. 


```yaml
name: Linux Container Node Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     
```

Можно также использовать [имя входа DOCKER](https://github.com/azure/docker-login) для одновременного входа в несколько реестров контейнеров. Этот пример включает два новых секрета GitHub для проверки подлинности с помощью docker.io. В этом примере предполагается, что на корневом уровне реестра имеется Dockerfile. 

```yml
name: Linux Container Node Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - uses: azure/docker-login@v1
      with:
        login-server: index.docker.io
        username: ${{ secrets.DOCKERIO_USERNAME }}
        password: ${{ secrets.DOCKERIO_PASSWORD }}
    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     
```

## <a name="deploy-to-an-app-service-container"></a>Развертывание в контейнер службы приложений

Чтобы развернуть образ в пользовательском контейнере в службе приложений, используйте `azure/webapps-deploy@v2` действие. Это действие имеет семь параметров:

| **Параметр**  | **Пояснение**  |
|---------|---------|
| **app-name** | (Обязательный) Имя приложения Службы приложений. | 
| **publish-profile** | Используемых Применяется к веб-приложениям (Windows и Linux) и к контейнерам веб-приложений (Linux). Сценарий с несколькими контейнерами не поддерживается. Публикация \* содержимого файла профиля (. publishsettings) с веб-развертывание секретами | 
| **slot-name** | (Необязательный) Введите существующий слот вместо рабочего. |
| **package** | Используемых Применяется только к веб-приложению: путь к пакету или папке. \*. zip, \* . war-файл, \* JAR-файл или папка для развертывания |
| **images** | Необходимости Применяется только к контейнерам веб-приложений: Укажите полное имя образа контейнера. Например, "myregistry.azurecr.io/nginx:latest" или "Python: 3.7.2-Alpine/". Для многоконтейнерного приложения можно указать несколько имен образов контейнеров (разделенных несколькими строками). |
| **файл конфигурации** | Используемых Применяется только к контейнерам веб-приложений: путь к файлу Docker-Compose. Должен быть полным путем или относительным по отношению к рабочему каталогу по умолчанию. Требуется для приложений с несколькими контейнерами. |
| **Команда запуска** | Используемых Введите команду запуска. Для ex Запуск DotNet или DotNet filename.dll |

# <a name="publish-profile"></a>[Опубликовать профиль](#tab/publish-profile)

```yaml
name: Linux Container Node Workflow

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}

    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     

    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'myapp'
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        images: 'mycontainer.azurecr.io/myapp:${{ github.sha }}'
```
# <a name="service-principal"></a>[Субъект-служба](#tab/service-principal)

```yaml
on: [push]

name: Linux_Container_Node_Workflow

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@main
    
    - name: 'Login via Azure CLI'
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - uses: azure/docker-login@v1
      with:
        login-server: mycontainer.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    - run: |
        docker build . -t mycontainer.azurecr.io/myapp:${{ github.sha }}
        docker push mycontainer.azurecr.io/myapp:${{ github.sha }}     
      
    - uses: azure/webapps-deploy@v2
      with:
        app-name: 'myapp'
        images: 'mycontainer.azurecr.io/myapp:${{ github.sha }}'
    
    - name: Azure logout
      run: |
        az logout
```

---

## <a name="next-steps"></a>Дальнейшие действия

Вы можете найти наш набор компонентов Actions, объединенных в разных репозитории, а также соответствующую документацию и примеры использования GitHub для CI/CD и развертыванию приложений в Azure на GitHub.

- [Рабочие процессы действий для развертывания в Azure](https://github.com/Azure/actions-workflow-samples)

- [Вход в Azure](https://github.com/Azure/login)

- [Веб-приложение Azure](https://github.com/Azure/webapps-deploy)

- [Вход в Docker и выход из него](https://github.com/Azure/docker-login)

- [События, инициирующие рабочие процессы](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows)

- [Развертывание Kubernetes](https://github.com/Azure/k8s-deploy)

- [Начальные рабочие процессы](https://github.com/actions/starter-workflows)