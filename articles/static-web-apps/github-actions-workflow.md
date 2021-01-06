---
title: Рабочие процессы GitHub Actions для Статических веб-приложений Azure
description: Сведения об использовании репозитория GitHub для настройки непрерывного развертывания в Статических веб-приложениях Azure.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 5e6188ca2e8e0972e86bed578144a29a96570876
ms.sourcegitcommit: 5e762a9d26e179d14eb19a28872fb673bf306fa7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/05/2021
ms.locfileid: "97901204"
---
# <a name="github-actions-workflows-for-azure-static-web-apps-preview"></a>Рабочие процессы GitHub Actions для предварительной версии Статических веб-приложений Azure

Azure создает рабочий процесс GitHub Actions для управления непрерывным развертыванием приложения при создании ресурса Статического веб-приложения Azure. Рабочий процесс управляется файлом YAML. В этой статье подробно описывается структура и параметры файла рабочего процесса.

Развертывания инициируются [триггерами](#triggers), которые выполняют [задания](#jobs), определяемые отдельными [шагами](#steps).

## <a name="file-location"></a>Размещение файла

При связывании репозитория GitHub со Статическими веб-приложениями Azure файл рабочего процесса добавляется в репозиторий.

Чтобы просмотреть созданный файл рабочего процесса, выполните следующие действия.

1. Откройте репозиторий приложения на сайте GitHub.
1. На вкладке _Code_ (Код) щелкните папку `.github/workflows`.
1. Щелкните файл с именем, которое выглядит как `azure-static-web-apps-<RANDOM_NAME>.yml`.

Файл YAML в репозитории будет выглядеть примерно так:

```yml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
    - master
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
    - master

jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
    - uses: actions/checkout@v2
      with:
        submodules: true
    - name: Build And Deploy
      id: builddeploy
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
        repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
        action: 'upload'
        ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
        app_location: '/' # App source code path
        api_location: 'api' # Api source code path - optional
        output_location: 'dist' # Built app content directory - optional
        ###### End of Repository/Build Configurations ######

  close_pull_request_job:
    if: github.event_name == 'pull_request' && github.event.action == 'closed'
    runs-on: ubuntu-latest
    name: Close Pull Request Job
    steps:
    - name: Close Pull Request
      id: closepullrequest
      uses: Azure/static-web-apps-deploy@v0.0.1-preview
      with:
        azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
        action: 'close'
```

## <a name="triggers"></a>Триггеры

[Триггер](https://help.github.com/actions/reference/events-that-trigger-workflows) GitHub Actions сообщает рабочему процессу GitHub Actions, что нужно запустить задание на основе триггеров событий. Триггеры перечислены с помощью свойства `on` в файле рабочего процесса.

```yml
on:
  push:
    branches:
    - master
  pull_request:
    types: [opened, synchronize, reopened, closed]
    branches:
    - master
```

С помощью параметров, связанных со свойством `on`, можно определить, какие ветви активируют задание, а также задать срабатывание триггеров для различных состояний запросов на вытягивание.

В этом примере рабочий процесс запускается при изменении _главной_ ветви. Изменения, запускающие рабочий процесс, включают в себя отправку фиксаций и открытие запросов на вытягивание для выбранной ветви.

## <a name="jobs"></a>Задания

Для каждого триггера события требуется обработчик событий. [Задания](https://help.github.com/actions/reference/workflow-syntax-for-github-actions#jobs) определяют, что происходит при запуске события.

В файле рабочего процесса Статических веб-приложений есть два доступных задания.

| Имя  | Описание |
|---------|---------|
|`build_and_deploy_job` | Выполняется при отправке фиксаций или открытии запроса на вытягивание для ветви, указанной в свойстве `on`. |
|`close_pull_request_job` | Выполняется только при закрытии запроса на включение внесенных изменений, который удаляет промежуточную среду, созданную из запросов на включение внесенных изменений. |

## <a name="steps"></a>Шаги

Шаги — это последовательные задачи задания. С помощью шага выполняются такие действия, как установка зависимостей, выполнение тестов и развертывание приложения в рабочей среде.

Файл рабочего процесса определяет следующие шаги.

| Задание  | Шаги  |
|---------|---------|
| `build_and_deploy_job` |<ol><li>Извлекает репозиторий в среде GitHub Actions.<li>Создает и развертывает репозиторий в Статических веб-приложениях Azure.</ol>|
| `close_pull_request_job` | <ol><li>Уведомляет Статические веб-приложения Azure о том, что запрос на вытягивание закрыт.</ol>|

## <a name="build-and-deploy"></a>Сборка и развертывание

Шаг с именем `Build and Deploy` выполняет сборку и развертывание в экземпляре Статических веб-приложений Azure. В разделе `with` можно настроить следующие значения для развертывания.

```yml
with:
    azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_MANGO_RIVER_0AFDB141E }}
    repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for GitHub integrations (i.e. PR comments)
    action: 'upload'
    ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
    app_location: '/' # App source code path
    api_location: 'api' # Api source code path - optional
    output_location: 'dist' # Built app content directory - optional
    ###### End of Repository/Build Configurations ######
```

| Свойство | Описание | Обязательно |
|---|---|---|
| `app_location` | Расположение кода приложения.<br><br>Например, введите `/`, если исходный код приложения находится в корне репозитория, или `/app`, если код приложения находится в каталоге с именем `app`. | Да |
| `api_location` | Расположение кода Функций Azure.<br><br>Например, введите `/api`, если код приложения находится в папке с именем `api`. Если в папке не обнаружено ни одного приложения Функций Azure, то в процессе сборки не произойдет сбой и в рабочем процессе предполагается, что API не нужен. | нет |
| `output_location` | Расположение выходного каталога сборки относительно `app_location`.<br><br>Например, если исходный код приложения находится в `/app`, а сценарий сборки выводит файлы в папку `/app/build`, установите `build` в качестве значения `output_location`. | нет |

Значения `repo_token`, `action` и `azure_static_web_apps_api_token`, настроенные с помощью Статических веб-приложений Azure, не должны изменяться вручную.

## <a name="custom-build-commands"></a>Команды настраиваемой сборки

Вы можете точно контролировать, какие команды выполняются во время развертывания. Следующие команды можно определить в разделе задания `with`.

Перед выполнением любой пользовательской команды для развертывания всегда вызывается `npm install`.

| Get-Help            | Описание |
|---------------------|-------------|
| `app_build_command` | Определяет пользовательскую команду, выполняемую во время развертывания приложения статического содержимого.<br><br>Например, чтобы настроить рабочую сборку для углового приложения, создайте сценарий NPM с именем `build-prod` для запуска `ng build --prod` и введите в `npm run build-prod` качестве пользовательской команды. Если оставить это поле пустым, рабочий процесс попытается выполнить команды `npm run build` или `npm run build:Azure`.  |
| `api_build_command` | Определяет пользовательскую команду, выполняемую во время развертывания приложения API Функций Azure. |

## <a name="route-file-location"></a>Расположение файла маршрута

Рабочий процесс можно настроить для поиска файла [routes.json](routes.md) в любой папке в репозитории. Следующие свойства можно определить в разделе задания `with`.

| Свойство            | Описание |
|---------------------|-------------|
| `routes_location` | Определяет расположение каталога, в котором найден файл _routes.json_. Это расположение задается относительно корня репозитория. |

 Явное расположение файла _routes.json_ особенно важно, если шаг сборки внешней платформы не перемещает этот файл в `output_location` по умолчанию.

## <a name="environment-variables"></a>Переменные среды

Вы можете задать переменные среды для сборки, используя `env` раздел конфигурации задания.

```yaml
jobs:
  build_and_deploy_job:
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.action != 'closed')
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
      - name: Build And Deploy
        id: builddeploy
        uses: Azure/static-web-apps-deploy@v0.0.1-preview
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          ###### Repository/Build Configurations
          app_location: "/"
          api_location: "api"
          output_location: "public"
          ###### End of Repository/Build Configurations ######
        env: # Add environment variables here
          HUGO_VERSION: 0.58.0
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Проверка запросов на вытягивание в подготовительных средах](review-publish-pull-requests.md)
