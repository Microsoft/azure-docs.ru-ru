---
title: Руководство по включению проверки подлинности в веб-приложении Python
titleSuffix: Azure AD B2C
description: Из этого руководства вы узнаете, как с помощью Azure Active Directory B2C обеспечить вход пользователя в веб-приложение Python Flask.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 06/12/2020
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.custom: devx-track-python
ms.openlocfilehash: 56e92a698446bd417c74820d6da662ad9ee55c77
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100555683"
---
# <a name="tutorial-enable-authentication-in-a-python-web-application-with-azure-ad-b2c"></a>Руководство по включению проверки подлинности в веб-приложении Python с помощью Azure AD B2C

В этом руководстве показано, как использовать Azure Active Directory B2C (Azure AD B2C) для регистрации и выполнения входа пользователей в веб-приложении Python Flask.

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * Добавление URL-адреса ответа в приложение, зарегистрированное в клиенте Azure AD B2C.
> * Скачивание примера кода из GitHub.
> * Изменение кода примера приложения для работы с вашим клиентом.
> * Регистрация с помощью потока пользователя для регистрации и выполнения входа.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для продолжения выполнения действий, описанных в этом учебнике, необходимы следующие ресурсы Azure AD B2C:

* [клиент Azure AD B2C](tutorial-create-tenant.md);
* [приложение, зарегистрированное в клиенте](tutorial-register-applications.md), а также его *идентификатор приложения (клиента)* и *секрет клиента*;
* [созданные маршруты пользователей](tutorial-create-user-flows.md) в клиенте.

Кроме того, вам понадобятся следующие компоненты в локальной среде разработки:

* [Visual Studio Code](https://code.visualstudio.com/) или любой другой редактор кода.
* [Python](https://nodejs.org/en/download/) версии 2.7 и выше или 3 и выше.

## <a name="add-a-redirect-uri"></a>Добавление URI перенаправления

При прохождении второго учебника для выполнения предварительных условий вы зарегистрировали веб-приложение в Azure AD B2C. Для обеспечения взаимодействия с примером из этого руководства необходимо добавить URL ответа (также называется URI перенаправления) в регистрацию приложения.

Чтобы обновить приложение в клиенте Azure AD B2C, можно использовать новый унифицированный интерфейс **Регистрация приложений** или устаревший интерфейс **Приложения (прежняя версия)** . [См. дополнительные сведения о новом интерфейсе](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Регистрация приложений](#tab/app-reg-ga/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. В меню слева выберите **Azure AD B2C**. Либо щелкните **Все службы**, а затем найдите и выберите **Azure AD B2C**
1. Выберите **Регистрация приложений**, щелкните вкладку **Собственные приложения**, а затем выберите приложение *webapp1*.
1. В разделе **Управление** выберите **Проверка подлинности**.
1. В разделе **Веб** щелкните ссылку **Добавить URI**, а затем введите значение `http://localhost:5000/getAToken` в текстовом поле.
1. Щелкните **Сохранить**.

#### <a name="applications-legacy"></a>[Приложения (прежние версии)](#tab/applications-legacy/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. Щелкните пункт **Приложения (прежняя версия)** , а затем выберите приложение *webapp1*.
1. В разделе **URL-адрес ответа** добавьте `http://localhost:5000/getAToken`.
1. Щелкните **Сохранить**.
* * *

## <a name="get-the-sample-code"></a>Получение кода примера

С помощью этого руководства вы настроите пример кода, который можно скачать из репозитория GitHub, для работы с клиентом B2C. В этом примере показано, как использовать Azure AD B2C в веб-приложении Python Flask для регистрации и входа пользователя.

[Скачайте архив в формате ZIP](https://github.com/Azure-Samples/ms-identity-python-webapp/archive/master.zip) или клонируйте [репозиторий с примером кода](https://github.com/Azure-Samples/ms-identity-python-webapp) с сайта GitHub.

```console
git clone https://github.com/Azure-Samples/ms-identity-python-webapp.git
```

## <a name="update-the-sample"></a>Обновление примера

Получив нужный пример, настройте в приложении работу с клиентом Azure AD B2C, регистрацией приложения и потоками пользователей.

В корневом каталоге проекта сделайте следующее:

1. Переименуйте файл *app_config.py* в *app_config.py.OLD*.
1. Переименуйте файл *pp_config_b2c.py* в *app_config.py*.

Обновите новый файл *app_config.ry*, указав значения зарегистрированного клиента Azure AD B2C и регистрации приложения, созданные при выполнении предварительных требований.

1. Откройте файл *app_config.py* в любом редакторе.
1. Замените значение `b2c_tenant` именем клиента Azure AD B2C, например *contosob2c*.
1. Обновите все вхождения значений `*_user_flow` так, чтобы они соответствовали именам потоков пользователей, созданных при выполнении предварительных требований.
1. Замените значение `CLIENT_ID` **идентификатором приложения (клиента)** для веб-приложения, которое вы зарегистрировали при выполнении предварительных требований.
1. Замените значение `CLIENT_SECRET` значением **секрета клиента**, которое вы создали при выполнении предварительных требований. Для повышения безопасности желательно применить для хранения этого значения **переменную среды** как предлагается в комментариях.

Теперь верхняя часть файла *app_config.py* должна выглядеть примерно так, как следующий фрагмент кода:

```python
import os

b2c_tenant = "contosob2c"
signupsignin_user_flow = "B2C_1_signupsignin1"
editprofile_user_flow = "B2C_1_profileediting1"
resetpassword_user_flow = "B2C_1_passwordreset1"
authority_template = "https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{user_flow}"

CLIENT_ID = "11111111-1111-1111-1111-111111111111" # Application (client) ID of app registration

CLIENT_SECRET = "22222222-2222-2222-2222-222222222222" # Placeholder - for use ONLY during testing.
# In a production app, we recommend you use a more secure method of storing your secret,
# like Azure Key Vault. Or, use an environment variable as described in Flask's documentation:
# https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables
# CLIENT_SECRET = os.getenv("CLIENT_SECRET")
# if not CLIENT_SECRET:
#     raise ValueError("Need to define CLIENT_SECRET environment variable")
```

> [!WARNING]
> Как указано в комментариях к фрагменту кода, **мы не рекомендуем хранить секреты в открытом тексте** в коде приложения. Жестко закодированная переменная в этом примере кода используется *исключительно для удобства*. Постарайтесь применить переменную среды или хранилище секретов, например Azure Key Vault.

## <a name="run-the-sample"></a>Запуск примера

1. В окне консоли или терминала перейдите к каталогу, где размещен этот пример. Пример:

    ```console
    cd ms-identity-python-webapp
    ```
1. Выполните следующие команды, чтобы установить необходимые пакеты из PyPi и запустить веб-приложение на локальном компьютере:

    ```console
    pip install -r requirements.txt
    flask run --host localhost --port 5000
    ```

    В окне консоли отображается номер порта приложения, выполняющегося локально:

    ```console
     * Serving Flask app "app" (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: off
     * Running on http://localhost:5000/ (Press CTRL+C to quit)
    ```

1. Перейдите по адресу `http://localhost:5000`, чтобы просмотреть веб-приложение, выполняемое на локальном компьютере.

    :::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-01.png" alt-text="Окно веб-браузера, где отображается локально выполняющееся веб-приложение Python Flask":::

### <a name="sign-up-using-an-email-address"></a>Регистрация с помощью адреса электронной почты

Этот пример приложения поддерживает регистрацию, вход и сброс пароля. При работе с этим руководством вы зарегистрируетесь в приложении с помощью адреса электронной почты.

1. Щелкните **Войти**, чтобы инициировать поток пользователя *B2C_1_signupsignin1*, указанный на предыдущем шаге.
1. Отобразится страница входа в Azure AD B2C со ссылкой для регистрации. Щелкните ссылку **Зарегистрироваться сейчас**, так как у вас пока нет учетной записи.
1. В рамках рабочего процесса регистрации отобразится страница для сбора данных и проверки личности пользователя с использованием адреса электронной почты. Рабочий процесс регистрации также собирает пароль пользователя и запрашиваемые атрибуты, определенные в потоке пользователя.

    Используйте действительный адрес электронной почты и подтвердите его с помощью кода проверки. Задайте пароль. Введите значения запрашиваемых атрибутов.

    :::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-02.png" alt-text="Страница регистрации, отображаемая потоком пользователя Azure AD B2C":::

1. Щелкните **Создать**, чтобы создать локальную учетную запись в каталоге Azure AD B2C.

При нажатии кнопки **Создать** приложение покажет имя выполнившего вход пользователя.

:::image type="content" source="media/tutorial-web-app-python/python-flask-web-app-03.png" alt-text="Окно веб-браузера, где отображается веб-приложение Python Flask после входа пользователя":::

Если вы хотите протестировать возможность входа, щелкните ссылку **Logout** (Выйти), а затем **Sign In** (Войти) и введите адрес электронной почты и пароль, которые вы указали при регистрации.

## <a name="next-steps"></a>Дальнейшие действия

С помощью инструкций из этого руководства вы настроили в приложении Python Flask работу с потоком пользователя из клиента Azure AD B2C, обеспечив поддержку регистрации и входа. Вы сделали следующее:

> [!div class="checklist"]
> * добавили URL-адрес ответа в приложение, зарегистрированное в клиенте Azure AD B2C;
> * скачали пример кода из GitHub;
> * изменили код примера приложения для работы с вашим арендатором;
> * выполнили регистрацию с помощью своего потока пользователя для регистрации и выполнения входа.

Теперь узнайте, как можно настроить пользовательский интерфейс или страницы потока пользователя, которые отображает Azure AD B2C.

> [!div class="nextstepaction"]
> [Настройка удобного для пользователей интерфейса в Azure Active Directory B2C >](customize-ui.md)