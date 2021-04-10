---
title: Краткое руководство. Создание первой функции в Azure с помощью Visual Studio
description: Из этого краткого руководства вы узнаете, как создать и опубликовать Функцию Azure, активируемую HTTP-запросом, с помощью Visual Studio.
ms.assetid: 82db1177-2295-4e39-bd42-763f6082e796
ms.topic: quickstart
ms.date: 09/30/2020
ms.custom: devx-track-csharp, mvc, devcenter, vs-azure, 23113853-34f2-4f
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./functions-create-your-first-function-visual-studio-uiex
ms.openlocfilehash: d691ee60f624f75c89e44e905e5343bf9c71fd4d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101701421"
---
# <a name="quickstart-create-your-first-function-in-azure-using-visual-studio"></a>Краткое руководство. Создание первой функции в Azure с помощью Visual Studio

Из этой статьи вы узнаете, как создать функцию класса C# на основе библиотеки, которая отвечает на HTTP-запросы, используя Visual Studio. После тестирования кода в локальной среде его необходимо развернуть в бессерверной среде Функций Azure.  

Выполнение этого краткого руководства предполагает небольшую дополнительную плату в несколько центов США в учетной записи Azure.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сначала установите [Visual Studio 2019](https://azure.microsoft.com/downloads/). Во время установки обязательно выберите рабочую нагрузку **Разработка для Azure**. Если вы хотите создать проект "Функции Azure" с помощью Visual Studio 2017, необходимо сначала установить [новейшие инструменты Функций Azure](functions-develop-vs.md#check-your-tools-version).

![Установка Visual Studio с рабочей нагрузкой разработки в Azure](media/functions-create-your-first-function-visual-studio/functions-vs-workloads.png)

Если у вас еще нет [подписки Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/dotnet/), прежде чем начать работу.

## <a name="create-a-function-app-project"></a>Создание проекта приложения-функции

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

Visual Studio создает проект и класс, содержащий стандартный код для типа функции триггера HTTP. Стандартный код отправляет HTTP-ответ, включающий в себя значение из текста или строки запроса. Атрибут `HttpTrigger` указывает, что функция вызывается HTTP-запросом. 

## <a name="rename-the-function"></a>Переименование функции

Атрибут метода `FunctionName` задает имя функции. По умолчанию генерируется значение `Function1`. Поскольку набор средств не позволяет переопределить имя функции по умолчанию при создании проекта, уделите минуту на то, чтобы создать более подходящее имя для класса функции, файла или метаданных.

1. В **проводнике** щелкните правой кнопкой мыши файл Function1.cs и переименуйте его в `HttpExample.cs`.

1. В коде переименуйте класс Function1 в HttpExample.

1. В методе `HttpTrigger` с именем `Run` переименуйте атрибут `FunctionName` в `HttpExample`.

После переименования функцию можно протестировать на локальном компьютере.

## <a name="run-the-function-locally"></a>Локальное выполнение функции

Visual Studio интегрируется с Azure Functions Core Tools. Поэтому вы можете тестировать функции локально, используя полную среду выполнения Функций Azure.  

[!INCLUDE [functions-run-function-test-local-vs](../../includes/functions-run-function-test-local-vs.md)]

Убедившись, что функция выполняется правильно на локальном компьютере, опубликуйте проект в Azure.

## <a name="publish-the-project-to-azure"></a>Публикация проекта в Azure

Перед публикацией проекта убедитесь, что в вашей подписке Azure есть приложение-функция. Для первой публикации проекта в Visual Studio создается приложение-функция.

[!INCLUDE [Publish the project to Azure](../../includes/functions-vstools-publish.md)]

## <a name="test-your-function-in-azure"></a>Тестирование функции в Azure

1. В Cloud Explorer приложения-функция уже должно быть выбрано. Если это не так, разверните свою подписку и выберите пункт **Службы приложений**, затем выберите новое приложение-функцию.

1. Щелкните правой кнопкой мыши приложение-функцию и выберите пункт **Открыть в браузере**. В веб-браузере по умолчанию откроется корень приложения-функции и отобразится страница, где указано, что приложение-функция работает. 

    :::image type="content" source="media/functions-create-your-first-function-visual-studio/function-app-running-azure.png" alt-text="Приложение-функция работает":::

1. В адресной строке браузера добавьте строку `/api/HttpExample?name=Functions` к базовому URL-адресу и выполните запрос.

    URL-адрес для вызова функции, активируемой HTTP-запросом, указывается в следующем формате:

    `http://<APP_NAME>.azurewebsites.net/api/HttpExample?name=Functions`

2. Перейдите по этому URL-адресу и увидите в браузере ответ на удаленный запрос GET, возвращаемый функцией. Он выглядит аналогично следующему примеру:

    :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-create-your-first-function-visual-studio-browser-azure.png" alt-text="Ответ функции в браузере":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Другие краткие руководства в этой коллекции созданы на основе этого документа. Если вы планируете работать с последующими краткими руководствами или с любыми службами, созданными при работе с этим руководством, не очищайте ресурсы.

Под *ресурсами* в Azure подразумеваются приложения-функции, функции, учетные записи хранения и т. д. Они объединяются в *группы ресурсов*, при удалении которых удаляются и все данные в них. 

Вы создали ресурсы для завершения этих кратких руководств. Вам могут быть выставлены счета за эти ресурсы в зависимости от [состояния учетной записи](https://azure.microsoft.com/account/) и [цен на службы](https://azure.microsoft.com/pricing/). Если вам больше не нужны ресурсы, их можно удалить следующим образом:

1. В Cloud Explorer разверните свою подписку и выберите пункт **Службы приложений**, щелкните правой кнопкой мыши приложение-функцию и выберите пункт **Открыть на портале**. 

1. На странице приложения-функции выберите вкладку **Обзор**, а затем щелкните ссылку в разделе **Группа ресурсов**.

   :::image type="content" source="media/functions-create-your-first-function-visual-studio/functions-app-delete-resource-group.png" alt-text="Выбор группы ресурсов, которую требуется удалить со страницы приложения-функции":::

2. На странице **Группа ресурсов** просмотрите список включенных ресурсов и убедитесь, что именно их нужно удалить.
 
3. Выберите **Удалить группу ресурсов** и следуйте инструкциям.

   Удаление может занять несколько минут. После этого на несколько секунд появится уведомление. Кроме того, можно выбрать значок колокольчика в верхней части страницы, чтобы просмотреть уведомление.

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы с помощью Visual Studio создали и опубликовали в Azure приложение-функцию C# с простой функцией, активируемой HTTP-запросом. 

Перейдите к следующей статье, чтобы узнать, как добавить привязку очереди службы хранилища Azure к этой функции:
> [!div class="nextstepaction"]
> [Connect functions to Azure Storage using Visual Studio Code](functions-add-output-binding-storage-queue-vs.md) (Подключение функций к службе хранилища Azure с помощью Visual Studio Code)

