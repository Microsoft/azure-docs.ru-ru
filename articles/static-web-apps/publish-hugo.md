---
title: Руководство по Публикация сайта Hugo в службе "Статические веб-приложения Azure"
description: Узнайте, как развернуть приложение Hugo в службе "Статические веб-приложения Azure".
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: aapowell
ms.openlocfilehash: 4539c32a367bb0974212d989176a96b530da21a4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100652361"
---
# <a name="tutorial-publish-a-hugo-site-to-azure-static-web-apps-preview"></a>Руководство по Публикация сайта Hugo в предварительной версии службы "Статические веб-приложения Azure"

В этой статье показано, как создать и развернуть веб-приложение [Hugo](https://gohugo.io/) в службе [Статические веб-приложения Azure](overview.md). В результате вы получите новый экземпляр Статических веб-приложений Azure со связанным компонентом GitHub Actions для управления процессом сборки и публикации приложения.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Создание приложения Hugo
> - Настройка Статических веб-приложений Azure
> - Развертывание приложения Hugo в Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Если ее нет, можно создать [учетную запись бесплатно](https://azure.microsoft.com/free/).
- Учетная запись GitHub. Если ее нет, можно создать [учетную запись бесплатно](https://github.com/join).

## <a name="create-a-hugo-app"></a>Создание приложения Hugo

Создайте приложение Hugo с помощью интерфейса командной строки (CLI) Hugo.

1. Выполните инструкции из [руководства по установке](https://gohugo.io/getting-started/installing/) Hugo в вашей операционной системе.

1. Откройте окно терминала.

1. Запустите интерфейс командной строки Hugo, чтобы создать новое приложение.

   ```bash
   hugo new site static-app
   ```

1. Перейдите к только что созданному приложению.

   ```bash
   cd static-app
   ```

1. Инициализируйте репозиторий Git.

   ```bash
    git init
   ```

1. Затем добавьте тему на сайт, установив ее в качестве подмодуля Git и указав ее в файле конфигурации Hugo.

   ```bash
   git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
   echo 'theme = "ananke"' >> config.toml
   ```

1. Зафиксируйте изменения.

   ```bash
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Отправьте приложение в GitHub.

Для подключения к Статическим веб-приложениям Azure требуется репозиторий на сайте GitHub. Ниже показано, как создать репозиторий для своего сайта.

1. Создайте пустой репозиторий GitHub (не создавая файл сведений) со страницы [https://github.com/new](https://github.com/new) с именем **hugo-static-app**.

1. Добавьте репозиторий GitHub в локальный репозиторий в качестве удаленного. В следующей команде не забудьте добавить имя пользователя GitHub вместо заполнителя `<YOUR_USER_NAME>`.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/hugo-static-app
   ```

1. Отправьте локальный репозиторий на сайт GitHub.

   ```bash
   git push --set-upstream origin main
   ```

## <a name="deploy-your-web-app"></a>Развертывание веб-приложения

Ниже описано, как создать новое статическое приложение сайта и развернуть его в рабочей среде.

### <a name="create-the-application"></a>Создание приложения

1. Перейдите на [портал Azure](https://portal.azure.com).
1. Щелкните **Создать ресурс**.
1. Найдите **Статические веб-приложения**.
1. Выберите **Статические веб-приложения (предварительная версия)** .
1. Нажмите кнопку **Создать**.

   :::image type="content" source="./media/publish-hugo/create-in-portal.png" alt-text="Создание ресурса Статические веб-приложения Azure на портале":::

1. Для параметра **Подписка** подтвердите предложенный вариант или выберите другой из раскрывающегося списка.

1. Для параметра _Группа ресурсов_ выберите **Создать**. В разделе _Новое имя группы ресурсов_ введите **hugo-static-app** и нажмите кнопку **ОК**.

1. В поле **Имя** введите имя приложения. Допустимые символы: `a-z`, `A-Z`, `0-9` и `-`.

1. В поле _Регион_ выберите ближайший доступный регион.

1. В поле _Номер SKU_ выберите **Бесплатный**.

   :::image type="content" source="./media/publish-hugo/basic-app-details.png" alt-text="Заполненные сведениями поля":::

1. Нажмите кнопку **Войти по учетным данным GitHub**.

1. В поле **Организация** выберите ту, в которой был создан репозиторий.

1. Выберите **hugo-static-app** в качестве _репозитория_.

1. В поле _Ветвь_ выберите **main**.

   :::image type="content" source="./media/publish-hugo/completed-github-info.png" alt-text="Поля, заполненные сведениями о GitHub":::

### <a name="build"></a>Сборка

Затем добавьте параметры конфигурации, используемые процессом сборки для компиляции приложения. Следующие параметры позволяют настроить файл рабочего процесса для действия GitHub.

1. Нажмите кнопку **Next: Build >** (Далее: сборка >) для изменения конфигурации сборки.

1. В поле _App location_ (Расположение приложения) выберите значение **/** .

1. В поле _App artifact location_ (Расположение артефакта приложения) задайте значение **общедоступное**.

   Поле _API location_ (Расположения API) заполнять не требуется, так как вы в данный момент не развертываете API.

### <a name="review-and-create"></a>Просмотр и создание

1. Нажмите кнопку **Просмотр и создание** и убедитесь, что вы указали правильные сведения.

1. Щелкните **Создать**, чтобы начать создание приложения службы "Статические веб-приложения Azure", а также подготовить действие GitHub к развертыванию.

1. Дождитесь завершения действия GitHub.

1. На портале Azure в окне _Обзор_ для только что созданного ресурса "Статические веб-приложения Azure" щелкните ссылку на _URL-адрес_, чтобы открыть развернутое приложение.

   :::image type="content" source="./media/publish-hugo/deployed-app.png" alt-text="Развернутое приложение":::

#### <a name="custom-hugo-version"></a>Пользовательская версия Hugo

При создании статического веб-приложения создается [файл рабочего процесса](./github-actions-workflow.md), который содержит параметры конфигурации публикации для приложения. Вы можете задать определенную версию Hugo в файле рабочего процесса, указав значение `HUGO_VERSION` в разделе `env`. В следующем примере конфигурации показано, как задать определенную версию для Hugo.

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
          repo_token: ${{ secrets.GITHUB_TOKEN }} # Used for Github integrations (i.e. PR comments)
          action: "upload"
          ###### Repository/Build Configurations - These values can be configured to match you app requirements. ######
          # For more information regarding Static Web App workflow configurations, please visit: https://aka.ms/swaworkflowconfig
          app_location: "/" # App source code path
          api_location: "api" # Api source code path - optional
          output_location: "public" # Built app content directory - optional
          ###### End of Repository/Build Configurations ######
        env:
          HUGO_VERSION: 0.58.0
```

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавить личный домен](custom-domain.md)
