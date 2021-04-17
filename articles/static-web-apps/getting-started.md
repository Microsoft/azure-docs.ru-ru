---
title: Краткое руководство. Создание первого статического сайта с помощью службы "Статические веб-приложения Azure"
description: Узнайте, как развернуть статический сайт в службе "Статические веб-приложения Azure".
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: cshoe
ms.openlocfilehash: 335f78bba24947b1b6c3d6132bc38f237b3298b9
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449214"
---
# <a name="quickstart-building-your-first-static-site-with-azure-static-web-apps"></a>Краткое руководство. Создание первого статического сайта с помощью службы "Статические веб-приложения Azure"

Служба "Статические веб-приложения Azure" публикует веб-сайты, создавая приложения из репозитория кода. В этом кратком руководстве рассказывается о том, как развернуть приложение в службе "Статические веб-приложения Azure" с помощью расширения Visual Studio Code.

Если у вас еще нет подписки Azure, создайте [бесплатную пробную учетную запись](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Предварительные требования

- [GitHub](https://github.com) .
- Учетная запись [Azure](https://portal.azure.com)
- [Visual Studio Code](https://code.visualstudio.com)
- [Расширение "Статические веб-приложения"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) для Visual Studio Code
- [установите Git](https://www.git-scm.com/downloads);

[!INCLUDE [create repository from template](../../includes/static-web-apps-get-started-create-repo.md)]

[!INCLUDE [clone the repository](../../includes/static-web-apps-get-started-clone-repo.md)]

Затем откройте Visual Studio Code и последовательно выберите **Файл > Открыть папку**, чтобы открыть репозиторий, который вы клонировали на компьютер в редакторе.

## <a name="create-a-static-web-app"></a>Создание статического веб-приложения

1. В Visual Studio Code на панели действий выберите логотип Azure, чтобы открыть окно расширений Azure.

    :::image type="content" source="media/getting-started/extension-azure-logo.png" alt-text="Логотип Azure":::

    > [!NOTE]
    > Необходимо войти в Azure и на GitHub. Если вы еще не вошли в Azure и на GitHub из Visual Studio Code, расширение предложит вам сделать это в процессе создания.

1. В разделе _Статические веб-приложения_ выберите **знак плюс**.

    :::image type="content" source="media/getting-started/extension-create-button.png" alt-text="Имя приложения":::

1. В верхней части редактора откроется палитра команд и будет предложено присвоить имя приложению.

    Введите **my-first-static-web-app** и нажмите клавишу **ВВОД**.

    :::image type="content" source="media/getting-started/extension-create-app.png" alt-text="Создание Статического веб-приложения":::

1. Выберите предустановки, соответствующие типу приложения.

    # <a name="no-framework"></a>[Без платформы](#tab/vanilla-javascript)
    :::image type="content" source="media/getting-started/extension-presets-no-framework.png" alt-text="Предустановки приложений: нет платформы":::

    Введите **./** в качестве расположения для файлов приложения.

    :::image type="content" source="media/getting-started/extension-app-location.png" alt-text="Расположение файлов приложения":::

    Выберите **Пропустить сейчас** в качестве расположения для API функций Azure.

    :::image type="content" source="media/getting-started/extension-api-location.png" alt-text="Расположение API":::

    Введите **./** в качестве расположения выходных данных сборки.

    :::image type="content" source="media/getting-started/extension-build-location.png" alt-text="Расположение выходных данных сборки приложения":::

    # <a name="angular"></a>[Angular](#tab/angular)

    :::image type="content" source="media/getting-started/extension-presets-angular.png" alt-text="Предустановки приложений: Angular":::

    # <a name="react"></a>[React](#tab/react)

    :::image type="content" source="media/getting-started/extension-presets-react.png" alt-text="Предустановки приложений: React":::

    # <a name="vue"></a>[Vue](#tab/vue)

    :::image type="content" source="media/getting-started/extension-presets-vue.png" alt-text="Предустановки приложений: Vue":::

    ---

1. Выберите ближайшее расположение и нажмите клавишу **ВВОД**.

    :::image type="content" source="media/getting-started/extension-location.png" alt-text="Расположение ресурса":::

1. После создания приложения в Visual Studio Code отобразится уведомление с подтверждением.

    :::image type="content" source="media/getting-started/extension-confirmation.png" alt-text="Подтверждение создания":::

    Затем нажмите кнопку **Открыть действия в GitHub**. На этой странице отображается состояние сборки приложения.

    После завершения действия GitHub можно перейти на опубликованный веб-сайт.

1. Чтобы просмотреть веб-сайт в браузере, щелкните правой кнопкой мыши проект в расширении "Статические веб-приложения" и выберите **Просмотр сайта**.

    :::image type="content" source="media/getting-started/extension-browse-site.png" alt-text="Просмотр сайта":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, вы можете удалить экземпляр службы "Статические веб-приложения Azure" с помощью расширения.

В окне обозревателя Visual Studio Code вернитесь в раздел _Статические веб-приложения_, щелкните правой кнопкой мыши **my-first-static-web-app** и выберите **Удалить**.

:::image type="content" source="media/getting-started/extension-delete.png" alt-text="Удаление приложения":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление API](add-api.md)
