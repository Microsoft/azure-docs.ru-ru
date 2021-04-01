---
title: Руководство. Проверка подлинности пользователей в собственном клиентском приложении
titleSuffix: Azure AD B2C
description: Руководство по предоставлению пользователю данных для входа в классическое приложение .NET с помощью Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 10/12/2019
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 06d27c3a3daa4702653a2063d0ac70fd094e2d74
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78186205"
---
# <a name="tutorial-authenticate-users-in-a-native-desktop-client-using-azure-active-directory-b2c"></a>Руководство. Проверка подлинности пользователей в собственном настольном клиенте с помощью Azure Active Directory B2C

В этом руководстве показано, как использовать Azure Active Directory (Azure AD) B2C для входа и регистрации пользователей в классическом приложении Windows Presentation Foundation (WPF). Azure AD B2C позволяет приложениям выполнять проверку подлинности учетных записей социальных сетей, корпоративных учетных записей и учетных записей Azure Active Directory с помощью стандартных протоколов.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * добавление собственного клиентского приложения;
> * настройка примера для использования приложения;
> * регистрация с помощью потока пользователя.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- [Создайте потоки пользователя](tutorial-create-user-flows.md), чтобы обеспечить взаимодействие с пользователями в приложении.
- Установите [Visual Studio 2019](https://www.visualstudio.com/downloads/) с рабочими нагрузками **Разработка классических приложений .NET** и **ASP.NET и веб-разработка**.

## <a name="add-the-native-client-application"></a>добавление собственного клиентского приложения;

[!INCLUDE [active-directory-b2c-appreg-native](../../includes/active-directory-b2c-appreg-native.md)]

Запишите значение параметра **Идентификатор приложения (клиент)** . Оно вам потребуется в дальнейшем.

## <a name="configure-the-sample"></a>Настройка примера

С помощью этого руководства вы настроите пример, который можно скачать из репозитория GitHub. Пример классического приложения WPF демонстрирует регистрацию, вход в систему и может вызывать защищенный веб-API в Azure AD B2C. [Скачайте ZIP-файл](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop/archive/msalv3.zip), [просмотрите репозиторий](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) или клонируйте пример с GitHub.

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop.git
```

Чтобы обновить приложение для работы с арендатором Azure AD B2C и вызвать его потоки пользователей, а не потоки в демонстрационном арендаторе по умолчанию, сделайте следующее.

1. Откройте решение **active-directory-b2c-wpf** (`active-directory-b2c-wpf.sln`) в Visual Studio.
2. В проекте **active-directory-b2c-wpf** откройте файл *App.xaml.cs* и найдите следующие определения переменных. Замените `{your-tenant-name}` именем своего арендатора Azure AD B2C, а `{application-ID}` — идентификатором приложения, записанным ранее.

    ```csharp
    private static readonly string Tenant = "{your-tenant-name}.onmicrosoft.com";
    private static readonly string AzureAdB2CHostname = "{your-tenant-name}.b2clogin.com";
    private static readonly string ClientId = "{application-ID}";
    ```

3. Обновите переменные имени политики, указав имена потоков пользователей, созданных в ходе выполнения предварительных требований. Пример:

    ```csharp
    public static string PolicySignUpSignIn = "B2C_1_signupsignin1";
    public static string PolicyEditProfile = "B2C_1_profileediting1";
    public static string PolicyResetPassword = "B2C_1_passwordreset1";
    ```

## <a name="run-the-sample"></a>Запуск примера

Нажмите клавишу **F5**, чтобы выполнить сборку и запустить пример.

### <a name="sign-up-using-an-email-address"></a>Регистрация с помощью адреса электронной почты

1. Выберите **Войти**, чтобы зарегистрироваться как пользователь. Используется поток пользователя **B2C_1_signupsignin1**.
2. Появляется страница входа в Azure AD B2C со ссылкой **Зарегистрироваться сейчас**. Щелкните ссылку **Зарегистрироваться сейчас**, так как у вас пока нет учетной записи.
3. В рамках рабочего процесса регистрации появляется страница для сбора данных и проверки личности пользователя с использованием адреса электронной почты. Рабочий процесс регистрации также собирает пароль пользователя и запрашиваемые атрибуты, определенные в потоке пользователя.

    Используйте действительный адрес электронной почты и подтвердите его с помощью кода проверки. Задайте пароль. Введите значения запрашиваемых атрибутов.

    ![Страница регистрации, отображаемая как часть рабочего процесса входа и регистрации](./media/tutorial-desktop-app/azure-ad-b2c-sign-up-workflow.png)

4. Выберите **Создать**, чтобы создать локальную учетную запись в клиенте Azure AD B2C.

Пользователь теперь может использовать свой адрес электронной почты для входа в классическое приложение и работы с ним. После успешной регистрации или входа в систему сведения о маркере отображаются в нижней части приложения WPF.

![Сведения о маркере, отображаемые в нижней части классического приложения WPF](./media/tutorial-desktop-app/desktop-app-01-post-signin.png)

При нажатии кнопки **вызова API** отображается **сообщение об ошибке**. Вы столкнулись с ошибкой, так как в своем текущем состоянии приложение пытается получить доступ к API, защищенному демонстрационным арендатором `fabrikamb2c.onmicrosoft.com`. Так как маркер доступа действителен только для вашего арендатора Azure AD B2C, вызов API считается неавторизованным.

Перейдите к следующему учебнику, чтобы зарегистрировать защищенный веб-API в собственном арендаторе и включить кнопку **вызова API**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * добавление собственного клиентского приложения;
> * настройка примера для использования приложения;
> * регистрация с помощью потока пользователя.

Затем, чтобы включить кнопку **вызова API**, предоставьте классическому приложению WPF доступ к веб-API, зарегистрированному в вашем собственном арендаторе Azure AD B2C:

> [!div class="nextstepaction"]
> [Руководство. Предоставление доступа к веб-API Node.js из классического приложения >](tutorial-desktop-app-webapi.md)
