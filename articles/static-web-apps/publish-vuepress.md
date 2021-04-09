---
title: Руководство по Публикация сайта VuePress в службе "Статические веб-приложения Azure"
description: В этом учебнике показано, как развернуть приложение VuePress в службе "Статические веб-приложения Azure".
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: aapowell
ms.custom: devx-track-js
ms.openlocfilehash: 6f0616df885a7f8fcd76337c810bc368aa02f3c8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100650457"
---
# <a name="tutorial-publish-a-vuepress-site-to-azure-static-web-apps-preview"></a>Руководство по Публикация сайта VuePress в предварительной версии службы "Статические веб-приложения Azure"

В этой статье показано, как создать и развернуть веб-приложение [VuePress](https://vuepress.vuejs.org/) в службе [Статические веб-приложения Azure](overview.md). Конечным результатом будет новое приложение службы "Статические веб-приложения Azure" со связанными действиями GitHub, позволяющее управлять процессом сборки и публикации приложения.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Создание приложения VuePress
> - Настройка Статических веб-приложений Azure
> - Развертывание приложения VuePress в Azure

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Если ее нет, можно создать [учетную запись бесплатно](https://azure.microsoft.com/free/).
- Учетная запись GitHub. Если ее нет, можно создать [учетную запись бесплатно](https://github.com/join).
- Установленный компонент [Node.js](https://nodejs.org).

## <a name="create-a-vuepress-app"></a>Создание приложения VuePress

Создайте приложения VuePress из интерфейса командной строки (CLI).

1. Создайте новую папку для приложения VuePress.

   ```bash
   mkdir static-site
   ```

1. Добавьте в нее файл _README.md_.

   ```bash
   echo '# Hello From VuePress' > README.md
   ```

1. Инициализируйте файл _package.json_.

   ```bash
   npm init -y
   ```

1. Добавьте VuePress в качестве `devDependency`.

   ```bash
   npm install --save-dev vuepress
   ```

1. Откройте файл _package.json_ в текстовом редакторе и добавьте команду сборки в раздел [`scripts`](https://docs.npmjs.com/cli-commands/run-script.html).

   ```json
   ...
   "scripts": {
       "build": "vuepress build"
   }
   ...
   ```

1. Создайте файл _.gitignore_, чтобы исключить папку _node\_modules_.

    ```bash
    echo 'node_modules' > .gitignore
    ```

1. Инициализируйте репозиторий Git.

   ```bash
    git init
    git add -A
    git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Отправьте приложение в GitHub.

Для подключения к Статическим веб-приложениям Azure требуется репозиторий на сайте GitHub. Ниже показано, как создать репозиторий для своего сайта.

1. Создайте пустой репозиторий GitHub (не создавая файл сведений) со страницы [https://github.com/new](https://github.com/new) с именем **vuepress-static-app**.

1. Добавьте репозиторий GitHub в локальный репозиторий в качестве удаленного. В следующей команде не забудьте добавить имя пользователя GitHub вместо заполнителя `<YOUR_USER_NAME>`.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/vuepress-static-app
   ```

1. Отправьте локальный репозиторий на сайт GitHub.

   ```bash
   git push --set-upstream origin main
   ```

## <a name="deploy-your-web-app"></a>Развертывание веб-приложения

Ниже описано, как создать новое приложение службы "Статические веб-приложения" и развернуть его в рабочей среде.

### <a name="create-the-application"></a>Создание приложения

1. Перейдите на [портал Azure](https://portal.azure.com).
1. Щелкните **Создать ресурс**.
1. Найдите **Статические веб-приложения**.
1. Выберите **Статические веб-приложения (предварительная версия)** .
1. Нажмите кнопку **Создать**.

   :::image type="content" source="./media/publish-vuepress/create-in-portal.png" alt-text="Создание ресурса Статические веб-приложения (предварительная версия) на портале":::

1. Для параметра **Подписка** подтвердите предложенный вариант или выберите другой из раскрывающегося списка.

1. Для параметра _Группа ресурсов_ выберите **Создать**. В разделе _Новое имя группы ресурсов_ введите **vuepress-static-app** и нажмите кнопку **ОК**.

1. В поле **Имя** введите имя приложения. Допустимые символы: `a-z`, `A-Z`, `0-9` и `-`.

1. В поле _Регион_ выберите ближайший доступный регион.

1. В поле _Номер SKU_ выберите **Бесплатный**.

   :::image type="content" source="./media/publish-vuepress/basic-app-details.png" alt-text="Заполненные сведениями поля":::

1. Нажмите кнопку **Войти по учетным данным GitHub**.

1. В поле **Организация** выберите ту, в которой был создан репозиторий.

1. Выберите **vuepress-static-app** в качестве _репозитория_.

1. В поле _Ветвь_ выберите **main**.

   :::image type="content" source="./media/publish-vuepress/completed-github-info.png" alt-text="Поля, заполненные сведениями о GitHub":::

### <a name="build"></a>Сборка

Затем добавьте параметры конфигурации, используемые процессом сборки для компиляции приложения. Следующие параметры позволяют настроить файл рабочего процесса для действия GitHub.

1. Нажмите кнопку **Next: Build >** (Далее: сборка >) для изменения конфигурации сборки.

1. В поле _App location_ (Расположение приложения) выберите значение **/** .

1. В поле _App artifact location_ (Расположение артефакта приложения) задайте значение **.vuepress/dist**.

Поле _API location_ (Расположения API) заполнять не требуется, так как вы в данный момент не развертываете API.

   :::image type="content" source="./media/publish-vuepress/build-details.png" alt-text="Параметры сборки":::

### <a name="review-and-create"></a>Просмотр и создание

1. Нажмите кнопку **Просмотр и создание** и убедитесь, что вы указали правильные сведения.

1. Щелкните **Создать**, чтобы начать создание приложения службы "Статические веб-приложения Azure", а также подготовить действие GitHub к развертыванию.

1. После завершения развертывания щелкните **Перейти к ресурсу**.

1. На экране ресурсов щелкните ссылку на _URL-адрес_, чтобы открыть развернутое приложение. Для завершения действия GitHub может потребоваться подождать одну или две минуты.

   :::image type="content" source="./media/publish-vuepress/deployed-app.png" alt-text="Развернутое приложение":::

### <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавить личный домен](custom-domain.md)
