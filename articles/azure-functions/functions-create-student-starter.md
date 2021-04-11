---
title: Создание функции с помощью службы "Начальный набор Microsoft Azure для учащихся"
description: Узнайте, как создать функцию Azure в подписке на службу "Начальный набор Microsoft Azure для учащихся"
ms.topic: how-to
ms.date: 04/29/2020
ms.openlocfilehash: 3ca91999182e717500eacfa73dda834e9cf89562
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106067152"
---
# <a name="create-a-function-using-azure-for-students-starter"></a>Создание функции с помощью службы "Начальный набор Microsoft Azure для учащихся"

В этом руководстве описывается, как создать функцию HTTP Hello World в подписке на службу "Начальный набор Microsoft Azure для учащихся". Мы также рассмотрим доступные возможности в Функциях Azure для этого типа подписки.

*Начальный набор Microsoft Azure для учащихся* предоставляет бесплатный доступ к ресурсам для начала работы с продуктами Azure, которые необходимо разработать в облаке. [Узнайте больше об этом предложении здесь.](https://azure.microsoft.com/offers/ms-azr-0144p/)

Решение "Функции Azure" позволяет выполнять код в [бессерверной](https://azure.microsoft.com/solutions/serverless/) среде без необходимости создавать виртуальную машину или публиковать веб-приложение. [Дополнительные сведения о Функциях Azure см. здесь.](./functions-overview.md)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-function"></a>Создание функции

 В этой статье описано, как создать функцию Hello World для триггеров HTTP на портале Azure с помощью Функций Azure.

![Создание приложения-функции на портале Azure](./media/functions-create-student-starter/function-app-in-portal-editor.png)

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com) с помощью своей учетной записи Azure.

## <a name="create-a-function-app"></a>Создание приложения-функции

Для выполнения функций вам понадобится приложение-функция, позволяющее группировать функции в логические единицы и упростить развертывание, масштабирование и совместное использование ресурсов, а также управление ими.

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

Затем создайте функцию в новом приложении-функции.

## <a name="create-an-http-trigger-function"></a><a name="create-function"></a>Создание функции, активируемой HTTP

1. В меню слева в окне **Функции** выберите **Функции**, а затем в верхнем меню выберите **Добавить**. 
 
1. В окне **Новая функция** выберите **Триггер HTTP**.

    ![Выбор функции, активируемой HTTP](./media/functions-create-student-starter/function-app-select-http-trigger.png)

1. В окне **Новая функция** для пункта **Новая функция** оставьте имя по умолчанию или введите новое имя. 

1. Из раскрывающегося списка **Уровень авторизации** выберите **Анонимный**, а затем выберите **Создать функцию**.

    Azure создает функцию, активируемую HTTP. Теперь вы можете запустить новую функцию, отправив HTTP-запрос.

## <a name="test-the-function"></a>Проверка функции

1. В новой функции, активируемой HTTP, выберите **Code + Test** (Код + Тест) в меню слева, а затем в верхнем меню выберите **Получить URL-адрес функции**.

    ![Выбор "Получить URL-адрес функции"](./media/functions-create-student-starter/function-app-select-get-function-url.png)

1. В диалоговом окне **Получить URL-адрес функции** в раскрывающемся списке выберите вариант **по умолчанию**, а затем выберите значок **Копировать в буфер обмена**. 

    ![Копирование URL-адреса функции с портала Azure](./media/functions-create-student-starter/function-app-develop-tab-testing.png)

1. Вставьте URL-адрес функции в адресную строку браузера. Добавьте значение строки запроса `?name=<your_name>` в конец этого URL-адреса и нажмите клавишу Enter, чтобы выполнить этот запрос. 

    Следующий пример демонстрирует ответ в браузере:

    ![Ответ функции в браузере.](./media/functions-create-student-starter/function-app-browser-testing.png)

    URL-адрес запроса включает ключ, который по умолчанию необходим для доступа к функции по протоколу HTTP.

1. При выполнении функции сведения о трассировке записываются в журналы. Чтобы просмотреть выходные данные трассировки, вернитесь на страницу **Code + Test** (Код + Тест) на портале и разверните список **Журналы** в нижней части страницы.

   ![Средство просмотра журналов Функций на портале Azure.](./media/functions-create-student-starter/function-view-logs.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Clean-up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="supported-features-in-azure-for-students-starter"></a>Поддерживаемые функции в службе "Начальный набор Microsoft Azure для учащихся"

В службе "Начальный набор Microsoft Azure для учащихся" вы получите доступ к большинству функций среды выполнения Функций Azure, принимая во внимание несколько основных ограничений, перечисленных ниже:

* Триггер HTTP является единственным поддерживаемым типом триггера.
    * Поддерживаются все входные и выходные привязки! [Полный список см. здесь.](functions-triggers-bindings.md)
* Поддерживаемые языки: 
    * C# (.NET Core 2);
    * JavaScript (Node.js 8 и 10)
    * F# (.NET Core 2).
    * [Поддерживаемые языки в планах высшего уровня см. здесь](supported-languages.md)
* Windows — единственная поддерживаемая операционная система.
* Масштаб будет ограничен [до одного бесплатного экземпляра уровня](https://azure.microsoft.com/pricing/details/app-service/windows/), выполняющегося ежедневно на протяжении не больше 60 минут. При получении HTTP-трафика вы сможете автоматически увеличить масштаб от 0 до 1 экземпляра без сервера, но не больше.
* Поддерживается среда выполнения функций только [версии 2.x и выше](functions-versions.md).
* Для редактирования и публикации функций поддерживаются все средства разработки. В их состав входят: VS Code, Visual Studio, Azure CLI и портал Azure. Если вы хотите использовать что-то кроме портала, необходимо сначала создать приложение на портале, а затем выбрать это приложение в качестве цели развертывания в предпочитаемом средстве.

## <a name="next-steps"></a>Дальнейшие действия

Вы завершили создание приложения-функции с простой функцией для триггеров HTTP. Теперь вы можете изучить локальные средства, дополнительные языки, мониторинг и интеграции.

 * [Создание первой функции с помощью Visual Studio](./functions-create-your-first-function-visual-studio.md)
 * [Создание функции в Azure с помощью Visual Studio Code](./create-first-function-vs-code-csharp.md)
 * [Руководство разработчика JavaScript для Функций Azure](./functions-reference-node.md)
 * [Подключение к базе данных SQL Azure с помощью Функций Azure](./functions-scenario-database-table-cleanup.md)
 * [Дополнительные сведения о привязках HTTP в Функциях Azure см. здесь](./functions-bindings-http-webhook.md).
 * [Мониторинг Функций Azure](./functions-monitoring.md)