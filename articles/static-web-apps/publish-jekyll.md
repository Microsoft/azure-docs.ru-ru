---
title: Руководство по Публикация сайта Jekyll в службе "Статические веб-приложения Azure"
description: Узнайте, как развернуть приложение Jekyll в службе "Статические веб-приложения Azure".
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 06/08/2020
ms.author: cshoe
ms.openlocfilehash: 8c6764ad5b63aa2fde07326ab986404ea4312316
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99585182"
---
# <a name="tutorial-publish-a-jekyll-site-to-azure-static-web-apps-preview"></a>Руководство по Публикация сайта Jekyll в службе "Статические веб-приложения Azure" — предварительный просмотр

В этой статье показано, как создать и развернуть веб-приложение [Jekyll](https://jekyllrb.com/) в службе [Статические веб-приложения Azure](overview.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> - Создание веб-сайта Jekyll
> - Настройка Статических веб-приложений Azure
> - Развертывание приложения Jekyll в Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Установка [Jekyll](https://jekyllrb.com/docs/installation/)
  - При необходимости можно использовать подсистему Windows для Linux и следовать инструкциям Ubuntu.
- Учетная запись Azure с активной подпиской. Если ее нет, можно создать [учетную запись бесплатно](https://azure.microsoft.com/free/).
- Учетная запись GitHub. Если ее нет, можно создать [учетную запись бесплатно](https://github.com/join).

## <a name="create-jekyll-app"></a>Создание приложения Jekyll

Создайте приложение Jekyll с помощью интерфейса командной строки (CLI) Jekyll:

1. Из терминала запустите интерфейс командной строки (CLI) Jekyll, чтобы создать новое приложение.

   ```bash
   jekyll new static-app
   ```

1. Перейдите к только что созданному приложению.

   ```bash
   cd static-app
   ```

1. Инициализируйте новый репозиторий Git.

   ```bash
    git init
   ```

1. Зафиксируйте изменения.

   ```bash
   git add -A
   git commit -m "initial commit"
   ```

## <a name="push-your-application-to-github"></a>Отправьте приложение в GitHub.

Статические веб-приложения Azure используют GitHub, чтобы опубликовать ваш веб-сайт. Ниже показано, как создать репозиторий GitHub.

1. Создайте пустой репозиторий GitHub (не создавая файл сведений) со страницы [https://github.com/new](https://github.com/new) с именем **jekyll-azure-static**.

1. Добавьте репозиторий GitHub в локальный репозиторий в качестве удаленного. В следующей команде не забудьте добавить имя пользователя GitHub вместо заполнителя `<YOUR_USER_NAME>`.

   ```bash
   git remote add origin https://github.com/<YOUR_USER_NAME>/jekyll-static-app
   ```

1. Отправьте локальный репозиторий на сайт GitHub.

   ```bash
   git push --set-upstream origin main
   ```

   > [!NOTE]
   > Ваша ветвь git может называться иначе, чем `main`. Замените `main` в этой команде правильным значением.

## <a name="deploy-your-web-app"></a>Развертывание веб-приложения

Ниже описано, как создать новое статическое приложение сайта и развернуть его в рабочей среде.

### <a name="create-the-application"></a>Создание приложения

1. Перейдите на [портал Azure](https://portal.azure.com).

1. Щелкните **Создать ресурс**.

1. Выполните поиск по запросу **Статические веб-приложения**.

1. Выберите **Статические веб-приложения (предварительная версия)** .

1. Нажмите кнопку **Создать**.

1. Для параметра **Подписка** подтвердите предложенный вариант или выберите другой из раскрывающегося списка.

1. Для параметра _Группа ресурсов_ выберите **Создать**. В разделе _Новое имя группы ресурсов_ введите **jekyll-static-app** и нажмите кнопку **ОК**.

1. В поле _Имя_ введите имя приложения. Допустимые символы: `a-z`, `A-Z`, `0-9` и `-`.

1. В поле _Регион_ выберите ближайший доступный регион.

1. В поле _Номер SKU_ выберите **Бесплатный**.

    :::image type="content" source="./media/publish-jekyll/basic-app-details.png" alt-text="Заполненные сведениями поля":::

1. Нажмите кнопку **Войти по учетным данным GitHub**.

1. В поле **Организация** выберите ту, в которой был создан репозиторий.

1. Выберите **jekyll-static-app** в качестве _репозитория_.

1. В поле _Ветвь_ выберите **main**.

    :::image type="content" source="./media/publish-jekyll/completed-github-info.png" alt-text="Поля, заполненные сведениями о GitHub":::

### <a name="build"></a>Сборка

Затем добавьте параметры конфигурации, используемые процессом сборки для компиляции приложения. Следующие параметры позволяют настроить файл рабочего процесса для действия GitHub.

1. Нажмите кнопку **Next: Build >** (Далее: сборка >) для изменения конфигурации сборки.

1. В поле _App location_ (Расположение приложения) выберите **/_site**.

1. Оставьте поле _App artifact location_ (Расположения артефакта приложения) пустым.

   Поле _API location_ (Расположения API) заполнять не требуется, так как вы в данный момент не развертываете API.

### <a name="review-and-create"></a>Просмотр и создание

1. Нажмите кнопку **Просмотр и создание** и убедитесь, что вы указали правильные сведения.

1. Щелкните **Создать**, чтобы начать создание приложения службы "Статические веб-приложения Azure", а также подготовить действие GitHub к развертыванию.

1. Сначала развертывание завершится сбоем, так как для файла рабочего процесса требуются некоторые параметры, относящиеся к Jekyll. Чтобы добавить параметры, перейдите к терминалу и зафиксируйте изменения с помощью действия GitHub на компьютере.

   ```bash
   git pull
   ```

1. Откройте приложение Jekyll в текстовом редакторе и откройте файл _.github/workflows/azure-pages-<WORKFLOW_NAME>.yml_.

1. Перед строкой `- name: Build And Deploy` добавьте следующий блок параметров.

    ```yml
    - name: Set up Ruby
      uses: ruby/setup-ruby@v1.59.1
      with:
        ruby-version: 2.6
    - name: Install dependencies
      run: bundle install
    - name: Jekyll build
      run: jekyll build
    ```

1. Зафиксируйте обновленный рабочий процесс и отправьте его в GitHub.

    ```bash
    git add -A
    git commit -m "Updating GitHub Actions workflow"
    git push
    ```

1. Дождитесь завершения действия GitHub.

1. В окне _Обзор_ портала Azure щелкните ссылку на _URL-адрес_, чтобы открыть развернутое приложение.

   :::image type="content" source="./media/publish-jekyll/deployed-app.png" alt-text="Развернутое приложение":::

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cleanup-resource](../../includes/static-web-apps-cleanup-resource.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавить личный домен](custom-domain.md)
