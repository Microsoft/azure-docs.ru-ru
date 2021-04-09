---
title: Руководство по Публикация сайта Gatsby в службе "Статические веб-приложения Azure"
description: В этом учебнике показано, как развернуть приложение Gatsby в службе "Статические веб-приложения Azure".
services: static-web-apps
author: aaronpowell
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 05/08/2020
ms.author: aapowell
ms.custom: devx-track-js
ms.openlocfilehash: 4430ed34858077b13b4fec69756c1c7e9f3ef7ac
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100652385"
---
# <a name="tutorial-publish-a-gatsby-site-to-azure-static-web-apps-preview"></a>Руководство по Публикация сайта Gatsby в предварительной версии службы "Статические веб-приложения Azure"

В этой статье показано, как создать и развернуть веб-приложение [Gatsby](https://gatsbyjs.org) в службе [Статические веб-приложения Azure](overview.md). Конечным результатом будет новый сайт Статических веб-приложений (со связанными действиями GitHub), позволяющий управлять процессом сборки и публикации приложения.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Создание приложения Gatsby
> - Настройка сайта Статических веб-приложений Azure
> - Развертывание приложения Gatsby в Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Если ее нет, можно создать [учетную запись бесплатно](https://azure.microsoft.com/free/).
- Учетная запись GitHub. Если ее нет, можно создать [учетную запись бесплатно](https://github.com/join).
- Установленный компонент [Node.js](https://nodejs.org).

## <a name="create-a-gatsby-app"></a>Создание приложения Gatsby

Создайте приложение Gatsby с помощью интерфейса командной строки (CLI) Gatsby.

1. Откройте окно терминала.
1. Используйте программу [npx](https://www.npmjs.com/package/npx) для создания приложения с помощью интерфейса командной строки Gatsby. Это может занять несколько минут.

   ```bash
   npx gatsby new static-web-app
   ```

1. Перейдите к только что созданному приложению.

   ```bash
   cd static-web-app
   ```

1. Инициализируйте репозиторий Git.

   ```bash
   git init
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Отправьте приложение в GitHub.

Для создания нового ресурса Статических веб-приложений Azure необходим репозиторий на сайте GitHub.

1. Создайте пустой репозиторий GitHub (не создавая файл сведений) со страницы [https://github.com/new](https://github.com/new) с именем **gatsby-static-web-app**.

1. Затем добавьте только что созданный репозиторий GitHub в локальный репозиторий в качестве удаленного. В следующей команде не забудьте добавить имя пользователя GitHub вместо заполнителя `<YOUR_USER_NAME>`.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/gatsby-static-web-app
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

   :::image type="content" source="./media/publish-gatsby/create-in-portal.png" alt-text="Создание ресурса Статические веб-приложения (предварительная версия) на портале":::

1. Для параметра _Подписка_ подтвердите предложенный вариант или выберите другой из раскрывающегося списка.

1. Для параметра _Группа ресурсов_ выберите **Создать**. В разделе _Новое имя группы ресурсов_ введите **gatsby-static-web-app** и нажмите кнопку **ОК**.

1. В поле **Имя** введите имя приложения. Допустимые символы: `a-z`, `A-Z`, `0-9` и `-`.

1. В поле _Регион_ выберите ближайший доступный регион.

1. В поле _Номер SKU_ выберите **Бесплатный**.

   :::image type="content" source="./media/publish-gatsby/basic-app-details.png" alt-text="Заполненные сведениями поля":::

1. Нажмите кнопку **Войти по учетным данным GitHub**.

1. В поле **Организация** выберите ту, в которой был создан репозиторий.

1. Выберите **gatsby-static-web-app** в качестве _репозитория_.

1. В поле _Ветвь_ выберите **main**.

   :::image type="content" source="./media/publish-gatsby/completed-github-info.png" alt-text="Поля, заполненные сведениями о GitHub":::

### <a name="build"></a>Сборка

Затем добавьте параметры конфигурации, используемые процессом сборки для компиляции приложения.

1. Нажмите кнопку **Next: Build >** (Далее: сборка >) для изменения конфигурации сборки.

1. Чтобы настроить параметры этапа в GitHub Actions, задайте в поле _App location_ (Расположение приложения) значение **/** .

1. В поле _App artifact location_ (Расположение артефакта приложения) задайте значение **общедоступное**.

   Поле _API location_ (Расположения API) заполнять не требуется, так как вы в данный момент не развертываете API.

   :::image type="content" source="./media/publish-gatsby/build-details.png" alt-text="Параметры сборки":::

### <a name="review-and-create"></a>Просмотр и создание

1. Нажмите кнопку **Просмотр и создание** и убедитесь, что вы указали правильные сведения.

1. Щелкните **Создать**, чтобы начать создание статического веб-приложения Службы приложений, а также подготовить действие GitHub к развертыванию.

1. После завершения развертывания щелкните **Перейти к ресурсу**.

1. На экране ресурсов щелкните ссылку на _URL-адрес_, чтобы открыть развернутое приложение. Для завершения действия GitHub может потребоваться подождать одну или две минуты.

   :::image type="content" source="./media/publish-gatsby/deployed-app.png" alt-text="Развернутое приложение":::

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавить личный домен](custom-domain.md)
