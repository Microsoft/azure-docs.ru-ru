---
title: Учебник по JavaScript. Развертывание веб-сайта с поддержкой поиска
titleSuffix: Azure Cognitive Search
description: Развертывание веб-сайта с поддержкой поиска в статическом веб-приложении Azure.
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 03/18/2021
ms.custom: devx-track-js
ms.devlang: javascript
ms.openlocfilehash: a49ede283899cec42898672f5a376221265dea10
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104723571"
---
# <a name="3---deploy-the-search-enabled-website"></a>3\. Развертывание веб-сайта с поддержкой поиска

Выполните развертывание веб-сайта с поддержкой поиска в качестве статического веб-приложения Azure. Для этого развертывания используется приложение React и приложение-функция.  

Статическое веб-приложение извлекает сведения и файлы для развертывания из GitHub с помощью развилки репозитория примеров.  

## <a name="create-a-static-web-app-in-visual-studio-code"></a>Создание статического веб-приложения в Visual Studio Code

1. Выберите **Azure** на панели действий, а затем на боковой панели — **Статические веб-приложения**. 
1. Щелкните правой кнопкой мыши имя подписки и выберите **Create Static Web App (Advanced)** (Создать статическое веб-приложение (дополнительно)).    

    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-create-static-web-app-resource-advanced.png" alt-text="Щелкните правой кнопкой мыши имя подписки и выберите **Create Static Web App (Advanced)** (Создать статическое веб-приложение (дополнительно)).":::

1. При появлении запросов введите представленные ниже сведения:

    |prompt|ВВОД|
    |--|--|
    |How do you want to create a Static Web App? (Как вы хотите создать статическое веб-приложение?)|Используйте имеющийся репозиторий GitHub|
    |Выберите организацию|Выберите _собственный_ псевдоним GitHub в качестве организации.|
    |Выберите репозиторий|В соответствующем списке выберите **azure-search-javascript-samples**. |
    |Choose branch of repository (Выберите ветвь репозитория)|Выберите из списка **master**. |
    |Enter the name for the new Static Web App (Введите имя нового статического веб-приложения).|Создайте уникальное имя ресурса. Например, вы можете добавить свое имя к имени репозитория: `joansmith-azure-search-javascript-samples`. |
    |Select a resource group for new resources (Выберите группу ресурсов для новых ресурсов).|Выберите группу ресурсов, созданную для работы с этим учебником.|
    |Choose build preset to configure default project structure (Выберите предустановку сборки, чтобы настроить структуру проекта по умолчанию).|выберите **Пользовательское**.|
    |Select the location of your application code (Выберите расположение для кода приложения)|`search-website`|
    |Select the location of your Azure Function code (Выберите расположение для кода Функций Azure)|`search-website/api`|
    |Enter the path of your build output... (Введите путь к выходным данным сборки...)|build;|
    |Select a location for new resources (Выберите расположение для новых ресурсов).|Выберите ближайший регион.|

1. Ресурс создан. Выберите **Open Actions in GitHub** (Открыть действия в GitHub) из раздела "Уведомления". Откроется окно браузера с указанием на разветвленный репозиторий. 

    В списке действий указывается, что веб-приложение, клиент и функции успешно отправлены в статическое веб-приложение Azure. 

    Дождитесь завершения сборки и развертывания, прежде чем продолжить. Это может занять несколько минут.

## <a name="get-cognitive-search-query-key-in-visual-studio-code"></a>Получение ключа запроса Когнитивного поиска в Visual Studio Code

1. В Visual Studio Code откройте [Панель действий](https://code.visualstudio.com/docs/getstarted/userinterface) и выберите значок Azure. 

1. На боковой панели выберите подписку Azure в области **Azure: Когнитивный поиск**. Щелкните правой кнопкой мыши ресурс поиска и выберите **Copy Query Key** (Копировать ключ запроса). 

    :::image type="content" source="./media/tutorial-javascript-create-load-index/visual-studio-code-copy-query-key.png" alt-text="На боковой панели выберите подписку Azure в области **Azure: Когнитивный поиск**, щелкните правой кнопкой мыши ресурс поиска и выберите **Copy Query Key** (Копировать ключ запроса).":::

1. Сохраните этот ключ запроса, так как его необходимо использовать в следующем разделе. Ключ запроса может запрашивать ваш индекс. 

## <a name="add-configuration-settings-in-visual-studio-code"></a>Добавление параметров конфигурации в Visual Studio Code

Приложение-функция Azure не будет возвращать данные поиска, пока в настройках не будут активированы секреты поиска. 

1. Выберите **Azure** на панели действий, а затем на боковой панели — **Статические веб-приложения**. 
1. Разверните новое статическое веб-приложение так, чтобы отображался раздел **Параметры приложения**.
1. Щелкните правой кнопкой мыши **Параметры приложения**, а затем выберите **Добавление нового параметра**.

    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-static-web-app-configure-settings.png" alt-text="Щелкните правой кнопкой мыши **Параметры приложения**, а затем выберите **Добавление нового параметра**.":::

1. Добавьте следующие параметры:

    |Параметр|Значение ресурса поиска|
    |--|--|
    |SearchApiKey|Ключ запроса поиска|
    |SearchServiceName|Имя ресурса поиска|
    |SearchIndexName|`good-books`|
    |SearchFacets|`authors*,language_code`|

## <a name="use-search-in-your-static-web-app"></a>Использование функции поиска в статическом веб-приложении

1. В Visual Studio Code откройте [Панель действий](https://code.visualstudio.com/docs/getstarted/userinterface) и выберите значок Azure.
1. На боковой панели **щелкните правой кнопкой мыши подписку Azure** в области `Static web apps` и найдите статическое веб-приложение, созданное для работы с этим учебником.
1. Щелкните правой кнопкой мыши имя статического веб-приложения и выберите **Browse site** (Обзор сайта).
    
    :::image type="content" source="media/tutorial-javascript-create-load-index/visual-studio-code-browse-static-web-app.png" alt-text="Щелкните правой кнопкой мыши имя статического веб-приложения и выберите **Browse site** (Обзор сайта).":::    

1. Выберите **Открыть** во всплывающем диалоговом окне.
1. На панели поиска веб-сайта введите поисковый запрос, например `code`, _slowly_, чтобы функция предложений предложила названия книг. Выберите предложение или продолжайте ввод собственного запроса. Завершив поисковый запрос, нажмите клавишу ВВОД. 
1. Просмотрите результаты и выберите одну из книг, чтобы ознакомиться с дополнительными сведениями. 

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим учебником, удалите группу ресурсов.

1. В Visual Studio Code откройте [Панель действий](https://code.visualstudio.com/docs/getstarted/userinterface) и выберите значок Azure. 

1. На боковой панели **щелкните правой кнопкой мыши подписку Azure** в области `Resource Groups` и найдите группу ресурсов, созданную для работы с этим учебником.
1. Щелкните правой кнопкой мыши имя группы ресурсов, а затем выберите **Удалить**.
    Это приведет к удалению ресурсов поиска и ресурсов статического веб-приложения.
1. Если вы больше не хотите использовать развилку GitHub для примера, не забудьте удалить ее на GitHub. Перейдите к разделу **Параметры** развилки и удалите ее. 


## <a name="next-steps"></a>Дальнейшие действия

* [Общие сведения об интеграции поиска для веб-сайта с поддержкой поиска](tutorial-javascript-search-query-integration.md)
