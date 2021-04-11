---
title: Учебник. Публикация Статических веб-приложений Azure с помощью Azure DevOps
description: Сведения о том, как использовать Azure DevOps для публикации Статических веб-приложений Azure.
services: static-web-apps
author: scubaninja
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 03/23/2021
ms.author: apedward
ms.openlocfilehash: 472cf7b69078b3247c393ff65139bc29e5683a32
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105639374"
---
# <a name="tutorial-publish-azure-static-web-apps-with-azure-devops"></a>Учебник. Публикация Статических веб-приложений Azure с помощью Azure DevOps

В этой статье демонстрируется процесс развертывания [Статических веб-приложений Azure](./overview.md) с помощью [Azure DevOps](https://dev.azure.com/).

Из этого руководства вы узнаете, как выполнять такие задачи.

- настройка сайта Статических веб-приложений Azure;
- создание конвейера Azure DevOps для сборки и публикации статического веб-приложения.

## <a name="prerequisites"></a>Предварительные требования

- **Действующая учетная запись Azure.** Если ее нет, можно [создать учетную запись бесплатно](https://azure.microsoft.com/free/).
- **Проект Azure DevOps.** Если у вас его нет, можно [создать проект бесплатно](https://azure.microsoft.com/pricing/details/devops/azure-devops-services/).
- **Конвейер Azure DevOps.** Если вам нужна помощь в работе с ним, изучите статью о [создании первого конвейера](https://docs.microsoft.com/azure/devops/pipelines/create-first-pipeline?view=azure-devops&preserve-view=true).

## <a name="create-a-static-web-app-in-an-azure-devops-repository"></a>Создание статического веб-приложения в репозитории Azure DevOps

  > [!NOTE]
  > Если в вашем репозитории уже есть готовое приложение, переходите сразу к следующему разделу.

1. Откройте нужный репозиторий Azure DevOps.

1. Щелкните **Import** (Импортировать), чтобы импортировать пример приложения.
  
    :::image type="content" source="media/publish-devops/devops-repo.png" alt-text="Репозиторий DevOps":::

1. В поле **Clone URL** (URL-адрес клонирования) введите `https://github.com/staticwebdev/vanilla-api.git`.

1. Выберите **Импорт**.

## <a name="create-a-static-web-app"></a>Создание статического веб-приложения

1. Перейдите на [портал Azure](https://portal.azure.com).

1. Выберите **Создать ресурс**.

1. Выполните поиск по запросу **Статические веб-приложения**.

1. Выберите **Статические веб-приложения (предварительная версия)** .

1. Щелкните **Создать**.

1. В разделе _Сведения о развертывании_ обязательно выберите **Другие**. Это нужно для того, чтобы использовать код из репозитория Azure DevOps.

    :::image type="content" source="media/publish-devops/create-resource.png" alt-text="Сведения о развертывании — другие":::

1. Когда развертывание успешно завершится, перейдите к созданному ресурсу Статических веб-приложений Azure.

1. Щелкните **Manage deployment token** (Управление маркером развертывания).

1. Скопируйте **маркер развертывания** и вставьте его в текстовый редактор, чтобы применить далее на другом экране.

    > [!NOTE]
    > Промежуточное сохранение значения важно, поскольку в следующих шагах вы будете копировать и вставлять другие значения.

    :::image type="content" source="media/publish-devops/deployment-token.png" alt-text="Маркер развертывания":::

## <a name="create-the-pipeline-task-in-azure-devops"></a>Создание задания конвейера в Azure DevOps

1. Перейдите к репозиторию Azure DevOps, который вы недавно создали.

1. Щелкните **Set up build** (Настроить сборку).

    :::image type="content" source="media/publish-devops/azdo-build.png" alt-text="Конвейер сборки":::

1. На экране *Configure your pipeline* (Настройка конвейера) выберите **Starter pipeline** (Простейший конвейер).

    :::image type="content" source="media/publish-devops/configure-pipeline.png" alt-text="Настройка конвейера":::

1. Скопируйте и вставьте в этот конвейер следующий код YAML.

    ```yaml
    trigger:
      - main
    
    pool:
      vmImage: ubuntu-latest
    
    steps:
      - task: AzureStaticWebApp@0
        inputs:
          app_location: "/" 
          api_location: "api"
          output_location: ""
        env:
          azure_static_web_apps_api_token: $(deployment_token)
    ```

    > [!NOTE]
    > Если вы не используете наш пример приложения, измените значения `app_location`, `api_location` и `output_location` так, чтобы они соответствовали параметрам вашего приложения.

    [!INCLUDE [static-web-apps-folder-structure](../../includes/static-web-apps-folder-structure.md)]

    Значение `azure_static_web_apps_api_token` вы задаете и настраиваете самостоятельно.

1. Выберите элемент **Variables** (Переменные).

1. Создать новую переменную.

1. Присвойте переменной имя **deployment_token** (которое уже используется в рабочем потоке).

1. Скопируйте маркер развертывания, который вы ранее скопировали в текстовый редактор.

1. Вставьте этот маркер развертывания в поле _Value_ (Значение).

    :::image type="content" source="media/publish-devops/variable-token.png" alt-text="Переменная маркера":::

1. Установите флажок **Keep this value secret** (Скрыть это значение).

1. Щелкните **ОК**.

1. Щелкните **Save** (Сохранить), чтобы вернуться к коду YAML вашего конвейера.

1. Щелкните **Save and run** (Сохранить и выполнить), чтобы открыть диалоговое окно _Save and run_ (Сохранить и выполнить).

    :::image type="content" source="media/publish-devops/save-and-run.png" alt-text="Конвейер":::

1. Щелкните **Save and run** (Сохранить и выполнить), чтобы запустить конвейер.

1. Когда развертывание успешно завершится, перейдите к разделу **Overview** (Обзор) для Статических веб-приложений Azure, где содержатся ссылки на конфигурацию развертывания. Обратите внимание, что ссылка _Source_ (Источник) теперь указывает ветвь и расположение репозитория Azure DevOps.

1. Щелкните **URL**, чтобы открыть развернутый веб-сайт.

    :::image type="content" source="media/publish-devops/deployment-location.png" alt-text="Расположение развертывания":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка Статических веб-приложений Azure](./configuration.md)
